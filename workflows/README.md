# workflows/

Pon aquí tus propios workflows de n8n.

Exporta desde n8n: **⋮ → Download** y guarda el JSON en esta carpeta.

## Estructura sugerida

```
workflows/
├── router.json          ← workflow principal / dispatcher
├── sub-[tipo].json      ← sub-workflows por tipo de proceso
├── util-[funcion].json  ← sub-workflows utilitarios reutilizables
└── starters/            ← plantillas base incluidas en el kit
    └── webhook-router-starter.json
```

## Naming convention sugerido

- Router/dispatcher: `[proyecto]-router.json`
- Sub-workflow: `[proyecto]-sub-[tipo].json`
- Utilitario: `[proyecto]-util-[funcion].json`

Los workflows no se suben al repo público por seguridad
(contienen IDs de credenciales específicos de tu instancia de n8n).
