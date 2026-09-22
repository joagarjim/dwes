# 6. Funciones en PHP

En los ejercicios anteriores hemos repetido operaciones como calcular una media, comprobar un dato o construir un mensaje. Una **función** agrupa instrucciones bajo un nombre para poder utilizarlas cada vez que las necesitemos. Ya hemos llamado a funciones de PHP como `count()` o `trim()`; ahora definiremos las nuestras.

## 6.1. Definir y llamar a una función

```php
<?php
function saludar(): string
{
    return 'Hola, clase';
}

$mensaje = saludar();
echo $mensaje; // Hola, clase
```

`function` inicia la definición; `saludar` es el nombre; los paréntesis contienen los parámetros, si los hay; el bloque contiene las instrucciones. **Definir** la función no muestra nada: las instrucciones se ejecutan al **llamarla** con `saludar()`. La anotación `: string` indica que devolverá una cadena. Veremos los tipos con detalle en el apartado 6.4.

Conviene que el nombre describa lo que hace la función y que cada función tenga una tarea clara.

## 6.2. Parámetros y argumentos

Los **parámetros** son variables declaradas en la función. Los **argumentos** son los valores que enviamos al llamarla:

```php
<?php
function sumar(int $a, int $b): int
{
    return $a + $b;
}

echo sumar(3, 5); // 8
echo sumar(10, 2); // 12
```

En `function sumar(int $a, int $b)`, `$a` y `$b` son parámetros. En `sumar(3, 5)`, `3` y `5` son argumentos: en esta llamada `$a` recibe `3` y `$b` recibe `5`. Podemos reutilizar la misma función con otros argumentos. Por ahora respetaremos el orden en que están definidos.

Una función puede recibir un array:

```php
<?php
function sumarLista(array $numeros): int
{
    $total = 0;
    foreach ($numeros as $numero) {
        $total += $numero;
    }
    return $total;
}

echo sumarLista([2, 4, 6]); // 12
```

## 6.3. Devolver un resultado con `return`

`return` termina la ejecución de la función y entrega un valor a quien la llamó. Podemos guardar ese valor, utilizarlo en otro cálculo o mostrarlo en HTML:

```php
<?php
function calcularMedia(array $notas): float
{
    $suma = 0;
    foreach ($notas as $nota) {
        $suma += $nota;
    }
    return $suma / count($notas);
}

$notas = [6, 8, 7];
$media = calcularMedia($notas);
?>
<p>Media: <?= number_format($media, 2, ',', '.') ?></p>
```

El resultado es `7,00`. Este ejemplo supone que el array tiene al menos un elemento; si pudiera estar vacío, habría que decidir qué devolver antes de dividir. La función calcula y el HTML presenta el resultado. `echo` dentro de una función escribiría directamente en la respuesta y dificultaría reutilizarla en otros lugares.

Podemos usar `return` para resolver pronto un caso especial:

```php
<?php
function esMayorDeEdad(int $edad): bool
{
    if ($edad < 0) {
        return false;
    }
    return $edad >= 18;
}
```

En cuanto se ejecuta un `return`, no se ejecutan las instrucciones posteriores de esa llamada.

## 6.4. Tipos de parámetros y del resultado

En los ejemplos, `int`, `float`, `string`, `bool` y `array` indican los tipos esperados. El tipo tras los paréntesis indica el resultado que la función debe devolver:

```php
<?php
function aplicarDescuento(float $precio, float $porcentaje): float
{
    return $precio * (1 - $porcentaje / 100);
}

echo aplicarDescuento(80.0, 25.0); // 60
```

Los tipos expresan qué espera la función. En `sumar(int $a, int $b): int`, los dos argumentos deben ser enteros y la función debe devolver un entero. Veamos qué pasa si le pasamos una cadena:

```php
<?php
function sumar(int $a, int $b): int
{
    return $a + $b;
}

echo sumar(10, 2);   // 12: los argumentos son enteros.
echo sumar('10', 2); // 12: PHP convierte la cadena numérica en entero.
```

