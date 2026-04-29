# Sub-Workflows — Arquitectura Modular en n8n

## El patrón Dopla: Router → Sub-workflows por tipo

```
Webhook (POST /dopla/content/generate)
    │
    ├── Validar entrada (IF nodes: tenant_id, content_id, tipo_contenido)
    ├── respondToWebhook {status: 202} ← responde rápido, procesa async
    ├── Enriquecer datos (HTTP Request → Supabase: leer tenant credentials)
    ├── Switch por tipo_contenido
    │
    ├── [carousel]   → executeWorkflow → "Dopla — Contenido — Carrusel"
    ├── [post]       → executeWorkflow → "Dopla — Contenido — Post Estático"
    ├── [reel]       → executeWorkflow → "Dopla — Contenido — Reel"
    └── [reel_ugc]   → executeWorkflow → "Dopla — Contenido — Reel UGC"
```

---

## ¿Cuándo crear un sub-workflow?

**Crear sub-workflow cuando:**
- La lógica tiene más de 8 nodos Y es reutilizable en otros workflows
- El mismo proceso se invoca desde múltiples workflows diferentes
- Quieres poder testear y debuggear esa parte en completo aislamiento
- Es un tipo de contenido/proceso diferente con lógica propia (carousel ≠ reel)

**NO crear sub-workflow cuando:**
- Es una variante menor del flujo — usa Switch en su lugar
- Solo se usa una vez y tiene menos de 5 nodos
- La latencia de invocación es crítica (executeWorkflow añade ~200-500ms de overhead)
- El estado completo del padre necesita estar disponible en el hijo (es complejo pasar todo)

---

## Estructura obligatoria de un sub-workflow

```
Nodo 1: Execute Workflow Trigger
        ↑ NUNCA un Webhook — este es el único trigger válido para sub-workflows

Nodo 2: Validar parámetros recibidos (IF node)
        ↑ tenant_id presente, content_id válido, etc.
        ↑ Si falla: Set node con {status: "error", reason: "..."} y STOP

Nodo 3-N: Lógica específica del sub-workflow
           ↑ Todo lo que hace este sub

Nodo final: Set node — campos de retorno al padre
            ↑ status: "completed" | "error"
            ↑ output_url, credits_used, processing_time_ms
            ↑ error_message (si status === "error")
```

---

## Pasar datos padre → hijo

### Configuración del executeWorkflow node (padre)

```json
{
  "workflowId": "={{ $json.sub_workflow_id }}",
  "mode": "Single Run",
  "fields": {
    "values": [
      {
        "name": "tenant_id",
        "stringValue": "={{ $json.tenant_id }}",
        "type": "string"
      },
      {
        "name": "content_id",
        "stringValue": "={{ $json.content_id }}",
        "type": "string"
      },
      {
        "name": "brand_data",
        "stringValue": "={{ JSON.stringify($json.brand_data) }}",
        "type": "string"
      }
    ]
  }
}
```

> ⚠️ **CRÍTICO:** `type: "string"` es OBLIGATORIO en cada campo.
> Sin él, el campo llega como `undefined` en el hijo.
> Este es el error más común en sub-workflows multi-tenant.

### En el hijo: acceder a los datos

```javascript
// En el hijo, los datos llegan directamente en $json:
$json.tenant_id     // el tenant_id que pasó el padre
$json.content_id    // el content_id
$json.brand_data    // como string JSON → parsearlo con JSON.parse()
```

---

## Patrón multi-tenant: tenant_id fluye en TODOS los nodos

**REGLA INVIOLABLE:** En proyectos multi-tenant, el `tenant_id` debe estar disponible en todos y cada uno de los nodos del sub-workflow.

```
Execute Workflow Trigger
    │ tenant_id: "abc123"
    ▼
Validar tenant_id
    │ $json.tenant_id → "abc123"
    ▼
HTTP Request: Leer credenciales del tenant
    │ GET /tenants?id=abc123 → heygen_key, elevenlabs_key
    ▼
Nodo de procesamiento
    │ Usa las credenciales del tenant (nunca hardcodeadas)
    ▼
Set — Return
    │ status, output_url, credits_used
    ▼
```

Nunca hardcodear:
- URLs de APIs del tenant
- API keys del tenant
- IDs de proyectos del tenant

Siempre resolver desde la tabla `tenants` o `tenant_credentials` en Supabase.

---

## IDs de sub-workflows: hardcoded vs dinámico

**❌ Hardcoded (frágil):**
```json
{
  "workflowId": "wf_abc123def456"
}
```
Si el ID cambia (nueva instancia de n8n, import/export), el Router se rompe.

**✅ Dinámico desde Supabase (recomendado):**
```sql
-- Tabla: workflow_registry
CREATE TABLE workflow_registry (
  name TEXT PRIMARY KEY,
  workflow_id TEXT NOT NULL,
  environment TEXT DEFAULT 'production'
);
```

```javascript
// En el Router, antes del Switch:
// HTTP Request → GET /workflow_registry?name=eq.sub-carousel
// → usa $json.workflow_id en el executeWorkflow
```

---

## Naming conventions obligatorios

| Tipo | Patrón | Ejemplo |
|---|---|---|
| Router principal | `[Cliente] — [Proceso] — ROUTER` | `Dopla — Contenido — ROUTER` |
| Sub-workflow de proceso | `[Cliente] — [Proceso] — [Tipo]` | `Dopla — Contenido — Carrusel` |
| Sub-workflow de utilidad | `[Cliente] — UTIL — [Función]` | `Dopla — UTIL — Enriquecer Tenant` |
| Sub-workflow de error | `[Cliente] — ERROR — [Proceso]` | `Dopla — ERROR — Notificar Discord` |

---

## Debug de sub-workflows

### El sub-workflow falla pero no sé dónde

```javascript
// En Claude Code:
"Muéstrame las últimas 5 ejecuciones fallidas del sub-workflow
 'Dopla — Contenido — Reel' y dime qué nodo causó el fallo"

// Claude invoca:
n8n_executions({workflowId: "[sub-id]", status: "error", limit: 5})
// → Muestra el nodo que falló y su output/error
```

### Testear sub-workflow en aislamiento

```
1. En n8n UI → abrir el sub-workflow
2. Click en "Execute Workflow" (botón en el Trigger node)
3. El Execute Workflow Trigger acepta datos manuales para prueba:
   {
     "tenant_id": "test-tenant",
     "content_id": "test-content-id",
     "brand_data": "{\"name\":\"Test Brand\"}"
   }
4. Verificar el output del nodo Set final
5. Si funciona en aislamiento pero falla desde el Router:
   → El problema está en cómo el padre pasa los datos (type: "string" missing)
```

### Usar Playwright para ver la ejecución

```javascript
// Claude con Playwright MCP:
browser_navigate({url: "https://n8n.tudominio.com/workflow/[sub-id]/executions"})
browser_take_screenshot()
// → Ver el historial de ejecuciones visualmente
// → Click en la ejecución fallida para ver los datos de cada nodo
```

---

## Anti-patrones en sub-workflows

❌ **Sub-workflow que llama a otro sub-workflow** (>2 niveles de profundidad) — se vuelve imposible de debuggear
❌ **Pasar el $json completo del padre** — pasar solo lo que el hijo necesita
❌ **Sub-workflow con más de 15 nodos** — dividir en sub-sub o refactorizar
❌ **No validar entrada en el hijo** — el padre puede enviar datos inválidos
❌ **Hardcodear el workflowId** — usar tabla de registry
