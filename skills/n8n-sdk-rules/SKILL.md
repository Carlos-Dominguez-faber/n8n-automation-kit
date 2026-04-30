---
name: n8n-sdk-rules
description: >
  Reglas canónicas de generación de workflows n8n extraídas del @n8n/workflow-sdk oficial.
  Cubre: reglas estrictas de flow control, cuándo $json es inseguro, selección de nodos por
  caso de uso, guías de parámetros para nodos críticos (IF, Set, Switch, Webhook, HTTP Request,
  Tool nodes, AI Agent), y buenas prácticas por categoría. Usar siempre que se cree o edite
  cualquier workflow n8n — estas reglas previenen los errores más comunes.
---

# n8n SDK Rules — Reglas Canónicas de Generación

Contenido extraído directamente del `@n8n/workflow-sdk` oficial de n8n (v0.12.1).
Escrito por el equipo de n8n específicamente para guiar a agentes de IA.

---

## Índice

1. **[workflow-rules.md](workflow-rules.md)** — 4 reglas estrictas de flow control (leer siempre)
2. **[expression-gotchas.md](expression-gotchas.md)** — Cuándo `$json` es inseguro + referencia completa de variables
3. **[node-selection.md](node-selection.md)** — Selección de nodo por caso de uso, triggers, AI nodes
4. **[parameter-guides.md](parameter-guides.md)** — Guías de parámetros: IF, Set, Switch, Webhook, HTTP Request, Tool nodes, AI Agent
5. **[best-practices.md](best-practices.md)** — Patrones por categoría: chatbot, notificaciones, scheduling, human-in-the-loop

---

## Las reglas más críticas (resumen ejecutivo)

### 1. Nunca uses `alwaysOutputData: true` para mantener la cadena viva

```
❌ alwaysOutputData: true  ← causa undefined reads, HTTP calls a "undefined", crashes
✅ filter node  ← emite 0 items cuando nada coincide, la cadena para limpiamente
```

### 2. `$json` es inseguro en 3 contextos

```
❌ $json.field  ← en sub-nodos de AI Agent (memory, model, parser)
❌ $json.field  ← después de IF/Switch/Merge (multi-branch fan-in)
❌ $json.field  ← cuando el dato viene de un nodo no inmediato
✅ $('NombreNodo').item.json.field  ← siempre seguro
```

### 3. En Set node, SIEMPRE usar "value", nunca "stringValue" / "numberValue"

```
❌ { "stringValue": "hola" }
✅ { "value": "hola", "type": "string" }
```

### 4. Webhook responseMode y RespondToWebhook van juntos

```
Si responseMode = 'responseNode'  →  DEBE haber un nodo RespondToWebhook downstream
Si hay un nodo RespondToWebhook  →  responseMode DEBE ser 'responseNode'
```

### 5. `$fromAI` SOLO en tool nodes

```
❌ Set node: { "value": "={{ $fromAI('campo') }}" }
✅ Gmail Tool: { "sendTo": "={{ $fromAI('to') }}" }
```

---

## Cuándo aplica cada módulo

| Situación | Módulo |
|---|---|
| Creando cualquier workflow | workflow-rules.md (siempre) |
| Escribiendo expresiones `{{ }}` | expression-gotchas.md |
| Eligiendo qué nodo usar | node-selection.md |
| Configurando IF, Set, Switch, Webhook | parameter-guides.md |
| Diseñando chatbot, notificaciones, scheduler | best-practices.md |
| Trabajando con AI Agent + tools | node-selection.md + parameter-guides.md |

---

## Origen de este contenido

Extraído de `packages/@n8n/workflow-sdk/src/prompts/` del repositorio oficial de n8n.
Fuente: https://github.com/n8n-io/n8n/tree/master/packages/%40n8n/workflow-sdk

El directorio `/prompts/` fue creado por el equipo de n8n para alimentar directamente
a agentes de IA que generan workflows. No es documentación de usuario — es guía para LLMs.
