# 1. Peticiones y respuestas HTTP en PHP

En la unidad 2 aprendimos a recoger datos mediante GET y POST. Antes de construir aplicaciones con formularios, archivos y usuarios, vamos a comprender **qué ocurre entre el navegador y el servidor** cada vez que abrimos una página o enviamos información.

## 1.1. El navegador solicita y el servidor responde

Cuando visitamos una dirección como esta:

```text
http://localhost:8080/unidad-04/catalogo.php
```

el navegador envía una **petición HTTP** solicitando esa página. El servidor recibe la petición, hace que PHP ejecute el archivo y devuelve una **respuesta HTTP** con el resultado.

El recorrido es el siguiente:

1. **El navegador solicita una página.** Puede hacerlo al escribir una dirección, pulsar un enlace o enviar un formulario.
2. **PHP procesa la petición en el servidor.** Ejecuta instrucciones, consulta los datos recibidos y prepara el contenido.
3. **El servidor devuelve una respuesta.** El navegador recibe el resultado y lo muestra.

Por ejemplo, si el archivo contiene:

```php
<?php
$producto = 'Teclado mecánico';
?>
<h1><?= $producto ?></h1>
```

el navegador recibe:

```html
<h1>Teclado mecánico</h1>
```

**El navegador no recibe ni ejecuta el código PHP**: recibe el contenido generado. En este ejemplo el texto está definido en el programa; cuando mostremos datos del usuario, aprenderemos a tratarlos correctamente.

!!! important "Utiliza nuestro servidor"
    Abre los ejemplos a través de `http://localhost:8080/...`, adaptando la ruta a tu proyecto. Si abres un archivo mediante `file:///...`, no estás solicitándolo al servidor y PHP no se ejecuta.

### Qué viaja en cada dirección

| Petición: del navegador al servidor | Respuesta: del servidor al navegador |
| --- | --- |
| Método, como GET o POST. | Código de estado que informa del resultado. |
| Ruta solicitada y parámetros de la URL. | Cabeceras, como el tipo de contenido o una redirección. |
| Cabeceras y, cuando corresponda, un cuerpo con datos. | Normalmente, un cuerpo con HTML u otro contenido. |

Las **cabeceras HTTP** son información que acompaña a la petición o a la respuesta. No son las etiquetas `<head>` o `<header>` de HTML.

No necesitamos memorizar ahora todos los códigos de estado. Nos encontraremos, por ejemplo, con **200** cuando una petición se atiende correctamente, **404** cuando no se encuentra un recurso y **302** en una redirección temporal.

## 1.2. GET y POST: dos formas de enviar información

Ya hemos utilizado estos métodos en los formularios. Recordemos su finalidad:

| Método | Uso habitual | Dónde viajan los campos del formulario | Cómo los recuperamos |
| --- | --- | --- | --- |
| **GET** | Consultar información: buscar, filtrar o paginar. | En los parámetros de la URL. | `$_GET` |
| **POST** | Enviar información para procesarla, especialmente al crear o modificar datos. | En el cuerpo de la petición. | `$_POST` |

### Ejemplo con GET

Si un formulario de búsqueda utiliza GET, podemos obtener una dirección como esta:

```text
http://localhost:8080/unidad-04/catalogo.php?categoria=teclados
```

La parte situada después de `?` es la **cadena de consulta**. PHP permite recuperar el parámetro así:

```php
$categoria = $_GET['categoria'] ?? '';
```

La clave es `categoria` y su valor es `teclados`. El operador `??` permite utilizar un valor alternativo cuando esa clave no existe.

Esta URL se puede copiar, compartir o guardar como marcador para volver a la misma consulta.

### Ejemplo con POST

Para enviar el nombre de un producto a un archivo que lo procese:

```html
<form action="procesar.php" method="post">
    <label for="nombre">Nombre del producto:</label>
    <input type="text" id="nombre" name="nombre">
    <button type="submit">Enviar</button>
</form>
```

En `procesar.php` recuperamos el campo mediante:

```php
$nombre = $_POST['nombre'] ?? '';
```

