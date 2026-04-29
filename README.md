# n8n-automation-kit

Kit completo para conectar Claude Code con n8n en producción. Incluye MCPs, skills especializadas, subagentes, templates y workflows reales como referencia.

Creado por [Carlos Domínguez](https://github.com/Carlos-Dominguez-faber) para la comunidad de [Imperio Digital](https://imperiodigital.club).

---

## ¿Qué incluye?

| Componente | Descripción |
|---|---|
| **`workshop/index.html`** | Workshop guide interactivo — material de la clase VibeCoding |
| **`skills/n8n-coolify-fullstack/`** | Skill exclusiva: Coolify MCP + sticky notes + validación E2E + sub-workflows |
| **`agents/workflow-architect/`** | Subagente que diseña planes completos de implementación |
| **`templates/CLAUDE.md`** | Template tipo cuestionario para darle contexto a Claude |
| **`templates/MEMORY.md`** | Sistema de autoaprendizaje — Claude lo actualiza solo |
| **`templates/.mcp.json`** | MCP pre-configurado con placeholders (n8n + Coolify + Playwright) |
| **`docs/mcp-vs-api.md`** | Comparativa completa: cuándo usar MCP vs API REST |
| **`docs/setup-guide.md`** | Instalación paso a paso del stack completo |
| **`workflows/dopla-base/`** | Workflows reales de Dopla SaaS como referencia de arquitectura |

---

## Inicio rápido (3 pasos)

### 1. Instalar el MCP de n8n

Agrega esto a tu `.mcp.json` o `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["-y", "n8n-mcp"],
      "env": {
        "N8N_API_URL": "https://TU_N8N_URL/",
        "N8N_API_KEY": "TU_API_KEY",
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true"
      }
    }
  }
}
```

Verifica: `/mcp` en Claude Code → `n8n-mcp ✅ connected`

### 2. Instalar la skill exclusiva

```bash
git clone https://github.com/getforja/n8n-automation-kit
cp -r n8n-automation-kit/skills/n8n-coolify-fullstack ~/.claude/skills/
```

### 3. Configurar tu proyecto

```bash
cp n8n-automation-kit/templates/CLAUDE.md ./CLAUDE.md
cp n8n-automation-kit/templates/MEMORY.md ./MEMORY.md
# Edita CLAUDE.md y rellena las secciones 1-3 (REQUERIDO)
```

---

## Skills disponibles en este kit

| Skill | Qué hace | Cuándo activa Claude la skill |
|---|---|---|
| `n8n-workflow-patterns` | 5 patrones arquitecturales probados | Al diseñar un workflow nuevo |
| `n8n-mcp-tools-expert` | Guía de herramientas MCP | Al usar cualquier tool MCP de n8n |
| `n8n-node-configuration` | Config por tipo de nodo | Al configurar un nodo específico |
| `n8n-code-javascript` | JS en Code nodes (sintaxis $input) | Al escribir código en n8n |
| `n8n-expression-syntax` | Expresiones {{ }} seguras | Al escribir expresiones |
| `n8n-validation-expert` | Interpreta errores de validación | Al haber errores de validación |
| `n8n-coolify-fullstack` | **Skill exclusiva** — Coolify + sticky notes + E2E + subs + microservicios | Al trabajar con el stack completo |

> Las skills 1-6 se instalan vía el marketplace de Claude Code.
> La skill `n8n-coolify-fullstack` se instala desde este repo (paso 2 arriba).

---

## MCPs incluidos en el template

| MCP | Paquete | Para qué |
|---|---|---|
| **n8n MCP** | `n8n-mcp` | Crear, editar y validar workflows |
| **Coolify MCP** | `@masonator/coolify-mcp` | Gestionar microservicios en Coolify |
| **Playwright MCP** | `@playwright/mcp@latest` | Validación E2E visual con sesión Chrome |

---

## Caso de estudio: Dopla SaaS

Los workflows en `workflows/dopla-base/` son adaptaciones sanitizadas del sistema real de [Dopla](https://dopla.app) — un SaaS multi-tenant de generación de contenido con IA.

**Arquitectura:** Webhook → Router → 4 sub-workflows por tipo de contenido (carrusel, post estático, reel, reel UGC)

Úsalos como referencia de:
- Patrón Router → Sub-workflows
- Multi-tenancy con `tenant_id` en cada nodo
- Respuesta async (respondToWebhook inmediato + callback)
- Sticky notes en las 4 zonas obligatorias

---

## Prerrequisitos

- n8n self-hosted (Coolify, Docker, o local)
- Claude Code instalado
- Node.js 18+ (para npx)
- Chrome (para Playwright con sesión persistente)

Ver [docs/setup-guide.md](docs/setup-guide.md) para instalación detallada.

---

## Contribuir

PRs bienvenidos para:
- Nuevos workflows starter en `workflows/starters/`
- Mejoras a la skill `n8n-coolify-fullstack`
- Nuevas entradas en el `node-catalog.md`
- Traducciones de la documentación

---

## Licencia

MIT — Úsalo, modifícalo, compártelo.

---

*Hecho con Claude Code + La Forja v3.2.0 — Imperio Digital 2026*
