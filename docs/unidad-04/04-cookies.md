# 4. Cookies

En los formularios hemos devuelto los valores recibidos dentro del HTML para facilitar su corrección. Pero, si cerramos esa página y volvemos más tarde, **las variables PHP de aquella petición ya no existen**.

¿Cómo puede recordar una tienda que preferimos ver el catálogo en formato lista? Una posibilidad es guardar esa preferencia en el navegador mediante una **cookie**.

## 4.1. Qué es una cookie

Una cookie es un pequeño dato que el navegador almacena para un sitio web. En las peticiones posteriores que cumplen sus condiciones de envío, el navegador lo devuelve al servidor.

Por ejemplo:

```text
ud04_vista=lista
```

| Parte | Significado |
| --- | --- |
| `ud04_vista` | Nombre de la cookie, elegido por nuestra aplicación. |
| `lista` | Valor guardado: la preferencia de visualización. |

**En este ejemplo nosotros elegimos tanto el nombre como el valor.**

El servidor puede solicitar su creación mediante una cabecera **`Set-Cookie`**. Cuando corresponde enviarla, el navegador utiliza una cabecera **`Cookie`**. PHP permite consultar lo recibido en **`$_COOKIE`**.

| Operación | Quién interviene | Herramienta en PHP |
| --- | --- | --- |
| Solicitar que se guarde o actualice una cookie. | El servidor envía una instrucción al navegador. | `setcookie()` |
| Almacenar la cookie. | El navegador, si acepta la instrucción. | No es una variable PHP que quede ejecutándose. |
| Enviar las cookies aplicables en otra petición. | El navegador las incorpora a la petición. | PHP las recibe en `$_COOKIE`. |

Las cookies sirven, por ejemplo, para recordar un idioma, un modo de visualización o una preferencia. Se envían también con peticiones POST: **no dependen exclusivamente de GET**.

!!! important "Un dato del navegador"
    El usuario puede borrar o modificar una cookie, y el navegador puede bloquearla. La aplicación debe funcionar con un valor predeterminado cuando no esté disponible y validar lo recibido antes de utilizarlo.

## 4.2. Antes de empezar: solicitudes y respuestas HTTP

Cada vez que abrimos una página, el navegador envía una **solicitud HTTP** y el servidor devuelve una **respuesta HTTP**. Si la dirección corresponde a un archivo PHP, el servidor lo ejecuta y devuelve el resultado, no el código PHP.

Representaremos los mensajes en HTTP/1.1:

```http
GET /preferencias.php HTTP/1.1
Host: localhost:8080

```

La primera línea es la **línea de petición**: método, ruta y versión de HTTP. Después van las cabeceras. Una línea en blanco las separa del cuerpo, si existe. Este GET no lleva cuerpo.

Una respuesta puede ser:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

<h1>Preferencias del catálogo</h1>
```

La primera línea es la **línea de estado**. `200 OK` indica que la petición se ha atendido correctamente. Las cabeceras describen la respuesta; el cuerpo contiene aquí el HTML que muestra el navegador. En otras respuestas podría contener JSON, una imagen o ningún contenido.

### Dónde viaja la cookie

Al ejecutar `setcookie()`, PHP incorpora **Set-Cookie a las cabeceras de la respuesta**. El navegador recibe la instrucción y guarda la cookie si la acepta:

```http
HTTP/1.1 200 OK
Set-Cookie: ud04_vista=lista; Path=/; HttpOnly; SameSite=Lax
Content-Type: text/html; charset=UTF-8

<p>La página puede enviar una cookie y HTML en la misma respuesta.</p>
```

Este es un ejemplo del intercambio, no el código del formulario completo que construiremos después. En una petición posterior, el navegador devuelve la cookie:

```http
GET /preferencias.php HTTP/1.1
Host: localhost:8080
Cookie: ud04_vista=lista

