# 2. Constructor y encapsulación

En el apartado anterior creábamos un producto y después asignábamos sus datos mediante propiedades públicas. Ahora aprenderemos a **inicializarlo al crearlo** y a **controlar las operaciones que pueden modificarlo**.

Son dos necesidades relacionadas, pero diferentes: el constructor prepara el objeto; la encapsulación establece cómo se utiliza desde el exterior.

## 2.1. ¿Qué problema queremos resolver?

Esta era nuestra forma de crear un producto:

```php
$producto = new Producto();
$producto->nombre = 'Teclado';
$producto->precio = 25.0;
$producto->stock = 10;
```

Funciona, pero podemos olvidar alguna asignación. Además, al ser públicas las propiedades, cualquier parte del programa puede hacer esto:

```php
$producto->precio = -25.0;
$producto->stock = -10;
```

Ambos valores respetan los tipos `float` e `int`, pero incumplen las reglas de nuestra tienda. **Los tipos no sustituyen a la validación**.

Abordaremos el problema en dos pasos: primero el constructor y después el acceso controlado a los datos.

## 2.2. El constructor: preparar el objeto al crearlo

El **constructor** es un método especial llamado `__construct`. PHP lo ejecuta automáticamente al crear un objeto mediante `new`.

Guarda esta primera versión en `Producto.php`:

```php
<?php
declare(strict_types=1);

class Producto
{
    public string $nombre;
    public float $precio;
    public int $stock;

    public function __construct(string $nombre, float $precio, int $stock)
    {
        $this->nombre = $nombre;
        $this->precio = $precio;
        $this->stock = $stock;
    }
}
```

En `index.php`, en la misma carpeta:

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Producto.php';

$teclado = new Producto('Teclado', 25.0, 10);
$raton = new Producto('Ratón', 15.0, 6);

echo htmlspecialchars($teclado->nombre, ENT_QUOTES, 'UTF-8'); // Teclado
echo '<br>';
echo $raton->stock; // 6
```

### Recorrido de la creación

Al ejecutar `new Producto('Teclado', 25.0, 10)`:

1. PHP crea un nuevo objeto de la clase `Producto`.
2. Llama automáticamente a su constructor con los argumentos indicados.
3. Los parámetros reciben `'Teclado'`, `25.0` y `10`.
4. El constructor asigna esos valores a las propiedades del objeto mediante `$this`.
5. Si la construcción termina correctamente, la expresión `new` proporciona el objeto y lo asignamos a `$teclado`.

No llamamos después a `$teclado->__construct(...)`: el constructor se utiliza automáticamente como parte de la creación. Tampoco lleva un tipo de retorno, ni siquiera `: void`, y no necesita devolver `$this`.

### Dos nombres iguales que representan cosas distintas

```php
$this->precio = $precio;
```

| Expresión | Qué representa |
| --- | --- |
| `$this->precio` | La propiedad que pertenece al objeto. |
| `$precio` | El parámetro que recibe el constructor en esa llamada. |
| `=` | Asigna a la propiedad el valor recibido en el parámetro. |

Los nombres pueden coincidir, pero no son la misma variable. El parámetro se utiliza durante la llamada; la propiedad conserva el dato en el objeto después de terminar el constructor.

!!! note "Todavía no hemos protegido los datos"
    Esta versión inicializa las propiedades, pero siguen siendo públicas y no valida los valores. El constructor no realiza esas comprobaciones automáticamente: debemos programarlas.

## 2.3. Parámetros obligatorios y valores por defecto

Podemos decidir que un producto nuevo tenga stock cero si no indicamos otro valor:

```php
public function __construct(string $nombre, float $precio, int $stock = 0)
{
    $this->nombre = $nombre;
    $this->precio = $precio;
    $this->stock = $stock;
}
```

Ahora son válidas estas dos llamadas:

```php
$teclado = new Producto('Teclado', 25.0, 10); // Stock 10.
$raton = new Producto('Ratón', 15.0);        // Stock 0.
```

`nombre` y `precio` siguen siendo obligatorios. `new Producto()` ya no es una llamada válida a esta versión de la clase: faltan esos argumentos. Colocaremos los parámetros opcionales después de los obligatorios, como hacíamos en las funciones.

Podemos añadir métodos con normalidad:

```php
public function calcularImporte(int $unidades): float
{
    return $this->precio * $unidades;
}
```

El constructor prepara los datos que utilizarán las siguientes llamadas a los métodos.

## 2.4. Encapsulación y visibilidad

La **encapsulación** consiste en reunir datos y operaciones en una clase y controlar el acceso a su estado. Permite que la clase decida qué operaciones ofrece y qué reglas deben respetarse.

En lugar de permitir cualquier asignación desde fuera, podremos ofrecer operaciones concretas como cambiar un precio, reponer unidades o realizar una venta.

PHP dispone de tres niveles básicos de visibilidad para propiedades y métodos:

| Visibilidad | Desde dónde se puede acceder |
| --- | --- |
| `public` | Desde dentro de la clase y desde el código exterior. |
| `private` | Desde el código de la propia clase que declara el elemento. |
| `protected` | Desde la clase y en el ámbito permitido de su jerarquía de herencia. Lo trabajaremos al estudiar herencia. |

En este apartado utilizaremos `private` para las propiedades y `public` para los métodos que necesitamos llamar desde la página.

```php
private string $nombre;
private float $precio;
private int $stock;
```

Con esas declaraciones:

```php
// Desde index.php: producirían un Error de acceso.
echo $teclado->precio;
$teclado->precio = -25.0;
```

En cambio, dentro de la clase podemos seguir utilizando `$this->precio`, por ejemplo en el constructor y en `calcularImporte()`.

**Privado no significa cifrado ni oculto al servidor.** Es una regla de acceso del lenguaje para organizar el código y evitar usos incorrectos. No sustituye a la seguridad de la aplicación.

## 2.5. Getters: consultar sin permitir la escritura directa

Un **getter** es un método que devuelve un dato del objeto. El nombre es una convención, no una palabra reservada de PHP.

```php
public function getNombre(): string
{
    return $this->nombre;
}

