# MCP vs API REST — Dos formas de conectar Claude Code con n8n

## El escenario

Claude Code puede interactuar con n8n de dos maneras fundamentalmente diferentes.
Ninguna es "mejor" — son complementarias y cada una tiene su escenario ideal.

---

## Método 1: n8n MCP (`n8n-mcp` vía npx)

### Cómo funciona

```
Claude Code
    │
    ▼ (protocolo MCP sobre stdio — proceso local)
n8n-mcp (proceso npx corriendo localmente)
    │
    ▼ (HTTP requests autenticados con API key)
n8n REST API
    │
    ▼
Tu instancia de n8n (ej: pod-1.dopla.app)
```

El MCP es un intermediario inteligente. No solo hace HTTP requests — entiende la
estructura de n8n. Sabe qué es un nodo, cómo se conectan, qué significa validar.
Es una capa de abstracción semántica sobre la API REST.

### Configuración mínima

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["-y", "n8n-mcp"],
      "env": {
        "N8N_API_URL": "https://TU_N8N_URL/",
        "N8N_API_KEY": "TU_API_KEY",
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true"
      }
    }
  }
}
```

> ⚠️ La URL debe terminar en `/`. Sin la barra final, la conexión falla.

### Herramientas disponibles

**Sin API key (análisis estático — siempre disponibles):**
- `search_nodes` — buscar nodos por keyword (<20ms)
- `get_node` — documentación y schema de un nodo (<10ms)
- `validate_node` — validar configuración de un nodo (<100ms)
- `validate_workflow` — validar estructura de un workflow JSON completo
- `search_templates` — buscar en 2,700+ templates de la comunidad
- `get_template` — obtener un template completo para importar

**Con API key (requiere n8n corriendo y accesible):**
- `n8n_list_workflows` — listar todos los workflows
- `n8n_get_workflow` — obtener un workflow por ID (con su JSON completo)
- `n8n_create_workflow` — crear un workflow nuevo
- `n8n_update_partial_workflow` — editar partes de un workflow existente ⭐ (más usada)
- `n8n_validate_workflow` — validar un workflow en la instancia real
- `n8n_executions` — ver historial de ejecuciones
- `n8n_deploy_template` — desplegar un template de la comunidad directamente
- `n8n_autofix_workflow` — aplicar fixes automáticos sugeridos por el validador

### Limitaciones del MCP

- ❌ No puede crear ni editar credenciales (el secreto nunca sale de n8n)
- ❌ No puede acceder a la UI visual (lo que ve el usuario en el canvas)
- ❌ No puede ejecutar workflows con datos de prueba personalizados complejos
- ⚠️ Requiere que n8n-mcp esté actualizado para nodes de n8n v1.x
- ⚠️ El proceso stdio puede desconectarse en sesiones muy largas (reiniciar Claude Code)

---

## Método 2: API REST directa

### Cómo funciona

```
Claude Code
    │
    ▼ (herramienta Bash con curl, o descripción en lenguaje natural)
n8n REST API (sin proceso intermedio)
    │
    ▼
Tu instancia de n8n
```

Sin intermediario. Claude habla directamente con la API REST de n8n.
Más rápido pero requiere que Claude "sepa" la estructura de la API.

### Cómo usarlo desde Claude Code

**Opción A: Con la herramienta Bash**
```bash
# Listar workflows
curl -s "https://pod-1.dopla.app/api/v1/workflows" \
  -H "X-N8N-API-KEY: TU_API_KEY" \
  -H "Content-Type: application/json" | jq '.data[].name'

# Ver últimas 5 ejecuciones fallidas
curl -s "https://pod-1.dopla.app/api/v1/executions?limit=5&status=error" \
  -H "X-N8N-API-KEY: TU_API_KEY" \
  | jq '.data[] | {id, workflowId, startedAt, stoppedAt}'
```

**Opción B: Pidiéndoselo a Claude en lenguaje natural**
```
"Haz una petición GET a https://pod-1.dopla.app/api/v1/executions
 con el header X-N8N-API-KEY=[key] y muéstrame el status de las
 últimas 10 ejecuciones del workflow con ID [workflow-id]"
