# Recetas

Los flujos que cierran de punta a punta, en orden de llamadas. Todos asumen que ya llamaste
a `emalia_whoami` y sabés en qué plan está la cuenta.

Al final están **los que no cierran**: los que necesitan un paso en Emalia. Esos son tan
importantes como los que sí, porque son los que no hay que prometer.

---

## 1. De una frase a una campaña lista para enviar

**"Mandale algo a los suscriptores del newsletter contando el lanzamiento."**

```
estado_cuenta                    ¿queda cupo de envíos este mes?
listar_listas                    elegir el destino, quedarse con el id
listar_plantillas                ¿hay algo reusable?
  └─ si no hay: crear_plantilla  escribir el email
       └─ con productos: antes buscar_productos (provider + id_externo)
crear_campana                    borrador, con el contenido ya cargado
enviar_prueba                    llega a tu bandeja
programar_campana                queda agendada
```

**Requiere plan de pago** en `crear_campana`, `crear_plantilla` y `programar_campana`.

El único paso humano es aprobar. Decilo así al cerrar: *"Te la dejé programada para el jueves
a las 10. Te mandé la prueba a tu bandeja: miralá antes."*

Si la persona no quiere programarla sino mandarla ya, la respuesta es que el envío se aprueba
desde Emalia. **No hay herramienta de envío.**

---

## 2. Un grupo que se actualiza solo, y una campaña para ese grupo

**"Escribile a los que abrieron en los últimos 30 días y todavía no compraron."**

```
listar_etiquetas                 ver qué existe — se usan los NOMBRES, no los ids
listar_listas                    idem
crear_segmento                   devuelve total_contactos ANTES de guardar
crear_campana                    listas_ids incluye el id del segmento
```

**Requiere plan Impulso o superior** para `crear_segmento`.

Ejemplo de condiciones para ese pedido:

```
operador: "todas"
condiciones: [
  { campo: "abrio_email", op: "en_ultimos_dias", dias: 30 },
  { campo: "etiqueta",    op: "no_tiene", etiqueta: "Cliente" }
]
```

**Mirá `total_contactos` y decilo.** Un segmento de 0 está mal armado. Uno que abarca toda la
base, también. Es el único momento en que el error se puede corregir barato.

---

## 3. "¿Por qué me va mal?"

```
salud_cuenta                     entregabilidad, rebotes, dominio, avisos
metricas_dashboard               el tamaño del negocio, para tener escala
ver_campana                      la campaña puntual que se quiere entender
```

Sin plan mínimo.

**Empezá por `avisos`**, que viene con frases accionables ya redactadas. Un puntaje de
entregabilidad solo no le dice a nadie qué hacer.

Una tasa de apertura no significa lo mismo con la lista limpia que con un 8% de rebote y el
dominio sin verificar. Por eso `salud_cuenta` va **antes** que las métricas, no después.

En `ver_campana`, los enlaces más clicados explican más que las tasas. Una apertura alta con
cero clics es un asunto que promete algo que el email no cumple.

---

## 4. "Esta automatización no le llega a nadie"

```
listar_automations               ubicar el id
ver_automation                   puede_activarse + lo_que_falta
resultados_automation            el embudo y los motivos de salida
```

`resultados_automation` **requiere plan de pago**; las otras dos no.

Las dos causas que la dejan muda, y `ver_automation` marca las dos:

- **Pasos de email sin plantilla asignada.** Al llegar ahí el envío falla y el contacto queda
  en error.
- **Disparador con la configuración incompleta.** Así no dispara para nadie, y no avisa.

Si el embudo viene en cero, es lo segundo casi siempre. Antes de decir que la automatización
"no funciona", verificá qué disparador tiene configurado.

---

## 5. Limpiar la base antes de un envío grande

```
salud_cuenta                     tasa de rebote y umbral recomendado
listar_contactos estado:"rebotado"
listar_contactos estado:"desuscrito"
crear_segmento                   los activos que sí abren, para mandarle a ese
```

Los rebotados y los desuscritos no se pueden **borrar** desde acá — ninguna herramienta
borra. Lo que sí se puede es dejar de mandarles: armá un segmento con
`{ campo: "estado", op: "=", valor: "activo" }` más lo que corresponda, y usá ese como
destino.

