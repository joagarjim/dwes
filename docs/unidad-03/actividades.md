# Unidad 3. Actividades de programación orientada a objetos

Los ejercicios se trabajan en casa. En clase revisaremos soluciones, dudas y errores frecuentes. Este documento se ampliará con los bloques siguientes.

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

Material elaborado por **Joaquín García** para DWES de 2.º DAW del **IES Portada Alta**. Licencia **CC BY-NC-SA**.
