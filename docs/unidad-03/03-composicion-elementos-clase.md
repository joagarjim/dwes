# 3. Composición y elementos de clase

Hasta ahora hemos creado objetos con propiedades sencillas, como un nombre, un precio o un saldo. Hemos utilizado constructores para inicializarlos y métodos para consultar y modificar sus datos de forma controlada.

Ahora veremos dos ideas diferentes: **cómo colaboran varios objetos** y **qué elementos pertenecen a la clase en lugar de a cada instancia**. Primero relacionaremos productos y categorías; después trabajaremos constantes, propiedades estáticas y métodos estáticos.

## 3.1. Una propiedad también puede contener un objeto

Un producto de nuestra tienda pertenece a una categoría. Podríamos guardar su nombre como una cadena:

```php
private string $categoria;
```

Pero una categoría puede tener sus propios datos y operaciones: nombre, descripción, código, etc. Podemos representarla mediante una clase y guardar un objeto de esa clase dentro de `Producto`:

```php
private Categoria $categoria;
```

El tipo `Categoria` indica que la propiedad debe contener un objeto de esa clase. Las clases que definimos también pueden utilizarse como tipos, igual que hasta ahora utilizábamos `string`, `int` o `float`.

**Un producto tiene una categoría.** No estamos diciendo que un producto sea una categoría: son elementos diferentes que colaboran.

En este apartado utilizaremos **composición de objetos en sentido amplio**: construir objetos que contienen o utilizan otros objetos.

!!! note "Una precisión sobre el modelado"
    En un diagrama UML, la composición estricta implica una relación de propiedad y un ciclo de vida ligado al objeto contenedor. Nuestro ejemplo permite compartir la misma categoría entre varios productos y crearla de forma independiente, por lo que representa una asociación compartida, no una composición estricta de UML. PHP no tiene una palabra reservada que imponga esas diferencias de diseño.

## 3.2. Definir la clase Categoria

Crea la carpeta `u3-apartado3` dentro del proyecto publicado por Apache. Guarda en ella `Categoria.php`:

```php
<?php
declare(strict_types=1);

class Categoria
{
    private string $nombre;

    public function __construct(string $nombre)
    {
        $nombre = trim($nombre);

        if ($nombre === '') {
            throw new InvalidArgumentException(
                'El nombre de la categoría no puede estar vacío.'
            );
        }

        $this->nombre = $nombre;
    }

    public function getNombre(): string
    {
        return $this->nombre;
    }
}
```

Es una clase con una responsabilidad pequeña: representar una categoría válida y permitir consultar su nombre. No genera HTML ni guarda productos.

## 3.3. Recibir otro objeto en el constructor

Guarda esta versión de `Producto` en `Producto.php`, en la misma carpeta:

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Categoria.php';

class Producto
{
    private string $nombre;
    private float $precio;
    private Categoria $categoria;

    public function __construct(
        string $nombre,
        float $precio,
        Categoria $categoria
    ) {
        $nombre = trim($nombre);

        if ($nombre === '') {
            throw new InvalidArgumentException('El nombre no puede estar vacío.');
        }

        if ($precio < 0) {
            throw new InvalidArgumentException('El precio no puede ser negativo.');
        }

        $this->nombre = $nombre;
        $this->precio = $precio;
        $this->categoria = $categoria;
    }

    public function getNombre(): string
    {
        return $this->nombre;
    }

    public function getPrecio(): float
    {
        return $this->precio;
    }

