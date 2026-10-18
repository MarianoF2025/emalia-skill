---
name: emalia-mcp
description: Trabajar con Emalia, la plataforma de email marketing — contactos, suscriptores, listas, etiquetas y segmentos; campañas y newsletters; plantillas de email; automatizaciones y disparadores; entregabilidad, aperturas y clics; formularios y landing pages. Usala cuando se hable de Emalia, de mandarle un email o una campaña a una audiencia, de armar una newsletter, de limpiar o segmentar una base de contactos, o de por qué un envío rinde mal. Sirve también para agencias que administran las cuentas de varios clientes.
---

# Emalia

Emalia es una plataforma de email marketing. Esta skill explica cómo operarla a través de
su conector, qué cierra de punta a punta y —sobre todo— qué **no** se puede hacer desde acá.

## 0. Antes de nada: ¿están las herramientas?

**Verificá que el conector esté conectado antes de decir una sola cosa sobre la cuenta.**
La forma de verificarlo es llamar a `emalia_whoami`.

Si esa llamada falla, o si las herramientas de Emalia directamente no aparecen entre las
que tenés disponibles, **frená ahí**. Decí esto y nada más:

> No tengo el conector de Emalia conectado, así que no puedo ver tu cuenta. Para
> conectarlo: entrá a Emalia → **integraciones** → **asistentes IA**, y seguí los pasos de
> ahí. Cuando esté, volvé a pedirme lo que necesitabas.

**No inventes.** Sin conector no sabés cuántos contactos hay, ni cómo rindió una campaña,
ni qué automatizaciones existen. Un número plausible sobre una cuenta que no consultaste es
peor que no responder: la persona lo va a repetir en una reunión. Si el conector no está,
podés explicar cómo funciona Emalia en general, pero no afirmar **nada** sobre esta cuenta
en particular.

Lo mismo vale por herramienta: si el conector está conectado pero una herramienta puntual no
aparece, no la simules. Decí qué falta.

### Los nombres de las herramientas

En esta skill se nombran sin prefijo: `listar_campanas`, `crear_campana`. Según el programa
que estés usando pueden aparecer con un prefijo del servidor —`Emalia:listar_campanas`,
`emalia__listar_campanas` o similar. Es la misma herramienta; usá la que tengas.

## 1. El orden importa

Empezá **siempre** por `emalia_whoami`. Devuelve el nombre de la cuenta y su **plan**, y el
plan decide qué se puede ofrecer. Proponerle a alguien un segmento automático cuando su plan
no lo incluye es hacerle perder el tiempo dos veces: primero ilusionándolo y después
explicándole el error.

Después, **resolvé los identificadores antes de escribir nada**. Las herramientas que crean
cosas piden ids, y los ids no se adivinan:

| Para llamar a | Antes necesitás |
|---|---|
| `crear_campana` | `plantilla_id` de `listar_plantillas` (o el que devolvió `crear_plantilla`) y `listas_ids` de `listar_listas` o `crear_segmento` |
| `agregar_contacto_a_lista` | `lista_id` de `listar_listas` |
| `ver_campana`, `programar_campana`, `enviar_prueba` | `id` / `campana_id` de `listar_campanas` |
| `ver_automation`, `resultados_automation`, `cambiar_estado_automation` | `id` de `listar_automations` |
| `ver_plantilla` | `id` de `listar_plantillas` |
| `crear_plantilla` con productos | `provider` e `id_externo` de cada producto, de `buscar_productos` |

Un id inventado no rompe de forma visible: devuelve "no encontrado en tu cuenta", y si el
error se maneja mal, termina en una respuesta que suena bien y no corresponde a nada.

**Las etiquetas y las listas de `crear_segmento` van por NOMBRE, no por id** — el nombre tal
cual lo devuelven `listar_etiquetas` y `listar_listas`. Si no existe, la herramienta falla y
te dice cuál; no lo tapes, preguntá.

Antes de proponer un envío grande, mirá `estado_cuenta`: dice cuántos envíos quedan en el
mes. Y si la pregunta es "por qué me va mal", `salud_cuenta` antes que cualquier métrica.

## 2. Qué NO se puede hacer

Esta es la sección que evita prometer de más. Leela antes de ofrecer nada.

