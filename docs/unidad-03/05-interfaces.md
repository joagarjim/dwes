# 5. Interfaces en PHP

En el apartado anterior utilizamos el tipo `Producto` para trabajar con productos físicos y digitales. Compartían una clase padre y cada uno aportaba su cálculo del envío.

Ahora queremos utilizar una misma operación con objetos que **no necesitan un padre común**. Un producto y una reserva son elementos diferentes, pero ambos pueden ofrecer una descripción. Una interfaz nos permite expresar esa capacidad común.

## 5.1. ¿Qué es una interfaz?

Una **interfaz** define operaciones que deben ofrecer las clases que la implementen. Indica cómo se podrán llamar, pero no escribe sus instrucciones.

A esa obligación la llamamos **contrato**: si una clase declara que cumple la interfaz, debe proporcionar los métodos públicos exigidos, con firmas compatibles.

«Contrato» no es una palabra reservada de PHP. Es una forma de explicar el compromiso que expresa el código.

| Elemento | Qué establece en nuestro ejemplo |
| --- | --- |
| Interfaz `Describible` | Debe existir un método público `obtenerDescripcion()` que devuelva `string`. |
| Clase `Producto` | Cómo describir un producto a partir de sus datos. |
| Clase `Reserva` | Cómo describir una reserva a partir de los suyos. |
| Código que recibe `Describible` | Puede solicitar la descripción sin depender de una clase concreta. |

**Una interfaz no crea objetos ni aporta por sí sola el cuerpo de los métodos.** Las clases concretas contienen la implementación que se ejecutará.

## 5.2. Declarar la interfaz con interface

Crea la carpeta `unidad-03/ejemplos05` dentro del proyecto publicado por Apache. Utilizaremos cuatro archivos: `Describible.php`, `Producto.php`, `Reserva.php` e `index.php`.

**Archivo `Describible.php`**

```php
<?php
declare(strict_types=1);

interface Describible
{
    public function obtenerDescripcion(): string;
}
```

| Parte | Significado |
| --- | --- |
| `interface` | Inicia la declaración de una interfaz. |
| `Describible` | Nombre elegido para expresar una capacidad. |
| `public` | El método debe poder utilizarse desde fuera. Los métodos declarados en una interfaz son públicos. |
| `obtenerDescripcion()` | Nombre de la operación; aquí no recibe argumentos. |
| `: string` | Exige un resultado de tipo cadena. |
| `;` | Cierra la declaración, que no tiene cuerpo. |

La firma expresa cómo se llama la operación: su nombre, parámetros y retorno. En nuestros ejemplos las clases repetirán esa misma firma.

No escribimos llaves con instrucciones en el método de la interfaz ni necesitamos añadirle `abstract`: su declaración ya expresa una operación pendiente de implementación.

No podemos hacer lo siguiente:

```php
// Incorrecto: una interfaz no se puede instanciar.
// $elemento = new Describible();
```

## 5.3. Implementar la interfaz con implements

**Archivo `Producto.php`**

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Describible.php';

class Producto implements Describible
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

`implements Describible` declara que `Producto` cumple esa interfaz. La clase aporta el cuerpo de `obtenerDescripcion()` utilizando su nombre.

La interfaz no exige precio, constructor ni `getPrecio()`. La clase puede añadir datos y métodos propios además de cumplir las operaciones del contrato.

!!! note "Ejemplo independiente"
    Este `Producto` es una clase concreta nueva para explicar interfaces. No sustituye al producto abstracto del apartado 4 ni debe cargarse en una misma petición con otras clases del mismo nombre. Ejecuta cada apartado desde su carpeta.

### ¿Basta con escribir el método sin implements?

Para ser reconocido como `Describible` en este ejemplo, el objeto debe pertenecer a una clase que implemente la interfaz, directamente o por herencia. Tener un método con el mismo nombre no establece por sí solo esa relación de tipo.

Si una clase concreta escribe `implements Describible` y no dispone de una implementación válida de `obtenerDescripcion()`, PHP produce un error. Añadir `implements` no genera automáticamente el cuerpo.

## 5.4. Otra clase con el mismo compromiso

**Archivo `Reserva.php`**

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Describible.php';

class Reserva implements Describible
{
    private string $cliente;
    private int $personas;

