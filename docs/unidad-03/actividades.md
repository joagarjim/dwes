# Unidad 3. Actividades de programación orientada a objetos

En clase revisaremos soluciones del bloque anterior, explicaremos los nuevos contenidos y comenzaremos sus ejercicios. Lo que no dé tiempo a terminar se completará en casa. Este documento se ampliará con los bloques siguientes.

## Indicaciones generales

- Dentro de `unidad_3`, crea una carpeta por ejercicio: `ej01`, `ej02`, etc. Así podrás reutilizar un nombre de clase sin mezclar versiones.
- Guarda cada clase en un archivo con su nombre y utiliza `index.php` como página de prueba. Cárgala mediante `require_once __DIR__ . '/NombreClase.php';`.
- Utiliza tipos en propiedades, parámetros y resultados, y escribe `declare(strict_types=1);` al principio de cada archivo PHP. La visibilidad y la inicialización se indicarán en cada bloque.
- Las clases no deben generar HTML ni imprimir resultados. La página de prueba mostrará los datos, escapará los textos con `htmlspecialchars()` y formateará los importes con `number_format()`.
- Comprueba los resultados y añade comentarios breves cuando ayuden a entender una prueba.
- Entrega un ZIP con las carpetas del bloque, según las indicaciones de Moodle.

## Bloque 1: Clases y objetos

Estos cinco ejercicios corresponden al [apartado 1](01-clases-objetos.md). Tiempo orientativo de trabajo en casa: **90–120 minutos**, según el repaso que necesites.

En este primer bloque utiliza propiedades públicas con valores iniciales. Todavía no necesitas constructores, getters ni setters.

### Ejercicio 1. Dos libros, una clase

Crea la clase `Libro` con estas propiedades públicas:

| Propiedad | Tipo | Valor inicial |
| --- | --- | --- |
| `titulo` | `string` | Cadena vacía |
| `autor` | `string` | Cadena vacía |
| `paginas` | `int` | `0` |

Añade el método `obtenerFicha(): string`, que devuelva un texto con el título, el autor y el número de páginas, sin etiquetas HTML.

En `index.php`, crea dos libros mediante dos instrucciones `new` y asigna datos diferentes. Muestra sus fichas en párrafos separados. Modifica las páginas del primero y muestra de nuevo ambos para comprobar que el segundo no cambia.

Al final del archivo, responde mediante comentarios: ¿cuál es la clase?, ¿cuántos objetos has creado?, ¿qué representa `$this` al llamar al método sobre cada libro?

### Ejercicio 2. Rectángulos que calculan

Crea una clase `Rectangulo` con propiedades públicas `base` y `altura`, ambas de tipo `float` e inicialmente `0.0`. Añade:

- `calcularArea(): float`: base por altura.
- `calcularPerimetro(): float`: dos veces la suma de base y altura.
- `esCuadrado(): bool`: indica si base y altura tienen el mismo valor.

Crea dos objetos: uno de base `5.0` y altura `3.0`, y otro de base `4.0` y altura `4.0`. Muestra sus resultados y un mensaje que indique si cada uno es cuadrado. Usa esos valores exactos en las pruebas; no es necesario introducir medidas mediante formulario.

Comprueba que obtienes áreas `15` y `16`, y perímetros `16` y `16`.

### Ejercicio 3. Un marcador que cambia

Crea la clase `Marcador` con una propiedad pública `puntos` de tipo `int`, inicialmente `0`, y los métodos:

- `anotarPunto(): void`: aumenta los puntos en uno.
- `reiniciar(): void`: deja los puntos a cero.
- `obtenerPuntos(): int`: devuelve los puntos actuales.

Crea dos marcadores. Anota tres puntos en el primero y uno en el segundo mediante llamadas a `anotarPunto()`. Muestra los resultados; reinicia el primero y vuelve a mostrar ambos. Debes obtener primero `3` y `1`, y después `0` y `1`.

Explica en un comentario por qué no utilizas `echo` con el resultado de `reiniciar()`.

### Ejercicio 4. ¿Dos variables o dos objetos?

Sin ejecutarlo primero, escribe en comentarios qué cuatro números mostrará este programa. Después ejecútalo y explica cualquier diferencia con tu predicción.

```php
<?php
declare(strict_types=1);

class Contador
{
    public int $valor = 0;

    public function incrementar(): void
    {
        $this->valor++;
    }
}

$a = new Contador();
$b = new Contador();
$c = $a;

$a->incrementar();
$c->incrementar();
$b->incrementar();

echo $a->valor . '<br>';
echo $b->valor . '<br>';
echo $c->valor . '<br>';

$c = new Contador();
echo $c->valor;
```

Indica cuántos objetos se crean antes de la última instrucción `new` y cuántos se han creado en total al terminar. Explica si cambiar la variable `$c` hace desaparecer el contador al que sigue accediendo `$a`.

