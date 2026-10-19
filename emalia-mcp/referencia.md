# Referencia de las 43 herramientas

Parámetros exactos, valores admitidos y qué devuelve cada una. Lo obligatorio va en
**negrita**; el resto es opcional.

**Todas las herramientas de esta página, salvo las once `agencia_*`, aceptan además un
parámetro `cliente`** (número `EM-XXXXX` o nombre) para operar sobre la cuenta de un cliente
de la cartera. Ver la sección 7 del SKILL.md.

Cuando algo falla, el mensaje de error viene redactado y es para leerlo, no para
reinterpretarlo.

---

## Cuenta y diagnóstico

### `emalia_whoami`
Sin parámetros. **Llamala primero.**

Devuelve `cuenta` (nombre), `plan` (el slug: `free`, `inicio`, `impulso`, `escala`,
`a_medida`), `slug`, `tenant_id` y `operando_como` — `null` en tu propia cuenta, o
`{agencia, cliente, numero_cliente, tu_rol}` si estás operando un cliente.

`tenant_id` es un identificador interno: ninguna herramienta lo pide y a la persona no le
dice nada. **No lo muestres en la respuesta.** Para nombrar una cuenta usá `cuenta`, y para
identificar a un cliente de una cartera, `numero_cliente`.

### `estado_cuenta`
Sin parámetros. Cuánto margen queda.

Devuelve `plan {nombre, slug, es_de_pago}`; `envios_del_mes {usado, tope, disponible,
agotado, usado_pct}`; cupos con esa misma forma para automatizaciones activas, landings
publicadas, plantillas, campos personalizados, integraciones y usuarios; `contactos` (sin
tope — los contactos son ilimitados en todos los planes, lo que se cobra es el envío);
`incluye` y `niveles`.

Consultala **antes** de proponer algo que consuma cupo. Si `usado_pct` está alto, avisalo
antes de que choque.

### `salud_cuenta`
Sin parámetros. Si la cuenta está en condiciones de mandar.

Devuelve `entregabilidad {score, etiqueta, factores}`; `dominio {dominio, estado, verificado,
alias_de_respaldo}`; `rebotes {ventana_dias, enviados, rebotados, tasa_rebote_pct,
umbral_recomendado_pct}`; `lista {contactos_activos, desuscritos, rebotados}`;
`arranque_gradual`; `envios_pausados`; y **`avisos`**, un arreglo de frases ya redactadas.

**Empezá por `avisos`.** Un puntaje solo no le dice a nadie qué hacer. Y `score: null`
significa "no se pudo calcular", no "cero".

### `metricas_dashboard`
Sin parámetros. La foto general en una llamada.

Devuelve números planos: `contactos_total`, `campanas_total`, `campanas_enviadas`,
`envios_mes`, `aperturas_mes`, `clicks_mes`, `rebotes_mes`, `automations_activas`,
`automations_total`, `total_listas`. Los que terminan en `_mes` son del mes calendario en
curso; el resto es histórico.

---

## Audiencia

### `listar_contactos`
- `buscar` — texto libre, busca en el email y en el nombre
- `estado` — `activo` · `pendiente` · `desuscrito` · `rebotado` · `spam`
- `pagina` — entero ≥ 1, por defecto 1
- `por_pagina` — 1 a 100, por defecto 25

Devuelve `total` (el conteo real, sirve aunque mires solo la primera página), `pagina`,
`por_pagina` y `contactos`. No trae a qué listas o etiquetas pertenece cada uno: eso es
`ver_contacto`, de a uno.

### `ver_contacto`
- `id` **o** `email` — hace falta uno de los dos

Devuelve `contacto`, `actividad` (con `tasa_apertura_pct` y `tasa_click_pct` como números),
`listas`, `etiquetas` y `automatizaciones`.

Las tasas se calculan sobre los **últimos 200 envíos**, no sobre todo el historial.

### `crear_contacto`
- **`email`**
- `nombre`, `telefono`
- `estado` — mismo enum que arriba, por defecto `activo`

El email se pasa a minúsculas y no puede repetirse: si ya existe, falla en vez de pisar los
datos. **Crear el contacto no lo mete en ninguna lista** — encadená con
`agregar_contacto_a_lista`. Queda marcado con origen `mcp`.

