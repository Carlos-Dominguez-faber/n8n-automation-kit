# n8n-automation-kit

Kit completo para conectar Claude Code con n8n en proyectos reales. Incluye MCPs pre-configurados, skills especializadas, un subagente arquitecto de workflows, y templates listos para usar.

Creado por [Carlos Domínguez](https://github.com/Carlos-Dominguez-faber) — [Imperio Digital](https://imperiodigital.club) · VibeCoding

---

## Qué incluye

```
n8n-automation-kit/
├── .mcp.json                        ← Template de MCPs (editar con tus credenciales)
├── CLAUDE.md                        ← Cuestionario de contexto para Claude (rellenar)
├── MEMORY.md                        ← Sistema de autoaprendizaje (Claude lo actualiza solo)
├── skills/
│   ├── n8n-coolify-fullstack/       ← Skill exclusiva — instalar en ~/.claude/skills/
│   └── make-to-n8n/                 ← Skill para migrar escenarios Make → n8n
├── agents/
│   └── workflow-architect/          ← Subagente que diseña planes completos de workflows
├── docs/
│   ├── mcp-vs-api.md                ← Cuándo usar MCP vs API REST
│   └── setup-guide.md               ← Guía de instalación detallada
├── workflows/
│   ├── README.md                    ← Instrucciones para poner tus workflows aquí
│   └── starters/
│       └── webhook-router-starter.json  ← Workflow base con sticky notes incluidas
└── workshop/
    └── index.html                   ← Workshop guide interactivo de la clase
```

---

## Inicio en 3 pasos

### Paso 1 — Clonar y abrir en Claude Code

```bash
git clone https://github.com/Carlos-Dominguez-faber/n8n-automation-kit mi-proyecto-n8n
cd mi-proyecto-n8n
claude   # Abrir Claude Code desde aquí
```

> **Por qué trabajar desde este directorio:** Claude Code lee el `.mcp.json`, `CLAUDE.md` y `MEMORY.md` desde la raíz del proyecto. Al abrir Claude Code aquí, ya tiene todo el contexto configurado.

### Paso 2 — Instalar las skills

```bash
cp -r skills/n8n-coolify-fullstack ~/.claude/skills/
cp -r skills/make-to-n8n ~/.claude/skills/
```

Reinicia Claude Code. Las skills aparecen disponibles automáticamente.

### Paso 3 — Configurar tus credenciales

**3a. Copiar el template de MCP y rellenar con tus datos:**

```bash
cp .mcp.json.example .mcp.json
```

> `.mcp.json` está en `.gitignore` — tus credenciales nunca se subirán al repo.

Editar `.mcp.json` y reemplazar los placeholders:

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "env": {
        "N8N_API_URL": "https://TU_N8N_URL/",   ← la / al final es obligatoria
        "N8N_API_KEY": "TU_API_KEY"
      }
    }
  }
}
```

Para obtener tu API key: **n8n → Settings → API → Enable API → Create API Key**

**3b. Rellenar `CLAUDE.md`** — las secciones 1, 2 y 3 son las más importantes:
- Sección 1: tu URL de n8n, si usas Coolify, microservicios activos
- Sección 2: nombre del proyecto, proceso que automatizas, reglas de negocio
- Sección 3: credenciales disponibles en n8n (nombres exactos), nodos que más usas

**3c. Configurar Playwright con tu perfil de Chrome** (para validación E2E con sesión activa):

```bash
# Encontrar tu perfil de Chrome
ls ~/Library/Application\ Support/Google/Chrome/ | grep -i profile
# Output ejemplo: Profile 2