El atributo **`name`** determina la clave del array. El atributo `id` relaciona el campo con su etiqueta, pero no da nombre al dato enviado.

!!! warning "POST no cifra los datos"
    Los campos no aparecen en la barra de direcciones, pero pueden consultarse en las herramientas del navegador. **HTTPS** protege la comunicación durante el transporte. No debemos enviar contraseñas como parámetros de una URL.

Utilizaremos GET para consultar información y POST para operaciones como dar de alta un producto. Un enlace que se visita mediante GET no debe borrar datos ni confirmar una compra.

!!! note "Una precisión sobre $_GET"
    `$_GET` recoge los parámetros de la URL, incluso si la petición utiliza POST. El método de la petición y la procedencia de cada dato son conceptos distintos.

## 1.3. Información de la petición: $_SERVER

Además de los campos de un formulario, PHP puede consultar información sobre la petición y el entorno en el que se ejecuta.

**`$_SERVER` es un array asociativo** que contiene datos como el método utilizado, la ruta solicitada y determinadas cabeceras del navegador. Es una superglobal: podemos acceder a ella también desde una función sin declararla con `global`.

Estas son algunas de sus claves:

| Clave | Qué contiene | Ejemplo orientativo |
| --- | --- | --- |
| `REQUEST_METHOD` | Método utilizado en la petición. | `GET` o `POST` |
| `REQUEST_URI` | Ruta solicitada, normalmente con los parámetros de consulta. | `/unidad-04/catalogo.php?categoria=teclados` |
| `QUERY_STRING` | Parámetros de consulta sin el signo `?`. | `categoria=teclados` |
| `SCRIPT_NAME` | Ruta web del script que se ejecuta. | `/unidad-04/catalogo.php` |
| `SERVER_NAME` | Nombre del servidor según su configuración. | `localhost` |
| `SERVER_SOFTWARE` | Información sobre el software del servidor. | Un valor que identifica Apache y su versión. |
| `HTTP_USER_AGENT` | Información con la que el cliente se identifica. | Una cadena que identifica el navegador. |
| `HTTP_REFERER` | Dirección de procedencia, si el navegador la envía. | URL de una página desde la que se ha seguido un enlace. |

Los valores dependen del servidor y de la petición. **Algunas claves pueden no existir**, por lo que utilizaremos `??` para proporcionar un texto alternativo.

No hay que memorizar toda la tabla: interesa saber qué información existe y dónde consultarla. Nos centraremos especialmente en `REQUEST_METHOD`.

### Ejemplo: mostrar los datos de la petición

Guarda este ejemplo como **`peticion.php`** en la carpeta de prácticas de la unidad:

```php
<?php
declare(strict_types=1);

$claves = [
    'REQUEST_METHOD',
    'REQUEST_URI',
    'QUERY_STRING',
    'SCRIPT_NAME',
    'SERVER_NAME',
    'SERVER_SOFTWARE',
    'HTTP_USER_AGENT',
    'HTTP_REFERER',
];
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Información de la petición</title>
</head>
<body>
    <h1>Información de la petición</h1>

    <table>
        <thead>
            <tr><th>Clave</th><th>Valor</th></tr>
        </thead>
        <tbody>
            <?php foreach ($claves as $clave): ?>
                <tr>
                    <td><?= $clave ?></td>
                    <td><?= htmlspecialchars(
                        $_SERVER[$clave] ?? 'No disponible',
                        ENT_QUOTES,
                        'UTF-8'
                    ) ?></td>
                </tr>
            <?php endforeach; ?>
        </tbody>
    </table>

    <h2>Enviar una petición GET</h2>
    <a href="peticion.php?categoria=monitores">Consultar monitores</a>

    <h2>Enviar una petición POST</h2>
    <form action="peticion.php" method="post">
        <label for="producto">Producto:</label>
        <input type="text" id="producto" name="producto">
        <button type="submit">Enviar</button>
    </form>
</body>
</html>
```

El código utiliza herramientas que ya conocemos: un array con las claves que queremos consultar y un `foreach` para construir las filas de la tabla.

