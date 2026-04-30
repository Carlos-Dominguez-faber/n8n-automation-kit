# Best Practices — Patrones por Categoría de Workflow

Fuente: `@n8n/workflow-sdk/src/prompts/best-practices/guides/`

---

## Chatbot

### Diseño del workflow

- Dividir la lógica en pasos manejables
- Usar IF/Switch con fallbacks para manejar inputs inesperados
- La mayoría de chatbots corren vía plataformas externas (Slack, Telegram, WhatsApp) — NO usar el nodo Chat de n8n a menos que el usuario no mencione una plataforma específica

**Regla crítica:** Si el usuario quiere chatear con un workflow Y también triggearlo por otro método (ej: schedule que recopila info + chatbot que consulta esa info), los dos workflows deben estar conectados a través de memoria compartida, vector stores o data storage.

```
Schedule Trigger → News Gathering Agent → [memory node via ai_memory]
Chat Trigger → Chatbot Agent → [MISMO memory node via ai_memory]
Resultado: ambos agentes comparten historial de conversación/contexto
```

**Regla de consistencia:** Usar siempre el mismo tipo de nodo de chat para trigger y respuesta. Si se solicitó Telegram, trigger via Telegram Y responder via Telegram.

### Memoria

- Siempre usar memoria en AI Agent nodes de chatbot
- Incluir metadata en el prompt (timestamp, user ID, session metadata)
- Compartir el mismo nodo de memoria entre múltiples agentes cuando tiene sentido

### Session keys en memory subnodes

No usar `$json` para session key custom — el sub-nodo de memoria no tiene el mismo contexto de predecesor:

| Plataforma | Configuración |
|---|---|
| Telegram | `sessionIdType = customKey`, `sessionKey = nodeJson(telegramTrigger, 'message.chat.id')` |
| Slack | `sessionIdType = customKey`, `sessionKey = nodeJson(slackTrigger, 'event.channel')` |
| WhatsApp | `sessionIdType = customKey`, `sessionKey = nodeJson(whatsAppTrigger, 'messages.0.from')` |
| n8n Chat Trigger | `sessionIdType = fromInput`, omitir sessionKey |

---

## Notificaciones

### Diseño del workflow

Estructura clara en secuencia:

```
Trigger → Data Retrieval/Processing → Condition Check → Notification Action → Post-Notification (log/tracking)
```

**Triggers:** Event-based (webhooks, form submissions) para notificaciones inmediatas; Schedule para monitoreo periódico de condiciones.

**Regla crítica — multi-canal:** Las notificaciones multi-canal deben bifurcarse desde un único condition check hacia múltiples nodos de notificación en paralelo, NO duplicar el workflow completo:

```
IF: threshold exceeded
  → true → Email
  → true → Slack
  → true → SMS
  → false → End/Log
```

### Prevención de notificaciones vacías

Siempre verificar que existan items que merezcan alerta antes de proceder a nodos de notificación:

```
IF: items.length > 0 → [nodos de notificación]
                     → false → End / Log "no alert needed"
```

### Construcción de mensajes

- Email: soporta HTML o plain text, usar subject lines claros
- Slack: usar markdown-like formatting, `\n` para saltos de línea
- SMS: mantener conciso por límite de caracteres, solo texto plano

---

## Scheduling

### Patterns de Schedule Trigger

**Modo Interval:** dropdowns user-friendly para schedules comunes (cada X minutos, diario a las 09:00, semanalmente los lunes).

**Modo Cron Expression:** sintaxis de 5 campos `m h dom mon dow`. Ejemplo: `0 9 * * 1` (cada lunes a las 09:00).

Múltiples schedules pueden combinarse en un único Schedule Trigger node usando múltiples Trigger Rules.

### Lógica condicional sobre cron complejo

**Preferir lógica condicional sobre expresiones cron complejas:**

```
Schedule Trigger → IF (¿es el último día del mes?) → [reporte mensual]
                → IF (¿es festivo?) → Skip
                → Switch (weekday vs weekend) → [procesamiento diferente]
```

