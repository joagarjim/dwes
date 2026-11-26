# 6. Autenticación básica con sesiones

Hasta ahora hemos utilizado sesiones para recordar un contador o un nombre. Ahora las utilizaremos para **mantener el acceso a una zona privada después de comprobar un usuario y una contraseña**.

Construiremos una pequeña zona privada del aula con tres archivos en la misma carpeta:

| Archivo | Función |
| --- | --- |
| `login.php` | Muestra el formulario y comprueba las credenciales. |
| `privada.php` | Muestra contenido solo si se ha iniciado sesión correctamente. |
| `logout.php` | Cierra la sesión y devuelve al formulario de acceso. |

Usaremos PHP local, sin JavaScript ni base de datos. GitHub Pages muestra los apuntes, pero no ejecuta estos archivos PHP. El código se centra en el funcionamiento de la autenticación; la seguridad avanzada se ampliará en la unidad 7.

## 6.1. Tener una sesión no significa estar autenticado

En el apartado anterior escribíamos:

```php
session_name('UD04DEMO');
session_start();
```

Esto inicia o recupera una sesión. **No comprueba quién está utilizando la aplicación.** También guardábamos un nombre recibido:

```php
$_SESSION['nombre'] = $_POST['nombre'];
```

Pero cualquiera puede escribir Ana. Guardar ese nombre no demuestra que la persona sea Ana.

### Qué significa autenticarse

Autenticarse consiste en **comprobar unas credenciales para verificar la identidad que alguien presenta**. Aquí serán un usuario y una contraseña. PHP comprobará que correspondan al usuario de prueba definido en la aplicación.

| Situación | Resultado |
| --- | --- |
| Usuario y contraseña correctos. | Permite iniciar sesión y acceder a la página privada. |
| Usuario o contraseña incorrectos, sin acceso previo. | Muestra un error y no autoriza el acceso. |

Después de comprobarlas guardaremos:

```php
$_SESSION['usuario'] = 'ana';
```

**Nuestra aplicación realizará esta asignación únicamente después de comprobar las credenciales.** La sesión permite recordar el resultado de esa comprobación entre peticiones.

| En el navegador | En el servidor |
| --- | --- |
| Cookie `UD04ACCESO=xyz789`. | Sesión identificada por `xyz789`, con `usuario = ana`. |

La contraseña no se guarda en la sesión ni necesita enviarse de nuevo en cada visita a la página privada.

### Cómo se protege una página

Cada página privada debe recuperar la sesión y comprobar el acceso antes de enviar su contenido:

```php
session_name('UD04ACCESO');
session_start();

if (!isset($_SESSION['usuario'])) {
    header('Location: login.php', true, 303);
    exit;
}
```

Si no existe `usuario`, redirige y termina. Si existe, continúa hasta el HTML. **Ocultar un enlace no protege una página:** alguien puede escribir su dirección directamente.

Autenticar es comprobar las credenciales. Autorizar es decidir si el acceso está permitido. En este ejemplo todos los usuarios autenticados podrán entrar en la única zona privada; no hay roles distintos.

## 6.2. Formulario de acceso y comprobación de credenciales

### 1. Preparar el usuario de prueba y su contraseña

Utilizaremos estas credenciales solo para clase:

```text
Usuario: ana
Contraseña: clave123
```

Como todavía no utilizamos una base de datos, definiremos el usuario y el hash de su contraseña en **`login.php`**:

```php
$usuarioCorrecto = 'ana';
$hashCorrecto = 'AQUI_EL_HASH_GENERADO_PARA_SU_CONTRASEÑA';
```

**Nosotros establecemos la relación entre Ana y ese hash.** El hash no contiene el nombre de Ana ni permite saber a qué usuario pertenece. Cuando utilicemos una base de datos, cada registro relacionará al usuario con el hash de su contraseña.

#### Qué es el hash y cuándo se genera

El hash es un resultado que obtenemos a partir de una contraseña y que sirve como referencia para comprobarla posteriormente. Guardamos ese resultado en lugar de guardar la contraseña como texto. No es una contraseña cifrada que después desciframos.

| Función | Cuándo la utilizamos | Para qué sirve |
| --- | --- | --- |
| `password_hash()` | Al establecer o cambiar una contraseña. En este ejemplo, al prepararlo una vez. | Genera el hash que guardaremos. |
| `password_verify()` | Cada vez que se intenta iniciar sesión. | Comprueba si la contraseña escrita corresponde al hash guardado. Devuelve `true` o `false`. |