En este ejercicio puedes mantener la clase y las pruebas en el mismo archivo, como en el enunciado.

### Ejercicio 5. Reparar y comprobar una clase

Este programa contiene **tres errores intencionados**, uno en cada línea marcada. Localízalos y corrígelos sin cambiar el resultado que debe calcular el programa:

```php
<?php
declare(strict_types=1);

class Entrada
{
    public string $evento = 'Concierto';
    public float $precio = 12.0;

    public function calcularTotal(int $cantidad): float
    {
        return $precio * $cantidad; // Error 1
    }
}

$entrada = Entrada(); // Error 2
echo $entrada['evento']; // Error 3
echo '<br>';
echo $entrada->calcularTotal(3);
```

Después de repararlo:

1. Separa la clase y la página de prueba en sus archivos correspondientes.
2. Muestra el evento y el importe de tres entradas: `36,00 €`.
3. Prueba también una entrada y cinco entradas. Debes obtener `12,00 €` y `60,00 €`.
4. Explica en comentarios la causa de cada error.
5. Indica si estos cálculos modifican el precio guardado en el objeto.

## Bloque 2: Constructor y encapsulación

Estos ejercicios corresponden al [apartado 2](02-constructor-encapsulacion.md). Tiempo orientativo de trabajo en casa: **2–3 horas**, según el repaso y las pruebas que necesites.

Crea las carpetas `unidad_3/ej06` a `unidad_3/ej10`. Guarda las propiedades como privadas y utiliza la forma explícita de declaración y asignación, salvo en la versión con promoción del ejercicio 9. No modifiques los archivos del bloque anterior: copia los que necesites en la nueva carpeta.

Las clases no deben imprimir mensajes. En las operaciones que devuelvan `bool`, muestra desde `index.php` si se han aceptado o rechazado. Si una operación se rechaza, el estado del objeto debe permanecer intacto.

Para las pruebas de datos iniciales inválidos utiliza `try/catch`, como en la teoría: captura `InvalidArgumentException` y muestra su mensaje mediante `getMessage()`, escapándolo para HTML. Realiza cada construcción inválida en un bloque independiente para comprobar todos los casos. Utiliza datos escritos en el código; no se requieren formularios.

### Ejercicio 6. Un libro desde su creación

Copia y adapta la clase `Libro` del ejercicio 1 en `ej06/Libro.php`:

1. Declara como privadas las propiedades `titulo`, `autor` y `paginas`, manteniendo sus tipos.
2. Crea un constructor que reciba título y autor obligatorios, y páginas opcionales con valor por defecto `0`. El cero representa un número de páginas todavía no indicado.
3. Elimina los espacios al principio y al final del título y del autor mediante `trim()`. Si alguno queda vacío o las páginas son negativas, lanza `InvalidArgumentException` con un mensaje que identifique el problema.
4. Añade `getTitulo(): string`, `getAutor(): string` y `getPaginas(): int`.
5. Conserva `obtenerFicha(): string`. No añadas setters: en este ejercicio los datos se establecen únicamente al construir el objeto.

En `index.php`, crea un libro con los tres argumentos y otro sin indicar páginas. Muestra sus fichas y utiliza también sus getters. Comprueba que el segundo tiene `0` páginas.

Prueba por separado un título formado solo por espacios, un autor vacío y unas páginas negativas. Explica en un comentario por qué ya no puedes escribir `$libro->paginas = 220;` desde la página. No dejes esa instrucción ejecutándose.

### Ejercicio 7. Una cuenta bancaria

Crea `CuentaBancaria.php` con propiedades privadas `titular` de tipo `string` y `saldo` de tipo `float`.

El constructor recibirá el titular y un saldo inicial opcional de `0.0`. Aplica `trim()` al titular. Rechaza mediante `InvalidArgumentException` un titular vacío o un saldo inicial negativo.

Implementa estos métodos:

| Método | Comportamiento |
| --- | --- |
| `getTitular(): string` | Devuelve el titular. |
| `getSaldo(): float` | Devuelve el saldo actual. |
| `ingresar(float $importe): bool` | Suma un importe estrictamente positivo; devuelve `false` si es cero o negativo. |
| `retirar(float $importe): bool` | Descuenta un importe positivo si no supera el saldo; en caso contrario devuelve `false`. |

No crees `setSaldo()`: los cambios deben realizarse mediante ingresos y retiradas.

En `index.php`, crea una cuenta con saldo `100.0` y realiza estas pruebas **en orden**, mostrando el resultado y el saldo después de cada operación:

| Operación | ¿Se acepta? | Saldo posterior |
| --- | --- | ---: |
| Ingresar `50.0` | Sí | 150,00 € |
| Retirar `30.0` | Sí | 120,00 € |
| Retirar `200.0` | No | 120,00 € |
| Ingresar `0.0` | No | 120,00 € |
| Retirar `-10.0` | No | 120,00 € |
| Retirar `120.0` | Sí | 0,00 € |