```

| Mensaje | Cabecera | Qué significa |
| --- | --- | --- |
| Respuesta del servidor. | `Set-Cookie: ud04_vista=lista; ...` | Solicita guardar la cookie con sus opciones. |
| Petición posterior del navegador. | `Cookie: ud04_vista=lista` | Devuelve el nombre y el valor guardados. |

Las opciones, como ruta o caducidad, no se repiten dentro de `Cookie`: el navegador las aplica para decidir cuándo enviar la cookie.

**No hace falta redirigir para enviar una cookie.** `setcookie()` prepara `Set-Cookie`; `header('Location: ...')` añade una instrucción diferente: solicitar otra página. En el ejemplo completo combinaremos ambas en una respuesta 303.

Las peticiones son independientes. **La continuidad se consigue porque el navegador vuelve a enviar el dato**, no porque las variables PHP sigan existiendo.

## 4.3. Crear una cookie con setcookie

Utilizaremos la forma de `setcookie` con un array de opciones:

```php
setcookie('ud04_vista', 'lista', [
    'expires' => time() + 7 * 24 * 60 * 60,
    'path' => '/',
    'secure' => false,
    'httponly' => true,
    'samesite' => 'Lax',
]);
```

El primer argumento es el **nombre**, el segundo el **valor** y el tercero configura su comportamiento.

| Opción | Qué significa en el ejemplo |
| --- | --- |
| `expires` | Fecha de caducidad: dentro de siete días. |
| `path` | `/` permite enviarla a las rutas del mismo host para las que resulte aplicable. |
| `secure` | `false` permite probar este ejemplo en nuestro HTTP local. En una aplicación publicada mediante HTTPS utilizaremos `true`. |
| `httponly` | Impide leer la cookie desde JavaScript con `document.cookie`. El navegador sigue enviándola al servidor. |
| `samesite` | `Lax` restringe determinados envíos entre sitios, manteniendo una navegación habitual cómoda. |

`time()` devuelve la fecha actual como un número de segundos. Sumamos siete días expresados en segundos. Para una hora usaríamos `time() + 3600`.

No indicamos `domain`: la cookie se limita al host que la establece, sin ampliarla a sus subdominios. El puerto no forma parte de esa separación: aplicaciones en distintos puertos de `localhost` pueden compartir cookies. Por eso utilizaremos un nombre específico como `ud04_vista`.

### Cabeceras antes del HTML

`setcookie` prepara una cabecera de respuesta y debe ejecutarse **antes de enviar HTML, espacios o mensajes con `echo`**. Aplicamos el mismo orden que en las redirecciones: primero procesamiento, después presentación.

Si devuelve `true`, PHP ha preparado la cabecera correctamente; eso **no demuestra que el navegador haya aceptado y almacenado la cookie**. Lo comprobaremos en una petición posterior y en las herramientas del navegador.

## 4.4. Leer una cookie: lo que llegó en la petición

Una vez guardada, el navegador puede devolver la cookie en otra solicitud:

```http
GET /preferencias.php HTTP/1.1
Host: localhost:8080
Cookie: ud04_vista=lista

```

PHP recoge las cookies recibidas en el array **`$_COOKIE`**:

```php
$vista = $_COOKIE['ud04_vista'] ?? 'cuadricula';
```

Significa: si llega la cookie, utiliza su valor; si no llega, utiliza `cuadricula`. **El valor predeterminado solo se asigna a una variable PHP: no crea una cookie.**

### 1. Comprobamos que el valor sea válido

El usuario puede modificar la cookie. Aceptamos únicamente los valores previstos:

```php
$vista = $_COOKIE['ud04_vista'] ?? 'cuadricula';

if ($vista !== 'cuadricula' && $vista !== 'lista') {
    $vista = 'cuadricula';
}
```

La condición significa: si no es Cuadrícula ni Lista, utiliza Cuadrícula. Las comparaciones estrictas también rechazan un valor recibido como array.

| Cookie recibida | Valor final de `$vista` |
| --- | --- |
| No llega. | `cuadricula` |
| `ud04_vista=lista` | `lista` |
| `ud04_vista=cuadricula` | `cuadricula` |
| `ud04_vista=inventada` | `cuadricula` |

### 2. Por qué setcookie() no cambia $_COOKIE inmediatamente

Imagina una primera petición sin nuestra cookie:

```http
GET /ejemplo.php HTTP/1.1
Host: localhost:8080

```

PHP ejecuta:

```php
setcookie('ud04_vista', 'lista', ['path' => '/']);
$vista = $_COOKIE['ud04_vista'] ?? 'cuadricula';
```

Aunque acabamos de llamar a `setcookie()`, **$vista vale cuadricula**. Cada instrucción trabaja con un mensaje diferente:

| Instrucción | Qué utiliza |
| --- | --- |
| `$_COOKIE['ud04_vista']` | Lo que llegó en la solicitud actual. |
| `setcookie(...)` | Prepara una instrucción para la respuesta. |

En esta solicitud no llegó la cookie. La respuesta solicita guardarla:

```http
HTTP/1.1 200 OK
Set-Cookie: ud04_vista=lista; Path=/
Content-Type: text/html; charset=UTF-8

