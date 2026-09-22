# 2. Formularios con PHP: GET y POST

En el apartado anterior asignamos valores directamente en el código. Ahora el visitante podrá introducir datos en una página y enviarlos al servidor. Ya conoces los formularios HTML: aquí nos centraremos en cómo llegan los datos a PHP y en qué cambia al usar `GET` o `POST`.

## 2.1. El recorrido de un formulario

Al pulsar «Enviar», el navegador crea una petición al archivo indicado en `action`. El atributo `method` establece cómo se envían los datos. El servidor ejecuta el archivo PHP y devuelve una respuesta HTML. Para que se envíe un campo, su elemento HTML necesita un atributo `name`.

```html
<form action="saluda-get.php" method="get">
    <label for="nombre">Nombre:</label>
    <input type="text" id="nombre" name="nombre">
    <button type="submit">Enviar</button>
</form>
```

| Elemento | Función |
| --- | --- |
| `action` | Archivo que procesará la petición. |
| `method` | Método de envío: en este apartado, `get` o `post`. |
| `name="nombre"` | Clave con la que PHP recuperará el valor. |
| `id="nombre"` y `for="nombre"` | Relacionan la etiqueta y el campo; `id` no sustituye a `name`. |

## 2.2. Enviar con GET

Con `method="get"`, los datos del formulario se añaden a la URL en la **cadena de consulta**. Si escribimos `Ana`, la dirección de destino puede quedar así:

```text
http://localhost:8080/unidad-02/saluda-get.php?nombre=Ana
```

Varios valores se separan con `&`, por ejemplo `?nombre=Ana&apellido=Lopez`. El navegador codifica los caracteres necesarios de la URL. PHP permite acceder a los parámetros mediante `$_GET`, un array asociativo predefinido. Por ahora lee `$_GET['nombre']` como «el valor asociado a la clave `nombre`»; estudiaremos los arrays con detalle más adelante.

**Formulario** (`saluda-get.html`):

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Saludo con GET</title>
</head>
<body>
    <h1>Saludo con GET</h1>
    <form action="saluda-get.php" method="get">
        <label for="nombre">Nombre:</label>
        <input type="text" id="nombre" name="nombre">
        <button type="submit">Enviar</button>
    </form>
</body>
</html>
```

**Respuesta** (`saluda-get.php`, en la misma carpeta):

```php
<?php
$nombre = $_GET['nombre'] ?? '';
$nombreHtml = htmlspecialchars($nombre, ENT_QUOTES, 'UTF-8');
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Resultado GET</title>
</head>
<body>
    <h1>Resultado</h1>
    <p>Hola, <?= $nombreHtml ?>.</p>
    <p><a href="saluda-get.html">Volver al formulario</a></p>
</body>
</html>
```

### ¿Qué hacen las dos líneas de PHP?

```php
$nombre = $_GET['nombre'] ?? '';
$nombreHtml = htmlspecialchars($nombre, ENT_QUOTES, 'UTF-8');
```

En la primera línea, `$_GET['nombre']` recupera el valor enviado con la clave `nombre`. El operador `??` significa «si ese valor existe y no es `null`, úsalo; en caso contrario, usa el valor de la derecha». Aquí el valor alternativo es `''`, una cadena vacía:

| Dirección solicitada | Valor de `$nombre` |
| --- | --- |
| `saluda-get.php?nombre=Ana` | `'Ana'` |
| `saluda-get.php` | `''` |

Así podemos abrir el archivo PHP directamente, sin pasar por el formulario, y no aparece un aviso por acceder a una clave inexistente. Si el usuario envía el campo vacío, su valor también será `''`; **`??` evita el aviso, pero no valida que se haya escrito un nombre**.

La segunda línea prepara el dato para **mostrarlo dentro de HTML**. `htmlspecialchars()` no quita contenido ni modifica `$nombre`: convierte los caracteres especiales en una nueva cadena, `$nombreHtml`, para que el navegador los presente literalmente. Por ejemplo:

```text
Texto escrito en el formulario:  <b>Ana</b>
Valor de $nombre:             <b>Ana</b>
Valor de $nombreHtml:         &lt;b&gt;Ana&lt;/b&gt;
Lo que se ve en la página:    <b>Ana</b> (como texto, sin negrita)
```

`ENT_QUOTES` incluye en la conversión las comillas simples y dobles. Si alguien escribe `Ana "La Rápida"`, el HTML de respuesta contiene entidades para esas comillas, pero el navegador las **muestra como comillas normales**. `'UTF-8'` indica la codificación que utilizamos. Escapar al mostrar y **validar** son tareas distintas: validar consistiría, por ejemplo, en comprobar si el nombre está vacío o supera una longitud permitida.

Prueba el formulario y observa la URL. Cambia `nombre` directamente en la dirección y recarga: el archivo PHP recibe el parámetro aunque no hayas enviado el formulario. Una URL con parámetros GET puede copiarse o guardarse; es útil en búsquedas y filtros que consultan información.

## 2.3. Enviar con POST

Con `method="post"`, el navegador envía los valores en el **cuerpo de la petición**. El nombre escrito no aparece como parámetro en la barra de direcciones. PHP recoge estos datos de un formulario POST mediante `$_POST`.

Compara este formulario con el anterior. Guarda ambos archivos juntos:

**Formulario** (`saluda-post.html`):

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Saludo con POST</title>
</head>
<body>
    <h1>Saludo con POST</h1>
    <form action="saluda-post.php" method="post">
        <label for="nombre">Nombre:</label>
        <input type="text" id="nombre" name="nombre">
        <button type="submit">Enviar</button>
    </form>
</body>
</html>
```

