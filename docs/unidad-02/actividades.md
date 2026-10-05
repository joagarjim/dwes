# Actividades · Unidad 2. El lenguaje PHP

Los ejercicios están numerados de forma consecutiva a lo largo de toda la unidad. **Todos los ejercicios de cada bloque se entregan juntos en una única tarea de Moodle.** La entrega de ejercicios se registrará como «entregada» o «no entregada». Más adelante habrá prácticas independientes, planteadas tras trabajar varios apartados, que sí se corregirán.

## Bloque 1. Fundamentos de PHP

**Ejercicios 1 a 6 · Entrega del bloque 1 en Moodle.** Resuélvelos en tu entorno de PHP y Apache. Guarda los archivos en una carpeta `unidad-02` y nómbralos `ej01.php`, `ej02.php`, ..., `ej06.php`. Entrega los seis archivos juntos en la tarea de Moodle correspondiente al bloque 1.

### Ejercicio 1 (`ej01.php`). Tres formas de mostrar contenido

Crea `ej01.php` con una página HTML que muestre tres párrafos. Genera el primero mediante `echo`, el segundo mediante `print` y el tercero mediante `<?= ... ?>`. Incluye en PHP un comentario de una línea y otro de bloque. Consulta el código fuente recibido por el navegador: ¿aparecen las instrucciones PHP o solo el HTML generado?

### Ejercicio 2 (`ej02.php`). Variables y cálculos

Crea `ej02.php`. Asigna `166` a `$x` y `999` a `$y`. Muestra el valor de ambas variables y el resultado de sumarlas, restarlas, multiplicarlas y dividirlas. Identifica cada operación en la página. Utiliza paréntesis cuando combines cálculos y concatenación de cadenas.

### Ejercicio 3 (`ej03.php`). Tabla de datos personales

Crea `ej03.php`. Almacena en variables un nombre, dos apellidos, un correo electrónico, un año de nacimiento y un teléfono **ficticios**. Muestra los seis valores en una tabla HTML de dos columnas tituladas «Dato» y «Valor». Escribe la estructura de la tabla en HTML e inserta los valores mediante `<?= ... ?>`. Guarda el teléfono como una cadena de texto.

### Ejercicio 4 (`ej04.php`). Precio final de una compra

Crea `ej04.php`. Un artículo cuesta 25 € por unidad; el cliente compra 4 unidades y recibe un 10 % de descuento. Después se aplica el 21 % de IVA. Guarda precio, unidades y descuento en variables, y define el IVA como constante. Calcula y muestra, por separado, el subtotal, el importe descontado, la base tras el descuento, el importe del IVA y el total final. Utiliza variables intermedias para no repetir cálculos. **Comprobación:** el total es 108,90 €.

### Ejercicio 5 (`ej05.php`). Tipos y comparaciones

Crea `ej05.php` con `$numero = 10;` y `$texto = '10';`. Utiliza `var_dump()` para mostrar el tipo y el valor de cada variable. Antes de ejecutar el archivo, predice el resultado de comparar ambas variables con `==`, `===`, `!=` y `!==`. Muestra cada comparación, su resultado y una breve explicación de la diferencia entre comparar valores y comparar también tipos.

### Ejercicio 6 (`ej06.php`). Puntos y condiciones lógicas

Crea `ej06.php`. Una persona empieza con 40 puntos, gana 15 y gasta 8. Actualiza el saldo utilizando `+=` y `-=` y muestra el resultado. Después utiliza `var_dump()` para comprobar estas dos expresiones, identificándolas en la página:

- El saldo está entre 40 y 50, incluidos ambos extremos (`&&`).
- El saldo es menor que 30 o mayor que 45 (`||`).

No utilices `if` todavía. **Comprobación:** el saldo final es 47 puntos y ambas expresiones devuelven `true`.


## Bloque 2. Formularios con GET y POST

**Ejercicios 7 y 8 · Entrega del bloque 2 en Moodle.** Entrega juntos los cuatro archivos de estos dos ejercicios en una única tarea de Moodle. Mantén los archivos de cada ejercicio juntos en `unidad-02`. Antes de programar, consulta el [apartado 2: Formularios con PHP](02-formularios-php.md).