Crea otra cuenta sin indicar el saldo y comprueba que empieza en cero. Prueba también una construcción con saldo negativo y muestra el mensaje de la excepción. Para este ejercicio utilizamos importes sencillos; no se requiere implementar un sistema monetario de producción.

### Ejercicio 8. Un producto con precio controlado

Crea `Producto.php` con propiedades privadas `nombre` de tipo `string` y `precio` de tipo `float`.

El constructor recibirá ambos datos. Aplica `trim()` al nombre y rechaza mediante `InvalidArgumentException` el nombre vacío y el precio negativo. El precio cero sí está permitido.

Añade:

- `getNombre(): string` y `getPrecio(): float`.
- `setPrecio(float $nuevoPrecio): bool`: actualiza el precio y devuelve `true` si es cero o positivo; si es negativo, devuelve `false` y conserva el anterior.
- `calcularImporte(int $unidades): float`: devuelve el precio actual multiplicado por las unidades, sin modificar el objeto. Si las unidades no son positivas, lanza `InvalidArgumentException`.

En `index.php`, crea un teclado de `25.0` euros y comprueba esta secuencia:

1. Calcula tres unidades: debes obtener `75,00 €`.
2. Cambia el precio a `30.0`: se acepta. Tres unidades pasan a costar `90,00 €`.
3. Intenta asignar `-5.0`: se rechaza y el precio sigue siendo `30.0`.
4. Cambia el precio a `0.0`: se acepta y el importe de tres unidades es `0,00 €`.
5. Intenta calcular cero unidades y muestra el mensaje de la excepción.

Explica mediante un comentario por qué tener un setter no obliga a aceptar cualquier valor y por qué `setPrecio()` devuelve un booleano, mientras que `getPrecio()` devuelve un número.

### Ejercicio 9. Constructor explícito y promoción

Crea dos versiones equivalentes de una clase `Curso` con propiedades privadas `nombre` de tipo `string` y `horas` de tipo `int`. El constructor recibirá el nombre y unas horas opcionales con valor por defecto `20`. Añade `getNombre(): string` y `getHoras(): int`.

Organiza las versiones así:

| Carpeta | Archivos | Forma del constructor |
| --- | --- | --- |
| `ej09/explicito` | `Curso.php` e `index.php` | Propiedades declaradas antes del constructor y asignadas mediante `$this`. |
| `ej09/promocion` | `Curso.php` e `index.php` | Propiedades promocionadas en los parámetros del constructor. |

En cada página crea un curso llamado `PHP` de `30` horas y otro llamado `Laravel` sin indicar horas. Muestra sus datos y comprueba que las dos versiones producen los mismos resultados: `PHP, 30 horas` y `Laravel, 20 horas`.

En este ejercicio de comparación utiliza esos datos válidos y no añadas validación: el objetivo es identificar el código que abrevia la promoción.

Responde mediante comentarios:

1. ¿Qué diferencia hay entre `string $nombre` y `private string $nombre` en los parámetros del constructor?
2. ¿Qué declaración y asignación evita repetir la promoción?
3. ¿La promoción comprueba automáticamente que las horas sean positivas?

Ejecuta cada versión por separado desde su carpeta. No cargues ambas clases `Curso` en una misma petición.

### Ejercicio 10. Reparar una clase encapsulada

El siguiente código contiene **cuatro errores intencionados**, uno en cada línea marcada. Explícalos y corrígelos. La clase debe conservar sus propiedades privadas y el constructor debe inicializarlas a partir de los argumentos recibidos.

```php
<?php
declare(strict_types=1);

class Reserva
{
    private string $cliente;
    private int $personas;

    public function __construct(string $cliente, int $personas = 1): void // Error 1
    {
        $cliente = $cliente; // Error 2
        $this->personas = $personas;
    }

    public function getCliente(): string
    {
        return $this->cliente;
    }

    public function getPersonas(): int
    {
        return $this->personas;
    }

    public function setPersonas(int $personas): bool
    {
        if ($personas <= 0) {
            return false;
        }

        $personas = $personas; // Error 3
        return true;
    }
}

$reserva = new Reserva('Ana', 2);
echo $reserva->cliente; // Error 4
```

Después de corregir los cuatro errores:

1. Separa el código en `ej10/Reserva.php` y `ej10/index.php`.
2. Añade validación al constructor: aplica `trim()` al cliente y lanza `InvalidArgumentException` si queda vacío o si las personas no son positivas.
3. Crea la reserva de Ana para dos personas y muestra sus datos mediante getters.
4. Cambia a cuatro personas: debe devolver `true` y quedar en cuatro.
5. Intenta cambiar a cero personas: debe devolver `false` y mantenerse en cuatro.
6. Crea otra reserva omitiendo las personas y comprueba que se asigna una.
7. Prueba construir una reserva para cero personas y muestra el mensaje de la excepción.

