# 4. Herencia y polimorfismo

Ya sabemos crear objetos, proteger sus datos y relacionarlos con otros objetos. Ahora aprenderemos a definir **clases que especializan una clase existente** y a utilizar distintos tipos de productos mediante operaciones comunes.

Trabajaremos con productos físicos y digitales. Ambos tienen nombre y precio, pero sus características y gastos de envío son diferentes. Las reglas de envío serán didácticas: para un producto físico, 2 euros por kilogramo; para uno digital, cero euros. Calcularemos el envío de una unidad, sin impuestos ni descuentos.

## 4.1. De «tiene un» a «es un»

En el apartado anterior, un producto **tiene una categoría**. Guardábamos un objeto `Categoria` en una propiedad de `Producto`.

La herencia expresa otra relación: un producto físico **es un producto**. Queremos conservar sus características comunes y añadir otras específicas.

| Relación | Ejemplo | Forma de representarla |
| --- | --- | --- |
| Tiene un objeto relacionado | Un producto tiene una categoría. | Una propiedad de tipo `Categoria`. |
| Es una especialización | Un producto físico es un producto. | `class ProductoFisico extends Producto`. |

La **clase padre**, también llamada base o superclase, define lo común. La **clase hija**, derivada o subclase, puede añadir características y especializar operaciones.

No debemos elegir herencia únicamente porque dos clases tengan algún dato parecido. La relación debe tener sentido: donde el programa espera un producto, debería poder trabajar también con un producto físico.

PHP permite que una clase extienda directamente **una sola clase padre**. Una jerarquía puede tener varios niveles, pero no utilizaremos jerarquías largas en esta unidad.

## 4.2. Primera herencia con extends

Crea la carpeta `unidad-03/ejemplos04/01-herencia`. Este ejemplo es independiente de los productos de apartados anteriores.

**Archivo `Producto.php`**

```php
<?php
declare(strict_types=1);

class Producto
{
    private string $nombre;
    private float $precio;

    public function __construct(string $nombre, float $precio)
    {
        $nombre = trim($nombre);
        if ($nombre === '') {
            throw new InvalidArgumentException('El nombre no puede estar vacío.');
        }
        if ($precio < 0) {
            throw new InvalidArgumentException('El precio no puede ser negativo.');
        }
        $this->nombre = $nombre;
        $this->precio = $precio;
    }

    public function getNombre(): string
    {
        return $this->nombre;
    }

    public function getPrecio(): float
    {
        return $this->precio;
    }

    public function obtenerDescripcion(): string
    {
        return 'Producto: ' . $this->nombre;
    }
}
```

**Archivo `ProductoFisico.php`**

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Producto.php';

class ProductoFisico extends Producto
{
}
```

`extends Producto` indica de qué clase hereda. Aunque el cuerpo esté vacío, `ProductoFisico` puede utilizar los métodos públicos heredados y, al no definir constructor propio, hereda el constructor público del padre.

**Archivo `index.php`**

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/ProductoFisico.php';

$teclado = new ProductoFisico('Teclado', 25.0);

echo htmlspecialchars($teclado->obtenerDescripcion(), ENT_QUOTES, 'UTF-8');
echo '<br>';
echo number_format($teclado->getPrecio(), 2, ',', '.') . ' €';
```

Abre `http://localhost:8080/unidad-03/ejemplos04/01-herencia/`.

```text
Producto: Teclado
25,00 €
```

Hemos creado **un único objeto**, de clase `ProductoFisico`. No hay un objeto padre separado dentro de otro objeto hijo. Los métodos definidos en `Producto` trabajan sobre el mismo objeto mediante `$this`.

Las propiedades privadas del padre forman parte del estado gestionado por su código. Los getters heredados pueden consultarlas, aunque el código de la hija no pueda acceder a ellas directamente.

## 4.3. Visibilidad: public, protected y private

La herencia no elimina la encapsulación. Esta tabla se refiere a miembros ordinarios de instancia; trabajamos los accesos sencillos dentro de esta jerarquía:

| Visibilidad | Código de la clase que lo declara | Código de una clase hija | Página exterior |
| --- | --- | --- | --- |
| `public` | Sí. | Sí. | Sí. |
| `protected` | Sí. | Sí. | No. |
| `private` | Sí. | No directamente. | No. |

En nuestra clase `Producto`, la hija puede consultar el nombre mediante `$this->getNombre()`, pero no mediante acceso directo a `$this->nombre`.

