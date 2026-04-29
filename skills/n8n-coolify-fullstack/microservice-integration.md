# Microservicios en Coolify — Integración con n8n

## El principio: n8n orquesta, el microservicio ejecuta

n8n NO hace el trabajo pesado (procesamiento de video, conversión de archivos, cálculos intensivos).
n8n ORQUESTA: llama al microservicio, espera el resultado, continúa con el output.

```
n8n Workflow
    │
    ├── Preparar payload (datos, rutas de archivos, credenciales del tenant)
    ├── [Verificar microservicio corriendo via Coolify MCP]
    ├── HTTP Request → microservicio (POST /process)
    │       ↑ Puede tardar 30-120s para video processing
    ├── Recibir resultado {status, output_url, metadata}
    └── Continuar: actualizar Supabase, notificar callback, descontar créditos
```

---

## Caso real: ffmpeg-merger en Dopla

### Payload que n8n envía al microservicio

```json
{
  "video_path": "s3://generated-media/tenant-abc/reel-raw.mp4",
  "audio_path": "s3://generated-media/tenant-abc/narration.mp3",
  "output_name": "reel-final-content-id-xyz.mp4",
  "tenant_id": "tenant-abc",
  "callback_url": "https://pod-1.dopla.app/webhook/ffmpeg/callback",
  "callback_secret": "[shared-hmac-secret]"
}
```

### Respuesta que n8n recibe

```json
{
  "job_id": "job_abc123",
  "status": "processing",
  "estimated_seconds": 45
}
```

Y luego, cuando termina, el microservicio llama el callback:

```json
{
  "job_id": "job_abc123",
  "status": "completed",
  "output_url": "https://storage.dopla.app/generated-media/tenant-abc/reel-final-xyz.mp4",
  "processing_time_ms": 38420,
  "file_size_bytes": 24891234
}
```

---

## Los 3 patrones de comunicación

### Patrón 1: Síncrono (para procesos < 30 segundos)

```
n8n → HTTP Request (POST /process, timeout 60s)
      [espera...]
      ← respuesta directa con resultado
n8n → continúa con el output
```

**Configuración del HTTP Request node:**
```json
{
  "url": "https://microservicio.tudominio.com/process",
  "method": "POST",
  "timeout": 60000,
  "body": "={{ JSON.stringify($json) }}"
}
```

**Cuándo usar:** conversiones de imágenes, llamadas a APIs rápidas, transformaciones de texto.

### Patrón 2: Asíncrono con callback (para procesos > 30 segundos) ← Dopla usa este

```
n8n → POST /start-job → recibe job_id inmediato
      [microservicio procesa en background]
      [microservicio llama POST al webhook de n8n cuando termina]
n8n webhook callback → recibe resultado → continúa el flujo
```

**Por qué:** n8n tiene timeout de ~120s en HTTP Request. Video processing puede tardar 2-5 min.

**Implementación en n8n:**
```
Nodo 1: HTTP Request → POST /start-job → {job_id}
Nodo 2: respondToWebhook {status: 202, job_id} ← avisa al cliente que está procesando
Nodo 3: [workflow termina — el callback lo reanuda]

Workflow de callback (trigger: Webhook /ffmpeg/callback):
Nodo 1: Webhook → recibe {job_id, status, output_url}
Nodo 2: Verificar firma HMAC (seguridad)
Nodo 3: HTTP Request → Supabase → actualizar content con output_url
Nodo 4: HTTP Request → Supabase → descontar créditos
Nodo 5: HTTP Request → dopla.app/api/n8n/callback → notificar frontend
```

### Patrón 3: Polling (cuando el microservicio no puede hacer callback)

```
n8n → POST /start-job → job_id
n8n → Wait node (30s)
n8n → GET /job/{job_id}/status
      ↳ Si "processing" → Wait (30s) → GET de nuevo (max 5 intentos)
      ↳ Si "completed" → continuar con resultado
      ↳ Si "failed" → manejar error
```

