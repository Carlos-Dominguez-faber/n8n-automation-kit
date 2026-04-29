# CLAUDE.md — Contexto del Proyecto n8n

> Este archivo guía a Claude Code sobre tu proyecto de automatización.
> Completa cada sección respondiendo las preguntas en los comentarios.
> Secciones [REQUERIDO]: esenciales para que Claude funcione bien.
> Secciones [OPCIONAL]: mejoran la calidad de las respuestas.

---

## SECCIÓN 1: Infraestructura [REQUERIDO]

### ¿Dónde está tu instancia de n8n?

- **URL de n8n:** <!-- Ejemplo: https://n8n.tudominio.com -->
- **Método de autenticación:** <!-- API Key / Basic Auth / Sin auth (local dev) -->
- **Versión de n8n:** <!-- Ejecuta en tu VPS: n8n --version, o revisa Settings > About -->

### ¿Usas Coolify como orquestador?

- **Coolify activo:** <!-- Sí / No -->
- **URL de Coolify:** <!-- Ejemplo: https://coolify.tudominio.com -->
- **MCP de Coolify configurado:** <!-- Sí / No / En proceso -->

### ¿Qué base de datos usa este proyecto?

- **BD principal:** <!-- Supabase / Airtable / PostgreSQL / MySQL / Google Sheets / Otra -->
- **URL o endpoint:** <!-- ej: https://xyz.supabase.co -->
- **Nombre de la credencial en n8n:** <!-- El nombre exacto como aparece en n8n Credentials -->

### ¿Hay microservicios externos que tus workflows usan?

<!-- Lista cada uno — si no tienes, escribe "Ninguno":
- Nombre: [nombre del servicio]
  URL externa: https://[servicio].[dominio]
  URL interna (red Docker Coolify): http://[container-name]:[puerto]
  Propósito: [qué hace: procesamiento de video, conversión PDF, etc.]
-->

### ¿Tienes Chrome profile configurado para Playwright?

- **Perfil Chrome creado:** <!-- Sí / No -->
- **Directorio del perfil:** <!-- Ejemplo: Profile 2 (resultado de ls ~/Library/.../Chrome/) -->
- **Servicios con login guardado en ese perfil:** <!-- n8n, Coolify, otros -->

---

## SECCIÓN 2: Cliente / Proyecto [REQUERIDO]

### ¿Para qué o quién es este proyecto?

- **Nombre del cliente/proyecto:** <!-- Ejemplo: Dopla, Upper Edge PM, MBM Investments -->
- **Tipo de negocio:** <!-- SaaS / Agencia / E-commerce / Servicios profesionales / Interno -->
- **Industria:** <!-- Para calibrar vocabulario, reglas y prioridades -->

### Describe el proceso principal que automatizas

<!--
En 2-3 oraciones, qué hace la automatización principal:
Ejemplo: "Recibe webhooks del SaaS cuando un usuario pide generar contenido,
determina el tipo (carrusel/reel/post), llama al sub-workflow correspondiente,
descuenta créditos y notifica al frontend con el resultado."
-->

### Reglas de negocio críticas que Claude NUNCA debe violar

<!--
Ejemplo: "Nunca activar un workflow sin validación previa"
Ejemplo: "El webhook debe responder en < 2 segundos (respuesta async siempre)"
Ejemplo: "Las credenciales de los tenants se leen siempre desde la tabla tenants — nunca hardcodeadas"
Ejemplo: "No hacer UPDATE sin WHERE — siempre filtrar por tenant_id"
-->

### ¿Es multi-tenant?

- **Multi-tenant:** <!-- Sí / No -->
- **Cómo se identifica el tenant:** <!-- Campo: tenant_id en el body / subdomain / API key del header -->
- **Dónde están las credenciales por tenant:** <!-- Tabla "tenants" en Supabase / Airtable / Env vars -->

---

## SECCIÓN 3: Recursos Frecuentes [REQUERIDO]

### Credenciales disponibles en n8n (nombres exactos)

<!-- Lista las credenciales configuradas. El nombre debe ser EXACTAMENTE como aparece en n8n:
| Nombre en n8n              | Servicio      | Para qué se usa             |
|----------------------------|---------------|-----------------------------|
| OpenRouter API             | OpenRouter    | LLM routing, generación     |
| Supabase HTTP — Prod       | Supabase      | Base de datos principal     |
| Gmail OAuth2 — Carlos      | Gmail         | Notificaciones y correos    |
-->