<!-- Contenido de la página -->
```

Si el navegador la acepta, la guarda. En una petición posterior donde corresponda enviarla:

```http
GET /ejemplo.php HTTP/1.1
Host: localhost:8080
Cookie: ud04_vista=lista

```

Ahora sí, en esta nueva ejecución:

```php
$vista = $_COOKIE['ud04_vista'] ?? 'cuadricula';
// $vista vale 'lista'.
```

Si ya existía una cookie con otro valor, `$_COOKIE` contendría ese valor anterior durante la petición en la que solicitamos actualizarla.

### 3. Mostrar la preferencia nueva después de guardarla

En el ejemplo completo utilizaremos:

```php
setcookie('ud04_vista', 'lista', $opciones);
header('Location: preferencias.php', true, 303);
exit;
```

El navegador recibe la cookie, la guarda y sigue la redirección mediante GET. **En ese nuevo GET devuelve la cookie actualizada**, y PHP puede leerla.

También podríamos mostrar el valor nuevo utilizando directamente la elección validada del formulario, pero la redirección nos permite practicar POST → redirección → GET y evita repetir el formulario al recargar.

Por último:

```php
$_COOKIE['ud04_vista'] = 'lista';
```

solo modifica el array de la ejecución actual. **No solicita guardar una cookie en el navegador.** Los mensajes HTTP se han simplificado para mostrar lo relevante.

## 4.5. Duración y ámbito

### Cookies persistentes y cookies de sesión

* Con una fecha futura en `expires`, solicitamos una **cookie persistente**, que puede mantenerse entre aperturas del navegador hasta su caducidad. El usuario puede borrarla antes.
* Sin `expires`, o con valor `0`, se crea una **cookie de sesión del navegador**. No tiene una fecha persistente y su duración depende de la sesión del navegador. Las funciones de restauración pueden recuperarla al reabrirlo; no debemos tomar el cierre como garantía de borrado inmediato.

Una cookie de sesión del navegador **no equivale a utilizar `$_SESSION` en PHP**. Estudiaremos las sesiones del servidor en el siguiente apartado.

### La opción path

| Ruta configurada | Dónde puede enviarse la cookie, si se cumplen las demás condiciones |
| --- | --- |
| `/` | En las rutas del host. |
| `/unidad-04/` | En esa ruta y sus subrutas. |

`path` es una **ruta web**, no una carpeta como `/var/www/html`. No es una barrera de seguridad entre aplicaciones del mismo sitio. Para facilitar la prueba sin depender de la carpeta donde coloques el ejemplo, utilizaremos `/` y un nombre específico.

`localhost` y `127.0.0.1` son hosts diferentes: si creas la cookie en uno y abres el otro, no esperes recibirla.

## 4.6. Modificar y eliminar una cookie

Para modificarla, enviamos un valor nuevo con el mismo **nombre, dominio y ruta**:

```php
setcookie('ud04_vista', 'cuadricula', [
    'expires' => time() + 7 * 24 * 60 * 60,
    'path' => '/',
    'secure' => false,
    'httponly' => true,
    'samesite' => 'Lax',
]);
```

La caducidad vuelve a contar siete días desde esta actualización.

Para borrarla, solicitamos una caducidad en el pasado:

```php
setcookie('ud04_vista', '', [
    'expires' => time() - 3600,
    'path' => '/',
    'secure' => false,
    'httponly' => true,
    'samesite' => 'Lax',
]);
```

Mantendremos las mismas opciones. En particular, **nombre, dominio y ruta deben identificar la misma cookie**; cambiar la ruta puede dejar otra cookie con el mismo nombre.

`unset($_COOKIE['ud04_vista']);` solo elimina una entrada del array actual: no ordena al navegador borrar la cookie. Después de solicitar el borrado, la cookie que llegó en esta petición aún puede estar en `$_COOKIE`. Tras redirigir, la siguiente petición ya reflejará el cambio si el navegador lo ha aplicado.

## 4.7. Ejemplo completo: recordar la vista del catálogo

Vamos a crear **un único archivo, preferencias.php**, que muestra una preferencia, permite guardarla durante siete días y permite borrarla. Mostramos el nombre de la vista; no necesitamos CSS ni construir todavía un catálogo real.

**Qué queremos comprobar:** las variables PHP se crean de nuevo en cada petición, pero el navegador puede conservar la preferencia y devolverla mediante una cookie.

### 1. El archivo completo: preferencias.php

Guárdalo en el servidor PHP local. GitHub Pages muestra los apuntes, pero no ejecuta PHP.

```php
<?php
// Valores y etiquetas definidos por la aplicación.
$vistas = ['cuadricula' => 'Cuadrícula', 'lista' => 'Lista'];

