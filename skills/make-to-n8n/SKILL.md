---
name: make-to-n8n
description: >
  Guía experta para migrar flujos de Make (Integromat) a n8n. Cubre el mapeo de módulos,
  conversión de expresiones, manejo de iterators/routers/aggregators, y traducción de
  estructuras JSON de Make a workflows n8n. Usar cuando el usuario tenga un escenario
  de Make y quiera recrearlo en n8n, o cuando analice un JSON exportado de Make.
---

# Make → n8n Conversion Guide

Migración completa de escenarios Make a workflows n8n.

---

## Conceptos: Make vs n8n

| Make | n8n | Notas |
|---|---|---|
| Scenario | Workflow | La unidad completa de automatización |
| Module | Node | Cada paso/acción individual |
| Route | Branch (IF / Switch) | Bifurcación condicional |
| Iterator | Split In Batches / Loop | Procesar items uno a uno |
| Aggregator | Merge / Code | Reunir resultados de un iterator |
| Router | Switch node | Múltiples rutas con condiciones |
| Filter | IF node | Condición entre dos módulos |
| Set Variable | Set node | Guardar un valor |
| Get Variable | Reference via expression | Acceder al valor guardado |
| Basic Feeder | Manual Trigger + Set | Datos de prueba |
| Webhook | Webhook node | Trigger por HTTP |
| Schedule | Schedule Trigger | Trigger por tiempo |
| `{{1.field}}` | `{{$node["NodeName"].json.field}}` | Referencia a datos de otro módulo |
| `{{now}}` | `{{$now}}` | Timestamp actual |
| `{{bundle.cursor}}` | `{{$json.cursor}}` | Paginación |

---

## Módulos Make → Nodos n8n (Tabla Maestra)

### Triggers

| Módulo Make | Nodo n8n | Configuración clave |
|---|---|---|
| `webhooks:CustomWebHook` | **Webhook** | Method: POST, Path: igual al de Make |
| `google-forms:watchResponses` | **Google Sheets Trigger** o **Webhook** | n8n no tiene trigger nativo de Google Forms; usar Apps Script → Webhook |
| `gmail:WatchEmails` | **Gmail Trigger** | Operation: Watch, Filters iguales |
| `airtable:watchRecords` | **Airtable Trigger** | Operation: On Row Created |
| `typeform:watchResponse` | **Typeform Trigger** | o Webhook desde Typeform |
| `slack:watchNewMessage` | **Slack Trigger** | Event: message |
| `google-sheets:watchRows` | **Google Sheets Trigger** | Event: Row Added |
| `rss:readRSS` | **RSS Feed Read** | URL del feed |
| `builtin:BasicRepeater` | **Schedule Trigger** + Loop | Ver sección Repeater |
| Schedule | **Schedule Trigger** | Cron expression equivalente |

### HTTP / APIs

| Módulo Make | Nodo n8n | Configuración clave |
|---|---|---|
| `http:ActionSendData` | **HTTP Request** | Method, URL, Body |
| `http:ActionGetFile` | **HTTP Request** | Response Format: File |
| `http:RetrieveOneRecord` | **HTTP Request** | Method: GET |
| `util:CallAPI` | **HTTP Request** | Headers, Auth type |

### Google Workspace

| Módulo Make | Nodo n8n |
|---|---|
| `google-sheets:ActionAddRow` | **Google Sheets** → Append Row |
| `google-sheets:ActionUpdateRow` | **Google Sheets** → Update Row |
| `google-sheets:searchRows` | **Google Sheets** → Lookup |
| `google-docs:ActionCreateDocument` | **Google Docs** → Create Document |
| `gmail:ActionSendEmail` | **Gmail** → Send Email |
| `google-drive:uploadFile` | **Google Drive** → Upload File |
| `google-calendar:ActionAddEvent` | **Google Calendar** → Create Event |

### OpenAI / IA