Muestra el resultado de cada prueba. No cambies las propiedades a públicas para evitar los errores: el objetivo es utilizar correctamente la encapsulación.

## Bloque 3: Relaciones entre objetos, constantes y miembros estáticos

Estos cinco ejercicios corresponden al [apartado 3](03-composicion-elementos-clase.md). Tiempo orientativo de trabajo total: **2–3 horas y 15 minutos**, incluyendo lo que se avance en clase y las comprobaciones.

Crea las carpetas `unidad_3/ej11` a `unidad_3/ej15`. Si tu proyecto utiliza el nombre `unidad-03`, mantén ese nombre de carpeta. Guarda cada clase en su propio archivo y utiliza `index.php` para las pruebas, salvo la excepción indicada en el ejercicio 15. No se requieren formularios, herencia ni bases de datos.

Las propiedades serán privadas, salvo las constantes públicas que se indiquen. Los constructores comprobarán sus argumentos y lanzarán `InvalidArgumentException` si no son válidos. Muestra los resultados y los mensajes de error desde la página, no desde las clases.

### Ejercicio 11. Libros que comparten autor

Crea las clases `Autor` y `Libro`:

| Clase | Propiedades privadas | Constructor |
| --- | --- | --- |
| `Autor` | `nombre` de tipo `string`. | Recibe el nombre. Aplica `trim()` y rechaza un nombre vacío. |
| `Libro` | `titulo` de tipo `string`, `precio` de tipo `float` y `autor` de tipo `Autor`. | Recibe los tres datos. Limpia el título y rechaza el título vacío o el precio negativo. |

En `Autor`, implementa:

- `getNombre(): string`.
- `setNombre(string $nombre): bool`: aplica `trim()`; si queda vacío, devuelve `false` sin modificar la propiedad. Si es válido, actualiza el nombre y devuelve `true`.

En `Libro`, implementa `getTitulo(): string`, `getPrecio(): float` y `getAutor(): Autor`. No crees otro autor dentro de su constructor: guarda el objeto recibido.

En `index.php`:

1. Crea un único autor llamado `Ana López`.
2. Crea dos libros asociados a ese mismo objeto: `Aprender PHP`, de `20.0` euros, y `Practicar PHP`, de `15.0` euros.
3. Muestra el título, precio y nombre del autor de cada libro. Obtén el nombre mediante `getAutor()->getNombre()`.
4. Cambia el nombre del autor a `Ana López García` y vuelve a mostrar ambos libros. Los dos deben mostrar el nombre actualizado.
5. Intenta cambiar el nombre por una cadena de espacios. Comprueba que se rechaza y que ambos libros mantienen el nombre anterior.

Responde mediante comentarios: ¿cuántos objetos has creado?, ¿por qué cambia el autor que muestran los dos libros?, ¿devolver el autor mediante un getter crea una copia independiente?

### Ejercicio 12. Una biblioteca con un array de libros

Copia las clases del ejercicio 11 en `ej12`, sin modificar los archivos originales. Añade `Biblioteca.php` con una propiedad privada `libros` de tipo `array`, inicialmente vacía.

Implementa estos métodos:

| Método | Comportamiento |
| --- | --- |
| `anadirLibro(Libro $libro): void` | Añade el objeto recibido al array. |
| `getLibros(): array` | Devuelve el array para que la página pueda recorrerlo. |
| `contarLibros(): int` | Devuelve el número de elementos del array. |
| `calcularValorTotal(): float` | Recorre los libros y acumula sus precios. Si no hay libros, devuelve `0.0`. |

Para añadir un elemento al array de la propiedad puedes utilizar `$this->libros[] = $libro;`. La propiedad privada y el parámetro de tipo `Libro` permiten controlar qué objetos se incorporan mediante ese método.

En `index.php`:

1. Crea una biblioteca vacía. Comprueba que contiene cero libros y que su valor es `0,00 €`.
2. Crea dos autores y tres libros: `Aprender PHP`, de `20.0` euros; `Practicar PHP`, de `15.0` euros; y `Bases de datos`, de `30.0` euros. Los dos primeros comparten autor y el tercero tiene otro.
3. Añade los tres objetos a la biblioteca mediante `anadirLibro()`.
4. Recorre el resultado de `getLibros()` con `foreach` y muestra una tabla con título, autor y precio.
5. Muestra el número de libros y su valor conjunto: **3 libros y 65,00 €**.

En este ejercicio se añadirá cada libro una sola vez. No es necesario programar identificadores, búsqueda ni control de duplicados. Explica en un comentario por qué `anadirLibro()` no necesita crear otro objeto `Libro`.

### Ejercicio 13. Una constante y un método estático

Crea `Tarifa.php` con una constante pública `IVA` de valor `0.21`. Es un porcentaje fijo para este ejercicio.

Añade el método público estático `calcularConIva(float $base): float`, que:

- Rechace una base negativa mediante `InvalidArgumentException`.
- Devuelva la base multiplicada por `1 + self::IVA`.

