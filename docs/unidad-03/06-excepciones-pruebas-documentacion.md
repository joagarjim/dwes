# 6. Excepciones, pruebas y documentación

Durante la unidad ya hemos rechazado datos incorrectos mediante `throw`, capturado excepciones con `try/catch` y comprobado resultados. En este último apartado ordenaremos esas herramientas y aprenderemos a dejar constancia de cómo funcionan nuestras clases.

Trabajaremos tres preguntas: **¿cómo comunicamos que una operación no puede completarse?, ¿cómo comprobamos que funciona?, ¿cómo explicamos cómo utilizarla?**

## 6.1. Comunicar una operación inválida

Declarar tipos no resuelve todas las reglas del problema. Un número negativo es un `int` válido, pero no una cantidad válida de plazas para reservar.

Podemos decidir que un método devuelva `false` ante una operación rechazada o que lance una excepción. Lo importante es establecer un comportamiento claro y que el código que llama lo conozca.

| Decisión | Cómo informa el método | Qué hace quien lo llama |
| --- | --- | --- |
| Resultado booleano | Devuelve `true` o `false`. | Comprueba el resultado con un `if`. |
| Excepción | Ejecuta `throw` con un objeto de excepción. | Puede capturarla con `try/catch` o dejar que se propague. |

En los primeros ejercicios utilizábamos booleanos para algunas modificaciones y excepciones para datos iniciales inválidos. No es una regla obligatoria de PHP: es una decisión de diseño.

Un constructor no puede devolver `false` para comunicar que no ha podido construir el objeto. Una excepción permite interrumpir esa construcción. En nuestro ejemplo utilizaremos también excepciones en las operaciones de reserva, para mantener un criterio uniforme.

## 6.2. Lanzar y capturar una excepción

```php
throw new InvalidArgumentException('Las plazas deben ser positivas.');
```

- `new InvalidArgumentException(...)` crea un objeto que representa el problema.
- El mensaje explica la causa.
- `throw` interrumpe el recorrido normal: no se ejecutan las instrucciones posteriores de esa llamada.

`InvalidArgumentException` es una clase que PHP ya proporciona para argumentos no válidos. No hace falta crear una clase propia para utilizarla.

La captura se realiza así:

```php
try {
    // Operación que puede lanzar una excepción.
} catch (InvalidArgumentException $error) {
    echo htmlspecialchars($error->getMessage(), ENT_QUOTES, 'UTF-8');
}
```

`try` contiene la operación que queremos intentar. Si lanza una excepción compatible con el `catch`, `$error` recibe ese objeto y `getMessage()` consulta su mensaje. Después de la captura puede continuar el código situado tras el bloque; no se reanuda automáticamente la instrucción que falló.

La clase comunica el problema; la página decide cómo presentarlo. No utilizaremos `echo` dentro de nuestras clases para mostrar mensajes de validación.

## 6.3. Ejemplo completo: plazas de un taller

Crea la carpeta `unidad-03/ejemplos06`. Usaremos `Taller.php` e `index.php`.

### Archivo Taller.php

```php
<?php
declare(strict_types=1);

class Taller
{
    private int $plazasDisponibles;

    public function __construct(int $plazas)
    {
        if ($plazas <= 0) {
            throw new InvalidArgumentException(
                'El taller debe comenzar con al menos una plaza.'
            );
        }
        $this->plazasDisponibles = $plazas;
    }

    public function getPlazasDisponibles(): int
    {
        return $this->plazasDisponibles;
    }

    public function reservar(int $cantidad): void
    {
        if ($cantidad <= 0) {
            throw new InvalidArgumentException(
                'La cantidad debe ser positiva.'
            );
        }
        if ($cantidad > $this->plazasDisponibles) {
            throw new InvalidArgumentException(
                'No hay plazas suficientes para esa cantidad.'
            );
        }

        $this->plazasDisponibles -= $cantidad;
    }
}
```

Primero comprobamos las reglas y después modificamos el estado. Si se rechaza una reserva, las plazas permanecen intactas.

