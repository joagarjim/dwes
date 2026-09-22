# 5. Arrays en PHP

Hasta ahora hemos guardado cada dato en una variable. Cuando queremos trabajar con varios valores relacionados, un **array** permite reunirlos y recorrerlos sin escribir una instrucción por cada elemento. En este apartado veremos arrays indexados, arrays asociativos y arrays que contienen otros arrays.

PHP representa los arrays como conjuntos ordenados de pares **clave → valor**. Podemos usar claves numéricas y claves de texto, incluso en el mismo array. Para aprender a manejarlos, empezaremos por listas con índices consecutivos y seguiremos con claves descriptivas.

## 5.1. Crear y consultar un array indexado

```php
<?php
$lenguajes = ['PHP', 'JavaScript', 'Python'];

echo $lenguajes[0];       // PHP
echo $lenguajes[2];       // Python
echo count($lenguajes);   // 3 elementos
```

Los corchetes `[]` son la sintaxis que usaremos habitualmente. También encontrarás `array('PHP', 'JavaScript', 'Python')` en proyectos existentes. Podemos empezar con un array vacío y asignar posiciones después:

```php
<?php
$otros = [];
$otros[0] = 'Java';
$otros[1] = 'C#';
$otros[] = 'Python'; // Recibe la siguiente clave numérica.
```

En estos ejemplos, los índices empiezan en `0`: `0`, `1` y `2`. `count()` devuelve el número de elementos, no el último índice. Acceder a una posición que no existe produce un aviso; por eso debemos controlar los límites.

Podemos modificar un elemento o añadir otro al final:

```php
<?php
$lenguajes[1] = 'TypeScript';
$lenguajes[] = 'Java';
```

La expresión `[]` añade un valor con la siguiente clave numérica disponible. En los ejemplos de listas de esta sección utilizaremos índices consecutivos; no supongas que todos los arrays de PHP los tienen.

PHP permite mezclar tipos de valores, pero para una colección con un mismo propósito conviene mantener una estructura coherente: una lista de nombres contiene nombres; una lista de personas contiene registros con las mismas claves.

## 5.2. Recorrer una lista: `for` y `foreach`

Cuando conocemos que los índices van de `0` a `count($lenguajes) - 1`, podemos usar `for`:

```php
<?php
for ($i = 0; $i < count($lenguajes); $i++) {
    echo '<p>' . htmlspecialchars($lenguajes[$i], ENT_QUOTES, 'UTF-8') . '</p>';
}
```

Para recorrer los valores sin gestionar índices, `foreach` resulta más directo:

```php
<ul>
    <?php foreach ($lenguajes as $lenguaje) { ?>
        <li><?= htmlspecialchars($lenguaje, ENT_QUOTES, 'UTF-8') ?></li>
    <?php } ?>
</ul>
```

En cada vuelta, `$lenguaje` recibe el siguiente valor. El array no desaparece ni se modifica por el mero hecho de recorrerlo. `foreach` funciona también con arrays cuyas claves no son números consecutivos. Seguimos escapando los textos antes de mostrarlos en HTML, especialmente si proceden de un formulario.

## 5.3. Arrays asociativos: claves con nombre

En lugar de una posición, podemos utilizar una clave que describa cada dato:

```php
<?php
$persona = [
    'nombre' => 'Lucía',
    'edad' => 20,
    'ciudad' => 'Málaga',
];

echo $persona['nombre']; // Lucía
$persona['edad'] = 21;                     // Modifica una clave existente.
$persona['correo'] = 'lucia@example.com';  // Crea una clave nueva.
```

Al asignar un valor a `$persona['edad']`, sustituimos el `20` por `21`. Como `'correo'` todavía no existe, la asignación incorpora esa clave al array. A partir de entonces podemos consultar `$persona['correo']`.

Podemos recuperar tanto la clave como el valor al recorrerlo:

```php
<?php
foreach ($persona as $campo => $valor) {
    echo '<p>' . htmlspecialchars($campo, ENT_QUOTES, 'UTF-8')
       . ': ' . htmlspecialchars((string) $valor, ENT_QUOTES, 'UTF-8')
       . '</p>';
}
```

La sintaxis `$campo => $valor` significa que en cada vuelta recibimos ambos. No utilizaremos un `for` con posiciones para recorrer este array: sus claves son palabras. Los arrays `$_GET` y `$_POST` que ya hemos usado también permiten acceder a valores mediante el nombre del campo.

