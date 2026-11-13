# 4. Cookies

En los formularios hemos devuelto los valores recibidos dentro del HTML para facilitar su corrección. Pero, si cerramos esa página y volvemos más tarde, **las variables PHP de aquella petición ya no existen**.

¿Cómo puede recordar una tienda que preferimos ver el catálogo en formato lista? Una posibilidad es guardar esa preferencia en el navegador mediante una **cookie**.

## 4.1. Qué es una cookie

Una cookie es un pequeño dato que el navegador almacena para un sitio web. En las peticiones posteriores que cumplen sus condiciones de envío, el navegador lo devuelve al servidor.

El servidor puede solicitar su creación mediante una cabecera **`Set-Cookie`**. Cuando corresponde enviarla, el navegador utiliza una cabecera **`Cookie`**. PHP permite consultar lo recibido en **`$_COOKIE`**.

| Operación | Quién interviene | Herramienta en PHP |
| --- | --- | --- |
| Solicitar que se guarde o actualice una cookie. | El servidor envía una instrucción al navegador. | `setcookie()` |
| Almacenar la cookie. | El navegador, si acepta la instrucción. | No es una variable PHP que quede ejecutándose. |
| Enviar las cookies aplicables en otra petición. | El navegador las incorpora a la petición. | PHP las recibe en `$_COOKIE`. |

Las cookies sirven, por ejemplo, para recordar un idioma, un modo de visualización o una preferencia. Se envían también con peticiones POST: **no dependen exclusivamente de GET**.

!!! important "Un dato del navegador"
    El usuario puede borrar o modificar una cookie, y el navegador puede bloquearla. La aplicación debe funcionar con un valor predeterminado cuando no esté disponible y validar lo recibido antes de utilizarlo.

## 4.2. El recorrido entre dos peticiones

Imaginemos una preferencia de catálogo:

1. El usuario elige «Lista» y envía el formulario.
2. PHP valida la elección y llama a `setcookie`.
3. La respuesta pide al navegador guardar `ud04_vista=lista`.
4. En una nueva petición, el navegador envía esa cookie si corresponde.
5. PHP la recoge en `$_COOKIE['ud04_vista']` y prepara la página con esa preferencia.

Representado de forma simplificada en HTTP/1.1:

**Respuesta del servidor que solicita el almacenamiento:**

```http
HTTP/1.1 200 OK
Set-Cookie: ud04_vista=lista; Path=/; SameSite=Lax; HttpOnly
```

**Petición posterior del navegador:**

```http
GET /unidad-04/preferencias.php HTTP/1.1
Host: localhost:8080
Cookie: ud04_vista=lista
```

El segundo intercambio no transporta de nuevo todas las opciones de la cookie. En `Cookie` viaja su nombre y valor; el navegador aplica las condiciones que recibió al guardarla.

Las peticiones son independientes. **La continuidad se consigue porque el navegador vuelve a enviar el dato**, no porque la variable PHP de la primera petición se conserve.

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

```php
$vista = $_COOKIE['ud04_vista'] ?? 'cuadricula';
```

Si falta la cookie, utilizamos el valor predeterminado. Pero también comprobaremos su tipo y las opciones que permite la aplicación:

```php
$permitidas = ['cuadricula', 'lista'];
$recibida = $_COOKIE['ud04_vista'] ?? '';

if (is_string($recibida) && in_array($recibida, $permitidas, true)) {
    $vista = $recibida;
} else {
    $vista = 'cuadricula';
}
```

No utilizaremos la cookie directamente como una ruta, un precio o un permiso. Para nuestra preferencia solo se aceptan dos valores conocidos.

### ¿Por qué no aparece justo después de crearla?

Este fragmento no actualiza automáticamente `$_COOKIE`:

```php
setcookie('ud04_vista', 'lista', ['path' => '/']);
$vista = $_COOKIE['ud04_vista'] ?? 'cuadricula';
```

`$_COOKIE` contiene lo que el navegador envió **al comenzar esta petición**. `setcookie` prepara una instrucción para la **respuesta**. Si antes no había cookie, esta lectura seguirá dando `cuadricula`; si había otra, leerá su valor anterior.

