# 7. Funciones predefinidas de PHP

En el bloque anterior aprendimos a crear funciones. PHP también proporciona funciones ya definidas para realizar tareas habituales: trabajar con textos, calcular, comprobar tipos y presentar datos.

Ya hemos utilizado varias, como `count()`, `trim()`, `rand()` y `htmlspecialchars()`. Ahora ampliaremos ese repertorio. No necesitamos memorizar todas las funciones del lenguaje: debemos comprender qué necesitamos hacer, reconocer las más habituales y saber consultar su documentación.

## 7.1. Cómo utilizar una función predefinida

Utilizamos su nombre, enviamos los argumentos necesarios y recogemos su resultado:

```php
<?php
$texto = 'Hola';
$longitud = strlen($texto);
echo $longitud;
```

**Salida:**

```text
4
```

`strlen()` recibe una cadena y devuelve su longitud en bytes. En este texto, cada letra ocupa un byte, así que el resultado coincide con el número de letras. Después veremos qué sucede con las tildes.

Muchas funciones de este apartado **devuelven un resultado sin modificar la variable original**:

```php
<?php
$nombre = '  Ana  ';
$limpio = trim($nombre);

echo '[' . $nombre . ']'; // [  Ana  ]
echo '[' . $limpio . ']'; // [Ana]
```

Para sustituir el dato original, debemos asignar el resultado: `$nombre = trim($nombre);`. Esto es distinto de funciones como `sort()`, que sí modifican el array recibido.

En los ejemplos, usaremos `echo` para resultados sencillos y `var_dump()` cuando queramos distinguir tipos o ver `true` y `false`. Para ver claramente la estructura de un array en el navegador, usaremos `print_r()` dentro de `<pre>`.

## 7.2. Funciones para trabajar con cadenas

### 7.2.1. Eliminar espacios de los extremos: `trim()`

`trim()` elimina los espacios y determinados caracteres de separación, como saltos de línea, al principio y al final. No elimina los espacios entre palabras.

```php
<?php
$texto = '   Ana María   ';
$resultado = trim($texto);

echo '[' . $resultado . ']';
```

**Salida:**

```text
[Ana María]
```

Los corchetes permiten observar dónde comienza y termina el resultado. Los espacios exteriores desaparecen y el espacio entre `Ana` y `María` permanece.

Es útil antes de comprobar un campo de formulario:

```php
<?php
$nombre = trim('    ');

if ($nombre === '') {
    echo 'El nombre está vacío.';
} else {
    echo 'El nombre contiene texto.';
}
```

**Salida:** `El nombre está vacío.`

Existen dos variantes: `ltrim()` elimina por la izquierda y `rtrim()` por la derecha.

```php
<?php
$texto = '  Ana  ';
echo '[' . ltrim($texto) . ']'; // [Ana  ]
echo '[' . rtrim($texto) . ']'; // [  Ana]
```

### 7.2.2. Medir un texto: `strlen()` y `mb_strlen()`

`strlen()` cuenta **bytes**. Para textos ASCII como `PHP`, coincide con el número de caracteres:

```php
<?php
echo strlen('PHP'); // 3
echo strlen('Hola mundo'); // 10: el espacio también cuenta.
```

En UTF-8, una letra con tilde puede ocupar varios bytes. `mb_strlen()` permite contar caracteres según la codificación indicada:

```php
<?php
$ciudad = 'Málaga';

echo strlen($ciudad);              // 7 bytes.
echo mb_strlen($ciudad, 'UTF-8');   // 6 caracteres.
```

`Málaga` tiene seis letras, pero `á` ocupa dos bytes en UTF-8. Para medir nombres o textos con tildes, preferiremos `mb_strlen()`.

Las funciones `mb_*` requieren la extensión **mbstring**. Si PHP indica que `mb_strlen()` no está definida, la extensión no está disponible en ese entorno. No sustituyas una función por la otra suponiendo que cuentan lo mismo.

