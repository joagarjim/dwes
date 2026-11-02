# 2. Formularios y validación

En la unidad 2 aprendimos a recoger campos con `$_GET` y `$_POST`. Ahora vamos a dar el siguiente paso: **comprobar los datos, explicar los errores y permitir que el usuario los corrija sin volver a escribirlo todo**.

Utilizaremos un formulario de producto como hilo conductor. Primero entenderemos cada operación por separado y después las reuniremos en un ejemplo completo.

## 2.1. El recorrido de un formulario

Un formulario no solo recoge información. La aplicación debe decidir si esa información se puede utilizar.

1. Mostrar el formulario con valores iniciales.
2. Recibir el envío y recoger los campos.
3. Comprobar que los valores cumplen las reglas.
4. Si hay errores, mostrar el formulario con mensajes y conservar lo introducido.
5. Si todo es correcto, realizar la operación y presentar el resultado.

Podemos repartir el formulario y su procesamiento en archivos distintos, como ya hemos hecho. En este apartado utilizaremos **un mismo archivo PHP** para mostrarlo y procesarlo: facilita devolver los errores junto a los campos.

```php
<?php
$errores = [];

if (($_SERVER['REQUEST_METHOD'] ?? '') === 'POST') {
    // Recoger, validar y decidir qué hacer.
}
?>
<!-- Después se muestra el formulario. -->
```

La primera visita normalmente llega mediante GET y no debe mostrar errores de campos que todavía no se han enviado.

## 2.2. Qué campos recibe PHP

En este formulario:

```html
<form action="producto.php" method="post">
    <label for="nombre">Nombre:</label>
    <input type="text" id="nombre" name="nombre">
    <button type="submit">Comprobar producto</button>
</form>
```

`action` indica el destino, `method` el método y **`name` la clave del dato enviado**. En PHP:

```php
$nombre = $_POST['nombre'] ?? '';
```

No todos los controles se envían siempre:

| Control | Qué debemos tener en cuenta |
| --- | --- |
| Campo con `name` | Permite enviar un valor identificado por ese nombre. |
| Campo sin `name` | No se incluye como campo del envío normal. |
| Campo `disabled` | No se envía. |
| Campo de texto `readonly` | Se envía, pero su valor sigue siendo manipulable por el cliente. |
| Checkbox sin marcar | No se envía. |
| Grupo de radio sin selección | No se envía un valor para el grupo. |
| Selección múltiple sin opciones elegidas | Puede no enviarse la clave. |

No debemos confiar en que un campo exista ni en que tenga la forma que esperamos. Los atributos HTML ayudan a utilizar el formulario, pero el cliente puede modificar el envío.

### Recoger un campo de texto

Un campo de formulario ordinario llega normalmente como una cadena, aunque se haya escrito un número. Un envío manipulado también podría convertirlo en un array.

```php
$entrada = $_POST['nombre'] ?? '';
$nombre = is_string($entrada) ? trim($entrada) : '';
```

* `?? ''` proporciona un valor si no existe la clave.
* `is_string` comprueba que hemos recibido un texto.
* `trim` elimina los espacios en blanco de los extremos.

No elimina espacios interiores, no valida el contenido y no protege por sí solo frente a HTML malicioso. Tampoco aplicaremos `trim` indiscriminadamente a contraseñas: sus espacios podrían formar parte de ellas.

En nuestros ejemplos podremos reunir esta recogida en una función:

```php
function campoTexto(string $clave): string
{
    $valor = $_POST[$clave] ?? '';
    return is_string($valor) ? trim($valor) : '';
}

$nombre = campoTexto('nombre');
```

Esta función devuelve una cadena vacía cuando falta el campo o tiene una forma inesperada. Es apropiada para los campos obligatorios del ejemplo, porque después rechazaremos ese valor vacío. **No sustituye la validación de las reglas del campo**.

## 2.3. Validación en el navegador y en el servidor

**Validar** significa comprobar que un dato cumple las condiciones que necesita la aplicación.

En HTML podemos escribir:

```html
<input type="text" name="nombre" required maxlength="80">
<input type="number" name="precio" required min="0.01" step="0.01">
<input type="email" name="email" required>
```

| Atributo | Para qué sirve |
| --- | --- |
| `required` | Impide el envío normal si el campo obligatorio está vacío. |
| `type="email"` | Pide al navegador comprobar un formato de correo. |
| `min` y `max` | Establecen límites numéricos. |
| `step` | Establece el incremento válido para un campo numérico. |
| `minlength` y `maxlength` | Establecen límites de longitud para determinados campos de texto. |