### Ejercicio 7 (`ej07-formulario.html` y `ej07-resultado.php`). Datos personales con GET

Retoma los seis datos del ejercicio 3: nombre, dos apellidos, correo electrónico, año de nacimiento y teléfono. Esta vez introdúcelos mediante un formulario HTML que utilice `method="get"` y envíalos a un archivo PHP. Muestra los valores recibidos en una tabla HTML de dos columnas («Dato» y «Valor»).

- Asocia cada campo con su etiqueta y asigna un atributo `name` diferente a cada uno.
- Lee los valores con `$_GET`; utiliza `?? ''` para los datos que puedan faltar.
- Prepara **cada valor** con `htmlspecialchars()` antes de insertarlo en la tabla.
- Prueba el formulario y observa cómo aparecen los datos en la URL. Prueba también a abrir directamente el archivo de resultado sin parámetros.

Utiliza datos inventados para hacer las pruebas. En este ejercicio no necesitas validar su formato ni calcular la edad.

### Ejercicio 8 (`ej08-formulario.html` y `ej08-resultado.php`). Cuestionario con POST

Crea un pequeño cuestionario sobre hábitos de programación. El formulario debe incluir:

- Un campo de texto para el nombre o alias de quien responde.
- **Dos preguntas** de elección única, cada una con al menos tres opciones mediante botones de radio. Los botones de cada pregunta deben compartir su propio atributo `name`.
- Una **casilla de verificación** opcional para indicar si desea recibir más ejercicios.
- Un **área de texto** donde pueda explicar qué tema de PHP le interesa más.

Envía el formulario con `method="post"`. En `ej08-resultado.php`, recupera los datos mediante `$_POST` y presenta un resumen legible con el alias, las respuestas elegidas, el estado de la casilla y el comentario. Usa `??` para los campos ausentes y `htmlspecialchars()` al mostrar los textos. Una casilla sin marcar no se envía: compruébalo marcándola y desmarcándola.

**No hay que corregir ni puntuar el cuestionario todavía**: el objetivo es recibir distintos tipos de campos y mostrar su contenido. Compara la URL del resultado con la del ejercicio 7 y explica dónde viajan los valores en cada método. Utiliza datos ficticios en las pruebas.


## Bloque 3. Condiciones

**Ejercicios 9 a 13 · Entrega del bloque 3 en Moodle.** Realiza los cinco ejercicios después de estudiar el [apartado 3: Condiciones en PHP](03-condiciones.md). Entrega juntos los archivos de este bloque en una única tarea de Moodle.

### Ejercicio 9 (`ej09.php`). Positivo, negativo o cero

Crea un formulario con un campo numérico y un botón de envío. Haz que el formulario envíe el número mediante POST a la misma página, ej09.php. Recoge el valor enviado y utiliza if, elseif y else para determinar si es positivo, negativo o cero. Muestra en la página el número introducido y el resultado. Al abrir la página por primera vez, debe mostrarse solo el formulario.

### Ejercicio 10 (`ej10-formulario.html` y `ej10-resultado.php`). El mayor de tres números

Crea en ej10-formulario.html un formulario con tres campos numéricos que envíe los valores mediante POST a ej10-resultado.php.
En ej10-resultado.php, recoge los tres números y muestra cuál es el mayor sin utilizar max(). Resuélvelo primero con condiciones anidadas y después con operadores lógicos en las condiciones. Prueba al menos un caso con números iguales y explica qué muestra cada versión cuando hay empate.

### Ejercicio 11 (`ej11.php`). Etapas de la vida

Crea en ej11.php un formulario con un campo numérico para introducir una edad. Haz que envíe el dato mediante POST a la misma página.
Recoge la edad y utiliza una cadena de if, elseif y else para mostrar la etapa correspondiente: «bebé» si es menor que 3; «niño o niña» entre 3 y 12; «adolescente» entre 13 y 17; «adulto» entre 18 y 66; y «jubilado» a partir de 67. Si la edad es negativa, muestra un mensaje de error.
Muestra también la edad introducida y comprueba los límites: 2, 3, 12, 13, 17, 18, 66 y 67. Al abrir la página por primera vez, debe aparecer solo el formulario.