**No generamos un hash nuevo en cada intento de acceso.** En nuestro ejemplo generaremos uno, lo pegaremos en `login.php` y lo reutilizaremos para las comprobaciones.

#### Generarlo desde PowerShell con Docker

En nuestro entorno PHP está instalado dentro del contenedor `apache_php`. Desde PowerShell, abre la carpeta del proyecto que contiene `compose.yaml` o `docker-compose.yml`, con los contenedores arrancados.

**1. Entra en el contenedor:**

```powershell
docker compose exec apache_php sh
```

**2. Ya dentro del contenedor, genera el hash:**

```bash
php -r 'echo password_hash("clave123", PASSWORD_DEFAULT), PHP_EOL;'
```

`php -r` ejecuta el código escrito a continuación sin crear un archivo PHP. `password_hash()` genera el hash y `echo` lo muestra en el terminal. `PHP_EOL` añade un salto de línea. `PASSWORD_DEFAULT` indica que PHP utilice su algoritmo predeterminado para contraseñas.

Entramos primero en el contenedor para evitar los problemas que pueden producirse con las comillas al enviar este comando directamente desde PowerShell.

**3. Copia el resultado completo y pégalo en la variable `$hashCorrecto` de `login.php`, entre comillas simples:**

```php
$usuarioCorrecto = 'ana';
$hashCorrecto = 'PEGA_AQUI_EL_RESULTADO_COMPLETO';
```

Las comillas simples conservan literalmente los caracteres del hash, incluidos los signos `$`. Sustituye el texto de ejemplo por el resultado real.

**4. Sal del contenedor para volver a PowerShell:**

```bash
exit
```

El comando solo muestra el hash: no crea ningún archivo ni modifica automáticamente `login.php`. **El archivo completo del siguiente punto ya incluye un hash de `clave123`**, por lo que puedes probarlo directamente o sustituirlo por el tuyo.

#### Por qué puede salir un hash diferente para la misma contraseña

`password_hash()` genera una **sal aleatoria** cada vez que lo llamamos. Por eso dos hashes de `clave123` pueden ser distintos y ambos ser válidos.

La sal y la información necesaria para comprobar la contraseña están incluidas en el resultado. No tenemos que crear ni guardar una sal aparte.

Al ejecutar:

```php
password_verify($clave, $hashCorrecto)
```

PHP utiliza la información del hash guardado para comprobar la contraseña recibida. **No genera otra sal aleatoria para esta comprobación y no descifra el hash.** Tampoco debemos generar un hash nuevo y compararlo con el anterior mediante `===`. No utilizaremos MD5 ni SHA-1 para almacenar contraseñas.

#### Qué ocurre cuando alguien envía el formulario

1. Recibimos el usuario y la contraseña escritos en el formulario.
2. Comprobamos que el usuario sea `ana`.
3. Comprobamos su contraseña con `password_verify($clave, $hashCorrecto)`.
4. Si ambas comprobaciones son correctas, renovamos el identificador de sesión y guardamos:

   ```php
   $_SESSION['usuario'] = $usuarioCorrecto;
   ```

**El hash permite comprobar la contraseña. La sesión permite recordar que ese navegador ha iniciado sesión como Ana.** No guardamos la contraseña ni su hash en `$_SESSION`.

Cuando utilicemos una base de datos, primero buscaremos el registro del usuario indicado y después comprobaremos la contraseña con el hash de ese registro. El funcionamiento de `password_verify()` será el mismo.

### 2. El archivo completo: login.php