$opciones = [
    'expires' => time() + 7 * 24 * 60 * 60,
    'path' => '/',
    'secure' => false, // HTTP local; true con HTTPS.
    'httponly' => true,
    'samesite' => 'Lax',
];

// Solo aceptamos los dos valores conocidos.
$vista = $_COOKIE['ud04_vista'] ?? 'cuadricula';
if ($vista !== 'cuadricula' && $vista !== 'lista') {
    $vista = 'cuadricula';
}

$error = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $accion = $_POST['accion'] ?? '';

    if ($accion === 'guardar') {
        $elegida = $_POST['vista'] ?? '';

        if ($elegida === 'cuadricula' || $elegida === 'lista') {
            setcookie('ud04_vista', $elegida, $opciones);
        } else {
            $error = 'Selecciona una vista válida.';
        }
    } elseif ($accion === 'borrar') {
        $opciones['expires'] = time() - 3600;
        setcookie('ud04_vista', '', $opciones);
    } else {
        $error = 'La acción no es válida.';
    }

    if ($error === '') {
        header('Location: preferencias.php', true, 303);
        exit;
    }
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Preferencias del catálogo</title>
</head>
<body>
    <h1>Preferencias del catálogo</h1>
    <p>Vista actual: <strong><?= $vistas[$vista] ?></strong></p>

    <?php if ($error !== ''): ?>
        <p><?= $error ?></p>
    <?php endif; ?>

    <form action="preferencias.php" method="post">
        <label for="vista">Vista del catálogo:</label>
        <select id="vista" name="vista">
            <option value="cuadricula"
                <?= $vista === 'cuadricula' ? 'selected' : '' ?>>
                Cuadrícula
            </option>
            <option value="lista"
                <?= $vista === 'lista' ? 'selected' : '' ?>>
                Lista
            </option>
        </select>
        <button name="accion" value="guardar">Guardar preferencia</button>
    </form>

    <form action="preferencias.php" method="post">
        <button name="accion" value="borrar">Borrar preferencia</button>
    </form>
</body>
</html>
```

Hemos mantenido las comprobaciones necesarias para aceptar únicamente dos valores conocidos. Las comparaciones estrictas también rechazan arrays sin utilizarlos como claves. No necesitamos una función de escape en este ejemplo: las etiquetas y los mensajes que mostramos son textos definidos por la aplicación, no la entrada recibida directamente.

Las opciones de `setcookie()` se mantienen porque son parte del contenido que estamos aprendiendo. Suponemos que PHP puede preparar las cabeceras correctamente; el significado de su valor de retorno se ha explicado en el 4.3.

### 2. Para qué sirven los formularios

Ambos formularios tienen `action="preferencias.php"` y `method="post"`: envían los datos al mismo archivo mediante POST.

| Control | Dato enviado |
| --- | --- |
| Desplegable con `name="vista"`. | `vista=cuadricula` o `vista=lista`. |
| Botón Guardar preferencia. | `accion=guardar`. |
| Botón del segundo formulario. | `accion=borrar`. |

El botón pulsado aporta su nombre y valor al envío. El formulario de borrado es independiente: no envía el desplegable del primer formulario.

### 3. Primera solicitud: abrir la página sin cookie

Abre `http://localhost:8080/preferencias.php` si el archivo está en la raíz pública. Si está en una subcarpeta, adapta la ruta. Partimos sin la cookie `ud04_vista`.

El navegador todavía no ha recibido el formulario. Solicita la página:

```http
GET /preferencias.php HTTP/1.1
Host: localhost:8080

```

PHP ejecuta el archivo desde el principio. Prepara el catálogo de vistas y las opciones, pero **crear esos arrays no crea ninguna cookie**.

