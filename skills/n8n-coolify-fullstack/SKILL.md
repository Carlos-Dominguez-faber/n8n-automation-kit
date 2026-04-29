---
name: n8n-coolify-fullstack
description: >
  Gestión fullstack de automatizaciones n8n en infraestructura Coolify self-hosted.
  Combina: Coolify MCP para gestionar microservicios, sticky notes para documentación
  visual obligatoria en workflows, validación dual MCP + Playwright E2E con sesión
  Chrome persistente, sub-workflows estructurados (patrón Router → Subs), e integración
  de microservicios Docker con n8n. Activar cuando el usuario gestione n8n en VPS con
  Coolify, trabaje con sub-workflows complejos, necesite validación visual E2E, o quiera
  documentar workflows con sticky notes estructuradas.
---

# n8n Coolify Fullstack

Stack completo: n8n + Coolify + sub-workflows + validación E2E + microservicios

Este skill combina 5 disciplinas que trabajan juntas en proyectos n8n de producción:

1. **Coolify MCP** — gestionar microservicios sin salir de Claude Code → [coolify-microservices.md](coolify-microservices.md)
2. **Sticky Notes** — documentación visual obligatoria dentro de workflows → [sticky-notes-patterns.md](sticky-notes-patterns.md)
3. **Validación Dual** — MCP validate + Playwright E2E con sesión Chrome → [dual-layer-validation.md](dual-layer-validation.md)
4. **Sub-Workflows** — arquitectura modular, invocación y debug → [sub-workflow-patterns.md](sub-workflow-patterns.md)
5. **Microservice Integration** — Docker en Coolify que n8n llama por HTTP → [microservice-integration.md](microservice-integration.md)

---

## Cuándo usar cada módulo

| Situación | Módulo |
|---|---|
| Necesito deployar/reiniciar/ver logs de un microservicio | coolify-microservices.md |
| Quiero documentar un workflow visualmente con sticky notes | sticky-notes-patterns.md |
| Tengo que validar que un workflow funciona (estructura + UI) | dual-layer-validation.md |
| Diseñando arquitectura con sub-workflows | sub-workflow-patterns.md |
| Conectar un servicio Docker a n8n via HTTP | microservice-integration.md |

---

## Flujo maestro de un proyecto completo

```
1. DISEÑO
   └── Subagente workflow-architect genera el plan
       ├── Define nodos, sub-workflows, sticky notes
       └── Produce secuencia de construcción ordenada

2. BUILD
   ├── n8n MCP crea los workflows (n8n_create_workflow)
   ├── Sticky notes se añaden en cada zona crítica
   └── Coolify MCP verifica microservicios de soporte

3. VALIDATE
   ├── MCP: validate_workflow() — estructura y configuración
   └── Playwright E2E: screenshot + snapshot visual del canvas

4. OPERATE
   ├── Coolify MCP: monitoring y restart de microservicios
   └── n8n executions: tracking de ejecuciones fallidas
```

---

## Reglas que SIEMPRE aplican en este stack

1. **Un sticky note por zona funcional** — mínimo 2 por workflow (header + output)
2. **Validación dual antes de activar** — nunca activar sin pasar ambas capas
3. **tenant_id fluye siempre** — en proyectos multi-tenant, en TODOS los nodos
4. **Referenciar nodos por nombre** — `$('Nombre').item.json.campo`, nunca `$json`
5. **Microservicio: verificar antes de llamar** — check status en Coolify antes del HTTP Request

---

## Integración con otros skills de n8n

Este skill se apoya en las 6 skills base:
- Para buscar nodos: **n8n-mcp-tools-expert**
- Para arquitectura: **n8n-workflow-patterns**
- Para validación de errores: **n8n-validation-expert**
- Para expresiones: **n8n-expression-syntax**
- Para código JS: **n8n-code-javascript**
- Para config de nodos: **n8n-node-configuration**