Más legible y mantenible que patrones cron complejos.

### Prevención de ejecuciones solapadas

Asegurar que el peor caso de tiempo de ejecución < intervalo de schedule. Para schedules frecuentes, implementar mecanismos de mutex/lock via sistemas externos si es necesario.

### Manejo de timezone

- Si el usuario especifica timezone, setearla en el parámetro `timezone` del Schedule Trigger
- Si menciona horas sin timezone, usar el schedule como se especificó (se aplica el default de la instancia)
- Wait nodes usan la hora del sistema del servidor, no la timezone del workflow

---

## Human-in-the-Loop

### Estructura en 3 etapas

```
1. Automatización inicial → pasos automatizados hasta el punto de decisión
2. Notificación humana → enviar notificación con resume URL
3. Wait Node → pausa hasta respuesta
4. Procesar decisión → IF/Switch basado en input humano
```

Ejemplo:
```
Trigger → Generate Content → Email (con resume URLs) → Wait Node → IF (decisión) → Publish/Reject
```

### Wait Node — Modos de reanudación

| Modo | Cuándo usarlo |
|---|---|
| `After Time Interval` | Delay fijo (NO para decisiones humanas) |
| `At Specified Time` | Fecha/hora específica (NO para decisiones humanas) |
| `On Webhook Call` | URL accedida — ideal para aprobaciones via link |
| `On Form Submitted` | Formulario n8n — mejor para input estructurado |

**Para human-in-the-loop usar:** `On Webhook Call` o `On Form Submitted`.

### URL de reanudación

**CRÍTICO:** Siempre incluir `$execution.resumeUrl` en los mensajes de notificación. Esta URL única reanuda la ejecución específica cuando se accede.

```
Email body: "Aprobar: {{ $execution.resumeUrl }}?approved=true
             Rechazar: {{ $execution.resumeUrl }}?approved=false"
```

### Configuración de Webhook (en Wait node)

- HTTP method: GET para links simples, POST para datos
- Habilitar "Ignore Bots" para prevenir que escáneres de email/bots activen el resume
- Usar Webhook Suffix para múltiples puntos de wait en el mismo workflow

### Timeout obligatorio

Siempre configurar "Limit Wait Time" para evitar esperas infinitas:
- Setear duración máxima (ej: 48 horas)
- O especificar deadline absoluta
- Manejar el caso de timeout en la lógica del workflow

---

## Data Transformation

### Principios core

- **Estructura:** Siempre seguir el patrón Input → Transform → Output
- **Optimización:** Filtrar y reducir datos temprano para mejorar performance
- **Batch:** Datasets de más de 100 items → usar Split In Batches para evitar timeouts

### Edit Fields (Set) — Puntos clave

- **"Keep Only Set" desactivado** (default): lleva todos los campos forward + los definidos
- **"Keep Only Set" activado:** SOLO los campos definidos pasan → riesgo de pérdida de datos
- Tip de testing: usar fallback `{{ $json.name || 'Jane Doe' }}` para probar con datos del trigger

### IF vs Filter

- **IF node:** procesamiento condicional y ruteo (dos caminos: true/false)
- **Filter node:** filtrar items basado en condiciones → elimina items, no rutea

**Regla:** Usar IF early para validar inputs y remover datos malos antes de procesar.

---

## Resumen de patrones por tipo

| Categoría | Trigger recomendado | Nodo clave | Anti-patrón común |
|---|---|---|---|
| Chatbot | Chat Trigger o Telegram/Slack Trigger | AI Agent + Memory | No compartir memoria entre agentes relacionados |
| Notificación | Webhook o Schedule | IF + Email/Slack/SMS | Duplicar el workflow para multi-canal |
| Scheduling | Schedule Trigger | IF/Switch para lógica condicional | Cron expressions complejas en lugar de IF |
| Human-in-the-loop | Webhook | Wait node | No incluir resumeUrl, no configurar timeout |
| Data transformation | Manual o Schedule | Set + Filter + Split In Batches | No filtrar temprano, no usar batch para datasets grandes |