| Módulo Make | Nodo n8n |
|---|---|
| `openai-gpt-3:CreateCompletion` | **OpenAI** → Text: Complete |
| `openai-gpt-3:CreateChatCompletion` | **OpenAI** → Chat: Send Message |
| `openai-gpt-3:CreateImage` | **OpenAI** → Image: Create |
| `openai-gpt-3:CreateTranscription` | **OpenAI** → Audio: Transcribe |
| `openai-gpt-4:*` | **OpenAI** → (mismo, cambiar modelo a gpt-4o) |

### Utilidades

| Módulo Make | Nodo n8n | Notas |
|---|---|---|
| `util:SetVariable2` | **Set** node | Guardar valor en item |
| `util:GetVariable2` | Expresión `{{$node["Set"].json.varName}}` | No hay nodo Get Variable |
| `util:FunctionV2` | **Code** node (JS) | Lógica custom |
| `util:TextAggregator` | **Code** node | Concatenar texto |
| `util:ArrayAggregator` | **Code** node / **Merge** | Acumular array |
| `util:JSONParser` | **Code** `JSON.parse($json.text)` | Parsear JSON string |
| `util:Compose` | **Set** node (con expresión) | Concatenar strings |
| `markdown:Compile` | **Code** node (marked.js) o **n8n** Built-in | Convertir Markdown → HTML |
| `builtin:BasicRepeater` | **Loop Over Items** + **Code** | Ver sección Repeater |

### PDF / Archivos

| Módulo Make | Nodo n8n |
|---|---|
| `pdf-co:HTMLtoPDF` | **HTTP Request** a PDF.co API (mismas credenciales) |
| `pdf-co:MergePDF` | **HTTP Request** a PDF.co API |
| `pdf:convertToPDF` | **HTTP Request** a LibreOffice/Gotenberg microservice |

### CRM / Bases de datos

| Módulo Make | Nodo n8n |
|---|---|
| `airtable:ActionCreate` | **Airtable** → Create Record |
| `airtable:ActionUpdate` | **Airtable** → Update Record |
| `notion:CreatePage` | **Notion** → Create Page |
| `hubspot:createContact` | **HubSpot** → Create Contact |

---

## Conversión de Expresiones

### Sintaxis Make → n8n

```
Make:  {{1.name}}                         → n8n: {{$node["Google Forms"].json.name}}
Make:  {{2.choices[0].message.content}}   → n8n: {{$node["OpenAI"].json.choices[0].message.content}}
Make:  {{now}}                            → n8n: {{$now}}
Make:  {{formatDate(now; "DD/MM/YYYY")}}  → n8n: {{DateTime.now().toFormat("dd/MM/yyyy")}}
Make:  {{length(array)}}                  → n8n: {{$json.array.length}}
Make:  {{join(array; ", ")}}              → n8n: {{$json.array.join(", ")}}
Make:  {{if(cond; val1; val2)}}           → n8n: {{condition ? val1 : val2}}
Make:  {{replace(str; "a"; "b")}}         → n8n: {{$json.str.replace("a", "b")}}
Make:  {{trim(str)}}                      → n8n: {{$json.str.trim()}}
Make:  {{toString(num)}}                  → n8n: {{String($json.num)}}
Make:  {{toNumber(str)}}                  → n8n: {{Number($json.str)}}
Make:  {{parseDate(str; "MM/DD/YYYY")}}   → n8n: {{DateTime.fromFormat($json.str, "MM/dd/yyyy")}}
```

### Regla del número de módulo

En Make, `{{1.field}}` hace referencia al módulo con ID 1. En n8n, debes referenciar por nombre del nodo:

```javascript
// Make: {{3.responseText}}
// n8n: {{$node["OpenAI"].json.responseText}}
//       ↑ nombre que le pusiste al nodo OpenAI
```

---

## Patrones Especiales

### Iterator → Loop en n8n