### `actualizar_contacto`
- `id` **o** `email_buscar` — hace falta uno de los dos
- `nombre`, `telefono`, `estado`

Solo toca lo que le pases. **El email no se puede cambiar.** Poner `estado: "desuscrito"`
equivale a dar de baja.

### `agregar_contacto_a_lista`
- **`lista_id`**
- `contacto_id` **o** `email` — hace falta uno de los dos

Es idempotente. **No dispara automatizaciones**, y los totales de `listar_listas` no se
actualizan hasta que alguien toque la lista desde la aplicación.

### `asignar_etiqueta`
- **`etiqueta`** — el nombre, tal como aparece en `listar_etiquetas`
- `etiqueta_id` — alternativa al nombre
- `contacto_ids` y/o `emails` — al menos uno de los dos, máximo **1000 contactos** por
  llamada

**Dispara** las automatizaciones con disparador de etiqueta agregada. Confirmalo antes.

Si la etiqueta no existe hay que crearla primero: esta herramienta no la crea sola, a
propósito, para que un error de tipeo no llene la cuenta de etiquetas. Es idempotente: a
quien ya la tenía no se le vuelve a asignar ni se le dispara nada.

Devuelve `etiqueta {id, nombre}`, `asignados`, `ya_la_tenian` y `no_encontrados`.

### `quitar_etiqueta`
Mismos parámetros que `asignar_etiqueta`. **No dispara nada**, es reversible, y no borra la
etiqueta de la cuenta — solo la desvincula de esos contactos.

Devuelve `etiqueta`, `quitados` y `no_encontrados`. `quitados` cuenta los que realmente la
tenían, no los que pediste.

### `listar_listas`
Sin parámetros. Devuelve `listas` con `id`, `nombre`, `descripcion`, `color`, `tipo`,
`total_contactos`, `contactos_activos` y `created_at`.

Los `id` son los que piden `crear_campana` y `agregar_contacto_a_lista`. **Los totales pueden
estar atrasados** — ver `agregar_contacto_a_lista`.

### `crear_lista`
- **`nombre`**
- `descripcion`, `color` (hexadecimal)

Crea listas **manuales**, donde los contactos se cargan a mano. Para un grupo que se
recalcule solo según condiciones, la herramienta es `crear_segmento`.

### `listar_etiquetas`
Sin parámetros. Devuelve `etiquetas` con `id`, `nombre`, `descripcion`, `color` y `activo`.

Los **nombres** que devuelve son los que piden `asignar_etiqueta`, `quitar_etiqueta` y las
condiciones de `crear_segmento`. En la aplicación estas se llaman **Intereses**.

### `crear_etiqueta`
- **`nombre`**
- `descripcion`, `color` (hexadecimal)

Nace vacía. Para ponérsela a alguien, `asignar_etiqueta`.

### `crear_segmento`
**Requiere plan Impulso o superior.**

- **`nombre`**
- `operador` — `todas` (por defecto, más restrictivo) o `cualquiera`
- **`condiciones`** — al menos una
- `descripcion`

Cada condición es `{campo, op, ...}` y **cada campo admite solo sus operadores**:

| `campo` | `op` | Y además |
|---|---|---|
| `estado` | `=` `!=` | `valor`: `activo` `pendiente` `desuscrito` `rebotado` `spam` |
| `origen` | `=` `!=` | `valor`: `formulario` `manual` `importado` `api` `onboarding` `mcp` |
| `etiqueta` | `tiene` `no_tiene` | `etiqueta`: el **nombre** |
| `lista` | `en` `no_en` | `lista`: el **nombre** |
| `abrio_email` | `alguna_vez` `nunca` `en_ultimos_dias` | `dias` (1 a 3650) solo con `en_ultimos_dias` |
| `hizo_clic` | `alguna_vez` `nunca` `en_ultimos_dias` | igual |

Cuenta la gente **antes** de guardar y lo devuelve en `total_contactos`. **Miralo y decilo**:
un segmento de 0 está mal armado, y uno que abarca toda la base probablemente también.

---

## Campañas

### `listar_campanas`
- `estado` — `borrador` · `programada` · `enviando` · `enviada`
- `pagina` (por defecto 1), `por_pagina` (1 a 50, por defecto 20)