Para ver `protected`, este ejemplo completo puede ejecutarse en un archivo independiente `prueba-visibilidad.php`:

```php
<?php
declare(strict_types=1);

class BaseVisibilidad
{
    private string $nombre = 'Teclado';
    protected string $familia = 'Informática';

    public function getNombre(): string
    {
        return $this->nombre;
    }
}

class HijaVisibilidad extends BaseVisibilidad
{
    public function obtenerResumen(): string
    {
        return $this->getNombre() . ' / ' . $this->familia;
    }
}

$objeto = new HijaVisibilidad();
echo htmlspecialchars($objeto->obtenerResumen(), ENT_QUOTES, 'UTF-8');
// Teclado / Informática

// No ejecutar: la página no tiene acceso a la propiedad protegida.
// echo $objeto->familia;
```

Dentro de la hija, el nombre se consulta mediante el getter público y la familia se consulta directamente porque es `protected`. Ambas consultas utilizan el mismo objeto.

**No convertiremos todas las propiedades en protected por utilizar herencia.** Mantenerlas privadas y ofrecer métodos adecuados permite que la clase padre siga controlando sus reglas. En el ejemplo conductor conservaremos nombre y precio privados.

## 4.4. Constructor de la hija y parent::__construct()

Ahora queremos que un producto físico tenga un peso. En la carpeta del primer ejemplo, conserva `Producto.php` y sustituye `ProductoFisico.php` por esta versión:

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Producto.php';

class ProductoFisico extends Producto
{
    private float $peso;

    public function __construct(string $nombre, float $precio, float $peso)
    {
        if ($peso <= 0) {
            throw new InvalidArgumentException('El peso debe ser positivo.');
        }

        parent::__construct($nombre, $precio);
        $this->peso = $peso;
    }

    public function getPeso(): float
    {
        return $this->peso;
    }
}
```

Adapta `index.php`:

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/ProductoFisico.php';

$teclado = new ProductoFisico('Teclado', 25.0, 0.5);

echo htmlspecialchars($teclado->getNombre(), ENT_QUOTES, 'UTF-8');
echo '<br>';
echo number_format($teclado->getPrecio(), 2, ',', '.') . ' €';
echo '<br>';
echo $teclado->getPeso() . ' kg';
```

Mostrará `Teclado`, `25,00 €` y `0.5 kg`, en líneas distintas.

### ¿Qué sucede durante new?

1. Se crea el objeto de tipo `ProductoFisico` y se ejecuta su constructor.
2. Se comprueba el peso.
3. `parent::__construct($nombre, $precio)` ejecuta el constructor del padre sobre **el mismo objeto**: valida y guarda nombre y precio.
4. La hija guarda su peso.
5. Si todo termina correctamente, obtenemos el objeto inicializado.

`parent` permite referirnos al código del padre desde la clase hija. **Esta llamada con `::` no convierte el constructor en estático**: estamos reutilizando el constructor del padre en el contexto del objeto actual.

| Situación | Comportamiento |
| --- | --- |
| La hija no define constructor. | Puede heredar el constructor accesible del padre. |
| La hija define constructor. | PHP no añade automáticamente la llamada al del padre. |
| Queremos ejecutar también el constructor del padre. | Escribimos `parent::__construct(...)`. |

Si omitimos esa llamada en nuestro ejemplo, nombre y precio quedarían sin inicializar. Al leerlos mediante los getters se produciría un error de propiedad tipada no inicializada.

## 4.5. Sobrescribir un método y reutilizar parent::

**Sobrescribir** consiste en definir en la hija un método heredado con el mismo nombre y una firma compatible para especializar su comportamiento.

Añade este método dentro de `ProductoFisico`, junto a `getPeso()`:

```php
public function obtenerDescripcion(): string
{
    return parent::obtenerDescripcion() . ' / Peso: ' . $this->peso . ' kg';
}
```

Y llama desde `index.php`:

```php
echo htmlspecialchars($teclado->obtenerDescripcion(), ENT_QUOTES, 'UTF-8');
// Producto: Teclado / Peso: 0.5 kg
```

PHP utiliza la versión de `ProductoFisico`. Dentro de ella, `parent::obtenerDescripcion()` ejecuta la implementación del padre y devuelve `Producto: Teclado`. La hija añade la información del peso.

Si escribiésemos `$this->obtenerDescripcion()` dentro de ese mismo método, volveríamos a entrar en la versión de la hija: provocaríamos llamadas recursivas sin fin. Para reutilizar expresamente la versión del padre usamos `parent::`.