Para añadir un país y su capital, escribiríamos `$capitales['Portugal'] = 'Lisboa';`. Si usamos `$capitales[] = 'Lisboa';`, el nuevo valor recibirá una **clave numérica**, no la clave `'Portugal'`. Por eso, en un array asociativo indicamos siempre la clave que queremos utilizar.

## 5.4. Operaciones y funciones habituales con arrays

Ya sabemos crear y recorrer arrays. Ahora veremos cómo inspeccionarlos, añadir y quitar elementos, buscar valores y ordenarlos. Algunas de estas operaciones son funciones (`count()`, `sort()`); `isset` y `unset` son construcciones del lenguaje. Nos centraremos en lo que hace cada una y en si modifica el array original.

### Ver el contenido durante el desarrollo

`print_r()` muestra las claves y los valores de un array. `var_dump()` añade información sobre los tipos, como vimos en el apartado 1:

```php
<?php $frutas = ['pera', 'manzana', 'naranja']; ?>

<h3>Salida de print_r()</h3>
<pre><?php print_r($frutas); ?></pre>

<h3>Salida de var_dump()</h3>
<pre><?php var_dump($frutas); ?></pre>
```

La primera salida indica las claves y sus valores:

```text
Array
(
    [0] => pera
    [1] => manzana
    [2] => naranja
)
```

La segunda también indica el tipo y la longitud de cada cadena:

```text
array(3) {
  [0]=>
  string(4) "pera"
  [1]=>
  string(7) "manzana"
  [2]=>
  string(7) "naranja"
}
```

`array(3)` significa que hay tres elementos; `string(7)` indica que `'manzana'` ocupa siete bytes. El elemento HTML `<pre>` conserva los saltos de línea y espacios de estas salidas al verlas en el navegador. `print_r()` y `var_dump()` sirven para investigar el contenido durante el desarrollo; para presentar los datos al visitante, recorreremos el array y construiremos el HTML adecuado.

### Contar, añadir y extraer

```php
<?php
$frutas = ['pera', 'manzana'];
echo count($frutas); // 2

$frutas[] = 'naranja';              // ['pera', 'manzana', 'naranja']
array_push($frutas, 'plátano');     // ['pera', 'manzana', 'naranja', 'plátano']

$ultima = array_pop($frutas);
echo $ultima;        // plátano
echo count($frutas); // 3: 'plátano' ya no está en el array.
```

`$frutas[] = ...` y `array_push()` añaden elementos; para añadir **uno solo** preferimos `[]` por sencillez. `array_pop()` hace dos cosas: quita el último elemento del array original y lo devuelve, por eso podemos guardarlo en `$ultima`.

### Buscar un valor y obtener las claves

```php
<?php
$valores = [10, 20, 30];
var_dump(in_array(20, $valores, true));   // true
var_dump(in_array('20', $valores, true)); // false: es una cadena.

$capitales = ['España' => 'Madrid', 'Italia' => 'Roma'];
$paises = array_keys($capitales);
print_r($paises); // [0] => España, [1] => Italia
```

`in_array()` busca **entre los valores**, no entre las claves. El tercer argumento `true` exige igualdad de valor y tipo. `array_keys()` crea un array nuevo con las claves: `$capitales` sigue conteniendo sus países y capitales.

### Comprobar y eliminar una clave

```php
<?php
$persona = ['nombre' => 'Ana', 'apodo' => null];

var_dump(isset($persona['nombre']));            // true
var_dump(isset($persona['apodo']));             // false: su valor es null.
var_dump(array_key_exists('apodo', $persona));  // true: la clave sí existe.
var_dump(isset($persona['ciudad']));            // false: no existe.

unset($persona['nombre']);
var_dump(isset($persona['nombre']));            // false: ya se eliminó.
```

`isset()` es útil cuando nos importa que exista un valor distinto de `null`. `array_key_exists()` distingue una clave presente con valor `null` de una clave ausente. `unset()` modifica el array original.

En una lista numérica, eliminar un elemento **no renumera** los demás:

```php
<?php
$numeros = [10, 20, 30];
unset($numeros[1]);
print_r($numeros); // [0] => 10, [2] => 30
echo count($numeros); // 2 elementos, pero la clave 1 no existe.

$renumerados = array_values($numeros);
print_r($renumerados); // [0] => 10, [1] => 30
```

Por eso `count($numeros) - 1` ya no indica necesariamente el último índice. Un `for` que intente acceder a `$numeros[1]` produciría un aviso; `foreach` recorre los elementos existentes. `array_values()` crea una nueva lista con índices consecutivos.

### Ordenar una lista

```php
<?php
$nombres = ['Luis', 'Ana', 'Marta'];
sort($nombres);
print_r($nombres); // [0] => Ana, [1] => Luis, [2] => Marta
```