# En .mcp.json → reemplazar en la sección "playwright":
# --user-data-dir=/Users/TU_USUARIO/Library/Application Support/Google/Chrome/Profile 2
```

> Antes de usar Playwright: crea un perfil Chrome dedicado, inicia sesión en tu n8n y Coolify en ese perfil, y usa esa ruta en el .mcp.json. Así Playwright ya tiene sesión activa cada vez que lo usa Claude.

---

## Verificar que todo funciona

Escribe esto en Claude Code después de configurar:

```
/mcp
```
→ Debe aparecer **n8n-mcp ✅ connected**

Luego prueba:
```
"lista mis workflows activos en n8n"
```
→ Claude debe devolver tus workflows reales via MCP.

---

## Las 6 skills de n8n

Claude las activa automáticamente — no necesitas invocarlas.

| Skill | Se activa cuando... |
|---|---|
| `n8n-workflow-patterns` | Diseñas un workflow nuevo o eliges arquitectura |
| `n8n-mcp-tools-expert` | Usas cualquier herramienta MCP de n8n |
| `n8n-node-configuration` | Configuras un nodo específico |
| `n8n-code-javascript` | Escribes código JS en un Code node |
| `n8n-expression-syntax` | Escribes expresiones `{{ }}` o hay un undefined |
| `n8n-validation-expert` | Hay errores de validación que interpretar |

Estas 6 skills se instalan desde el marketplace de Claude Code. La skill **`n8n-coolify-fullstack`** (la exclusiva de este kit) la instalaste en el Paso 2.

---

## La skill exclusiva: n8n-coolify-fullstack

Combina 5 módulos que trabajan juntos en proyectos de producción:

| Módulo | Qué hace |
|---|---|
| `coolify-microservices.md` | Gestiona microservicios en Coolify MCP desde Claude: list, get, restart, logs |
| `sticky-notes-patterns.md` | 4 zonas de documentación obligatorias con posiciones y colores estándar |
| `dual-layer-validation.md` | Validación en dos capas: MCP (estructura) + Playwright E2E (visual con sesión) |
| `sub-workflow-patterns.md` | Cuándo crear sub-workflows, cómo pasar datos, el gotcha del `type: "string"` |
| `microservice-integration.md` | Integrar microservicios Docker con n8n: patrones síncrono, callback, polling |

---

## La skill de migración: make-to-n8n

Convierte escenarios Make (Integromat) a workflows n8n. Se activa automáticamente cuando pegas un JSON exportado de Make o describes un flujo que quieres migrar.

| Módulo | Qué contiene |
|---|---|
| `SKILL.md` | Tabla de conceptos Make vs n8n, proceso paso a paso, patrones especiales (Iterator, Router, Repeater, Aggregator), ejemplo completo de migración |
| `module-mapping.md` | Mapeo de ~80 módulos Make → nodo n8n exacto, con tipo de equivalencia y snippets para módulos sin equivalente directo |
| `expression-conversion.md` | Traducción completa de sintaxis: `{{N.field}}` → `{{$node["X"].json.field}}`, todas las funciones de texto, fecha, número, array y hashing |

**Cómo usarla:**
```
"tengo este JSON de Make, conviértelo a n8n"
"migra este escenario de Make a un workflow n8n"
"cómo traduzo este módulo de Make al nodo equivalente en n8n"
```

---

## El subagente workflow-architect

Diseña planes completos de implementación en lenguaje natural:

```
"actúa como workflow-architect y diseña el plan para:
 recibir un formulario de contacto por webhook, calificar el lead
 con un Code node, guardar en Google Sheets, y enviar email con
 o sin botón de agenda según la calificación"
```

El subagente produce:
- Diagrama ASCII del flujo
- Tabla de nodos con configuración clave
- Posiciones y contenido de sticky notes por zona
- Secuencia de construcción paso a paso
- Credenciales requeridas y riesgos

---

## MCP vs API REST — cuándo usar cada uno

Ver [docs/mcp-vs-api.md](docs/mcp-vs-api.md) para la comparativa completa.

**Resumen:**
- **MCP**: para construir, editar y validar workflows. Claude entiende n8n semánticamente.
- **API REST directa**: para operar y monitorear. Scripts, CI/CD, ejecuciones en tiempo real.
- **Los dos viven en el mismo `.mcp.json`** — no son excluyentes.

---

## Tus workflows

La carpeta `workflows/` está vacía intencionalmente. Pon ahí tus propios workflows:

```bash
# Exportar desde n8n: ⋮ → Download → guardar el .json aquí
workflows/mi-workflow.json
workflows/sub-mi-proceso.json
```

Hay un workflow base en `workflows/starters/webhook-router-starter.json` — un dispatcher con sticky notes en las 4 zonas obligatorias, listo para adaptar.

---

## CLAUDE.md y MEMORY.md

**`CLAUDE.md`** — Lo rellenas tú una vez. Claude lo lee al inicio de cada sesión y sabe exactamente qué instancia de n8n usas, qué credenciales están disponibles, y qué convenciones seguir. Sin él, Claude empieza sin contexto cada vez.

**`MEMORY.md`** — Lo actualiza Claude solo. Cada vez que encuentra y resuelve un error, descubre un gotcha, o valida un patrón que funciona, agrega una entrada. Con el tiempo acumula el conocimiento específico de tu proyecto.

---

## Coolify MCP (opcional)

Si usas Coolify para alojar n8n y tus microservicios, agrega el MCP de Coolify en `.mcp.json`:

```json
"coolify": {
  "command": "npx",
  "args": ["-y", "@masonator/coolify-mcp"],
  "env": {
    "COOLIFY_ACCESS_TOKEN": "TU_TOKEN",
    "COOLIFY_BASE_URL": "https://coolify.tudominio.com/"
  }
}
```

Token en: **Coolify → Profile → API Tokens → Create**

Con Coolify MCP puedes pedirle a Claude cosas como:
- `"verifica que todos mis microservicios están corriendo"`
- `"reinicia el servicio X"`
- `"muéstrame los últimos logs del servicio Y"`

---

## Requisitos

- **n8n self-hosted** corriendo y accesible (Coolify, Docker, o local)
- **Claude Code** instalado: `npm install -g @anthropic-ai/claude-code`
- **Node.js 18+** (para npx)
- **Chrome** instalado (para Playwright con sesión persistente)

Ver [docs/setup-guide.md](docs/setup-guide.md) para instalación paso a paso.

---

## Licencia

MIT — Úsalo, modifícalo, compártelo.

---

*Creado con Claude Code + [La Forja v3.2.0](https://github.com/Carlos-Dominguez-faber/forge) · Imperio Digital 2026*