### Ejercicio 12 (`ej12.php`). Día de la semana con `switch` y `match`

Crea en ej12.php un formulario con un campo numérico para introducir un número del 1 al 7. Haz que envíe el dato mediante GET a la misma página.
Recoge el número y muestra el día de la semana correspondiente dos veces: primero usando switch y después usando match. Si el número no está entre 1 y 7, muestra «Día no válido» en ambas versiones.
Compara los resultados y explica por qué match no necesita break. Al abrir la página por primera vez, debe mostrarse solo el formulario.

### Ejercicio 13 (`ej13-formulario.html` y `ej13-resultado.php`). Comprobar un nombre enviado por POST

Crea un formulario POST con un campo para el nombre. En el archivo de resultado, recupera el valor con `$_POST['nombre'] ?? ''` y elimina los espacios de los extremos con `trim()`. Si queda vacío, muestra «Escribe un nombre antes de continuar»; en otro caso, saluda al usuario. Escapa el nombre antes de insertarlo en HTML. Prueba con un nombre normal, con espacios solamente y con texto que incluya etiquetas HTML.

No te apoyes únicamente en `required`: comprueba la entrada en el servidor aunque el navegador permita enviar el formulario sin ese atributo.


## Bloque 4. Bucles

**Ejercicios 14 a 20 · Entrega del bloque 4 en Moodle.** Realiza los siete ejercicios después de estudiar el [apartado 4: Bucles en PHP](04-bucles.md). Entrega juntos los archivos de este bloque en una única tarea de Moodle. Guarda las soluciones en la carpeta `unidad-02`.

### Ejercicio 14 (`ej14.php`). Cuenta atrás con `while`

Crea una página que muestre los números del 10 al 0 mediante `while`, cada uno en un elemento de una lista HTML. Tras el cero, muestra «¡Despegue!». Indica en un comentario cuál es el valor inicial, qué condición controla el bucle y cómo cambia el contador. Comprueba que la lista contiene **11 números**.

### Ejercicio 15 (`ej15.php`). `while` frente a `do...while`

Asigna `8` a una variable `$numero`. Crea un primer bucle `while` que muestre el número mientras sea menor o igual que `5`. Reinicia `$numero` a `8` y repite la tarea con `do...while`. Presenta los resultados en dos secciones y explica por qué uno no muestra números y el otro muestra el `8` una vez. Después cambia el valor inicial a `3` y compara de nuevo ambos resultados.

### Ejercicio 16 (`ej16.php`). Tabla de multiplicar con `for`

Crea en `ej16.php` un formulario que envíe mediante `GET` un número entero a la **misma página**. Si no se ha enviado todavía, muestra solo el formulario. Si se recibe un entero del 1 al 10, genera con `for` una tabla HTML con las multiplicaciones de ese número por los valores del 1 al 10. Muestra el número elegido en el título del resultado. Si el dato falta, no es un entero o queda fuera del intervalo, no generes la tabla y muestra un mensaje de error después de un envío. Comprueba en el servidor el valor recibido, aunque el formulario tenga límites HTML.

### Ejercicio 17 (`ej17.php`). Cuadrícula de productos

Genera una tabla HTML de multiplicar del 1 al 5 mediante **dos bucles `for` anidados**. El bucle exterior debe generar las filas y el interior las celdas de cada fila. Añade encabezados del 1 al 5 para filas y columnas, y muestra en cada celda el producto correspondiente. Comprueba, por ejemplo, que la intersección de la fila 4 y la columna 5 contiene `20`. No escribas a mano las 25 celdas de resultados.

### Ejercicio 18 (`ej18.php`). Saltar y detener el recorrido

Recorre con `for` los números del 1 al 20. Utiliza `continue` para omitir los múltiplos de 3 y `break` para detener el bucle al llegar a 12. Muestra los números que sí se procesan en una lista HTML. Antes de ejecutar el programa, predice qué valores aparecerán; después, compara tu predicción con el resultado. Explica por qué no se muestran ni el 3 ni el 12.