El método tiene retorno `void`: si termina normalmente, la reserva se ha realizado; si no puede realizarse, lanza una excepción. No devuelve un booleano, por lo que no escribimos `if ($taller->reservar(2))` ni intentamos mostrar su retorno.

### Archivo index.php

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Taller.php';

$taller = new Taller(5);

try {
    $taller->reservar(2);
    echo '<p>Reserva de dos plazas realizada.</p>';
} catch (InvalidArgumentException $error) {
    echo '<p>'
        . htmlspecialchars($error->getMessage(), ENT_QUOTES, 'UTF-8')
        . '</p>';
}

echo '<p>Plazas disponibles: ' . $taller->getPlazasDisponibles() . '</p>';

try {
    $taller->reservar(4);
    echo '<p>Reserva de cuatro plazas realizada.</p>';
} catch (InvalidArgumentException $error) {
    echo '<p>'
        . htmlspecialchars($error->getMessage(), ENT_QUOTES, 'UTF-8')
        . '</p>';
}

echo '<p>Plazas disponibles: ' . $taller->getPlazasDisponibles() . '</p>';
```

Abre `http://localhost:8080/unidad-03/ejemplos06/`, si mantienes el puerto 8080. Obtendrás:

```text
Reserva de dos plazas realizada.
Plazas disponibles: 3
No hay plazas suficientes para esa cantidad.
Plazas disponibles: 3
```

En la segunda reserva se lanza la excepción antes de restar plazas. El mensaje de éxito de cuatro plazas no se ejecuta. La captura informa del problema y la consulta posterior confirma que siguen disponibles tres plazas.

### Capturar una excepción del constructor

El constructor se ejecuta al crear el objeto mediante `new`. Por tanto, si queremos capturar una excepción que pueda lanzar, debemos poner **la creación del objeto dentro del try**, en la página que lo utiliza.

Crea `prueba-constructor.php` en la misma carpeta que `Taller.php`:

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Taller.php';

try {
    $taller = new Taller(0);

    echo '<p>Taller creado correctamente.</p>';
    echo '<p>Plazas: ' . $taller->getPlazasDisponibles() . '</p>';
} catch (InvalidArgumentException $error) {
    echo '<p>'
        . htmlspecialchars($error->getMessage(), ENT_QUOTES, 'UTF-8')
        . '</p>';
}

echo '<p>La comprobación ha terminado.</p>';
```

Abre `http://localhost:8080/unidad-03/ejemplos06/prueba-constructor.php`. Como pasamos cero, verás:

```text
El taller debe comenzar con al menos una plaza.
La comprobación ha terminado.
```

Sigue el recorrido:

1. `new Taller(0)` comienza la construcción y ejecuta `__construct(0)`.
2. El constructor detecta que las plazas no son positivas y ejecuta `throw`.
3. La construcción se interrumpe: la expresión `new` no entrega un objeto construido correctamente y la asignación a `$taller` no se completa.
4. Se salta al `catch`; no se ejecutan el mensaje de éxito ni la consulta de plazas que estaban después del `new` dentro del `try`.
5. La captura muestra el mensaje y después continúa la instrucción situada tras el bloque.

Si cambias `new Taller(0)` por `new Taller(5)`, el constructor termina correctamente: se muestra el mensaje de éxito, se consultan las cinco plazas y no se entra en el `catch`.

**El constructor detecta y lanza el problema; la página que intenta crear el objeto lo captura.** No necesitamos introducir un `try/catch` dentro del constructor para capturar la misma excepción que estamos lanzando.

| Dónde puede fallar | Qué ponemos dentro del try |
| --- | --- |
| Construcción del objeto | `$taller = new Taller(0);` |
| Operación sobre un objeto ya construido | `$taller->reservar(6);` |