public function getPrecio(): float
{
    return $this->precio;
}

public function getStock(): int
{
    return $this->stock;
}
```

Desde la página consultamos el precio mediante una llamada:

```php
echo $teclado->getPrecio();
```

El método es público y puede leer la propiedad privada porque su código está definido dentro de la clase. La página recibe el resultado, pero no adquiere por ello acceso directo a la propiedad.

Esto permite mantener un dato consultable sin ofrecer una operación para modificarlo. Por ejemplo, en nuestra clase el nombre se establecerá al crear el producto y no ofreceremos por ahora un método para cambiarlo.

## 2.6. Setters y cambios controlados

Un **setter** es un método que permite asignar o modificar una propiedad. También es una convención de diseño: PHP no obliga a crearlo.

Este setter no aporta validación:

```php
public function setPrecio(float $precio): void
{
    $this->precio = $precio;
}
```

Aunque la propiedad sea privada, el método acepta precios negativos. Encapsular no consiste simplemente en crear getters y setters para todo: debemos decidir qué operaciones necesita el programa y qué reglas deben cumplir.

Vamos a controlar el cambio y devolver un booleano para informar de si se ha realizado:

```php
public function setPrecio(float $nuevoPrecio): bool
{
    if ($nuevoPrecio < 0) {
        return false;
    }

    $this->precio = $nuevoPrecio;
    return true;
}
```

Para los valores finitos que utilizamos en estos ejemplos, permitimos cero y precios positivos. No convertimos un precio negativo en cero ni lo aceptamos silenciosamente.

```php
if ($teclado->setPrecio(30.0)) {
    echo 'Precio actualizado';
} else {
    echo 'El precio no puede ser negativo';
}

echo $teclado->getPrecio(); // 30

$aceptado = $teclado->setPrecio(-5.0);
var_dump($aceptado);       // bool(false)
echo $teclado->getPrecio(); // 30: mantiene el valor anterior.
```

El primer `return false` termina el método antes de llegar a la asignación. Un rechazo debe dejar el objeto con su estado anterior.

## 2.7. Operaciones con significado: reponer y vender

En vez de ofrecer `setStock()` para asignar cualquier cantidad, podemos expresar las operaciones habituales de la tienda:

```php
public function reponer(int $unidades): bool
{
    if ($unidades <= 0) {
        return false;
    }

    $this->stock += $unidades;
    return true;
}