La clase no necesita propiedades de instancia ni constructor: esta operación utiliza únicamente su argumento y la constante.

En `index.php`, sin crear ningún objeto `Tarifa`:

1. Consulta `Tarifa::IVA` y muestra el porcentaje como `21 %`.
2. Llama a `Tarifa::calcularConIva()` con estas bases y comprueba los resultados:

| Base | Importe con IVA |
| ---: | ---: |
| 100,00 € | 121,00 € |
| 50,00 € | 60,50 € |
| 0,00 € | 0,00 € |

3. Prueba una base negativa y muestra el mensaje de la excepción.

Responde mediante comentarios: ¿por qué no hace falta `new`?, ¿por qué no utilizamos `$this` en este método?, ¿por qué escribimos `self::IVA` sin `$`?, ¿puede reasignarse la constante durante la ejecución?

### Ejercicio 14. Contar reservas creadas

Crea `Reserva.php` con estos elementos:

- Propiedad privada de instancia `cliente`, de tipo `string`.
- Propiedad privada estática `totalCreadas`, de tipo `int`, inicialmente `0`.
- Constructor que reciba el cliente, aplique `trim()` y rechace el nombre vacío mediante `InvalidArgumentException`. Después de validar y guardar el nombre, incrementará el contador estático.
- Método de instancia `getCliente(): string`.
- Método público estático `getTotalCreadas(): int`.

Desde `index.php`, realiza las siguientes acciones **en este orden**. Antes de ejecutarlas, anota en comentarios el valor del contador que esperas después de cada paso:

1. Consulta el contador sin crear reservas.
2. Crea una reserva para `Ana` y consulta el contador.
3. Crea otra para `Luis` y vuelve a consultarlo.
4. Asigna la primera reserva a otra variable, sin utilizar `new`, y consulta el contador.
5. Intenta crear una reserva con el cliente formado por espacios; captura la excepción y vuelve a consultar el contador.
6. Crea una tercera reserva válida para `María` y consulta el contador.

Muestra también los clientes de las tres reservas válidas mediante sus getters. Comprueba que cada objeto conserva su cliente, mientras que las consultas estáticas acceden al mismo contador de clase.

Por último, recarga la página y responde mediante comentarios:

- ¿Se vuelve a ejecutar el programa desde el principio?
- ¿Se acumulan las reservas creadas en la petición anterior?
- ¿El contador representa reservas construidas correctamente en la ejecución o personas atendidas desde que existe la aplicación?

No añadas un método para reiniciar el contador ni un destructor: en este ejercicio contamos construcciones válidas durante una ejecución.

### Ejercicio 15. Tipo examen: completa el catálogo

**Tiempo orientativo: 15 minutos.** Resuelve primero sin ejecutar el código ni consultar los apuntes. Después comprueba la solución en el ordenador.

Completa los **seis huecos numerados** para que el programa cumpla estos requisitos:

- Cada artículo debe guardar el objeto `Categoria` recibido en su constructor.
- El precio debe seguir siendo un dato independiente de cada artículo.
- El contador debe ser un dato compartido de clase e incrementarse por cada artículo creado.
- El método de cálculo debe utilizar la constante de clase `IVA`.
- La página debe mostrar el nombre de la categoría y consultar el contador sin utilizar un artículo concreto.

Sustituye cada marcador `/* HUECO n */` por el fragmento necesario. No modifiques el resto del programa. Los datos proporcionados son válidos; no se pide añadir validaciones.

```php
<?php
declare(strict_types=1);

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

class Articulo
{
    public const IVA = 0.21;

    private /* HUECO 1 */ int $totalCreados = 0;
    private float $precio;
    private Categoria $categoria;

    public function __construct(float $precio, Categoria $categoria)
    {
        $this->precio = $precio;
        $this->categoria = /* HUECO 2 */;
        /* HUECO 3 */++;
    }

    public function getCategoria(): Categoria
    {
        return $this->categoria;
    }

    public function calcularPrecioConIva(): float
    {
        return $this->precio * (1 + /* HUECO 4 */);
    }

    public static function getTotalCreados(): int
    {
        return self::$totalCreados;
    }
}

$categoria = new Categoria('Periféricos');
$articulo1 = new Articulo(100.0, $categoria);
$articulo2 = new Articulo(50.0, $categoria);

$articulos = [$articulo1, $articulo2];

foreach ($articulos as $articulo) {
    echo htmlspecialchars(
        $articulo->getCategoria()->/* HUECO 5 */,
        ENT_QUOTES,
        'UTF-8'
    );
    echo ': ';
    echo number_format($articulo->calcularPrecioConIva(), 2, ',', '.');
    echo ' €<br>';
}

echo 'Artículos creados: ' . /* HUECO 6 */;
```

La salida debe ser:

```text
Periféricos: 121,00 €
Periféricos: 60,50 €
Artículos creados: 2
```