!!! warning "No utilizar un objeto que no se ha creado"
    En esta página no existía antes la variable `$taller`. Si el constructor falla, no debemos escribir `$taller->getPlazasDisponibles()` después del `catch` suponiendo que hay un objeto válido. En el ejemplo, su utilización está dentro del `try`, después del `new`, y solo se alcanza si la construcción termina correctamente.

### Dónde poner las capturas

Si una excepción sale de un método sin capturarse allí, puede capturarse en el código que lo llamó. No es obligatorio que cada método tenga su propio `try/catch`.

En el ejemplo capturamos en la página porque es donde podemos informar al usuario. El constructor se ejecuta antes de los bloques y recibe un dato válido; si quisiéramos probar `new Taller(0)`, esa construcción tendría que estar dentro del `try` correspondiente.

Un único `try` con varias pruebas se interrumpe en el primer fallo. Para comprobar distintos rechazos de forma independiente, utiliza un bloque por caso.

### Un mismo try: al fallar una operación se saltan las siguientes

Guarda este ejemplo en `prueba-un-bloque.php`, junto a `Taller.php`:

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Taller.php';

try {
    $taller = new Taller(0); // Falla y salta al catch.
    $taller->reservar(2);    // No se ejecuta.
    echo '<p>Reserva realizada.</p>'; // Tampoco se ejecuta.
} catch (InvalidArgumentException $error) {
    echo '<p>'
        . htmlspecialchars($error->getMessage(), ENT_QUOTES, 'UTF-8')
        . '</p>';
}

echo '<p>El programa continúa después del bloque.</p>';
```

Salida:

```text
El taller debe comenzar con al menos una plaza.
El programa continúa después del bloque.
```

**No se vuelve a la instrucción siguiente al new que falló.** La captura informa del problema y el programa continúa después del bloque completo. Como el constructor ha fallado, no intentamos utilizar `$taller` fuera de él.

### Dos try/catch: comprobar dos errores independientes

Si queremos probar tanto un constructor inválido como una reserva excesiva, usamos bloques separados. Guarda este código en `prueba-dos-bloques.php`:

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Taller.php';

// Primera prueba: rechazar un taller de cero plazas.
try {
    $tallerInvalido = new Taller(0);
} catch (InvalidArgumentException $error) {
    echo '<p>Prueba 1: '
        . htmlspecialchars($error->getMessage(), ENT_QUOTES, 'UTF-8')
        . '</p>';
}

// Segunda prueba: crear un taller válido e intentar reservar de más.
try {
    $taller = new Taller(5);
    $taller->reservar(6);
} catch (InvalidArgumentException $error) {
    echo '<p>Prueba 2: '
        . htmlspecialchars($error->getMessage(), ENT_QUOTES, 'UTF-8')
        . '</p>';
}
```

Salida:

```text
Prueba 1: El taller debe comenzar con al menos una plaza.
Prueba 2: No hay plazas suficientes para esa cantidad.
```

La primera captura permite continuar hasta la segunda prueba. Esta utiliza **otro objeto creado con datos válidos**: no intenta utilizar el que no se pudo construir.

| Organización | Qué ocurre al lanzar una excepción |
| --- | --- |
| Varias operaciones dentro del mismo try | Se salta al catch y se omiten las operaciones restantes de ese try. |
| Pruebas en bloques try/catch separados | Tras gestionar el fallo de una prueba, puede ejecutarse la siguiente. |

**La clase lanza el problema; la página lo captura. Para comprobar errores independientes, cada prueba tiene su propio bloque.**


## 6.4. Otros fallos y capturas: qué reconocer

| Situación | En nuestros ejemplos |
| --- | --- |
| Argumento incompatible con el tipo declarado | Puede producir un `TypeError`. |
| Regla del negocio incumplida, como reservar cero plazas | La detectamos y lanzamos una excepción. |
| Error de programación, como llamar a un método inexistente | Hay que revisar y corregir el código. |

`Exception` es la clase base de las excepciones y permite capturar, por ejemplo, `InvalidArgumentException`. No incluye todos los errores del lenguaje: un `TypeError` pertenece a la familia `Error`.

