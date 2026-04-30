# Workflow Rules — Reglas Estrictas de Generación

Fuente: `@n8n/workflow-sdk/src/prompts/sdk-reference/workflow-rules.ts`

Seguir estas reglas estrictamente al generar o editar cualquier workflow n8n.

---

## Regla 1: Autenticación siempre con `newCredential()`

Cuando un nodo necesita credenciales, NUNCA usar placeholders, API keys falsas o valores hardcodeados.

```
✅ credentials: { slackApi: newCredential('Slack Bot') }
❌ credentials: { slackApi: { token: "xoxb-fake-token" } }
```

El tipo de credencial debe coincidir con lo que el nodo espera.

---

## Regla 2: Confía en listas vacías — no sintetices items falsos

Cuando una query devuelve 0 items, los nodos downstream simplemente no ejecutan en esa corrida. Para triggers con schedule o polling, esto es la señal correcta de "nada que hacer esta vez".

### Anti-patrones que NUNCA usar

**`alwaysOutputData: true` solo para "mantener la cadena viva":**
```
❌ Forzar un item vacío {} downstream
   → Causa: undefined reads, HTTP calls a GET undefined, crashes en Code nodes
✅ Dejar que la cadena pare limpiamente
```

**IF gate antes de un loop para chequear "¿hay items?":**
```
❌ IF (items.length > 0) → splitInBatches
   → El IF gate es redundante y agrega superficie de fallo
✅ splitInBatches, filter y loops ya hacen no-op en input vacío
```

### Cuándo SÍ usar `alwaysOutputData: true`

Solo cuando genuinamente necesitas que una rama downstream corra en el caso "vacío". Ejemplo: una rama dedicada de "no se encontraron resultados" que envía una notificación.

En ese caso, combinar con un IF que chequee explícitamente el caso vacío y rutee en consecuencia. **Nunca usarlo como default.**

### Para dropear items inválidos mid-pipeline

```
✅ filter node → rechaza items → emite 0 items → la cadena para limpiamente
❌ IF + splitInBatches composition solo para filtrar
```

---

## Regla 3: `executeOnce: true` para nodos de ejecución única

Cuando un nodo recibe N items pero debe ejecutarse solo una vez (no N veces), usar `executeOnce: true`.

**Casos comunes:**
- Enviar una notificación resumen (no una por item)
- Generar un reporte
- Llamar una API que no necesita ejecución por item

```json
{
  "config": {
    "executeOnce": true
  }
}
```

---

## Regla 4: Elegir el primitivo de control de flujo correcto

| Caso | Primitivo correcto |
|---|---|
| Loop por item con side effects (fetch, embed, write) | `splitInBatches` con `batchSize: 1` → loop back via `nextBatch`. Sin IF gate antes. |
| Dropear items que no cumplen un predicado | `filter`. Emite 0 items si nada coincide, la cadena para limpiamente. |
| Dos rutas mutuamente exclusivas que hacen trabajo real | `IF` (onTrue / onFalse) |
| Múltiples rutas mutuamente exclusivas por un valor | `switch` (onCase) |

### Control flow anidado

El control flow anidado es válido cuando la semántica lo requiere genuinamente:

```
ifNode.onTrue(loopBuilder)
switchNode.onCase(0, loopBuilder)
splitInBatches(sib).onEachBatch(ifElseBuilder)
```

No usar como workaround para manejo de listas vacías.

---

## Resumen de anti-patrones prohibidos

| Anti-patrón | Por qué es malo | Alternativa |
|---|---|---|
| `alwaysOutputData: true` como default | Causa undefined reads y crashes | Dejar que la cadena pare |
| IF gate antes de loop | Redundante, superficie de fallo extra | splitInBatches ya hace no-op |
| Hardcodear credenciales | Seguridad, portabilidad | `newCredential()` |
| Code node para filtrar items | Más frágil que nodo nativo | Filter node |
| ejecutar N veces un nodo de resumen | Envía N notificaciones en lugar de 1 | `executeOnce: true` |
