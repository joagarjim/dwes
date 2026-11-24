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

## Bloque 3. Subida de archivos

### Ejercicio 5. Entrega de una memoria en PDF

**Archivo:** `ej05/entrega.php`.

Crea un formulario que permita entregar una memoria de prácticas con estos campos:

| Campo (`name`) | Control | Reglas en PHP |
| --- | --- | --- |
| Título de la práctica (`titulo`) | Texto | Obligatorio; entre 5 y 80 caracteres después de quitar espacios de los extremos. |
| Memoria (`memoria`) | Archivo | Un único PDF, no vacío y de hasta **3 MiB** (3 × 1024 × 1024 bytes). |

El formulario utilizará POST y `multipart/form-data`. En la primera visita se mostrará vacío y sin errores. Puedes orientar el selector con `accept="application/pdf"`, pero deberás comprobar el archivo en PHP.

**La aplicación debe:**

1. Recoger el título mediante `$_POST` y el archivo mediante `$_FILES`, comprobando la forma esperada de cada dato.
2. Comprobar los errores de recepción, el tamaño y el tipo del contenido con **Fileinfo**. El tipo admitido será `application/pdf`; no basta con comprobar la extensión o el tipo enviado por el navegador.
3. **Guardar el archivo solo si todos los datos son válidos**, incluido el título. Utiliza `move_uploaded_file`, un nombre generado por la aplicación y la extensión `.pdf`. No utilices el nombre original como destino.
4. Guardarlo en `/var/www/almacen-ud04`, preparada fuera de la raíz pública y con los permisos indicados en los apuntes.
5. Si hay errores, mostrarlos junto al campo correspondiente y conservar el título. El archivo deberá seleccionarse otra vez.
6. Si todo es correcto, mostrar **«Memoria recibida»**, el título y el nombre con el que se ha guardado, en la misma página. Escapa los datos al mostrarlos en HTML.

No se utilizarán sesiones ni base de datos. En este ejercicio sí se conserva el archivo recibido. No es necesario mostrar ni descargar el PDF desde la página.

**Preparación:** habilita Fileinfo y utiliza `upload_max_filesize = 4M` y `post_max_size = 5M` en el PHP del contenedor. De este modo podrás probar con un PDF de más de 3 MiB y menos de 4 MiB el rechazo realizado por tu propio código. Para contar los caracteres del título puedes utilizar `mb_strlen`, como en el bloque anterior.

**Comprueba:**

| Prueba | Resultado esperado |
| --- | --- |
| Primera visita. | Formulario vacío, sin mensajes de error. |
| Título de menos de 5 caracteres y PDF válido. | Error de título; el archivo no se guarda. |
| Título válido y ningún archivo. | Error de memoria; el título se conserva. |
| Título y PDF válidos. | Se guarda y se muestra el nombre generado. |
| Archivo de texto u otro formato renombrado a `.pdf`. | Se rechaza por tipo de contenido. |
| Archivo vacío o PDF de más de 3 MiB. | Se rechaza por tamaño si llega a recibirse; si PHP impide su recepción, se informa del error de subida. |
| Dos PDF con el mismo nombre original. | Se guardan con nombres nuevos diferentes. |

Utiliza el terminal del contenedor para comprobar qué archivos se han guardado. Verifica especialmente que **un título incorrecto no provoca un guardado parcial**. Puedes desactivar temporalmente las comprobaciones HTML para probar las de PHP.

## Bloque 4. Cookies

### Ejercicio 6. Una web que recuerda cómo prefieres leer

**Archivo:** `ej06/lectura.php`.

Una web de noticias quiere que sus visitantes puedan elegir un tamaño de texto cómodo para leer sus artículos. Crea una página que **recuerde esa elección durante 30 días**, aunque el visitante cierre la pestaña y vuelva más tarde.

#### 1. Prepara la página

La página debe mostrar:

* Un título y un pequeño artículo con, al menos, tres párrafos. Puedes inventar una noticia relacionada con la informática.
* El tamaño de lectura utilizado actualmente: **Normal**, **Grande** o **Muy grande**.
* Un formulario para elegir el tamaño.
* Un botón independiente para **Restablecer preferencia**.

Utiliza este catálogo de opciones:

| Valor que procesa PHP | Texto mostrado | Tamaño del artículo |
| --- | --- | --- |
| `normal` | Normal | `1rem` |
| `grande` | Grande | `1.25rem` |
| `muy-grande` | Muy grande | `1.5rem` |

