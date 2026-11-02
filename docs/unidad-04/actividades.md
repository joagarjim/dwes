# Unidad 4. Actividades

## Bloque 1. Peticiones y respuestas HTTP

Este bloque no tiene ejercicios independientes. Sus conceptos se practicarán en los ejercicios de formularios.

## Bloque 2. Formularios y validación

### Indicaciones comunes

Resuelve los ejercicios en carpetas independientes `ej01`, `ej02`, `ej03` y `ej04`. Utiliza PHP y el entorno Docker de clase. Cada formulario se envía por POST al mismo archivo que lo muestra.

* En la primera visita, muestra el formulario sin errores.
* Recoge los campos comprobando la forma esperada: texto o array.
* Valida en PHP todas las reglas, aunque también utilices atributos HTML.
* Si hay errores, muestra mensajes junto a los campos y conserva los valores introducidos, incluidas las selecciones válidas.
* Escapa los datos al insertarlos en HTML. Puedes definir las funciones `e` y `campoTexto` de los apuntes.
* Procesa los datos solo si no hay errores. En los ejercicios 1–3 muestra el resumen en la misma página y conserva el formulario. En el ejercicio 4 redirige.
* No utilices sesiones, cookies, archivos de datos ni base de datos. No se enviarán correos ni se realizarán reservas reales.
* No es necesario añadir CSS, JavaScript ni clases. Se valoran la claridad y el funcionamiento.

Para comprobar la validación PHP, añade temporalmente `novalidate` al formulario; retíralo al terminar. Prueba también envíos manipulados con las herramientas del navegador. En los filtros numéricos compara el fallo mediante `=== false`.

### Ejercicio 1. Inscripción a un taller

**Archivo:** `ej01/taller.php`.

Prepara un formulario para solicitar una inscripción a un taller de informática.

| Campo (`name`) | Control | Reglas en PHP |
| --- | --- | --- |
| Nombre (`nombre`) | Texto | Obligatorio; entre 3 y 60 caracteres después de quitar espacios de los extremos. |
| Correo (`email`) | Email | Obligatorio; formato de correo válido. |
| Edad (`edad`) | Número | Obligatoria; entero entre 16 y 99, ambos incluidos. |

Los campos se muestran inicialmente vacíos. Si el envío es válido, muestra «Datos de inscripción válidos» y un resumen con nombre, correo y edad. Aclara que no se ha guardado ni enviado la inscripción.

**Comprueba:** primera visita; nombre vacío o de dos letras; correo incorrecto; edades 15, 16, 99 y 100; corrección de un campo sin perder los demás. Un nombre con comillas debe mostrarse como texto.

**Orientación:** utiliza `FILTER_VALIDATE_EMAIL` y `FILTER_VALIDATE_INT` con límites.

### Ejercicio 2. Solicitud de reparación

**Archivo:** `ej02/reparacion.php`.

Crea un formulario de solicitud de reparación de equipos.

| Campo (`name`) | Control | Reglas en PHP |
| --- | --- | --- |
| Equipo (`equipo`) | Desplegable | Obligatorio: `portatil`, `sobremesa` o `impresora`. |
| Avería (`averia`) | Textarea | Obligatoria; entre 15 y 500 caracteres después de quitar espacios de los extremos. |
| Urgencia (`urgencia`) | Radio | Obligatoria: `normal` o `urgente`. |

El desplegable comienza con «Selecciona un equipo» de valor vacío. No hay radio marcado inicialmente y el textarea comienza vacío.

Si los datos son válidos, muestra «Solicitud comprobada», el equipo con su etiqueta legible, la descripción y la urgencia. No se guarda ni se envía la solicitud. En caso de error, conserva el contenido del textarea y las selecciones válidas. No aceptes opciones inventadas aunque procedan de un select o un radio.

**Comprueba:** falta de equipo y urgencia; avería de menos de 15 caracteres; opción de equipo inventada; descripción con `<` y comillas; corrección conservando las selecciones.