Para mostrar el valor nuevo inmediatamente podríamos usar la variable validada del POST. En el ejemplo completo aplicaremos **POST/Redirect/GET**, para leerlo en la siguiente petición después de que el navegador haya recibido la cookie.

Asignar `$_COOKIE['ud04_vista'] = 'lista';` solo modifica el array de la ejecución actual: **no guarda una cookie en el navegador**.

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

Guarda el archivo como **`preferencias.php`**. Permite elegir una vista, recordarla durante siete días y borrar la preferencia. Mostramos el nombre de la vista; no necesitamos construir todavía un catálogo ni añadir CSS.

```php
<?php
declare(strict_types=1);

function e(string $texto): string
{
    return htmlspecialchars($texto, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}

$vistas = [
    'cuadricula' => 'Cuadrícula',
    'lista' => 'Lista',
];

// HTTP local de clase. En el despliegue mediante HTTPS: secure => true.
$opciones = [
    'expires' => time() + 7 * 24 * 60 * 60,
    'path' => '/',
    'secure' => false,
    'httponly' => true,
    'samesite' => 'Lax',
];

$error = '';
$recibida = $_COOKIE['ud04_vista'] ?? '';
$vista = 'cuadricula';

if (is_string($recibida) && array_key_exists($recibida, $vistas)) {
    $vista = $recibida;
}

if (($_SERVER['REQUEST_METHOD'] ?? '') === 'POST') {
    $accion = $_POST['accion'] ?? '';

    if ($accion === 'guardar') {
        $elegida = $_POST['vista'] ?? '';

        if (!is_string($elegida) || !array_key_exists($elegida, $vistas)) {
            $error = 'Selecciona una vista válida.';
        } elseif (setcookie('ud04_vista', $elegida, $opciones)) {
            header('Location: preferencias.php', true, 303);
            exit;
        } else {
            $error = 'No se ha podido preparar la cookie.';
        }
    } elseif ($accion === 'borrar') {
        $opciones['expires'] = time() - 3600;

        if (setcookie('ud04_vista', '', $opciones)) {
            header('Location: preferencias.php', true, 303);
            exit;
        }
        $error = 'No se ha podido preparar el borrado.';
    } else {
        $error = 'La acción no es válida.';
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
    <p>Vista actual: <strong><?= e($vistas[$vista]) ?></strong></p>

    <?php if ($error !== ''): ?>
        <p role="alert"><?= e($error) ?></p>
    <?php endif; ?>

    <form action="preferencias.php" method="post">
        <label for="vista">Vista del catálogo:</label>
        <select id="vista" name="vista">
            <?php foreach ($vistas as $codigo => $etiqueta): ?>
                <option value="<?= e($codigo) ?>"
                    <?= $vista === $codigo ? 'selected' : '' ?>>
                    <?= e($etiqueta) ?>
                </option>
            <?php endforeach; ?>
        </select>
        <button type="submit" name="accion" value="guardar">Guardar preferencia</button>
    </form>

    <form action="preferencias.php" method="post">
        <button type="submit" name="accion" value="borrar">Borrar preferencia</button>
    </form>
</body>
</html>
```

### El recorrido completo de la petición

En este ejemplo, **la cookie guarda una preferencia de visualización para recuperarla en futuras peticiones**. Vamos a seguir el recorrido de `preferencias.php`, suponiendo que todavía no existe la cookie.

#### 1. Entramos por primera vez

Al escribir la dirección en el navegador, este realiza una petición **GET**:

```http
GET /preferencias.php
```

Todavía no envía nuestra cookie porque no la tiene guardada.

PHP ejecuta el fichero desde el principio:

```php
$recibida = $_COOKIE['ud04_vista'] ?? '';
$vista = 'cuadricula';
```

Como la cookie no existe, `$recibida` vale `''` y se mantiene la opción predeterminada: `cuadricula`.

No se ejecuta el bloque de procesamiento del formulario porque la petición es GET. PHP genera el HTML con la opción **Cuadrícula** seleccionada y lo devuelve al navegador.