La interfaz `Throwable` es el tipo común de `Exception` y `Error`. Podemos reconocer `catch (Throwable $error)`, pero en estas actividades capturaremos los fallos concretos que esperamos. Una captura general no sustituye a corregir un error de programación.

También existe `finally`, cuyo bloque se ejecuta al salir del `try/catch` en el flujo habitual, tanto si se ha producido una excepción como si no. Se utiliza, por ejemplo, para liberar recursos. Nuestro taller no necesita recursos que cerrar; lo utilizaremos a continuación solo para observar el orden de ejecución.

### Ejemplo 1. Cantidad inválida y tipo incorrecto

Guarda `prueba-tipos.php` junto a `Taller.php`. Mantén `declare(strict_types=1);` al principio del archivo que realiza las llamadas, para que la cadena del segundo caso no se convierta automáticamente a entero.

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Taller.php';

$taller = new Taller(5);

try {
    $taller->reservar(0);
} catch (InvalidArgumentException $error) {
    echo '<p>Cantidad inválida: '
        . htmlspecialchars($error->getMessage(), ENT_QUOTES, 'UTF-8')
        . '</p>';
}

try {
    $taller->reservar('2');
} catch (TypeError $error) {
    echo '<p>Tipo incorrecto: reservar() necesita un entero.</p>';
}
```

Salida:

```text
Cantidad inválida: La cantidad debe ser positiva.
Tipo incorrecto: reservar() necesita un entero.
```

| Caso | Quién detecta el fallo | Qué se lanza |
| --- | --- | --- |
| `reservar(0)` | Nuestro if: cero es entero, pero no una cantidad admitida. | Nosotros lanzamos `InvalidArgumentException`. |
| `reservar('2')`, con modo estricto | PHP, antes de ejecutar el cuerpo del método: recibe una cadena donde se exige int. | PHP lanza `TypeError`. |

Un `catch (Exception $error)` podría capturar el primer caso, porque `InvalidArgumentException` deriva de `Exception`. **No capturaría el TypeError**, que pertenece a la familia `Error`.

### Ejemplo 2. Throwable permite capturar ambas familias

Guarda este ejemplo independiente en `prueba-throwable.php`:

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Taller.php';

$taller = new Taller(5);

try {
    $taller->reservar(0);
} catch (Throwable $error) {
    echo '<p>Primera prueba: fallo capturado con Throwable.</p>';
}

try {
    $taller->reservar('2');
} catch (Throwable $error) {
    echo '<p>Segunda prueba: fallo capturado con Throwable.</p>';
}
```

Salida:

```text
Primera prueba: fallo capturado con Throwable.
Segunda prueba: fallo capturado con Throwable.
```

El primer objeto capturado es una `InvalidArgumentException`; el segundo es un `TypeError`. Ambos cumplen `Throwable`, por lo que el tipo de la captura admite los dos.

Esto conecta con el apartado de interfaces: la captura puede aceptar objetos de distintas clases mediante un tipo común. **Throwable no es el nombre de una excepción concreta ni algo que tengamos que instanciar aquí.**

Para las actividades mantendremos capturas específicas cuando conozcamos el problema esperado: dan más precisión y evitan tratar cualquier fallo como si fuera una validación normal.

Una llamada a un método inexistente también puede producir un `Error` capturable mediante `Throwable`. Capturarlo no crea el método que falta: debemos corregir la llamada o implementar la operación correcta.

### Ejemplo 3. Finally con una operación correcta y otra rechazada

En este ejemplo usaremos `finally` solo para mostrar el orden de ejecución. En una aplicación real podría cerrar un archivo u otro recurso que debamos liberar.

Guarda `prueba-finally.php`:

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Taller.php';

$taller = new Taller(5);

try {
    $taller->reservar(2);
    echo '<p>Primera reserva realizada.</p>';
} catch (InvalidArgumentException $error) {
    echo '<p>Primera reserva rechazada.</p>';
} finally {
    echo '<p>Fin del primer intento.</p>';
}

