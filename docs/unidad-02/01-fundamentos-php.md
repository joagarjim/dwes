# 1. Fundamentos de PHP

En la unidad anterior comprobamos que el navegador solicita un recurso y el servidor prepara una respuesta. Ahora empezaremos a escribir el código que genera parte de esa respuesta. Trabajaremos en el entorno de VS Code, Docker y Apache que ya tenemos preparado.

Al terminar este apartado podrás integrar PHP en HTML, mostrar información, reconocer los tipos básicos, declarar variables y constantes y construir expresiones con operadores. Los formularios se estudiarán en el apartado siguiente.

## 1.1. PHP dentro de HTML

Un archivo `.php` puede contener HTML y fragmentos de PHP. El intérprete ejecuta las instrucciones entre `<?php` y `?>`; el HTML que queda fuera se incorpora a la respuesta. El navegador recibe el resultado, no el código PHP.

```php
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Primer ejemplo</title>
</head>
<body>
    <h1>Mi primera página PHP</h1>
    <p>Este párrafo está escrito en HTML.</p>
    <?php
        echo '<p>Este párrafo lo genera PHP.</p>';
    ?>
</body>
</html>
```

Abre el archivo a través de Apache y consulta después el código fuente de la página en el navegador: encontrarás dos párrafos HTML. Una instrucción PHP suele terminar en `;`. Podemos abrir y cerrar varios bloques PHP en la misma página, aunque conviene hacerlo con claridad para que el archivo resulte legible.

Si un archivo contiene exclusivamente código PHP, se acostumbra a omitir la etiqueta final `?>`. En una página que vuelve a HTML después del bloque, sí cerramos el bloque PHP.

## 1.2. Generar contenido: `echo`, `print` y `<?= ... ?>`

Las tres formas siguientes escriben contenido en la respuesta:

```php
<?php
    echo '<p>Mensaje generado con echo.</p>';
    print '<p>Mensaje generado con print.</p>';
?>
<p>Mensaje generado con <?= 'la etiqueta corta de echo' ?>.</p>
```

`echo` es una construcción del lenguaje y puede mostrar una o varias expresiones separadas por comas. `print` muestra una sola expresión y devuelve el valor `1`; en este curso rara vez necesitaremos ese valor. `<?= $expresion ?>` equivale a `<?php echo $expresion; ?>` y resulta cómoda para colocar un valor dentro de HTML. La variable del ejemplo siguiente se explica en el apartado 1.5.

```php
<?php $modulo = 'DWES'; ?>
<h1><?= $modulo ?></h1>
```

**Atención:** si escribimos `echo '<strong>Hola</strong>';`, el navegador interpretará las etiquetas como HTML. Si mostramos datos introducidos por una persona, tendremos que tratarlos antes de insertarlos en HTML; lo practicaremos al abordar los formularios.

## 1.3. Comentarios y lectura de errores

Los comentarios explican decisiones o aportan contexto; no aparecen en la respuesta. PHP admite comentarios de una línea y de varias:

```php
<?php
// Calculamos el importe antes de mostrarlo.
# También es un comentario de una línea.
/* Este comentario puede ocupar
   más de una línea. */
echo 'Página preparada';
?>
```

Conviene escribir comentarios útiles, no repetir literalmente lo que ya expresa el código. Los comentarios HTML (`<!-- ... -->`) son distintos: pueden aparecer en el código fuente recibido por el navegador.

Al escribir PHP, cometer errores es normal. Por ejemplo, la falta de `;` o de una comilla puede producir un error de sintaxis:

```php
<?php
    echo 'Primer mensaje'
    echo 'Segundo mensaje';
?>
```

Para corregirlo, lee el mensaje, identifica el archivo y la línea señalados y revisa también la instrucción inmediatamente anterior. La línea mostrada puede ser aquella en la que el intérprete detectó el problema, aunque la causa esté antes. Durante el desarrollo observaremos los errores en nuestro entorno; en una aplicación publicada no se deben mostrar detalles internos al visitante.

## 1.4. Variables y tipos de datos

Las variables comienzan por `$`. PHP distingue mayúsculas y minúsculas: `$nombre` y `$Nombre` son variables diferentes. No hace falta declarar su tipo antes de asignarles un valor.

```php
<?php
$nombre = 'Lucía';       // string
$edad = 20;             // int
$nota = 8.5;            // float
$aprobado = true;       // bool
$segundaNota = null;    // null: ausencia de valor

var_dump($nombre, $edad, $nota, $aprobado, $segundaNota);
```

`var_dump()` muestra el tipo y el valor, por lo que resulta útil para investigar qué contiene una variable. La usaremos para depurar, no para construir la presentación final de una página.

| Tipo | Ejemplo | Uso habitual |
| --- | --- | --- |
| `string` | `'DWES'` | Texto |
| `int` | `20` | Números enteros |
| `float` | `8.5` | Números con decimales; en PHP el literal usa punto |
| `bool` | `true` | Valores verdadero/falso |
| `null` | `null` | Ausencia de valor |
| `array` | `['PHP', 'HTML']` | Conjunto de valores; lo estudiaremos después |

PHP tiene tipado dinámico: una variable puede recibir valores de distintos tipos. Que sea posible no significa que resulte aconsejable cambiar su significado durante el programa.

```php
<?php
$valor = 12;
var_dump($valor); // int(12)
$valor = 'doce';
var_dump($valor); // string(4) "doce"
```

Inicializa las variables antes de utilizarlas. Una variable sin valor previo puede provocar un aviso; no debemos confiar en que PHP le asigne uno por nosotros.