Esta validación ofrece ayuda inmediata, pero **no garantiza lo que recibe el servidor**: se puede modificar el HTML o enviar una petición sin utilizar el formulario.

Por eso validaremos también en PHP. La validación del navegador mejora la experiencia y la del servidor decide si se acepta la información.

### Reglas concretas

Para nuestro producto utilizaremos estas reglas:

| Campo | Regla |
| --- | --- |
| Nombre | Obligatorio; entre 3 y 80 caracteres. |
| Precio | Número entre 0,01 y 99999. |
| Stock | Entero entre 0 y 10000. El cero es válido. |
| Categoría | Debe coincidir con una de las categorías permitidas. |

### Mensajes de error por campo

Un array asociativo permite relacionar cada error con su campo:

```php
$errores = [];

if ($nombre === '') {
    $errores['nombre'] = 'Escribe el nombre del producto.';
}
```

Después podemos consultar `isset($errores['nombre'])` para mostrarlo junto al nombre. Si no hay errores, `$errores === []` será verdadero.

Conviene indicar qué debe corregirse: «El stock debe ser un entero entre 0 y 10000» ayuda más que «Dato incorrecto».

## 2.4. Comprobar textos, números y opciones

### Textos y longitud

```php
if ($nombre === '') {
    $errores['nombre'] = 'Escribe el nombre del producto.';
} elseif (mb_strlen($nombre, 'UTF-8') < 3
    || mb_strlen($nombre, 'UTF-8') > 80) {
    $errores['nombre'] = 'El nombre debe tener entre 3 y 80 caracteres.';
}
```

`mb_strlen` permite contar caracteres de textos UTF-8, incluyendo letras acentuadas. Requiere la extensión **mbstring** de PHP; debe estar habilitada en el contenedor. `strlen` cuenta bytes y puede dar un resultado diferente para esos textos.

### Enteros: no convertir antes de validar

Convertir directamente no permite saber si el dato original era válido:

```php
$stock = (int) 'abc'; // Se convierte en 0.
```

Podemos comprobar el dato con `filter_var` y un filtro de validación:

```php
$stock = filter_var($stockTexto, FILTER_VALIDATE_INT, [
    'options' => ['min_range' => 0, 'max_range' => 10000],
]);

if ($stock === false) {
    $errores['stock'] = 'El stock debe ser un entero entre 0 y 10000.';
}
```

El filtro devuelve el entero si es válido y `false` si falla. La comparación debe ser **`=== false`**: `0` es un stock válido y no debemos confundirlo con un error.

### Decimales

```php
$precio = filter_var($precioTexto, FILTER_VALIDATE_FLOAT);

if ($precio === false || $precio < 0.01 || $precio > 99999) {
    $errores['precio'] = 'El precio debe estar entre 0,01 y 99999.';
}
```

El filtro predeterminado espera un punto decimal en el texto recibido. Un control HTML `type="number"` envía su valor numérico con punto, aunque la interfaz del navegador pueda mostrar otro separador según el idioma. En pruebas manuales utilizaremos `19.90`.

Aquí el precio nos sirve para practicar validación. En aplicaciones de facturación evitaremos depender de cálculos monetarios con `float`: podremos trabajar con céntimos enteros y columnas decimales en la base de datos.

### Correo y URL

Otros filtros permiten comprobar formatos habituales:

```php
$emailValido = filter_var($email, FILTER_VALIDATE_EMAIL);
$urlValida = filter_var($url, FILTER_VALIDATE_URL);
```

También compararemos sus resultados con `=== false`. Que un correo tenga un formato válido **no demuestra que exista ni que pertenezca al usuario**. Una URL válida tampoco es automáticamente un destino seguro para descargar contenido desde el servidor. Las reglas de negocio se comprueban aparte.

### Categorías y valores permitidos

Un desplegable limita las opciones en la interfaz, pero no impide que se envíe un valor inventado.

```php
$categorias = [
    'perifericos' => 'Periféricos',
    'componentes' => 'Componentes',
    'almacenamiento' => 'Almacenamiento',
];

if (!array_key_exists($categoria, $categorias)) {
    $errores['categoria'] = 'Selecciona una categoría válida.';
}
```

La aplicación comprueba la clave contra **su propia lista de opciones**, sin confiar en el dato recibido.

## 2.5. Validar y escapar HTML son operaciones distintas

