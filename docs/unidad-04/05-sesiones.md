# 5. Sesiones

### ¿Qué es una sesión y para qué sirve?

Cuando navegas por una tienda, puedes añadir un producto al carrito y después visitar otras páginas. Aunque cada página se solicita mediante una petición distinta, la aplicación necesita recordar qué productos has añadido.

Como hemos visto, **las variables normales de PHP no se conservan entre peticiones**. Para mantener información mientras un usuario utiliza la aplicación, podemos emplear **sesiones**.

Una sesión permite **guardar datos en el servidor y recuperarlos en peticiones posteriores asociadas al mismo navegador**. En PHP trabajamos con esos datos mediante el array `$_SESSION`.

Las sesiones se utilizan, por ejemplo, para:

* Mantener los productos y cantidades de un carrito.
* Recordar que un usuario ha iniciado sesión.
* Conservar información durante un proceso de varios pasos, como una reserva.

**¿Cómo sabe el servidor qué datos debe recuperar?** Habitualmente, el navegador conserva una cookie con un **identificador de sesión** y lo envía en las siguientes peticiones. PHP utiliza ese identificador para localizar los datos correspondientes.

| En el navegador | En el servidor |
| --- | --- |
| Una cookie con el identificador de sesión. | Los datos asociados: carrito, usuario identificado o información temporal. |

Por tanto, seguimos utilizando una cookie, pero **en ella viaja el identificador; los datos de la sesión permanecen en el servidor**.

Una sesión puede existir aunque el visitante no haya iniciado sesión con un usuario y una contraseña. Por ejemplo, una tienda puede mantener el carrito de un visitante anónimo.

### ¿Qué vamos a construir?

Vamos a crear una página que **recuerde un nombre y cuente sus visitas**. La construiremos paso a paso para comprobar cómo se guardan, recuperan y eliminan los datos de una sesión.

Después abriremos una segunda página para comprobar que puede recuperar la misma información y utilizaremos las herramientas del navegador para observar qué viaja entre el cliente y el servidor.

Para cerrar el apartado, aplicarás lo aprendido en **un pequeño carrito de compra con sesiones**.

**Trabaja en el servidor PHP local de Docker. GitHub Pages muestra estos apuntes, pero no ejecuta PHP.** Crea una carpeta `sesiones` para los ejemplos. Necesitas la extensión `mbstring`, que ya utilizamos al validar formularios.

## 5.1. Primera prueba: una página con memoria

Queremos comprobar qué sucede con dos contadores cuando recargamos una página:

* Uno guardado en **`$_SESSION`**.
* Otro guardado en una **variable normal de PHP**.

### 1. Preparamos el archivo

Guarda este código en **`sesiones.php`**:

```php
<?php
declare(strict_types=1);

session_name('UD04DEMO');
session_start();

if (!isset($_SESSION['visitas'])) {
    $_SESSION['visitas'] = 0;
}

$_SESSION['visitas']++;

$contadorNormal = 0;
$contadorNormal++;
?>
<h1>Una página con memoria</h1>
<p>Contador de sesión: <?= $_SESSION['visitas'] ?></p>
<p>Contador normal: <?= $contadorNormal ?></p>
```

Lo abriremos mediante el servidor PHP local. Si está en la raíz pública, su dirección será:

```text
http://localhost:8080/sesiones.php
```

Si lo has colocado en una subcarpeta, añade su ruta a la dirección. Para seguir el ejemplo, **partimos sin una cookie llamada `UD04DEMO`**. Puedes eliminar únicamente esa cookie desde Aplicación > Cookies antes de empezar.

### 2. Primera solicitud: abrimos la página

El navegador envía una petición como esta, representada en HTTP/1.1:

```http
GET /sesiones.php HTTP/1.1
Host: localhost:8080

```

| Parte | Qué significa |
| --- | --- |
| `GET` | Solicita obtener la página. |
| `/sesiones.php` | Ruta del archivo solicitado. |
| `HTTP/1.1` | Versión de HTTP utilizada en esta representación. |
| `Host: localhost:8080` | Servidor al que se dirige la petición. |

La primera línea es la **línea de petición**; después aparecen las cabeceras. Este GET no lleva cuerpo. Tampoco incluye nuestra cookie, porque todavía no existe.

### 3. PHP ejecuta el archivo desde el principio

El servidor recibe la solicitud y ejecuta `sesiones.php`.

```php
declare(strict_types=1);
```

Activa el modo estricto para las declaraciones de tipos de PHP. No interviene en la creación de la sesión ni en el contador.

Después:

```php
session_name('UD04DEMO');
```

Elegimos **el nombre de la cookie que PHP utilizará para identificar la sesión**. Esta instrucción no crea la sesión ni guarda el contador.

A continuación:

```php
session_start();
```

PHP intenta recuperar una sesión mediante el identificador recibido. Como en esta primera petición no llega nuestra cookie, **crea una sesión nueva y genera su identificador**.

Imaginemos que ese identificador es `abc123`. Es solo un valor ilustrativo: el identificador real lo genera PHP.

En nuestro entorno, PHP utiliza cookies para transportar el identificador. `session_start()` prepara automáticamente una cabecera para solicitar al navegador que guarde:

```text
UD04DEMO=abc123
```

| Parte de la cookie | Quién la decide |
| --- | --- |
| Nombre: `UD04DEMO`. | Nosotros, mediante `session_name()`. |
| Valor: `abc123`. | PHP al generar el identificador de sesión. |

No tenemos que llamar a `setcookie()` para crear esta cookie: PHP gestiona su creación al iniciar la sesión. **Todavía no hemos guardado ninguna visita.** Hemos iniciado la sesión y podemos trabajar con sus datos mediante el array `$_SESSION`.

### 4. PHP crea e incrementa el contador de sesión

La siguiente instrucción es:

```php
if (!isset($_SESSION['visitas'])) {
    $_SESSION['visitas'] = 0;
}
```

La leemos así:

> Si todavía no existe un dato llamado `visitas` en la sesión, créalo con valor cero.