public function vender(int $unidades): bool
{
    if ($unidades <= 0 || $unidades > $this->stock) {
        return false;
    }

    $this->stock -= $unidades;
    return true;
}
```

`reponer()` acepta una cantidad positiva y la suma al stock. `vender()` solo descuenta si la cantidad es positiva y existen unidades suficientes.

En el `if` de `vender()`, `||` significa «o»: rechazamos la operación si se cumple cualquiera de las dos condiciones.

| Llamada, partiendo de stock 10 | Resultado | Stock posterior |
| --- | --- | ---: |
| `reponer(5)` | `true` | 15 |
| Después, `vender(4)` | `true` | 11 |
| Después, `vender(20)` | `false` | 11 |
| Después, `reponer(0)` | `false` | 11 |

La página debe consultar el resultado para informar de lo ocurrido. No confundas el `bool` devuelto con el número de unidades: para consultar ese dato utilizamos `getStock()`.

## 2.8. Validar también la construcción

No basta con validar los cambios posteriores. Si el constructor aceptara un precio negativo, el objeto ya empezaría con datos incorrectos.

Un constructor no puede devolver `false` como señal de fracaso. Para rechazar la construcción utilizaremos una **excepción**: una forma de interrumpir el flujo normal y comunicar que no se puede completar la operación.

Introducimos ahora la instrucción mínima necesaria; estudiaremos su tratamiento con detalle en el apartado de excepciones:

```php
throw new InvalidArgumentException('El precio no puede ser negativo.');
```

- `InvalidArgumentException` es una clase de excepción que PHP ya proporciona.
- `new` crea el objeto que describe el problema.
- El texto es el mensaje asociado.
- `throw` lanza la excepción y detiene el recorrido normal de ese método.

Si no se captura, la petición termina con un error. Por ahora utilizaremos datos válidos en la página principal y veremos también una prueba controlada al final del apartado.

En nuestro diseño utilizamos `false` para rechazar una operación habitual y una excepción para rechazar la construcción de un objeto inválido. Son decisiones explícitas de esta clase; PHP no impone este reparto.

## 2.9. Ejemplo completo: un producto con acceso controlado

Crea una carpeta `u3-apartado2` dentro del proyecto publicado por Apache. Guarda estos archivos en ella. Esta versión sustituye las versiones anteriores del ejemplo dentro de esa carpeta: no declares varias veces `Producto` en una misma ejecución.

### Archivo `Producto.php`

```php
<?php
declare(strict_types=1);

class Producto
{
    private string $nombre;
    private float $precio;
    private int $stock;

    public function __construct(string $nombre, float $precio, int $stock = 0)
    {
        $nombre = trim($nombre);

        if ($nombre === '') {
            throw new InvalidArgumentException('El nombre no puede estar vacío.');
        }
        if ($precio < 0) {
            throw new InvalidArgumentException('El precio no puede ser negativo.');
        }
        if ($stock < 0) {
            throw new InvalidArgumentException('El stock no puede ser negativo.');
        }

        $this->nombre = $nombre;
        $this->precio = $precio;
        $this->stock = $stock;
    }

    public function getNombre(): string
    {
        return $this->nombre;
    }

    public function getPrecio(): float
    {
        return $this->precio;
    }

    public function getStock(): int
    {
        return $this->stock;
    }

    public function setPrecio(float $nuevoPrecio): bool
    {
        if ($nuevoPrecio < 0) {
            return false;
        }
        $this->precio = $nuevoPrecio;
        return true;
    }

    public function reponer(int $unidades): bool
    {
        if ($unidades <= 0) {
            return false;
        }
        $this->stock += $unidades;
        return true;
    }

    public function vender(int $unidades): bool
    {
        if ($unidades <= 0 || $unidades > $this->stock) {
            return false;
        }
        $this->stock -= $unidades;
        return true;
    }