Escribe tus respuestas identificando los huecos del 1 al 6. Para comprobarlas, puedes guardar todo el programa completado en `ej15/index.php`, sin separar las clases en este ejercicio.

Responde también brevemente:

1. ¿Cuántos objetos se han creado en total, contando también la categoría?
2. Si al final añadimos `$otro = $articulo1;`, ¿aumenta el contador? Justifica la respuesta.

## Bloque 4: Herencia y polimorfismo

Estos **seis ejercicios, del 16 al 21**, corresponden al [apartado 4](04-herencia-polimorfismo.md). Están pensados para practicar una vez cada idea principal y reservar tiempo para la actividad final. Tiempo orientativo total: **2–3 horas**, según el apoyo que necesites.

Comienza en clase y termina en casa lo pendiente. Crea una carpeta por ejercicio dentro de `unidad-03`, guarda cada clase en su archivo y utiliza `index.php` para las pruebas. En los ejercicios 20 y 21 puedes mantener todo el código en un único archivo.

Utiliza tipos, `declare(strict_types=1);` y salida HTML segura, como en los bloques anteriores. No se requieren formularios, bases de datos ni interfaces. Las clases devuelven datos; la página los muestra. No añadas funcionalidades que no se soliciten.

### Ejercicio 16. Una persona que también es alumno

**Objetivo:** practicar herencia, constructor de la hija y sobrescritura en un único ejemplo pequeño.

Crea `Persona.php` con una propiedad privada `nombre` de tipo `string`. Su constructor recibe el nombre, aplica `trim()` y lanza `InvalidArgumentException` si queda vacío. Añade:

- `getNombre(): string`, que devuelve el nombre.
- `obtenerDescripcion(): string`, que devuelve `Persona: ` seguido del nombre.

Crea `Alumno.php`. La clase `Alumno` hereda de `Persona` y añade una propiedad privada `ciclo` de tipo `string`.

Su constructor recibe nombre y ciclo, limpia y valida el ciclo, llama al constructor del padre para inicializar el nombre y guarda el ciclo. No vuelvas a declarar la propiedad nombre en la hija.

Añade `getCiclo(): string` y sobrescribe `obtenerDescripcion(): string` para **reutilizar la descripción del padre mediante `parent::`** y añadir el ciclo.

En `index.php`, crea una persona llamada `Luis` y una alumna llamada `Ana` del ciclo `DAW`. Muestra las descripciones:

```text
Persona: Luis
Persona: Ana / Ciclo: DAW
```

Consulta también el nombre de Ana mediante el getter heredado. Prueba una construcción con ciclo vacío y captura su excepción.

Responde en comentarios: ¿por qué no accedemos directamente a `$this->nombre` desde la hija?, ¿qué inicializa `parent::__construct()`?, ¿cuántos objetos se crean al construir únicamente la alumna?

### Ejercicio 17. Tarifas de aparcamiento

**Objetivo:** comprender polimorfismo primero sin array y después con un recorrido.

Crea estas tres clases, cada una en su archivo:

| Clase | Requisitos |
| --- | --- |
| `Tarifa` | Clase concreta con el método público `calcularImporte(int $horas): float`. Rechaza horas menores o iguales a cero mediante `InvalidArgumentException`; en caso válido devuelve horas por `2.0`. |
| `TarifaCoche` | Hereda de `Tarifa` sin añadir código: utiliza la implementación heredada. |
| `TarifaMoto` | Hereda de `Tarifa` y sobrescribe `calcularImporte()` con la misma validación, pero cobrando `1.0` euro por hora. |

No necesitas propiedades ni constructores: en este ejercicio las horas se reciben en cada llamada y los métodos son **de instancia**, no estáticos.

En `index.php`, escribe una función `obtenerImporte(Tarifa $tarifa, int $horas): float` que devuelva el resultado de llamar a `calcularImporte()` sobre el objeto recibido. No debe comprobar la clase mediante `if`, `switch` ni `instanceof`.

1. Crea un objeto `TarifaCoche` y otro `TarifaMoto`.
2. Llama por separado a la función con cada objeto y tres horas. Debes obtener **6,00 € y 3,00 €**.
3. Guarda esos mismos objetos en un array y repite las llamadas mediante `foreach`. Debes obtener los mismos importes, sin crear nuevas tarifas en el recorrido.
4. Completa esta tabla, primero sobre papel y después como comentarios en el código:

| Llamada | Tipo declarado del parámetro | Clase real del objeto | Clase donde se ejecuta calcularImporte() | Resultado |
| --- | --- | --- | --- | --- |
| Coche, 3 horas | | | | |
| Moto, 3 horas | | | | |

Explica por qué el objeto conserva su clase real al pasarlo a un parámetro `Tarifa` y por qué el `foreach` no es quien decide la fórmula.

### Ejercicio 18. Figuras con una operación pendiente