## 1.5. Constantes

Una constante tiene un nombre y un valor que no cambia durante la ejecución. Se utiliza por convenio un nombre en mayúsculas, sin `$`.

```php
<?php
const IVA = 0.21;
define('NOMBRE_CENTRO', 'IES Ejemplo');

$precio = 100;
$total = $precio * (1 + IVA);
echo "El total con IVA es $total €";
```

Para los ejemplos habituales de esta unidad preferiremos `const`. `define()` permite definir una constante durante la ejecución. Más adelante veremos otras diferencias cuando sean necesarias.

## 1.6. Operadores aritméticos y concatenación

| Operador | Significado | Ejemplo |
| --- | --- | --- |
| `+` | Suma | `$a + $b` |
| `-` | Resta | `$a - $b` |
| `*` | Multiplicación | `$a * $b` |
| `/` | División | `$a / $b` |
| `%` | Resto | `$a % $b` |
| `**` | Potencia | `$a ** $b` |
| `.` | Concatenación de cadenas | `'Hola, ' . $nombre` |

```php
<?php
$precio = 24.50;
$unidades = 3;
$total = $precio * $unidades;

// Usamos paréntesis para que la intención resulte evidente.
$precioConDescuento = $total * (1 - 0.10);

echo 'Total: ' . $total . ' €';
echo "<p>Tras el descuento: $precioConDescuento €</p>";
```

Las variables se interpolan dentro de cadenas delimitadas por comillas dobles; dentro de comillas simples no se sustituyen por su valor. El operador `.` une fragmentos de texto. Observa el resultado de `echo 'Hola, $nombre';` frente a `echo "Hola, $nombre";`.

## 1.7. Operadores de comparación

Una comparación produce un valor booleano (`true` o `false`).

| Operador | Significado |
| --- | --- |
| `==` | Valores iguales tras posibles conversiones de tipo |
| `===` | Mismo valor y mismo tipo |
| `!=` | Valores diferentes tras posibles conversiones |
| `!==` | Distinto valor o distinto tipo |
| `<`, `>`, `<=`, `>=` | Comparaciones de orden |

```php
<?php
$numero = 10;
$texto = '10';

var_dump($numero == $texto);  // true
var_dump($numero === $texto); // false
```

Como regla inicial, preferiremos `===` y `!==` cuando queramos evitar conversiones implícitas. Conviene recordar que los datos procedentes de formularios suelen llegar como texto; veremos cómo tratarlos en el siguiente apartado.

## 1.8. Operadores lógicos

Permiten combinar condiciones:

| Operador | Se cumple cuando... |
| --- | --- |
| `&&` | Se cumplen ambas condiciones |
| `||` | Se cumple al menos una |
| `!` | La condición original no se cumple |

```php
<?php
$edad = 20;
$tieneEntrada = true;

var_dump($edad >= 18 && $tieneEntrada); // true
var_dump($edad < 18 || $tieneEntrada);  // true
var_dump(!$tieneEntrada);              // false
```

PHP también dispone de `and`, `or` y `xor`, pero `and` y `or` tienen distinta precedencia que `&&` y `||`. Para los primeros programas usaremos `&&`, `||` y `!`, con paréntesis si una expresión resulta difícil de leer.

## 1.9. Operadores de asignación y valores por defecto

`=` asigna un valor. Otros operadores combinan un cálculo con la asignación:

| Operador | Equivale a |
| --- | --- |
| `$a += $b` | `$a = $a + $b` |
| `$a -= $b` | `$a = $a - $b` |
| `$a *= $b` | `$a = $a * $b` |
| `$a /= $b` | `$a = $a / $b` |
| `$texto .= $fragmento` | `$texto = $texto . $fragmento` |
| `$a++` | Incrementa `$a` en una unidad |
| `$a--` | Decrementa `$a` en una unidad |

```php
<?php
$puntos = 10;
$puntos += 5;
$puntos++;
echo $puntos; // 16

$mensaje = 'Hola';
$mensaje .= ', clase';
echo $mensaje; // Hola, clase
```

El operador `??` toma el valor situado a su izquierda si existe y no es `null`; en caso contrario, utiliza el de la derecha:

```php
<?php
$apodo = null;
$nombreVisible = $apodo ?? 'Sin apodo';
echo $nombreVisible; // Sin apodo
```

Más adelante será útil al trabajar con datos opcionales. No necesitamos memorizar toda la tabla de precedencia: usaremos paréntesis para expresar con claridad el orden de evaluación.

## Comprueba lo aprendido

Antes de continuar, deberías poder explicar qué HTML recibe el navegador al ejecutar un archivo PHP, distinguir `echo`, `print` y `<?= ... ?>`, identificar el tipo de una variable con `var_dump()`, reconocer la diferencia entre `==` y `===` y calcular el resultado de una expresión que combine operadores.

Realiza los **ejercicios 1 a 6** del [bloque 1: Fundamentos de PHP](actividades.md#bloque-1-fundamentos-de-php). Entrega los seis archivos juntos en la tarea del bloque 1 en Moodle. Los formularios comienzan en el apartado 2.

## Fuentes de consulta

- [Aitor Medrano: El lenguaje PHP](https://aitor-medrano.github.io/dwes2122/02php.html). Referencia de organización y progresión didáctica; ejemplos redactados y actualizados para esta unidad.
- [Manual oficial de PHP: sintaxis básica](https://www.php.net/manual/es/language.basic-syntax.php).
- [Manual oficial de PHP: operadores](https://www.php.net/manual/es/language.operators.php).