### 7.2.3. Cambiar mayúsculas y minúsculas

`strtoupper()` convierte las letras ASCII en mayúsculas; `strtolower()` las convierte en minúsculas:

```php
<?php
$texto = 'Hola PHP';

echo strtoupper($texto); // HOLA PHP
echo strtolower($texto); // hola php
echo $texto;             // Hola PHP: el original no cambia.
```

Para letras como `á` y `ñ`, utilizaremos las variantes multibyte:

```php
<?php
$nombre = 'María Muñoz';

echo mb_strtoupper($nombre, 'UTF-8'); // MARÍA MUÑOZ
echo mb_strtolower($nombre, 'UTF-8'); // maría muñoz
```

Para observar la diferencia directamente, aplicamos las dos funciones **al mismo texto**:

```php
<?php $texto = 'Málaga y España'; ?>

<p>Con strtoupper(): <?= strtoupper($texto) ?></p>
<p>Con mb_strtoupper(): <?= mb_strtoupper($texto, 'UTF-8') ?></p>
```

**En el navegador:**

```text
Con strtoupper(): MáLAGA Y ESPAñA
Con mb_strtoupper(): MÁLAGA Y ESPAÑA
```

`strtoupper()` convierte las letras ASCII, pero deja `á` y `ñ` tal como estaban. `mb_strtoupper()` también convierte esas letras en `Á` y `Ñ`. Para nombres y textos en español, utilizaremos la variante `mb_*` con UTF-8.

La misma diferencia se observa al pasar a minúsculas:

```php
<?php
$texto = 'MÁLAGA Y ESPAÑA';
echo strtolower($texto);              // mÁlaga y espaÑa
echo mb_strtolower($texto, 'UTF-8');   // málaga y españa
```

### 7.2.4. Extraer una parte: `substr()` y `mb_substr()`

`substr($texto, $inicio, $longitud)` devuelve un fragmento. La primera posición es `0`; si omitimos la longitud, obtiene el resto del texto:

```php
<?php
$codigo = 'ABC12345';

echo substr($codigo, 0, 3);  // ABC
echo substr($codigo, 3);     // 12345
echo substr($codigo, -2);    // 45
```

En la primera llamada, empezamos en la posición `0` y tomamos tres bytes. En la segunda, empezamos en la posición `3`. Un inicio negativo cuenta desde el final.

`substr()` trabaja por bytes. Para no cortar en medio de una letra con tilde, usaremos `mb_substr()`:

```php
<?php
$ciudad = 'Málaga';
echo mb_substr($ciudad, 0, 2, 'UTF-8'); // Má
```

### 7.2.5. Buscar una posición: `strpos()`

`strpos($texto, $buscado)` devuelve la posición de la primera coincidencia o `false` si no la encuentra. Distingue mayúsculas y minúsculas.

```php
<?php
$texto = 'PHP y Laravel';

var_dump(strpos($texto, 'PHP'));     // int(0)
var_dump(strpos($texto, 'Laravel')); // int(6)
var_dump(strpos($texto, 'Python'));  // bool(false)
```

Para comprobar si encontró el texto, usamos **`!== false`**, porque la posición `0` es una coincidencia válida:

```php
<?php
$posicion = strpos('PHP y Laravel', 'PHP');

if ($posicion !== false) {
    echo 'Encontrado en la posición ' . $posicion;
} else {
    echo 'No encontrado.';
}
```

**Salida:** `Encontrado en la posición 0`.

Un `if ($posicion)` trataría el `0` como falso y dejaría pasar una coincidencia al principio. Aquí debemos distinguir **una posición entera** de **`false`**. Con texto UTF-8, `strpos()` devuelve posiciones en bytes; `mb_strpos()` permite trabajar en caracteres.

### 7.2.6. Comprobar si contiene, empieza o termina

Si solo queremos una respuesta `true` o `false`, PHP proporciona `str_contains()`, `str_starts_with()` y `str_ends_with()`:

```php
<?php
$archivo = 'ejercicio.php';

var_dump(str_contains($archivo, 'ejercicio')); // bool(true)
var_dump(str_starts_with($archivo, 'ej'));     // bool(true)
var_dump(str_ends_with($archivo, '.php'));     // bool(true)
var_dump(str_ends_with($archivo, '.html'));    // bool(false)
```

- `str_contains()` busca el fragmento en cualquier lugar.
- `str_starts_with()` comprueba el comienzo.
- `str_ends_with()` comprueba el final.

Las tres distinguen mayúsculas y minúsculas. Se incorporaron en PHP 8 y están disponibles en el entorno de PHP 8.4 que utilizamos.

### 7.2.7. Sustituir texto: `str_replace()`

`str_replace($buscado, $sustituto, $texto)` sustituye todas las coincidencias y devuelve la nueva cadena:

```php
<?php
$texto = 'Aprendo PHP. PHP genera páginas.';
$resultado = str_replace('PHP', 'Laravel', $texto);

echo $resultado;
```

**Salida:**

```text
Aprendo Laravel. Laravel genera páginas.
```

El orden es importante: primero **qué buscamos**, después **por qué lo sustituimos** y finalmente **en qué texto**.

Podemos obtener también el número de sustituciones mediante un cuarto argumento:

```php
<?php
$texto = 'rojo, azul, rojo';
$resultado = str_replace('rojo', 'verde', $texto, $cantidad);

echo $resultado; // verde, azul, verde
echo $cantidad;  // 2
```

`$cantidad` recibe el número de cambios; el texto original permanece igual. `str_replace()` distingue mayúsculas y minúsculas.

### 7.2.8. Dividir una cadena: `explode()`

`explode($separador, $texto)` devuelve un array con los fragmentos delimitados por el separador:

```php
<?php
$texto = 'PHP,JavaScript,Python';
$lenguajes = explode(',', $texto);
?>
<pre><?php print_r($lenguajes); ?></pre>
```

**Salida:**

```text
Array
(
    [0] => PHP
    [1] => JavaScript
    [2] => Python
)
```

El primer argumento es la coma que marca los cortes. La coma no forma parte de los elementos devueltos. Si hay espacios después de las comas, esos espacios sí permanecen; podemos quitarlos al recorrer los valores:

```php
<?php
$lenguajes = explode(',', 'PHP, JavaScript, Python');
?>
<ul>
    <?php foreach ($lenguajes as $lenguaje) { ?>
        <li><?= htmlspecialchars(trim($lenguaje), ENT_QUOTES, 'UTF-8') ?></li>
    <?php } ?>
</ul>
```

**En el navegador:** una lista con `PHP`, `JavaScript` y `Python`.

### 7.2.9. Unir un array: `implode()`

`implode($separador, $array)` une los valores de un array y devuelve una cadena:

```php
<?php
$lenguajes = ['PHP', 'JavaScript', 'Python'];
$texto = implode(' · ', $lenguajes);
echo $texto;
```

**Salida:** `PHP · JavaScript · Python`.

`explode()` pasa de cadena a array; `implode()` pasa de array a cadena. En esta unidad utilizaremos arrays de cadenas para este tipo de operación.

### 7.2.10. Mostrar texto en HTML: `htmlspecialchars()`

Esta función convierte caracteres con significado especial en HTML, como `<`, `>` y `&`, en entidades. Permite mostrar el texto sin que el navegador lo interprete como etiquetas:

```php
<?php
$texto = '<strong>Ana</strong>';
?>
<p><?= htmlspecialchars($texto, ENT_QUOTES, 'UTF-8') ?></p>
```

**En el navegador:**

```text
<strong>Ana</strong>
```

El navegador muestra las etiquetas como texto, en lugar de poner `Ana` en negrita. En el HTML generado aparecen `&lt;strong&gt;Ana&lt;/strong&gt;`.