### Ejercicio 19 (`ej19.php`). Suma acumulada

Crea en `ej19.php` un formulario que envíe mediante `GET` dos números enteros, `inicio` y `fin`, a la **misma página**. Comprueba en PHP que ambos son enteros entre 1 y 100 y que `inicio` es menor o igual que `fin`.

Utiliza un bucle `for` y una variable acumuladora para sumar **todos los enteros entre ambos extremos, incluidos estos**. Muestra el intervalo y la suma obtenida. Si los datos no son válidos, muestra un mensaje y no calcules la suma. Al abrir la página por primera vez, muestra solo el formulario. **Comprobaciones:** del 1 al 10, la suma es `55`; del 4 al 4, la suma es `4`.

### Ejercicio 20 (`ej20.php`). Potencia mediante productos

Crea en `ej20.php` un formulario que envíe una base y un exponente mediante `GET` a la **misma página**. Comprueba en PHP que la base sea un número entero (puede ser negativo) y que el exponente sea un entero **no negativo**. Si los datos no son válidos, muestra un mensaje y no calcules la potencia. Antes del primer envío, muestra solo el formulario.

Calcula la potencia multiplicando la base por sí misma tantas veces como indique el exponente, mediante un bucle `for`. No utilices `**` ni `pow()` para realizar el cálculo. Muestra la base, el exponente y el resultado. Comprueba `2³ = 8`, `5¹ = 5`, `7⁰ = 1` y `(-2)³ = -8`; explica por qué la variable acumuladora debe comenzar en `1`.


## Bloque 5. Arrays

**Ejercicios 21 a 28 · Entrega del bloque 5 en Moodle.** Realiza los ocho ejercicios después de estudiar el [apartado 5: Arrays en PHP](05-arrays.md). Entrega juntos los archivos de este bloque en una única tarea de Moodle. Guarda las soluciones en la carpeta `unidad-02`.

### Ejercicio 21 (`ej21.php`). Lista de lenguajes

Crea un array indexado con cinco lenguajes de programación. Muestra el primero, el último y la cantidad de elementos. Cambia uno de los valores y añade otro al final con `[]`. Genera una lista HTML con todos los valores mediante `foreach`. Después muestra también sus índices y valores con la forma `$indice => $lenguaje`. Comprueba cómo cambian la cantidad y el último índice tras añadir el nuevo elemento.

### Ejercicio 22 (`ej22.php`). Números aleatorios y estadísticas

Rellena un array con **20 enteros aleatorios del 0 al 99** mediante un bucle `for` y `rand(0, 99)`. Muestra sus valores en una lista HTML. Recorre después el array con `foreach` para calcular el menor, el mayor y la media. Haz los cálculos con variables y comparaciones, sin usar `min()`, `max()` ni funciones de suma de arrays. Comprueba que la media queda entre el menor y el mayor.

### Ejercicio 23 (`ej23.php`). Bola de respuestas

Crea en `ej23.php` un formulario que envíe mediante `POST` una pregunta a la **misma página**. Guarda al menos ocho respuestas breves en un array indexado. Si la pregunta no está vacía después de aplicar `trim()`, elige una respuesta al azar mediante `rand(0, count($respuestas) - 1)` y muéstrala junto con la pregunta. Escapa la pregunta antes de insertarla en HTML. Si llega vacía o solo contiene espacios, muestra un mensaje de error y no elijas respuesta. Antes del primer envío, muestra solo el formulario.

### Ejercicio 24 (`ej24.php`). Contar categorías

Genera un array con **50 valores aleatorios** `'A'` o `'B'`. Puedes elegir cada valor usando `rand(0, 1)` y una condición. Crea otro array asociativo con las claves `'A'` y `'B'`, ambas iniciadas a cero. Recorre el primer array y actualiza el contador correspondiente en el segundo. Muestra cuántas veces ha aparecido cada valor y comprueba que la suma de ambos contadores es `50`. No utilices dos variables independientes para contar las categorías.

### Ejercicio 25 (`ej25.php`). Alturas por nombre