`sort()` cambia `$nombres` directamente: ordena sus **valores** y asigna índices numéricos consecutivos. Es adecuado para esta lista. En un array asociativo perderíamos la relación entre las claves originales y sus valores; si queremos conservarla al ordenar por valores, podemos usar `asort()`.

### Copiar un array

```php
<?php
$original = ['Luis', 'Ana', 'Marta'];
$copia = $original;
$copia[0] = 'Elena';

print_r($original); // [0] => Luis,  [1] => Ana, [2] => Marta
print_r($copia);    // [0] => Elena, [1] => Ana, [2] => Marta
```

La asignación deja dos variables con el mismo contenido inicial. Si modificamos un elemento de `$copia`, `$original` conserva su valor. Hemos separado este ejemplo del de `sort()` para observar una sola operación cada vez.

## 5.5. Arrays de varias dimensiones

Un array puede contener otros arrays. Podemos mezclar una clave descriptiva con una lista de valores:

```php
<?php
$contacto = [
    'nombre' => 'Ana',
    'telefonos' => ['600 111 222', '600 333 444'],
];

echo $contacto['telefonos'][0]; // Primer teléfono.
```

También podemos construir una cuadrícula con índices de fila y columna:

```php
<?php
$asientos = [
    ['A1', 'A2'],
    ['B1', 'B2'],
];

echo $asientos[1][0]; // B1: segunda fila, primera columna.
```

Cada pareja de corchetes baja un nivel: primero elegimos el array interior y después uno de sus elementos. Para recorrer una cuadrícula podemos anidar dos `foreach`: el exterior recorre las filas y el interior sus celdas.

Otra aplicación habitual es una lista de registros. Cada elemento del siguiente ejemplo representa una persona:

```php
<?php
$personas = [
    ['nombre' => 'Ana', 'ciudad' => 'Málaga'],
    ['nombre' => 'Luis', 'ciudad' => 'Jaén'],
];

echo $personas[0]['nombre']; // Ana
```

Podemos convertir esos datos en filas de una tabla:

```php
<table>
    <thead><tr><th>Nombre</th><th>Ciudad</th></tr></thead>
    <tbody>
        <?php foreach ($personas as $persona) { ?>
            <tr>
                <td><?= htmlspecialchars($persona['nombre'], ENT_QUOTES, 'UTF-8') ?></td>
                <td><?= htmlspecialchars($persona['ciudad'], ENT_QUOTES, 'UTF-8') ?></td>
            </tr>
        <?php } ?>
    </tbody>
</table>
```

El bucle genera una fila por persona. La forma de los datos queda separada del HTML fijo de la tabla. Cuando cada registro tiene las mismas claves, resulta sencillo añadir más personas sin duplicar filas en el código.

Si los datos tienen más profundidad, podemos combinar recorridos:

```php
<?php
$menus = [
    ['primero' => 'Sopa', 'segundo' => 'Pescado'],
    ['primero' => 'Ensalada', 'segundo' => 'Pasta'],
];

foreach ($menus as $menu) {
    foreach ($menu as $tipo => $plato) {
        echo '<p>' . htmlspecialchars($tipo, ENT_QUOTES, 'UTF-8')
           . ': ' . htmlspecialchars($plato, ENT_QUOTES, 'UTF-8') . '</p>';
    }
}
```

En el siguiente apartado estudiaremos funciones para organizar mejor programas que crezcan; no hace falta convertir cada array de varias dimensiones en una estructura todavía más compleja.

## Comprueba lo aprendido

Antes de continuar, deberías poder distinguir una clave de un valor, explicar cuándo `count($lista) - 1` deja de ser el último índice, elegir entre `for` y `foreach`, reconocer qué operaciones modifican el array, acceder a datos con varios niveles de corchetes y generar una tabla a partir de varios registros.

## Practica lo aprendido

Realiza los **ejercicios 21 a 28** del [bloque 5: Arrays](actividades.md#bloque-5-arrays). Entrega juntos los archivos del bloque en la tarea correspondiente de Moodle.

## Fuentes de consulta

- [Aitor Medrano: El lenguaje PHP](https://aitor-medrano.github.io/dwes2122/02php.html), apartados «Arrays» y «Actividades: Arrays».
- Manual oficial de PHP: [arrays](https://www.php.net/manual/es/language.types.array.php), [`foreach`](https://www.php.net/manual/es/control-structures.foreach.php), [`count()`](https://www.php.net/manual/es/function.count.php) y [funciones de arrays](https://www.php.net/manual/es/ref.array.php).
