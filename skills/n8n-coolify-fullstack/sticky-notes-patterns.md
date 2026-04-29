# Sticky Notes — Documentación Visual Obligatoria en n8n

## Por qué son obligatorias en producción

Un workflow sin sticky notes es código sin comentarios.
En n8n, los sticky notes son el ÚNICO mecanismo de documentación que:
- Sobrevive al JSON exportado e importado
- Es visible directamente en el canvas sin abrir cada nodo
- Permite que otro desarrollador (o Claude en una sesión futura) entienda el flujo de un vistazo

**Regla:** Todo workflow que va a producción tiene mínimo 2 sticky notes: header y output.

---

## Las 4 zonas obligatorias

### Zona 1: Header del workflow (OBLIGATORIO)

```markdown
## [Nombre del Workflow]

**Propósito:** [qué automatiza, en una frase]
**Trigger:** [webhook /ruta | schedule 0 9 * * * | manual]
**Dependencias:**
  - Sub-workflows: [lista de nombres]
  - Microservicios: [lista de servicios]
  - APIs externas: [lista]
**Última modificación:** YYYY-MM-DD — [quién]
```

### Zona 2: Validación de entrada (OBLIGATORIO si hay IF nodes)

```markdown
## Validación de entrada

Qué se valida:
- tenant_id: presente y no vacío → si falla: 400 Bad Request
- content_id: UUID válido → si falla: 404 Not Found
- tipo_contenido: enum(carousel|post|reel|reel_ugc) → si falla: 422

Si pasa todas las validaciones → continúa al enriquecimiento
```

### Zona 3: Lógica principal (RECOMENDADO)

```markdown
## Lógica de despacho

Switch por tipo_contenido:
- carousel → executeWorkflow Sub: Carrusel (ID dinámico desde Supabase)
- post → executeWorkflow Sub: Post Estático
- reel → executeWorkflow Sub: Reel (llama ffmpeg-merger al final)
- reel_ugc → executeWorkflow Sub: Reel UGC (requiere HeyGen del tenant)

Prioridad de fallback: si el sub-workflow falla → marcar content como "error" en Supabase
```

### Zona 4: Outputs y efectos secundarios (OBLIGATORIO)

```markdown
## Outputs

Actualiza:
  - Supabase tabla "contents": status → "completed" | "error"
  - Supabase tabla "credit_balances": descuenta según tipo

Notifica:
  - Webhook callback a dopla.app/api/n8n/callback (HMAC verificado)
  - Discord #monitoring si hay error

Almacena:
  - Supabase Storage bucket "generated-media": [tenant_id]/[content_id]/output.*
```

---

## Dimensiones y colores recomendados

| Zona | Width | Height | Color (n8n) | Color hex |
|---|---|---|---|---|
| Header | 450px | 220px | 5 (morado) | #7C3AED |
| Validación | 320px | 160px | 1 (amarillo) | #FBBF24 |
| Lógica principal | 420px | 200px | 3 (verde) | #10B981 |
| Output | 320px | 180px | 4 (azul) | #3B82F6 |

> Colores en n8n: 1=amarillo, 2=rojo, 3=verde, 4=azul, 5=morado, 6=gris

---

## Posicionamiento relativo al canvas

Los nodos en n8n empiezan alrededor de x=-2000 a x=0.
El sticky note header va ANTES del primer nodo:

```
x=-2400 a x=-2000 → zona del sticky note header
x=-2000 → primer nodo (Webhook / Execute Workflow Trigger)
x=-1500 → zona de validaciones (IF nodes)
x=-1000 → lógica principal (Switch)
x=-400  → executeWorkflow nodes (despacho a subs)
x=0+    → final / respondToWebhook
```

---

## Cómo crear sticky notes vía n8n MCP

```javascript
// Comando para Claude: agregar sticky note header via MCP
// Claude invoca n8n_update_partial_workflow con:

{
  type: "addNode",
  node: {
    type: "n8n-nodes-base.stickyNote",
    name: "Doc — Header",
    parameters: {
      content: "## Router Dopla\n\n**Propósito:** Recibe webhooks del SaaS y despacha al sub-workflow correcto según tipo_contenido\n**Trigger:** POST /dopla/content/generate\n**Deps:** Sub-Carrusel, Sub-Post, Sub-Reel, Sub-Reel-UGC, ffmpeg-merger-dopla\n**Mod:** 2026-04-29 — Carlos",
      color: 5,
      width: 450,
      height: 220
    },
    position: [-2400, -100]
  }
}
```

**Cómo pedírselo a Claude:**
```
"Agrega las 4 sticky notes de documentación al workflow Router Dopla
 usando las posiciones y contenidos estándar del skill n8n-coolify-fullstack"
```

---

## Anti-patrones a evitar

❌ **Sticky note con solo el nombre del nodo** — ya se ve en el nodo mismo
❌ **Una sola sticky note para todo el workflow** — divide por zonas
❌ **Sticky notes con código** — para código usa el propio Code node con comentarios
❌ **Actualizar sticky notes manualmente olvidando actualizar la fecha** — incluir fecha siempre
❌ **Poner URLs con credenciales en sticky notes** — son visibles en el canvas para cualquiera con acceso