```

**Opción C: Desde dentro de un workflow n8n (auto-referencia)**
```json
{
  "url": "={{ $env.N8N_API_URL }}api/v1/workflows",
  "method": "GET",
  "sendHeaders": true,
  "headerParameters": {
    "parameters": [{"name": "X-N8N-API-KEY", "value": "={{ $env.N8N_API_KEY }}"}]
  }
}
```

### Endpoints más útiles

| Endpoint | Método | Para qué |
|---|---|---|
| `/api/v1/workflows` | GET | Listar todos los workflows |
| `/api/v1/workflows/{id}` | GET | Obtener JSON completo de un workflow |
| `/api/v1/workflows/{id}/activate` | POST | Activar workflow |
| `/api/v1/workflows/{id}/deactivate` | POST | Desactivar workflow |
| `/api/v1/executions?workflowId={id}&limit=10` | GET | Historial de ejecuciones |
| `/api/v1/executions/{id}` | GET | Detalle completo de una ejecución |
| `/api/v1/credentials` | GET | Listar credenciales (sin valores secretos) |
| `/api/v1/workflows/{id}/run` | POST | Ejecutar workflow con datos manuales |

### Limitaciones de la API REST directa

- ❌ Claude no tiene contexto semántico de n8n (no "entiende" la estructura de nodos)
- ❌ No hay herramientas de validación integradas — los errores llegan como JSON crudo
- ❌ Para crear/editar workflows, necesitas conocer el JSON schema completo
- ⚠️ Gestión de autenticación manual en cada llamada (no usa credenciales guardadas)

---

## Tabla comparativa

| Dimensión | MCP n8n-mcp | API REST directa |
|---|---|---|
| **Setup** | Configurar en .mcp.json | Solo necesitas la API key |
| **Buscar nodos** | ✅ search_nodes (semántico) | ❌ No disponible |
| **Validar configuración** | ✅ validate_node / validate_workflow | ❌ No disponible |
| **Crear workflows** | ✅ con validación integrada | ⚠️ POST raw JSON |
| **Editar workflows** | ✅ operaciones parciales | ⚠️ PUT/PATCH JSON completo |
| **Templates (2700+)** | ✅ search + deploy | ❌ No disponible |
| **Ver ejecuciones** | ✅ n8n_executions | ✅ GET /api/v1/executions |
| **Credenciales** | ❌ No gestiona | ✅ GET /api/v1/credentials |
| **Velocidad** | ~100-500ms (proceso intermedio) | ~20-50ms (directo) |
| **Debug de errores** | Interpretación semántica | Solo JSON crudo |
| **Conocimiento previo** | Bajo — MCP guía | Alto — conocer la API |

---

## Cuándo usar cada método

### Usar MCP cuando:

- Estás **construyendo o editando workflows** — el MCP entiende la estructura
- Necesitas **buscar nodos** — search_nodes es semántico, no literalista
- Quieres **validar antes de deploy** — validate_workflow detecta errores antes de que fallen en runtime
- Eres nuevo en n8n o en un nodo específico — el MCP actúa como experto guía
- Necesitas **desplegar un template** de la comunidad rápidamente
- Quieres **autofix** — n8n_autofix_workflow detecta y repara problemas comunes

### Usar API REST cuando:

- Necesitas **leer datos de ejecuciones** en tiempo real desde un script o sistema externo
- Quieres **disparar workflows** desde una app externa (Stripe webhook → activar workflow)
- Necesitas **listar credenciales** disponibles para auditoría
- Estás construyendo **integración CI/CD** (GitHub Actions que activa/desactiva workflows)
- Necesitas la **velocidad máxima** sin overhead del proceso npx
- El MCP está caído o no disponible — fallback de emergencia
- Quieres **activar/desactivar workflows en masa** via script

---

## El patrón híbrido (lo que funciona en producción)

```
CONSTRUCCIÓN Y DESARROLLO:
    Claude Code + MCP
    → crear workflows, editar nodos, validar, buscar templates
    → Tiempo: meses de desarrollo activo

OPERACIÓN Y MONITOREO:
    Scripts bash + API REST
    → chequear ejecuciones fallidas, activar/desactivar, reportes
    → Tiempo: rutina diaria/semanal

INTEGRACIÓN ENTRE SISTEMAS:
    HTTP Request node dentro de n8n → llama API REST de n8n mismo
    → Un workflow que gestiona otros workflows (meta-automatización)
    → Caso de uso: script de deployment que activa los workflows después de importar
```

---

## Ejemplo práctico: crear el mismo workflow con cada método

### Con MCP (Claude describe, MCP construye):
```
"Crea un workflow llamado 'Lead Qualifier' con un webhook POST en /leads/qualify
 que reciba {name, email, rds_score} y si rds_score > 500 guarde en Supabase
 y envíe email de confirmación"

Claude invoca automáticamente:
1. n8n_create_workflow({name: "Lead Qualifier"})
2. n8n_update_partial_workflow → webhook node
3. n8n_update_partial_workflow → IF rds_score > 500
4. n8n_update_partial_workflow → HTTP Request a Supabase
5. n8n_update_partial_workflow → Gmail node
6. validate_workflow({profile: "runtime"})
```

### Con API REST (requiere conocer el schema JSON):
```bash
curl -X POST "https://pod-1.dopla.app/api/v1/workflows" \
  -H "X-N8N-API-KEY: TU_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Lead Qualifier",
    "nodes": [
      {
        "id": "node-1",
        "name": "Webhook",
        "type": "n8n-nodes-base.webhook",
        "position": [0, 0],
        "parameters": {"httpMethod": "POST", "path": "leads/qualify"}
      }
      // ... todos los nodos con su schema completo ...
    ],
    "connections": { "Webhook": { "main": [[{"node": "IF", "type": "main", "index": 0}]] } },
    "settings": {},
    "active": false
  }'
```

La diferencia es evidente: con MCP son instrucciones en lenguaje natural, con API REST necesitas el JSON completo con todos los schemas correctos.
