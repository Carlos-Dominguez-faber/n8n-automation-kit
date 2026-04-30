# Conversión de Expresiones Make → n8n

Guía completa para traducir la sintaxis de expresiones de Make a n8n.

---

## Regla fundamental

En Make, los datos se referencian por número de módulo:
```
{{1.field}}   ← datos del módulo ID 1
{{2.name}}    ← datos del módulo ID 2
```

En n8n, se referencian por nombre del nodo:
```
{{$node["Google Forms"].json.field}}
{{$node["OpenAI"].json.name}}
{{$json.field}}  ← datos del nodo ACTUAL (el que contiene la expresión)
```

---

## Variables especiales

| Make | n8n | Descripción |
|---|---|---|
| `{{now}}` | `{{$now}}` | Timestamp actual (ISO) |
| `{{timestamp}}` | `{{Date.now()}}` | Unix timestamp en ms |
| `{{randomNumber}}` | `{{Math.random()}}` | Número aleatorio 0-1 |
| `{{uuid}}` | `{{$execution.id}}` | ID único (o `crypto.randomUUID()` en Code) |
| `{{moduleName}}` | `{{$workflow.name}}` | Nombre del workflow |
| `{{scenarioName}}` | `{{$workflow.name}}` | Nombre del escenario |

---

## Funciones de texto

| Make | n8n (expresión) |
|---|---|
| `{{trim(str)}}` | `{{$json.str.trim()}}` |
| `{{lower(str)}}` | `{{$json.str.toLowerCase()}}` |
| `{{upper(str)}}` | `{{$json.str.toUpperCase()}}` |
| `{{length(str)}}` | `{{$json.str.length}}` |
| `{{replace(str; "a"; "b")}}` | `{{$json.str.replace("a", "b")}}` |
| `{{replace(str; /regex/; "b")}}` | `{{$json.str.replace(/regex/, "b")}}` |
| `{{substr(str; 0; 5)}}` | `{{$json.str.substring(0, 5)}}` |
| `{{indexOf(str; "x")}}` | `{{$json.str.indexOf("x")}}` |
| `{{contains(str; "x")}}` | `{{$json.str.includes("x")}}` |
| `{{startsWith(str; "x")}}` | `{{$json.str.startsWith("x")}}` |
| `{{endsWith(str; "x")}}` | `{{$json.str.endsWith("x")}}` |
| `{{split(str; ",")}}` | `{{$json.str.split(",")}}` |
| `{{join(array; ", ")}}` | `{{$json.array.join(", ")}}` |
| `{{capitalize(str)}}` | `{{$json.str.charAt(0).toUpperCase() + $json.str.slice(1)}}` |
| `{{repeat(str; 3)}}` | `{{$json.str.repeat(3)}}` |
| `{{pad(str; 10; "0")}}` | `{{$json.str.padStart(10, "0")}}` |
| `{{ascii(str)}}` | `{{"char".charCodeAt(0)}}` |
| `{{toBinary(str)}}` | En Code node: `Buffer.from(str).toString('base64')` |

---

## Funciones numéricas

| Make | n8n (expresión) |
|---|---|
| `{{add(a; b)}}` | `{{$json.a + $json.b}}` |
| `{{subtract(a; b)}}` | `{{$json.a - $json.b}}` |
| `{{multiply(a; b)}}` | `{{$json.a * $json.b}}` |
| `{{divide(a; b)}}` | `{{$json.a / $json.b}}` |
| `{{ceil(n)}}` | `{{Math.ceil($json.n)}}` |
| `{{floor(n)}}` | `{{Math.floor($json.n)}}` |
| `{{round(n)}}` | `{{Math.round($json.n)}}` |
| `{{abs(n)}}` | `{{Math.abs($json.n)}}` |
| `{{max(a; b)}}` | `{{Math.max($json.a, $json.b)}}` |
| `{{min(a; b)}}` | `{{Math.min($json.a, $json.b)}}` |
| `{{mod(a; b)}}` | `{{$json.a % $json.b}}` |
| `{{power(a; b)}}` | `{{Math.pow($json.a, $json.b)}}` |
| `{{sqrt(n)}}` | `{{Math.sqrt($json.n)}}` |
| `{{sum(array)}}` | `{{$json.array.reduce((a,b) => a+b, 0)}}` |
| `{{average(array)}}` | `{{$json.array.reduce((a,b) => a+b, 0) / $json.array.length}}` |
| `{{toNumber(str)}}` | `{{Number($json.str)}}` |
| `{{toString(num)}}` | `{{String($json.num)}}` |
| `{{formatNumber(n; 2; "."; ",")}}` | En Code: `n.toLocaleString('es-MX', {minimumFractionDigits: 2})` |

