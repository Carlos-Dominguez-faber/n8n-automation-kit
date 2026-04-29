# Coolify MCP — Gestión de Microservicios

## Herramientas disponibles vía Coolify MCP

| Tool | Qué hace | Parámetro clave |
|---|---|---|
| `list_applications` | Lista todas las apps desplegadas | — |
| `get_application` | Detalle de una app (status, URL, env) | `uuid` |
| `restart_application` | Reinicia sin redeploy | `uuid` |
| `redeploy_application` | Pull + rebuild + restart | `uuid` |
| `update_application_env` | Cambia variables de entorno | `uuid`, `key`, `value` |
| `get_application_logs` | Logs recientes del contenedor | `uuid` |
| `list_services` | Lista servicios (Supabase, Redis, etc.) | — |
| `get_service` | Detalle de un servicio | `uuid` |

## Patrón: verificar antes de llamar desde n8n

Antes de que un workflow de n8n llame a un microservicio via HTTP:

```
1. list_applications() → encontrar el uuid del servicio
2. get_application({uuid}) → verificar status: "running" | "stopped" | "error"
3. Si STOPPED → restart_application({uuid}) → esperar status: "running"
4. Si ERROR → get_application_logs({uuid}) → diagnóstico antes de reintentar
5. Solo si RUNNING → el workflow puede hacer el HTTP Request de forma segura
```

**Cómo pedírselo a Claude:**
```
"Verifica que el servicio ffmpeg-merger está corriendo en Coolify
 antes de ejecutar el workflow de generación de reel"
```

## Microservicios típicos en el stack Dopla

### ffmpeg-merger-dopla
- **Propósito:** Mezcla de audio/video para reels (MP4 + MP3 → MP4)
- **URL externa:** `https://ffmpeg.dopla.app` (Coolify con dominio asignado)
- **Puerto interno:** 3000
- **Señales de error en logs:**
  - `ECONNREFUSED` → el microservicio no puede conectar a otro servicio
  - `Storage upload failed` → credenciales de Supabase Storage incorrectas
  - `timeout` → n8n tardó más de 120s en responder al callback

### n8n (el mismo n8n como microservicio)
- Coolify MCP puede reiniciar n8n si se cuelga
- Útil cuando workflows largan excepciones no manejadas y dejan ejecuciones zombie

## Gestión de variables de entorno

Para actualizar una variable de entorno de un microservicio sin redeploy completo:

```
"Actualiza la variable SUPABASE_SERVICE_KEY del servicio ffmpeg-merger
 con el nuevo valor [valor]"
```

Claude invoca:
```
update_application_env({
  uuid: "[app-uuid]",
  key: "SUPABASE_SERVICE_KEY",
  value: "[nuevo-valor]"
})
```

> ⚠️ Algunos servicios requieren restart después de cambiar env vars.
> Usar `restart_application` inmediatamente después de `update_application_env`.

## Debug con logs en tiempo real

Señales de alarma a buscar en `get_application_logs`:
- `Error: ECONNREFUSED 127.0.0.1:5432` → PostgreSQL no accesible
- `401 Unauthorized` → API key del webhook de n8n incorrecta
- `SIGTERM received` → Coolify reinició el contenedor (health check timeout)
- `Out of memory` → el servicio necesita más RAM en el VPS

## Variables de entorno críticas para microservicios en Coolify

```bash
# Estándar para microservicios que se conectan a n8n y Supabase
N8N_WEBHOOK_URL=https://pod-1.dopla.app/webhook/[path]
SUPABASE_URL=https://[project-ref].supabase.co
SUPABASE_SERVICE_KEY=[service-role-key]
STORAGE_BUCKET=generated-media
PORT=3000

# Para callbacks de n8n (seguridad)
WEBHOOK_SECRET=[shared-secret-con-n8n]
```

> NUNCA pasar API keys de tenants en variables globales del microservicio.
> El tenant_id se pasa en el body del request → el microservicio resuelve internamente.