try {
    $taller->reservar(10);
    echo '<p>Segunda reserva realizada.</p>';
} catch (InvalidArgumentException $error) {
    echo '<p>Segunda reserva rechazada.</p>';
} finally {
    echo '<p>Fin del segundo intento.</p>';
}

echo '<p>Plazas restantes: ' . $taller->getPlazasDisponibles() . '</p>';
```

Salida:

```text
Primera reserva realizada.
Fin del primer intento.
Segunda reserva rechazada.
Fin del segundo intento.
Plazas restantes: 3
```

| Bloque | En el intento correcto | En el intento rechazado |
| --- | --- | --- |
| `try` | Ejecuta la reserva y su mensaje de éxito. | Se interrumpe al lanzarse la excepción. |
| `catch` | No se ejecuta. | Gestiona la excepción. |
| `finally` | Se ejecuta después del try. | Se ejecuta después del catch. |

**Try intenta la operación, catch gestiona un fallo compatible y finally ejecuta el cierre común de ambos recorridos.** En nuestro ejemplo finally no deshace reservas ni modifica las plazas.

Estos son los recorridos normales del ejemplo. Si una excepción no encuentra un catch compatible, finally también se ejecuta antes de que la excepción siga propagándose. No utilizamos aquí `exit`, terminaciones del proceso ni otros casos especiales.

'''
## 6.5. Comprobar casos normales, límites y errores

Una prueba consiste en **preparar una situación, ejecutar una operación y comparar el resultado real con el esperado**. Que una página se abra no demuestra que todas sus reglas funcionen.

Para comprobar `Taller`, empieza cada fila con un objeto nuevo de cinco plazas. Así una prueba no depende de lo ocurrido en las anteriores:

| Caso | Operación | Resultado esperado | Plazas posteriores |
| --- | --- | --- | ---: |
| Normal | Reservar 2. | Se completa. | 3 |
| Límite válido | Reservar 5. | Se completa: se permite agotar las plazas. | 0 |
| Exceso | Reservar 6. | Excepción por falta de plazas. | 5 |
| Cero | Reservar 0. | Excepción por cantidad no positiva. | 5 |
| Negativo | Reservar -1. | Excepción por cantidad no positiva. | 5 |

Comprueba también el constructor por separado: `new Taller(1)` debe aceptarse y `new Taller(0)` debe rechazarse.

**Después de un rechazo, comprueba el estado**, no solo que aparezca el mensaje. Una implementación que resta antes de validar podría informar del error y dejar el objeto incorrecto.

### Comparación sencilla desde PHP

Crea `pruebas.php` en la misma carpeta:

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Taller.php';

$taller = new Taller(5);
$taller->reservar(2);

if ($taller->getPlazasDisponibles() === 3) {
    echo '<p>Correcto: reservar dos deja tres plazas.</p>';
} else {
    echo '<p>FALLO: el número de plazas no coincide.</p>';
}

$taller = new Taller(5);

try {
    $taller->reservar(6);
    echo '<p>FALLO: se ha aceptado una reserva excesiva.</p>';
} catch (InvalidArgumentException $error) {
    if ($taller->getPlazasDisponibles() === 5) {
        echo '<p>Correcto: se rechaza el exceso y se conserva el estado.</p>';
    } else {
        echo '<p>FALLO: se ha rechazado, pero las plazas han cambiado.</p>';
    }
}
```

La primera comprobación debe mostrar el mensaje de resultado correcto. La segunda solo se considera correcta si se produce el rechazo **y** se conservan las cinco plazas. Los demás casos de la tabla se comprueban siguiendo el mismo procedimiento.

Esto es una comprobación manual programada sencilla. Más adelante podremos automatizar y organizar pruebas con herramientas específicas; no necesitamos instalar ninguna para este apartado.

## 6.6. Documentar lo que necesita saber quien utiliza la clase