    public function getCategoria(): Categoria
    {
        return $this->categoria;
    }
}
```

Esta versión se centra en la relación entre objetos. No incorpora el stock ni todos los métodos del apartado anterior. Sustituye la clase solo en esta nueva carpeta: no cargues dos definiciones de `Producto` en una misma petición.

El parámetro `Categoria $categoria` exige un objeto. Pasar el texto `'Periféricos'` no equivale a pasar una categoría:

```php
// Incorrecto: el tercer argumento es una cadena.
// $teclado = new Producto('Teclado', 25.0, 'Periféricos');
```

### Crear y relacionar los objetos

Crea `index.php` en la misma carpeta:

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Producto.php';

$perifericos = new Categoria('Periféricos');

$teclado = new Producto('Teclado', 25.0, $perifericos);
$raton = new Producto('Ratón', 15.0, $perifericos);

$categoriaDelTeclado = $teclado->getCategoria();

echo htmlspecialchars($categoriaDelTeclado->getNombre(), ENT_QUOTES, 'UTF-8');
```

Abre `http://localhost:8080/u3-apartado3/`, si mantienes el puerto 8080. Verás **Periféricos**.

Se han creado **tres objetos**: una categoría y dos productos. El constructor de `Producto` no hace otro `new Categoria()`: guarda el objeto que recibe.

| Expresión | Qué contiene o devuelve |
| --- | --- |
| `$perifericos` | El objeto de categoría creado al principio. |
| `$teclado` | El primer producto. |
| `$raton` | Otro producto independiente. |
| `$teclado->getCategoria()` | El objeto de categoría asociado al teclado. |

Los productos son distintos, pero los dos están asociados a **la misma categoría**. Pasar el objeto al constructor no crea una copia independiente.

## 3.4. Acceder a los datos de un objeto relacionado

Podemos consultar el nombre de la categoría en dos instrucciones:

```php
$categoria = $teclado->getCategoria();
$nombreCategoria = $categoria->getNombre();
```

O escribir las dos llamadas en una expresión:

```php
$nombreCategoria = $teclado->getCategoria()->getNombre();
```

Se interpreta de izquierda a derecha:

1. `$teclado->getCategoria()` devuelve un objeto `Categoria`.
2. Sobre ese objeto llamamos a `getNombre()`.
3. El resultado final es una cadena.

Esto no accede directamente a propiedades privadas. Utiliza dos métodos públicos, cada uno responsable de consultar su propio objeto.

No debe confundirse con el encadenamiento de setters que devuelven `$this`: aquí el primer método devuelve **otro objeto**, la categoría.

### ¿Devolver el objeto crea una copia?

No. `getCategoria()` proporciona acceso al mismo objeto que guarda el producto. En esta versión `Categoria` no tiene métodos de modificación, por lo que desde su API pública solo podemos consultar su nombre.

Si más adelante añadiéramos un método público que cambiase ese nombre, el cambio sería visible desde todos los productos que compartiesen la categoría. Que la propiedad de `Producto` sea privada no convierte automáticamente en inmutable al objeto relacionado.

## 3.5. Arrays de objetos

Un array puede almacenar objetos. En el ejemplo anterior, añadimos:

```php
$productos = [$teclado, $raton];

foreach ($productos as $producto) {
    echo htmlspecialchars($producto->getNombre(), ENT_QUOTES, 'UTF-8');
    echo '<br>';
}
```

Mostrará `Teclado` y `Ratón` en líneas distintas. En cada vuelta, `$producto` permite acceder al objeto que ocupa esa posición. No se crea un nuevo producto al recorrer el array.

También podemos calcular el precio conjunto de una unidad de cada producto:

```php
$total = 0.0;

foreach ($productos as $producto) {
    $total += $producto->getPrecio();
}

echo number_format($total, 2, ',', '.') . ' €'; // 40,00 €
```

Es el mismo patrón de acumulación que ya conocemos, pero cada precio se obtiene mediante un método del objeto.

!!! warning "El tipo array no indica el tipo de sus elementos"
    PHP permite mezclar valores en un array. Si añadimos una cadena a `$productos`, la llamada a `getPrecio()` sobre ese elemento fallará. En estos ejemplos controlamos que todos los elementos sean productos. Cuando una clase gestione una colección, podremos ofrecer un método `anadirProducto(Producto $producto)` para controlar las incorporaciones.

## 3.6. Ejemplo de presentación: catálogo con categorías