    public function __construct(string $cliente, int $personas)
    {
        $cliente = trim($cliente);

        if ($cliente === '') {
            throw new InvalidArgumentException('El cliente no puede estar vacío.');
        }
        if ($personas <= 0) {
            throw new InvalidArgumentException('Las personas deben ser positivas.');
        }

        $this->cliente = $cliente;
        $this->personas = $personas;
    }

    public function obtenerDescripcion(): string
    {
        return 'Reserva de ' . $this->cliente
            . ' para ' . $this->personas . ' personas';
    }
}
```

`Reserva` no hereda de `Producto` y `Producto` no hereda de `Reserva`. Ambas cumplen `Describible`, pero cada una utiliza sus datos y sus propias instrucciones.

| Aspecto | Producto | Reserva |
| --- | --- | --- |
| Datos propios del ejemplo | Nombre y precio. | Cliente y personas. |
| Interfaz implementada | `Describible` | `Describible` |
| Operación común | `obtenerDescripcion(): string` | `obtenerDescripcion(): string` |
| Resultado de ejemplo | Producto: Teclado | Reserva de Ana para 3 personas |

La interfaz expresa **qué pueden hacer**, sin obligarlas a compartir datos ni una implementación del constructor.

## 5.5. Polimorfismo mediante una interfaz

Antes de recorrer un array, veamos dos llamadas independientes. Crea `prueba-llamadas.php` en la misma carpeta:

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Producto.php';
require_once __DIR__ . '/Reserva.php';

function describir(Describible $elemento): string
{
    return $elemento->obtenerDescripcion();
}

$producto = new Producto('Teclado', 25.0);
$reserva = new Reserva('Ana', 3);

echo htmlspecialchars(describir($producto), ENT_QUOTES, 'UTF-8');
echo '<br>';
echo htmlspecialchars(describir($reserva), ENT_QUOTES, 'UTF-8');
```

La salida será:

```text
Producto: Teclado
Reserva de Ana para 3 personas
```

**La función acepta cualquier objeto que cumpla Describible y ejecuta la implementación correspondiente a su clase real.** El tipo de su parámetro es ahora una interfaz en lugar de una clase padre.

### Recorrido de la primera llamada

1. Creamos un objeto de clase `Producto` y le asignamos sus datos mediante el constructor.
2. `describir($producto)` acepta ese objeto porque su clase implementa `Describible`.
3. El parámetro `$elemento` permite acceder al mismo objeto. No lo convierte en otro objeto ni cambia su clase real.
4. `$elemento->obtenerDescripcion()` ejecuta el método definido en `Producto`.
5. En ese método, `$this` representa el producto y devuelve `Producto: Teclado`.
6. La función devuelve esa cadena y la página la muestra de forma segura.

Para la reserva se repite el recorrido, pero se ejecuta la implementación de `Reserva`. La interfaz no contiene un método con instrucciones que se ejecute primero.

| Llamada | Tipo del parámetro | Clase real | Implementación ejecutada |
| --- | --- | --- | --- |
| `describir($producto)` | `Describible` | `Producto` | La de `Producto`. |
| `describir($reserva)` | `Describible` | `Reserva` | La de `Reserva`. |

### ¿Qué operaciones garantiza el tipo de la interfaz?

Dentro de `describir()` solo necesitamos la descripción. No podemos suponer que todos los objetos recibidos ofrecen `getPrecio()`: `Reserva` no tiene ese método y la interfaz no lo exige.

La interfaz nos permite expresar la dependencia necesaria: «necesito algo que pueda describirse», sin exigir que sea un producto.

## 5.6. Ejemplo completo de página

Conserva los tres archivos de interfaz y clases y crea `index.php`:

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Producto.php';
require_once __DIR__ . '/Reserva.php';

function describir(Describible $elemento): string
{
    return $elemento->obtenerDescripcion();
}

$producto = new Producto('Teclado', 25.0);
$reserva = new Reserva('Ana', 3);
$elementos = [$producto, $reserva];
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Interfaces en PHP</title>
</head>
<body>
    <h1>Elementos que pueden describirse</h1>
    <ul>
        <?php foreach ($elementos as $elemento): ?>
            <li>
                <?= htmlspecialchars(describir($elemento), ENT_QUOTES, 'UTF-8') ?>
            </li>
        <?php endforeach; ?>
    </ul>