**Objetivo:** utilizar una clase abstracta y ejecutar sus implementaciones mediante un tipo común.

Crea `Figura.php` con una clase abstracta `Figura` que declare el método público abstracto `calcularArea(): float`. No necesita propiedades ni constructor.

Crea dos clases hijas concretas:

| Clase | Propiedades privadas | Constructor y operación |
| --- | --- | --- |
| `Rectangulo` | `base` y `altura`, ambas `float`. | Recibe ambas medidas, exige que sean positivas y calcula base por altura. |
| `Circulo` | `radio`, de tipo `float`. | Recibe el radio, exige que sea positivo y calcula pi por radio al cuadrado. |

Utiliza `pi()` para obtener pi. Puedes multiplicar el radio por sí mismo. Las medidas inválidas deben provocar `InvalidArgumentException`. No necesitas getters ni un constructor vacío en `Figura`; por tanto, no hace falta llamar a `parent::__construct()` en este ejercicio.

En `index.php`, define `obtenerArea(Figura $figura): float`, que llame a `calcularArea()` del objeto recibido. Crea un rectángulo de base `5.0` y altura `3.0`, y un círculo de radio `2.0`.

Recórrelos mediante un array, muestra sus áreas con dos decimales y acumula los resultados numéricos antes de formatear:

| Resultado | Valor mostrado |
| --- | ---: |
| Área del rectángulo | 15,00 |
| Área del círculo | 12,57 |
| Suma de áreas | 27,57 |

Responde en comentarios: ¿por qué no podemos crear `new Figura()`?, ¿qué ocurriría si una hija concreta no implementase `calcularArea()`? No dejes esas instrucciones o declaraciones inválidas ejecutándose.

### Ejercicio 19. Catálogo de productos físicos y digitales

**Objetivo:** construir una jerarquía completa que reúna abstracción, constructor del padre, sobrescritura, polimorfismo y `final`. Utiliza el catálogo del apartado 4.9 como referencia, pero escribe y comprueba tus propios archivos.

Crea `Producto.php`, `ProductoFisico.php`, `ProductoDigital.php` e `index.php` dentro de `unidad-03/ej19`. No incorpores formularios, stock, categorías ni bases de datos: en este ejercicio nos centramos en la jerarquía.

#### A. Clase abstracta Producto

Declara propiedades privadas `nombre` de tipo `string` y `precio` de tipo `float`. Su constructor recibe ambos datos, aplica `trim()` al nombre y rechaza mediante `InvalidArgumentException` un nombre vacío o un precio negativo.

Implementa estas operaciones:

| Método | Requisito |
| --- | --- |
| `getNombre(): string` | Devuelve el nombre. |
| `getPrecio(): float` | Devuelve el precio. Debe ser público y **final**. |
| `obtenerDescripcion(): string` | Devuelve `Producto: ` seguido del nombre. |
| `calcularTotal(): float` | Devuelve el precio más el envío obtenido mediante `$this->calcularGastosEnvio()`. |
| `calcularGastosEnvio(): float` | Decláralo público y **abstracto**, sin cuerpo. |

#### B. Clase ProductoFisico

Hereda de `Producto` y añade una propiedad privada `peso` de tipo `float`.

- El constructor recibe nombre, precio y peso. Exige un peso positivo, llama al constructor del padre y guarda el peso.
- Añade `getPeso(): float`.
- Sobrescribe `obtenerDescripcion()` para reutilizar la descripción del padre y añadir ` / Peso: `, el peso y ` kg`.
- Implementa el envío: **2 euros por kilogramo**.

#### C. Clase ProductoDigital

Declárala **final** y haz que herede de `Producto`.

- No añadas constructor propio: utiliza el heredado.
- Sobrescribe la descripción para reutilizar la del padre y añadir ` / Descarga digital`.
- Implementa el envío devolviendo `0.0`.

#### D. Página de prueba y polimorfismo

Define una función `obtenerTotal(Producto $producto): float` que devuelva el resultado de `calcularTotal()` del objeto recibido. No utilices comprobaciones de clase para elegir fórmulas.

Crea estos tres productos y guárdalos en un array:

| Producto | Tipo | Precio | Peso |
| --- | --- | ---: | ---: |
| Teclado | Físico | 25,00 € | 0.5 kg |
| Curso PHP | Digital | 20,00 € | No corresponde. |
| Monitor | Físico | 180,00 € | 4 kg |

Recorre el array y muestra una tabla HTML sencilla con descripción, precio, envío y total. Acumula el resultado de `obtenerTotal()` para obtener el valor conjunto.

| Producto | Envío esperado | Total esperado |
| --- | ---: | ---: |
| Teclado | 1,00 € | 26,00 € |
| Curso PHP | 0,00 € | 20,00 € |
| Monitor | 8,00 € | 188,00 € |

La suma será **234,00 €**. El cálculo corresponde a una unidad de cada producto con su envío individual; no se calcula un paquete conjunto.

