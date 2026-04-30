# Make Module → n8n Node Mapping (Completo)

Referencia exhaustiva de módulos Make y su equivalente en n8n.

---

## Criterios de equivalencia

- **Exacto**: el nodo n8n hace exactamente lo mismo
- **Aproximado**: funcionalidad similar, requiere ajuste de configuración
- **Manual**: no hay nodo equivalente, requiere HTTP Request a la API o Code node

---

## Triggers

| Make Module | n8n Node | Tipo | Notas |
|---|---|---|---|
| `webhooks:CustomWebHook` | Webhook | Exacto | Mismo path, mismo método |
| `webhooks:respondToWebhook` | Respond to Webhook | Exacto | Status code + body |
| `gmail:WatchEmails` | Gmail Trigger | Exacto | Filtros equivalentes |
| `google-sheets:watchRows` | Google Sheets Trigger | Exacto | Sheet ID + evento |
| `google-forms:watchResponses` | ❌ Sin equivalente | Manual | Usar Apps Script → Webhook |
| `airtable:watchRecords` | Airtable Trigger | Exacto | Table + vista |
| `typeform:watchResponse` | Typeform Trigger | Exacto | Form ID |
| `slack:watchNewMessage` | Slack Trigger | Exacto | Canal + evento |
| `discord:watchMessages` | Discord Trigger | Exacto | |
| `telegram-bot-api:watchMessages` | Telegram Trigger | Exacto | |
| `twitter:watchNewTweet` | Twitter/X Trigger | Exacto | |
| `notion:watchDatabaseItems` | Notion Trigger | Exacto | Database ID |
| `hubspot:watchContacts` | HubSpot Trigger | Exacto | |
| `stripe:watchEvents` | Stripe Trigger | Exacto | Event types |
| `github:watchEvents` | GitHub Trigger | Exacto | Repo + events |
| `shopify:watchEvents` | Shopify Trigger | Exacto | |
| `rss:readRSS` | RSS Feed Read | Exacto | URL |
| `builtin:BasicRepeater` | Schedule Trigger | Aproximado | Ver patrón Repeater |

---

## HTTP / Requests

| Make Module | n8n Node | Tipo | Notas |
|---|---|---|---|
| `http:ActionSendData` | HTTP Request | Exacto | Method, URL, Headers, Body |
| `http:RetrieveOneRecord` | HTTP Request (GET) | Exacto | |
| `http:ActionGetFile` | HTTP Request | Exacto | Response Format: File |
| `http:ParseHTML` | HTML Extract | Exacto | CSS selectors |

---

## Google Workspace

| Make Module | n8n Node | Tipo |
|---|---|---|
| `google-sheets:ActionAddRow` | Google Sheets → Append Row | Exacto |
| `google-sheets:ActionUpdateRow` | Google Sheets → Update Row | Exacto |
| `google-sheets:ActionDeleteRow` | Google Sheets → Delete Row | Exacto |
| `google-sheets:searchRows` | Google Sheets → Lookup | Exacto |
| `google-sheets:ActionGetRows` | Google Sheets → Read Rows | Exacto |
| `google-docs:ActionCreateDocument` | Google Docs → Create Document | Exacto |
| `google-docs:ActionUpdateDocument` | Google Docs → Update Document | Exacto |
| `google-docs:ActionGetDocument` | Google Docs → Get Document | Exacto |
| `gmail:ActionSendEmail` | Gmail → Send Email | Exacto |
| `gmail:ActionGetEmail` | Gmail → Get Email | Exacto |
| `gmail:ActionCreateDraft` | Gmail → Create Draft | Exacto |
| `google-drive:uploadFile` | Google Drive → Upload File | Exacto |
| `google-drive:downloadFile` | Google Drive → Download File | Exacto |
| `google-drive:createFolder` | Google Drive → Create Folder | Exacto |
| `google-drive:listFiles` | Google Drive → List Files | Exacto |
| `google-calendar:ActionAddEvent` | Google Calendar → Create Event | Exacto |
| `google-calendar:ActionUpdateEvent` | Google Calendar → Update Event | Exacto |
| `google-calendar:searchEvents` | Google Calendar → Get Events | Exacto |

---

## OpenAI / IA

| Make Module | n8n Node | Tipo | Notas |
|---|---|---|---|
| `openai-gpt-3:CreateCompletion` | OpenAI → Text: Complete | Exacto | Cambiar modelo a gpt-4o |
| `openai-gpt-3:CreateChatCompletion` | OpenAI → Chat: Send Message | Exacto | |
| `openai-gpt-3:CreateImage` | OpenAI → Image: Create | Exacto | |
| `openai-gpt-3:CreateTranscription` | OpenAI → Audio: Transcribe | Exacto | |
| `openai-gpt-3:CreateTranslation` | OpenAI → Audio: Translate | Exacto | |
| `openai-gpt-3:CreateEmbedding` | OpenAI → Embedding: Create | Exacto | |
| `openai-gpt-3:AnalyzeImage` | OpenAI → Chat (con imagen adjunta) | Aproximado | |
| `anthropic:CreateMessage` | Anthropic Chat Model / HTTP Request | Exacto | |