Guarda en un array asociativo los nombres ficticios y las alturas en centímetros de cinco personas: cada nombre será una clave y su altura, el valor. Recorre el array con `foreach ($alturas as $nombre => $altura)` y presenta los datos en una tabla HTML. Añade una última fila con la altura media, calculada durante el recorrido. Escapa los nombres al mostrarlos y utiliza `count()` para obtener el número de personas.

### Ejercicio 26 (`ej26.php`). Personas en una tabla

Crea un array con al menos cinco personas ficticias. Cada persona será un array asociativo con las claves `nombre`, `altura` (en centímetros) y `email`. Genera mediante `foreach` una tabla HTML con una fila por persona y una columna para cada dato. Escapa los textos al insertarlos en HTML. Después, mediante otro recorrido, muestra el nombre y la altura de la persona más alta. No escribas a mano las filas de datos.

### Ejercicio 27 (`ej27.php`). Operaciones con una lista

Parte del array `$frutas = ['pera', 'manzana', 'naranja'];` y realiza estas operaciones **en orden**, mostrando después de cada paso el estado del array con `print_r()` dentro de `<pre>`:

1. Muestra su cantidad con `count()`, recórrelo mediante `for` usando sus índices consecutivos y examina los tipos con `var_dump()`.
2. Añade `'plátano'` con `[]` y `'kiwi'` con `array_push()`. Extrae el último valor con `array_pop()` y muestra cuál se ha extraído.
3. Comprueba con `in_array(..., true)` si `'pera'` sigue presente.
4. Copia el array en `$copia`, ordena la copia con `sort()` y muestra ambos arrays para comprobar cuál ha cambiado.
5. Elimina con `unset()` el elemento de índice `1` del array original. Muestra sus claves con `array_keys()` y su cantidad con `count()`. Recorre los elementos restantes con `foreach` y explica por qué un `for` desde `0` hasta `count($frutas) - 1` ya no sería seguro. Obtén una lista renumerada con `array_values()`.

Utiliza `print_r()` y `var_dump()` para **comprobar** los cambios; el recorrido final con `foreach` debe presentar los valores en una lista HTML.

### Ejercicio 28 (`ej28.php`). Menús y claves asociativas

Crea un array `$menus` con **tres menús**. Cada menú será un array asociativo con las claves `'primero'`, `'segundo'` y `'postre'`, cuyos valores serán nombres de platos ficticios. Recorre `$menus` con un `foreach` exterior y, dentro, recorre cada menú con otro `foreach ($menu as $tipo => $plato)`. Muestra cada menú en una sección HTML con los tipos y platos correspondientes; escapa los textos antes de insertarlos.

Después, asigna `$menu = $menus[0]` para trabajar con el primero: muestra sus claves mediante `array_keys()`; comprueba con `isset()` si tiene `'postre'`; añade la clave `'bebida'` y elimina `'segundo'` mediante `unset()`. Añade también `'observaciones' => null` y compara el resultado de `isset($menu['observaciones'])` con `array_key_exists('observaciones', $menu)`. Explica la diferencia. Finalmente, copia el menú y ordena la copia por sus valores con `asort()`; comprueba que cada plato conserva su clave y que el menú original no ha cambiado.

## Bloque 6. Funciones

**Ejercicios 29 a 37 · Entrega del bloque 6 en Moodle.** Realiza los ejercicios después de estudiar el [apartado 6: Funciones en PHP](06-funciones.md). Entrega juntos los archivos en una única tarea de Moodle. Guarda las soluciones en la carpeta `unidad-02`.

### Ejercicio 29 (`ej29.php`). Tu primera función

Define `saludar(string $nombre): string`, que devuelva un saludo personalizado. Llámala con tres nombres distintos y muestra los resultados en una lista HTML. Explica en un comentario la diferencia entre definir la función y llamarla, e identifica un parámetro y un argumento. Escapa el texto al insertarlo en HTML.

### Ejercicio 30 (`ej30.php`). Precio con descuento