    public function calcularImporte(int $unidades): float
    {
        if ($unidades <= 0) {
            throw new InvalidArgumentException('La cantidad debe ser positiva.');
        }
        return $this->precio * $unidades;
    }
}
```

`trim()` elimina los espacios al principio y al final del nombre. Primero comprobamos los datos y después los guardamos. El nombre vacío y las cantidades negativas quedan rechazados.

`calcularImporte()` calcula un presupuesto: no comprueba existencias ni realiza una venta. Puede presupuestar más unidades de las disponibles. A diferencia de una venta rechazada, un cálculo con cantidad inválida no dispone de un importe que devolver: hemos decidido comunicar ese caso mediante una excepción.

### Archivo `index.php`

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Producto.php';

$producto = new Producto('Teclado', 25.0, 10);

$reposicionCorrecta = $producto->reponer(5);
$ventaCorrecta = $producto->vender(4);
$ventaExcesiva = $producto->vender(20);
$precioAceptado = $producto->setPrecio(-5.0);

$importe = $producto->calcularImporte(3);
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Constructor y encapsulación</title>
</head>
<body>
    <h1><?= htmlspecialchars($producto->getNombre(), ENT_QUOTES, 'UTF-8') ?></h1>

    <p>Precio: <?= number_format($producto->getPrecio(), 2, ',', '.') ?> €</p>
    <p>Stock final: <?= $producto->getStock() ?></p>
    <p>Importe de tres unidades: <?= number_format($importe, 2, ',', '.') ?> €</p>

    <?php if ($reposicionCorrecta): ?>
        <p>Reposición de cinco unidades realizada.</p>
    <?php endif; ?>

    <?php if ($ventaCorrecta): ?>
        <p>Venta de cuatro unidades realizada.</p>
    <?php endif; ?>

    <?php if (!$ventaExcesiva): ?>
        <p>No se han podido vender veinte unidades.</p>
    <?php endif; ?>

    <?php if (!$precioAceptado): ?>
        <p>No se ha aceptado el precio negativo.</p>
    <?php endif; ?>
</body>
</html>
```

El operador `!` niega un booleano: `!$ventaExcesiva` es verdadero cuando `$ventaExcesiva` es falso. Los métodos ya se han ejecutado antes del HTML; las variables guardan el resultado de cada operación para poder mostrarlo después.

Abre `http://localhost:8080/u3-apartado2/` si utilizas el puerto 8080. El precio final será **25,00 €**, el stock **11** y el importe de tres unidades **75,00 €**. Las dos operaciones rechazadas mantienen los valores anteriores.

### Prueba controlada de un constructor inválido

Crea otro archivo, `prueba-error.php`, en la misma carpeta:

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Producto.php';

try {
    $producto = new Producto('Teclado', -25.0, 10);
    echo 'Producto creado';
} catch (InvalidArgumentException $error) {
    echo htmlspecialchars($error->getMessage(), ENT_QUOTES, 'UTF-8');
}
```

El navegador mostrará **«El precio no puede ser negativo.»**. La instrucción `echo 'Producto creado'` no llega a ejecutarse.

`try` contiene la operación que puede fallar. `catch` captura la excepción de ese tipo y permite informar del problema. `$error` es la variable que recibe la excepción y `getMessage()` devuelve su mensaje. No necesitamos memorizar todavía toda la gestión de excepciones; aquí nos interesa comprender por qué no se entrega un producto con datos inválidos.

## 2.10. Promoción de propiedades del constructor

Desde PHP 8 podemos declarar algunas propiedades directamente en los parámetros del constructor. Esta sintaxis se llama **promoción de propiedades**.

Compáralas con una clase pequeña, independiente de `Producto`.

### Forma explícita

```php
class Categoria
{
    private string $nombre;

    public function __construct(string $nombre)
    {
        $this->nombre = $nombre;
    }

    public function getNombre(): string
    {
        return $this->nombre;
    }
}
```

### Forma con promoción

```php
class Categoria
{
    public function __construct(private string $nombre)
    {
    }

