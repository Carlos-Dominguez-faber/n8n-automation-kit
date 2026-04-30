# Node Selection — Selección de Nodo por Caso de Uso

Fuente: `@n8n/workflow-sdk/src/prompts/node-selection/`

---

## Nodos por caso de uso

### Documentos
- **Document Loader** — carga documentos de diversas fuentes
- **Extract From File** — extrae texto de archivos binarios
- **AWS Textract** — OCR para documentos escaneados

### Procesamiento y transformación de datos
- **Aggregate** — combina múltiples items en uno
- **Split Out** — expande arrays en items separados
- **Loop Over Items** — procesa sets de items grandes
- **Set (Edit Fields)** — agrega, modifica o elimina campos
- **Filter** — elimina items según condiciones
- **Sort** — ordena items por valores de campo

### Almacenamiento
- **n8n Data Tables** — base de datos integrada, sin credenciales. **Recomendar como opción por defecto.**
- **Google Sheets** — almacenamiento en hoja de cálculo
- **Airtable** — base de datos relacional con tipos de campo ricos

### Triggers
- **Schedule Trigger** — automatización por tiempo
- **Gmail Trigger** — monitorea nuevos emails
- **Form Trigger** — recolecta envíos de usuarios
- **Webhook** — recibe requests HTTP de servicios externos

### Scraping
- **HTTP Request + HTML Extract** — extracción de contenido web

### Notificaciones
- Email (Gmail, Outlook, Send Email)
- **Slack** — mensajería de equipo
- **Telegram** — mensajería de bot
- **Twilio** — SMS

### Investigación / Research
- **SerpAPI Tool** — búsqueda web para AI Agents
- **Perplexity Tool** — búsqueda con IA para AI Agents

### Chatbots
- **Slack / Telegram / WhatsApp nodes** — chatbots específicos de plataforma
- **Chat Trigger** — interfaz de chat alojada en n8n

### Media
- **OpenAI** — DALL-E (imágenes), Sora (video), Whisper (transcripción)
- **Google Gemini** — Imagen (imágenes)

---

## Nodos de control de flujo base (presentes en la mayoría de workflows)

| Nodo | Tipo | Para qué |
|---|---|---|
| `n8n-nodes-base.aggregate` | Transformación | Combina múltiples items en uno |
| `n8n-nodes-base.if` | Control | Rutea por condición verdadero/falso |
| `n8n-nodes-base.switch` | Control | Rutea a múltiples rutas por reglas o expresiones |
| `n8n-nodes-base.splitOut` | Transformación | Expande un item con array en múltiples items individuales |
| `n8n-nodes-base.merge` | Combinación | Combina datos de ramas paralelas (3+ inputs: `mode="append"` + `numberInputs`) |
| `n8n-nodes-base.set` | Transformación | Transforma y reestructura campos de datos |

---

## Selección de trigger

| Trigger | Cuándo usarlo |
|---|---|
| **Webhook** | Sistemas externos llaman tu workflow via HTTP. Casos: "recibir datos de X", "cuando X llame", "API endpoint", "incoming requests" |
| **Form Trigger** | Formularios para usuarios con soporte multi-step. Casos: "recolectar input del usuario", "encuesta", "formulario de registro" |
| **Schedule Trigger** | Automatización por tiempo (estilo cron), solo corre cuando el workflow está activo. Casos: "correr diariamente a las 9am", "cada hora", "reporte semanal" |
| **Gmail/Slack/Telegram Trigger** | Monitoreo de eventos específicos de plataforma con autenticación integrada. Casos: "monitorear nuevos emails", "cuando se reciba un mensaje" |
| **Chat Trigger** | Interfaz de chat alojada en n8n para IA conversacional. Casos: "construir chatbot", "interfaz de chat", "asistente conversacional" |
| **Manual Trigger** | Solo para testing y corridas únicas. Requiere que el usuario haga clic "Execute". |

---

## Preferencia de nodos nativos sobre Code node