---

## Funciones de fecha

| Make | n8n (expresión) |
|---|---|
| `{{now}}` | `{{$now}}` |
| `{{formatDate(now; "DD/MM/YYYY")}}` | `{{DateTime.now().toFormat("dd/MM/yyyy")}}` |
| `{{formatDate(date; "YYYY-MM-DD")}}` | `{{DateTime.fromISO($json.date).toFormat("yyyy-MM-dd")}}` |
| `{{parseDate(str; "MM/DD/YYYY")}}` | `{{DateTime.fromFormat($json.str, "MM/dd/yyyy").toISO()}}` |
| `{{addDays(date; 7)}}` | `{{DateTime.fromISO($json.date).plus({days: 7}).toISO()}}` |
| `{{addHours(date; 2)}}` | `{{DateTime.fromISO($json.date).plus({hours: 2}).toISO()}}` |
| `{{addMonths(date; 1)}}` | `{{DateTime.fromISO($json.date).plus({months: 1}).toISO()}}` |
| `{{dateDifference(d1; d2; "days")}}` | `{{DateTime.fromISO($json.d2).diff(DateTime.fromISO($json.d1), 'days').days}}` |
| `{{startOfMonth(date)}}` | `{{DateTime.fromISO($json.date).startOf('month').toISO()}}` |
| `{{endOfMonth(date)}}` | `{{DateTime.fromISO($json.date).endOf('month').toISO()}}` |
| `{{dayOfWeek(date)}}` | `{{DateTime.fromISO($json.date).weekday}}` (1=Lun, 7=Dom) |

> n8n usa **Luxon** para fechas. `DateTime` está disponible directamente en expresiones.

---

## Funciones de array

| Make | n8n (expresión o Code) |
|---|---|
| `{{length(array)}}` | `{{$json.array.length}}` |
| `{{first(array)}}` | `{{$json.array[0]}}` |
| `{{last(array)}}` | `{{$json.array[$json.array.length - 1]}}` |
| `{{get(array; 2)}}` | `{{$json.array[2]}}` (índice 0-based) |
| `{{contains(array; val)}}` | `{{$json.array.includes(val)}}` |
| `{{map(array; "field")}}` | `{{$json.array.map(i => i.field)}}` |
| `{{filter(array; cond)}}` | En Code: `array.filter(i => i.field === "val")` |
| `{{sort(array; "field")}}` | En Code: `array.sort((a,b) => a.field.localeCompare(b.field))` |
| `{{reverse(array)}}` | `{{[...$json.array].reverse()}}` |
| `{{join(array; ", ")}}` | `{{$json.array.join(", ")}}` |
| `{{distinct(array; "field")}}` | En Code: `[...new Map(array.map(i => [i.field, i])).values()]` |
| `{{arrayDifference(a; b)}}` | En Code: `a.filter(x => !b.includes(x))` |
| `{{arrayUnion(a; b)}}` | En Code: `[...new Set([...a, ...b])]` |
| `{{emptyArray}}` | `{{[]}}` |
| `{{addItem(array; item)}}` | `{{[...$json.array, newItem]}}` |
| `{{removeItem(array; idx)}}` | En Code: `array.filter((_, i) => i !== idx)` |