### Nodos que más usas en este proyecto

<!-- Marca con [x] los que aplican y agrega notas:
- [ ] Webhook — path habitual: /[ruta]
- [ ] HTTP Request — URL base principal: https://...
- [ ] Switch — criterio principal: [campo que decide la rama]
- [ ] IF — validaciones típicas: [campos que validas]
- [ ] Code (JS) — para: [tipo de transformaciones]
- [ ] executeWorkflow — sub-workflows activos: [N] workflows
- [ ] Set — para: mapeo y preparación de datos
- [ ] Schedule — frecuencia: [cron expression]
- [ ] respondToWebhook — patrón: [async siempre / sync cuando]
- [ ] AI (LLM node) — modelo preferido: [GPT-4o-mini / Claude Haiku / etc.]
-->

### Integraciones activas en este proyecto

<!-- Marca las que aplican:
- [ ] OpenAI / Anthropic / OpenRouter
- [ ] Supabase / Firebase / Airtable / PostgreSQL
- [ ] Slack / Discord / Telegram
- [ ] Gmail / Resend / SendGrid
- [ ] Stripe / PayPal
- [ ] Google Sheets / Drive / Calendar
- [ ] HeyGen / ElevenLabs / Apify / Kie.ai
- [ ] Otro: [nombre y URL base]
-->

---

## SECCIÓN 4: Convenciones [OPCIONAL — muy recomendado]

### Naming de workflows

<!-- Define el patrón que usas:
Opciones comunes:
  A: "[Cliente] — [Proceso] — [Tipo]"
     Ejemplo: "Dopla — Contenido — ROUTER", "Dopla — Contenido — Carrusel"
  B: "[Proceso] / [Sub-proceso]"
     Ejemplo: "Lead Management / Calificación"
  C: "[Verbo] [Objeto]"
     Ejemplo: "Calificar Lead", "Enviar Newsletter"
-->
**Tu patrón:** <!-- ROUTER: [patrón], Sub-workflows: [patrón] -->

### Naming de webhooks (paths)

<!-- Ejemplo: /[cliente]/[proceso]/[acción]
dopla/content/generate, dopla/ffmpeg/callback, leads/qualify
-->
**Tu patrón:** <!-- /[...] -->

### Naming de nodos dentro de workflows

<!-- Qué convención usas:
  A: Verbos cortos ("Validar tenant", "Leer Supabase", "Despachar sub")
  B: Numerados ("1. Validar", "2. Enriquecer", "3. Switch")
  C: Mixto: zona + acción ("ENTRADA: Validar", "LÓGICA: Switch")
-->
**Tu convención:** <!-- Describe -->

### Idioma en n8n

- **Nombres de workflows:** <!-- Español / Inglés / Mixto -->
- **Sticky notes:** <!-- Español / Inglés -->
- **Nombres de nodos:** <!-- Español / Inglés -->

---

## SECCIÓN 5: Skills y MCPs activos [OPCIONAL]

### Skills instaladas en ~/.claude/skills/

<!-- Marca las que tienes instaladas:
- [ ] n8n-workflow-patterns
- [ ] n8n-mcp-tools-expert
- [ ] n8n-node-configuration
- [ ] n8n-code-javascript
- [ ] n8n-expression-syntax
- [ ] n8n-validation-expert
- [ ] n8n-coolify-fullstack (la exclusiva de este kit)
-->

### MCPs activos en este proyecto

<!-- Los que están en tu .mcp.json para este proyecto:
- [ ] n8n-mcp — URL: [tu URL de n8n]
- [ ] Coolify MCP — URL: [tu URL de Coolify]
- [ ] Playwright MCP — con perfil Chrome: [sí/no]
- [ ] Supabase MCP — project-ref: [ref]
- [ ] Otro: [nombre]
-->

---

## SECCIÓN 6: Instrucciones especiales para Claude [OPCIONAL]

<!--
Cualquier regla específica de cómo trabajas:

Ejemplos:
- "Siempre muéstrame el JSON del nodo antes de aplicarlo al workflow"
- "Pregunta antes de crear sub-workflows nuevos — prefiero confirmar la arquitectura"
- "Usa español para explicaciones, inglés para nombres de nodos y código"
- "Después de crear un workflow, siempre agrega las 4 sticky notes obligatorias"
- "Si hay un error de validación, no lo ignores — siempre proponme el fix"
- "Nunca actives un workflow sin mi confirmación explícita"
-->
