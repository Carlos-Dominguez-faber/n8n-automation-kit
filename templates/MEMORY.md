# MEMORY.md — Autoaprendizaje del Proyecto n8n

> Sistema de memoria acumulada para proyectos de automatización n8n.
> Claude actualiza este archivo automáticamente al final de cada sesión donde:
>   - Se resolvió un error de configuración o validación
>   - Se descubrió un gotcha del setup específico
>   - Un patrón de workflow funcionó especialmente bien
>   - Una credencial tuvo problemas o comportamiento inesperado
>
> Formato de entrada: [YYYY-MM-DD] — [resumen en una línea]
> Entradas más recientes primero en cada sección.

---

## ERRORES ENCONTRADOS Y FIXES

> Errores de validación, configuración o ejecución con su solución exacta.
> El campo "Afecta a" ayuda a saber qué otros workflows revisar.

<!--
### [YYYY-MM-DD] — [descripción del error en una línea]

**Contexto:** [qué workflow, qué nodo]
**Síntoma:** [qué se veía, qué mensaje de error]
**Causa raíz:** [por qué ocurrió]
**Fix aplicado:**
```
[código o configuración que resolvió el problema]
```
**Aprendizaje:** [regla general que se puede aplicar en el futuro]
**Afecta a:** [otros workflows o nodos donde puede ocurrir lo mismo]
-->

---

## CREDENCIALES PROBLEMÁTICAS

> Credenciales que han causado problemas específicos y cómo se resolvieron.

<!--
### [YYYY-MM-DD] — [nombre de la credencial] — [problema]

**Credencial en n8n:** [nombre exacto]
**Problema:** [descripción del comportamiento inesperado]
**Señal de alarma:** [qué nodo/error aparece cuando falla]
**Fix:** [cómo resolverlo]
**Fecha de vencimiento/rotación (si aplica):** [fecha]
**Nota:** [cualquier información adicional sobre esta credencial]
-->

---

## PATRONES QUE FUNCIONARON

> Patrones de workflow o configuraciones que dieron resultado en este proyecto.

<!--
### [YYYY-MM-DD] — [nombre del patrón]

**Contexto:** [dónde se aplicó]
**Problema que resolvía:** [qué se intentaba solucionar]
**Solución implementada:**
```
[descripción del patrón o snippet de código]
```
**Resultado medible:** [tiempo de respuesta, tasa de éxito, etc.]
**Reutilizable en:** [otros workflows donde aplicar este patrón]
-->

---

## GOTCHAS DEL SETUP

> Comportamientos inesperados específicos de esta instalación de n8n, versión, o Coolify.

<!--
### [YYYY-MM-DD] — [título del gotcha]

**Setup afectado:** [versión de n8n, Coolify, VPS]
**Síntoma:** [qué pasó inesperadamente]
**Causa:** [por qué ocurre en este setup específico]
**Fix o workaround:** [cómo evitarlo o resolverlo]
**Alternativa si el fix no es posible:** [plan B]
-->

---

## NODOS CON COMPORTAMIENTO ESPECIAL

> Nodos que en este proyecto se comportan diferente a la documentación oficial.

<!--
### [Nombre del nodo] — [comportamiento especial]

**Versión de n8n:** [versión donde se observó]
**Comportamiento documentado:** [lo que dice la doc]
**Comportamiento real en este setup:** [lo que realmente hace]
**Workaround:** [cómo adaptarse a este comportamiento]
**¿Reportado a n8n?:** Sí / No / Pendiente
-->

---

## CONFIGURACIONES GUARDADAS

> Configuraciones específicas que tardaron en encontrar — guardadas para reusar sin buscar de nuevo.

<!--
### [Nombre descriptivo de la configuración]

**Para qué nodo / situación:** [descripción]
**Configuración:**
```json
{
  // La configuración exacta que funciona
}
```
**Nota:** [cualquier condición especial para que funcione]
-->

---

## HISTORIAL DE SESIONES

> Log de sesiones de trabajo relevantes con Claude Code.

<!--
### [YYYY-MM-DD] — [resumen en una línea de qué se hizo]

**Duración aproximada:** [X minutos]
**Qué se hizo:**
- [acción 1]
- [acción 2]
**Workflows creados/modificados:** [lista]
**Microservicios tocados:** [lista]
**Errores encontrados y resueltos:** [número]
**Pendiente para próxima sesión:**
- [ ] [tarea 1]
- [ ] [tarea 2]
-->