`ENT_QUOTES` convierte también las comillas simples y dobles; `'UTF-8'` indica la codificación. Escaparemos los textos al insertarlos en HTML, especialmente si proceden de formularios. Escapar no equivale a comprobar que el contenido cumpla nuestros requisitos ni es una protección universal para cualquier contexto: aquí estamos presentando texto en HTML.

### 7.2.11. Conservar saltos de línea: `nl2br()`

Un salto de línea en una cadena no produce por sí mismo un salto visible en un párrafo HTML. `nl2br()` inserta etiquetas `<br>` antes de los saltos de línea:

```php
<?php
$mensaje = "Primera línea\nSegunda línea";
?>
<p><?= nl2br(htmlspecialchars($mensaje, ENT_QUOTES, 'UTF-8')) ?></p>
```

**En el navegador:**

```text
Primera línea
Segunda línea
```

Primero escapamos el texto y después añadimos los `<br>` mediante `nl2br()`. Así mostramos el texto recibido sin interpretar sus posibles etiquetas y conservamos los saltos de línea.

## 7.3. Funciones matemáticas y presentación de números

### 7.3.1. Valor absoluto: `abs()`

Devuelve la distancia de un número a cero, sin signo negativo:

```php
<?php
echo abs(-8); // 8
echo abs(8);  // 8
echo abs(0);  // 0
```

Es útil, por ejemplo, para obtener la diferencia absoluta entre dos alturas:

```php
<?php
$altura1 = 165;
$altura2 = 180;
echo abs($altura1 - $altura2); // 15
```

### 7.3.2. Potencias y raíces: `pow()` y `sqrt()`

`pow($base, $exponente)` calcula una potencia. `sqrt($numero)` calcula la raíz cuadrada:

```php
<?php
echo pow(2, 3); // 8: 2 × 2 × 2.
echo pow(7, 0); // 1.
echo sqrt(81);  // 9.
```

La potencia también puede escribirse con el operador `**`: `2 ** 3`. En el ejercicio 20 la calculamos mediante un bucle para practicar la acumulación; ahora conocemos la función que permite hacerlo directamente.

### 7.3.3. Redondear: `round()`, `floor()` y `ceil()`

`round()` redondea al valor más cercano. Su segundo argumento indica cuántos decimales conservamos:

```php
<?php
echo round(7.46);    // 7
echo round(7.56);    // 8
echo round(7.456, 2); // 7.46
```

`floor()` redondea hacia abajo y `ceil()` hacia arriba, hacia un valor entero:

```php
<?php
echo floor(7.9);  // 7
echo ceil(7.1);   // 8
echo floor(-7.1); // -8
echo ceil(-7.1);  // -7
```

Con negativos, «hacia abajo» significa acercarse a un número menor, no quitar simplemente los decimales. `floor()` y `ceil()` devuelven valores `float`, aunque al mostrarlos con `echo` no aparezcan decimales.

### 7.3.4. Menor y mayor: `min()` y `max()`

Pueden recibir varios números o un array de números:

```php
<?php
echo min(8, 3, 12); // 3
echo max(8, 3, 12); // 12

$numeros = [10, 4, 18, 7];
echo min($numeros); // 4
echo max($numeros); // 18
```

En el ejercicio 22 hicimos estas operaciones mediante comparaciones para aprender el algoritmo. Cuando el objetivo sea resolver una tarea real, podemos aprovechar las funciones disponibles. No les pasaremos un array vacío: necesitan valores que comparar.

### 7.3.5. Números aleatorios: `rand()`

`rand($minimo, $maximo)` devuelve un entero aleatorio entre ambos extremos, **incluidos**:

```php
<?php
$dado = rand(1, 6);
echo $dado;
```

**Posible salida:** `4`. Otras ejecuciones pueden producir cualquier entero del `1` al `6`, y también pueden repetir el resultado anterior.

Para elegir un elemento de una lista con índices consecutivos:

```php
<?php
$respuestas = ['Sí', 'No', 'Quizás'];
$indice = rand(0, count($respuestas) - 1);
echo $respuestas[$indice];
```