Como la sesión es nueva, ese dato no existe. Por eso entra en el `if` y ejecuta:

```php
$_SESSION['visitas'] = 0;
```

Después:

```php
$_SESSION['visitas']++;
```

Suma uno. Ahora tenemos:

```php
$_SESSION['visitas'] // Vale 1.
```

**Nosotros hemos elegido la clave `visitas` y el valor que guardamos.** PHP conservará ese dato en el servidor, asociado a la sesión. Podemos añadir otras claves y valores a `$_SESSION`: no necesitamos una cookie por cada dato.

### 5. PHP crea e incrementa la variable normal

```php
$contadorNormal = 0;
$contadorNormal++;
```

Creamos una variable normal con valor cero y le sumamos uno. Ahora `$contadorNormal` también vale **1**.

Estas instrucciones no utilizan `$_SESSION`: su valor no se conserva automáticamente para la siguiente petición.

### 6. Primera respuesta: el navegador recibe la cookie y el HTML

PHP llega a este fragmento:

```php
<h1>Una página con memoria</h1>
<p>Contador de sesión: <?= $_SESSION['visitas'] ?></p>
<p>Contador normal: <?= $contadorNormal ?></p>
```

Las expresiones `<?= ... ?>` escriben los valores en el HTML. El servidor devuelve una respuesta como esta, simplificada:

```http
HTTP/1.1 200 OK
Set-Cookie: UD04DEMO=abc123; Path=/
Content-Type: text/html; charset=UTF-8

<h1>Una página con memoria</h1>
<p>Contador de sesión: 1</p>
<p>Contador normal: 1</p>
```

| Parte de la respuesta | Función |
| --- | --- |
| `HTTP/1.1 200 OK` | Línea de estado: indica que la petición se ha atendido correctamente. |
| `Set-Cookie` | Cabecera que solicita guardar la cookie con el identificador. |
| `Content-Type` | Cabecera que indica que el cuerpo contiene HTML. |
| Línea en blanco | Separa las cabeceras del cuerpo. |
| HTML del cuerpo | Contenido que el navegador mostrará. |

**El navegador recibe el HTML resultante, no las instrucciones PHP.** `session_start()` y el resto del código PHP se ejecutan en el servidor.

Al finalizar normalmente la petición, PHP guarda los datos de la sesión en el servidor. El navegador recibe la respuesta, guarda la cookie si la acepta y muestra:

```text
Una página con memoria

Contador de sesión: 1
Contador normal: 1
```

En este momento:

| En el navegador | En el servidor |
| --- | --- |
| Cookie `UD04DEMO=abc123`. | Sesión identificada por `abc123`, con `visitas = 1`. |

**El contador no está dentro de la cookie.** La cookie contiene el identificador que permite recuperar la sesión. La variable normal no se conserva para la próxima ejecución.

### 7. Segunda solicitud: recargamos la página

Al recargar, el navegador envía **otra petición GET**. Esta vez incorpora la cookie:

```http
GET /sesiones.php HTTP/1.1
Host: localhost:8080
Cookie: UD04DEMO=abc123

```

El navegador no envía el número de visitas. Solo envía el identificador en la cabecera `Cookie`.

### 8. PHP vuelve a ejecutar todo el archivo

Se ejecutan de nuevo:

```php
session_name('UD04DEMO');
session_start();
```

`session_name()` indica qué nombre de cookie utilizamos. Esta vez `session_start()`:

1. Busca la cookie llamada `UD04DEMO` en la petición.
2. Obtiene su valor: `abc123`.
3. Localiza en el servidor la sesión correspondiente.
4. Recupera sus datos y los pone en `$_SESSION`.

Después de `session_start()`, ya tenemos:

```php
$_SESSION['visitas'] // Contiene el 1 guardado anteriormente.
```

**No crea una sesión nueva: recupera la existente.** `session_start()` significa iniciar o recuperar, según la sesión disponible.

Ahora encuentra:

```php
if (!isset($_SESSION['visitas'])) {
    $_SESSION['visitas'] = 0;
}
```

Como `visitas` ya existe, la condición es falsa. **Se salta la asignación a cero.**

Después ejecuta:

```php
$_SESSION['visitas']++;
```

El valor pasa de **1 a 2**.

En cambio, estas instrucciones vuelven a ejecutarse sin ninguna condición:

```php
$contadorNormal = 0;
$contadorNormal++;
```

La variable normal vuelve a empezar en cero y termina en **1**.

### 9. Segunda respuesta: los contadores ya son diferentes

El servidor responde, de forma simplificada:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