En nuestros ejemplos mantendremos los mismos parámetros y tipos de retorno al sobrescribir. También mantendremos la visibilidad pública: no podemos restringir a `private` o `protected` un método heredado como público. Las reglas avanzadas de compatibilidad de tipos quedan fuera de esta introducción.

## 4.6. Polimorfismo: el mismo mensaje, distintos comportamientos

El **polimorfismo permite tratar un objeto hijo como del tipo padre y ejecutar la versión del método que corresponde al hijo**. En esta explicación nos referimos a métodos de instancia definidos en el tipo común y especializados por las hijas.

Dicho de forma cotidiana: damos la misma orden a productos diferentes y cada uno la realiza según sus características. «Calcula tu envío» significa calcular según el peso para el físico y devolver cero para el digital.

El objeto **conserva su clase real**. Que un parámetro esté declarado como `Producto` no transforma un producto físico en un objeto genérico, ni crea una copia. La declaración expresa el tipo de objetos que admite la función; PHP selecciona la implementación según el objeto recibido.

Antes de introducir abstracción, vamos a probarlo con métodos concretos. En la carpeta del ejemplo, añade a `Producto`:

```php
public function calcularGastosEnvio(): float
{
    return 0.0;
}
```

Sobrescríbelo dentro de `ProductoFisico`:

```php
public function calcularGastosEnvio(): float
{
    return $this->peso * 2.0;
}
```

Crea `ProductoDigital.php`:

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Producto.php';

class ProductoDigital extends Producto
{
    public function calcularGastosEnvio(): float
    {
        return 0.0;
    }
}
```

El digital hereda el constructor del padre. Su método hace explícita la regla de envío gratuito, aunque en esta versión coincida con la implementación base.

### Primero, dos llamadas sin array

Antes de recorrer una lista, crea `prueba-polimorfismo.php` en la misma carpeta. Utiliza las clases de este punto, todavía sin convertir `Producto` en abstracta:

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/ProductoFisico.php';
require_once __DIR__ . '/ProductoDigital.php';

function consultarEnvio(Producto $producto): float
{
    return $producto->calcularGastosEnvio();
}

$teclado = new ProductoFisico('Teclado', 25.0, 0.5);
$curso = new ProductoDigital('Curso PHP', 20.0);

echo consultarEnvio($teclado); // 1
echo '<br>';
echo consultarEnvio($curso);   // 0
```

La función siempre ejecuta la misma instrucción. No necesita conocer la fórmula del producto físico ni la del digital:

| Llamada | Tipo que acepta el parámetro | Objeto real recibido | Implementación ejecutada |
| --- | --- | --- | --- |
| `consultarEnvio($teclado)` | `Producto` | `ProductoFisico` | Multiplica su peso por 2. |
| `consultarEnvio($curso)` | `Producto` | `ProductoDigital` | Devuelve cero. |

Nosotros programamos las implementaciones en las clases hijas; **PHP selecciona cuál ejecutar en cada llamada**. Si añadimos otra especialización compatible que tenga su propio cálculo, esta función podrá utilizarla sin cambiar su cuerpo.

El tipo padre describe las operaciones comunes que queremos utilizar. En este ejemplo llamamos a `calcularGastosEnvio()`, que forma parte de `Producto`; no estamos afirmando que todas las hijas tengan métodos exclusivos como `getPeso()`.

### Después, recorrer un array

Sustituye `index.php`:

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/ProductoFisico.php';
require_once __DIR__ . '/ProductoDigital.php';

function calcularTotal(Producto $producto): float
{
    return $producto->getPrecio() + $producto->calcularGastosEnvio();
}

$productos = [
    new ProductoFisico('Teclado', 25.0, 0.5),
    new ProductoDigital('Curso PHP', 20.0),
];