### No existe ninguna herramienta que envíe una campaña, y no va a existir

`enviar_prueba` manda **solo a la persona conectada**. `programar_campana` deja una fecha
agendada, no dispara nada en el momento. **El botón de enviar vive únicamente en Emalia.**
Si alguien pide "mandala ahora", la respuesta es que la dejás lista y el envío lo aprueba
esa persona desde la aplicación.

### Lo demás que no se puede

| Qué | Dónde se hace |
|---|---|
| Enviar una campaña | Solo en Emalia |
| Desprogramar o cancelar una campaña ya programada | Solo en Emalia |
| Editar una plantilla que ya existe | Se puede **crear** con `crear_plantilla`; editar es del editor |
| Crear o editar formularios | Solo se listan |
| Crear o publicar landing pages | Solo se listan |
| Cambiarle el email a un contacto | No es modificable |
| **Borrar** contactos, listas, etiquetas, campañas o automatizaciones | **Ninguna herramienta borra nada** |
| Asignar una plantilla a un paso de una automatización | Solo en Emalia — ver el aviso 3 |
| Ver a qué listas o etiquetas pertenece cada contacto de una lista larga | `ver_contacto` lo trae de a uno |
| Aprobar o enviar el reporte mensual de un cliente de agencia | El borrador sí; aprobar es de la persona dueña o socia, en la aplicación |
| Armar bloques de campañas, prioridades o subcampañas | No hay herramientas |

### Y tres cosas que parecen funcionar y no

- **Los totales de `listar_listas` quedan atrasados.** No se recalculan cuando agregás
  contactos por acá. Si acabás de tocar una lista, no afirmes su número.
- **Las tasas de `ver_contacto`** se calculan sobre los últimos 200 envíos, no sobre todo el
  historial.
- **`agencia_campanas_cartera` trae como mucho 120 campañas** y puede recortar sin avisar:
  no sirve para afirmar que algo *no* existe.
- **Un `null` no es un cero.** Si `salud_cuenta` devuelve `entregabilidad.score: null`,
  significa que no se pudo calcular, no que la entregabilidad sea pésima. Lo mismo con
  `factura_mes: null` en `agencia_facturacion`.
- **`periodo` mal escrito se descarta en silencio.** Las tres herramientas de agencia que lo
  aceptan quieren `YYYY-MM`; cualquier otra cosa se ignora y usan el mes actual sin decirlo.
  Confirmá siempre de qué mes están hablando.

## 3. Los tres avisos

Estos son los que hacen quedar mal si no se dicen a tiempo.

### 1. `agregar_contacto_a_lista` NO dispara automatizaciones

Mete a la persona en la lista, sí. Pero las automatizaciones con disparador de suscripción a
lista **no se activan**. Si la idea era que arrancara la bienvenida, eso hay que hacerlo
desde Emalia. Decilo en el momento, no cuando pregunten por qué nadie recibió nada.

### 2. `crear_automation` nace sin plantillas, y no se puede cerrar por chat

Deja el esqueleto: el disparador configurado y los pasos encadenados. Lo que **no** trae es
el contenido — los pasos de email nacen sin plantilla asignada, y **no hay ninguna
herramienta para asignárselas**. Hay que abrir la automatización en Emalia, elegir qué email
manda cada paso, y recién después activarla.

O sea: **una automatización andando no se puede prometer por conversación.** Decilo al
crearla, siempre, o la persona se queda creyendo que ya quedó funcionando.

Usá `ver_automation` después de crearla: devuelve `puede_activarse` y `lo_que_falta`, que es
la lista concreta de lo que hay que arreglar.

### 3. `enviar_prueba` con `cliente` te manda la prueba a VOS

Si sos una agencia y pasás `cliente`, la **campaña** es la de ese cliente, pero la prueba
llega a **tu** bandeja, no a la del cliente. El parámetro cambia la cuenta sobre la que se
opera, nunca la persona destinataria. Está bien que sea así —la prueba la revisa la
agencia— pero decilo, porque no es lo que se espera.

### Y uno más: `asignar_etiqueta` SÍ dispara

Es la única herramienta de audiencia con efecto de envío. Si hay una automatización con
disparador de etiqueta agregada, asignar esa etiqueta **hace que empiecen a salir emails**.
Confirmalo antes, sobre todo si son varios contactos. `quitar_etiqueta` no dispara nada.