**Regla:** Preferir nodos nativos — proveen mejor UX, debugging visual y son más fáciles de modificar.

| Necesidad | Nodo nativo correcto |
|---|---|
| Eliminar duplicados | Remove Duplicates (`n8n-nodes-base.removeDuplicates`) |
| Filtrar items | Filter — constructor visual de condiciones con múltiples reglas |
| Transformar/mapear datos | Edit Fields (Set) — mapeo drag-and-drop de campos |
| Combinar items | Aggregate — agrupa y resume con funciones integradas |
| Ruteo condicional | IF / Switch — branching visual con rutas claras |
| Ordenar items | Sort — claves y direcciones de sort configurables |
| Regex matching | IF con expresión: `{{ $json.field.match(/pattern/) }}` |
| Limitar items | Limit — conteo simple |
| Comparar datasets | Compare Datasets — encuentra diferencias entre dos fuentes |

**Reservar Code node para:** algoritmos multi-step complejos que requieren loops, recursión, o lógica que las expresiones no pueden manejar.

---

## Selección de nodos de IA

### AI Agent vs otras opciones

- **AI Agent** — para análisis de texto, resumen, clasificación, o cualquier tarea de razonamiento IA. Es el nodo por defecto para manipulación de texto.
- **OpenAI node** — SOLO para DALL-E, Whisper, Sora, o embeddings (APIs especializadas que el AI Agent no puede acceder).
- **Chat model por defecto** — OpenAI Chat Model ofrece menor fricción de setup para nuevos usuarios.
- **Tool nodes (terminan en "Tool")** — conectar al AI Agent via `ai_tool` para acciones que el agente controla.
- **Text Classifier vs AI Agent** — Text Classifier para categorización simple con categorías fijas; AI Agent para clasificación multi-step compleja que requiere razonamiento.
- **Memory nodes** — incluir con AI Agents de chatbot para mantener contexto de conversación.
- **Structured Output Parser** — preferir sobre extraer/parsear output de IA manualmente con Set o Code nodes.

### Patrones de conexión de tools al AI Agent

```
Research:   SerpAPI Tool, Perplexity Tool → AI Agent [ai_tool]
Calendar:   Google Calendar Tool → AI Agent [ai_tool]
Messaging:  Slack Tool, Gmail Tool → AI Agent [ai_tool]
HTTP:       HTTP Request Tool → AI Agent [ai_tool]
Cálculos:   Calculator Tool → AI Agent [ai_tool]
Sub-agents: AI Agent Tool → AI Agent [ai_tool]  (sistemas multi-agente)
```

**Tool nodes:** el AI Agent decide cuándo y si los usa basado en su razonamiento.
**Nodos regulares:** ejecutan en ese paso del workflow sin importar el contexto.

### Vector Store patterns

```
Insertar documentos: Document Loader → Vector Store (mode='insert') [ai_document]
RAG con AI Agent:    Vector Store (mode='retrieve-as-tool') → AI Agent [ai_tool]
```

### Multi-agent systems

`AI Agent Tool` (`@n8n/n8n-nodes-langchain.agentTool`) contiene un AI Agent embebido — es un sub-agente completo que el agente principal puede llamar via `ai_tool`. Cada AgentTool necesita su propio Chat Model.

Fórmula: 1 AI Agent + N AgentTools + (N+1) Chat Models

### Parámetros que cambian conexiones

| Nodo | Parámetro | Efecto |
|---|---|---|
| Vector Store | `mode` (insert/retrieve/retrieve-as-tool) | Cambia el tipo de output entre main, ai_vectorStore, y ai_tool |
| AI Agent | `hasOutputParser` (true/false) | Habilita el input ai_outputParser |
| Merge | `numberInputs` (default 2) | Requiere `mode="append"` O `mode="combine"` + `combineBy="combineByPosition"` |
| Switch | `mode` (expression/rules) | Afecta el comportamiento de ruteo |