foreach ($productos as $producto) {
    echo htmlspecialchars($producto->getNombre(), ENT_QUOTES, 'UTF-8');
    echo ': ' . number_format(calcularTotal($producto), 2, ',', '.') . ' €<br>';
}
```

Resultado:

```text
Teclado: 26,00 €
Curso PHP: 20,00 €
```

La función recibe un `Producto`. Puede recibir tanto un `ProductoFisico` como un `ProductoDigital` porque ambos son especializaciones de ese tipo.

No hemos escrito un `if` para averiguar la clase y elegir la fórmula: cada objeto responde a `calcularGastosEnvio()` con su implementación.

### ¿El polimorfismo lo hace el foreach?

El array almacena los objetos que hemos creado. En cada vuelta, `$producto` permite acceder a uno de ellos:

| Vuelta | Objeto real | Resultado de calcularGastosEnvio() |
| --- | --- | ---: |
| Primera | El teclado físico. | 1 euro. |
| Segunda | El curso digital. | 0 euros. |

**El foreach recorre los objetos; PHP selecciona el método según el objeto real.** El bucle no transforma objetos ni decide las fórmulas. La prueba anterior ya mostraba polimorfismo sin utilizar arrays.

### Comprobar el tipo con instanceof

```php
$teclado = new ProductoFisico('Teclado', 25.0, 0.5);
var_dump($teclado instanceof ProductoFisico); // bool(true)
var_dump($teclado instanceof Producto);       // bool(true)
var_dump($teclado instanceof ProductoDigital); // bool(false)
```

`instanceof` comprueba si un objeto pertenece al tipo indicado, incluyendo la relación de herencia. Es útil para comprobar tipos; no necesitamos utilizarlo para seleccionar el cálculo del envío en este diseño.

## 4.7. Clases y métodos abstractos

En nuestra aplicación queremos crear productos físicos o digitales, pero no productos genéricos cuyo tipo de envío no esté definido.

Podemos expresar esa decisión mediante una **clase abstracta**: sirve como base y no puede instanciarse directamente. Además, un **método abstracto** declara una operación cuya implementación deberá proporcionar una clase concreta descendiente.

En una nueva carpeta `unidad-03/ejemplos04/02-catalogo-abstracto` construiremos la versión completa del punto 4.9. Estas son las dos declaraciones nuevas que utilizaremos:

```php
abstract class Producto
{
    // Propiedades, constructor y métodos comunes...
    abstract public function calcularGastosEnvio(): float;
}
```

Este fragmento muestra la sintaxis; el archivo completo aparece en el punto 4.9.

| Declaración | Significado |
| --- | --- |
| `abstract class Producto` | No podemos hacer `new Producto(...)`. |
| `abstract public function calcularGastosEnvio(): float;` | Declara nombre, visibilidad, parámetros y retorno; no contiene instrucciones. |
| Una hija concreta implementa ese método. | Debe aportar su cuerpo con una firma compatible. |

El método abstracto termina en **punto y coma**, sin llaves. Una clase con métodos abstractos debe declararse abstracta. Una clase hija puede aplazar la implementación si también es abstracta; una hija concreta debe tener implementados todos los métodos abstractos heredados.

Una clase abstracta sí puede tener propiedades privadas, constructor y métodos completos. Su constructor se ejecutará al construir una hija cuando corresponda llamarlo; no crea un objeto padre separado.

La abstracción no es necesaria para que exista polimorfismo: ya lo hemos utilizado en el punto anterior. Ahora añadimos una obligación de diseño: cada producto concreto debe proporcionar el cálculo del envío.

## 4.8. Clases y métodos final

`final` permite impedir determinadas ampliaciones de una jerarquía.

### Clase final

```php
final class ProductoDigital extends Producto
{
    // Implementación completa en el siguiente punto.
}
```

Una clase final puede instanciarse si es concreta y puede heredar de otra clase, pero **no puede utilizarse como padre de una nueva clase**.

```php
// No ejecutar: ProductoDigital es final.
// class ProductoDigitalEspecial extends ProductoDigital {}
```

### Método final

En el catálogo completo declararemos así la consulta del precio:

```php
final public function getPrecio(): float
{
    return $this->precio;
}
```

Las hijas pueden utilizar ese método, pero no sobrescribirlo. Es una decisión del ejemplo: queremos conservar la consulta del precio base tal como la define `Producto`.

**Un método final no convierte en inmutables las propiedades ni todo el objeto.** Solo impide sobrescribir ese método en las clases hijas.

| Declaración | ¿Permite crear objetos directamente? | Efecto sobre la herencia |
| --- | --- | --- |
| Clase concreta ordinaria | Sí. | Puede tener hijas, si no hay otra restricción. |
| Clase abstracta | No. | Sirve de base; puede exigir implementaciones. |
| Clase final concreta | Sí. | No puede tener hijas. |
| Método abstracto | No tiene cuerpo. | Una descendiente concreta debe implementarlo. |
| Método final | Tiene implementación. | Las hijas no pueden sobrescribirlo. |

No combinamos `abstract` y `final` en una misma clase o método: en esos casos las restricciones serían incompatibles. No son modificadores de visibilidad ni equivalen a `static`.

## 4.9. Catálogo completo con herencia y polimorfismo

Crea los cuatro archivos siguientes dentro de `unidad-03/ejemplos04/02-catalogo-abstracto`. Esta versión es independiente de la primera carpeta: no cargues sus clases simultáneamente con otras clases del mismo nombre.

### Archivo Producto.php

```php
<?php
declare(strict_types=1);