Después ejecuta:

```php
$vista = $_COOKIE['ud04_vista'] ?? 'cuadricula';
```

Como la cookie no existe, usa `cuadricula`. `??` evita consultar una clave inexistente sin alternativa. La comprobación posterior mantiene esa opción.

La petición es GET, de modo que se salta todo el bloque POST y genera el HTML. La respuesta es, simplificada:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

<h1>Preferencias del catálogo</h1>
<p>Vista actual: <strong>Cuadrícula</strong></p>
<form action="preferencias.php" method="post">
    <!-- Desplegable con Cuadrícula seleccionada y botón Guardar -->
</form>
<form action="preferencias.php" method="post">
    <!-- Botón Borrar preferencia -->
</form>
```

**No hay Set-Cookie en esta respuesta:** mostramos el valor predeterminado sin guardarlo automáticamente. El navegador interpreta el HTML y muestra los formularios.

### 4. Segunda solicitud: elegir Lista y guardar

Seleccionamos Lista y pulsamos Guardar preferencia. El navegador envía:

```http
POST /preferencias.php HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded
Content-Length: 26

vista=lista&accion=guardar
```

Todavía no envía nuestra cookie, porque no la hemos creado. Los datos del formulario viajan en el cuerpo y PHP los recibe en `$_POST`.

PHP vuelve a ejecutar el archivo completo. La lectura inicial de `$_COOKIE` sigue sin encontrar una preferencia: al comenzar este POST, el navegador aún no la ha enviado.

Ahora sí entra en el bloque POST. Obtiene `accion=guardar` y `vista=lista`. Comprueba que la vista es una de las dos opciones permitidas y ejecuta:

```php
setcookie('ud04_vista', $elegida, $opciones);
```

En esta ejecución `$elegida` vale `lista`. `setcookie()` **prepara una cabecera de respuesta**, no modifica automáticamente `$_COOKIE`.

Después ejecuta:

```php
header('Location: preferencias.php', true, 303);
exit;
```

`header()` prepara la redirección. `exit` termina el script sin llegar al HTML. La respuesta lleva ambas instrucciones:

```http
HTTP/1.1 303 See Other
Set-Cookie: ud04_vista=lista; Max-Age=604800; Path=/; HttpOnly; SameSite=Lax
Location: preferencias.php

```

| Parte | Qué significa |
| --- | --- |
| 303 See Other. | Solicita continuar mediante GET en la dirección indicada. |
| Set-Cookie. | Solicita guardar `ud04_vista=lista` durante siete días. |
| Location. | Indica el destino: `preferencias.php`. |

604800 son los segundos de siete días. PHP puede incluir además `Expires` con la fecha correspondiente. No aparece `Secure` porque hemos utilizado `secure=false` para HTTP local.

**La cookie no depende de la redirección para enviarse:** ambas instrucciones viajan en las cabeceras de la misma respuesta. En este caso usamos la redirección para leer la preferencia nueva en el GET posterior y evitar repetir el POST al recargar.

### 5. Tercera solicitud: seguir la redirección

El navegador guarda la cookie si la acepta y sigue la redirección automáticamente:

```http
GET /preferencias.php HTTP/1.1
Host: localhost:8080
Cookie: ud04_vista=lista

```

Ahora sí envía la preferencia, dentro de la cabecera `Cookie`. No vuelve a enviar el formulario: este GET no contiene `vista=lista&accion=guardar` en el cuerpo.

PHP comienza otra ejecución y encuentra:

```php
$vista = $_COOKIE['ud04_vista'] ?? 'cuadricula';
```

Esta vez `$vista` recibe `lista`, porque la cookie llegó en la solicitud. La comprobación acepta ese valor. Como es GET, se salta el procesamiento POST y genera el HTML:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

<h1>Preferencias del catálogo</h1>
<p>Vista actual: <strong>Lista</strong></p>
<form action="preferencias.php" method="post">
    <!-- Desplegable con Lista seleccionada y botón Guardar -->
</form>
<form action="preferencias.php" method="post">
    <!-- Botón Borrar preferencia -->
</form>
```

No necesita reenviar `Set-Cookie` para leer la preferencia. El navegador ya la tiene guardada.

El atributo `selected` del desplegable se genera mediante:

```php
<?= $vista === 'lista' ? 'selected' : '' ?>
```

