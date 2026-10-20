# 1. Clases y objetos en PHP

Hasta ahora hemos utilizado variables, arrays y funciones para almacenar datos y trabajar con ellos. En esta unidad aprenderemos otra forma de organizar nuestros programas: la **programación orientada a objetos (POO)**.

Empezaremos por una clase sencilla y aprenderemos a crear y utilizar varios objetos. Trabajaremos en el mismo entorno Apache y PHP de Docker que ya tenemos preparado. No necesitamos Laravel ni una base de datos.

## 1.1. Del array al objeto

En la unidad anterior podíamos representar un producto mediante un array asociativo:

```php
<?php
declare(strict_types=1);

$producto = [
    'nombre' => 'Teclado',
    'precio' => 25.0,
    'stock' => 10,
];

function calcularImporte(array $producto, int $unidades): float
{
    return $producto['precio'] * $unidades;
}

echo $producto['nombre'];             // Teclado
echo calcularImporte($producto, 3);   // 75
```

El array guarda los **datos** y la función define una **operación** sobre esos datos. En la llamada a `calcularImporte()` debemos enviar el producto que queremos utilizar.

La POO nos permite reunir los datos y las operaciones relacionadas en un objeto. Un producto podrá guardar su nombre, precio y stock y ofrecer operaciones como calcular un importe o consultar si hay existencias.

Esto no significa que los arrays dejen de ser útiles: seguiremos utilizándolos, por ejemplo, para guardar una lista de objetos. Elegiremos cada herramienta según el problema.

!!! question "Antes de continuar"
    Si tenemos diez productos, ¿qué datos se repiten como estructura y qué valores cambian de un producto a otro?

## 1.2. Clase y objeto: dos conceptos diferentes

Una **clase** define una estructura y un conjunto de operaciones. Describe qué datos tendrán sus objetos y qué podrán hacer.

Un **objeto** es una instancia concreta de una clase: existe durante la ejecución del programa y tiene sus propios datos.

| Concepto | En nuestro ejemplo |
| --- | --- |
| Clase | `Producto`: define que los productos tienen nombre, precio y stock. |
| Primer objeto | Un teclado con un precio de 25 euros y 10 unidades en stock. |
| Segundo objeto | Un ratón con un precio de 15 euros y 6 unidades en stock. |

Podemos comparar la clase con una ficha vacía que indica qué información debemos guardar. Cada objeto sería una ficha concreta con sus valores. La comparación ayuda a entender los datos, pero una clase también define **comportamiento mediante métodos**.

**Definir una clase no crea automáticamente ningún objeto.** Primero describiremos la clase y después crearemos las instancias que necesitemos.

## 1.3. Nuestra primera clase

Crea una carpeta `u3-apartado1` dentro de la carpeta del proyecto que Apache publica. Dentro, crea el archivo `index.php`:

```php
<?php
declare(strict_types=1);

class Producto
{
    public string $nombre = 'Sin nombre';
    public float $precio = 0.0;
    public int $stock = 0;
}

$teclado = new Producto();

echo $teclado->nombre; // Sin nombre
```

Accede desde el navegador a `http://localhost:8080/u3-apartado1/index.php`, si mantienes el puerto 8080 de nuestro entorno. El resultado será `Sin nombre`.

### ¿Qué significa cada elemento?

| Elemento | Significado |
| --- | --- |
| `class` | Inicia la definición de una clase. |
| `Producto` | Es el nombre que elegimos para la clase. |
| `{ ... }` | Delimita el contenido de la clase. |
| `public` | Permite acceder al elemento desde fuera de la clase. |
| `string`, `float`, `int` | Indican el tipo de cada propiedad. |
| `$nombre`, `$precio`, `$stock` | Son las propiedades: los datos que tendrá cada objeto. |
| `= 'Sin nombre'`, `= 0.0`, `= 0` | Establecen sus valores iniciales. |
| `new Producto()` | Crea un objeto de la clase `Producto`. |
| `$teclado` | Es la variable con la que accedemos al objeto creado. |
| `->` | Permite acceder a una propiedad o llamar a un método del objeto. |