abstract class Producto
{
    private string $nombre;
    private float $precio;

    public function __construct(string $nombre, float $precio)
    {
        $nombre = trim($nombre);
        if ($nombre === '') {
            throw new InvalidArgumentException('El nombre no puede estar vacío.');
        }
        if ($precio < 0) {
            throw new InvalidArgumentException('El precio no puede ser negativo.');
        }
        $this->nombre = $nombre;
        $this->precio = $precio;
    }

    public function getNombre(): string
    {
        return $this->nombre;
    }

    final public function getPrecio(): float
    {
        return $this->precio;
    }

    public function obtenerDescripcion(): string
    {
        return 'Producto: ' . $this->nombre;
    }

    public function calcularTotal(): float
    {
        return $this->precio + $this->calcularGastosEnvio();
    }

    abstract public function calcularGastosEnvio(): float;
}
```

`calcularTotal()` está implementado en el padre, pero llama a `$this->calcularGastosEnvio()`. Como `$this` representa el objeto real, se ejecutará el cálculo de la hija correspondiente.

### Archivo ProductoFisico.php

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Producto.php';

class ProductoFisico extends Producto
{
    private float $peso;

    public function __construct(string $nombre, float $precio, float $peso)
    {
        if ($peso <= 0) {
            throw new InvalidArgumentException('El peso debe ser positivo.');
        }
        parent::__construct($nombre, $precio);
        $this->peso = $peso;
    }

    public function getPeso(): float
    {
        return $this->peso;
    }

    public function obtenerDescripcion(): string
    {
        return parent::obtenerDescripcion() . ' / Peso: ' . $this->peso . ' kg';
    }

    public function calcularGastosEnvio(): float
    {
        return $this->peso * 2.0;
    }
}
```

### Archivo ProductoDigital.php

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Producto.php';

final class ProductoDigital extends Producto
{
    public function obtenerDescripcion(): string
    {
        return parent::obtenerDescripcion() . ' / Descarga digital';
    }

    public function calcularGastosEnvio(): float
    {
        return 0.0;
    }
}
```

El digital no necesita constructor propio en este ejemplo: utiliza el constructor heredado. Es una clase final por decisión del diseño, no porque todos los productos digitales deban ser finales en cualquier aplicación.

### Archivo index.php

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/ProductoFisico.php';
require_once __DIR__ . '/ProductoDigital.php';

function obtenerTotal(Producto $producto): float
{
    return $producto->calcularTotal();
}

$productos = [
    new ProductoFisico('Teclado', 25.0, 0.5),
    new ProductoDigital('Curso PHP', 20.0),
    new ProductoFisico('Monitor', 180.0, 4.0),
];

$totalCatalogo = 0.0;
foreach ($productos as $producto) {
    $totalCatalogo += obtenerTotal($producto);
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Herencia y polimorfismo</title>
</head>
<body>
    <h1>Catálogo de productos</h1>
    <table>
        <thead>
            <tr>
                <th scope="col">Descripción</th>
                <th scope="col">Precio</th>
                <th scope="col">Envío</th>
                <th scope="col">Total</th>
            </tr>
        </thead>
        <tbody>
            <?php foreach ($productos as $producto): ?>
                <tr>
                    <td><?= htmlspecialchars($producto->obtenerDescripcion(), ENT_QUOTES, 'UTF-8') ?></td>
                    <td><?= number_format($producto->getPrecio(), 2, ',', '.') ?> €</td>
                    <td><?= number_format($producto->calcularGastosEnvio(), 2, ',', '.') ?> €</td>
                    <td><?= number_format(obtenerTotal($producto), 2, ',', '.') ?> €</td>
                </tr>
            <?php endforeach; ?>
        </tbody>
    </table>
    <p>Total del catálogo: <?= number_format($totalCatalogo, 2, ',', '.') ?> €</p>
</body>
</html>
```

Abre `http://localhost:8080/unidad-03/ejemplos04/02-catalogo-abstracto/`.