Si la vista actual es Lista, PHP escribe `selected` en esa opción. Si no, escribe una cadena vacía. Así la selección coincide con el texto de la vista actual.

### 6. Recargar o volver más tarde

Mientras la cookie siga disponible y corresponda enviarla, el navegador incorpora:

```http
Cookie: ud04_vista=lista
```

En cada nueva petición, PHP la lee, comprueba el valor y genera la página. **Lo que ha permanecido es la cookie del navegador, no la variable $vista de la ejecución anterior.**

Si guardamos después Cuadrícula, el POST puede enviar a la vez la cookie anterior `ud04_vista=lista` en sus cabeceras y la nueva elección `vista=cuadricula` en el cuerpo. PHP responde con `Set-Cookie` para actualizarla; el GET posterior recibe el nuevo valor.

### 7. Solicitud para borrar la preferencia

Pulsamos el botón del segundo formulario. El navegador envía:

```http
POST /preferencias.php HTTP/1.1
Host: localhost:8080
Cookie: ud04_vista=lista
Content-Type: application/x-www-form-urlencoded
Content-Length: 13

accion=borrar
```

La cookie todavía llega en este POST: el navegador no la ha borrado por pulsar un botón. Es el código PHP el que prepara la instrucción de borrado.

PHP detecta la acción y ejecuta:

```php
$opciones['expires'] = time() - 3600;
setcookie('ud04_vista', '', $opciones);
```

Mantiene el nombre y el ámbito de la cookie, pero indica una caducidad pasada. Después redirige y termina:

```php
header('Location: preferencias.php', true, 303);
exit;
```

La respuesta es, simplificada:

```http
HTTP/1.1 303 See Other
Set-Cookie: ud04_vista=deleted; Max-Age=0; Path=/; HttpOnly; SameSite=Lax
Location: preferencias.php

```

Puede aparecer también `Expires` con una fecha pasada. `deleted` es un valor que PHP puede utilizar al preparar el borrado: no representa una preferencia nueva.

### 8. Solicitud posterior al borrado

Si el navegador procesa la instrucción, elimina la cookie y sigue la redirección:

```http
GET /preferencias.php HTTP/1.1
Host: localhost:8080

```

Ya no envía `ud04_vista`. PHP vuelve a utilizar `cuadricula`, se salta el bloque POST y responde:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

<h1>Preferencias del catálogo</h1>
<p>Vista actual: <strong>Cuadrícula</strong></p>
<!-- Los dos formularios; Cuadrícula vuelve a estar seleccionada -->
```

**Volver al valor predeterminado no crea otra cookie.** Solo se crea o actualiza al guardar una elección válida.

### 9. Resumen del recorrido

| Navegador | Solicitud o respuesta | Qué hace PHP |
| --- | --- | --- |
| Abre la página sin preferencia. | GET → 200 con HTML. | Utiliza Cuadrícula sin crear la cookie. |
| Elige Lista y guarda. | POST con vista y acción. | Comprueba la elección y prepara la cookie. |
| Recibe las instrucciones. | 303 con Set-Cookie y Location. | Termina el POST sin generar HTML. |
| Guarda la cookie y sigue la redirección. | GET con Cookie → 200. | Lee la preferencia y muestra Lista. |
| Vuelve más tarde. | GET con Cookie → 200. | Recupera la preferencia recibida. |
| Pulsa Borrar preferencia. | POST con accion=borrar y la cookie anterior. | Prepara el borrado. |
| Recibe las instrucciones. | 303 con caducidad pasada y Location. | Termina el POST. |
| Elimina la cookie y sigue la redirección. | GET sin nuestra cookie → 200. | Vuelve a Cuadrícula sin crear otra cookie. |

### Tres ideas clave

* **La cookie se guarda en el navegador; las variables PHP se reconstruyen en cada petición.**
* **Set-Cookie va en la respuesta; Cookie vuelve en una petición posterior.**
* **setcookie() solicita guardar o borrar la cookie; $_COOKIE contiene lo recibido en la petición actual y no cambia automáticamente al llamar a setcookie().**

Aquí recordamos una preferencia del navegador, no un historial ni una identidad. Nosotros elegimos nombre y valor (`ud04_vista=lista`). En las sesiones del siguiente apartado, la cookie contendrá un identificador gestionado por PHP y los datos de `$_SESSION` permanecerán en el servidor.

Los mensajes HTTP son representaciones didácticas en HTTP/1.1: omitimos cabeceras adicionales y abreviamos el HTML. Las fechas de caducidad reales las calcula PHP. Suponemos que el navegador acepta y procesa las cookies; si las bloquea, la página utiliza el valor predeterminado.

## 4.8. Inspeccionar y probar la cookie en Chrome

Vamos a comprobar **qué guarda el navegador y qué viaja en cada solicitud y respuesta**. Abre `preferencias.php` desde el servidor PHP local:

```text
http://localhost:8080/preferencias.php
```

Adapta la dirección si el archivo está en una subcarpeta.

### 1. Ver la cookie guardada: Aplicación

1. Pulsa **F12** para abrir las herramientas de desarrollo.
2. Entra en **Aplicación (Application)**. Si no aparece, búscala en **»**.
3. En el panel izquierdo, despliega **Almacenamiento → Cookies**.
4. Selecciona tu servidor, por ejemplo `http://localhost:8080`.
5. En la página, selecciona Lista y pulsa **Guardar preferencia**.
6. Vuelve a la tabla de cookies.