Define `aplicarDescuento(float $precio, float $porcentaje): float`. Devuelve el precio rebajado sin mostrar HTML dentro de la función. Llámala con varios precios y descuentos y presenta los resultados en una tabla. Comprueba `80 €` con `25 %` de descuento: el resultado es `60 €`. Añade un parámetro por defecto para el porcentaje (`10`) y prueba una llamada que lo omita.

### Ejercicio 31 (`ej31.php`). Estadísticas de un array

Escribe `calcularMedia(array $numeros): float` y `contarMayoresQue(array $numeros, int $limite): int`. Recorre los arrays mediante `foreach`, sin `array_sum()`. Utiliza ambas funciones con una lista de al menos cinco números y muestra los resultados. Asegúrate de enviar un array no vacío a `calcularMedia()` y explica qué problema aparecería si estuviera vacío.

### Ejercicio 32 (`ej32.php`). Validar y calcular

Crea un formulario `POST` con un precio y un porcentaje de descuento que se envíe a la misma página. Comprueba en el servidor que ambos son números, que el precio no es negativo y que el porcentaje está entre `0` y `100`. Define una función de cálculo con parámetros y resultado `float`, y llámala **solo con datos válidos**. Muestra el precio final; ante datos inválidos, muestra un error. Antes del primer envío, muestra solo el formulario.

### Ejercicio 33 (`ej33.php`). Ámbito y copia de arrays

Define `agregarPostre(array $menu, string $postre): array`, que devuelva un menú ampliado. Parte de un menú con `'primero'` y `'segundo'`; muestra el original y el devuelto para comprobar que el primero no ha cambiado. Dentro de la función utiliza una variable local e indica mediante un comentario por qué no puedes leerla desde fuera. Después crea otra función `agregarBebida(array &$menu, string $bebida): void` y comprueba que esta sí modifica el array que recibe. Presenta los resultados de forma legible.

### Ejercicio 34 (`funciones.php` y `ej34.php`). Reutilizar funciones

Traslada `aplicarDescuento()` y una función `formatearPrecio(float $precio): string` a `funciones.php`. Cárgalo desde `ej34.php` con `require_once __DIR__ . '/funciones.php'`. Muestra al menos tres productos con precio original y precio rebajado, calculados mediante las funciones. Comprueba que la página funciona sin copiar sus definiciones en `ej34.php` y observa qué sucede si cambias temporalmente el nombre del archivo requerido; restaura el nombre al terminar.

### Ejercicio 35 (`ej35.php`). Cantidad variable de números

Escribe `sumarVarios(int ...$numeros): int` sin utilizar `array_sum()`. Prueba las llamadas sin argumentos, con uno y con tres. Define además `multiplicar(int $a, int $b): int` y pásale los dos valores de un array indexado mediante `...$factores`. Explica con un comentario qué hace `...` en la definición y en la llamada. Llama a una función anterior usando un argumento con nombre para omitir uno opcional.

### Ejercicio 36 (`ej36.php`). Funciones como valores

Define una función con nombre que duplique un entero y llámala mediante una variable que contiene su nombre. Después define una función anónima que triplique un entero y guárdala en otra variable. Utiliza ambas con el mismo número y muestra los resultados. Crea además una función flecha que sume un incremento definido fuera y comprueba el resultado. Identifica qué variable se captura del ámbito exterior.

### Ejercicio 37 (`encabezado.php`, `pie.php` y `ej37.php`). Página con fragmentos compartidos

Crea una cabecera con el comienzo del documento HTML y un título variable, y un pie con el cierre del documento. Incluye ambos desde `ej37.php`, que contendrá un encabezado `<h1>` y un párrafo propios. Define `$titulo` antes de incluir la cabecera y escápalo al mostrarlo. Crea una segunda página con otro título y contenido que reutilice los mismos fragmentos. Explica por qué el archivo incluido puede leer `$titulo` y en qué se diferencia esto del ámbito local de una función.


## Bloque 7. Funciones predefinidas de PHP

**Ejercicios 38 a 45 · Entrega del bloque 7 en Moodle.** Realiza los ocho ejercicios después de estudiar el [apartado 7: Funciones predefinidas de PHP](07-funciones-predefinidas.md). Entrega juntos los archivos del bloque en una única tarea de Moodle. Guarda las soluciones en la carpeta `unidad-02`.

