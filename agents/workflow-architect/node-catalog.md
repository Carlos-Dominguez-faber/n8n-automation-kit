# Node Catalog — Configuraciones típicas por nodo

## Webhook

```json
{
  "httpMethod": "POST",
  "path": "/[cliente]/[proceso]/[accion]",
  "responseMode": "responseNode",
  "options": {}
}
```

**Nota:** `responseMode: "responseNode"` permite usar respondToWebhook más adelante.
Si no, el webhook responde automáticamente al terminar (sincrónico).

---

## Execute Workflow Trigger (trigger de sub-workflows)

No necesita configuración. Es el único trigger válido para sub-workflows.
Los datos del padre llegan directamente en `$json`.

---

## IF (validación)

```json
{
  "conditions": {
    "options": {"caseSensitive": true},
    "conditions": [
      {
        "id": "cond-1",
        "leftValue": "={{ $json.tenant_id }}",
        "rightValue": "",
        "operator": {"type": "string", "operation": "notEmpty"}
      }
    ],
    "combinator": "and"
  }
}
```

---

## Switch (despacho por tipo)

```json
{
  "mode": "rules",
  "options": {},
  "rules": {
    "values": [
      {
        "conditions": {
          "conditions": [{
            "leftValue": "={{ $json.tipo_contenido }}",
            "rightValue": "carousel",
            "operator": {"type": "string", "operation": "equals"}
          }]
        },
        "outputKey": "carousel"
      },
      {
        "conditions": {
          "conditions": [{
            "leftValue": "={{ $json.tipo_contenido }}",
            "rightValue": "post",
            "operator": {"type": "string", "operation": "equals"}
          }]
        },
        "outputKey": "post"
      }
    ]
  }
}
```

---

## HTTP Request (GET a Supabase REST API)

```json
{
  "method": "GET",
  "url": "={{ $env.SUPABASE_URL }}/rest/v1/tenants?id=eq.{{ $json.tenant_id }}&select=*",
  "authentication": "genericCredentialType",
  "genericAuthType": "httpHeaderAuth",
  "sendHeaders": true,
  "headerParameters": {
    "parameters": [
      {"name": "apikey", "value": "={{ $credentials.supabaseHttpHeaderAuth.value }}"},
      {"name": "Authorization", "value": "Bearer ={{ $credentials.supabaseHttpHeaderAuth.value }}"}
    ]
  }
}
```

## HTTP Request (PATCH a Supabase — actualizar)

```json
{
  "method": "PATCH",
  "url": "={{ $env.SUPABASE_URL }}/rest/v1/contents?id=eq.{{ $json.content_id }}",
  "sendHeaders": true,
  "headerParameters": {
    "parameters": [
      {"name": "apikey", "value": "={{ $credentials.supabaseHttpHeaderAuth.value }}"},
      {"name": "Prefer", "value": "return=representation"}
    ]
  },
  "sendBody": true,
  "contentType": "json",
  "body": "={{ JSON.stringify({status: 'completed', output_url: $json.output_url}) }}"
}
```

---

## executeWorkflow (invocar sub-workflow)

```json
{
  "source": "database",
  "workflowId": "={{ $json.sub_workflow_id }}",
  "mode": "Single Run",
  "fields": {
    "values": [
      {"name": "tenant_id", "stringValue": "={{ $json.tenant_id }}", "type": "string"},
      {"name": "content_id", "stringValue": "={{ $json.content_id }}", "type": "string"}
    ]
  }
}
```

> ⚠️ `type: "string"` es CRÍTICO. Sin él, el campo llega como undefined en el hijo.

---

## respondToWebhook (respuesta async rápida)

```json
{
  "respondWith": "json",
  "responseBody": "={{ JSON.stringify({status: 'accepted', message: 'Processing', job_id: $json.content_id}) }}",
  "options": {
    "responseCode": 202
  }
}
```

---

## Set (mapear datos)

```json
{
  "mode": "manual",
  "duplicateItem": false,
  "assignments": {
    "assignments": [
      {"id": "1", "name": "tenant_id", "value": "={{ $json.tenant_id }}", "type": "string"},
      {"id": "2", "name": "status", "value": "processing", "type": "string"},
      {"id": "3", "name": "started_at", "value": "={{ $now.toISO() }}", "type": "string"}
    ]
  }
}
```

---

## Code (JavaScript) — template estándar

```javascript
// Modo: Run Once for All Items (recomendado)
const items = $input.all();

const results = items.map(item => {
  const data = item.json;
  
  // Tu lógica aquí
  const processed = {
    tenant_id: data.tenant_id,
    result: data.value * 2  // ejemplo
  };
  
  return { json: processed };
});

return results;
```

> SIEMPRE retornar `[{json: {...}}]`
> NUNCA retornar `{json: {...}}` sin el array wrapper — causa error

---

## Sticky Note

```json
{
  "content": "## Header del Workflow\n\n**Propósito:** ...\n**Trigger:** ...\n**Deps:** ...\n**Mod:** YYYY-MM-DD",
  "color": 5,
  "width": 450,
  "height": 220
}
```

Colores: 1=amarillo, 2=rojo, 3=verde, 4=azul, 5=morado, 6=gris
