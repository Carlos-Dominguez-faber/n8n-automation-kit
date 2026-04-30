# Parameter Guides — Guías de Parámetros para Nodos Críticos

Fuente: `@n8n/workflow-sdk/src/prompts/node-guidance/parameter-guides/` y `node-tips/`

---

## IF Node — Referencia completa de operadores

El IF node usa una estructura de filtro compleja. Entender el formato correcto del operador es crítico.

### Estructura del IF node

```json
{
  "conditions": {
    "options": {
      "caseSensitive": false,
      "leftValue": "",
      "typeValidation": "loose"
    },
    "conditions": [
      {
        "leftValue": "={{ $('Node').item.json.field }}",
        "rightValue": "value",
        "operator": {
          "type": "string|number|boolean|dateTime|array|object",
          "operation": "operacion-especifica"
        }
      }
    ],
    "combinator": "and"
  }
}
```

### Operadores de String

| Operación | `operation` | ¿Necesita `rightValue`? |
|---|---|---|
| Existe | `exists` | No (`singleValue: true`) |
| No existe | `notExists` | No |
| Vacío | `empty` | No |
| No vacío | `notEmpty` | No |
| Igual | `equals` | Sí |
| No igual | `notEquals` | Sí |
| Contiene | `contains` | Sí |
| No contiene | `notContains` | Sí |
| Empieza con | `startsWith` | Sí |
| No empieza con | `notStartsWith` | Sí |
| Termina con | `endsWith` | Sí |
| No termina con | `notEndsWith` | Sí |
| Regex | `regex` | Sí |
| No regex | `notRegex` | Sí |

### Operadores de Number

| Operación | `operation` |
|---|---|
| Igual | `equals` |
| No igual | `notEquals` |
| Mayor que | `gt` |
| Menor que | `lt` |
| Mayor o igual | `gte` |
| Menor o igual | `lte` |

### Operadores de Boolean / DateTime / Array / Object

- Boolean: `true`, `false`, `equals`, `notEquals`
- DateTime: `equals`, `notEquals`, `after`, `before`, `afterOrEquals`, `beforeOrEquals`
- Array: `contains`, `notContains`, `lengthEquals`, `lengthNotEquals`, `lengthGreaterThan`, `lengthLessThan`, `empty`, `notEmpty`
- Object: `empty`, `notEmpty`

---

## Set Node — Manejo crítico de tipos

**CRÍTICO:** SIEMPRE usar el campo `"value"` para TODOS los tipos. NUNCA usar campos específicos de tipo como `"stringValue"`, `"numberValue"`, `"booleanValue"`, etc.

```json
{
  "id": "unique-id",
  "name": "nombre_del_campo",
  "value": "valor",
  "type": "string|number|boolean|array|object"
}
```

### Formatos por tipo

| Tipo | Formato del value | Ejemplo |
|---|---|---|
| `string` | String directo o expresión | `"Hola {{ $json.nombre }}"` |
| `number` | Número real (no string) | `123` o `45.67` ← NO `"123"` |
| `boolean` | Boolean real (no string) | `true` o `false` ← NO `"true"` |
| `array` | Array JSON stringificado | `"[1, 2, 3]"` o `"[\"a\", \"b\"]"` |
| `object` | Objeto JSON stringificado | `"{ \"nombre\": \"Juan\", \"edad\": 30 }"` |

### Keep Only Set

- **Desactivado (default):** lleva todos los campos forward + los que defines
- **Activado:** SOLO los campos que defines pasan (riesgo de pérdida de datos)

Siempre verificar el output después de cambiar esta configuración.

---

## Switch Node — Ruteo multi-rama

El Switch usa la misma estructura de filtro que el IF pero para múltiples rutas.

```json
{
  "mode": "rules",
  "rules": {
    "values": [
      {
        "conditions": {
          "conditions": [
            {
              "leftValue": "={{ $json.amount }}",
              "rightValue": 100,
              "operator": { "type": "number", "operation": "lt" }
            }
          ],
          "combinator": "and"
        },
        "renameOutput": true,
        "outputKey": "Menos de $100"
      }
    ]
  }
}
```

**Cada entrada en `rules.values[]` crea UN output.**
Usar `renameOutput: true` + `outputKey` para etiquetar los outputs descriptivamente.

### Rango numérico (ejemplo: $100–$1000)

Usar dos condiciones con `combinator: "and"`:
```json
{ "operation": "gte", "rightValue": 100 },
{ "operation": "lte", "rightValue": 1000 }
```

---

## Webhook Node — Reglas de responseMode

**REGLA 1:** Si `responseMode = 'responseNode'` → **DEBE** haber un nodo RespondToWebhook downstream.
**REGLA 2:** Si existe un nodo RespondToWebhook → `responseMode` **DEBE** ser `'responseNode'`.

### Cuándo usar cada modo