Prueba también un producto físico de peso cero y un producto digital de precio negativo. Captura cada excepción por separado y muestra su mensaje desde la página.

#### E. Explicar las decisiones

Responde mediante comentarios, sin dejar declaraciones inválidas ejecutándose:

1. ¿Por qué no puedes crear directamente un objeto `Producto`?
2. ¿Qué obliga a implementar el método abstracto?
3. ¿Qué impide `final` en `getPrecio()`? ¿Pueden las hijas utilizarlo?
4. ¿Qué impide `final` en `ProductoDigital`?
5. Cuando `calcularTotal()` se ejecuta sobre el teclado, ¿qué objeto representa `$this` y qué versión del cálculo de envío se utiliza?

Anota el recorrido completo del teclado desde `obtenerTotal()` hasta el resultado `26.0`. No basta con copiar el código: debes poder explicar qué método se ejecuta en cada paso.

### Ejercicio 20. Reparar una jerarquía pequeña

**Tiempo orientativo: 10–15 minutos.** El siguiente programa no funciona correctamente. Localiza y corrige sus errores manteniendo `Aviso` abstracta y el método `getTexto()` final. No cambies la instrucción que crea el aviso.

```php
<?php
declare(strict_types=1);

abstract class Aviso
{
    private string $texto;

    public function __construct(string $texto)
    {
        $this->texto = $texto;
    }

    final public function getTexto(): string
    {
        return $this->texto;
    }

    abstract public function obtenerMensaje(): string;
}

class AvisoUrgente extends Aviso
{
    public function __construct(string $texto)
    {
    }

    public function getTexto(): string
    {
        return 'Texto sustituido';
    }

}

$aviso = new AvisoUrgente('Revisar entrega');
echo htmlspecialchars($aviso->obtenerMensaje(), ENT_QUOTES, 'UTF-8');
```

El aviso urgente debe permitir consultar un mensaje formado por `URGENTE: ` seguido del texto guardado en el objeto. Tras corregir el programa debe mostrarse:

```text
URGENTE: Revisar entrega
```

Explica en comentarios cada corrección. Para este ejercicio puedes guardar todo el programa en `ej20/index.php`; los datos son válidos y no se pide añadir validaciones.

### Ejercicio 21. Tipo examen: completa y sigue la jerarquía

**Tiempo orientativo: 15–20 minutos.** Resuelve primero sin apuntes ni ordenador. Completa los **cinco huecos**, predice las dos líneas de salida y después comprueba el programa.

El ticket general cuesta `5.0` euros. Un ticket reducido cuesta la mitad. La hija debe utilizar el constructor del padre y su cálculo, sin repetir el precio base como un número literal.

```php
<?php
declare(strict_types=1);

class Ticket
{
    private string $nombre;
    protected float $precioBase = 5.0;

    public function __construct(string $nombre)
    {
        $this->nombre = $nombre;
    }

    final public function getNombre(): string
    {
        return $this->nombre;
    }

    public function calcularPrecio(): float
    {
        return $this->precioBase;
    }
}

final class TicketReducido /* HUECO 1 */ Ticket
{
    public function __construct(string $nombre)
    {
        /* HUECO 2 */($nombre);
    }

    public function calcularPrecio(): float
    {
        return /* HUECO 3 */() / 2;
    }
}

function mostrarPrecio(/* HUECO 4 */ $ticket): string
{
    return $ticket->getNombre() . ': '
        . number_format($ticket->calcularPrecio(), 2, ',', '.') . ' €';
}

$normal = new Ticket('General');
$reducido = new TicketReducido('Reducido');

echo htmlspecialchars(mostrarPrecio($normal), ENT_QUOTES, 'UTF-8');
echo '<br>';
echo htmlspecialchars(mostrarPrecio(/* HUECO 5 */), ENT_QUOTES, 'UTF-8');
```

No se requiere añadir validación: los argumentos proporcionados son válidos. El hueco 4 debe expresar el tipo común que admite los dos objetos.

Además de completar los huecos y escribir la salida, responde:

1. ¿Puede `TicketReducido` acceder directamente a `$this->precioBase`? ¿Y a `$this->nombre`? Justifica según su visibilidad.
2. ¿Se puede crear una clase hija de `TicketReducido`? ¿Por qué?
3. ¿Puede una hija de `Ticket` sobrescribir `getNombre()`? ¿Y utilizarlo sin sobrescribirlo?
4. Al llamar a `mostrarPrecio()` con el ticket reducido, ¿qué implementación de `calcularPrecio()` se selecciona? ¿Qué hace después la llamada al padre dentro de ese método?

Guarda la comprobación en `ej21/index.php`. No elimines los modificadores `final` ni cambies la visibilidad para resolver el ejercicio.

Material elaborado por **Joaquín García** para DWES de 2.º DAW del **IES Portada Alta**. Licencia **CC BY-NC-SA**.
