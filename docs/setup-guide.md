# Guía de Instalación — n8n-automation-kit

## Prerrequisitos

- **n8n self-hosted** corriendo y accesible (Coolify, Docker, o local)
- **Claude Code** instalado: `npm install -g @anthropic-ai/claude-code`
- **Node.js 18+** para npx
- **Chrome** instalado (para Playwright con sesión persistente)

---

## Paso 1: Habilitar la API de n8n

1. En tu instancia de n8n: **Settings → API**
2. Toggle: **Enable API** → ON
3. Click **Create API Key** → copiar la key (solo se muestra una vez)
4. Guardar en un lugar seguro

---

## Paso 2: Instalar el MCP de n8n

Agregar al `.mcp.json` de tu proyecto (o al `settings.json` global de Claude Code):

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["-y", "n8n-mcp"],
      "env": {
        "N8N_API_URL": "https://TU_N8N_URL/",
        "N8N_API_KEY": "TU_API_KEY_AQUI",
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true"
      }
    }
  }
}
```

**Verificar:** Reinicia Claude Code → escribe `/mcp` → debe aparecer `n8n-mcp ✅ connected`

---

## Paso 3: Instalar las skills de n8n

```bash
# Clonar el repo
git clone https://github.com/getforja/n8n-automation-kit
cd n8n-automation-kit

# Instalar las skills base
cp -r skills/n8n-coolify-fullstack ~/.claude/skills/

# Las otras 6 skills se instalan desde el marketplace de Claude Code:
# En Claude Code: /add-skill n8n-workflow-patterns
# O manualmente (si ya las tienes):
ls ~/.claude/skills/ | grep n8n
```

**Verificar:** Reinicia Claude Code → las skills aparecen disponibles automáticamente.

---

## Paso 4: Configurar Playwright con Chrome profile

### 4a. Crear el perfil Chrome

1. Abre Chrome
2. Click en tu avatar → **Agregar perfil**
3. Nombre: `N8N Dev` (o el nombre de tu proyecto)
4. En ese perfil: inicia sesión en tu n8n y Coolify
5. Activa "Guardar contraseñas" en ese perfil

### 4b. Encontrar el directorio del perfil

```bash
ls ~/Library/Application\ Support/Google/Chrome/ | grep -i profile
# Output ejemplo: Default  Profile 2  Profile 3
# El nuevo perfil suele ser "Profile 2" o el número más alto
```

### 4c. Agregar Playwright al .mcp.json

```json
{
  "playwright": {
    "command": "npx",
    "args": [
      "@playwright/mcp@latest",
      "--browser=chrome",
      "--user-data-dir=/Users/TU_USUARIO/Library/Application Support/Google/Chrome/Profile 2"
    ]
  }
}
```

> Reemplaza `TU_USUARIO` con tu usuario de macOS y `Profile 2` con tu perfil real.

---

## Paso 5: Instalar Coolify MCP (opcional, si usas Coolify)

```json
{
  "coolify": {
    "command": "npx",
    "args": ["-y", "@masonator/coolify-mcp"],
    "env": {
      "COOLIFY_ACCESS_TOKEN": "TU_COOLIFY_TOKEN",
      "COOLIFY_BASE_URL": "https://coolify.tudominio.com/"
    }
  }
}
```

**Obtener el token:** Coolify → Profile → API Tokens → Create Token

---

## Paso 6: Configurar el CLAUDE.md de tu proyecto

```bash
# Copiar el template al directorio raíz de tu proyecto n8n
cp templates/CLAUDE.md /tu/proyecto/CLAUDE.md
cp templates/MEMORY.md /tu/proyecto/MEMORY.md

# Editar CLAUDE.md y rellenar las 6 secciones
# (Secciones 1-3 son REQUERIDAS para que Claude funcione bien)
```

---

## Paso 7: Copiar workflows de referencia (opcional)

```bash
# Los workflows de Dopla como referencia de arquitectura
ls workflows/dopla-base/

# Importar en n8n: Settings → Workflows → Import
# O via Claude Code: "importa el workflow router.json en mi n8n"
```

---

## Verificación final

```bash
# En Claude Code, verificar todo está conectado:
# 1. MCP de n8n
/mcp
# → n8n-mcp ✅ connected

# 2. Skills cargadas (Claude las reporta al inicio de sesión)
# → Aparecen en el contexto de Claude automáticamente

# 3. Primer workflow via MCP
# Escribe en Claude Code:
# "lista mis workflows activos en n8n"
# → Claude debe devolver tus workflows reales

# 4. Playwright con sesión
# Escribe en Claude Code:
# "navega a mi n8n y toma un screenshot de la lista de workflows"
# → Claude debe abrir Chrome con tu sesión activa
```

---

## Estructura del .mcp.json completo de referencia

Ver `templates/.mcp.json` en el repo para el archivo completo con todos los MCPs pre-configurados con placeholders.