**Posible salida:** `Quizás`. Los índices válidos son `0`, `1` y `2`.

### 7.3.6. Formatear para mostrar: `number_format()`

Prepara un número con una cantidad fija de decimales y los separadores indicados:

```php
<?php
$precio = 1234.567;
$texto = number_format($precio, 2, ',', '.');

echo $texto; // 1.234,57
var_dump($texto); // string(8) "1.234,57"
```

Los argumentos significan:

1. `$precio`: número que queremos presentar.
2. `2`: dos cifras decimales, redondeando para la presentación.
3. `','`: separador decimal.
4. `'.'`: separador de miles.

La función devuelve una **cadena**, no un número para seguir calculando. `$precio` mantiene su valor numérico original. Realizaremos los cálculos con los números y aplicaremos el formato al mostrarlos:

```php
<?php
$precio = 80;
$descuento = 25;
$precioFinal = $precio * (1 - $descuento / 100);
?>
<p>Precio final: <?= number_format($precioFinal, 2, ',', '.') ?> €</p>
```

**En el navegador:** `Precio final: 60,00 €`.

## 7.4. Comprobar y convertir tipos

### 7.4.1. Ver el tipo: `gettype()` y `var_dump()`

`gettype()` devuelve el nombre del tipo. `var_dump()` muestra el tipo y el valor:

```php
<?php
$edad = 20;
$precio = 12.5;
$nombre = 'Ana';

echo gettype($edad);   // integer
echo gettype($precio); // double: nombre que gettype() utiliza para float.
echo gettype($nombre); // string

var_dump($edad);       // int(20)
var_dump($precio);     // float(12.5)
```

Sirven para entender qué dato tenemos, especialmente cuando algo no se comporta como esperábamos.

### 7.4.2. Preguntar por un tipo concreto

Estas funciones devuelven `true` o `false`:

```php
<?php
var_dump(is_int(20));          // bool(true)
var_dump(is_int('20'));        // bool(false)
var_dump(is_float(20.5));      // bool(true)
var_dump(is_float(20));        // bool(false)
var_dump(is_string('20'));     // bool(true)
var_dump(is_bool(false));      // bool(true): false es un booleano.
var_dump(is_array(['PHP']));   // bool(true)
var_dump(is_null(null));      // bool(true)
```

**El contenido y el tipo son cosas distintas.** `'20'` parece un número al leerlo, pero es una cadena porque está entre comillas. `is_int()` comprueba el tipo, sin convertir el dato.

### 7.4.3. Comprobar contenido numérico: `is_numeric()`

Devuelve `true` si el dato es un número o una cadena numérica:

```php
<?php
var_dump(is_numeric(20));     // bool(true)
var_dump(is_numeric('20'));   // bool(true)
var_dump(is_numeric('12.5')); // bool(true)
var_dump(is_numeric('-3'));   // bool(true)
var_dump(is_numeric('hola')); // bool(false)
var_dump(is_numeric(''));     // bool(false)
var_dump(is_numeric('12,5')); // bool(false)
```

`is_numeric('20')` y `is_int('20')` responden a preguntas diferentes: el primero comprueba si la cadena representa un número; el segundo comprueba si el dato ya tiene tipo entero.

Para cadenas numéricas, PHP utiliza el punto decimal. La coma que usamos con `number_format()` es un recurso de presentación y no transforma el texto en un número válido para PHP.

### 7.4.4. Comprobar dígitos: `ctype_digit()`

Cuando necesitamos una cadena formada exclusivamente por dígitos del `0` al `9`, podemos usar `ctype_digit()`:

```php
<?php
var_dump(ctype_digit('20'));   // bool(true)
var_dump(ctype_digit('0'));    // bool(true)
var_dump(ctype_digit('-3'));   // bool(false): contiene un signo.
var_dump(ctype_digit('12.5')); // bool(false): contiene un punto.
var_dump(ctype_digit(''));     // bool(false)
```