Las **propiedades** son variables declaradas dentro de una clase para guardar el estado de sus objetos. El estado es el conjunto de valores que tiene un objeto en un momento concreto.

Los nombres de las clases se escribirán con inicial mayúscula, por ejemplo `Producto` o `CuentaBancaria`. Las propiedades y métodos usarán nombres como `precio` o `calcularImporte`.

!!! note "Un primer paso"
    Utilizamos propiedades públicas para comprender cómo funcionan los objetos. En el siguiente apartado veremos qué problemas puede causar modificarlas directamente y cómo controlarlo mediante encapsulación.

## 1.4. Leer y modificar propiedades

Después de crear el objeto, podemos asignar valores a sus propiedades públicas:

```php
$teclado->nombre = 'Teclado';
$teclado->precio = 25.0;
$teclado->stock = 10;

echo $teclado->nombre; // Teclado
echo $teclado->precio; // 25
echo $teclado->stock;  // 10
```

Estas instrucciones se escriben **después** de `$teclado = new Producto();`, fuera de las llaves de la clase. Sustituyen los valores iniciales de ese objeto.

Compara la consulta del array con la del objeto:

| Array asociativo | Objeto |
| --- | --- |
| `$producto['nombre']` | `$teclado->nombre` |
| `$producto['precio']` | `$teclado->precio` |
| `$producto['stock']` | `$teclado->stock` |

En la declaración escribimos `$nombre`, pero en el acceso habitual escribimos `->nombre`, **sin otro signo `$`**.

Cada propiedad tiene un tipo. `precio` contiene un número, no el texto `'25 euros'`. El símbolo de moneda y el formato se añadirán al presentar el resultado.

También debemos distinguir **tipo** y **validez para el problema**: `-5.0` es un `float`, pero no sería un precio válido para nuestra tienda. Declarar el tipo no impide por sí solo ese valor. Lo resolveremos en el apartado de encapsulación y validación.

## 1.5. Métodos: las operaciones de un objeto

Un **método** es una función definida dentro de una clase. Describe una operación que podrán realizar sus objetos.

Sustituye el contenido de `index.php` por este ejemplo completo:

```php
<?php
declare(strict_types=1);

class Producto
{
    public string $nombre = 'Sin nombre';
    public float $precio = 0.0;
    public int $stock = 0;

    public function calcularImporte(int $unidades): float
    {
        return $this->precio * $unidades;
    }

    public function hayStock(): bool
    {
        return $this->stock > 0;
    }
}

$teclado = new Producto();
$teclado->nombre = 'Teclado';
$teclado->precio = 25.0;
$teclado->stock = 10;

echo $teclado->calcularImporte(3); // 75
echo '<br>';

if ($teclado->hayStock()) {
    echo 'Hay existencias';
} else {
    echo 'No hay existencias';
}
```

El navegador mostrará:

```text
75
Hay existencias
```

`public function calcularImporte(int $unidades): float` se interpreta así:

- `public`: el método puede llamarse desde fuera de la clase.
- `function`: inicia la definición del método.
- `calcularImporte`: es el nombre elegido.
- `int $unidades`: recibe un parámetro entero.
- `: float`: devuelve un número de tipo `float`.
- `return`: entrega el resultado a la instrucción que realizó la llamada.

En `$teclado->calcularImporte(3)`, `3` es el argumento que recibe `$unidades`. Los paréntesis indican que **llamamos a un método**. En `$teclado->precio` consultamos una propiedad y no escribimos paréntesis.

`hayStock()` no recibe argumentos, pero también necesita los paréntesis. Devuelve `true` si el stock es mayor que cero y `false` en caso contrario. Utilizamos ese resultado en un `if`; mostrar directamente un booleano con `echo` no produciría los textos «true» y «false».