</body>
</html>
```

Abre `http://localhost:8080/unidad-03/ejemplos05/`, si mantienes el puerto 8080. Obtendrás una lista con las dos descripciones anteriores.

Se crean **dos objetos**. El array y el `foreach` no crean otros objetos; el recorrido solo aplica la misma función a cada uno. La selección de la implementación depende de su clase real.

`prueba-llamadas.php` e `index.php` son páginas alternativas que se ejecutan por separado: no incluyas una dentro de la otra, porque ambas definen la función `describir()`.

## 5.7. Interfaz o clase abstracta

Las dos permiten expresar un tipo común y utilizar polimorfismo. La diferencia principal está en si necesitamos **compartir una base con estado y comportamiento** o **exigir una capacidad común**.

| Pregunta | Clase abstracta | Interfaz |
| --- | --- | --- |
| ¿Se puede instanciar directamente? | No. | No. |
| ¿Cómo se utiliza desde una clase? | Con `extends`. | Con `implements`. |
| ¿Puede guardar estado de instancia con propiedades como nombre y precio? | Sí. | No proporciona almacenamiento de estado de instancia. |
| ¿Puede aportar métodos con cuerpo? | Sí. | No: declara las operaciones sin implementarlas. |
| ¿Puede exigir métodos pendientes? | Sí, mediante métodos abstractos. | Sí, mediante las declaraciones de sus métodos. |
| ¿Puede proporcionar un constructor con instrucciones comunes? | Sí. | No aporta una implementación de constructor. |
| ¿Cuántas puede utilizar directamente una clase? | Solo una clase padre. | Puede implementar varias interfaces. |
| ¿Permite clases concretas sin un padre común? | La relación se organiza mediante la clase base. | Sí, pueden cumplir la misma interfaz. |

En el apartado 4, `Producto` abstracto tenía sentido para **compartir nombre, precio, validación y cálculo del total**. Las hijas implementaban el envío.

Aquí, `Describible` tiene sentido porque **producto y reserva ofrecen una descripción**, sin que necesiten el mismo estado ni constructor.

!!! note "Alcance de esta introducción"
    Utilizaremos interfaces sencillas con métodos públicos de instancia. PHP permite también constantes en interfaces y, desde PHP 8.4, requisitos de acceso a propiedades. Estos últimos no proporcionan almacenamiento de estado y quedan fuera de este apartado. No necesitamos esas características para comprender y practicar el contrato de métodos.

## 5.8. Combinar herencia e interfaces

Una clase puede heredar de un padre e implementar una interfaz. No son decisiones excluyentes. El orden de las palabras es:

```php
class ClaseHija extends ClasePadre implements NombreInterfaz
{
    // Código de la clase.
}
```

Es un esquema de sintaxis, no un archivo completo para ejecutar. Si el padre ya aporta un método público compatible, la hija puede utilizarlo para cumplir el contrato; no necesita duplicarlo. Una implementación también puede quedar pendiente en una clase abstracta, pero una descendiente concreta debe satisfacerla.

Si una clase ya implementa una interfaz, sus hijas también cumplen ese tipo: no hace falta repetir `implements` en todas ellas.

Una clase puede implementar varias interfaces, separadas por comas:

```php
class Documento implements Describible, Exportable
{
    // Debe disponer de las operaciones exigidas por ambas interfaces.
}
```

Este segundo fragmento también muestra solo la sintaxis. No necesitamos programar una interfaz `Exportable` en nuestro ejemplo principal ni ampliar sus requisitos.

| Relación | Palabra | Idea |
| --- | --- | --- |
| Clase hija con clase padre | `extends` | Es una especialización y hereda miembros accesibles. |
| Clase con interfaz | `implements` | Cumple las operaciones exigidas. |
| Interfaz con otra interfaz | `extends` | Amplía un contrato existente. Lo reconoceremos, sin desarrollarlo aquí. |

En nuestras implementaciones mantendremos los mismos nombres de parámetros, tipos y retornos que la interfaz. Esto facilita la lectura y evita problemas con llamadas mediante argumentos con nombre.

## 5.9. Errores frecuentes y resumen