```php
<?php
session_name('UD04ACCESO');
session_start();

$usuarioCorrecto = 'ana';
$hashCorrecto = '$2y$12$D8aQJJi9mtCLztixV0FY.eODOLnQcw6bX6h9R2i6UAAoSyOAgXZTi';
$error = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $usuario = $_POST['usuario'] ?? '';
    $clave = $_POST['clave'] ?? '';

    if (
        $usuario === $usuarioCorrecto
        && password_verify($clave, $hashCorrecto)
    ) {
        session_regenerate_id(true);
        $_SESSION['usuario'] = $usuarioCorrecto;

        header('Location: privada.php', true, 303);
        exit;
    }

    $error = 'Usuario o contraseña incorrectos.';
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Acceso al aula</title>
</head>
<body>
    <h1>Acceso al aula</h1>
    <?php if ($error !== ''): ?>
        <p><?= $error ?></p>
    <?php endif; ?>

    <form action="login.php" method="post">
        <p>
            <label for="usuario">Usuario:</label>
            <input type="text" id="usuario" name="usuario" required>
        </p>
        <p>
            <label for="clave">Contraseña:</label>
            <input type="password" id="clave" name="clave" required>
        </p>
        <button type="submit">Entrar</button>
    </form>
</body>
</html>
```

`action="login.php"` envía el formulario al mismo archivo. `method="post"` envía sus campos en el cuerpo. Los nombres `usuario` y `clave` determinan las claves de `$_POST`.

`type="password"` oculta visualmente los caracteres, pero no cifra la transmisión. **POST tampoco cifra los datos: al publicar la aplicación utilizaremos HTTPS.**

### 3. Primera solicitud: abrir el formulario

Suponemos que todavía no tenemos la cookie `UD04ACCESO`:

```http
GET /login.php HTTP/1.1
Host: localhost:8080

```

PHP ejecuta `session_name()` y `session_start()`. Crea una sesión y prepara una cookie cuyo identificador representaremos como `abc123`. Como es GET, se salta el procesamiento POST y genera el formulario.

```http
HTTP/1.1 200 OK
Set-Cookie: UD04ACCESO=abc123; Path=/
Content-Type: text/html; charset=UTF-8

<h1>Acceso al aula</h1>
<!-- Formulario de usuario, contraseña y botón Entrar -->
```

El navegador guarda la cookie si la acepta y muestra la página. **Ya existe una sesión, pero todavía no estamos autenticados:** no hemos comprobado las credenciales ni creado `$_SESSION['usuario']`.

### 4. Segunda solicitud: enviar las credenciales

Escribimos `ana` y `clave123` y pulsamos Entrar:

```http
POST /login.php HTTP/1.1
Host: localhost:8080
Cookie: UD04ACCESO=abc123
Content-Type: application/x-www-form-urlencoded
Content-Length: 26

usuario=ana&clave=clave123
```

| Lugar | Datos |
| --- | --- |
| Cabecera Cookie. | Identificador de la sesión: `abc123`. |
| Cuerpo del POST. | Usuario y contraseña del formulario. |

PHP vuelve a ejecutar `login.php` desde el principio. `session_start()` recupera la sesión y el bloque POST recoge los campos:

```php
$usuario = $_POST['usuario'] ?? '';
$clave = $_POST['clave'] ?? '';
```

En este ejemplo recogemos los campos de texto del formulario. La condición comprueba primero que el usuario sea `ana`. Si no lo es, `&&` deja de evaluar y no se comprueba la contraseña. Si lo es, exige además que:

```php
password_verify($clave, $hashCorrecto)
```

devuelva `true`. Esta función no descifra el hash: comprueba la correspondencia.

### 5. Renovar el identificador al autenticarse

Con credenciales correctas ejecutamos:

```php
session_regenerate_id(true);
```

Antes de autenticarse el visitante utilizaba `abc123`. PHP genera otro identificador, por ejemplo `xyz789`, y prepara una cabecera para actualizar la cookie.

| Antes | Después |
| --- | --- |
| `UD04ACCESO=abc123` | `UD04ACCESO=xyz789` |

**El nombre de la cookie no cambia; cambia su valor. Los datos actuales de $_SESSION se conservan.** Si había otros datos, como un carrito, no se borran por regenerar el identificador.

**`session_regenerate_id(true)` cambia el identificador de la sesión conservando su contenido. El argumento `true` indica que se elimine del servidor el almacenamiento correspondiente al identificador anterior.**

¿Por qué cambiarlo? Para que el acceso autenticado no continúe usando el identificador del visitante anónimo. Esto ayuda a evitar la **fijación de sesión**, donde un atacante intenta que la víctima use un identificador que él ya conoce.

En esta demostración lo hacemos al iniciar sesión correctamente, no en cada recarga. Trabajamos con peticiones secuenciales; la gestión de regeneración con peticiones simultáneas y conexiones inestables se ampliará en seguridad.