Un nombre puede cumplir las reglas del formulario y contener caracteres con significado en HTML. Por ejemplo:

```text
Monitor <Pro> de 27"
```

Si lo insertamos directamente en el documento, `<Pro>` podría interpretarse como marcado. Para mostrarlo como texto utilizamos **`htmlspecialchars` al generar la salida HTML**.

```php
echo htmlspecialchars($nombre, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
```

| Operación | Qué hace | Cuándo la aplicamos |
| --- | --- | --- |
| Normalizar | Ajustar de forma deliberada el dato, como quitar espacios de los extremos. | Al preparar los datos recibidos. |
| Validar | Comprobar reglas, formatos y límites. | Antes de aceptar y utilizar los datos. |
| Escapar HTML | Representar caracteres especiales sin que se interpreten como marcado. | Al insertar valores en HTML. |

`ENT_QUOTES` incluye la conversión de comillas simples y dobles. `ENT_SUBSTITUTE` permite sustituir secuencias de caracteres inválidas en lugar de perder toda la cadena. El símbolo `|` combina estas opciones.

Para evitar repetir la llamada completa, definiremos una función auxiliar:

```php
function e(string $texto): string
{
    return htmlspecialchars($texto, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}
```

`e` es un nombre elegido por nosotros, no una función predefinida de PHP. Dentro del HTML podremos escribir:

```php
<p><?= e($nombre) ?></p>
<input type="text" name="nombre" value="<?= e($nombre) ?>">
```

En el atributo, las comillas que rodean `value` también son necesarias. Esta función es adecuada para texto HTML y estos atributos entre comillas; **no es un escape universal para JavaScript, CSS o cualquier URL**. Tampoco evita inyección SQL: eso se tratará con consultas preparadas en la unidad de PDO.

Conservaremos los datos sin convertirlos en entidades HTML y escaparemos al mostrarlos. No guardaremos `&lt;` en lugar de `<` como una supuesta limpieza general.

## 2.6. Conservar los valores introducidos

Si el precio es incorrecto, no deberíamos obligar a escribir de nuevo el nombre y la categoría. Al volver a generar el formulario, utilizaremos las variables con los datos recibidos.

### Campos input

```php
<input type="text" name="nombre" value="<?= e($nombre) ?>">
```

### Áreas de texto

En un `textarea`, el contenido va **entre las etiquetas**, no en un atributo `value`:

```php
<textarea name="descripcion"><?= e($descripcion) ?></textarea>
```

### Desplegables

Añadimos `selected` a la opción que coincide con el valor recibido:

```php
<option value="perifericos"
    <?= $categoria === 'perifericos' ? 'selected' : '' ?>>
    Periféricos
</option>
```

### Radio y checkbox

Utilizamos `checked` para marcar el control correspondiente:

```php
<input type="radio" name="estado" value="nuevo"
    <?= $estado === 'nuevo' ? 'checked' : '' ?>>
```

Estos atributos son booleanos: se incluyen cuando corresponden. **`checked="false"` también marca el control**, porque el atributo está presente.

## 2.7. Ejemplo completo: comprobar un producto

Guarda el siguiente archivo como **`producto.php`**. Reúne lo explicado: primera visita sin errores, recogida, validación y conservación de campos. Todavía no almacena productos ni utiliza una base de datos.