| Producto | Precio | Envío | Total |
| --- | ---: | ---: | ---: |
| Teclado, 0.5 kg | 25,00 € | 1,00 € | 26,00 € |
| Curso PHP, digital | 20,00 € | 0,00 € | 20,00 € |
| Monitor, 4 kg | 180,00 € | 8,00 € | 188,00 € |

Total del catálogo: **234,00 €**. Es la suma didáctica de una unidad de cada producto y su envío individual, no el cálculo de un paquete conjunto.

### Entender el ejemplo final antes de recorrer el catálogo

Para seguirlo sobre papel, empecemos por un solo producto. Los fragmentos siguientes explican el código completo anterior: no es necesario añadirlos al catálogo.

**Paso 1. Crear el teclado**

```php
$teclado = new ProductoFisico('Teclado', 25.0, 0.5);
```

La hija recibe los tres argumentos y comprueba que el peso sea positivo. Después ejecuta:

```php
parent::__construct($nombre, $precio);
$this->peso = $peso;
```

El constructor del padre valida y guarda nombre y precio; la hija guarda peso. Se inicializa **un único objeto de tipo ProductoFisico**.

| Dato | Código que lo gestiona | Valor en el teclado |
| --- | --- | --- |
| Nombre | Clase padre `Producto`. | Teclado |
| Precio | Clase padre `Producto`. | 25.0 |
| Peso | Clase hija `ProductoFisico`. | 0.5 |

Las propiedades privadas del padre siguen siendo privadas: su propio código puede utilizarlas al ejecutarse sobre este objeto. La hija no tiene acceso directo a ellas por haber llamado a su constructor.

**Paso 2. Pasar el teclado a una función que acepta Producto**

```php
obtenerTotal($teclado);
```

La función está definida así:

```php
function obtenerTotal(Producto $producto): float
{
    return $producto->calcularTotal();
}
```

El parámetro admite el teclado porque un producto físico es un producto. `$producto` permite acceder al mismo teclado: no crea otro objeto ni cambia su clase real.

**Paso 3. Ejecutar el método común del padre**

El método `calcularTotal()` está escrito en `Producto`:

```php
public function calcularTotal(): float
{
    return $this->precio + $this->calcularGastosEnvio();
}
```

En esa ejecución, **$this representa el teclado**, aunque estemos leyendo código escrito en el padre. `$this->precio` obtiene su precio, `25.0`.

**Paso 4. Pedir al mismo objeto su cálculo de envío**

La expresión `$this->calcularGastosEnvio()` utiliza la implementación de `ProductoFisico`:

```php
public function calcularGastosEnvio(): float
{
    return $this->peso * 2.0;
}
```

Devuelve `0.5 * 2.0 = 1.0`. El padre ha indicado qué operación necesita; la hija aporta cómo se realiza para un producto físico.

No utilizamos `parent::calcularGastosEnvio()`: en el catálogo abstracto el padre solo declara esa operación y no tiene un cuerpo que podamos ejecutar.

**Paso 5. Devolver el resultado hacia la página**

El envío vuelve al cálculo del total:

```text
calcularGastosEnvio() devuelve 1.0
calcularTotal() devuelve 25.0 + 1.0 = 26.0
obtenerTotal() devuelve 26.0 a quien la llamó
```

La página utiliza `number_format()` para presentar ese número como `26,00 €`. Los métodos han calculado números; el formato se aplica en el HTML.

### Repetir el recorrido con el producto digital

```php
$curso = new ProductoDigital('Curso PHP', 20.0);
obtenerTotal($curso);
```

`ProductoDigital` no define constructor propio: utiliza el constructor público heredado del padre. Este valida y guarda nombre y precio en el objeto digital.

| Paso | Teclado físico | Curso digital |
| --- | --- | --- |
| Clase real del objeto | `ProductoFisico` | `ProductoDigital` |
| Constructor utilizado | El de la hija, que llama al padre. | El heredado del padre. |
| Tipo admitido por obtenerTotal() | `Producto` | `Producto` |
| Implementación de calcularTotal() | La común, definida en `Producto`. | La misma implementación común. |
| Objeto representado por $this | El teclado. | El curso. |
| Precio consultado en el padre | 25.0 | 20.0 |
| Implementación de calcularGastosEnvio() | La del físico: peso por 2. | La del digital: cero. |
| Total devuelto | 26.0 | 20.0 |

No cambia la función `obtenerTotal()` ni el cuerpo del método `calcularTotal()`. Cambia el objeto sobre el que trabajan y, con él, la implementación del envío.