En cada fila, **`$_SERVER[$clave]`** consulta el valor correspondiente. Si no existe, `?? 'No disponible'` proporciona el texto alternativo.

`htmlspecialchars` permite mostrar esos valores como texto en HTML, evitando interpretar caracteres como `<` como parte del marcado. Profundizaremos en esta función en el apartado de formularios y validación. Las claves de la primera columna son textos fijos definidos por nosotros.

### Qué observar al ejecutarlo

| Acción | Qué cambia |
| --- | --- |
| Abrir `peticion.php` desde la barra de direcciones. | El método será GET y normalmente no habrá parámetros de consulta. |
| Pulsar «Consultar monitores». | El método seguirá siendo GET y aparecerá `categoria=monitores` en `QUERY_STRING` y en la URL. |
| Escribir un producto y enviar el formulario. | El método será POST. El producto viajará en el cuerpo, sin aparecer en la URL. |

La tabla no muestra el valor del campo `producto`, porque este se recoge en **`$_POST`**, no en `$_SERVER`.

!!! note "Información del cliente"
    `HTTP_REFERER` puede faltar, por ejemplo por motivos de privacidad. Esa cabecera y `HTTP_USER_AGENT` pueden manipularse: nos ayudan a observar una petición, pero no sirven para demostrar quién es un usuario.

### Saber si se ha enviado un formulario

Una aplicación puede mostrar un formulario al recibir GET y procesarlo al recibir POST:

```php
if (($_SERVER['REQUEST_METHOD'] ?? '') === 'POST') {
    // Recoger y procesar los datos enviados.
} else {
    // En nuestro ejemplo, mostrar el formulario en la visita inicial.
}
```

No necesitamos deducirlo a partir de si un campo está vacío: **`REQUEST_METHOD` indica el método de la petición actual**. Utilizaremos esta comprobación en los ejemplos del siguiente apartado.

## 1.4. Cabeceras HTTP

Las **cabeceras HTTP** son información adicional que acompaña a una petición o a una respuesta. Describen aspectos de la comunicación: qué contenido acepta el navegador, qué devuelve el servidor o cómo debe actuar el cliente.

Se escriben como parejas de **nombre y valor**, separados por dos puntos:

```http
Content-Type: text/html; charset=UTF-8
```

### Cabeceras de petición y de respuesta

| Dirección | Cabecera | Qué comunica |
| --- | --- | --- |
| Navegador → servidor | `Host` | Servidor al que se dirige la petición y, cuando corresponde, puerto. |
| Navegador → servidor | `Accept` | Tipos de contenido que acepta el cliente. |
| Navegador → servidor | `Accept-Language` | Preferencias de idioma del cliente. |
| Navegador → servidor | `User-Agent` | Información con la que se identifica el navegador. |
| Servidor → navegador | `Content-Type` | Tipo de contenido del cuerpo de la respuesta. |
| Servidor → navegador | `Location` | Dirección de destino de una redirección. |
| Servidor → navegador | `Set-Cookie` | Instrucción para guardar o actualizar una cookie. |

Algunas cabeceras de petición se pueden consultar desde `$_SERVER`. Por ejemplo, la cabecera `User-Agent` aparece en `$_SERVER['HTTP_USER_AGENT']`, como hemos visto en la tabla anterior.

### Un intercambio en ambas direcciones

Imaginemos que el navegador solicita el catálogo y el servidor lo redirige a una página de acceso. Este ejemplo muestra **todas las cabeceras de la tabla** dentro de un mismo intercambio simplificado, representado en formato HTTP/1.1. Las cabeceras concretas pueden variar según el navegador y el servidor.

**1. Petición del navegador al servidor:**

```http
GET /unidad-04/catalogo.php HTTP/1.1
Host: localhost:8080
Accept: text/html
Accept-Language: es-ES,es;q=0.9
User-Agent: Mozilla/5.0 ...
```

El navegador comunica lo siguiente:

* La primera línea solicita `/unidad-04/catalogo.php` mediante **GET**. Esa línea identifica el método, la ruta y la versión; no es una cabecera.
* **`Host: localhost:8080`** indica el servidor y el puerto de destino.
* **`Accept: text/html`** indica que acepta recibir contenido HTML.
* **`Accept-Language: es-ES,es;q=0.9`** expresa una preferencia por español de España y, después, por español en general. `q=0.9` indica una prioridad relativa menor; la primera opción tiene prioridad 1 por defecto. Es una preferencia, no obliga al servidor a traducir el contenido.
* **`User-Agent: Mozilla/5.0 ...`** representa la información con la que se identifica el navegador. La hemos abreviado para facilitar la lectura.

**2. Respuesta del servidor al navegador:**

```http
HTTP/1.1 302 Found
Content-Type: text/html; charset=UTF-8
Location: /unidad-04/acceso.php
Set-Cookie: idioma=es; Path=/; SameSite=Lax

<p>Redirigiendo a la página de acceso...</p>
```

El servidor comunica lo siguiente:

* La primera línea indica el estado **302**, una redirección temporal. No es una cabecera.
* **`Content-Type`** indica que el cuerpo de esta respuesta contiene HTML codificado en UTF-8.
* **`Location`** señala la nueva dirección. La ruta empieza por `/`, por lo que se interpreta desde la raíz del sitio: en nuestro ejemplo, el destino es `http://localhost:8080/unidad-04/acceso.php`.
* **`Set-Cookie`** solicita guardar la preferencia `idioma=es`. `Path=/` establece su ámbito de ruta y `SameSite=Lax` limita su envío en determinados contextos entre sitios. Estudiaremos estas opciones en el apartado de cookies.

**La línea en blanco separa las cabeceras del cuerpo.** El pequeño fragmento HTML es el cuerpo de esta respuesta. Normalmente el navegador seguirá la redirección y ese texto no llegará a verse: realizará otra petición a `acceso.php`.

El `Content-Type` de esta respuesta describe su propio cuerpo, no el contenido que devolverá `acceso.php`. La nueva petición tendrá su propia respuesta. Tampoco es obligatorio que todas las redirecciones incluyan una cookie: la añadimos aquí para observar su dirección y finalidad.

### Enviar cabeceras desde PHP

PHP permite establecer cabeceras de **respuesta** mediante `header()`. No utilizamos esta función para modificar las cabeceras de la petición que ya hemos recibido.

Por ejemplo, podemos indicar que vamos a devolver texto sin formato:

```php
<?php
header('Content-Type: text/plain; charset=UTF-8');

echo '<h1>Hola</h1>';
```

El navegador mostrará literalmente:

```text
<h1>Hola</h1>
```

No lo presentará como un título, porque hemos indicado que el contenido es **texto plano**. Si cambiamos la cabecera por esta:

```php
header('Content-Type: text/html; charset=UTF-8');
```

el navegador interpretará las etiquetas y mostrará **Hola** como un encabezado.

La cabecera tiene dos partes: **`text/html` o `text/plain`** indica el tipo de contenido y **`charset=UTF-8`** indica la codificación del texto.

!!! important "Las cabeceras van antes del cuerpo"
    Llamaremos a `header()` antes de generar HTML o utilizar `echo`. Las cabeceras HTTP no son las etiquetas HTML `<head>`, `<header>` ni `<meta>`. Podemos observarlas en **Red / Network**, seleccionando una petición y consultando **Cabeceras / Headers**.

Una de las cabeceras que acabamos de ver, **`Location`**, nos permite redirigir al navegador. Veamos cómo utilizarla desde PHP.

## 1.5. Redirecciones con header y exit

A veces, después de procesar una petición, queremos que el navegador abra otra página. Por ejemplo, al terminar un formulario podemos llevarlo de vuelta al catálogo.

Una **redirección** es una respuesta del servidor que indica al navegador una nueva dirección:

```php
<?php
header('Location: catalogo.php');
exit;
```

* **`header`** establece una cabecera HTTP de respuesta.
* **`Location: catalogo.php`** indica el destino. En este ejemplo está en la misma carpeta que la página solicitada.
* **`exit`** detiene el script actual. Enviar la cabecera no detiene PHP automáticamente.