```php
<?php
declare(strict_types=1);

function e(string $texto): string
{
    return htmlspecialchars($texto, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}

function campoTexto(string $clave): string
{
    $valor = $_POST[$clave] ?? '';
    return is_string($valor) ? trim($valor) : '';
}

$categorias = [
    'perifericos' => 'Periféricos',
    'componentes' => 'Componentes',
    'almacenamiento' => 'Almacenamiento',
];

// Valores iniciales: también se utilizan para volver a rellenar el formulario.
$nombre = '';
$precioTexto = '';
$stockTexto = '';
$categoria = '';
$errores = [];
$correcto = false;

if (($_SERVER['REQUEST_METHOD'] ?? '') === 'POST') {
    $nombre = campoTexto('nombre');
    $precioTexto = campoTexto('precio');
    $stockTexto = campoTexto('stock');
    $categoria = campoTexto('categoria');

    if ($nombre === '') {
        $errores['nombre'] = 'Escribe el nombre del producto.';
    } elseif (mb_strlen($nombre, 'UTF-8') < 3
        || mb_strlen($nombre, 'UTF-8') > 80) {
        $errores['nombre'] = 'El nombre debe tener entre 3 y 80 caracteres.';
    }

    $precio = filter_var($precioTexto, FILTER_VALIDATE_FLOAT);
    if ($precio === false || $precio < 0.01 || $precio > 99999) {
        $errores['precio'] = 'El precio debe estar entre 0,01 y 99999.';
    }

    $stock = filter_var($stockTexto, FILTER_VALIDATE_INT, [
        'options' => ['min_range' => 0, 'max_range' => 10000],
    ]);
    if ($stock === false) {
        $errores['stock'] = 'El stock debe ser un entero entre 0 y 10000.';
    }

    if (!array_key_exists($categoria, $categorias)) {
        $errores['categoria'] = 'Selecciona una categoría válida.';
    }

    $correcto = $errores === [];
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Comprobar producto</title>
</head>
<body>
    <h1>Comprobar producto</h1>

    <?php if ($correcto): ?>
        <p role="status">Los datos son válidos. No se ha almacenado el producto.</p>
        <ul>
            <li>Nombre: <?= e($nombre) ?></li>
            <li>Precio: <?= number_format($precio, 2, ',', '.') ?> €</li>
            <li>Stock: <?= $stock ?></li>
            <li>Categoría: <?= e($categorias[$categoria]) ?></li>
        </ul>
    <?php endif; ?>

    <form action="producto.php" method="post">
        <p>
            <label for="nombre">Nombre:</label>
            <input type="text" id="nombre" name="nombre"
                   required minlength="3" maxlength="80"
                   value="<?= e($nombre) ?>">
            <?php if (isset($errores['nombre'])): ?>
                <span role="alert"><?= e($errores['nombre']) ?></span>
            <?php endif; ?>
        </p>

        <p>
            <label for="precio">Precio (€):</label>
            <input type="number" id="precio" name="precio"
                   required min="0.01" max="99999" step="0.01"
                   value="<?= e($precioTexto) ?>">
            <?php if (isset($errores['precio'])): ?>
                <span role="alert"><?= e($errores['precio']) ?></span>
            <?php endif; ?>
        </p>

        <p>
            <label for="stock">Stock:</label>
            <input type="number" id="stock" name="stock"
                   required min="0" max="10000" step="1"
                   value="<?= e($stockTexto) ?>">
            <?php if (isset($errores['stock'])): ?>
                <span role="alert"><?= e($errores['stock']) ?></span>
            <?php endif; ?>
        </p>

        <p>
            <label for="categoria">Categoría:</label>
            <select id="categoria" name="categoria" required>
                <option value="">Selecciona una categoría</option>
                <?php foreach ($categorias as $codigo => $etiqueta): ?>
                    <option value="<?= e($codigo) ?>"
                        <?= $categoria === $codigo ? 'selected' : '' ?>>
                        <?= e($etiqueta) ?>
                    </option>
                <?php endforeach; ?>
            </select>
            <?php if (isset($errores['categoria'])): ?>
                <span role="alert"><?= e($errores['categoria']) ?></span>
            <?php endif; ?>
        </p>

        <button type="submit">Comprobar producto</button>
    </form>
</body>
</html>
```

### Cómo leer el ejemplo

* **Primera visita:** se muestran valores vacíos; el bloque POST no se ejecuta y no hay errores.
* **Envío incorrecto:** se rellenan las variables y `$errores`; el formulario vuelve a mostrar lo escrito con sus mensajes.
* **Envío correcto:** `$correcto` pasa a `true` y aparece un resumen. `$precio` y `$stock` solo se utilizan en ese resumen cuando su validación ha sido correcta.

Conservamos `$precioTexto` y `$stockTexto` por separado de los resultados de los filtros: permiten mantener el texto recibido aunque no se pueda convertir a un número válido. Si un valor manipulado es incompatible con `type="number"`, el navegador puede mostrar ese control vacío; para probar textos arbitrarios podemos cambiar temporalmente el tipo a `text`.

`role="alert"` identifica mensajes de error y `role="status"` el mensaje de resultado para tecnologías de apoyo. Las etiquetas `label` ayudan a identificar los controles.

Este ejemplo muestra el resultado directamente tras POST para centrarnos en la validación. Cuando incorporemos operaciones reales, utilizaremos la redirección que veremos al final del apartado.

## 2.8. Casillas, radios y selecciones múltiples

### Una casilla opcional

```html
<label>
    <input type="checkbox" name="destacado" value="1">
    Producto destacado
</label>
```