Utiliza las funciones indicadas en cada enunciado. Presenta los resultados en HTML de forma legible y escapa los textos al mostrarlos. Las funciones `mb_*` requieren la extensión `mbstring` en el entorno PHP.

### Ejercicio 38 (`ej38.php`). Preparar y analizar un nombre

Parte de la cadena `'   María Muñoz   '`. Muestra el original y el resultado de aplicar `trim()`, `ltrim()` y `rtrim()`. Utiliza `<pre>` o corchetes visibles para apreciar los espacios. Comprueba que el nombre original no cambia si guardas cada resultado en otra variable.

Con el nombre limpio, muestra:

1. Su longitud mediante `strlen()` y `mb_strlen(..., 'UTF-8')`. Explica por qué los resultados difieren y recuerda que el espacio entre nombre y apellido también cuenta.
2. Sus versiones en mayúsculas y minúsculas con `mb_strtoupper()` y `mb_strtolower()`.
3. Los dos primeros caracteres con `mb_substr()`.

Después aplica `strtoupper()` y `strtolower()` a `'Hola PHP'`, y `substr()` a `'ABC12345'` para obtener `'ABC'`, `'12345'` y `'45'`. Presenta cada operación junto a su resultado.

### Ejercicio 39 (`ej39.php`). Buscar sin perder la posición cero

Parte del texto `'PHP y Laravel'`. Busca `'PHP'`, `'Laravel'` y `'Python'` mediante `strpos()`. Para cada búsqueda, muestra la posición encontrada o «No encontrado», utilizando una comparación estricta con `false`. Añade una explicación de por qué la posición `0` no significa que la búsqueda haya fallado.

Después, con el nombre de archivo `'ejercicio.php'`, muestra el resultado de comprobar si contiene `'ejercicio'`, si empieza por `'ej'` y si termina en `'.php'` o `'.html'`. Utiliza `str_contains()`, `str_starts_with()` y `str_ends_with()` y presenta los booleanos como «Sí» o «No». Prueba también con `'Ejercicio.PHP'` y explica qué cambia al distinguir mayúsculas y minúsculas.

### Ejercicio 40 (`ej40.php`). Sustituir, separar y reunir

Parte de `'PHP, JavaScript, PHP, Python'`. Sustituye todas las apariciones de `'PHP'` por `'Laravel'` con `str_replace()` y recoge en su cuarto argumento la cantidad de sustituciones realizadas. Muestra el texto original, el nuevo y el número de cambios.

Divide el nuevo texto con `explode()` usando la coma como separador. Recorre el array, aplica `trim()` a cada elemento y guarda los valores limpios en otro array. Presenta los lenguajes en una lista HTML y vuelve a unirlos con `implode()`, separados por `' · '`. Comprueba que hay cuatro elementos y que se han realizado dos sustituciones. No elimines los valores repetidos.

### Ejercicio 41 (`ej41.php`). Mensaje con saltos de línea

Crea un formulario `POST` con un área de texto que se envíe a la misma página. Antes del primer envío, muestra solo el formulario. Comprueba que el dato recibido sea una cadena y que no esté vacío después de aplicar `trim()`; si no es válido, muestra un error.

Presenta el mensaje conservando los saltos de línea mediante `nl2br()`, después de escapar el texto con `htmlspecialchars()`. Prueba con dos líneas y con el texto `<strong>Hola</strong>`: las etiquetas deben verse literalmente, sin poner el texto en negrita. Explica en un comentario por qué primero escapas el texto y después añades los saltos HTML.

### Ejercicio 42 (`ej42.php`). Laboratorio de cálculos y redondeo

Presenta en una tabla HTML la operación y el resultado de cada una de estas pruebas:

1. `abs()` con `-8`, `8` y `0`.
2. `pow()` para calcular `2³` y `7⁰`, y `sqrt()` para obtener la raíz de `81`.
3. `round()` con `7.46` y `7.56` sin indicar decimales, y con `7.456` conservando dos decimales.
4. `floor()` y `ceil()` con `7.9` y `-7.1`. Explica por qué redondear hacia abajo un negativo no equivale a quitarle los decimales.