### Ejercicio 3. Configura tus servicios

**Archivo:** `ej03/servicios.php`.

Prepara un formulario para elegir servicios informáticos.

| Servicio | Valor enviado | Precio |
| --- | --- | ---: |
| Limpieza interna | `limpieza` | 20 € |
| Instalación del sistema | `sistema` | 35 € |
| Copia de seguridad | `copia` | 25 € |
| Revisión de seguridad | `seguridad` | 15 € |

Utiliza cuatro checkboxes con `name="servicios[]"`. Añade una casilla independiente con `name="acepta"`, valor `1` y texto «He leído las condiciones de la solicitud».

**Reglas:**

* Es obligatorio elegir al menos un servicio de la lista.
* El conjunto recibido debe ser un array y todos sus elementos deben ser cadenas con valores permitidos. Rechaza el envío completo si aparece un elemento inválido.
* Elimina valores duplicados para no cobrar dos veces el mismo servicio.
* Es obligatorio aceptar las condiciones; solo se admite el valor exacto `1`.
* Los precios se definen en PHP: no se reciben del formulario ni de campos ocultos.

Inicialmente no habrá ninguna casilla marcada. Si todo es válido, muestra los servicios elegidos con su precio y el total. No se realiza una contratación real. Si hay errores, conserva las selecciones válidas y la aceptación válida.

**Comprueba:** sin servicios; sin aceptación; varios servicios; todos los servicios (95 €); una opción inventada; un valor repetido (se cobra una vez); `servicios` enviado como texto en lugar de array.

### Ejercicio 4. Reserva de una actividad

**Archivos:** `ej04/reserva.php` y `ej04/resultado.php`.

Crea un formulario de solicitud de reserva para actividades de un centro cultural.

| Campo (`name`) | Control | Reglas en PHP |
| --- | --- | --- |
| Nombre (`nombre`) | Texto | Obligatorio; entre 3 y 60 caracteres tras quitar espacios de los extremos. |
| Correo (`email`) | Email | Obligatorio; formato válido. |
| Actividad (`actividad`) | Desplegable | Obligatoria: `robotica`, `fotografia` o `astronomia`. |
| Plazas (`plazas`) | Número | Obligatorio; entero entre 1 y 5. |

El nombre y el correo comienzan vacíos; la actividad con una opción vacía «Selecciona una actividad»; las plazas con valor inicial `1`.

Si hay errores, devuelve el formulario con mensajes y valores. Si todo es válido, responde con una **redirección 303** a `resultado.php` y detén el script con `exit`. Sitúa esa lógica antes de generar HTML.

La página de resultado mostrará «Fin de la demostración de reserva», un texto que aclare que no se ha guardado ninguna reserva y un enlace para volver al formulario. No debe intentar mostrar las variables del otro archivo ni recibir los datos personales por la URL. Indica que visitar esa página directamente no demuestra un envío previo.

**Comprueba:** campos vacíos; correo incorrecto; actividad inventada; plazas 0, 1, 5 y 6. Con un envío correcto, observa en Red / Network el POST con respuesta 303 y el GET a `resultado.php`. Al actualizar la página final no debe repetirse el POST.

**Explica en un comentario:** por qué los datos pueden volver a aparecer en el formulario con errores y por qué no están disponibles automáticamente en `resultado.php`.

## Entrega y comprobación

Entrega las cuatro carpetas con el código comentado de forma breve. Comprueba cada regla y evita avisos de claves inexistentes o errores al recibir arrays inesperados. La forma concreta de entrega y el plazo se indicarán en Moodle.

**Relación con la programación:** RA3.e y RA3.f (formularios y recuperación de información); RA4.f (uso del entorno para prueba y depuración).

## Autoría

Material elaborado por Joaquín García para DWES de 2.º DAW, IES Portada Alta. Actividades propias vinculadas a los apuntes de la unidad. CC BY-NC-SA.