Mandarle a una lista sucia sube el rebote, y el rebote alto arrastra la entregabilidad de
todo lo demás. Por eso este flujo va antes del envío grande y no después.

---

## 6. Clasificar contactos y que eso dispare algo

```
listar_etiquetas                 ¿ya existe la etiqueta?
  └─ si no: crear_etiqueta
asignar_etiqueta                 hasta 1000 contactos por llamada
```

**Esto puede hacer que empiecen a salir emails.** Si hay una automatización con disparador de
etiqueta agregada usando esa etiqueta, asignarla la dispara. **Confirmalo antes**, sobre todo
con varios contactos.

Para saber si hay alguna, `listar_automations` y después `ver_automation` sobre las que
parezcan candidatas — la configuración del disparador dice qué etiqueta mira.

`quitar_etiqueta` no dispara nada y es reversible.

---

## 7. Revisión semanal de una agencia

```
agencia_salud_cartera            a quién mirar: pausas, dominio, rebote, envíos, nada programado
agencia_campanas_cartera         la agenda de la semana
agencia_reporte_cliente          los números de uno
agencia_comparar_clientes        de 2 a 6, lado a lado
agencia_crear_borrador_reporte   la nota del mes, hasta 1200 caracteres
```

Cierra hasta el borrador. **Aprobarlo y enviarlo es de la persona dueña o socia, en Emalia.**

`agencia_listar_cartera` primero si no tenés los números `EM-XXXXX` a mano.

Para operar **dentro** de la cuenta de un cliente —armarle una campaña, mirarle los
contactos— no son estas herramientas: es el parámetro `cliente` en las normales. Y ahí el
plan que se aplica es el **del cliente**.

**El mismo embudo para varios clientes:**

```
listar_automations (cliente: origen)   el flujo a llevar
agencia_copiar_flujo                   una copia en pausa en cada destino
```

Al cerrar, decí en cuáles se copió y **los pendientes de cada uno**: sin completarlos no se
puede prender. Los emails llevan el logo, los colores y los links del cliente de origen.

**El mismo email para varios clientes:**

```
listar_plantillas (cliente: origen)   la plantilla a llevar
agencia_copiar_plantilla              una copia en cada destino, sin los productos
crear_campana (cliente: destino)      borrador en cada uno, con la copia
```

La copia no se puede editar desde acá. Si el email tiene que llevar los productos de cada
cliente, no conviene copiarlo: armalo en cada uno con `buscar_productos` y `crear_plantilla`
pasando ese `cliente`.

Al cerrar, decí en cuáles se copió, en cuáles no y por qué, y que el logo, los colores y los
links son los de la original.

---

## 8. Alta de contacto — ⚠ cierra a medias

```
crear_contacto
agregar_contacto_a_lista
```

Funciona, **pero no dispara la bienvenida**. Las automatizaciones con disparador de
suscripción a lista no se activan por esta vía.

Si el objetivo era que arrancara un flujo de bienvenida, **decilo**: eso hay que hacerlo
desde Emalia. Y no afirmes el total de la lista después de agregar: no se recalcula hasta que
alguien la toque desde la aplicación.

---

## 9. Automatización desde cero — ❌ no cierra

```
crear_automation                 el esqueleto: disparador + pasos encadenados
ver_automation                   confirma que faltan las plantillas
     ↓
[ ASIGNAR LAS PLANTILLAS EN EMALIA ]   ← no hay herramienta para esto
     ↓
cambiar_estado_automation        activar
```

**El paso del medio no tiene herramienta y no se puede saltear.** Los pasos de email nacen
sin plantilla asignada, y una automatización activa con un paso sin plantilla falla en ese
paso y deja al contacto en error.

O sea: **una automatización andando no se puede prometer por conversación.** Lo que sí podés
prometer es dejarle el esqueleto armado con el disparador bien configurado, que es la parte
que más se equivoca la gente.

Decilo al crearla, no cuando pregunten por qué no salió nada.

Antes de llamar, pedí el dato que necesita el disparador: la lista, la campaña o la etiqueta.
Sin eso la automatización se crea igual pero **no dispara para nadie y no avisa**.