| Error | Qué debemos revisar |
| --- | --- |
| Crear `new Describible()`. | La interfaz no se instancia; creamos objetos de las clases concretas. |
| Escribir `class Producto extends Describible`. | Para cumplir una interfaz desde una clase usamos `implements`. |
| Escribir instrucciones dentro del método de la interfaz. | La declaración no tiene cuerpo y termina en `;`. |
| Declarar la implementación como privada. | El método exigido por la interfaz debe ser público. |
| Omitir el método en una clase concreta. | Debe disponer de una implementación compatible, propia o heredada. |
| Devolver un número desde el método que exige string. | Debemos cumplir el resultado declarado; en nuestros ejemplos devolvemos una cadena explícita. |
| Suponer que implementar copia código de la interfaz. | La interfaz declara el contrato; la clase aporta o hereda la implementación. |
| Suponer que el parámetro Describible cambia la clase del objeto. | El objeto conserva su clase real. |
| Llamar a un método exclusivo de Producto sobre cualquier Describible. | La interfaz solo garantiza las operaciones que exige. |
| Mezclar las distintas clases Producto de la unidad en una petición. | Ejecuta cada ejemplo desde su carpeta independiente. |

En una frase: **una interfaz indica qué operaciones debe ofrecer una clase; cada clase decide cómo implementarlas**.

El polimorfismo se apoya en ese acuerdo: **el código solicita una operación común y cada objeto ejecuta su propia implementación**.

## 5.10. Caso práctico final: empleados y clientes de una empresa

Una empresa necesita calcular las nóminas de sus empleados y consultar los correos de empleados y clientes para preparar avisos. Son dos necesidades distintas:

- Los empleados comparten nombre, correo, sueldo base y código de consulta. Cada tipo calcula su nómina de una forma: utilizaremos una **clase abstracta Empleado**.
- Empleados y clientes pueden proporcionar un correo, aunque un cliente no sea un empleado ni tenga nómina: utilizaremos una **interfaz Contactable**.

La aplicación de ejemplo solo consulta y muestra los correos; no envía mensajes. Simplificamos la nómina: el administrativo cobra su sueldo base y el comercial recibe además una comisión fija de 200 euros.

Crea la carpeta independiente `unidad-03/ejemplos05-personal` con estos seis archivos. No cargues en la misma petición otras versiones de clases de ejercicios anteriores.

### 1. Contactable.php: la capacidad común

```php
<?php
declare(strict_types=1);

interface Contactable
{
    public function getEmail(): string;
}
```

La interfaz exige poder consultar un correo. No guarda el correo ni escribe las instrucciones para obtenerlo.

### 2. Empleado.php: datos y código compartidos

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Contactable.php';

abstract class Empleado implements Contactable
{
    private string $nombre;
    private string $email;
    protected float $sueldoBase;

    public function __construct(
        string $nombre,
        string $email,
        float $sueldoBase
    ) {
        $nombre = trim($nombre);
        $email = trim($email);

        if ($nombre === '') {
            throw new InvalidArgumentException(
                'El nombre no puede estar vacío.'
            );
        }
        if (filter_var($email, FILTER_VALIDATE_EMAIL) === false) {
            throw new InvalidArgumentException(
                'El correo no es válido.'
            );
        }
        if ($sueldoBase < 0) {
            throw new InvalidArgumentException(
                'El sueldo base no puede ser negativo.'
            );
        }

        $this->nombre = $nombre;
        $this->email = $email;
        $this->sueldoBase = $sueldoBase;
    }

    public function getNombre(): string
    {
        return $this->nombre;
    }

    public function getEmail(): string
    {
        return $this->email;
    }

    abstract public function calcularNomina(): float;
}
```

Esta clase realiza dos tareas de diseño compatibles:

1. **Comparte una base para los empleados:** propiedades, constructor y getters.
2. **Cumple Contactable:** proporciona una implementación pública de `getEmail()`.

Deja pendiente el cálculo de nómina mediante un método abstracto. `protected` permite que las hijas consulten el sueldo base; nombre y correo permanecen privados y se consultan mediante sus getters.

`filter_var($email, FILTER_VALIDATE_EMAIL)` comprueba el formato del correo, no que la dirección exista. Escribimos `=== false` para detectar el rechazo de la validación.

### 3. Administrativo.php: primera especialización

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Empleado.php';

class Administrativo extends Empleado
{
    public function calcularNomina(): float
    {
        return $this->sueldoBase;
    }
}
```