**Cuándo usar:** microservicios legacy que no soportan callbacks, o APIs externas sin webhooks.

---

## Autenticación microservicio → n8n (callback seguro)

Para verificar que el callback viene del microservicio real (no de un ataque):

```javascript
// En el microservicio (envía HMAC en header):
const signature = crypto.createHmac('sha256', WEBHOOK_SECRET)
  .update(JSON.stringify(payload))
  .digest('hex');
headers['X-Webhook-Signature'] = `sha256=${signature}`;

// En n8n (Code node para verificar):
const receivedSig = $json.headers['x-webhook-signature'];
const expectedSig = 'sha256=' + crypto.createHmac('sha256', $env.WEBHOOK_SECRET)
  .update(JSON.stringify($json.body))
  .digest('hex');

if (receivedSig !== expectedSig) {
  throw new Error('Invalid webhook signature');
}
return [{json: $json.body}];
```

---

## Networking en Coolify: interno vs externo

### Red interna Docker (misma red en Coolify)

Si n8n y el microservicio están en el mismo proyecto/network de Coolify:

```bash
# Hostname interno — no sale a internet, más rápido
http://[container-name]:3000

# Encontrar el container name:
# Coolify UI → recurso → Settings → Container Name
# O SSH al VPS: docker ps --format '{{.Names}}'
```

**Ventajas:** Más rápido (no pasa por Traefik), sin latencia de DNS, sin SSL overhead.
**Cómo usarlo en n8n:** HTTP Request node → URL: `http://ffmpeg-merger-dopla:3000/process`

### URL externa (diferentes pods o necesitas HTTPS)

```bash
# URL con dominio asignado en Coolify
https://ffmpeg.tudominio.com/process

# Con autenticación básica si el servicio lo requiere
# Agregar header: Authorization: Basic [base64(user:password)]
```

---

## Variables de entorno estándar (patrón recomendado)

```bash
# En el microservicio — variables requeridas
N8N_WEBHOOK_CALLBACK_URL=https://pod-1.dopla.app/webhook/[servicio]/callback
WEBHOOK_SECRET=[shared-secret-igual-en-n8n]
SUPABASE_URL=https://[project-ref].supabase.co
SUPABASE_SERVICE_KEY=[service-role-key]
STORAGE_BUCKET=generated-media
PORT=3000

# NUNCA agregar como variable global del microservicio:
# - API keys de tenants individuales
# - Tokens de acceso de usuarios
# Esos siempre van en el BODY del request, nunca en env vars compartidas
```

---

## Deploy de microservicio nuevo via Coolify MCP

```
// Pedirle a Claude:
"Deploya el nuevo microservicio subtitle-generator desde el repo
 github.com/Carlos/subtitle-generator en Coolify, expuesto en
 subtitles.dopla.app, con estas variables de entorno: [lista]"

// Claude invoca Coolify MCP:
1. create_application({
     name: "subtitle-generator",
     repository: "github.com/Carlos/subtitle-generator",
     branch: "main"
   })
2. update_application_env({uuid, key, value}) para cada env var
3. redeploy_application({uuid})
4. get_application({uuid}) → verificar status: "running"
5. get_application_logs({uuid}) → verificar no hay errores de inicio
```

---

## Health check de microservicios antes de producción

```
"Antes de activar el workflow Router Dopla, verifica que todos los
 microservicios que usa están corriendo y sanos"

// Claude invoca Coolify MCP en secuencia:
1. list_applications() → encontrar todos los servicios del proyecto Dopla
2. get_application(ffmpeg-merger) → status: running ✅
3. get_application(subtitle-generator) → status: stopped ❌ → restart
4. [Esperar 10s]
5. get_application(subtitle-generator) → status: running ✅
6. Confirmar: todos los microservicios en RUNNING antes de activar el Router
```