En nuestros ejemplos le pasaremos cadenas. Es útil para validar enteros no negativos recibidos mediante un formulario; no acepta signos ni decimales. `is_numeric()` acepta un repertorio más amplio, como decimales o notación científica; no garantiza que el dato sea un entero.

### 7.4.5. Convertir después de comprobar

Los operadores `(int)`, `(float)` y `(string)` convierten un valor. **No son funciones de validación**:

```php
<?php
$texto = '20';
$edad = (int) $texto;

var_dump($texto); // string(2) "20"
var_dump($edad);  // int(20)

$precio = (float) '12.5';
var_dump($precio); // float(12.5)
```

Convertir no demuestra que el dato original fuera adecuado:

```php
<?php
var_dump((int) 'hola'); // int(0)
var_dump((int) 12.9);   // int(12): trunca los decimales, no redondea.
```

Si convertimos `'hola'` antes de comprobarlo, obtenemos `0` y perdemos la información que nos permitía detectar el error. Por eso seguimos este orden: **recibir → comprobar → convertir → utilizar**.

El siguiente fragmento procesa una edad recibida mediante POST:

```php
<?php
$dato = $_POST['edad'] ?? '';

if (!is_string($dato) || !ctype_digit($dato)) {
    echo 'Introduce una edad con dígitos.';
} elseif ((int) $dato > 120) {
    echo 'La edad debe estar entre 0 y 120.';
} else {
    $edad = (int) $dato;
    echo 'Edad válida: ' . $edad;
}
```

Con `'20'`, muestra `Edad válida: 20`; con `'hola'`, el primer error; con `'150'`, el segundo. Este fragmento corresponde al procesamiento del envío: para una página completa, lo colocaríamos dentro de la comprobación de que se ha recibido una petición POST.

## 7.5. Consultar el manual y elegir una función

Cuando consultes una función en el manual oficial, localiza:

1. **Qué hace** y si modifica el dato original o devuelve uno nuevo.
2. **Qué argumentos recibe**, en qué orden y cuáles son opcionales.
3. **Qué devuelve**, incluidos los valores especiales como `false`.
4. **Un ejemplo** y las observaciones sobre codificación o tipos.

Por ejemplo, para buscar con `strpos()`, saber que devuelve `int` o `false` es tan importante como saber que busca texto: determina cómo debemos escribir el `if`.

Algunas funciones básicas se incluyen con PHP; otras requieren extensiones, como las funciones `mb_*` que hemos visto. No es necesario aprender toda la biblioteca para empezar: elegiremos la función que resuelve cada tarea y consultaremos sus detalles cuando haga falta.

## Comprueba lo aprendido

Antes de continuar, deberías poder:

* Diferenciar el número de bytes del número de caracteres en un texto con tildes.
* Limpiar, transformar, extraer, buscar y sustituir texto.
* Pasar de una cadena a un array y volver a unir sus valores.
* Explicar por qué `strpos()` se comprueba con `!== false`.
* Escapar texto para mostrarlo en HTML y conservar sus saltos de línea.
* Distinguir redondear un número de formatearlo para su presentación.
* Diferenciar `is_int()`, `is_numeric()` y `ctype_digit()`.
* Comprobar un dato antes de convertirlo.

## Practica lo aprendido

Realiza los **ejercicios 38 a 45** del [bloque 7: Funciones predefinidas de PHP](actividades.md#bloque-7-funciones-predefinidas-de-php). Entrega juntos los archivos en la tarea correspondiente de Moodle.

## Fuentes de consulta

- Manual de PHP: [funciones de cadenas](https://www.php.net/manual/es/ref.strings.php) y [funciones multibyte](https://www.php.net/manual/es/ref.mbstring.php).
- Manual de PHP: [funciones matemáticas](https://www.php.net/manual/es/ref.math.php).
- Manual de PHP: [funciones de variables](https://www.php.net/manual/es/ref.var.php) y [`ctype_digit()`](https://www.php.net/manual/es/function.ctype-digit.php).