| Campo | Valor esperado |
| --- | --- |
| Nombre (Name) | `ud04_vista` |
| Valor (Value) | `lista` |
| Ruta (Path) | `/` |
| Caducidad (Expires / Max-Age) | Aproximadamente siete días después del guardado. |
| HttpOnly | Activado. |
| Secure | Desactivado para HTTP local. |
| SameSite | `Lax` |

Si todavía no has guardado ninguna preferencia, **es normal que esta cookie no exista**: la primera visita solo utiliza Cuadrícula como valor predeterminado.

### 2. Ver lo que viaja: Red

1. Entra en **Red (Network)**.
2. Marca **Conservar registro (Preserve log / Keep log)** para mantener visibles las peticiones después de la redirección.
3. Selecciona Lista y pulsa **Guardar preferencia**.

Verás dos peticiones a `preferencias.php`:

| Petición | Estado de la respuesta | Motivo |
| --- | ---: | --- |
| POST | 303 | El formulario envía la elección y PHP responde con una redirección. |
| GET | 200 | El navegador sigue la redirección y recibe la página actualizada. |

**Son dos solicitudes al mismo archivo, no dos archivos diferentes.** Si no distingues el método, consulta Request Method en General dentro de Headers. Los nombres de algunos controles pueden variar según la versión o el idioma.

### 3. Revisar el POST y su respuesta

Selecciona el POST. En **Carga útil (Payload) → Datos del formulario (Form Data)**, busca:

```text
vista: lista
accion: guardar
```

Son los datos que PHP recibe en `$_POST`. Después abre **Cabeceras (Headers) → Cabeceras de respuesta (Response Headers)**:

```http
Set-Cookie: ud04_vista=lista; ...
Location: preferencias.php
```

`Set-Cookie` solicita guardar la preferencia. `Location`, junto con 303, indica la redirección. Si ya había una cookie, el POST también puede incluir su valor anterior en las cabeceras de solicitud.

### 4. Revisar el GET posterior

Selecciona el GET posterior al POST. En **Cabeceras de solicitud (Request Headers)**, busca:

```http
Cookie: ud04_vista=lista
```

**El navegador devuelve la cookie que acaba de guardar.** PHP la recibe en `$_COOKIE` y genera la página con Lista seleccionada.

En **Respuesta (Response)** puedes consultar el HTML devuelto. No aparecerá el código PHP: ya se ha ejecutado.

### 5. Comprobar el borrado

Pulsa **Borrar preferencia** y observa:

1. Un POST que envía `accion=borrar`.
2. Una respuesta 303 con `Set-Cookie` que solicita borrar la cookie.
3. Un GET posterior que ya no envía `ud04_vista`, si el navegador ha procesado el borrado.
4. Una respuesta 200 que vuelve a mostrar Cuadrícula.

En Aplicación → Cookies comprueba que `ud04_vista` ha desaparecido.

### 6. Pruebas finales