```
Make: [Array Module] → [Iterator] → [Process] → [Aggregator]
n8n:  [Array Node]   → [Loop Over Items] → [Process] → (los items ya vienen como lista)
```

**n8n Loop Over Items** ya itera automáticamente sobre arrays. Si el nodo anterior devuelve múltiples items, los nodos siguientes los procesan uno por uno. No necesitas un nodo Iterator explícito.

Para acumular resultados (Aggregator en Make), usa un **Code** node al final:
```javascript
// Recopilar todos los items anteriores
const items = $input.all();
return [{ json: { results: items.map(i => i.json) } }];
```

### Router → Switch en n8n

```
Make:  Router con Filtros → Ruta A / Ruta B / Ruta C
n8n:   Switch node → Output 0 / Output 1 / Output 2
```

Cada "ruta" del Router de Make se convierte en un output del Switch. Las condiciones del filtro se trasladan a las "Rules" del Switch.

### Set Variable + Get Variable → Set + Expresión

Make permite guardar variables con scope de escenario. n8n no tiene scope global, pero puedes:

1. **Opción simple**: Guardar en el item con un **Set** node y referenciar después con `$node["Set"].json.varName`
2. **Opción para datos globales**: Usar un **Code** node con `$workflow.staticData`

```javascript
// En Code node — guardar
$workflow.staticData.myVar = "valor";

// En Code node — leer
const val = $workflow.staticData.myVar;
```

### Basic Repeater → Schedule + Loop

El `builtin:BasicRepeater` de Make ejecuta N veces un bloque. En n8n:

```
Opción 1: Schedule Trigger (si es por tiempo)
Opción 2: Code node que genere N items → Loop Over Items
```

```javascript
// Code node para generar N repeticiones
const n = 5;
return Array.from({length: n}, (_, i) => ({ json: { iteration: i + 1 } }));
```

---

## Proceso de Conversión Paso a Paso

### 1. Analizar el JSON de Make

```javascript
// Estructura del export de Make
{
  "name": "Nombre del Escenario",
  "flow": [
    { "id": 1, "module": "app:moduleName", "version": 1, "parameters": {}, "mapper": {} },
    ...
  ],
  "metadata": { ... }
}
```

**Lo que necesitas extraer por cada módulo:**
- `module` → determina qué nodo n8n usar (ver tabla maestra)
- `parameters` → configuración fija (credenciales, IDs, URLs)
- `mapper` → mapeo de campos (aquí están las expresiones `{{N.field}}`)
- `metadata.expect` → campos de configuración de la UI

### 2. Mapear el flujo

Dibuja el grafo de módulos Make y su equivalente en nodos n8n:

```
Make:
[Google Forms Trigger] → [OpenAI GPT-4] → [Set Variable] → [Repeater] → [Get Var] → [OpenAI] → [Set] → [Set] → [Markdown] → [PDF.co]

n8n:
[Webhook/Apps Script] → [OpenAI] → [Set] → [Loop] → [OpenAI] → [Set] → [Set] → [Code:MD→HTML] → [HTTP:PDF.co]
```

### 3. Convertir expresiones del mapper

El campo `mapper` de cada módulo Make contiene las expresiones. Tradúcelas a sintaxis n8n antes de configurar los nodos.

### 4. Crear el workflow en n8n

Usa el skill **n8n MCP Tools Expert** para crear el workflow:
1. `create_workflow` con el nombre del escenario
2. Añadir nodos en orden topológico
3. Conectar nodos en el mismo orden que Make
4. Configurar credenciales

### 5. Validar

Usa el skill **n8n Validation Expert** para validar la estructura antes de activar.

---

## Ejemplo Completo: Crea Ebooks Personalizados

Este es el escenario Make del kit convertido a n8n.