Devuelve `total`, `pagina`, `por_pagina` y `campanas` con los contadores **en crudo, sin
tasas**. Para las tasas, `ver_campana`.

### `ver_campana`
- **`id`**

Devuelve `remitente`, `programada_at`, `enviada_at`, `totales` (destinatarios, enviados,
entregados, abiertos, clics, rebotes, desuscripciones, spam), `tasas` **como números** —
apertura, clics, rebote, desuscripción—, `top_links` (hasta 20) y `listas_destino`.

Si todavía no se envió, los totales vienen en cero y no hay enlaces: es normal, no un error.
**Los enlaces más clicados explican más que las tasas solas.**

### `crear_campana`
**Requiere plan de pago.**

- **`nombre`** — interno, lo ve el equipo
- **`asunto`** — esto sí lo ve quien lo recibe
- **`plantilla_id`**
- **`listas_ids`** — al menos una
- `preview_text`

Deja un **borrador**, nunca envía. El contenido de la plantilla se **copia adentro** al
crearla, así que la campaña queda enviable sin ningún paso intermedio. Esa copia se congela:
si después alguien edita la plantilla, esta campaña no cambia.

Si la plantilla elegida no tiene contenido, falla y te dice cuál — en vez de dejar una
campaña que el envío iba a rechazar.

Devuelve `id`, `estado: "borrador"`, `listas` (los nombres) y `total_contactos_estimado`.

### `enviar_prueba`
- **`campana_id`**

Manda la campaña **al email de quien está conectado, y a nadie más**. La lista destino no
recibe nada.

No deja rastro: no cuenta como envío del mes, no aparece en las métricas y no lleva
seguimiento de aperturas. **No se puede elegir el destinatario**, a propósito.

Con `cliente`, la campaña es la del cliente pero **la prueba te llega a vos**.

Con conexión por clave de API no funciona: no hay un usuario del que sacar el email.

### `programar_campana`
**Requiere plan de pago.**

- **`campana_id`**
- **`fecha`** — ISO 8601 **con zona horaria**, por ejemplo `2026-08-15T14:30:00-03:00`, y
  tiene que ser futura

Deja la campaña agendada. Verifica antes que tenga asunto, contenido y listas destino: si
falta algo falla acá y no a la hora del envío, cuando no hay nadie mirando.

**Para desprogramar hay que entrar a Emalia.**

---

## Automatizaciones

### `listar_automations`
Sin parámetros. Devuelve `automations` con `id`, `nombre`, `descripcion`, `estado` (`activa`
o `pausada`), `prioridad`, `es_sistema` y los contadores de iniciados, completados y en
proceso.

Incluye las automatizaciones internas del sistema (`es_sistema: true`): **esas no conviene
tocarlas.**

### `ver_automation`
- **`id`**

La estructura y —lo más útil— **qué le falta para poder activarse**.

Devuelve `disparadores` con su configuración, `pasos`, `condiciones_de_salida`,
**`puede_activarse`** y **`lo_que_falta`**.

Marca los dos problemas que dejan muda a una automatización: pasos de email **sin plantilla
asignada** y disparadores con la **configuración incompleta**. Si `puede_activarse` es
`false`, decí exactamente qué hay que arreglar antes de ofrecer activarla.

### `resultados_automation`
**Requiere plan de pago.**

- **`automation_id`**

Devuelve `pasos`, `funnel {total_contactos, por_estado, completados, tasa_completado_pct}` y
**`motivos_salida`**.

Los motivos de salida diagnostican más que la tasa de completado. Si nunca entró nadie, el
embudo viene en cero: eso casi siempre significa que el disparador está mal configurado, no
que la automatización sea mala.

### `crear_automation`
**No requiere plan de pago** — consume cupo de automatizaciones, y el plan gratuito llega a
una.

- **`nombre`**
- **`trigger_tipo`** — uno de los once
- `trigger_config` — opcional en el esquema, **obligatorio en la práctica**
- `prioridad` — 1 alta, 2 normal (por defecto), 3 baja

Los once disparadores y qué configuración necesita cada uno:

| `trigger_tipo` | `trigger_config` |
|---|---|
| `suscripcion_lista` | `{ lista_id }` |
| `formulario_enviado` | `{ formulario_id }` |
| `campana_enviada` | `{ campana_id }` |
| `campana_no_abierta` | `{ campana_id }` |
| `tag_agregado` | `{ tag }` o `{ tag_id }` |
| `campo_actualizado` | `{ campo }` |
| `fecha` | `{ campo }` |
| `cumpleanos` | nada |
| `contacto_creado` | nada |
| `inactividad` | nada |
| `evento_custom` | nada |

Para que dispare a propósito en todos los casos, `{ todas: true }`. **Sin la configuración
correcta no dispara para nadie y no avisa.** Pedile el dato a la persona; no lo adivines.

Tres disparadores traen una secuencia ya pensada: `suscripcion_lista` (bienvenida + espera de
3 días + seguimiento), `campana_no_abierta` (reintento + espera de 2 días + último intento) y
`cumpleanos` (un saludo). El resto arranca con un solo paso de email.

Nace **pausada y sin plantillas en los pasos**. Ver el aviso 2 del SKILL.md.

### `cambiar_estado_automation`
- **`id`**
- **`accion`** — `activar` o `pausar`

**Activar tiene efecto inmediato:** desde ese momento, quien cumpla el disparador empieza a
recibir los emails del flujo. Confirmalo antes. Antes de activar una recién creada, revisá
con `ver_automation` que sus pasos tengan plantilla.

Activar consume cupo de automatizaciones y puede fallar por tope. Pausar es seguro,
reversible y nunca falla por plan.

---

## Plantillas

### `listar_plantillas`
- `buscar` — busca en el nombre y la descripción
- `excluir_sistema` — booleano, **por defecto `true`**: esconde las plantillas internas de
  las automatizaciones del sistema

Devuelve `plantillas` **sin el HTML**, de la editada más recientemente a la más vieja.

### `ver_plantilla`
- **`id`**

Devuelve la plantilla con `html_content` **entero, sin recortar**. Puede ser largo: no la
llames en lote sobre muchas plantillas. Para elegir una alcanza con `listar_plantillas`.

### `buscar_productos`
- `buscar` — palabras del nombre del producto; tienen que estar **todas** en el título. Sin
  esto, los más nuevos
- `cantidad` — de 1 a 25, por defecto 10

Busca en el catálogo de productos de la cuenta: lo que trajo la tienda conectada y lo que se
cargó a mano. Solo lo que sigue publicado en la tienda. "Los más nuevos" son los que entraron
último a Emalia, no la fecha de publicación en la tienda. No hay datos de stock.

Devuelve `productos` con `provider`, `id_externo`, `titulo`, `precio`, `moneda`, `imagen` y
`url`, y `hay_mas` si quedaron más resultados que `cantidad`. Para ponerlos en un email,
pasá `provider` e `id_externo` a un bloque `productos` de `crear_plantilla`.

Un texto que no deja ninguna palabra para buscar no trae todo el catálogo: no trae nada.

### `crear_plantilla`
**Requiere plan de pago.**

- **`nombre`** — interno
- **`asunto`** — obligatorio; sin asunto la plantilla no sirve en automatizaciones
- **`bloques`** — al menos uno
- `preheader`, `descripcion`

Los siete tipos de bloque y sus campos **exactos** están en la sección 6 del SKILL.md.
Resumen: `titulo` lleva `texto`/`nivel`/`alineacion`; `parrafo` lleva **`html`**; `boton`
lleva `texto` y **`link`**; `imagen` lleva **`src`**, `alt` y `link` opcional; `separador` no
lleva nada; `espacio` lleva **`alto`** (8 a 120 px); `productos` lleva **`productos`**, de 1
a 8 `{ provider, id_externo }` de `buscar_productos`, y `texto_boton` opcional.

Los datos de cada producto —foto, título, precio, link— se leen del catálogo de la cuenta al
crear. Si un id no está ahí, **la plantilla entera se rechaza** y el mensaje dice cuál.

No se puede **editar** una plantilla existente: eso es el editor de Emalia.

---

## Captación

### `listar_formularios`
**Requiere plan de pago.** Sin parámetros.

Devuelve `formularios` con `lista_vinculada` (el nombre de la lista donde caen los contactos,
o `null`), `total_vistas`, `total_suscripciones`, `tasa_conversion_pct` y `activo`.

Muchas vistas y pocas suscripciones es un problema de formulario, no de tráfico.