!!! warning "Qué hace realmente calcularImporte"
    Calcula precio por unidades. No realiza una venta, no descuenta stock y no comprueba si hay unidades suficientes. En estos primeros ejemplos usamos cantidades positivas. Una operación de venta necesitará otras comprobaciones.

## 1.6. Entender `$this`

Dentro de un método, **`$this` representa el objeto sobre el que se está ejecutando esa llamada**.

```php
$teclado->precio = 25.0;
echo $teclado->calcularImporte(3);
```

El recorrido es el siguiente:

1. Llamamos a `calcularImporte(3)` sobre el objeto al que accedemos mediante `$teclado`.
2. Dentro de esa llamada, `$unidades` vale `3`.
3. `$this` representa ese objeto, de modo que `$this->precio` vale `25.0`.
4. El método calcula `25.0 * 3` y devuelve `75.0`.
5. El `echo` exterior muestra el resultado como `75`.

No escribimos `$teclado` dentro del método: ese nombre pertenece al código exterior y puede ser distinto en cada programa. `$this` permite que el mismo método funcione con cualquier objeto de la clase.

### Propiedad, parámetro y variable local

Este método ilustra los tres elementos:

```php
public function calcularImporte(int $unidades): float
{
    $importe = $this->precio * $unidades;
    return $importe;
}
```

| Expresión | Qué representa | Dónde se utiliza |
| --- | --- | --- |
| `$this->precio` | Una propiedad del objeto actual. | Su valor permanece en el objeto entre llamadas, salvo que se modifique. |
| `$unidades` | Un parámetro del método. | Recibe un valor nuevo en cada llamada. |
| `$importe` | Una variable local del método. | Solo existe dentro de esa ejecución del método. |

`$precio` y `$this->precio` **no son lo mismo**. Escribir `$precio` dentro del método buscaría una variable local con ese nombre; no consultaría automáticamente la propiedad del objeto.

En nuestros métodos de instancia, `$this` estará disponible al llamar al método sobre un objeto. No se utiliza en el código exterior como sustituto de `$teclado`.

## 1.7. Varios objetos de la misma clase

Después de la definición de `Producto`, podemos crear tantos objetos como necesitemos:

```php
$teclado = new Producto();
$teclado->nombre = 'Teclado';
$teclado->precio = 25.0;
$teclado->stock = 10;

$raton = new Producto();
$raton->nombre = 'Ratón';
$raton->precio = 15.0;
$raton->stock = 6;

echo $teclado->calcularImporte(2); // 50
echo '<br>';
echo $raton->calcularImporte(2);   // 30

$teclado->precio = 30.0;

echo '<br>';
echo $teclado->precio; // 30
echo '<br>';
echo $raton->precio;   // 15: no ha cambiado
```

Los dos objetos tienen las mismas propiedades y métodos porque pertenecen a la misma clase, pero cada uno conserva sus valores. En la primera llamada, `$this` representa el teclado; en la segunda, representa el ratón.

**Cada ejecución de `new Producto()` crea un objeto distinto.** Esto es diferente de asignar a otra variable un objeto que ya existe:

```php
$otroNombre = $teclado;
$otroNombre->precio = 40.0;

echo $teclado->precio; // 40
```

Aquí no hemos creado otro producto. Las dos variables permiten acceder al **mismo objeto**. No debemos interpretar `$otroNombre = $teclado` como una copia independiente. Más adelante podremos estudiar el clonado; por ahora basta con distinguir esta asignación de un nuevo `new`.

## 1.8. Métodos que cambian el estado

Los métodos anteriores consultan datos y devuelven resultados. Un método también puede modificar el objeto. Añade este método dentro de la clase:

```php
public function reponerUnaUnidad(): void
{
    $this->stock++;
}
```

Después de crear el objeto, podemos utilizarlo así:

```php
$teclado->stock = 10;
$teclado->reponerUnaUnidad();
echo $teclado->stock; // 11
```

`$this->stock++` incrementa la propiedad del objeto actual. El cambio permanece después de terminar el método, mientras ese objeto siga existiendo.