**Respuesta** (`saluda-post.php`):

```php
<?php
$nombre = $_POST['nombre'] ?? '';
$nombreHtml = htmlspecialchars($nombre, ENT_QUOTES, 'UTF-8');
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Resultado POST</title>
</head>
<body>
    <h1>Resultado</h1>
    <p>Hola, <?= $nombreHtml ?>.</p>
    <p><a href="saluda-post.html">Volver al formulario</a></p>
</body>
</html>
```

Si cambias `method="post"` por `method="get"` sin modificar el archivo PHP, `$_POST['nombre']` no tendrá el valor enviado. El método del formulario y la variable de PHP deben corresponderse. El operador `??` y `htmlspecialchars()` cumplen aquí las mismas funciones que en el ejemplo GET: contemplan la ausencia del dato y preparan su presentación en HTML.

**POST no cifra los datos por sí mismo.** Aunque no se vean en la URL, se pueden inspeccionar en las herramientas de desarrollo del navegador; al trabajar en una web real, HTTPS protege los datos durante el transporte. Tampoco debemos confiar automáticamente en los datos recibidos por POST: se validarán según lo que necesite la aplicación.

## 2.4. Otros campos de un formulario POST

Además de los campos de texto, los botones de radio, las casillas y las áreas de texto envían valores según su atributo `name`:

```html
<label><input type="radio" name="lenguaje" value="php"> PHP</label>
<label><input type="radio" name="lenguaje" value="javascript"> JavaScript</label>

<label><input type="checkbox" name="novedades" value="si"> Recibir novedades</label>

<label for="comentario">Comentario:</label>
<textarea id="comentario" name="comentario"></textarea>
```

Los botones de radio que comparten `name="lenguaje"` forman un grupo: solo se envía el valor de la opción elegida. La casilla `novedades` solo envía su valor cuando está marcada; si no lo está, esa clave no existe en `$_POST`. El área de texto envía lo que la persona escriba. Podemos recuperar un campo opcional mediante `$_POST['novedades'] ?? ''` y, antes de mostrarlo en HTML, tratarlo con `htmlspecialchars()`.

Por ahora utilizaremos una sola casilla. Varias casillas bajo un mismo nombre requieren trabajar con arrays, que estudiaremos en otro apartado. Tampoco puntuaremos las respuestas del cuestionario hasta conocer las condiciones.

## 2.5. GET y POST frente a frente

| Pregunta | GET | POST |
| --- | --- | --- |
| ¿Dónde van los datos del formulario? | En los parámetros de la URL. | En el cuerpo de la petición. |
| ¿Cómo los lee PHP? | `$_GET['campo']` | `$_POST['campo']` |
| ¿Sirve copiar la URL para repetir la consulta? | Sí, cuando los parámetros contienen lo necesario. | No: la URL por sí sola no incluye los valores del cuerpo. |
| Uso habitual | Consultar, buscar o filtrar información. | Enviar datos que modifican algo, como crear un registro. |

La diferencia principal **no es que POST sea «seguro» y GET «inseguro»**: ambos reciben datos que deben tratarse correctamente. Elegimos el método según lo que hace la petición. En los ejemplos ambos se limitan a mostrar un saludo; su finalidad es comparar cómo llegan los valores.

## 2.6. Comprobaciones en el navegador

1. Envía el mismo nombre con los dos formularios y compara las direcciones finales.
2. Abre directamente `saluda-get.php?nombre=Ana`. ¿Qué ocurre?
3. Abre directamente `saluda-post.php`. ¿Qué ocurre y por qué?
4. Elimina temporalmente `name="nombre"` de uno de los formularios y observa el resultado.
5. Introduce `<b>Ana</b>` y consulta el HTML recibido. ¿Qué función evita que se interprete como una etiqueta?

En las próximas actividades pasaremos varios valores desde formularios, incluidos botones de radio, una casilla opcional y un área de texto. La validación completa y la gestión de errores de usuario se abordarán cuando conozcamos condiciones y funciones.

## Practica lo aprendido

Realiza los **ejercicios 7 y 8** del [bloque 2: Formularios con GET y POST](actividades.md#bloque-2-formularios-con-get-y-post) y entrega sus cuatro archivos juntos en la tarea del bloque 2 en Moodle. Primero convertirás la tabla de datos personales en un formulario GET; después crearás un cuestionario POST con distintos tipos de campos.

## Fuentes de consulta

- [Aitor Medrano: El lenguaje PHP](https://aitor-medrano.github.io/dwes2122/02php.html), sección «Trabajando con formularios».
- [Manual oficial de PHP: formularios](https://www.php.net/manual/es/tutorial.forms.php), [`$_GET`](https://www.php.net/manual/es/reserved.variables.get.php) y [`$_POST`](https://www.php.net/manual/es/reserved.variables.post.php).
- [Manual oficial de PHP: `htmlspecialchars()`](https://www.php.net/manual/es/function.htmlspecialchars.php).