**Por defecto**, PHP permite ciertas conversiones como la del ejemplo. Si queremos impedir que una cadena `'10'` se acepte como `int`, añadimos al comienzo del archivo que realiza la llamada:

```php
<?php
declare(strict_types=1);

function sumar(int $a, int $b): int
{
    return $a + $b;
}

echo sumar(10, 2);   // 12
echo sumar('10', 2); // TypeError: '10' es una cadena, no un entero.
```

`strict_types` se configura **en cada archivo que llama a la función**, inmediatamente después de `<?php`. Si `sumar()` estuviera en `funciones.php` y la llamáramos desde `pagina.php`, la declaración para comprobar esa llamada iría en `pagina.php`. Como excepción útil, incluso con modo estricto se puede pasar un `int` a un parámetro `float`.

**¿Y los formularios?** En este archivo completo, el modo estricto aparece en la segunda línea. Aunque el visitante escriba `20`, `$_POST['edad']` llega como la cadena `'20'`:

```php
<?php
declare(strict_types=1);

function mostrarEdad(int $edad): string
{
    return "Tienes $edad años";
}

$dato = $_POST['edad'] ?? '';

if (is_string($dato) && ctype_digit($dato) && (int) $dato <= 120) {
    $edad = (int) $dato;
    echo mostrarEdad($edad); // Correcto: recibe un int.
} else {
    echo 'Introduce una edad válida entre 0 y 120.';
}
```

Si escribiéramos `mostrarEdad($dato)` directamente, el modo estricto produciría un `TypeError`, porque `$dato` es una **cadena**. Las comprobaciones `ctype_digit()` y `<= 120` son decisiones de **validación del formulario**; `strict_types` no las realiza. Después de comprobar el dato, `(int)` lo convierte y la función recibe el tipo esperado.

## 6.5. Valores por defecto

Un parámetro puede tener un valor que se utiliza si omitimos ese argumento:

```php
<?php
function presentar(string $nombre, string $saludo = 'Hola'): string
{
    return $saludo . ', ' . $nombre;
}

echo presentar('Ana');              // Hola, Ana
echo presentar('Ana', 'Buenas');   // Buenas, Ana
```

Situaremos los parámetros con valor por defecto después de los obligatorios. El valor por defecto facilita llamadas habituales sin repetir siempre el mismo argumento.

## 6.6. Número variable de argumentos y argumentos con nombre

Cuando una función debe aceptar cualquier cantidad de números, `...` reúne los argumentos en un array:

```php
<?php
function sumarVarios(int ...$numeros): int
{
    $suma = 0;
    foreach ($numeros as $numero) {
        $suma += $numero;
    }
    return $suma;
}

echo sumarVarios(1, 5, 9); // 15
echo sumarVarios();        // 0: el total inicial no cambia.
```

Podemos resolver exactamente el mismo problema con las funciones `func_num_args()` y `func_get_arg()`, sin escribir parámetros en la definición:

```php
<?php
function sumarVariosClasica(): int
{
    $suma = 0;
    for ($i = 0; $i < func_num_args(); $i++) {
        $suma += func_get_arg($i);
    }
    return $suma;
}

echo sumarVariosClasica(1, 5, 9); // 15
echo sumarVariosClasica();        // 0
```

`func_num_args()` devuelve cuántos argumentos llegaron (en la primera llamada, `3`); `func_get_arg($i)` obtiene uno por su posición (`0`, `1`, `2`). También existe `func_get_args()`, que devuelve **todos** los argumentos juntos en un array; podríamos recorrerlo con `foreach` en lugar de usar el `for`. En ambos ejemplos recibimos una cantidad variable de datos y calculamos lo mismo. Con `int ...$numeros`, PHP ya los reúne en `$numeros`, permite declarar el tipo de cada uno y nos evita las llamadas a `func_get_arg()`. Por eso lo utilizaremos normalmente.

En sentido inverso, `...` **desempaqueta** un array para pasarlo como argumentos separados:

```php
<?php
$pareja = [3, 5];
echo sumar(...$pareja); // 8: equivale a sumar(3, 5).
```