Hereda el constructor y los getters. Solo necesita implementar el cálculo pendiente.

### 4. Comercial.php: otra forma de calcular

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Empleado.php';

class Comercial extends Empleado
{
    public function calcularNomina(): float
    {
        return $this->sueldoBase + 200.0;
    }
}
```

También hereda el constructor y los getters. En este ejemplo añade una comisión fija al sueldo base.

No repetimos `implements Contactable` en las hijas: **al heredar de Empleado, sus objetos también cumplen esa interfaz y heredan getEmail()**.

### 5. Cliente.php: mismo contrato sin ser empleado

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Contactable.php';

class Cliente implements Contactable
{
    private string $nombre;
    private string $email;

    public function __construct(string $nombre, string $email)
    {
        $nombre = trim($nombre);
        $email = trim($email);

        if ($nombre === '') {
            throw new InvalidArgumentException(
                'El nombre no puede estar vacío.'
            );
        }
        if (filter_var($email, FILTER_VALIDATE_EMAIL) === false) {
            throw new InvalidArgumentException(
                'El correo no es válido.'
            );
        }

        $this->nombre = $nombre;
        $this->email = $email;
    }

    public function getNombre(): string
    {
        return $this->nombre;
    }

    public function getEmail(): string
    {
        return $this->email;
    }
}
```

Un cliente no necesita sueldo ni cálculo de nómina. No lo hacemos heredar de `Empleado` únicamente para aprovechar el correo: cumple la interfaz por su cuenta.

En este ejemplo la validación del nombre y correo aparece en las dos clases. La interfaz **no evita esa repetición de instrucciones**: su finalidad es establecer el tipo y la operación común. Podríamos estudiar otras formas de reutilización en aplicaciones mayores, pero no las necesitamos para comprender este caso.

### 6. index.php: dos funciones con necesidades diferentes

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/Administrativo.php';
require_once __DIR__ . '/Comercial.php';
require_once __DIR__ . '/Cliente.php';

function consultarNomina(Empleado $empleado): float
{
    return $empleado->calcularNomina();
}

function consultarCorreo(Contactable $persona): string
{
    return $persona->getEmail();
}

$administrativa = new Administrativo(
    'Ana',
    'ana@example.com',
    1500.0
);

$comercial = new Comercial(
    'Luis',
    'luis@example.com',
    1500.0
);

$cliente = new Cliente(
    'Marta',
    'marta@example.com'
);

$empleados = [$administrativa, $comercial];
$contactos = [$administrativa, $comercial, $cliente];
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Clase abstracta e interfaz</title>
</head>
<body>
    <h1>Personal y contactos de la empresa</h1>

    <h2>Nóminas: necesitamos empleados</h2>

    <?php foreach ($empleados as $empleado): ?>
        <p>
            <?= htmlspecialchars(
                $empleado->getNombre(), ENT_QUOTES, 'UTF-8'
            ) ?>:
            <?= number_format(
                consultarNomina($empleado), 2, ',', '.'
            ) ?> €
        </p>
    <?php endforeach; ?>

    <h2>Correos: necesitamos objetos contactables</h2>

    <ul>
        <?php foreach ($contactos as $contacto): ?>
            <li>
                <?= htmlspecialchars(
                    consultarCorreo($contacto), ENT_QUOTES, 'UTF-8'
                ) ?>
            </li>
        <?php endforeach; ?>
    </ul>
</body>
</html>
```

Abre `http://localhost:8080/unidad-03/ejemplos05-personal/`, si utilizas el puerto 8080.

```text
Nóminas: necesitamos empleados
Ana: 1.500,00 €
Luis: 1.700,00 €

Correos: necesitamos objetos contactables
ana@example.com
luis@example.com
marta@example.com
```

### Por qué elegimos un tipo diferente en cada función

| Objeto | consultarNomina(Empleado …) | consultarCorreo(Contactable …) |
| --- | --- | --- |
| Administrativo | Sí: es un empleado. | Sí: cumple la interfaz a través de Empleado. |
| Comercial | Sí: es un empleado. | Sí: cumple la interfaz a través de Empleado. |
| Cliente | No: no es un empleado. | Sí: implementa la interfaz directamente. |