### Volver ahora al array y los dos recorridos

El catálogo completo crea tres objetos. Sus dos bucles tienen fines distintos:

1. El primer `foreach` obtiene cada total y lo acumula: `26 + 20 + 188 = 234`.
2. El segundo genera las filas de la tabla y vuelve a consultar descripción, precio, envío y total.

El segundo recorrido no crea otros productos. Tampoco cobra dos veces: los métodos de cálculo de este ejemplo no realizan ventas ni modifican el estado; devuelven resultados que podemos consultar varias veces.

### Las cuatro aclaraciones clave

- **El objeto conserva su clase real** al pasarlo a un parámetro del tipo padre.
- **El foreach solo recorre**: la implementación se selecciona según el objeto real.
- **$this sigue siendo el mismo objeto**, aunque el método esté escrito en el padre.
- Un método implementado en el padre **puede llamar a una operación cuya implementación aporta la hija**.

### Comprobaciones adicionales

En un archivo aparte `pruebas.php`, dentro de esta segunda carpeta:

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/ProductoFisico.php';
require_once __DIR__ . '/ProductoDigital.php';

try {
    $producto = new ProductoFisico('Teclado', 25.0, 0.0);
} catch (InvalidArgumentException $error) {
    echo htmlspecialchars($error->getMessage(), ENT_QUOTES, 'UTF-8');
}
echo '<br>';

try {
    $producto = new ProductoDigital('Curso PHP', -20.0);
} catch (InvalidArgumentException $error) {
    echo htmlspecialchars($error->getMessage(), ENT_QUOTES, 'UTF-8');
}
```

Se muestran los mensajes de peso no positivo y precio negativo. También debes reconocer estas situaciones inválidas, que dejamos comentadas para no romper los ejemplos:

```php
// No puede instanciarse una clase abstracta:
// $p = new Producto('Genérico', 10.0);