### `listar_landings`
**Requiere plan de pago.** Sin parámetros.

Devuelve `landings` con `slug`, `estado`, `visitas`, `conversiones`, `tasa_conversion_pct` y
`publicada_at`.

Va de la mano con `listar_formularios`: la landing trae la visita, el formulario la convierte.

No se pueden crear ni publicar landings desde acá.

---

## Agencia

Estas once **no** llevan `cliente`. Llevan `agencia`, opcional, y solo hace falta si
pertenecés a más de una.

### `agencia_listar_cartera`
Sin parámetros. **Empezá por acá:** devuelve el `numero_cliente` (`EM-XXXXX`) que piden todas
las demás y el parámetro `cliente`.

Devuelve `agencia`, `tu_rol` y `clientes` con `cliente`, `numero_cliente`, `plan`, `estado`,
`contactos_activos`, `envios_este_mes`, `limite_envios_mes` y `uso_envios_pct`.

Si sos colaborador ves solo tus clientes asignados; si sos dueño o socio, todos.

Con `uso_envios_pct` se ve de una llamada a quién le queda margen. Para el resto del consumo
de una cuenta (automatizaciones, landings, usuarios), `estado_cuenta` con `cliente`.

### `agencia_salud_cartera`
Sin parámetros. **"¿Qué clientes necesitan atención?"** en una llamada.

Por cada cliente revisa lo mismo que `salud_cuenta` —envíos pausados, dominio propio sin
verificar o sin usar, rebote por encima del 2%, lista gastada— y además si usó el 80% o más de
los envíos del mes y si no tiene ninguna campaña programada.

Devuelve `clientes`, `necesitan_atencion` y `cartera`, **ordenada de más avisos a menos**, con
`cliente`, `numero_cliente`, `plan`, `se_puede_operar`, `avisos` (frases ya redactadas),
`entregabilidad`, `dominio_verificado`, `tasa_rebote_pct`, `envios_pausados`, `uso_envios_pct`
y `campanas_programadas`.

Empezá por los avisos. Un cliente sin plan pago viene con `se_puede_operar: false` y sin más
detalle: en modo agencia no se opera. `se_puede_operar: null` es que esa cuenta no se pudo
revisar ahora, **no** que esté sana.

### `agencia_resumen_cartera`
Sin parámetros. El pulso de la cartera para la revisión semanal.

Devuelve `envios_del_mes_total` y **`alertas`**: los clientes sin campañas programadas. Esa
lista es la de a quién llamar.

### `agencia_reporte_cliente`
- **`cliente`** — número o nombre
- `periodo` — `YYYY-MM`; **cualquier otro formato se descarta en silencio** y usa el mes
  actual

Devuelve `cliente`, `numero_cliente` y `metricas`. El contenido exacto de `metricas` lo
define la base: leelo de lo que llega, no des por sentado que trae un campo puntual.

### `agencia_comparar_clientes`
- **`clientes`** — entre **2 y 6**
- `periodo` — mismo comportamiento silencioso

Devuelve `periodo` y `comparacion` con `envios`, `tasa_apertura_pct`, `tasa_clic_pct` y
`contactos_nuevos`, todos como números.

Si alguno de los clientes no se puede identificar, **falla entera** y no devuelve nada.

### `agencia_campanas_cartera`
Sin parámetros. La agenda: `proximas` y `enviadas_recientes`, en una ventana de ±30 días.

**Trae como mucho 120 campañas y puede recortar sin avisar.** No sirve para afirmar que algo
no existe.

### `agencia_crear_borrador_reporte`
- **`cliente`**
- **`nota`** — máximo **1200** caracteres
- `periodo`

Deja escrita la nota del mes como **borrador**. Nunca lo envía. Aprobarlo y mandarlo es de la
persona dueña o socia, desde Emalia.

Si el reporte de ese período ya fue aprobado o enviado, falla en vez de pisarlo.

### `agencia_facturacion`
**Requiere ser dueño, socio, o tener el permiso de facturación.** Sin parámetros.

Devuelve `factura_mes`, `comisiones_pendientes_usd` y `comisiones_historico_usd`.

`factura_mes: null` significa que el cálculo falló, no que no haya nada que facturar.