Desde PHP 8 podemos indicar a qué parámetro corresponde cada argumento **escribiendo su nombre**. Utilizaremos siempre esta misma función para comparar las llamadas:

```php
<?php
function etiqueta(string $nombre, string $saludo = 'Hola', string $signo = '!'): string
{
    return $saludo . ', ' . $nombre . $signo;
}
```

### Llamadas por posición

```php
echo etiqueta('Ana');                 // Hola, Ana!
echo etiqueta('Ana', 'Buenas');      // Buenas, Ana!
echo etiqueta('Ana', 'Buenas', '.'); // Buenas, Ana.
```

PHP asigna los valores de izquierda a derecha: primero `$nombre`, después `$saludo` y luego `$signo`. Si no enviamos los dos últimos, se usan `'Hola'` y `'!'`.

### Llamadas por nombre

```php
echo etiqueta(nombre: 'Ana', signo: '.');       // Hola, Ana.
echo etiqueta(signo: '?', nombre: 'Luis');     // Hola, Luis?
echo etiqueta(saludo: 'Buenas', nombre: 'Ana'); // Buenas, Ana!
```

Ahora `nombre:`, `saludo:` y `signo:` indican **a qué parámetro** va cada valor. Podemos cambiar el orden y omitir `$saludo` o `$signo` porque tienen valores por defecto. `$nombre` es obligatorio: no podemos omitirlo.

### Mezclar ambos estilos

```php
echo etiqueta('Ana', signo: '?'); // Hola, Ana?
```

`'Ana'` va por posición al primer parámetro (`$nombre`); `signo: '?'` va al tercero (`$signo`). El segundo (`$saludo`) conserva `'Hola'`. Los argumentos por posición deben ir **antes** de los argumentos por nombre; tampoco podemos enviar dos veces el mismo parámetro.

Los nombres escritos en la llamada forman parte de la forma de utilizar la función. Si cambiamos `$signo` por `$puntuacion` en su definición, habrá que cambiar las llamadas que usan `signo:`.

## 6.7. Ámbito de las variables y cambios en arrays

Las variables creadas dentro de una función son **locales**: solo se usan allí. Una variable definida fuera no aparece automáticamente dentro de la función, aunque tenga el mismo nombre:

```php
<?php
$precio = 100;

function duplicar(int $precio): int
{
    $resultado = $precio * 2;
    return $resultado;
}

$nuevoPrecio = duplicar($precio);
echo $precio;      // 100
echo $nuevoPrecio; // 200
// Aquí no existe $resultado: es una variable local de duplicar().
```

Una variable de ámbito global **no es accesible directamente dentro de una función**. PHP permite importarla con `global $precio;` o consultar `$GLOBALS`, pero evitaremos ambas opciones aquí. Es preferible pasar los datos por parámetros y devolver resultados. Al pasar un array de la forma habitual, cambiarlo dentro de la función no cambia el array de quien llama:

```php
<?php
function conPostre(array $menu): array
{
    $menu['postre'] = 'Fruta';
    return $menu;
}

$original = ['primero' => 'Sopa'];
$ampliado = conPostre($original);
// $original sigue teniendo solo 'primero'; $ampliado también tiene 'postre'.
```

Si necesitamos modificar expresamente la variable que recibió la función, PHP permite un parámetro **por referencia** con `&`:

```php
<?php
function anadirPostre(array &$menu): void
{
    $menu['postre'] = 'Fruta';
}

$menu = ['primero' => 'Sopa'];
anadirPostre($menu);
// Ahora $menu tiene 'primero' y 'postre'.
```

`void` indica que la función no devuelve un valor. Usaremos las referencias solo cuando queramos ese efecto y lo dejemos claro en el nombre y en la llamada. En nuestros primeros ejercicios preferiremos `return`.

## 6.8. Funciones guardadas en variables y funciones anónimas

Una variable puede contener el nombre de una función y llamarla. También puede guardar una **función anónima**, definida sin nombre:

```php
<?php
function duplicar(int $numero): int
{
    return $numero * 2;
}

$operacion = 'duplicar';
echo $operacion(4); // 8

$triplicar = function (int $numero): int {
    return $numero * 3;
};
echo $triplicar(4); // 12
```