Finalmente, guarda `1234.567` en una variable y muestra su valor redondeado a dos decimales con `round()` y su versión para presentación con `number_format(..., 2, ',', '.')`. Utiliza `var_dump()` dentro de `<pre>` para comprobar que el primer resultado es numérico y el segundo es una cadena. Conserva la variable original para los cálculos.

### Ejercicio 43 (`ej43.php`). Tiradas de dados y estadísticas

Genera diez tiradas de un dado mediante `rand(1, 6)` y guárdalas en un array. Muéstralas en una lista HTML. Obtén la menor y la mayor mediante `min()` y `max()` y calcula la media con una suma acumulada mediante `foreach`. Presenta la media con dos decimales.

Comprueba que todas las tiradas están entre `1` y `6` y que la media queda entre el menor y el mayor. Recarga varias veces y observa que pueden repetirse los resultados. Añade un comentario que explique por qué ahora podemos utilizar `min()` y `max()`, aunque en el ejercicio 22 calculáramos los extremos mediante comparaciones.

### Ejercicio 44 (`ej44.php`). ¿Qué tipo tiene cada dato?

Crea una lista con estos valores: `20`, `'20'`, `20.5`, `'12.5'`, `'hola'`, `false`, `null` y un array con dos lenguajes. Recórrela y muestra para cada dato su tipo con `gettype()` y su contenido con `var_dump()` dentro de `<pre>`.

Comprueba también, mostrando «Sí» o «No», el resultado de `is_int()`, `is_float()`, `is_string()`, `is_bool()`, `is_array()`, `is_null()` e `is_numeric()` para cada valor.

Después compara `is_numeric()` y `ctype_digit()` con las cadenas `'20'`, `'0'`, `'-3'`, `'12.5'`, `''` y `'12,5'`. Presenta los resultados en una tabla. Explica por qué `'20'` es numérico pero no tiene tipo `int`, y por qué `'-3'` es numérico pero no está formado exclusivamente por dígitos.

### Ejercicio 45 (`ej45.php`). Comprobar antes de convertir

Crea un formulario `POST` con un campo de texto para una edad, enviado a la misma página. Antes del primer envío, muestra solo el formulario. Comprueba en el servidor que el dato recibido sea una cadena formada por dígitos mediante `ctype_digit()`, después de aplicar `trim()`. Comprueba que representa una edad entre `0` y `120`; si no es válido, muestra un mensaje y no lo uses como edad.

Si es válido, guárdalo como `int`, muestra la edad y comprueba con `var_dump()` que el dato recibido es una cadena y que la edad convertida es un entero. Prueba `'20'`, `'0'`, `'120'`, `'150'`, `'-3'`, `'12.5'`, `'hola'` y un envío vacío.

En una sección independiente, compara con `var_dump()` el resultado de convertir `'hola'` a `int`, `12.9` a `int`, `'12.5'` a `float` y `20` a `string`. Explica por qué convertir no sustituye a validar.

Consulta en el manual oficial la función `ctype_digit()` y añade al final de la página un enlace a su documentación y una frase que indique qué recibe y qué devuelve. Escapa los textos al insertarlos en HTML.

---

*Referencia didáctica: la secuencia parte de las actividades de [Aitor Medrano](https://aitor-medrano.github.io/dwes2122/02php.html) y se adapta al orden y los objetivos de esta unidad. Los ejercicios 4 a 6 amplían la práctica de operadores y tipos; el ejercicio 8 amplía el trabajo con formularios POST y distintos tipos de campos. Los ejercicios 9 a 11 retoman problemas de condiciones de Aitor, los ejercicios 12 y 13 incorporan `match` y la comprobación de formularios, los ejercicios 14 a 20 practican bucles y acumulación, los ejercicios 21 a 28 adaptan problemas de arrays, operaciones y generación de tablas; los ejercicios 29 a 37 practican funciones y reutilización de archivos; y los ejercicios 38 a 45 trabajan las funciones predefinidas explicadas en nuestros apuntes.*