<h1>Una página con memoria</h1>
<p>Contador de sesión: 2</p>
<p>Contador normal: 1</p>
```

No hace falta volver a enviar `Set-Cookie` para actualizar el contador: el navegador ya tiene el identificador y PHP modifica los datos asociados en el servidor. Al terminar la petición conserva `visitas = 2`.

Si volvemos a recargar, se repite el proceso:

| Solicitud a sesiones.php | Contador de sesión | Contador normal |
| --- | ---: | ---: |
| Primera | 1 | 1 |
| Segunda | 2 | 1 |
| Tercera | 3 | 1 |

**Cada GET tiene su propia respuesta 200 y ejecuta de nuevo el archivo completo.** En este ejemplo no hay formulario, POST ni redirección 303.

### 10. Comprueba el recorrido en Chrome

1. Abre las herramientas con F12 y entra en Aplicación > Cookies. Elimina solo `UD04DEMO` para comenzar la prueba.
2. En Red, activa **Keep log / Conservar registro** y recarga `sesiones.php`.
3. Selecciona la petición: en las cabeceras de respuesta busca `Set-Cookie` con el identificador generado por PHP.
4. En Aplicación, comprueba la cookie guardada. Su valor no es el contador.
5. Recarga de nuevo. En las cabeceras de solicitud de este segundo GET busca `Cookie` con el mismo identificador.
6. Comprueba que el contador de sesión aumenta y el normal sigue valiendo 1.
7. Abre la misma dirección en otra pestaña del mismo perfil: comparte la cookie y continúa el contador. La pestaña anterior debe recargarse para mostrar el nuevo valor.
8. Abre una ventana de incógnito sin una sesión previa del ejemplo: tendrá un contador independiente. Las ventanas de incógnito abiertas entre sí pueden compartir cookies.

### Qué debemos recordar de esta prueba

* **Nosotros elegimos el nombre de la cookie; PHP genera su identificador; nosotros elegimos los datos de `$_SESSION`.**
* **La cookie viaja en `Set-Cookie` dentro de una respuesta y vuelve en `Cookie` dentro de otra petición. El contador permanece en el servidor.**
* **Las variables PHP se reconstruyen en cada petición. `session_start()` recupera los datos de sesión que permiten continuar el contador.**

Los mensajes anteriores son representaciones didácticas en HTTP/1.1: omitimos cabeceras adicionales. Las opciones concretas de la cookie dependen de la configuración de PHP. `abc123` representa el identificador real generado por PHP; no debemos escribirlo manualmente.

## 5.2. Qué recuerda el navegador y qué guarda el servidor

Seguimos con el contador del apartado anterior. Imaginemos que **Ana y Luis abren nuestra página desde navegadores diferentes**.

Ambos ejecutan el mismo archivo, que comienza con:

```php
session_name('UD04DEMO');
session_start();
```

El nombre de la cookie será el mismo, pero PHP genera un identificador para cada sesión:

| Navegador | Cookie guardada |
| --- | --- |
| Ana | `UD04DEMO=abc123` |
| Luis | `UD04DEMO=xyz789` |

Los identificadores son ilustrativos: en la aplicación real los genera PHP. **El nombre indica qué cookie buscar; el valor identifica qué sesión recuperar.** No se genera un identificador diferente en cada petición: normalmente el navegador devuelve el mismo para continuar la sesión.

### Cada sesión tiene sus propios datos

Supongamos que Ana ha visitado la página tres veces y Luis una. En el servidor tendremos:

| Sesión | Datos guardados |
| --- | --- |
| `abc123` | `visitas = 3` |
| `xyz789` | `visitas = 1` |

**Cada sesión tiene su propio contador**, aunque ambos visitantes utilicen la misma página y el mismo nombre de cookie.

### Cuando Ana recarga la página

Su navegador envía esta petición, simplificada en HTTP/1.1:

```http
GET /sesiones.php HTTP/1.1
Host: localhost:8080
Cookie: UD04DEMO=abc123

```

PHP ejecuta:

```php
session_name('UD04DEMO');
session_start();
```

Busca el identificador `abc123`, recupera sus datos y deja disponible:

```php
$_SESSION['visitas'] // Vale 3.
```

Después, nuestro código incrementa el contador:

```php
$_SESSION['visitas']++;
```

Ahora vale **4**. Esto no modifica el contador de Luis, porque pertenece a otra sesión.

### Tres lugares diferentes

Conviene distinguir dónde está cada elemento durante esta petición de Ana:

| Lugar | Qué contiene |
| --- | --- |
| **Navegador** | La cookie `UD04DEMO=abc123`. No contiene el contador. |
| **Almacenamiento de sesiones del servidor** | Los datos asociados a `abc123`, entre ellos `visitas = 3` antes de esta petición. |
| **Ejecución actual de PHP** | El array `$_SESSION`, donde recuperamos el 3 y lo cambiamos a 4. |

Al terminar normalmente la petición, PHP guarda el nuevo valor en el servidor. **La ejecución termina, pero los datos de la sesión quedan disponibles para otra petición.**

Por eso `$_SESSION` no es un programa que sigue ejecutándose entre visitas: **es el array con el que trabajamos en cada ejecución después de recuperar la sesión**.

### El nombre personalizado y el nombre predeterminado

```php
session_name('UD04DEMO');
```

Personaliza el nombre de la cookie. Si no lo indicamos, PHP utiliza el nombre configurado en el servidor, habitualmente **`PHPSESSID`**. El funcionamiento es el mismo: **nombre de cookie → identificador → datos de sesión**.

### Una sesión no equivale necesariamente a una persona

Ana y Luis sirven para representar dos navegadores con sesiones distintas. El identificador localiza una sesión; no demuestra por sí mismo quién está detrás. Nuestro contador funciona sin usuario ni contraseña.

Varias pestañas del mismo perfil comparten normalmente la cookie y, por tanto, la sesión. Una misma persona puede tener sesiones diferentes si utiliza distintos navegadores o perfiles.

El identificador no contiene los datos de `$_SESSION`, pero permite acceder a la sesión asociada. **No debemos copiarlo ni compartirlo con otras personas.**

## 5.3. Guardar un nombre y recuperar los datos en otra página

Hasta ahora el contador lo generaba PHP. Ahora será el visitante quien escriba un dato: vamos a **guardar un nombre enviado mediante un formulario y recuperarlo desde otra página**.

Trabajaremos con **dos archivos en la misma carpeta**:

| Archivo | Para qué sirve |
| --- | --- |
| `nombre.php` | Muestra un formulario y guarda el nombre recibido en la sesión. |
| `resumen.php` | Recupera la misma sesión y muestra el nombre guardado. |

Utilizamos `nombre.php` para conservar el ejemplo del contador en `sesiones.php`. Antes de probar el envío, crea los dos archivos siguientes. Mantenemos el código centrado en las sesiones; la validación completa de entradas se ha trabajado en el apartado de formularios.

### 1. El formulario y su procesamiento: nombre.php

```php
<?php
session_name('UD04DEMO');
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $_SESSION['nombre'] = $_POST['nombre'];

    header('Location: resumen.php', true, 303);
    exit;
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Guardar un nombre</title>
</head>
<body>
    <h1>Guardar un nombre en la sesión</h1>

    <form action="nombre.php" method="post">
        <label for="nombre">Escribe tu nombre:</label>
        <input type="text" id="nombre" name="nombre" required>

        <button type="submit">Guardar y ver resumen</button>
    </form>
