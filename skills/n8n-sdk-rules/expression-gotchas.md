# Expression Gotchas — Cuándo $json es Inseguro

Fuente: `@n8n/workflow-sdk/src/prompts/sdk-reference/expressions.ts`

---

## Variables disponibles en expresiones `{{ }}`

| Variable | Qué es |
|---|---|
| `$json` | Datos JSON del item actual del nodo predecesor **inmediato** |
| `$('NombreNodo').item.json` | Output de cualquier nodo por nombre (siempre seguro) |
| `$input.first()` | Primer item del predecesor inmediato |
| `$input.all()` | Todos los items del predecesor inmediato |
| `$input.item` | Item actual siendo procesado |
| `$binary` | Datos binarios del item actual |
| `$now` | DateTime actual (Luxon). Ej: `$now.toISO()` |
| `$today` | Inicio del día actual (Luxon). Ej: `$today.plus(1, 'days')` |
| `$itemIndex` | Índice del item actual |
| `$runIndex` | Índice de la corrida actual |
| `$execution.id` | ID único de la ejecución |
| `$execution.mode` | `'test'` o `'production'` |
| `$execution.resumeUrl` | URL para resumir ejecuciones en pausa (human-in-the-loop) |
| `$workflow.id` | ID del workflow |
| `$workflow.name` | Nombre del workflow |

---

## Los 3 contextos donde `$json` es inseguro

### Contexto 1: Sub-nodos de AI Agent

Los sub-nodos (memory, language model, parser, retriever, vector store, tool) NO tienen el mismo contexto de predecesor inmediato que un nodo del flujo principal.

```javascript
// AI Agent memory subnode
❌ sessionKey: "={{ $json.chatId }}"
✅ sessionKey: "={{ $('Telegram Trigger').item.json.message.chat.id }}"
```

**Regla:** En cualquier subnode conectado vía `ai_memory`, `ai_languageModel`, `ai_tool`, `ai_vectorStore`, o `ai_outputParser`, usar siempre `$('NombreNodo').item.json`.

### Contexto 2: Multi-branch fan-in (después de IF/Switch/Merge)

Si un nodo recibe datos después de un IF/Switch/Merge, `$json` solo representa el item entrante actual y puede no contener el campo que necesitas del nodo fuente.

```javascript
// Nodo después de un IF
❌ "={{ $json.userId }}"
✅ "={{ $('User Lookup').item.json.user.id }}"
```

### Contexto 3: Datos de nodos no inmediatos

Si el valor viene de cualquier nodo que no es el predecesor inmediato directo, referenciar ese nodo explícitamente.

```javascript
// El email viene del Form Trigger, varios nodos antes
❌ "={{ $json.email }}"
✅ "={{ $('Form Trigger').item.json.body.email }}"
```

---

## Composición de strings con expresiones

Las variables SIEMPRE deben ir dentro de `{{ }}`, nunca fuera como variables JS:

```javascript
// Correcto — variable embebida en texto
"Hello {{ $json.name }}, welcome!"

// Correcto — múltiples variables con llamada a método
"Report for {{ $now.toFormat('MMMM d, yyyy') }} - {{ $json.title }}"

// Correcto — combinar campos
"{{ $json.firstName }} {{ $json.lastName }}"

// Correcto — expresión con ternario
"Status: {{ $json.count > 0 ? 'active' : 'empty' }}"
```

---

## Datos dinámicos de otros nodos — `$()` siempre dentro de `{{ }}`

```javascript
// INCORRECTO — $() fuera de {{ }}
❌ "={{ ' + JSON.stringify($('Source').all().map(i => i.json.name)) + ' }}"

// CORRECTO — $() dentro de {{ }}
✅ "={{ $('Source').all().map(i => ({ option: i.json.name })) }}"

// CORRECTO — JSON complejo dentro de {{ }}
✅ "={{ { 'fields': [{ 'values': $('Fetch Projects').all().map(i => ({ option: i.json.name })) }] } }}"
```

---

## Regla de sesión en chatbots (memory session keys)

En sub-nodos de AI Agent, NO usar `$json` para la session key — el sub-nodo no tiene el mismo contexto de predecesor:

| Plataforma | Configuración correcta |
|---|---|
| Telegram | `sessionIdType = customKey`, `sessionKey = $('Telegram Trigger').item.json.message.chat.id` |
| Slack | `sessionIdType = customKey`, `sessionKey = $('Slack Trigger').item.json.event.channel` |
| WhatsApp | `sessionIdType = customKey`, `sessionKey = $('WhatsApp Trigger').item.json.messages[0].from` |
| n8n Chat Trigger | `sessionIdType = fromInput`, omitir sessionKey — el Chat Trigger provee el ID directamente |

---

## Luxon — Referencia rápida de fechas

```javascript
$now.toISO()                          // "2026-04-30T10:00:00.000Z"
$now.toFormat("dd/MM/yyyy")           // "30/04/2026"
$now.plus({ days: 7 }).toISO()        // 7 días en el futuro
$now.minus({ hours: 2 }).toISO()      // 2 horas antes
$today.startOf('month').toISO()       // Inicio del mes
$today.endOf('month').toISO()         // Fin del mes
DateTime.fromISO($json.fecha).toFormat("dd/MM/yyyy")  // Formatear fecha existente
DateTime.fromISO($json.d2).diff(DateTime.fromISO($json.d1), 'days').days  // Diferencia en días
```