**Hasta aquí no hemos creado ninguna cookie:** estamos mostrando un valor predeterminado.

#### 2. Seleccionamos «Lista» y pulsamos «Guardar»

El navegador envía una nueva petición, ahora **POST**, al mismo fichero. Entre sus datos aparecen:

```text
vista=lista
accion=guardar
```

PHP vuelve a ejecutar el fichero **desde el principio**, con variables nuevas. Después entra en el bloque del POST y comprueba que `lista` es una opción permitida.

Si es válida, ejecuta:

```php
setcookie('ud04_vista', $elegida, $opciones);
header('Location: preferencias.php', true, 303);
exit;
```

En este recorrido, `$elegida` vale `'lista'`. El ejemplo completo también comprueba que `setcookie()` haya podido preparar la cabecera antes de redirigir.

Esta respuesta incluye dos instrucciones para el navegador:

```http
HTTP/1.1 303 See Other
Set-Cookie: ud04_vista=lista; ...
Location: preferencias.php
```

- **`Set-Cookie`** le indica que guarde la preferencia.
- **`Location` junto con el estado 303** le indica que solicite `preferencias.php` mediante GET.

El `exit` termina la ejecución. En esta respuesta no necesitamos generar el formulario porque hemos indicado al navegador que haga otra petición.

> `setcookie()` envía la instrucción al navegador. No modifica automáticamente el contenido de `$_COOKIE` durante esa ejecución de PHP.

#### 3. El navegador guarda la cookie y sigue la redirección

Si acepta la cookie, el navegador la guarda y realiza el GET indicado. Ahora la petición incluye:

```http
GET /preferencias.php
Cookie: ud04_vista=lista
```

**Aquí está el paso fundamental: el navegador devuelve al servidor la cookie que había recibido.**

PHP comienza otra ejecución y prepara `$_COOKIE` con las cookies recibidas. Por eso ahora:

```php
$recibida = $_COOKIE['ud04_vista'] ?? '';
```

asigna `'lista'` a `$recibida`.

El código comprueba que es una cadena y que corresponde a una opción del catálogo. Al ser válida:

```php
$vista = $recibida;
```

Por tanto, `$vista` pasa a valer `'lista'`. PHP genera el HTML mostrando **Lista** como preferencia actual y como opción seleccionada.

#### 4. Recargamos o volvemos más tarde

En cada petición a una dirección donde corresponda enviar esa cookie, el navegador vuelve a incluir:

```http
Cookie: ud04_vista=lista
```

PHP la lee, la valida y recupera la preferencia. **No hace falta volver a ejecutar `setcookie()` para leerla**; en nuestro ejemplo solo lo hacemos al guardar o borrar.

Las variables PHP desaparecieron al terminar cada ejecución. Lo que permanece entre peticiones es la cookie almacenada en el navegador.

#### 5. Pulsamos «Borrar preferencia»

El navegador envía un POST con:

```text
accion=borrar
```

PHP responde con una cookie del mismo nombre y ámbito, pero con una fecha de caducidad pasada:

```php
$opciones['expires'] = time() - 3600;
setcookie('ud04_vista', '', $opciones);
```

Eso indica al navegador que la elimine. Después se produce otra redirección 303.

En el nuevo GET, el navegador ya no envía `ud04_vista`. PHP vuelve a utilizar el valor predeterminado: **Cuadrícula**.

#### ¿Qué estamos «siguiendo» en este ejemplo?

Únicamente esta información:

```text
ud04_vista = lista
```

No estamos guardando un historial de visitas ni identificando a una persona. **Estamos recordando una preferencia del navegador.** El nombre de la cookie y su valor viajan entre navegador y servidor; opciones como la caducidad y la ruta determinan cómo la conserva y cuándo la envía el navegador.

La conexión con las sesiones es muy útil: **habitualmente las sesiones también utilizan una cookie**, pero esta guarda un identificador de sesión. Los datos que manejamos con `$_SESSION` se conservan en el servidor. En nuestro ejemplo, en cambio, el propio valor de la preferencia —`lista`— está guardado en la cookie del navegador.