### `agencia_alta_cliente`
**Solo el dueño o un socio de la agencia.** Es la única herramienta de agencia que crea algo
y no se deshace desde el chat: **confirmá los datos con la persona antes de llamarla.**

- **`nombre`** — como va a figurar la cuenta
- **`email_contacto`** — el email del dueño de la cuenta, sin acentos ni ñ
- **`billing_mode`** — `cliente` o `paquete`:
  - `cliente`: el cliente paga su plan con el descuento de agencia. La cuenta nace en el plan
    gratuito y el plan lo elige el cliente al pagar. **Hasta que pague, la agencia no la
    puede operar.**
  - `paquete`: la cuenta usa envíos del paquete de la agencia (la agencia tiene que tener uno
    activo), con el cupo de `cupo`.
- `cupo` — solo con `paquete`: los envíos por mes que le tocan
- `precio_reventa` — lo que la agencia le cobra al cliente; solo informativo
- `plan_slug` — no hace falta

Que **la pague la agencia** con su tarjeta no se puede desde el chat: es desde la consola de
la agencia. El acceso le llega por mail al cliente; la herramienta no devuelve ningún link.

Devuelve `numero_cliente`, que es lo que piden las demás. Si el email ya tiene una cuenta en
Emalia —también escrito de otra forma, como `nombre+algo@`— falla y no crea nada.

### `agencia_copiar_plantilla`
**Crea una plantilla en cada destino: confirmá con la persona a qué clientes va.**

- **`plantilla_id`** — de `listar_plantillas` pasando el cliente de origen
- **`desde`** — el cliente de origen (mejor el número `EM-XXXXX`)
- **`hacia`** — los clientes destino, de 1 a 20
- `nombre` — el nombre de la copia; por defecto, el de la original. Si la original nombra al
  cliente de origen, cambialo

Se copian los bloques, el estilo, el asunto y el preheader **tal cual**: el logo, los colores y
los links son los del cliente de origen. Decile a la persona que la revise en cada cliente.

**Los productos no se copian**: son del catálogo del cliente de origen. Se sacan y viene
`productos_sacados`; en cada destino se ponen los suyos con `buscar_productos` y
`crear_plantilla`. Tampoco se copia la descripción.

Origen y destinos tienen que ser de la cartera y tener plan pago. Un destino que no puede —sin
plan pago, fuera de la cartera, con el tope de plantillas lleno— no frena a los demás: viene
en `copias` con `copiada: false` y su `motivo`. No se copian las plantillas internas del
sistema ni las de HTML importado sin bloques.

Devuelve `origen`, `plantilla`, `productos_sacados` y `copias`, una por destino, con
`copiada` y `plantilla_id` o `motivo`.

### `agencia_copiar_flujo`
**Crea en cada destino un flujo, sus emails y, si faltan, listas y etiquetas: confirmá con la
persona a qué clientes va.**

- **`automation_id`** — de `listar_automations` pasando el cliente de origen
- **`desde`** — el cliente de origen (mejor el número `EM-XXXXX`)
- **`hacia`** — los clientes destino, de 1 a 20

El flujo llega **en pausa**, con sus pasos conectados igual y el mismo nombre (o "(copia)" si
ya hay uno así). Lo que usa y el destino no tiene se resuelve así:

| El flujo usa… | En el destino |
|---|---|
| Emails | Una plantilla nueva por email, **sin los bloques de productos** |
| Listas fijas y etiquetas | La del mismo nombre; si no existe, se crea vacía |
| Listas y etiquetas automáticas | Quedan sin elegir |
| Campañas, formularios, landings, otro flujo | Quedan sin elegir |
| Campos personalizados | El del mismo nombre; si no está, se avisa |

**No se copian, a propósito:** la dirección de un webhook, a quién avisa "notificar al dueño"
y el remitente propio de un paso. Mandarían datos de los contactos de un cliente a algo del
otro.

Origen y destinos tienen que ser de la cartera y tener plan pago. Si a un destino no le alcanza
el tope de plantillas, ese no se copia y no queda nada a medias; los demás sí se copian.

Devuelve `origen` y `copias`: una por destino, con `copiado`, `automation_id`, `flujo`,
**`pendientes`** (frases ya redactadas: lo que hay que completar allá) y `creadas`
(plantillas, listas, etiquetas), o `motivo`.
