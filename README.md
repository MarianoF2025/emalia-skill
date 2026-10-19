# emalia-mcp — skill para asistentes de IA

La skill que le enseña a un asistente de IA a operar [Emalia](https://emaliamail.com) a
través de su conector MCP: armar campañas, segmentar la base, diagnosticar entregabilidad y
administrar la cartera de una agencia.

## Instalar

Descargá el `.zip` desde
[Releases](https://github.com/MarianoF2025/emalia-skill/releases) y cargalo en el programa que uses
como skill. Después conectá el conector de Emalia desde
**Emalia → integraciones → asistentes IA**.

**Sin el conector, la skill no sirve** — y está escrita para que el asistente lo diga en vez
de improvisar números. Es lo primero que verifica.

## Qué hay adentro

```
emalia-mcp/
├── SKILL.md        el núcleo — se basta solo
├── referencia.md   las 43 herramientas con sus parámetros exactos
├── recetas.md      los flujos que cierran de punta a punta
└── glosario.md     la terminología de Emalia
```

`SKILL.md` es autosuficiente: lo crítico —verificar el conector, qué **no** se puede hacer,
el orden de las llamadas y los avisos que hacen quedar mal si no se dicen— está ahí adentro.
Los otros tres enriquecen, no son requisito.

## Formato

[Agent Skills](https://code.claude.com/docs/en/skills) estándar. El frontmatter tiene
únicamente `name` y `description`, sin campos propietarios.

**Es portable a propósito.** No hay nada específico de un asistente en particular: el
conector de Emalia habla MCP y funciona igual en cualquier programa que lo soporte. Los
nombres de las herramientas se escriben sin prefijo (`listar_campanas`), porque cada programa
les pone el suyo.

## Armar el zip

```bash
zip -r emalia-mcp.zip emalia-mcp -x '.*'
```

## Mantenerla

La fuente de verdad es el código del conector, no este repo. Cuando cambien las herramientas
—una nueva, un parámetro, un cambio de plan mínimo— hay que actualizar `referencia.md` y, si
cambia lo que **se puede o no se puede** hacer, también la sección 2 de `SKILL.md`.

Esa sección es la que evita que el asistente prometa de más. Es la que más caro sale dejar
vieja.