---

## Bases de datos / CRM

| Make Module | n8n Node | Tipo |
|---|---|---|
| `airtable:ActionCreate` | Airtable → Create Record | Exacto |
| `airtable:ActionUpdate` | Airtable → Update Record | Exacto |
| `airtable:ActionDelete` | Airtable → Delete Record | Exacto |
| `airtable:searchRecords` | Airtable → Search Records | Exacto |
| `notion:CreatePage` | Notion → Create Page | Exacto |
| `notion:UpdatePage` | Notion → Update Page | Exacto |
| `notion:GetPage` | Notion → Get Page | Exacto |
| `notion:searchPages` | Notion → Search | Exacto |
| `hubspot:createContact` | HubSpot → Create Contact | Exacto |
| `hubspot:updateContact` | HubSpot → Update Contact | Exacto |
| `mysql:ActionExecuteQuery` | MySQL → Execute Query | Exacto |
| `postgresql:ActionExecuteQuery` | Postgres → Execute Query | Exacto |
| `mongodb:ActionFindDocuments` | MongoDB → Find | Exacto |

---

## Comunicación

| Make Module | n8n Node | Tipo |
|---|---|---|
| `slack:ActionCreateMessage` | Slack → Post Message | Exacto |
| `slack:ActionUpdateMessage` | Slack → Update Message | Exacto |
| `slack:ActionUploadFile` | Slack → Upload File | Exacto |
| `discord:ActionSendMessage` | Discord → Send Message | Exacto |
| `telegram-bot-api:ActionSendTextMessage` | Telegram → Send Message | Exacto |
| `twilio:ActionSendSMS` | Twilio → Send SMS | Exacto |
| `sendgrid:ActionSendEmail` | SendGrid → Send Email | Exacto |
| `mailchimp:addMember` | Mailchimp → Add Member | Exacto |

---

## Utilidades / Flow Control

| Make Module | n8n Node | Tipo | Notas |
|---|---|---|---|
| `util:SetVariable2` | Set | Exacto | Guarda valor en item actual |
| `util:GetVariable2` | Expresión referencia | Aproximado | `{{$node["Set"].json.var}}` |
| `util:FunctionV2` | Code (JavaScript) | Exacto | Misma lógica JS |
| `util:TextAggregator` | Code (agregación) | Manual | Ver patrón Aggregator |
| `util:ArrayAggregator` | Code / Merge | Manual | Ver patrón Aggregator |
| `util:JSONParser` | Code `JSON.parse()` | Manual | Una línea de código |
| `util:Compose` | Set (con expresión) | Exacto | Concatenar strings |
| `util:TextParser` | Code / Regex node | Aproximado | |
| `markdown:Compile` | Code (marked.js) | Manual | `require('marked').parse(text)` |
| `builtin:BasicRepeater` | Loop Over Items + Code | Aproximado | Ver patrón Repeater |
| `builtin:Sleep` | Wait node | Exacto | Configurar tiempo |
| `builtin:BasicRouter` | Switch | Exacto | Ver patrón Router |
| `builtin:BasicFeeder` | Manual Trigger + Set | Aproximado | Datos fijos de test |

---

## Archivos / PDF

| Make Module | n8n Node | Tipo | Notas |
|---|---|---|---|
| `pdf-co:HTMLtoPDF` | HTTP Request → PDF.co | Manual | Misma API key |
| `pdf-co:MergePDF` | HTTP Request → PDF.co | Manual | |
| `pdf-co:ExtractText` | HTTP Request → PDF.co | Manual | |
| `cloudconvert:*` | HTTP Request → CloudConvert API | Manual | |
| `dropbox:uploadFile` | Dropbox → Upload | Exacto | |
| `dropbox:downloadFile` | Dropbox → Download | Exacto | |
| `aws-s3:putObject` | AWS S3 → Upload | Exacto | |
| `aws-s3:getObject` | AWS S3 → Download | Exacto | |

---

## E-commerce

| Make Module | n8n Node | Tipo |
|---|---|---|
| `shopify:createOrder` | Shopify → Create Order | Exacto |
| `shopify:getProduct` | Shopify → Get Product | Exacto |
| `woocommerce:createOrder` | WooCommerce → Create Order | Exacto |
| `stripe:createPaymentIntent` | Stripe → Create Payment Intent | Exacto |
| `stripe:createCustomer` | Stripe → Create Customer | Exacto |

---

## Patrones sin equivalente directo

### `util:TextAggregator` → Code node

```javascript
// Reunir texto de todos los items del iterator anterior
const allItems = $input.all();
return [{
  json: {
    aggregatedText: allItems.map(i => i.json.content).join("\n\n")
  }
}];
```

### `markdown:Compile` → Code node

```javascript
// n8n tiene marked disponible como módulo CommonJS
const { marked } = require('marked');
return [{ json: { html: marked($json.markdown) } }];
```

### `builtin:BasicRepeater` como generador de iteraciones

```javascript
// Generar N items para iterar
const count = 5; // número de repeticiones
return Array.from({ length: count }, (_, i) => ({
  json: { iteration: i + 1, total: count }
}));
```