---

## Funciones condicionales

| Make | n8n |
|---|---|
| `{{if(cond; val1; val2)}}` | `{{condition ? val1 : val2}}` |
| `{{ifempty(val; default)}}` | `{{$json.val ?? "default"}}` |
| `{{and(a; b)}}` | `{{a && b}}` |
| `{{or(a; b)}}` | `{{a \|\| b}}` |
| `{{not(a)}}` | `{{!a}}` |

---

## Funciones de objeto

| Make | n8n |
|---|---|
| `{{keys(obj)}}` | `{{Object.keys($json.obj)}}` |
| `{{values(obj)}}` | `{{Object.values($json.obj)}}` |
| `{{get(obj; "key")}}` | `{{$json.obj.key}}` o `{{$json.obj["key"]}}` |
| `{{isEmpty(obj)}}` | `{{Object.keys($json.obj).length === 0}}` |
| `{{merge(obj1; obj2)}}` | `{{({...$json.obj1, ...$json.obj2})}}` |

---

## Codificación / Hashing

| Make | n8n (Code node) |
|---|---|
| `{{base64(str)}}` | `Buffer.from(str).toString('base64')` |
| `{{base64Decode(str)}}` | `Buffer.from(str, 'base64').toString('utf-8')` |
| `{{sha256(str)}}` | `require('crypto').createHash('sha256').update(str).digest('hex')` |
| `{{md5(str)}}` | `require('crypto').createHash('md5').update(str).digest('hex')` |
| `{{encodeURL(str)}}` | `encodeURIComponent(str)` |
| `{{decodeURL(str)}}` | `decodeURIComponent(str)` |
| `{{parseJSON(str)}}` | `JSON.parse(str)` |
| `{{stringify(obj)}}` | `JSON.stringify(obj)` |

---

## Expresiones multi-módulo

### Referenciar módulo por ID de Make

Cuando el JSON de Make dice `{{3.texto}}` (módulo ID 3), en n8n debes:

1. Identificar qué nodo tiene el ID 3 (ver campo `id` en el JSON de Make)
2. Reemplazar con el nombre del nodo n8n: `{{$node["NombreNodo"].json.texto}}`

### Acceder a datos del item actual

```
Make:  {{bundle.inputBundle.data.field}}
n8n:   {{$json.field}}
```

### Acceder al item anterior inmediato

```
Make:  (siempre {{N.field}} del módulo anterior)
n8n:   {{$json.field}}   ← si es el nodo inmediatamente anterior
       {{$node["NombreNodo"].json.field}}  ← si es más atrás
```

### Acceder a múltiples outputs de un Switch/Router

```
Make:  Cada ruta tiene su propia cadena de módulos
n8n:   Cada output del Switch conecta a su propia cadena de nodos
       Los datos siguen siendo {{$json.field}} en cada rama
```

---

## Casos especiales del JSON de Make

### El campo `mapper` vs `parameters`

```json
{
  "module": "util:SetVariable2",
  "parameters": {
    "name": "myVar"
  },
  "mapper": {
    "value": "{{2.choices[0].message.content}}"
  }
}
```

- `parameters`: configuración estática (IDs, nombres, opciones)
- `mapper`: valores dinámicos con expresiones → **aquí están las expresiones a traducir**

### Expresiones anidadas complejas

```json
// Make mapper:
{"content": "{{join(map(5.items; \"title\"); \"\\n\")}}"}

// n8n equivalente (en Set node):
{{$node["GetVariable"].json.items.map(i => i.title).join("\n")}}
```

### Filtros entre módulos (no en el JSON principal)

Los filtros de Make aparecen en el campo `filter` del módulo siguiente:
```json
{
  "id": 3,
  "filter": {
    "condition": "{{2.status}} === 'active'"
  }
}
```

En n8n, esto se convierte en un nodo **IF** entre los dos nodos con la condición equivalente.