La función anónima puede necesitar un valor definido fuera de ella. `use` captura el valor en el momento de crearla:

```php
<?php
$incremento = 2;
$sumarIncremento = function (int $numero) use ($incremento): int {
    return $numero + $incremento;
};
echo $sumarIncremento(5); // 7
```

Una **función flecha** expresa de forma breve una operación que devuelve una expresión; también puede usar `$incremento` del ámbito exterior sin `use` explícito:

```php
<?php
$incremento = 2;
$sumarIncremento = fn (int $numero): int => $numero + $incremento;
echo $sumarIncremento(5); // 7
```

Las veremos de nuevo cuando usemos funciones que reciben otras funciones, por ejemplo para transformar u ordenar arrays. Por ahora basta con reconocer la sintaxis y hacer una llamada sencilla.

## 6.9. Organizar funciones en otro archivo

Cuando varias páginas necesitan las mismas funciones, podemos definirlas una vez en un archivo y cargarlo:

**`funciones.php`**

```php
<?php
function formatearPrecio(float $precio): string
{
    return number_format($precio, 2, ',', '.') . ' €';
}
```

**`producto.php`**

```php
<?php
require_once __DIR__ . '/funciones.php';
$precio = 12.5;
?>
<p>Precio: <?= formatearPrecio($precio) ?></p>
```

`__DIR__` señala la carpeta del archivo actual. `require_once` carga el archivo una sola vez y detiene la ejecución si falta: conviene cuando las funciones son necesarias para que la página funcione. Más adelante utilizaremos también `include` para reutilizar partes de una plantilla HTML.

## 6.10. Reutilizar fragmentos HTML con `include`

También podemos cargar archivos que contengan HTML. Es una primera forma de compartir la cabecera y el pie de varias páginas:

**`encabezado.php`**

```php
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title><?= htmlspecialchars($titulo, ENT_QUOTES, 'UTF-8') ?></title>
</head>
<body>
```

**`pie.php`**

```php
<footer>IES Portada Alta</footer>
</body>
</html>
```

**`pagina.php`**

```php
<?php
$titulo = 'Mi página';
include __DIR__ . '/encabezado.php';
?>
<h1><?= htmlspecialchars($titulo, ENT_QUOTES, 'UTF-8') ?></h1>
<?php include __DIR__ . '/pie.php'; ?>
```

El archivo incluido se ejecuta en el punto donde aparece `include`; en este ejemplo ve `$titulo` porque se definió antes de incluir la cabecera. No confundas esa situación con el ámbito de una función. `include` emite un aviso si el archivo falta y puede continuar; `require` provoca un error y detiene la ejecución. Las variantes `_once` impiden cargar el mismo archivo más de una vez durante la petición. Elegiremos `require_once` para bibliotecas de funciones indispensables y, en este ejemplo, `include` para los fragmentos de presentación.

## Comprueba lo aprendido

Antes de continuar, deberías poder distinguir definición y llamada, parámetros y argumentos, explicar qué devuelve `return`, escribir una función con tipos y un parámetro por defecto, reconocer dónde existe una variable local y decidir cuándo pasar un array y devolver uno nuevo. También deberías reconocer `...`, los argumentos con nombre y una función anónima, y poder cargar funciones comunes o fragmentos HTML desde otro archivo.

## Practica lo aprendido

Realiza los **ejercicios 29 a 37** del [bloque 6: Funciones](actividades.md#bloque-6-funciones). Entrega juntos los archivos del bloque en Moodle.

## Fuentes de consulta

- [Aitor Medrano: El lenguaje PHP](https://aitor-medrano.github.io/dwes2122/02php.html), apartados «Funciones» y «Actividades: Funciones».
- Manual oficial de PHP: [funciones definidas por el usuario](https://www.php.net/manual/es/functions.user-defined.php), [argumentos](https://www.php.net/manual/es/functions.arguments.php), [ámbito de variables](https://www.php.net/manual/es/language.variables.scope.php) y [`require_once`](https://www.php.net/manual/es/function.require-once.php).