Si no se marca, la clave no llegará. Podemos recoger un booleano comprobando el valor esperado:

```php
$destacado = ($_POST['destacado'] ?? '') === '1';
```

Para conservarlo:

```php
<input type="checkbox" name="destacado" value="1"
       <?= $destacado ? 'checked' : '' ?>>
```

### Botones de radio

Los controles del grupo comparten `name` y tienen distintos valores:

```html
<label><input type="radio" name="estado" value="nuevo"> Nuevo</label>
<label><input type="radio" name="estado" value="reacondicionado"> Reacondicionado</label>
```

En el procesamiento, utilizando la función ya definida:

```php
$estado = campoTexto('estado');

if (!in_array($estado, ['nuevo', 'reacondicionado'], true)) {
    $errores['estado'] = 'Selecciona un estado válido.';
}
```

El tercer argumento `true` de `in_array` exige comparación estricta.

### Varias casillas con un mismo nombre

Para recoger varios valores utilizamos corchetes en `name`:

```html
<fieldset>
    <legend>Conexiones del producto</legend>
    <label><input type="checkbox" name="conexiones[]" value="usb"> USB</label>
    <label><input type="checkbox" name="conexiones[]" value="bluetooth"> Bluetooth</label>
    <label><input type="checkbox" name="conexiones[]" value="wifi"> Wi-Fi</label>
</fieldset>
```

Si se marcan USB y Wi-Fi, PHP recibe un array equivalente a:

```php
$_POST['conexiones'] = ['usb', 'wifi'];
```

En PHP la clave se consulta **sin los corchetes**:

```php
$recibidas = $_POST['conexiones'] ?? [];
$permitidas = ['usb', 'bluetooth', 'wifi'];
$conexiones = [];

if (!is_array($recibidas)) {
    $errores['conexiones'] = 'El formato de las conexiones no es válido.';
} else {
    foreach ($recibidas as $conexion) {
        if (!is_string($conexion)
            || !in_array($conexion, $permitidas, true)) {
            $errores['conexiones'] = 'Hay una conexión no permitida.';
        } else {
            $conexiones[] = $conexion;
        }
    }
    $conexiones = array_values(array_unique($conexiones));
}
```

Comprobamos tanto **la forma del conjunto** como **cada elemento**. `array_unique` elimina duplicados y `array_values` reorganiza los índices. Si hay errores, no aceptaremos el envío completo, aunque conservemos las opciones válidas para corregirlo.

Si las conexiones fueran obligatorias, añadiríamos la comprobación de que `$conexiones` no está vacío. En este ejemplo son opcionales.

Para volver a marcar una opción:

```php
<input type="checkbox" name="conexiones[]" value="usb"
       <?= in_array('usb', $conexiones, true) ? 'checked' : '' ?>>
```

Un desplegable múltiple se recoge de la misma manera:

```html
<select name="conexiones[]" multiple>
    <option value="usb">USB</option>
    <option value="bluetooth">Bluetooth</option>
    <option value="wifi">Wi-Fi</option>
</select>
```

El atributo `multiple` permite elegir varias opciones y `[]` hace que PHP las organice como un array. Para conservarlas, aplicamos `selected` a las que estén en el array validado.

## 2.9. Después de un envío correcto: POST/Redirect/GET

Si después de un POST devolvemos directamente el resultado, actualizar esa página puede provocar que el navegador proponga **reenviar el formulario**. En el ejemplo de comprobación no guardamos nada, pero en un alta real podría repetirse la operación.

Podemos organizar el proceso así:

1. **POST:** recibir y validar el formulario; si todo es correcto, realizar la operación.
2. **Redirect:** responder con una redirección.
3. **GET:** el navegador consulta una página con el resultado.

Este patrón se llama **POST/Redirect/GET (PRG)**.

En el ejemplo completo, dentro del bloque POST, sustituiríamos la asignación de `$correcto` por este bloque:

```php
if ($errores === []) {
    // Aquí realizaríamos la operación real con los datos validados.
    // En esta unidad todavía no almacenamos productos en una base de datos.
    header('Location: resultado.php', true, 303);
    exit;
}
```

El procesamiento continúa situado **antes del HTML**. No redirigimos cuando hay errores: mostramos los mensajes y conservamos el formulario en esa misma petición.

* `Location` indica el destino.
* `true` permite reemplazar una cabecera anterior del mismo tipo.
* **`303`** indica que el resultado debe consultarse en otra dirección; tras este POST el navegador la solicitará mediante GET.
* `exit` detiene el script actual.

