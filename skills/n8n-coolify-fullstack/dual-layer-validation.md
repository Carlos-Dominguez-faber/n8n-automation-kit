# Validación Dual: MCP validate + Playwright E2E

## Por qué dos capas son necesarias

| Capa | Detecta | No detecta |
|---|---|---|
| MCP validate_workflow | Estructura JSON, nodos mal configurados, campos faltantes, conexiones rotas | Si la UI muestra el canvas correcto |
| Playwright E2E | Lo que el usuario ve en el browser, sticky notes visibles, estado del toggle | Problemas internos de configuración JSON |

Son complementarias. Un workflow puede pasar MCP y fallar visualmente (o viceversa).

---

## SETUP REQUERIDO: Sesión Chrome con login activo

Sin configuración previa, Playwright abre Chrome sin sesión y ve la pantalla de login, no el workflow.

### Setup inicial (una sola vez por proyecto)

**Paso 1:** Crear perfil Chrome dedicado
```
Chrome → chrome://settings/ → Perfiles → Agregar perfil
Nombre: "N8N Dev" (o el nombre del proyecto)
```

**Paso 2:** Loguearse en n8n y Coolify en ese perfil
```
- Abrir n8n (ej: https://pod-1.dopla.app) → hacer login
- Abrir Coolify → hacer login
- Activar "Guardar contraseña" en Chrome
```

**Paso 3:** Encontrar el directorio del perfil
```bash
ls ~/Library/Application\ Support/Google/Chrome/ | grep -i profile
# Ejemplo output: Default  Profile 1  Profile 2  Profile 3
# El perfil nuevo generalmente es "Profile 2" o "Profile 3"
```

**Paso 4:** Configurar en .mcp.json (nunca en git — las rutas son locales)
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

**Paso 5:** Guardar variables en .env del proyecto
```bash
PLAYWRIGHT_CHROME_PROFILE_NAME="Profile 2"
PLAYWRIGHT_CDP_PORT=9222
```

### Modo alternativo: CDP (Chrome DevTools Protocol)

Para conectar a Chrome ya corriendo con sesión activa:

```bash
# Lanzar Chrome con remote debugging:
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir="$HOME/Library/Application Support/Google/Chrome/Profile 2" \
  --no-first-run

# Luego en .mcp.json:
"playwright": {
  "command": "npx",
  "args": ["@playwright/mcp@latest", "--cdp-endpoint=http://localhost:9222"]
}
```

> Modo CDP permite que tanto el usuario como Playwright vean el MISMO Chrome en tiempo real — útil para debug en clase o demostraciones.

---

## Capa 1: Validación MCP

### Secuencia estándar de validación

```
1. validate_workflow({id: "[workflow-id]", profile: "runtime"})
   → Lista de errores y warnings

2. Si hay errores → n8n_autofix_workflow({id, applyFixes: false})
   → Preview de los fixes automáticos disponibles

3. Aprobar → n8n_autofix_workflow({id, applyFixes: true})
   → Aplica los fixes

4. validate_workflow({id}) de nuevo → confirmar limpio

5. Si quedan errores manuales → fix con n8n_update_partial_workflow
```

### Profiles de validación

| Profile | Cuándo usar |
|---|---|
| `minimal` | Durante desarrollo rápido — solo campos requeridos |
| `runtime` | Antes de activar — recomendado para pre-deploy |
| `ai-friendly` | Si hay muchos falsos positivos |
| `strict` | Antes de entregar a cliente — máxima rigurosidad |

### Checklist MCP pre-producción

- [ ] `validate_workflow` sin errores con profile "runtime"
- [ ] Todos los nodos tienen nombre descriptivo (no "HTTP Request 1")
- [ ] No hay stale connections (`cleanStaleConnections` si necesario)
- [ ] Workflow tiene estado `active: true`
- [ ] Hay un workflow de error trigger conectado (o al menos un nodo de notificación en el catch)

---

## Capa 2: Playwright E2E Visual

### Cuándo usar Playwright para validar n8n

- Después de importar un workflow — verificar que se importó visualmente bien
- Antes de entregar a cliente — screenshot como evidencia de entrega
- Cuando el cliente reporta "el workflow no funciona" — ver con sus ojos
- Verificar que los sticky notes están presentes y son legibles
- Confirmar que el toggle de activación está en verde

### Secuencia estándar

```javascript
// Claude invoca Playwright MCP en este orden:

// 1. Navegar al workflow
browser_navigate({url: "https://pod-1.dopla.app/workflow/[WORKFLOW_ID]"})

// 2. Esperar a que cargue el canvas
browser_wait_for({selector: ".workflow-canvas", timeout: 10000})

// 3. Screenshot general
browser_take_screenshot()
// → Verificar visualmente: nodos en canvas, sticky notes visibles, toggle activo

// 4. Snapshot del accessibility tree (para verificar nodos por texto)
browser_snapshot()
// → Buscar en el texto: nombres de nodos, contenido de sticky notes

// 5. Opcional: verificar ejecución
browser_click({selector: "[data-test='execute-workflow-button']"})
browser_take_screenshot()
// → Capturar el resultado de la ejecución
```

### Validaciones visuales específicas para n8n

```javascript
// Verificar que el workflow está activo (toggle verde):
// En el screenshot buscar visualmente el toggle en la esquina superior

// Verificar sticky notes en el canvas:
// browser_snapshot() → buscar texto del sticky note header en el árbol

// Verificar nodo específico existe:
// browser_snapshot() → buscar el nombre del nodo en el accessibility tree
// Si no aparece → el nodo puede no estar correctamente conectado o nombrado
```

---

## Plantilla de reporte de validación dual

```markdown
## Validación Workflow: [Nombre]
**Fecha:** [YYYY-MM-DD]
**Ejecutado por:** Claude Code + n8n-coolify-fullstack skill

### Capa 1: MCP Validation
- validate_workflow (runtime): ✅ Sin errores | ❌ [N] errores
- autofix aplicado: Sí / No
- Errores manuales resueltos: [lista]
- Status final: active / inactive

### Capa 2: Playwright E2E
- Browser navegó al workflow: ✅ / ❌
- Canvas cargó correctamente: ✅ / ❌
- Sticky notes visibles: ✅ [N] notas | ❌ No encontradas
- Toggle activo (verde): ✅ / ❌
- Screenshot guardado: Sí / No

### Resultado: ✅ APROBADO / ❌ RECHAZADO
[Notas adicionales si hay items pendientes]
```

---

## Falsos positivos comunes en validate_workflow

No todos los errores MCP son reales. Ignorar:
- `"Missing error handling"` — aceptable en workflows simples sin procesos críticos
- `"No retry logic"` — aceptable si la API llamada tiene su propia idempotencia
- `"Unbounded query"` — aceptable para datasets conocidamente pequeños

Nunca ignorar:
- `missing_required` — un campo requerido realmente falta
- `invalid_reference` — referencia a nodo que no existe (workflow roto)
- `type_mismatch` — tipo de dato incorrecto causará fallo en runtime