Después guardamos:

```php
$_SESSION['usuario'] = $usuarioCorrecto;
```

La sesión `xyz789` contiene ahora `usuario = ana`. **No guardamos la contraseña en $_SESSION.**

### 6. Respuesta al acceso correcto

Estas instrucciones están dentro del bloque que acepta las credenciales:

```php
header('Location: privada.php', true, 303);
exit;
```

PHP responde:

```http
HTTP/1.1 303 See Other
Set-Cookie: UD04ACCESO=xyz789; Path=/
Location: privada.php

```

El navegador actualiza la cookie y sigue la redirección mediante GET. `exit` termina este POST sin generar el HTML del formulario. Los datos de sesión se conservan al finalizar normalmente la petición.

### 7. Si las credenciales son incorrectas

Partimos de una sesión sin autenticar. PHP no registra el usuario ni renueva el identificador. Ejecuta:

```php
$error = 'Usuario o contraseña incorrectos.';
```

Continúa hasta el HTML y devuelve un 200 con el formulario y el mensaje:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

<h1>Acceso al aula</h1>
<p>Usuario o contraseña incorrectos.</p>
<!-- Formulario; la contraseña no se vuelve a escribir en el HTML -->
```

El mensaje no revela si falló el usuario o la contraseña. **Un 200 significa que se atendió la petición; no significa que las credenciales sean correctas.**

En este ejemplo un intento incorrecto no elimina un acceso que ya existiera en la sesión. Para probar el rechazo desde cero, cierra sesión o utiliza un perfil sin autenticar.

## 6.3. Una página protegida

No basta con llegar desde `login.php`: **privada.php debe comprobar por sí misma el acceso**, porque alguien podría escribir su dirección directamente.

### 1. El archivo completo: privada.php

```php
<?php
session_name('UD04ACCESO');
session_start();