## 4. Las 42 herramientas

**Cuenta y diagnóstico** — `emalia_whoami`, `estado_cuenta`, `salud_cuenta`,
`metricas_dashboard`.

**Audiencia** — `listar_contactos`, `ver_contacto`, `crear_contacto`, `actualizar_contacto`,
`agregar_contacto_a_lista`, `asignar_etiqueta`, `quitar_etiqueta`, `listar_listas`,
`crear_lista`, `listar_etiquetas`, `crear_etiqueta`, `crear_segmento`.

**Campañas** — `listar_campanas`, `ver_campana`, `crear_campana`, `enviar_prueba`,
`programar_campana`.

**Automatizaciones** — `listar_automations`, `ver_automation`, `resultados_automation`,
`crear_automation`, `cambiar_estado_automation`.

**Plantillas** — `listar_plantillas`, `ver_plantilla`, `buscar_productos`, `crear_plantilla`.

**Captación** — `listar_formularios`, `listar_landings`.

**Agencia** — `agencia_listar_cartera`, `agencia_resumen_cartera`, `agencia_reporte_cliente`,
`agencia_comparar_clientes`, `agencia_campanas_cartera`, `agencia_crear_borrador_reporte`,
`agencia_facturacion`, `agencia_alta_cliente`, `agencia_salud_cartera`,
`agencia_copiar_plantilla`.

Los parámetros exactos de cada una están en `referencia.md`.

## 5. El plan decide qué se puede ofrecer

`emalia_whoami` devuelve el plan. Los planes son `free`, `inicio`, `impulso`, `escala` y
`a_medida`; los cuatro últimos son de pago.

| Necesita | Herramientas |
|---|---|
| Cualquier plan de pago | `crear_campana`, `crear_plantilla`, `programar_campana`, `resultados_automation`, `listar_formularios`, `listar_landings` |
| Impulso o superior | `crear_segmento` |
| Nada de plan, solo cupo | `crear_automation` y activar con `cambiar_estado_automation` — el plan gratuito llega a una automatización |
| Nada | Todo lo demás |

Cuando una herramienta rechaza por plan, el mensaje ya viene redactado y dice el plan actual
y qué se gana subiendo. **Pasalo tal cual, no lo adornes ni lo suavices.** No inventes
precios: están en emaliamail.com/precios.

Si el rechazo es por **permisos** en vez de por plan —"no tenés permiso de editar en
audiencia"— no es algo que se arregle pagando: hay que pedírselo a quien administra el
equipo en esa cuenta.

## 6. Escribir un email: los nombres de campo exactos

`crear_plantilla` recibe `bloques`, una lista en orden de arriba hacia abajo. Hay siete
tipos, y **los nombres de campo son estrictos**: si mandás `texto` donde va `html`, la
llamada se rechaza entera.

| `tipo` | Campos |
|---|---|
| `titulo` | `texto`, `nivel` (`h1` `h2` `h3`, por defecto `h2`), `alineacion` (`left` `center` `right`, por defecto `center`) |
| `parrafo` | **`html`** — no `texto` —, `alineacion` (por defecto `left`) |
| `boton` | `texto`, **`link`** — no `url` —, `alineacion` (por defecto `center`) |
| `imagen` | **`src`** — no `url` —, URL pública; `alt`; `link` opcional |
| `separador` | ninguno |
| `espacio` | **`alto`** en píxeles, de 8 a 120, por defecto 32 |
| `productos` | **`productos`**: de 1 a 8 `{ provider, id_externo }` de `buscar_productos`; `texto_boton` opcional (por defecto "Comprar") |

**Productos de la tienda.** Si el email lleva productos, buscalos primero con
`buscar_productos` (por nombre, o sin `buscar` para los más nuevos) y pasá solo su
`provider` e `id_externo`. **No escribas precios, fotos ni links de productos a mano**: la
foto, el título, el precio y el link se toman del catálogo de la cuenta al crear. Uno solo
sale como una tarjeta; de 2 a 8, en filas que en el celular quedan uno abajo del otro. Si un
id no está en el catálogo —inventado, de otra cuenta o que ya salió de la tienda—, la
plantilla entera se rechaza diciendo cuál: volvé a buscarlo, no lo reemplaces por otro sin
preguntar. "Los más nuevos" son los que entraron último a Emalia, no la fecha de la tienda, y
no hay datos de stock.