| Modo | Cuándo usarlo |
|---|---|
| `onReceived` | Acknowledgment rápido, el procesamiento ocurre async |
| `lastNode` | Devolver datos procesados, flujos simples request-response |
| `responseNode` | Control total sobre timing, headers y status codes de la respuesta |

### Patrón para control de respuesta personalizada

```
Webhook (responseMode: responseNode) → [Procesamiento] → RespondToWebhook
```

**Cuándo NO usar `responseNode`:**
- Acknowledgments simples → usar `onReceived`
- Devolver output del último nodo directamente → usar `lastNode`

---

## HTTP Request Node — Seguridad de credenciales

**NUNCA hardcodear credenciales** (API keys, tokens, passwords, secrets) en los parámetros del HTTP Request node.

Usar SIEMPRE el sistema de credenciales integrado de n8n:

```json
{
  "authentication": "genericCredentialType",
  "genericAuthType": "httpHeaderAuth"
}
```

| `genericAuthType` | Para qué |
|---|---|
| `httpHeaderAuth` | API keys en headers (X-API-Key, Authorization, etc.) |
| `httpBearerAuth` | Bearer token |
| `httpQueryAuth` | API keys como query parameters |
| `httpBasicAuth` | Username/password |
| `oAuth2Api` | OAuth 2.0 |

### Estructura de headers

```json
{
  "sendHeaders": true,
  "headerParameters": {
    "parameters": [
      { "name": "Header-Name", "value": "{{ expresion }}" }
    ]
  }
}
```

### Estructura de body (JSON)

```json
{
  "sendBody": true,
  "contentType": "json",
  "bodyParameters": {
    "parameters": [
      { "name": "campo", "value": "valor o {{ expresion }}" }
    ]
  }
}
```

---

## Tool Nodes — `$fromAI` expression

Los tool nodes (terminan en "Tool": Gmail Tool, Google Calendar Tool, etc.) soportan `$fromAI` para que el AI Agent llene parámetros dinámicamente en runtime.

**SOLO disponible en tool nodes.**

### Sintaxis

```
={{ $fromAI('key', 'descripcion', 'tipo', valorDefault) }}
```

| Parámetro | Descripción |
|---|---|
| `key` | Identificador único (1-64 chars, alfanumérico/guión bajo/guión) |
| `description` | Descripción opcional para el AI (usar `''` si no se necesita) |
| `type` | `'string'` \| `'number'` \| `'boolean'` \| `'json'` (default: `'string'`) |
| `defaultValue` | Valor fallback opcional |

### Ejemplos

```json
// Gmail Tool
{ "sendTo": "={{ $fromAI('to') }}", "subject": "={{ $fromAI('subject') }}", "message": "={{ $fromAI('message_html') }}" }

// Google Calendar Tool
{ "timeMin": "={{ $fromAI('After', '', 'string') }}", "timeMax": "={{ $fromAI('Before', '', 'string') }}" }

// Uso mixto — $fromAI embebido en texto
"Subject: {{ $fromAI('subject') }} - Automatizado"
```

### Reglas importantes

1. SOLO usar `$fromAI` en tool nodes
2. Para campos de fecha/tiempo, usar nombres de key descriptivos
3. El AI llenará estos valores basado en el contexto durante la ejecución
4. NO usar `$fromAI` en nodos regulares como Set, IF, HTTP Request, etc.

---

## AI Agent — System Message vs User Message

Dos campos distintos que DEBEN usarse correctamente:

| Campo | Nombres según nodo | Para qué |
|---|---|---|
| **System Message** | AI Agent: `options.systemMessage`; LLM Chain: `messages.messageValues[]` role system; Anthropic: `options.system` | Contenido ESTÁTICO que define el rol: identidad del agente, instrucciones paso a paso, guidelines de comportamiento |
| **User Message / Text** | `text` en AI Agent y LLM Chain | Contenido DINÁMICO específico de la ejecución: input del usuario, contexto de nodos anteriores |

```
✅ System: "Eres un agente de soporte al cliente. Tu tarea es..."
✅ Text: "={{ $json.chatInput }}"

❌ Text: "Eres un agente de soporte... El mensaje del usuario es: {{ $json.chatInput }}"
   (mezcla rol estático con input dinámico)
```

---

## Structured Output Parser

Usar cuando:
- El output de IA se usará programáticamente (condiciones, formateo, base de datos, API calls)
- La IA necesita extraer campos específicos (score, categoría, prioridad, items)
- Nodos downstream necesitan acceder a campos específicos (`$json.score`, `$json.categoria`)
- El output se mostrará en formato estructurado (email HTML con secciones específicas)

**Conexión:**
```
Structured Output Parser → AI Agent (ai_outputParser connection)
AI Agent: hasOutputParser: true
```

**Preferir sobre Code node** para parsear output de IA — los parsers nativos son más confiables y manejan edge cases mejor que código custom.