`void` indica que el método no devuelve un valor. Por eso lo llamamos como una instrucción y después consultamos el stock; no escribimos `echo $teclado->reponerUnaUnidad()`.

| Método | Efecto sobre el objeto | Retorno |
| --- | --- | --- |
| `calcularImporte(3)` | No cambia sus propiedades. | Un `float`. |
| `hayStock()` | No cambia sus propiedades. | Un `bool`. |
| `reponerUnaUnidad()` | Incrementa el stock en una unidad. | Ningún valor (`void`). |

## 1.9. Separar la clase del archivo que la utiliza

Al principio hemos escrito todo en un archivo para ver el funcionamiento. Cuando el ejemplo crece, conviene guardar cada clase en su propio archivo y cargarla desde la página que la utiliza.

La carpeta `u3-apartado1` contendrá dos archivos: `Producto.php` e `index.php`.

### Archivo `Producto.php`

```php
<?php
declare(strict_types=1);

class Producto
{
    public string $nombre = 'Sin nombre';
    public float $precio = 0.0;
    public int $stock = 0;

    public function calcularImporte(int $unidades): float
    {
        return $this->precio * $unidades;
    }

    public function hayStock(): bool
    {
        return $this->stock > 0;
    }

    public function reponerUnaUnidad(): void
    {
        $this->stock++;
    }
}
```

Este archivo solo define la clase. No crea productos ni genera HTML. Al contener únicamente PHP, omitimos el cierre `?>` para evitar enviar accidentalmente espacios o saltos de línea.

### Archivo `index.php`

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Producto.php';

$teclado = new Producto();
$teclado->nombre = 'Teclado';
$teclado->precio = 25.0;
$teclado->stock = 10;

$importe = $teclado->calcularImporte(3);
$teclado->reponerUnaUnidad();
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mi primer producto</title>
</head>
<body>
    <h1><?= htmlspecialchars($teclado->nombre, ENT_QUOTES, 'UTF-8') ?></h1>
    <p>Precio: <?= number_format($teclado->precio, 2, ',', '.') ?> €</p>
    <p>Importe de 3 unidades: <?= number_format($importe, 2, ',', '.') ?> €</p>
    <p>Stock después de reponer: <?= $teclado->stock ?></p>

    <?php if ($teclado->hayStock()): ?>
        <p>Hay existencias.</p>
    <?php else: ?>
        <p>No hay existencias.</p>
    <?php endif; ?>