if (!isset($_SESSION['usuario'])) {
    header('Location: login.php', true, 303);
    exit;
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Zona privada del aula</title>
</head>
<body>
    <h1>Zona privada del aula</h1>
    <p>Has iniciado sesión como
        <strong><?= htmlspecialchars($_SESSION['usuario']) ?></strong>.
    </p>
    <p>Este contenido solo se muestra después de comprobar el acceso.</p>

    <form action="logout.php" method="post">
        <button type="submit">Cerrar sesión</button>
    </form>
</body>
</html>
```

La comprobación va antes del HTML. El usuario mostrado procede del dato que nuestra aplicación guardó después de validar las credenciales.

### 2. Solicitud después del acceso correcto

El navegador sigue la redirección y envía el identificador nuevo:

```http
GET /privada.php HTTP/1.1
Host: localhost:8080
Cookie: UD04ACCESO=xyz789

```

No envía otra vez usuario y contraseña. PHP ejecuta:

```php
session_name('UD04ACCESO');
session_start();
```

Recupera la sesión `xyz789`, que contiene `$_SESSION['usuario'] = 'ana'`. Después comprueba:

```php
if (!isset($_SESSION['usuario'])) {
    header('Location: login.php', true, 303);
    exit;
}
```

La condición pregunta si falta `usuario`. Como existe, es falsa y se salta la redirección. PHP genera el contenido privado:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

<h1>Zona privada del aula</h1>
<p>Has iniciado sesión como <strong>ana</strong>.</p>
<!-- Contenido privado y formulario para cerrar sesión -->
```

### 3. Recargar la página privada

Cada recarga es otro GET. PHP vuelve a ejecutar todo el archivo, recupera la sesión y comprueba el usuario. **La comprobación se hace en cada petición**, no solamente al entrar por primera vez.

### 4. Intentar entrar sin autenticarse

En una ventana de incógnito sin sesión previa, abre directamente `privada.php`:

```http
GET /privada.php HTTP/1.1
Host: localhost:8080

```

`session_start()` inicia una sesión nueva, pero no contiene un usuario autenticado. La condición se cumple y responde:

```http
HTTP/1.1 303 See Other
Set-Cookie: UD04ACCESO=nuevo456; Path=/
Location: login.php

```

**exit impide llegar al HTML privado: ese contenido no se envía.** El navegador sigue la redirección:

```http
GET /login.php HTTP/1.1
Host: localhost:8080
Cookie: UD04ACCESO=nuevo456

```

El servidor devuelve un 200 con el formulario.

| Situación | Resultado |
| --- | --- |
| Sin sesión previa. | Inicia una sesión y redirige al formulario. |
| Con sesión, pero sin `usuario`. | Redirige al formulario. |
| Con el usuario registrado tras comprobar las credenciales. | Muestra el contenido privado. |

**Tener una cookie no basta para estar autenticado.** La aplicación comprueba los datos de la sesión del servidor.

## 6.4. Cerrar sesión

Cerrar sesión debe conseguir que **una nueva petición a privada.php ya no permita entrar**.

### 1. El botón en privada.php

El archivo completo ya incluye este formulario:

```html
<form action="logout.php" method="post">
    <button type="submit">Cerrar sesión</button>
</form>
```

Envía un POST a `logout.php`. No envía usuario ni contraseña: la cookie identifica la sesión que vamos a cerrar.

### 2. El archivo completo: logout.php

```php
<?php
session_name('UD04ACCESO');
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $_SESSION = [];
    session_destroy();
    setcookie('UD04ACCESO', '', time() - 3600, '/');
}

header('Location: login.php', true, 303);
exit;
```

| Instrucción | Qué hace |
| --- | --- |
| `$_SESSION = [];` | Vacía el array de la ejecución actual. |
| `session_destroy();` | Elimina los datos almacenados de esta sesión en el servidor. |
| `setcookie()` con caducidad pasada. | Solicita al navegador borrar la cookie del identificador. |

Suponemos que la cookie se creó con **ruta / y sin dominio explícito**, como en el entorno del ejemplo. Al borrarla debemos mantener nombre, dominio y ruta; comprueba su ámbito en Aplicación > Cookies.

Si abrimos `logout.php` mediante GET, el código no cierra la sesión: simplemente redirige al formulario. La operación de cierre se realiza mediante POST.

### 3. Solicitud para cerrar

```http
POST /logout.php HTTP/1.1
Host: localhost:8080
Cookie: UD04ACCESO=xyz789
Content-Length: 0

```

No hay campos con nombre en el formulario; el destino identifica la operación y la cookie identifica la sesión. PHP la recupera, vacía sus datos, la destruye y prepara el borrado de la cookie.

### 4. Respuesta de cierre

```http
HTTP/1.1 303 See Other
Set-Cookie: UD04ACCESO=deleted; Max-Age=0; Path=/
Location: login.php

```

`Set-Cookie` solicita borrar la cookie y `Location` solicita el GET al formulario. Puede aparecer también `Expires` con fecha pasada. `deleted` es un valor que PHP puede utilizar para el borrado, no una nueva identidad.

### 5. Volver al formulario

El navegador procesa el borrado y solicita:

```http
GET /login.php HTTP/1.1
Host: localhost:8080

```

`login.php` vuelve a ejecutar `session_start()`. Como no recibe la cookie anterior, crea una sesión anónima nueva:

```http
HTTP/1.1 200 OK
Set-Cookie: UD04ACCESO=nuevo456; Path=/
Content-Type: text/html; charset=UTF-8

<h1>Acceso al aula</h1>
<!-- Formulario de usuario y contraseña -->
```

**Que aparezca otra cookie no significa que sigamos autenticados.** La sesión nueva no contiene `$_SESSION['usuario']`.

### 6. Comprobar que el acceso se cerró

Escribe de nuevo la dirección de `privada.php`:

```http
GET /privada.php HTTP/1.1
Host: localhost:8080
Cookie: UD04ACCESO=nuevo456

```

La sesión no contiene usuario; PHP responde con 303 y `Location: login.php`. El contenido privado no se envía. Necesitamos credenciales correctas para entrar otra vez.

## 6.5. Ejemplo completo: todos los archivos

Guarda los tres archivos en la misma carpeta. El hash incluido corresponde a `clave123`; puedes sustituirlo por el tuyo generado con `password_hash()`.

| Archivo | Función |
| --- | --- |
| `login.php` | Formulario, comprobación y registro del usuario en la sesión. |
| `privada.php` | Comprobación antes de mostrar HTML privado. |
| `logout.php` | Cierre completo y vuelta al formulario. |

### login.php

```php
<?php
session_name('UD04ACCESO');
session_start();

$usuarioCorrecto = 'ana';
$hashCorrecto = '$2y$12$D8aQJJi9mtCLztixV0FY.eODOLnQcw6bX6h9R2i6UAAoSyOAgXZTi';
$error = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $usuario = $_POST['usuario'] ?? '';
    $clave = $_POST['clave'] ?? '';

    if (
        $usuario === $usuarioCorrecto
        && password_verify($clave, $hashCorrecto)
    ) {
        session_regenerate_id(true);
        $_SESSION['usuario'] = $usuarioCorrecto;

        header('Location: privada.php', true, 303);
        exit;
    }

    $error = 'Usuario o contraseña incorrectos.';
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Acceso al aula</title>
</head>
<body>
    <h1>Acceso al aula</h1>
    <?php if ($error !== ''): ?>
        <p><?= $error ?></p>
    <?php endif; ?>

    <form action="login.php" method="post">
        <p>
            <label for="usuario">Usuario:</label>
            <input type="text" id="usuario" name="usuario" required>
        </p>
        <p>
            <label for="clave">Contraseña:</label>
            <input type="password" id="clave" name="clave" required>
        </p>
        <button type="submit">Entrar</button>
    </form>
</body>
</html>
```

### privada.php

```php
<?php
session_name('UD04ACCESO');
session_start();

if (!isset($_SESSION['usuario'])) {
    header('Location: login.php', true, 303);
    exit;
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Zona privada del aula</title>
</head>
<body>
    <h1>Zona privada del aula</h1>
    <p>Has iniciado sesión como
        <strong><?= htmlspecialchars($_SESSION['usuario']) ?></strong>.
    </p>
    <p>Este contenido solo se muestra después de comprobar el acceso.</p>

    <form action="logout.php" method="post">
        <button type="submit">Cerrar sesión</button>
    </form>
</body>
</html>
```

### logout.php

```php
<?php
session_name('UD04ACCESO');
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $_SESSION = [];
    session_destroy();
    setcookie('UD04ACCESO', '', time() - 3600, '/');
}

header('Location: login.php', true, 303);
exit;
```

Abre `login.php` desde el servidor PHP, entra con `ana` y `clave123`, cierra sesión e intenta acceder directamente a `privada.php`. La página debe devolverte al formulario.

## 6.6. Resumen del recorrido

| Acción | Solicitud | Qué hace PHP | Respuesta |
| --- | --- | --- | --- |
| Abrir el formulario. | GET a login.php. | Inicia o recupera sesión. | 200 con HTML; Set-Cookie si debe establecerla. |
| Credenciales correctas. | POST a login.php con campos y cookie. | Comprueba, renueva identificador y guarda usuario. | 303 con Set-Cookie y Location: privada.php. |
| Seguir redirección. | GET a privada.php con identificador nuevo. | Recupera sesión y comprueba usuario. | 200 con contenido privado. |
| Credenciales incorrectas desde sesión anónima. | POST a login.php. | No registra un acceso nuevo. | 200 con formulario y error. |
| Acceso directo sin autenticar. | GET a privada.php. | Detiene la ejecución antes del HTML privado. | 303 a login.php. |
| Cerrar sesión. | POST a logout.php. | Vacía datos, destruye sesión y solicita borrar cookie. | 303 a login.php con borrado. |
| Volver al formulario. | GET a login.php sin cookie anterior. | Crea sesión anónima nueva. | 200 con formulario y cookie nueva. |

El navegador conserva el identificador; el servidor conserva el usuario autenticado. Cada petición ejecuta únicamente el archivo solicitado desde el principio.

## 6.7. Comprobar el funcionamiento en Chrome

### Preparar la prueba

1. Abre el ejemplo en localhost y pulsa F12.
2. En Aplicación > Cookies selecciona tu servidor. Si necesitas empezar desde cero, elimina solo `UD04ACCESO`.
3. En Red activa **Keep log / Conservar registro**.

### Observar el acceso correcto

1. Abre `login.php` y anota mentalmente que existe una cookie de sesión anónima. No compartas su valor real.
2. Entra con `ana` y `clave123`.
3. En Red selecciona el POST a login.php. En Payload aparecen `usuario` y `clave` (son credenciales de prueba). No compartas capturas con contraseñas reales.
4. En las cabeceras de respuesta localiza el 303, `Location: privada.php` y `Set-Cookie` con el identificador nuevo.
5. Selecciona el GET a privada.php: envía el identificador nuevo en Cookie. No vuelve a enviar las credenciales.
6. En Aplicación comprueba que el valor de la cookie ha cambiado, pero su nombre sigue siendo `UD04ACCESO`.

### Observar el rechazo y el cierre

1. Cierra sesión y prueba una contraseña incorrecta. El POST devuelve 200 con el formulario y el mensaje, sin un acceso nuevo.
2. Entra correctamente y pulsa Cerrar sesión. El POST a logout.php devuelve 303 y solicita borrar la cookie.
3. El GET a login.php devuelve el formulario y crea otra sesión, todavía sin usuario.
4. Escribe directamente la dirección de privada.php. Debe responder con 303 y devolverte al formulario.

| Prueba | Resultado esperado |
| --- | --- |
| Primera visita a login.php. | Formulario sin errores; sesión anónima. |
| Usuario desconocido o contraseña incorrecta desde sesión anónima. | Mensaje genérico; no permite acceso privado. |
| Credenciales correctas. | Renueva identificador y muestra la página privada. |
| Recargar privada.php. | Sigue comprobando la sesión y permite el acceso autorizado. |
| Entrar directamente en privada.php sin autenticar. | Redirige al formulario sin enviar el HTML privado. |
| Otra pestaña del mismo perfil. | Comparte normalmente la sesión y el acceso. |
| Incógnito sin sesión previa. | No comparte el acceso del perfil normal. |
| Cerrar sesión. | Elimina el acceso anterior. Una nueva petición privada redirige al formulario. |
| Otra pestaña abierta antes del cierre. | Puede conservar el HTML ya mostrado; al recargar deja de acceder. |

El botón Atrás puede mostrar una copia de una página que ya fue recibida. Para comprobar la protección, realiza una **nueva solicitud** escribiendo la dirección o recargando y observa Red. El servidor controla lo que envía ahora; no puede retirar un HTML que el navegador ya recibió.

## 6.8. Ideas clave

* **Tener una sesión no significa estar autenticado.** Nuestra aplicación registra el usuario solo después de comprobar sus credenciales.
* **El hash se genera con password_hash() y la contraseña se comprueba con password_verify().** No guardamos la contraseña en cookies ni en la sesión.
* **Renovamos el identificador al autenticar mediante session_regenerate_id(true).** Los datos actuales se conservan; se retira el almacenamiento bajo el identificador anterior.
* **Cada página protegida comprueba la sesión antes del HTML.** Ocultar enlaces no impide entrar por una dirección directa.
* **Cerrar sesión elimina el acceso del servidor y solicita borrar la cookie.** La nueva sesión anónima que pueda crearse después no acredita un acceso.

Este ejemplo es una base didáctica local. En despliegue utilizaremos HTTPS y configuraremos las cookies de sesión adecuadamente; en la unidad 7 ampliaremos CSRF, límites de intentos, caducidad y protección de sesiones. Utilizar POST por sí solo no constituye una protección CSRF completa.

## Practica lo aprendido

Realiza el **ejercicio 8 del bloque 6: Autenticación**, «Zona privada del aula con dos usuarios», de la [relación de actividades](actividades.md#bloque-6-autenticacion). El enunciado y la solución se mantienen separados.

## Referencias

* [Manual de PHP: password_hash](https://www.php.net/manual/es/function.password-hash.php).
* [Manual de PHP: password_verify](https://www.php.net/manual/es/function.password-verify.php).
* [Manual de PHP: session_regenerate_id](https://www.php.net/manual/es/function.session-regenerate-id.php).
* [Manual de PHP: session_destroy](https://www.php.net/manual/es/function.session-destroy.php).

Los mensajes HTTP están simplificados en HTTP/1.1, con HTML abreviado y cabeceras adicionales omitidas. Las longitudes mostradas corresponden al cuerpo del ejemplo, y los identificadores son ilustrativos. Suponemos que el navegador acepta y procesa las cookies.

## Autoría y licencia

Material elaborado por **Joaquín García**, DWES de 2.º DAW del **IES Portada Alta**. Continúa la adaptación didáctica de los materiales de Aitor Medrano y Luis Alemañ y se contrasta con el manual de PHP. **CC BY-NC-SA**.