**En la primera visita, si no existe ninguna preferencia válida, se utilizará Normal.** Entrar en la página no debe crear automáticamente la cookie.

#### 2. Permite guardar la preferencia

El formulario debe enviarse mediante **POST** e incluir un desplegable llamado `tamano`, con las tres opciones anteriores, y un botón **Guardar preferencia**.

Al recibir el formulario:

1. Comprueba que el tamaño recibido sea una cadena y que pertenezca al catálogo permitido.
2. Si es válido, solicita el almacenamiento de una cookie llamada **`ud04_tamano_lectura`**, con el valor elegido.
3. Configura la cookie con estas opciones:

| Opción | Configuración |
| --- | --- |
| Caducidad | 30 días desde que se guarda la preferencia. |
| Ruta | `/` |
| `secure` | `false` para nuestro entorno HTTP local. |
| `httponly` | `true` |
| `samesite` | `Lax` |

4. Si se ha podido preparar la cookie, realiza una **redirección 303 a `lectura.php`** y termina la ejecución.

Si el dato recibido no es válido, muestra un mensaje de error y **no modifiques la cookie**.

#### 3. Recupera la preferencia y aplícala

En cada petición, consulta la cookie y valida su contenido antes de utilizarlo.

* Si contiene una opción permitida, aplica ese tamaño.
* Si no existe o contiene un valor incorrecto, utiliza **Normal**.
* El desplegable debe mostrar seleccionada la opción que se está utilizando.
* El texto que indica el tamaño actual debe coincidir con el tamaño aplicado al artículo.

**El cambio debe apreciarse de verdad en los párrafos del artículo.** Para facilitar la parte visual, puedes utilizar estas clases:

```css
.lectura-normal {
    font-size: 1rem;
}

.lectura-grande {
    font-size: 1.25rem;
}

.lectura-muy-grande {
    font-size: 1.5rem;
}

.articulo {
    line-height: 1.7;
    max-width: 65ch;
}
```

PHP decidirá qué clase permitida añadir al artículo. **La clase debe proceder de tu catálogo validado**, evitando introducir directamente en el HTML cualquier valor recibido del navegador. Escapa los datos al insertarlos en HTML.

No necesitas JavaScript: el tamaño se aplicará en el HTML generado después de la redirección.

#### 4. Permite restablecer la preferencia

El botón **Restablecer preferencia** debe utilizar un formulario POST independiente.

Al pulsarlo:

1. Solicita el borrado de la cookie mediante una fecha de caducidad pasada y las mismas opciones de ámbito utilizadas al crearla.
2. Si se ha podido preparar el borrado, redirige con estado **303** a `lectura.php` y termina la ejecución.
3. La página volverá a mostrar el artículo con tamaño **Normal**.

Para distinguir las operaciones, utiliza los valores **`guardar`** y **`restablecer`** en un campo llamado `accion`. Si llega una acción desconocida, muestra un error sin modificar la cookie.

#### 5. Comprueba el funcionamiento en Chrome

Realiza estas pruebas con las herramientas de desarrollo:

| Prueba | Resultado esperado |
| --- | --- |
| Entrar sin la cookie. | Tamaño Normal y ninguna cookie creada automáticamente. |
| Elegir Grande y guardar. | El artículo aumenta de tamaño y el desplegable muestra Grande. |
| Elegir Muy grande y guardar. | Se actualiza la misma cookie y aumenta de nuevo el tamaño. |
| Recargar la página. | Se mantiene el tamaño elegido. |
| Abrir la misma dirección en otra pestaña del mismo perfil. | Se recupera la preferencia. |
| Cambiar manualmente el valor de la cookie a `gigante`. | La página utiliza Normal y sigue funcionando sin avisos de PHP. |
| Restablecer la preferencia. | Desaparece la cookie y vuelve el tamaño Normal. |

Además, observa en **Red**, con **Conservar registro** activado:

* El **POST** que guarda la preferencia y su respuesta **303** con `Set-Cookie`.
* El **GET** posterior, que envía la cookie mediante la cabecera `Cookie`.

#### Entrega del ejercicio

Entrega la carpeta **`ej06`** con el archivo `lectura.php` y un documento breve llamado **`comprobaciones.md`** que incluya:

* Una captura de la cookie almacenada en Chrome.
* Una captura donde se vea el artículo con tamaño Muy grande.
* Tus respuestas a estas preguntas:

1. ¿Por qué `$_COOKIE` no cambia automáticamente al ejecutar `setcookie()`?
2. ¿Qué permite recuperar la preferencia al abrir otra pestaña?
3. ¿Por qué debemos validar la cookie si la hemos creado desde nuestra propia aplicación?

**En este ejercicio utilizaremos cookies, sin sesiones ni base de datos.** El objetivo es completar el recorrido: recibir una elección, validarla, guardarla en el navegador, recuperarla y aplicarla a una funcionalidad visible.

## Bloque 5. Sesiones

### Ejercicio 7. Un carrito que recuerda tus productos

**Archivo:** `ej07/carrito.php`. Utiliza sesiones de PHP, sin JavaScript ni base de datos.

Una tienda necesita conservar el carrito mientras se navega. No hay usuarios ni compras reales.

#### 1. Catálogo y primera visita

Define este catálogo en PHP. Trabaja los precios en céntimos y conviértelos a euros al mostrarlos:

| Identificador | Producto | Precio |
| --- | --- | ---: |
| teclado | Teclado USB | 20,00 € |
| raton | Ratón óptico | 12,50 € |
| memoria | Memoria USB 64 GB | 9,90 € |

Inicia una sesión con el nombre `UD04CARRITO`, antes del HTML. Guarda el carrito en `$_SESSION['carrito']` como un array que relaciona el identificador del producto con su cantidad. Inicialmente estará vacío. Iníciala con estas instrucciones:

```php
session_name('UD04CARRITO');
session_start();
```

#### 2. Añadir productos

Muestra el catálogo y un formulario POST por producto con `accion=anadir` y `producto` como identificador. Cada envío añade una unidad; si ya existe, incrementa su cantidad. Admite un máximo de 10 unidades por producto. El precio y el nombre proceden siempre del catálogo del servidor.

Valida que el producto sea una cadena y una clave del catálogo. Rechaza productos o acciones desconocidos, campos enviados como arrays y cantidades que excedan el máximo, mostrando un error sin modificar el carrito.

Después de una operación válida, redirige con estado 303 a `carrito.php` y termina la ejecución. Recargar el GET no debe añadir otra unidad.

#### 3. Mostrar y vaciar

Muestra una tabla con producto, precio unitario, cantidad y subtotal, y calcula el total. Escapa el texto al generar HTML y presenta los importes con dos decimales. Si está vacío, muestra «El carrito está vacío».

Añade un formulario POST independiente con `accion=vaciar` que elimine solo `$_SESSION['carrito']`, sin destruir toda la sesión, y redirija con 303. La siguiente petición debe volver a disponer de un carrito vacío.

#### 4. Comprobaciones

| Prueba | Resultado esperado |
| --- | --- |
| Primera visita. | Carrito vacío. |
| Añadir dos teclados y un ratón. | Cantidades 2 y 1; total 52,50 €. |
| Recargar tras añadir. | No se añade otra unidad. |
| Abrir otra pestaña del mismo perfil. | Comparte el carrito; recarga para actualizar su vista. |
| Incógnito sin sesión previa. | Carrito independiente. |
| Añadir una unidad del producto que ya tiene 10. | Error; conserva las 10 unidades. |
| Producto inventado, producto como array o acción desconocida. | Error; carrito sin cambios. |
| Vaciar y recargar. | Carrito vacío. |

#### Entrega

Entrega `ej07/carrito.php` y `comprobaciones.md` con una captura del carrito, otra del POST 303 y GET 200 en Red, y las respuestas:

1. ¿Qué guarda el navegador y qué guarda el servidor?
2. ¿Por qué hay que llamar a `session_start()` en cada página que utilice la sesión?
3. ¿Por qué vaciar el carrito no exige destruir toda la sesión?
4. ¿Por qué no recibimos el precio del producto desde un campo oculto?

## Entrega y comprobación

Entrega las carpetas de los ejercicios solicitados con el código comentado de forma breve. Comprueba cada regla y evita avisos de claves inexistentes o errores al recibir arrays inesperados. La forma concreta de entrega y el plazo se indicarán en Moodle.

**Relación con la programación:** RA3.e y RA3.f (formularios y recuperación de información); RA4.f (uso del entorno para prueba y depuración).

## Autoría

Material elaborado por Joaquín García para DWES de 2.º DAW, IES Portada Alta. Actividades propias vinculadas a los apuntes de la unidad. CC BY-NC-SA.