</body>
</html>
```

Abre **`index.php`** desde el navegador. La página mostrará el nombre, el precio `25,00 €`, el importe `75,00 €`, el stock `11` y el mensaje de disponibilidad.

### Recordamos la carga del archivo

- `__DIR__` contiene la ruta de la carpeta del archivo actual.
- El punto `.` **concatena** esa ruta con `'/Producto.php'`.
- `require_once` carga y ejecuta el archivo una sola vez durante la petición. Si no puede cargarlo, se produce un error que interrumpe la ejecución si no se gestiona.
- La clase debe estar disponible cuando ejecutamos `new Producto()`.

El archivo incluido tiene su propio `declare(strict_types=1);`; esta directiva se configura por archivo. No debe haber HTML antes de ella.

`htmlspecialchars()` permite mostrar el nombre como texto en HTML. `number_format()` prepara los números para su presentación: la propiedad y el cálculo siguen siendo numéricos.

La forma `if (...): ... else: ... endif;` es una sintaxis alternativa de PHP, útil al intercalar HTML. Equivale a un `if` con llaves; no es una instrucción nueva.

La clase calcula y cambia su estado; el HTML decide cómo presentar los datos. Así podremos reutilizar la clase en otra página sin que sus métodos impriman contenido inesperadamente.

!!! warning "No declares la clase dos veces"
    Al separar los archivos, elimina la definición de `Producto` de `index.php`. La clase debe quedar en `Producto.php`; en `index.php` escribimos la carga y el código que la utiliza.

## 1.10. ¿Cuánto dura un objeto en una página PHP?

En nuestro entorno habitual, cada petición ejecuta el programa de nuevo. Los objetos que creamos durante una petición no se conservan automáticamente para la siguiente.

Si recargas el ejemplo, se vuelve a crear el producto, se asigna stock `10` y se repone una unidad. Verás **11 otra vez**, no 12. Para conservar información entre peticiones necesitaremos mecanismos que estudiaremos después, como sesiones o una base de datos.

Crear un objeto no crea una tabla ni guarda automáticamente sus datos en disco.

## 1.11. Errores frecuentes

| Error | Motivo y corrección |
| --- | --- |
| Usar `$teclado` sin crearlo. | Antes necesitamos `$teclado = new Producto();`. |
| Escribir `$teclado['precio']`. | Es la sintaxis de un array. Para este objeto usamos `$teclado->precio`. |
| Escribir `$teclado->calcularImporte`. | Para ejecutar el método faltan la llamada y su argumento: `$teclado->calcularImporte(3)`. |
| Escribir `$precio` dentro del método. | No accede a la propiedad. Debemos escribir `$this->precio`. |
| Escribir `$this` en el código exterior. | Allí utilizamos la variable del objeto, por ejemplo `$teclado`. |
| Confundir `return` con `echo`. | `return` entrega el resultado; `echo` escribe en la respuesta. Nuestros métodos devuelven resultados y la página los muestra. |
| Escribir `preico` en vez de `precio`. | No es la propiedad declarada. En PHP 8.2 y posteriores crear propiedades dinámicas de esta forma está obsoleto para clases como esta y puede generar un aviso de obsolescencia. Corrige el nombre. |
| Declarar `public float $precio;` y leerlo sin asignarlo. | Una propiedad tipada sin valor inicial no puede leerse antes de inicializarla. En este apartado utilizamos valores iniciales explícitos. |
| Guardar `'25 euros'` en una propiedad `float`. | Esa cadena no es un precio numérico válido para la propiedad. Guarda `25.0` y añade la moneda al mostrarlo. |
| Esperar una copia con `$otro = $teclado`. | Ambas variables acceden al mismo objeto. Para crear otro objeto usamos `new`. |

## Comprueba lo aprendido

Antes de continuar, deberías poder:

1. Explicar la diferencia entre una clase y un objeto.
2. Identificar las propiedades y los métodos de una clase.
3. Crear dos objetos y asignar valores distintos a sus propiedades.
4. Diferenciar la lectura de una propiedad de la llamada a un método.
5. Explicar qué representa `$this` en cada llamada.
6. Distinguir una propiedad, un parámetro y una variable local.
7. Utilizar un método que devuelve un valor y otro con retorno `void`.
8. Explicar por qué asignar un objeto a otra variable no crea una copia independiente.
9. Guardar la clase en otro archivo y cargarla con `require_once`.
10. Explicar por qué recargar la página no conserva automáticamente los cambios del objeto.

## Practica lo aprendido

Realiza los **ejercicios 1 a 5 del bloque 1: Clases y objetos** de la [relación de actividades](actividades.md#bloque-1-clases-y-objetos). Se trabajan en casa; en clase revisaremos soluciones, dudas y errores frecuentes.

En el siguiente apartado aprenderemos a **inicializar objetos mediante un constructor y controlar el acceso a sus propiedades**.

## Fuentes de consulta

- [Aitor Medrano y Luis Alemañ: PHP orientado a objetos](https://aitor-medrano.github.io/dwes2122/03phpoo.html).
- Manual oficial de PHP: [fundamentos de clases y objetos](https://www.php.net/manual/es/language.oop5.basic.php), [propiedades](https://www.php.net/manual/es/language.oop5.properties.php) y [objetos y referencias](https://www.php.net/manual/es/language.oop5.references.php).

Material elaborado por **Joaquín García**, para DWES de 2.º DAW del **IES Portada Alta**. Adapta y amplía contenidos de Aitor Medrano y Luis Alemañ. Licencia **CC BY-NC-SA**.