</body>
</html>
```

Hay **dos destinos diferentes**, cada uno con una función:

| Instrucción | Qué provoca |
| --- | --- |
| `action="nombre.php"` | El formulario envía sus datos a `nombre.php` mediante POST. |
| `header('Location: resumen.php', true, 303)` | Después de guardar el nombre, indica al navegador que solicite `resumen.php` mediante GET. |

**El formulario no envía directamente el nombre a resumen.php.** Primero lo envía a `nombre.php`, donde PHP lo guarda en la sesión. El atributo `name="nombre"` del campo determina la clave `$_POST['nombre']`.

### 2. La segunda página: resumen.php

Guarda este archivo junto a `nombre.php`:

```php
<?php
session_name('UD04DEMO');
session_start();
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Resumen de la sesión</title>
</head>
<body>
    <h1>Resumen de la sesión</h1>

    <?php if (isset($_SESSION['nombre'])): ?>
        <p>
            Nombre guardado:
            <?= htmlspecialchars($_SESSION['nombre']) ?>
        </p>
    <?php else: ?>
        <p>Todavía no hay ningún nombre guardado.</p>
    <?php endif; ?>

    <a href="nombre.php">Volver al formulario</a>
</body>
</html>
```

Esta página también llama a `session_name()` y `session_start()` **antes del HTML** para recuperar la misma sesión. `isset()` permite mostrar un mensaje si aún no se ha guardado un nombre.

Utilizamos directamente `htmlspecialchars()` al mostrar el nombre, sin una función auxiliar: convierte el contenido en texto para insertarlo en el HTML.

### 3. Primera solicitud: abrir nombre.php

Si el archivo está en la raíz pública, abre:

```text
http://localhost:8080/nombre.php
```

Si está en una subcarpeta, añade su ruta. Suponemos que seguimos usando el navegador del contador, que conserva la cookie `UD04DEMO=abc123`.

El navegador envía:

```http
GET /nombre.php HTTP/1.1
Host: localhost:8080
Cookie: UD04DEMO=abc123

```

PHP ejecuta:

```php
session_name('UD04DEMO');
session_start();
```

Recupera la sesión identificada por `abc123`. **Los datos anteriores, como visitas, pueden seguir ahí:** añadir después la clave `nombre` no elimina el contador.

A continuación encuentra:

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
```

Como la petición es GET, se salta todo el bloque y genera el HTML del formulario. La respuesta simplificada es:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html lang="es">
<body>
    <h1>Guardar un nombre en la sesión</h1>

    <form action="nombre.php" method="post">
        <label for="nombre">Escribe tu nombre:</label>
        <input type="text" id="nombre" name="nombre" required>
        <button type="submit">Guardar y ver resumen</button>
    </form>
</body>
</html>
```

Si entramos sin una cookie válida, `session_start()` iniciará una sesión nueva y solicitará guardar su identificador, como vimos en el 5.1.

**Hasta aquí tenemos el formulario en pantalla. Todavía no hemos enviado ningún nombre ni visitado resumen.php.**

### 4. Segunda solicitud: enviar el formulario

Escribimos **Ana** y pulsamos **Guardar y ver resumen**. El navegador envía:

```http
POST /nombre.php HTTP/1.1
Host: localhost:8080
Cookie: UD04DEMO=abc123
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

nombre=Ana
```

| Parte | Qué contiene |
| --- | --- |
| Cabecera `Cookie` | El identificador de la sesión: `abc123`. |
| Cuerpo | El dato escrito en el formulario: `nombre=Ana`. |

### 5. PHP recupera la sesión y guarda el nombre

El servidor vuelve a ejecutar `nombre.php` desde el principio:

```php
session_name('UD04DEMO');
session_start();
```

PHP busca la cookie `UD04DEMO`, obtiene `abc123` y recupera los datos de esa sesión. No crea una sesión distinta para el segundo archivo o para cada petición.

Después llega a este bloque, que está escrito en `nombre.php`:

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $_SESSION['nombre'] = $_POST['nombre'];

    header('Location: resumen.php', true, 303);
    exit;
}
```

Como la petición es POST, entra. Esta instrucción:

```php
$_SESSION['nombre'] = $_POST['nombre'];
```

copia el nombre recibido a la sesión. En este envío equivale a:

```php
$_SESSION['nombre'] = 'Ana';
```

| Array | Qué contiene |
| --- | --- |
| `$_POST` | Los datos enviados en esta petición: `nombre = Ana`. |
| `$_SESSION` | Los datos recuperados de la sesión, a los que añadimos `nombre = Ana`. |

Si seguíamos con la sesión del contador, ahora puede contener ambos datos:

```php
// Ejemplo de los datos disponibles; no hay que añadir estas asignaciones.
$_SESSION['visitas'] // Por ejemplo, 3.
$_SESSION['nombre']  // 'Ana'.
```

**Añadir el nombre no elimina el contador:** son dos claves del mismo array. La cookie sigue siendo `UD04DEMO=abc123`; el nombre se guarda en el servidor.

A diferencia de `setcookie()`, **asignar un valor a $_SESSION sí permite consultarlo inmediatamente en ese array**. PHP conserva los datos de la sesión al finalizar normalmente la petición.

### 6. Respuesta al POST: redirigir a resumen.php

Después de guardar, PHP ejecuta:

```php
header('Location: resumen.php', true, 303);
exit;
```

La respuesta simplificada es:

```http
HTTP/1.1 303 See Other
Location: resumen.php

```

El `exit` termina esta ejecución e impide continuar hasta el HTML de `nombre.php`. No impide el guardado normal de los datos de la sesión.

El navegador interpreta la respuesta:

> Ahora debo solicitar resumen.php mediante GET.

No necesita recibir otro `Set-Cookie` para guardar el nombre: el identificador ya está en el navegador y el dato se conserva en el servidor.

### 7. Tercera solicitud: visitar resumen.php

Al seguir la redirección, el navegador envía:

```http
GET /resumen.php HTTP/1.1
Host: localhost:8080
Cookie: UD04DEMO=abc123

```

**No envía nombre=Ana en esta petición.** Ese dato se envió en el POST anterior y ya está guardado en el servidor.

El servidor ejecuta `resumen.php`. Al comienzo del archivo encuentra:

```php
session_name('UD04DEMO');
session_start();
```

Estas instrucciones recuperan la misma sesión mediante la cookie. Después de `session_start()`, tenemos disponible:

```php
$_SESSION['nombre'] // Contiene 'Ana'.
```

**No necesitamos incluir nombre.php ni pasar el nombre por la URL.** Ambos archivos acceden a los datos de la misma sesión al recuperarla.

### 8. Respuesta final: mostrar el nombre

En `resumen.php`, esta condición se cumple:

```php
if (isset($_SESSION['nombre'])):
```

Por eso PHP genera el mensaje con el nombre guardado y responde, de forma simplificada:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html lang="es">
<body>
    <h1>Resumen de la sesión</h1>
    <p>Nombre guardado: Ana</p>
    <a href="nombre.php">Volver al formulario</a>
</body>
</html>
```

El navegador muestra **Nombre guardado: Ana**. Recargar el resumen repite un GET, no el envío del formulario.

### 9. El recorrido completo

| Solicitud | Qué hace PHP | Respuesta |
| --- | --- | --- |
| GET a `nombre.php`. | Recupera o inicia la sesión y muestra el formulario. | 200 con HTML. |
| POST a `nombre.php`. | Recupera la sesión y guarda el nombre. | 303 hacia `resumen.php`. |
| GET a `resumen.php`. | Recupera la misma sesión y muestra el nombre. | 200 con HTML. |

**Son dos archivos y tres peticiones independientes, conectadas por la misma sesión.** El servidor ejecuta únicamente el archivo solicitado en cada petición.

### 10. Comprueba el funcionamiento

1. Crea ambos archivos y abre `nombre.php` desde el servidor local.
2. En Red, activa Keep log / Conservar registro.
3. Escribe `Ana` y guarda. Observa el POST a `nombre.php` con respuesta 303 y el GET a `resumen.php` con respuesta 200.
4. En Payload del POST localiza `nombre=Ana`. Comprueba que el GET posterior no vuelve a enviar ese campo.
5. En las cabeceras de solicitud comprueba que ambas peticiones envían el mismo identificador de sesión.
6. Recarga `resumen.php`: sigue mostrando el nombre sin repetir el POST.
7. Vuelve al formulario, escribe otro nombre y guarda: se sustituye el valor de la clave `nombre` en la misma sesión.
8. En una sesión nueva, sin un nombre guardado, visita directamente `resumen.php`: muestra que todavía no hay ningún nombre.

**Guardar un nombre no autentica a nadie.** Aquí solo practicamos el mantenimiento de datos entre páginas; el acceso con contraseña se estudiará después.

Los mensajes HTTP son representaciones didácticas en HTTP/1.1, con cabeceras adicionales omitidas y HTML abreviado. `abc123` representa un identificador generado por PHP.

## 5.4. Eliminar un dato o cerrar la sesión

Seguimos con la sesión de los ejemplos anteriores. Puede contener `visitas = 3` y `nombre = Ana`, y el navegador conserva la cookie `UD04DEMO=abc123`.

Vamos a distinguir tres operaciones: **eliminar un dato, vaciar todos los datos y cerrar la sesión completa**. Primero probaremos cada una; al final tendremos tres botones en `resumen.php`.

### 1. Eliminar solo el nombre: unset()

```php
unset($_SESSION['nombre']);
```

Esta instrucción elimina la clave `nombre` del array de sesión. El contador continúa y la sesión mantiene su identificador.

| Antes | Después |
| --- | --- |
| `visitas = 3` | `visitas = 3` |
| `nombre = Ana` | La clave `nombre` ya no existe. |

Añade a `resumen.php`, antes del enlace de vuelta, este formulario:

```html
<form action="olvidar.php" method="post">
    <button type="submit">Olvidar el nombre</button>
</form>
```

Al pulsarlo se enviará un POST a `olvidar.php`. No hay que enviar el nombre: el archivo sabe qué clave debe eliminar y la cookie identifica la sesión.

Crea **`olvidar.php`** en la misma carpeta:

```php
<?php
session_name('UD04DEMO');
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    unset($_SESSION['nombre']);
}

header('Location: resumen.php', true, 303);
exit;
```

Este archivo no genera HTML. Recupera la sesión, elimina el nombre si recibe un POST y redirige. Si lo abrimos directamente mediante GET, no elimina el dato: simplemente redirige.

#### Solicitud: pulsamos Olvidar el nombre

```http
POST /olvidar.php HTTP/1.1
Host: localhost:8080
Cookie: UD04DEMO=abc123
Content-Length: 0

```

El formulario no tiene campos con nombre que enviar. **El destino identifica la operación; la cookie identifica la sesión.**

PHP ejecuta `session_name()` y `session_start()`, recupera la sesión `abc123` y, al comprobar que es POST, ejecuta `unset($_SESSION['nombre'])`. Después responde:

```http
HTTP/1.1 303 See Other
Location: resumen.php

```

Al finalizar normalmente la petición, PHP conserva los datos actualizados, sin el nombre.

El navegador sigue la redirección:

```http
GET /resumen.php HTTP/1.1
Host: localhost:8080
Cookie: UD04DEMO=abc123

```

`resumen.php` recupera la misma sesión. Ahora `isset($_SESSION['nombre'])` es falso y responde:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

<!-- Resto del HTML de resumen.php -->
<p>Todavía no hay ningún nombre guardado.</p>
```

**La cookie y la sesión siguen siendo las mismas.** Si abrimos el contador en `sesiones.php`, recuperará las visitas anteriores y sumará una más.

### 2. Vaciar todos los datos: $_SESSION = []

Si queremos eliminar todas las claves del array:

```php
$_SESSION = [];
```

| Antes | Después |
| --- | --- |
| `visitas = 3` | La clave `visitas` ya no existe. |
| `nombre = Ana` | La clave `nombre` ya no existe. |

**La sesión sigue existiendo y conserva su identificador.** No hemos destruido la sesión ni borrado su cookie.

Añade otro formulario a `resumen.php`:

```html
<form action="vaciar.php" method="post">
    <button type="submit">Vaciar los datos de la sesión</button>
</form>
```

Crea **`vaciar.php`** en la misma carpeta:

```php
<?php
session_name('UD04DEMO');
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $_SESSION = [];
}

header('Location: resumen.php', true, 303);
exit;
```

#### Solicitud: pulsamos Vaciar los datos de la sesión

```http
POST /vaciar.php HTTP/1.1
Host: localhost:8080
Cookie: UD04DEMO=abc123
Content-Length: 0

```

PHP recupera la sesión mediante `session_start()`, vacía su array y responde:

```http
HTTP/1.1 303 See Other
Location: resumen.php

```

Al terminar normalmente la petición conserva la sesión con sus datos vacíos. El navegador vuelve al resumen:

```http
GET /resumen.php HTTP/1.1
Host: localhost:8080
Cookie: UD04DEMO=abc123

```

La sesión recuperada ya no contiene nombre ni visitas. El resumen responde con 200 y muestra que no hay nombre guardado.

#### ¿Qué ocurre al volver al contador?

En `sesiones.php` se ejecuta:

```php
if (!isset($_SESSION['visitas'])) {
    $_SESSION['visitas'] = 0;
}

$_SESSION['visitas']++;
```

Como la clave `visitas` ya no existe, la crea con cero y suma uno. El contador vuelve a mostrar **1**.

**No ha empezado otra sesión: nuestro código ha creado otra vez el contador dentro de la misma sesión.**

### 3. Cerrar la sesión completa

Para cerrar la sesión anterior y eliminar la referencia que conserva el navegador, realizaremos tres operaciones:

| Operación | Instrucción |
| --- | --- |
| Vaciar los datos de la ejecución actual. | `$_SESSION = [];` |
| Eliminar los datos almacenados de esta sesión en el servidor. | `session_destroy();` |
| Solicitar borrar la cookie del identificador. | `setcookie()` con caducidad pasada. |

**session_destroy() no vacía automáticamente el array actual ni borra la cookie del navegador.**

Añade el tercer formulario a `resumen.php`:

```html
<form action="cerrar.php" method="post">
    <button type="submit">Cerrar la sesión completa</button>
</form>
```

Crea **`cerrar.php`** en la misma carpeta:

```php
<?php
session_name('UD04DEMO');
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    // Vaciamos los datos de la ejecución actual.
    $_SESSION = [];

    // Eliminamos los datos de esta sesión en el servidor.
    session_destroy();

    // Solicitamos borrar la cookie del identificador.
    setcookie('UD04DEMO', '', time() - 3600, '/');
}

header('Location: resumen.php', true, 303);
exit;
```

Para esta demostración suponemos que la cookie se creó con **ruta `/` y sin un dominio explícito**. Compruébalo en Aplicación > Cookies. El borrado debe identificar la misma cookie por nombre, dominio y ruta; si el entorno utiliza otro ámbito, habrá que adaptarlo.

La llamada a `setcookie()` indica:

| Argumento | Significado |
| --- | --- |
| `'UD04DEMO'` | Nombre de la cookie que queremos borrar. |
| `''` | Valor vacío. |
| `time() - 3600` | Fecha de caducidad de hace una hora. |
| `'/'` | Ruta de la cookie. |

Poner una fecha pasada solicita el borrado al navegador. No basta con borrar una entrada de `$_COOKIE`, porque eso solo modificaría el array de la ejecución actual.

#### Solicitud: pulsamos Cerrar la sesión completa

```http
POST /cerrar.php HTTP/1.1
Host: localhost:8080
Cookie: UD04DEMO=abc123
Content-Length: 0

```

PHP recupera la sesión `abc123`. Como es POST, vacía el array, destruye los datos almacenados y prepara el borrado de la cookie. Responde con dos instrucciones:

```http
HTTP/1.1 303 See Other
Set-Cookie: UD04DEMO=deleted; Max-Age=0; Path=/
Location: resumen.php

```

La representación es simplificada: PHP puede incluir también `Expires` con una fecha pasada. `deleted` es un valor de borrado que puede aparecer en la cabecera, no un nuevo identificador que debamos guardar.

| Cabecera | Qué indica al navegador |
| --- | --- |
| `Set-Cookie` con caducidad inmediata. | Elimina la cookie `UD04DEMO`. |
| `Location`. | Solicita `resumen.php` mediante GET. |

El navegador procesa el borrado y sigue la redirección. La sesión anterior ya no contiene sus datos en el servidor.

#### Nueva solicitud: volver al resumen sin la cookie anterior

```http
GET /resumen.php HTTP/1.1
Host: localhost:8080

```

No se envía `UD04DEMO=abc123`. Pero `resumen.php` comienza con:

```php
session_name('UD04DEMO');
session_start();
```

Como no recibe el identificador anterior, **PHP inicia una sesión nueva**. Por ejemplo, genera `def456` y responde:

```http
HTTP/1.1 200 OK
Set-Cookie: UD04DEMO=def456; Path=/
Content-Type: text/html; charset=UTF-8

<!-- Resto del HTML de resumen.php -->
<p>Todavía no hay ningún nombre guardado.</p>
```

**Volver a ver una cookie no significa que el cierre haya fallado.** La sesión anterior se cerró; la página de destino acaba de iniciar otra.

| Antes de cerrar | Después de volver al resumen |
| --- | --- |
| Cookie `UD04DEMO=abc123`. | Cookie nueva `UD04DEMO=def456`. |
| La sesión podía contener nombre y visitas. | La nueva sesión no contiene esos datos. |

Si abrimos después `sesiones.php`, su código creará el contador y mostrará 1.

### 4. resumen.php con los tres botones

Este es el archivo completo después de añadir las operaciones anteriores. Los tres archivos de procesamiento deben estar en su misma carpeta.