| Prueba | Resultado esperado |
| --- | --- |
| Entrar sin la cookie. | Cuadrícula, sin crear automáticamente la cookie. |
| Guardar Lista. | Cookie con valor `lista` y Lista seleccionada. |
| Recargar. | Se mantiene la preferencia. |
| Abrir otra pestaña del mismo perfil. | Recupera la misma preferencia. |
| Guardar Cuadrícula. | Actualiza la misma cookie y su caducidad. |
| Cambiar manualmente su valor a `inventada` y recargar. | La página utiliza Cuadrícula; no acepta el valor desconocido. |
| Borrar la preferencia. | Desaparece la cookie y vuelve Cuadrícula. |
| Cerrar la pestaña y volver durante la vigencia. | La preferencia persistente puede seguir disponible. |
| Incógnito sin cookies previas del ejemplo. | No comparte la preferencia del perfil normal. |

**Aplicación muestra lo que guarda el navegador; Red muestra lo que viaja en cada solicitud y respuesta.**

## 4.9. Buenas prácticas y errores habituales

| Situación | Cómo actuar |
| --- | --- |
| Leer una cookie inexistente. | Utilizar un valor predeterminado con `??`. |
| Confiar en el valor porque procede de una cookie. | Validar tipo y valores permitidos; escapar al mostrar datos en HTML. |
| Llamar a `setcookie` después del HTML. | Preparar cookies antes de cualquier salida. |
| Esperar el valor nuevo inmediatamente en `$_COOKIE`. | Recordar que contiene lo recibido, no lo que estamos enviando. |
| Borrar con una ruta diferente. | Utilizar el mismo nombre, dominio y ruta. |
| Guardar contraseñas, precios finales o roles como datos confiables. | No utilizar cookies manipulables como prueba de identidad, autoridad o importe. |
| Confundir `HttpOnly` con cifrado. | Impide lectura desde JavaScript, pero no cifra el valor ni evita que el usuario lo cambie. |
| Confundir `SameSite` con una protección completa. | Es una ayuda; no sustituye todos los controles de seguridad ni los tokens CSRF. |

Las cookies deben contener pocos datos. Sus tamaños y cantidades están limitados por los navegadores, y se envían repetidamente en peticiones aplicables: no guardaremos imágenes, archivos o catálogos completos en ellas.

`Secure` limita el envío a HTTPS; no cifra la cookie almacenada. En nuestro ejemplo hemos indicado explícitamente `false` para el entorno HTTP local. Al publicar con HTTPS, configuraremos `true`, sin depender de excepciones particulares del navegador para localhost.

## 4.10. Qué debemos recordar

* Las variables PHP se reconstruyen en cada petición; una cookie introduce continuidad mediante el navegador.
* `setcookie` prepara una cabecera de respuesta y `$_COOKIE` contiene los datos recibidos del cliente.
* La cookie nueva se recibe en una petición posterior si el navegador la acepta y corresponde enviarla.
* Para actualizar y borrar hay que identificar la misma cookie por nombre, dominio y ruta.
* Una cookie es entrada del cliente: validamos sus valores y prevemos que falte.
* En el siguiente apartado veremos **sesiones**, para mantener datos en el servidor asociados a un navegador.

## Anexo · Esquema de pizarra

[Descargar el PDF: Recordar la vista del catálogo](../Pizarra_cookies_PHP.pdf)

## Practica lo aprendido

Realiza el **ejercicio 6 del bloque 4: Cookies**, «Una web que recuerda cómo prefieres leer», de la [relación de actividades](actividades.md#bloque-4-cookies). Crearás una preferencia de tamaño de lectura que podrás guardar, recuperar y restablecer, y comprobarás su funcionamiento con las herramientas de Chrome.

Comenzaremos el ejercicio en clase y completarás en casa lo pendiente. Revisaremos la solución por separado de los enunciados.

## Referencias

* [Aitor Medrano y Luis Alemañ: Programación web](https://aitor-medrano.github.io/dwes2122/04web.html), apartado de cookies como referencia didáctica.
* [Manual de PHP: setcookie](https://www.php.net/manual/es/function.setcookie.php).
* [Manual de PHP: $_COOKIE](https://www.php.net/manual/es/reserved.variables.cookies.php).

## Autoría y licencia

Material elaborado por **Joaquín García** para DWES de 2.º DAW del **IES Portada Alta**. Adapta y amplía los materiales de **Aitor Medrano** y **Luis Alemañ**, contrastando el comportamiento con la documentación de PHP.

Licencia **Creative Commons Reconocimiento-NoComercial-CompartirIgual (CC BY-NC-SA)**.