Sustituye `index.php` por esta página completa. Conserva los dos archivos de clases anteriores:

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Producto.php';

$perifericos = new Categoria('Periféricos');
$pantallas = new Categoria('Pantallas');

$productos = [
    new Producto('Teclado', 25.0, $perifericos),
    new Producto('Ratón', 15.0, $perifericos),
    new Producto('Monitor', 180.0, $pantallas),
];

$total = 0.0;
foreach ($productos as $producto) {
    $total += $producto->getPrecio();
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Catálogo con categorías</title>
</head>
<body>
    <h1>Catálogo</h1>
    <table>
        <thead>
            <tr>
                <th scope="col">Producto</th>
                <th scope="col">Categoría</th>
                <th scope="col">Precio</th>
            </tr>
        </thead>
        <tbody>
            <?php foreach ($productos as $producto): ?>
                <tr>
                    <td><?= htmlspecialchars($producto->getNombre(), ENT_QUOTES, 'UTF-8') ?></td>
                    <td><?= htmlspecialchars($producto->getCategoria()->getNombre(), ENT_QUOTES, 'UTF-8') ?></td>
                    <td><?= number_format($producto->getPrecio(), 2, ',', '.') ?> €</td>
                </tr>
            <?php endforeach; ?>
        </tbody>
    </table>
    <p>Una unidad de cada producto: <?= number_format($total, 2, ',', '.') ?> €</p>
</body>
</html>
```

`foreach (...): ... endforeach;` es la sintaxis alternativa del bucle cuando intercalamos HTML. Cumple la misma función que `foreach` con llaves.

Verás tres filas, dos categorías diferentes y un total de **220,00 €**. En esta ejecución se crean cinco objetos: dos categorías y tres productos.

## 3.7. Datos de cada objeto y elementos de clase

El nombre y el precio pertenecen a cada producto: el teclado cuesta 25 euros y el ratón 15. Son **propiedades de instancia**.

Otras necesidades se refieren a la clase, sin depender de un producto concreto: un valor fijo utilizado por todos los productos o un contador de construcciones realizadas durante la ejecución.

| Elemento | A qué se refiere | Ejemplo |
| --- | --- | --- |
| Propiedad de instancia | Al estado de cada objeto. | Precio del teclado. |
| Constante de clase | A un valor definido en la clase que no se reasigna. | Porcentaje de impuesto fijado para un ejemplo. |
| Propiedad estática | A un dato de clase que puede cambiar. | Total de productos creados en la ejecución. |
| Método estático | A una operación invocable sin un objeto concreto. | Consultar ese contador. |

Estudiaremos los tres últimos con ejemplos separados para no mezclar todas las novedades en una sola clase.

## 3.8. Constantes de clase y operador `::`

Una constante permite dar un nombre a un valor que no queremos reasignar. Dentro de una clase se declara con `const` y su nombre no lleva `$`.

Este ejemplo utiliza un impuesto didáctico fijo del 21 % para todos sus artículos. Es una regla del ejemplo, no una afirmación sobre todos los productos reales.

Crea `Articulo.php` en la carpeta del apartado:

```php
<?php
declare(strict_types=1);

class Articulo
{
    public const IVA = 0.21;

    public function __construct(private float $precio)
    {
        if ($precio < 0) {
            throw new InvalidArgumentException('El precio no puede ser negativo.');
        }
    }

    public function calcularPrecioConIva(): float
    {
        return $this->precio * (1 + self::IVA);
    }
}
```

En un archivo independiente `prueba-articulo.php`:

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Articulo.php';

echo Articulo::IVA; // 0.21: no necesitamos crear un artículo.
echo '<br>';

$articulo = new Articulo(100.0);
echo number_format($articulo->calcularPrecioConIva(), 2, ',', '.') . ' €';
// 121,00 €
```

- `Articulo::IVA` consulta la constante desde fuera de la clase.
- `self::IVA` consulta esa constante desde el código de la propia clase.
- `self` hace referencia a la clase donde está escrito ese código; no representa un objeto.
- `::` es el operador de resolución de ámbito. Aquí lo usamos para acceder a elementos de clase.
- `calcularPrecioConIva()` sigue siendo un método de instancia: necesita el precio del artículo actual mediante `$this`.

No escribimos `static` para declarar una constante. Tampoco podemos hacer `Articulo::IVA = 0.10;`: una constante no se reasigna durante la ejecución.

## 3.9. Propiedades estáticas: un dato compartido

Una propiedad declarada con `static` pertenece a la clase, en lugar de existir como un dato independiente por cada objeto de esa clase.

Para verlo, crea `ProductoContado.php`:

```php
<?php
declare(strict_types=1);

class ProductoContado
{
    private static int $totalCreados = 0;
    private string $nombre;

    public function __construct(string $nombre)
    {
        $nombre = trim($nombre);

        if ($nombre === '') {
            throw new InvalidArgumentException('El nombre no puede estar vacío.');
        }

        $this->nombre = $nombre;
        self::$totalCreados++;
    }

    public function getNombre(): string
    {
        return $this->nombre;
    }

    public static function getTotalCreados(): int
    {
        return self::$totalCreados;
    }
}
```

`$nombre` tiene un valor por objeto. `$totalCreados` es un único contador de la clase en este ejemplo, sin herencia. Cada construcción válida lo incrementa.

La propiedad estática es privada: no podemos consultarla directamente desde la página. Ofrecemos un método público estático que devuelve su valor.

### Prueba paso a paso

Guarda este código en `prueba-contador.php`:

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/ProductoContado.php';

echo ProductoContado::getTotalCreados(); // 0
echo '<br>';

$primero = new ProductoContado('Teclado');
echo ProductoContado::getTotalCreados(); // 1
echo '<br>';

$segundo = new ProductoContado('Ratón');
echo ProductoContado::getTotalCreados(); // 2
echo '<br>';

$otroNombre = $primero;
echo ProductoContado::getTotalCreados(); // 2
```

Salida: **0, 1, 2, 2**, cada número en una línea. La asignación final no ejecuta `new` y no llama al constructor: no incrementa el contador.

El incremento está después de la validación. Si intentamos construir un producto con nombre vacío y capturamos la excepción, el contador no aumenta.

!!! warning "Qué estamos contando"
    Contamos construcciones válidas realizadas, no unidades de stock ni objetos que siguen vivos. El contador no se reduce al dejar de usar un objeto. Tampoco sirve como contador permanente: en nuestras peticiones web habituales se inicializa de nuevo al ejecutar otra petición. No se comparte automáticamente entre visitantes ni sustituye a una base de datos.

## 3.10. Métodos estáticos y ausencia de `$this`

Declaramos un método estático con `public static function` y lo llamamos mediante el nombre de la clase:

```php
ProductoContado::getTotalCreados();
```

No necesitamos crear un producto para realizar esa consulta. Dentro de un método estático **no existe `$this`**, porque no se está trabajando sobre una instancia concreta.

Por eso no podríamos implementar así el método:

```php
// Incorrecto dentro de un método static:
// return $this->nombre;
```

Un método estático puede recibir objetos como argumentos y utilizar sus métodos públicos, pero no dispone automáticamente de un objeto actual. No convertiremos todos los métodos en estáticos para evitar escribir `new`: perderíamos el sentido de los datos y operaciones de cada objeto.

También puede haber métodos estáticos sin propiedades estáticas. Por ejemplo, podríamos agrupar una operación de formato que solo depende de sus argumentos. Elegiremos esa organización cuando tenga sentido, no porque toda función deba convertirse en una clase.

## 3.11. Comparar las formas de acceso

| Qué utilizamos | Desde fuera de la clase | Desde dentro de la clase |
| --- | --- | --- |
| Método de instancia | `$producto->getNombre()` | `$this->getNombre()` |
| Propiedad privada de instancia | A través de un método público. | `$this->nombre` |
| Constante pública de clase | `Articulo::IVA` | `self::IVA` |
| Propiedad estática privada | A través de un método público. | `self::$totalCreados` |
| Método estático público | `ProductoContado::getTotalCreados()` | `self::getTotalCreados()` |

Fíjate en el signo `$`: la propiedad estática se escribe `self::$totalCreados`, mientras que la constante se escribe `self::IVA` y el método se llama `self::getTotalCreados()`.

El operador no elimina las reglas de visibilidad: escribir `ProductoContado::$totalCreados` desde la página no permite saltarse su declaración privada.

## 3.12. Errores frecuentes

| Error | Explicación y corrección |
| --- | --- |
| Pasar `'Periféricos'` donde se exige `Categoria`. | Debemos crear y pasar el objeto, no solo su nombre. |
| Creer que cada producto copia su categoría. | En el ejemplo ambos guardan el mismo objeto recibido. |
| Tratar el resultado de `getCategoria()` como una cadena. | Devuelve un objeto; llamamos a `getNombre()` para obtener el texto. |
| Mezclar cadenas y productos en la lista. | El recorrido espera objetos con los métodos utilizados. |
| Usar `ProductoContado->getTotalCreados()`. | Para nuestra llamada de clase usamos `ProductoContado::getTotalCreados()`. |
| Utilizar `$this` en un método estático. | No existe una instancia actual. |
| Escribir `self::totalCreados`. | Una propiedad estática conserva el `$`: `self::$totalCreados`. |
| Escribir `self::$IVA`. | Una constante no lleva `$`: `self::IVA`. |
| Declarar el precio como estático. | Los productos necesitan precios propios; aquí debe ser de instancia. |
| Esperar un contador acumulado tras recargar. | La propiedad estática no proporciona persistencia entre peticiones. |

## Comprueba lo aprendido

Antes de continuar, deberías poder:

1. Explicar qué significa que una propiedad tenga tipo `Categoria`.
2. Crear dos productos asociados a la misma categoría e identificar cuántos objetos existen.
3. Interpretar `$producto->getCategoria()->getNombre()` paso a paso.
4. Recorrer un array de productos y acumular sus precios.
5. Distinguir composición de objetos en sentido amplio de composición estricta en UML.
6. Diferenciar propiedad de instancia, constante y propiedad estática.
7. Utilizar `->`, `::`, `$this` y `self` en los ejemplos correspondientes.
8. Explicar por qué un método estático no dispone de `$this`.
9. Predecir el contador al construir objetos y al asignarlos a otras variables.
10. Explicar por qué `static` no guarda automáticamente información entre peticiones web.

## Practica lo aprendido

El **bloque 3: Composición y elementos de clase** de la [relación de actividades](actividades.md) se incorporará después de revisar este apartado. Trabajará relaciones entre objetos, arrays de objetos, constantes y elementos estáticos.

Los ejercicios se trabajan en casa; en clase revisaremos soluciones, dudas y errores frecuentes. Entrega juntos los archivos del bloque en Moodle, según las indicaciones de clase.

En el siguiente apartado estudiaremos **herencia y polimorfismo**: cómo especializar una clase y utilizar objetos de diferentes clases mediante un tipo común.

## Fuentes de consulta

- [Aitor Medrano y Luis Alemañ: PHP orientado a objetos](https://aitor-medrano.github.io/dwes2122/03phpoo.html).
- Manual oficial de PHP: [objetos y referencias](https://www.php.net/manual/es/language.oop5.references.php), [constantes de clase](https://www.php.net/manual/es/language.oop5.constants.php) y [palabra reservada static](https://www.php.net/manual/es/language.oop5.static.php).

Material elaborado por **Joaquín García** para DWES de 2.º DAW del **IES Portada Alta**. Adapta y amplía contenidos de Aitor Medrano y Luis Alemañ. Licencia **CC BY-NC-SA**.