Los nombres y tipos explican parte del funcionamiento, pero no todas sus reglas. `reservar(int $cantidad): void` no indica por sí solo que se rechazan el cero y las cantidades superiores a las plazas disponibles.

Podemos añadir este comentario **encima del método existente** en `Taller.php`, sin duplicarlo:

```php
/**
 * Reserva una cantidad positiva de plazas.
 *
 * Si la operación se rechaza, las plazas permanecen sin cambios.
 *
 * @param int $cantidad Número de plazas que se quieren reservar.
 * @throws InvalidArgumentException Si la cantidad no es positiva
 *         o supera las plazas disponibles.
 * @return void
 */
public function reservar(int $cantidad): void
{
    // Aquí se conserva el cuerpo del método ya programado.
}
```

Este fragmento muestra dónde colocar la documentación; **no sustituyas el cuerpo real por ese comentario**. Los bloques que comienzan por `/**` se utilizan como documentación PHPDoc y pueden ayudar al editor a ofrecer información.

| Elemento | Qué explica |
| --- | --- |
| Descripción inicial | Propósito del método. |
| `@param` | Significado del argumento, más allá de su tipo. |
| `@throws` | Excepciones que puede comunicar y sus causas. |
| `@return` | Resultado o ausencia de valor de retorno. |

Estos comentarios no ejecutan validaciones ni sustituyen a los tipos y comprobaciones del código. No es necesario comentar cada asignación; interesa explicar reglas, decisiones y resultados.

En un archivo `README.md` del ejercicio o proyecto basta con indicar qué hace, cómo ejecutarlo, los archivos principales y una pequeña tabla de pruebas con resultado esperado y observado.

## 6.7. Resumen y errores frecuentes

| Error | Qué debemos hacer |
| --- | --- |
| Suponer que el tipo int impide cantidades negativas. | Programar la regla de validez. |
| Modificar el objeto antes de validar. | Comprobar primero y modificar después. |
| Mostrar un mensaje de éxito antes de completar la operación. | Mostrarlo tras la llamada correcta. |
| Dejar fuera del try la operación inválida que queremos capturar. | Incluir esa operación dentro del bloque. |
| Probar solo un caso que funciona. | Incluir límites, rechazos y estado posterior. |
| Reutilizar el objeto sin tener en cuenta pruebas anteriores. | Preparar un estado conocido en cada prueba o documentar una secuencia. |
| Usar comentarios como sustituto de las comprobaciones. | Las reglas deben existir en el código. |

## Comprueba lo aprendido

Deberías poder explicar qué instrucciones dejan de ejecutarse al lanzar una excepción, dónde continúa el programa después de capturarla, por qué el taller conserva sus plazas ante una reserva rechazada, cómo elegir casos de prueba y qué reglas deben quedar documentadas.

## Practica lo aprendido

Aplica estos contenidos en la [actividad final de la unidad: Tienda informática, catálogo orientado a objetos](07-actividad-final.md). No habrá ejercicios adicionales de este apartado: validarás los datos, capturarás excepciones, comprobarás casos normales, límites y rechazos, y documentarás las operaciones principales. Los ejercicios por bloques terminan en el 24.

Con este apartado terminamos los contenidos principales de la unidad. La **actividad final** integrará las clases, encapsulación, relaciones entre objetos, herencia, interfaces y comprobaciones practicadas.

## Fuentes de consulta

- [Aitor Medrano y Luis Alemañ: PHP orientado a objetos](https://aitor-medrano.github.io/dwes2122/03phpoo.html), apartados sobre excepciones.
- Manual oficial de PHP: [excepciones](https://www.php.net/manual/es/language.exceptions.php), [InvalidArgumentException](https://www.php.net/manual/es/class.invalidargumentexception.php) y [Throwable](https://www.php.net/manual/es/class.throwable.php).

Material elaborado por **Joaquín García** para DWES de 2.º DAW del **IES Portada Alta**. Adapta y amplía contenidos de Aitor Medrano y Luis Alemañ. Licencia **CC BY-NC-SA**.