```php
<?php
session_name('UD04DEMO');
session_start();
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Resumen de la sesión</title>
</head>
<body>
    <h1>Resumen de la sesión</h1>

    <?php if (isset($_SESSION['nombre'])): ?>
        <p>
            Nombre guardado:
            <?= htmlspecialchars($_SESSION['nombre']) ?>
        </p>
    <?php else: ?>
        <p>Todavía no hay ningún nombre guardado.</p>
    <?php endif; ?>

    <form action="olvidar.php" method="post">
        <button type="submit">Olvidar el nombre</button>
    </form>

    <form action="vaciar.php" method="post">
        <button type="submit">Vaciar los datos de la sesión</button>
    </form>

    <form action="cerrar.php" method="post">
        <button type="submit">Cerrar la sesión completa</button>
    </form>

    <a href="nombre.php">Volver al formulario</a>
    <a href="sesiones.php">Consultar el contador</a>
</body>
</html>
```

### 5. Comparar las tres operaciones

| Operación | Nombre | Contador | Identificador tras volver al resumen |
| --- | --- | --- | --- |
| Olvidar el nombre. | Se elimina. | Se conserva. | El mismo. |
| Vaciar todos los datos. | Se elimina. | Se elimina; al visitar el contador se inicializa otra vez. | El mismo. |
| Cerrar la sesión completa. | Se elimina. | Se elimina; al visitar el contador se inicializa otra vez. | Uno nuevo, al iniciar otra sesión en el resumen. |

Para comprobar cada caso, prepara antes los datos: visita el contador varias veces y guarda un nombre desde `nombre.php`. En Chrome activa Keep log / Conservar registro, pulsa un botón y revisa el POST 303 y el GET 200 posterior. En el cierre completo, observa además la cabecera de borrado y el nuevo identificador.

Los mensajes HTTP son representaciones simplificadas en HTTP/1.1. Suponemos que el navegador acepta y procesa las cookies. Los identificadores `abc123` y `def456` son ilustrativos; las cabeceras y opciones reales dependen de la configuración del entorno.

## 5.5. Ejemplo completo: todos los archivos

Estos son los archivos que hemos construido en los apartados anteriores. Guárdalos **todos en la misma carpeta** del servidor PHP local.

| Archivo | Función |
| --- | --- |
| `sesiones.php` | Cuenta las visitas y compara una variable de sesión con una normal. |
| `nombre.php` | Muestra el formulario y guarda el nombre. |
| `resumen.php` | Muestra el nombre y los botones para eliminar datos. |
| `olvidar.php` | Elimina solo el nombre. |
| `vaciar.php` | Vacía todos los datos de la sesión. |
| `cerrar.php` | Destruye la sesión y solicita borrar su cookie. |

Es el mismo ejemplo guiado, no un ejercicio nuevo. Mantenemos el código centrado en las sesiones; las comprobaciones de entrada ya estudiadas se incorporarán al desarrollar aplicaciones completas. Para el borrado de la cookie mantenemos el ámbito del ejemplo: ruta `/` y sin dominio explícito.

### sesiones.php

```php
<?php
declare(strict_types=1);

session_name('UD04DEMO');
session_start();

if (!isset($_SESSION['visitas'])) {
    $_SESSION['visitas'] = 0;
}
$_SESSION['visitas']++;

$contadorNormal = 0;
$contadorNormal++;
?>
<h1>Una página con memoria</h1>
<p>Contador de sesión: <?= $_SESSION['visitas'] ?></p>
<p>Contador normal: <?= $contadorNormal ?></p>
<p><a href="nombre.php">Guardar un nombre</a></p>
<p><a href="resumen.php">Ver resumen</a></p>
```

### nombre.php

```php
<?php
session_name('UD04DEMO');
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $_SESSION['nombre'] = $_POST['nombre'];

    header('Location: resumen.php', true, 303);
    exit;
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Guardar un nombre</title>
</head>
<body>
    <h1>Guardar un nombre en la sesión</h1>

    <form action="nombre.php" method="post">
        <label for="nombre">Escribe tu nombre:</label>
        <input type="text" id="nombre" name="nombre" required>
        <button type="submit">Guardar y ver resumen</button>
    </form>
</body>
</html>
```

### resumen.php

```php
<?php
session_name('UD04DEMO');
session_start();
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Resumen de la sesión</title>
</head>
<body>
    <h1>Resumen de la sesión</h1>

    <?php if (isset($_SESSION['nombre'])): ?>
        <p>Nombre guardado:
            <?= htmlspecialchars($_SESSION['nombre']) ?>
        </p>
    <?php else: ?>
        <p>Todavía no hay ningún nombre guardado.</p>
    <?php endif; ?>

    <form action="olvidar.php" method="post">
        <button type="submit">Olvidar el nombre</button>
    </form>

    <form action="vaciar.php" method="post">
        <button type="submit">Vaciar los datos de la sesión</button>
    </form>

    <form action="cerrar.php" method="post">
        <button type="submit">Cerrar la sesión completa</button>
    </form>

    <a href="nombre.php">Volver al formulario</a>
    <a href="sesiones.php">Consultar el contador</a>
</body>
</html>
```

### olvidar.php

```php
<?php
session_name('UD04DEMO');
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    unset($_SESSION['nombre']);
}

header('Location: resumen.php', true, 303);
exit;
```

### vaciar.php

```php
<?php
session_name('UD04DEMO');
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $_SESSION = [];
}

header('Location: resumen.php', true, 303);
exit;
```

### cerrar.php

```php
<?php
session_name('UD04DEMO');
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $_SESSION = [];
    session_destroy();
    setcookie('UD04DEMO', '', time() - 3600, '/');
}

header('Location: resumen.php', true, 303);
exit;
```

## 5.6. Resumen de los recorridos

| Operación | Solicitudes y respuestas | Resultado |
| --- | --- | --- |
| Visitar el contador. | GET a `sesiones.php` → 200. | Inicia o recupera la sesión e incrementa las visitas. |
| Guardar un nombre. | GET a `nombre.php` → 200; POST a `nombre.php` → 303; GET a `resumen.php` → 200. | Guarda el nombre y lo muestra en otra página. |
| Olvidar el nombre. | POST a `olvidar.php` → 303; GET a `resumen.php` → 200. | Elimina solo el nombre; conserva la sesión y los demás datos. |
| Vaciar los datos. | POST a `vaciar.php` → 303; GET a `resumen.php` → 200. | Conserva la sesión, pero sin sus datos anteriores. |
| Cerrar la sesión. | POST a `cerrar.php` → 303; GET a `resumen.php` → 200. | Destruye la sesión anterior y solicita borrar su cookie; el resumen inicia una nueva. |