    public function getNombre(): string
    {
        return $this->nombre;
    }
}
```

Son **dos alternativas**, no dos clases que debamos declarar juntas. En este ejemplo ambas permiten:

```php
$categoria = new Categoria('Periféricos');
echo $categoria->getNombre(); // Periféricos
```

Al escribir `private string $nombre` en el constructor, PHP declara la propiedad y realiza su inicialización a partir del argumento. No debemos declarar otra vez esa misma propiedad en el cuerpo de la clase.

La visibilidad es la diferencia clave: `string $nombre` declara solo un parámetro; `private string $nombre` lo promociona a propiedad privada.

La promoción no añade validación automáticamente. Para aprender y depurar utilizaremos primero la forma explícita; la abreviada será útil cuando comprendamos qué código evita repetir. En el ejemplo `Producto` mantendremos la forma explícita para seguir con claridad las comprobaciones y asignaciones.

## 2.11. Decidir qué operaciones ofrecer

No todas las propiedades necesitan un setter. Debemos pensar qué cambios tienen sentido para la aplicación:

| Dato | Cómo se establece | Qué operaciones ofrecemos |
| --- | --- | --- |
| Nombre | En el constructor. | Consulta con `getNombre()`. |
| Precio | En el constructor. | Consulta y cambio validado con `setPrecio()`. |
| Stock | En el constructor. | Consulta, reposición y venta controladas. |

Esta decisión no significa que PHP impida crear otros métodos. Expresa las reglas que hemos elegido para esta versión de nuestra tienda.

Las clases tampoco necesitan imprimir mensajes de éxito ni HTML. Devuelven información o comunican errores; la página decide qué mostrar. Así mantenemos responsabilidades claras y podemos reutilizar la clase.

## 2.12. Errores frecuentes

| Error | Explicación y corrección |
| --- | --- |
| Escribir `__constructor` o `_construct`. | El nombre especial es `__construct`, con dos guiones bajos. |
| Añadir `: void` al constructor. | Los constructores no declaran tipo de retorno. |
| Seguir utilizando `new Producto()` sin argumentos. | La nueva versión necesita nombre y precio. |
| Asignar `$nombre = $nombre`. | Solo reasigna el parámetro. Para guardar el dato usamos `$this->nombre = $nombre`. |
| Leer `$producto->precio` después de hacerlo privado. | Desde fuera utilizamos `$producto->getPrecio()`. |
| Crear un setter que acepta cualquier valor. | Si hay reglas de validez, deben comprobarse antes de modificar la propiedad. |
| Restar stock antes de comprobar la venta. | Primero validamos; si se rechaza, el stock debe permanecer igual. |
| Ignorar el booleano que devuelve una operación. | Podríamos mostrar un mensaje de éxito aunque no se haya realizado. |
| Esperar que `private` valide automáticamente. | La visibilidad controla el acceso; las reglas se programan en los métodos. |
| Declarar dos veces una propiedad promocionada. | La declaración dentro del constructor ya crea esa propiedad. |
| Copiar todas las versiones de `Producto` en la misma petición. | Son alternativas progresivas. Mantén una definición por ejecución. |

## Comprueba lo aprendido

Antes de continuar, deberías poder:

1. Explicar cuándo se ejecuta `__construct` y qué argumentos recibe.
2. Distinguir un parámetro del constructor de una propiedad del objeto.
3. Crear objetos con argumentos obligatorios y con valores por defecto.
4. Explicar la diferencia entre acceso público y privado.
5. Consultar una propiedad privada mediante un método público.
6. Justificar por qué no creamos setters para todas las propiedades.
7. Programar una modificación que deje el estado intacto si se rechaza.
8. Distinguir validación, tipos y visibilidad.
9. Reconocer una excepción lanzada por un constructor inválido.
10. Comparar la declaración explícita y la promoción de propiedades.

## Practica lo aprendido

Realiza los **ejercicios 6 a 10 del bloque 2: Constructor y encapsulación** de la [relación de actividades](actividades.md#bloque-2-constructor-y-encapsulacion).

Trabajarás constructores, argumentos opcionales, propiedades privadas, getters, modificaciones controladas y promoción de propiedades. También comprobarás que las operaciones rechazadas mantienen el estado anterior del objeto.

Los ejercicios se trabajan en casa; en clase revisaremos soluciones, dudas y errores frecuentes. Entrega juntos los archivos del bloque en Moodle, según las indicaciones de clase.

En el siguiente apartado aprenderemos a **relacionar objetos mediante composición y distinguir los elementos de instancia de los elementos de clase**.

## Fuentes de consulta

- [Aitor Medrano y Luis Alemañ: PHP orientado a objetos](https://aitor-medrano.github.io/dwes2122/03phpoo.html).
- Manual oficial de PHP: [constructores](https://www.php.net/manual/es/language.oop5.decon.php), [visibilidad](https://www.php.net/manual/es/language.oop5.visibility.php), [promoción de propiedades](https://www.php.net/manual/es/language.oop5.decon.php#language.oop5.decon.constructor.promotion) y [excepciones](https://www.php.net/manual/es/language.exceptions.php).

Material elaborado por **Joaquín García** para DWES de 2.º DAW del **IES Portada Alta**. Adapta y amplía contenidos de Aitor Medrano y Luis Alemañ. Licencia **CC BY-NC-SA**.