El navegador recibe la redirección y realiza **otra petición** a `catalogo.php`. La URL cambia.

### Primero procesar, después mostrar HTML

La llamada a `header` debe colocarse antes de enviar contenido al navegador. Por eso organizaremos el código de procesamiento al principio del archivo y el HTML después.

Este orden es incorrecto:

```php
<p>Volviendo al catálogo...</p>
<?php
header('Location: catalogo.php');
exit;
```

Si las cabeceras ya se han enviado, PHP no podrá modificarlas y puede mostrar el aviso **«headers already sent»**. Lo mismo puede ocurrir por un `echo` o por un archivo incluido que genere salida antes de la redirección. Algunos entornos retrasan la salida y ocultan el problema; no dependeremos de ello.

### Redirigir y cargar un archivo son operaciones distintas

| Instrucción | Qué hace |
| --- | --- |
| `require_once __DIR__ . '/funciones.php';` | PHP carga un archivo en el servidor dentro de la ejecución actual. La URL no cambia. |
| `header('Location: catalogo.php'); exit;` | El servidor pide al navegador que visite otra dirección, provocando una nueva petición. |

`__DIR__` representa una carpeta del sistema de archivos. No lo utilizaremos para construir el destino de una redirección: **una ruta del contenedor no es una dirección web**.

En el siguiente apartado aplicaremos las redirecciones a formularios y aprenderemos por qué resultan útiles después de un envío POST.

## 1.6. Observar las peticiones en el navegador

Podemos comprobar lo anterior desde las herramientas de desarrollo:

1. Abre `peticion.php` a través del servidor.
2. Pulsa **F12** o abre «Inspeccionar» y selecciona **Red / Network**.
3. Pulsa el enlace GET o envía el formulario POST del ejemplo.
4. Selecciona la petición a `peticion.php` y observa su método, dirección y código de estado.
5. Si has enviado el formulario, busca sus campos en **Carga útil / Payload** o **Datos del formulario**.

Los nombres pueden variar según el navegador. Si la lista se vacía al navegar, activa **Conservar registro / Preserve log**.

Lo importante es relacionar la acción realizada con la petición que aparece: **pulsar un enlace y enviar un formulario POST no producen el mismo intercambio**.

## 1.7. Qué debemos recordar

* El navegador envía una petición y el servidor devuelve una respuesta.
* PHP se ejecuta en el servidor; el navegador recibe el resultado generado.
* GET se utiliza para consultar información y POST para enviar datos que deben procesarse, especialmente cuando se realizan cambios.
* `$_SERVER` permite consultar información de la petición. `REQUEST_METHOD` indica el método utilizado.
* Una redirección envía una cabecera `Location`; después utilizamos `exit` para detener el script.
* **Cada petición supone una nueva ejecución.** Las variables de una página no pasan automáticamente a la siguiente, tampoco cuando redirigimos. Más adelante aprenderemos a mantener información mediante cookies y sesiones.

No plantearemos una batería específica de ejercicios para este bloque: practicaremos estos conceptos al construir y procesar los formularios del siguiente apartado.

## Referencias

* [Aitor Medrano y Luis Alemañ: Programación web](https://aitor-medrano.github.io/dwes2122/04web.html).
* [Manual de PHP: `$_SERVER`](https://www.php.net/manual/es/reserved.variables.server.php).
* [Manual de PHP: `$_GET`](https://www.php.net/manual/es/reserved.variables.get.php).
* [Manual de PHP: `header`](https://www.php.net/manual/es/function.header.php).

## Autoría y licencia

Material elaborado por **Joaquín García** para el módulo de **Desarrollo Web en Entorno Servidor** de 2.º DAW del **IES Portada Alta**.

Este material adapta y amplía contenidos de *Desarrollo Web en Entorno Servidor*, elaborados por **Aitor Medrano** y **Luis Alemañ**, publicados bajo licencia **Creative Commons Reconocimiento-NoComercial-CompartirIgual**.

Esta adaptación se distribuye bajo licencia **Creative Commons Reconocimiento-NoComercial-CompartirIgual (CC BY-NC-SA)**.