**La cookie transporta el identificador; los datos de `$_SESSION` permanecen en el servidor.** Cada petición ejecuta el archivo solicitado desde el principio. `Set-Cookie` establece o elimina la cookie cuando corresponde; no hace falta reenviarla para actualizar cada dato de la sesión.

## 5.7. Comprueba lo que ocurre en Chrome

### Aplicación: el identificador

1. Abre `sesiones.php`, pulsa F12 y entra en **Aplicación (Application)**.
2. En **Almacenamiento > Cookies**, selecciona tu servidor.
3. Busca `UD04DEMO`. Su valor es un identificador, no el nombre ni el contador.
4. Abre `nombre.php`, guarda un nombre y recarga el resumen: el nombre se mantiene sin copiarlo a una cookie propia.
5. Pulsa **Olvidar el nombre**: desaparece el nombre y se conserva el identificador. Al consultar `sesiones.php`, el contador continúa y suma una visita.
6. Prepara de nuevo un nombre y varias visitas. Pulsa **Vaciar los datos de la sesión**: conserva el identificador, pero elimina nombre y visitas. Al abrir `sesiones.php`, su código vuelve a crear el contador y muestra 1.
7. Pulsa **Cerrar la sesión completa**: la respuesta solicita borrar la cookie anterior. Tras redirigir, `resumen.php` inicia una sesión nueva con otro identificador, sin nombre ni visitas. El contador se inicializa cuando vuelves a `sesiones.php`.

### Red: las peticiones

1. En **Red (Network)** activa **Conservar registro (Preserve log / Keep log)**.
2. Abre `nombre.php` y guarda un nombre. Observa el **POST a nombre.php con respuesta 303** y el **GET a resumen.php con respuesta 200**.
3. En **Carga útil (Payload)** del POST aparece el campo `nombre`. En las cabeceras de solicitud viaja la cookie con el identificador.
4. En el GET posterior no se vuelve a enviar el nombre como campo del formulario: PHP lo recupera de la sesión mediante el identificador.
5. Prueba cada botón del resumen. Localiza el POST a `olvidar.php`, `vaciar.php` o `cerrar.php` y el GET posterior a `resumen.php`.
6. En el cierre completo, compara la respuesta del POST, que solicita borrar la cookie mediante `Set-Cookie`, con la respuesta del GET posterior, que establece la cookie de la nueva sesión.

Para observar el `Set-Cookie` inicial, elimina manualmente solo `UD04DEMO` y vuelve a cargar `sesiones.php`. Borrar la cookie desde Chrome no elimina por sí mismo los datos del servidor: el navegador pierde la referencia a ellos.

| Prueba | Resultado esperado |
| --- | --- |
| Recargar `sesiones.php`. | Aumenta el contador de sesión; el normal sigue en 1. |
| Guardar un nombre desde `nombre.php`. | `resumen.php` lo muestra después de la redirección. |
| Guardar otro nombre. | Sustituye el valor anterior en la misma sesión. |
| Recargar `resumen.php`. | Mantiene el nombre sin repetir el POST; no incrementa el contador. |
| Nombre con etiquetas HTML, escrito como texto en el campo. | Se muestra como texto mediante `htmlspecialchars()`. |
| Visitar directamente `resumen.php` en una sesión sin nombre. | Muestra que todavía no hay ningún nombre guardado. |
| Otra pestaña del mismo perfil. | Comparte los datos; recarga la página para mostrar cambios. |
| Incógnito sin sesión previa. | Datos independientes de la sesión del perfil normal. |
| Olvidar el nombre. | Desaparece solo el nombre; conserva visitas e identificador. |
| Vaciar los datos. | Elimina nombre y visitas; conserva el identificador. |
| Cerrar la sesión completa. | Elimina la sesión anterior y su cookie; el resumen inicia otra sin los datos anteriores. |
| Abrir `sesiones.php` después de vaciar o cerrar. | Inicializa el contador y muestra 1. |

Estas pruebas corresponden al ejemplo simplificado. Las validaciones de entradas se han trabajado en el apartado de formularios.

## 5.8. Tres ideas clave

* **Los datos de sesión se guardan en el servidor; el navegador conserva normalmente una cookie con su identificador.**
* **Cada página que necesite esos datos debe iniciar o recuperar la misma sesión antes de enviar HTML.**
* **Eliminar un dato, vaciar el array, destruir la sesión y borrar la cookie son operaciones distintas.**

Cerrar una pestaña no equivale a cerrar la sesión. Tampoco debemos garantizar que cerrar el navegador elimine inmediatamente una cookie de sesión: la restauración del navegador y la configuración del servidor afectan a su duración. Una sesión no sustituye a una base de datos para información que deba conservarse de forma duradera.

## Anexo · Esquema de pizarra

[Descargar el PDF: Una página con memoria](../Pizarra_sesiones_PHP.pdf)

## Practica lo aprendido

Realiza el **ejercicio 7 del bloque 5: Sesiones**, «Un carrito que recuerda tus productos», en la [relación de actividades](actividades.md#bloque-5-sesiones). El enunciado y la solución del profesor se mantienen separados.

## Referencias

* [Manual de PHP: session_start](https://www.php.net/manual/es/function.session-start.php).
* [Manual de PHP: session_destroy](https://www.php.net/manual/es/function.session-destroy.php).
* [Manual de PHP: $_SESSION](https://www.php.net/manual/es/reserved.variables.session.php).

## Autoría y licencia

Material elaborado por **Joaquín García** para DWES de 2.º DAW del **IES Portada Alta**. Continúa la adaptación didáctica de los materiales de Aitor Medrano y Luis Alemañ, contrastada con el manual de PHP. **CC BY-NC-SA**.