El botón de guardar tiene `name="accion"` y `value="guardar"`: al pulsarlo, su valor se envía y distingue la operación. El formulario de borrado envía `accion=borrar`.

Si el navegador bloquea las cookies, la página volverá a la vista predeterminada. Que se haya preparado la cabecera no confirma su almacenamiento. El ejemplo solo modifica una preferencia de presentación; la protección CSRF de operaciones con estado se explicará al trabajar con sesiones.

## 4.8. Inspeccionar y probar la cookie en Chrome

Abre `preferencias.php` con nuestro servidor, por ejemplo:

```text
http://localhost:8080/unidad-04/preferencias.php
```

Adapta la ruta a la carpeta donde hayas guardado el archivo. Vamos a observar tanto la cookie almacenada como su recorrido en las peticiones y respuestas.

### Ver la cookie guardada: Aplicación

1. Abre `preferencias.php` en Chrome.
2. Pulsa **F12** o **Ctrl + Mayús + I** para abrir las herramientas de desarrollo.
3. En la barra superior de las herramientas, busca **Aplicación** (*Application*). Si no aparece, pulsa **`»`** para ver las pestañas ocultas.
4. En el panel izquierdo, busca **Almacenamiento** (*Storage*) y despliega **Cookies**.
5. Selecciona la dirección de tu servidor, por ejemplo **`http://localhost:8080`**.
6. En el formulario de la página, selecciona **Lista** y pulsa **Guardar preferencia**.
7. Vuelve a la tabla de cookies: debería aparecer **`ud04_vista`**, con el valor **`lista`**. Comprueba también su ruta, caducidad y opciones.

Si todavía no has guardado ninguna preferencia, es normal que no aparezca esta cookie: la primera visita solo utiliza el valor predeterminado.

### Ver lo que viaja: Red

1. Entra en la pestaña **Red** (*Network*) de las herramientas. Si está oculta, búscala también en **`»`**.
2. Marca **Conservar registro** (*Preserve log*) para que las peticiones sigan visibles después de la redirección.
3. Vuelve a guardar una preferencia desde el formulario.
4. Localiza la petición **POST** a `preferencias.php`, con estado **303**. Selecciónala y abre **Cabeceras** (*Headers*).
5. Dentro de **Cabeceras de respuesta** (*Response Headers*), busca **`Set-Cookie`**: es la instrucción que el servidor envía para guardar la preferencia. También verás **`Location`**, que indica el destino de la redirección.
6. Selecciona el **GET** posterior a `preferencias.php`. En **Cabeceras de solicitud** (*Request Headers*), busca **`Cookie`**: contiene la preferencia que el navegador devuelve al servidor.

Si no distingues el POST del GET, consulta **Método de solicitud** (*Request Method*) en la sección **General** de sus cabeceras. Los nombres de algunos controles pueden variar ligeramente según el idioma o la versión de Chrome.

**Aplicación muestra lo que guarda el navegador; Red muestra lo que viaja en cada petición y respuesta.**

Por último, pulsa **Borrar preferencia**. Comprueba en Aplicación que la cookie desaparece y en la página que vuelve **Cuadrícula**. En Red podrás observar la respuesta que solicita el borrado y el GET posterior.

### Pruebas para comprobar el comportamiento

| Prueba | Qué esperamos |
| --- | --- |
| Primera visita sin cookie. | Cuadrícula. |
| Guardar Lista. | Lista tras la redirección. |
| Volver a la misma URL desde otra pestaña del mismo perfil. | Se mantiene Lista. |
| Cerrar la pestaña y volver durante la vigencia. | La preferencia persistente puede seguir disponible. |
| Abrir un perfil privado independiente. | No comparte las cookies del perfil normal. |
| Cambiar el valor de la cookie a `inventada` en las herramientas. | La aplicación utiliza Cuadrícula; no acepta el valor desconocido. |
| Borrar la preferencia. | Desaparece la cookie y se utiliza Cuadrícula. |

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