// No puede crearse una hija de ProductoDigital porque es final.
// No puede sobrescribirse getPrecio() porque es final.
```

Los errores de definición de jerarquías no deben tratarse como si fueran validaciones ordinarias del formulario: corrige la declaración de las clases.

## 4.10. Resumen y errores frecuentes

### Los siete conceptos del bloque

| Concepto | Idea que debes recordar | Ejemplo del catálogo |
| --- | --- | --- |
| Herencia | Crear una clase que especializa otra: «es un». | `ProductoFisico extends Producto`. |
| Visibilidad | Controlar quién puede acceder a cada miembro. | Nombre y precio privados; getters públicos. |
| Constructor de la hija | Preparar sus datos y llamar al padre para los comunes. | Validar peso y ejecutar `parent::__construct(...)`. |
| Sobrescritura | Proporcionar una versión especializada de un método heredado. | Añadir el peso a la descripción. |
| Polimorfismo | Tratar el objeto hijo como del tipo padre y ejecutar la versión del método que corresponde al hijo. | Una función recibe `Producto` y el envío depende del objeto real. |
| Abstracción | Definir una base que no instanciamos y operaciones pendientes de implementación. | `Producto` abstracto y envío abstracto. |
| Final | Impedir nuevas hijas o la sobrescritura de un método. | `ProductoDigital` final y `getPrecio()` final. |

### Una guía para seguir el código sobre papel

Para cada llamada, anota estas cinco preguntas:

1. ¿Qué objeto concreto se creó con `new`?
2. ¿Qué datos tiene en este momento?
3. ¿Qué función o método estamos ejecutando y en qué clase está escrito?
4. Si hay `$this`, ¿qué objeto representa? Si el método está sobrescrito, ¿qué implementación se selecciona?
5. ¿Qué valor devuelve y a qué llamada regresa?

Practica con el monitor del catálogo: empieza por `new ProductoFisico('Monitor', 180.0, 4.0)` y termina en el importe mostrado. Debes obtener un envío de `8.0` y un total de `188.0`.

### Tres referencias diferentes

| Expresión | Significado | Ejemplo |
| --- | --- | --- |
| `$this` | El objeto actual. | `$this->getNombre()` |
| `self` | La clase donde está escrito el código. | `self::IVA` en los ejemplos anteriores. |
| `parent` | La clase padre desde la hija. | `parent::__construct($nombre, $precio)` |

`::` no se limita a elementos estáticos: también lo utilizamos en `parent::` para seleccionar la implementación del padre de un método de instancia. En esos ejemplos sigue existiendo el objeto actual.

### Operaciones sobre la jerarquía

| Operación | Qué hacemos |
| --- | --- |
| Heredar | Definir una especialización con `extends`. |
| Añadir | Incorporar una propiedad o método nuevo en la hija. |
| Sobrescribir | Proporcionar una versión especializada de un método heredado. |
| Implementar un método abstracto | Escribir el cuerpo exigido por la clase base. |
| Usar polimorfismo | Trabajar mediante un tipo común y ejecutar el comportamiento de cada objeto. |

### Errores que debemos evitar

| Error | Corrección o explicación |
| --- | --- |
| Utilizar herencia para «un producto tiene una categoría». | Esa relación se representa guardando el objeto relacionado. |
| Suponer que la hija contiene un objeto padre separado. | La instancia es única; se ejecuta código de ambas clases sobre ella. |
| Acceder directamente a una propiedad privada del padre desde la hija. | Utiliza métodos accesibles que la clase padre proporcione. |
| Olvidar `parent::__construct()` al definir el constructor de la hija. | Llama al padre si necesitas que realice su inicialización. |
| Pensar que `parent::` obliga a que el método sea estático. | Permite seleccionar también la implementación de instancia del padre. |
| Llamar al método sobrescrito con `$this` desde su propia implementación sin condición de salida. | Usa `parent::` si buscas reutilizar la versión del padre. |
| Cambiar un método público heredado a privado. | No se puede restringir su visibilidad al sobrescribirlo. |
| Crear un objeto de una clase abstracta. | Crea una instancia de una hija concreta. |
| Escribir llaves con instrucciones en un método declarado abstracto. | El método abstracto solo declara la firma y termina en `;`. |
| Olvidar implementar un método abstracto en una hija concreta. | Implementa los métodos pendientes con firmas compatibles. |
| Confundir `final` con constante o inmutable. | Limita la herencia o la sobrescritura; no congela el estado. |
| Cargar distintas versiones de Producto en la misma petición. | Ejecuta cada versión desde su carpeta independiente. |

## Comprueba lo aprendido

Antes de continuar, deberías poder:

1. Distinguir «tiene un» de «es un» con ejemplos propios.
2. Declarar una hija mediante `extends` y utilizar un método heredado.
3. Explicar qué permite cada nivel de visibilidad dentro de una jerarquía.
4. Inicializar los datos comunes y específicos de un objeto hijo.
5. Sobrescribir un método y ampliar su resultado mediante `parent::`.
6. Explicar por qué una función que recibe `Producto` acepta sus especializaciones.
7. Predecir qué implementación utiliza `$this->calcularGastosEnvio()`.
8. Definir una clase abstracta con métodos comunes y una operación pendiente.
9. Diferenciar `abstract`, `final`, `static` y los modificadores de visibilidad.
10. Ejecutar y comprobar los resultados del catálogo sin mezclar versiones.

## Practica lo aprendido

Realiza los **ejercicios 16 a 21 del bloque 4: Herencia y polimorfismo** de la [relación de actividades](actividades.md#bloque-4-herencia-y-polimorfismo). Son seis ejercicios progresivos: personas y alumnos; tarifas de aparcamiento; figuras abstractas; un catálogo integrador con `final`; reparación de una jerarquía; y un ejercicio tipo examen para completar y seguir el código.

Comenzaremos los ejercicios en clase y completarás en casa lo pendiente. En la siguiente sesión pondremos en común soluciones, dudas y errores frecuentes. Las soluciones del profesor se mantendrán separadas de los enunciados.

En el siguiente apartado estudiaremos **interfaces y contratos**, incluyendo cómo expresar contratos y en qué se diferencian de las clases abstractas.

## Fuentes de consulta

- [Aitor Medrano y Luis Alemañ: PHP orientado a objetos](https://aitor-medrano.github.io/dwes2122/03phpoo.html).
- Manual oficial de PHP: [herencia](https://www.php.net/manual/es/language.oop5.inheritance.php), [visibilidad](https://www.php.net/manual/es/language.oop5.visibility.php), [constructores](https://www.php.net/manual/es/language.oop5.decon.php), [clases abstractas](https://www.php.net/manual/es/language.oop5.abstract.php) y [final](https://www.php.net/manual/es/language.oop5.final.php).

Material elaborado por **Joaquín García** para DWES de 2.º DAW del **IES Portada Alta**. Adapta y amplía contenidos de Aitor Medrano y Luis Alemañ. Licencia **CC BY-NC-SA**.