Pasar `$cliente` a `consultarNomina()` produciría un `TypeError`: no pertenece al tipo exigido. Pasarlo a `consultarCorreo()` es válido.

**Recorrido de la nómina de Luis:**

1. Creamos el objeto `Comercial`; el constructor heredado guarda nombre, correo y sueldo base.
2. `consultarNomina($comercial)` lo acepta como `Empleado`.
3. La llamada a `calcularNomina()` utiliza la implementación de `Comercial`.
4. `$this` representa a Luis; devuelve `1500 + 200 = 1700`.

**Recorrido del correo de Marta:**

1. Creamos un objeto `Cliente` con su propio constructor.
2. `consultarCorreo($cliente)` lo acepta porque su clase implementa `Contactable`.
3. Se ejecuta `getEmail()` de `Cliente` y devuelve `marta@example.com`.
4. No interviene la clase `Empleado`: no es su padre ni se necesita para esta llamada.

Con Ana o Luis, la función de correo ejecuta el getter implementado en `Empleado`, que sus clases heredan. La interfaz garantiza que todos ofrecen esa consulta, sin aportar ella misma instrucciones.

### La decisión de diseño en una tabla

| Necesidad | Herramienta elegida | Qué aporta |
| --- | --- | --- |
| Compartir datos y código entre administrativos y comerciales. | Clase abstracta Empleado. | Constructor, propiedades y getters comunes; exige calcular la nómina. |
| Consultar correos de empleados y clientes. | Interfaz Contactable. | Un contrato público común, sin obligarlos a compartir padre. |

**La clase abstracta comparte una base dentro de una familia de clases. La interfaz permite utilizar juntos objetos que ofrecen una capacidad común.** En este ejemplo utilizamos ambas porque las dos necesidades existen a la vez.

Para comprobar que lo has comprendido, pregúntate: si incorporamos un proveedor que también ofrece `getEmail()` e implementa `Contactable`, ¿hay que cambiar la función `consultarCorreo()`? ¿Tendría sentido hacerlo heredar de `Empleado`?

## Comprueba lo aprendido

Antes de continuar, deberías poder:

1. Explicar con tus palabras qué significa contrato en este contexto.
2. Declarar una interfaz con un método público sin cuerpo.
3. Implementarla en una clase concreta con `implements`.
4. Diferenciar la declaración de una operación de su implementación.
5. Seguir una llamada a una función que recibe una interfaz como tipo.
6. Explicar por qué dos clases sin padre común pueden utilizarse con esa función.
7. Comparar una interfaz con una clase abstracta y elegir según lo que quieras compartir o exigir.
8. Reconocer que una clase puede heredar de un padre y cumplir interfaces.

## Practica lo aprendido

Realiza los **ejercicios 22 a 24 del bloque 5: Interfaces en PHP** de la [relación de actividades](actividades.md#bloque-5-interfaces-en-php). Son tres ejercicios acotados: una interfaz con dos implementaciones, un caso que combina clase abstracta e interfaz y una comprobación de tipo examen.

Comenzaremos los ejercicios en clase y completarás en casa lo pendiente. Revisaremos las soluciones una a una y mantendremos las del profesor separadas de los enunciados.

En el siguiente apartado ordenaremos y completaremos el uso de **excepciones, pruebas y documentación**, apoyándonos en las validaciones y comprobaciones realizadas durante la unidad. Los espacios de nombres y la autocarga se retomarán en una guía complementaria y al trabajar con Composer y Laravel.

## Fuentes de consulta

- [Aitor Medrano y Luis Alemañ: PHP orientado a objetos](https://aitor-medrano.github.io/dwes2122/03phpoo.html), apartado de interfaces.
- Manual oficial de PHP: [interfaces de objetos](https://www.php.net/manual/es/language.oop5.interfaces.php) y [clases abstractas](https://www.php.net/manual/es/language.oop5.abstract.php).

Material elaborado por **Joaquín García** para DWES de 2.º DAW del **IES Portada Alta**. Adapta y amplía contenidos de Aitor Medrano y Luis Alemañ. Licencia **CC BY-NC-SA**.