`resultado.php` podría contener una página sencilla:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Resultado</title>
</head>
<body>
    <h1>Fin de la demostración</h1>
    <p>Esta página se consulta mediante GET. No se ha almacenado un producto.</p>
    <a href="producto.php">Volver al formulario</a>
</body>
</html>
```

Al actualizar la página final se repite el GET, no el POST anterior. Las variables del formulario **no pasan automáticamente** a `resultado.php`; con sesiones podremos conservar mensajes o datos entre peticiones.

!!! note "Alcance de este patrón"
    PRG evita repetir el POST al actualizar la página final, pero no impide por sí solo dobles clics o envíos repetidos. Abrir directamente `resultado.php` tampoco demuestra que se haya procesado antes el formulario. La validación y PRG no sustituyen la autenticación ni la protección CSRF, que incorporaremos al trabajar con sesiones.

## 2.10. Cómo comprobar el formulario

No basta con un envío correcto. Prueba también los errores y los valores límite:

| Prueba | Resultado esperado |
| --- | --- |
| Abrir el formulario por primera vez. | Campos iniciales sin errores. |
| Enviar un nombre formado solo por espacios. | Error de nombre obligatorio en el servidor. |
| Enviar un nombre de dos caracteres. | Error de longitud. |
| Enviar stock `0`. | Se acepta. |
| Enviar stock negativo o decimal. | Se rechaza. |
| Enviar precio `0`. | Se rechaza. |
| Enviar una categoría inventada. | Se rechaza en el servidor. |
| Introducir un error junto con otros campos correctos. | Se mantienen los datos válidos al mostrar los errores. |
| Escribir un nombre con comillas o `<Pro>`. | Se conserva y se muestra como texto. |
| Actualizar la página final de la variante PRG. | Se realiza un GET. |

Para comprobar la validación del servidor podemos añadir temporalmente **`novalidate`** al formulario. Esto desactiva las comprobaciones HTML durante su envío, pero no las de PHP. Después retiraremos ese atributo. Para cambiar opciones o formas de los campos podemos utilizar las herramientas del navegador; seguiremos considerando todos los datos recibidos como información que debe comprobarse.

## 2.11. Qué debemos recordar

* Recogemos los campos atendiendo a su forma: texto, casilla o conjunto de valores.
* Validamos en el servidor aunque el navegador ya haya comprobado el formulario.
* Los datos numéricos se validan antes de utilizarlos; `0` no equivale a un error.
* Mostramos mensajes por campo y conservamos lo introducido.
* Escapamos al generar HTML; validar y escapar no son lo mismo.
* En un envío correcto podemos aplicar POST/Redirect/GET.

Los ejercicios de este bloque y sus soluciones se publicarán por separado.

## Practica lo aprendido

Realiza los **ejercicios 1 a 4 del bloque 2: Formularios y validación** de la [relación de actividades](actividades.md#bloque-2-formularios-y-validacion).

Comenzaremos los ejercicios en clase y completarás en casa lo pendiente. Revisaremos las soluciones una a una y mantendremos las del profesor separadas de los enunciados.

## Referencias

* [Aitor Medrano y Luis Alemañ: Programación web](https://aitor-medrano.github.io/dwes2122/04web.html), apartados de formularios, validación, parámetros multivalor y conservación de campos.
* [Manual de PHP: `filter_var`](https://www.php.net/manual/es/function.filter-var.php).
* [Manual de PHP: filtros de validación](https://www.php.net/manual/es/filter.constants.php).
* [Manual de PHP: `htmlspecialchars`](https://www.php.net/manual/es/function.htmlspecialchars.php).
* [Manual de PHP: `mb_strlen`](https://www.php.net/manual/es/function.mb-strlen.php).
* [Manual de PHP: `header`](https://www.php.net/manual/es/function.header.php).

## Autoría y licencia

Material elaborado por **Joaquín García** para el módulo de **Desarrollo Web en Entorno Servidor** de 2.º DAW del **IES Portada Alta**.

Este material adapta y amplía contenidos de *Desarrollo Web en Entorno Servidor*, elaborados por **Aitor Medrano** y **Luis Alemañ**, publicados bajo licencia **Creative Commons Reconocimiento-NoComercial-CompartirIgual**. Los ejemplos propios se apoyan en la documentación de PHP.

Esta adaptación se distribuye bajo licencia **Creative Commons Reconocimiento-NoComercial-CompartirIgual (CC BY-NC-SA)**.