### Make (original)
```
1. google-forms:watchResponses    → Trigger cuando alguien llena el form
2. openai-gpt-3:CreateCompletion  → Genera índice del ebook
3. util:SetVariable2              → Guarda el índice
4. builtin:BasicRepeater          → Repite N veces (por capítulo)
5. util:GetVariable2              → Lee el índice
6. openai-gpt-3:CreateCompletion  → Genera cada capítulo
7. util:SetVariable2              → Guarda el capítulo
8. util:SetVariable2              → Acumula capítulos
9. markdown:Compile               → Convierte MD a HTML
10. pdf-co:HTMLtoPDF              → Genera PDF final
```

### n8n (equivalente)
```
1. Webhook ← Apps Script de Google Forms
2. OpenAI (gpt-4o) → genera índice
3. Set → guarda índice en item
4. Code → genera array de capítulos [{ chapter: 1 }, { chapter: 2 }, ...]
5. Split In Batches (tamaño 1)
6. OpenAI (gpt-4o) → genera contenido del capítulo (usa {{$node["Set"].json.indice}})
7. Set → guarda capítulo generado
8. Code (last batch) → concatena todos los capítulos
9. Code → convierte Markdown a HTML (o HTTP Request a microservicio)
10. HTTP Request → PDF.co API (HTMLtoPDF endpoint)
```

---

## Gotchas Comunes

### 1. Google Forms no tiene trigger nativo en n8n

**Solución**: Configurar un Apps Script en Google Forms que haga POST a un Webhook de n8n cuando se recibe una respuesta.

```javascript
// Apps Script en Google Forms
function onFormSubmit(e) {
  const payload = {};
  e.response.getItemResponses().forEach(r => {
    payload[r.getItem().getTitle()] = r.getResponse();
  });
  UrlFetchApp.fetch("https://tu-n8n.com/webhook/forms", {
    method: "post",
    contentType: "application/json",
    payload: JSON.stringify(payload)
  });
}
```

### 2. Variables de escenario vs item data

En Make, las variables de Set/Get Variable tienen scope de escenario (persisten entre iteraciones del Repeater). En n8n, los datos viven en el item. Para pasar datos entre loops, usa `$workflow.staticData`.

### 3. Aggregator de texto

Make tiene un `util:TextAggregator` que concatena texto de múltiples bundles. En n8n:

```javascript
// Code node — concatenar texto de todos los items anteriores
const allItems = $input.all();
const combined = allItems.map(i => i.json.content).join("\n\n");
return [{ json: { fullText: combined } }];
```

### 4. Expresiones en el mapper de Make

El campo `mapper` puede contener expresiones complejas anidadas:
```json
{"content": "{{join(map(5.chapters; \"title\"); \", \")}}"}
```

En n8n esto se convierte en una expresión JavaScript:
```javascript
{{ $node["GetVariable"].json.chapters.map(c => c.title).join(", ") }}
```

### 5. Error handling

Make tiene "Error handler routes". En n8n, activa "Continue On Fail" en cada nodo y añade un **Error Trigger** workflow separado.

---

## Checklist de Migración

- [ ] Exportar JSON del escenario Make
- [ ] Listar todos los módulos con su tipo (`module` field)
- [ ] Identificar el trigger principal
- [ ] Mapear módulo → nodo n8n (usar tabla maestra)
- [ ] Traducir todas las expresiones `{{N.field}}` a `{{$node["X"].json.field}}`
- [ ] Identificar iterators/aggregators y planear el loop en n8n
- [ ] Identificar routers y planear Switch/IF en n8n
- [ ] Configurar credenciales equivalentes en n8n
- [ ] Crear el workflow en n8n
- [ ] Testear con datos de prueba
- [ ] Validar con n8n Validation Expert
- [ ] Activar y monitorear primeras ejecuciones

---

## Skills relacionados

- **n8n MCP Tools Expert** — crear y configurar nodos
- **n8n Expression Syntax** — escribir expresiones correctamente
- **n8n Workflow Patterns** — elegir la arquitectura correcta
- **n8n Validation Expert** — validar antes de activar
- **n8n Node Configuration** — configurar operaciones específicas