En `parrafo.html` se acepta HTML simple: `<p>`, `<strong>`, `<em>`, `<a href>`, `<ul>`/`<li>`.
`{{nombre}}` en cualquier texto se reemplaza por el nombre de cada contacto.

El **asunto es obligatorio** y no es un detalle: sin asunto la plantilla no se puede usar en
una automatización.

Lo que hace que un email se lea: un título `h1` arriba y `h2` para las secciones, párrafos
cortos, **un** botón principal y no cinco.

## 7. Agencias

Si la persona administra varias cuentas, hay dos formas de trabajar y conviene no
confundirlas.

**Las diez herramientas `agencia_*`** miran la cartera entera: quiénes son los clientes,
cómo vienen, qué hay programado. Para "¿qué clientes tengo que mirar?", `agencia_salud_cartera`
lo responde en una llamada. Dos crean cosas, y en las dos se confirma con la persona antes:
`agencia_alta_cliente` crea una cuenta de cliente nueva, y `agencia_copiar_plantilla` copia
una plantilla de un cliente a otros (sin los productos, que son del catálogo de cada uno). Empezá por `agencia_listar_cartera`, que además devuelve el
**número de cliente** (`EM-XXXXX`) que todo lo demás pide.

**El parámetro `cliente`** está en las 32 herramientas normales y sirve para operar **dentro
de** la cuenta de un cliente: `listar_contactos` con `cliente: "EM-00000"` lista los
contactos de ese cliente, no los tuyos. El número real sale de `agencia_listar_cartera`.

Tres cosas que hay que saber:

- **Solo se operan clientes con plan pago**: como mínimo Inicio, o dentro del paquete de la
  agencia. Con un cliente en el plan gratuito, cualquier herramienta con `cliente` rechaza y
  el mensaje dice por qué. Una cuenta dada de alta con `billing_mode: "cliente"` nace en el
  plan gratuito: no se puede operar hasta que el cliente pague su plan.
- **El plan que se aplica es el del cliente, no el de la agencia.** Lo que el plan del cliente
  no incluye no se habilita porque la agencia tenga uno más alto.
- **Pasá el número (`EM-XXXXX`) y no el nombre** siempre que lo tengas. Con el nombre, si hay
  ambigüedad la herramienta pide el número; y si hay parecido, es fácil operar sobre la
  cuenta equivocada.
- **Con conexión por clave de API el parámetro `cliente` no funciona**, y las herramientas
  `agencia_*` tampoco: una clave está atada a una sola cuenta. Para trabajar como agencia hay
  que conectarse con el usuario.

Confirmá sobre qué cuenta estás operando cuando la conversación cambie de cliente.
`emalia_whoami` con `cliente` lo responde en una llamada.

## 8. Cómo hablar de esto

Emalia habla en castellano y sin anglicismos. No es cosmético: la persona que usa el
producto ve una palabra en la pantalla y otra en tu respuesta, y tiene que traducir.

Decí **automatizaciones** y no *workflows*; **disparadores** y no *triggers*;
**entregabilidad** y no *deliverability*; **hacer clic** y no *clickear*; **pruebas A/B**;
**mejor horario de envío**. En la aplicación las etiquetas aparecen como **Intereses**, así
que si estás mirando la pantalla con alguien, nombralas así.

El glosario completo está en `glosario.md`.

Dos hábitos más:

- **Los números salen de las herramientas.** Si no lo consultaste, no lo digas.
- **Cuando una herramienta escribe algo, decí qué quedó hecho y qué no.** "Creé la campaña
  como borrador con el contenido cargado; falta que la revises y la envíes vos" es una
  respuesta completa. "Listo" no lo es.

## Archivos de apoyo

- `referencia.md` — las 42 herramientas con sus parámetros, valores admitidos y qué devuelve
  cada una.
- `recetas.md` — los flujos que cierran de punta a punta, en orden de llamadas.
- `glosario.md` — la terminología de Emalia, y los anglicismos que reemplaza.
