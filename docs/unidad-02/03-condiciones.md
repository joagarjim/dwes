# 3. Condiciones en PHP

Hasta ahora hemos calculado valores y recibido datos de formularios. Las **condiciones** permiten decidir qué instrucciones se ejecutan según esos valores. La idea ya la conoces de otros lenguajes; aquí practicaremos la sintaxis de PHP y su uso para generar respuestas HTML distintas.

En los ejemplos utilizaremos llaves `{}` incluso cuando el bloque contenga una sola instrucción. También preferiremos comparaciones estrictas (`===`, `!==`) cuando queramos distinguir valores de tipos diferentes.

## 3.1. Una decisión: `if`

`if` ejecuta un bloque cuando la condición es verdadera:

```php
<?php
$nota = 7;

if ($nota >= 5) {
    echo '<p>La asignatura está aprobada.</p>';
}
```

La expresión `$nota >= 5` produce un booleano. Si cambias `$nota` por `4`, el párrafo no se genera. Observa el código fuente de la respuesta: el navegador solo recibe el HTML que PHP haya generado.

Podemos combinar comparaciones con `&&`, `||` y `!`:

```php
<?php
$edad = 20;
$tieneEntrada = true;

if ($edad >= 18 && $tieneEntrada) {
    echo '<p>Puede acceder.</p>';
}
```

## 3.2. Dos o más caminos: `else` y `elseif`

Con `else` establecemos qué ocurre cuando la condición no se cumple:

```php
<?php
$nota = 4;

if ($nota >= 5) {
    echo '<p>Aprobado.</p>';
} else {
    echo '<p>Suspenso.</p>';
}
```

Si necesitamos varios casos, usamos `elseif`. Las condiciones se comprueban en orden y se ejecuta **el primer bloque** cuya condición resulte verdadera:

```php
<?php
$nota = 8;

if ($nota < 0 || $nota > 10) {
    $resultado = 'Nota no válida';
} elseif ($nota < 5) {
    $resultado = 'Suspenso';
} elseif ($nota < 7) {
    $resultado = 'Aprobado';
} elseif ($nota < 9) {
    $resultado = 'Notable';
} else {
    $resultado = 'Sobresaliente';
}

// Mostrar el resultado es una tarea separada de decidirlo.
echo '<p>' . $resultado . '</p>';
```

Cuando llegamos a `$nota < 7`, ya sabemos que la nota no es menor que 5. Por eso no hace falta repetir `$nota >= 5` en esa condición. Antes de programar una cadena de `elseif`, comprueba que los rangos no dejan huecos ni se solapan de manera inesperada.

## 3.3. Aplicar condiciones a un formulario

En el apartado anterior utilizamos `?? ''` para evitar un aviso si falta un parámetro. Ahora podemos decidir qué mostrar cuando el nombre llega vacío. Por ejemplo, después de recibir un formulario por POST:

```php
<?php
$nombre = trim($_POST['nombre'] ?? '');

if ($nombre === '') {
    $mensaje = 'Escribe un nombre antes de continuar.';
} else {
    $nombreHtml = htmlspecialchars($nombre, ENT_QUOTES, 'UTF-8');
    $mensaje = 'Hola, ' . $nombreHtml . '.';
}
?>
<p><?= $mensaje ?></p>
```

`trim()` elimina los espacios al principio y al final. Así, un campo que solo contenga espacios se considera vacío. `htmlspecialchars()` prepara el dato recibido para mostrarlo en HTML; **no sustituye la comprobación de que el nombre tenga contenido**. El mensaje fijo de la primera rama no procede del formulario y no necesita esa conversión.

Este es un primer ejemplo de validación en el servidor. El atributo HTML `required` ayuda al usuario en el navegador, pero una aplicación no debe basar sus decisiones únicamente en él: el servidor debe comprobar los valores que recibe.

## 3.4. Elegir entre valores: `switch`

Cuando comparamos la misma variable con varias opciones, `switch` puede resultar más legible que una cadena de `elseif`:

```php
<?php
$dia = 'sabado';

switch ($dia) {
    case 'lunes':
        $mensaje = 'Comienza la semana.';
        break;
    case 'sabado':
    case 'domingo':
        $mensaje = 'Es fin de semana.';
        break;
    default:
        $mensaje = 'Es un día laborable.';
}

echo '<p>' . $mensaje . '</p>';
```

`break` sale del `switch`. Si lo olvidamos, PHP puede continuar ejecutando las instrucciones de los casos siguientes. En el ejemplo, agrupamos sábado y domingo intencionadamente: ambos conducen al mismo bloque. `default` atiende los valores que no coinciden con ningún `case`.

**Atención a los tipos:** `switch` utiliza comparaciones no estrictas. Si los tipos importan, valora `if` con `===` o `match`.

## 3.5. Una alternativa de PHP 8: `match`

`match` compara de forma estricta y **devuelve un valor**. No utiliza `break`:

```php
<?php
$dia = 'sabado';

$mensaje = match ($dia) {
    'lunes' => 'Comienza la semana.',
    'sabado', 'domingo' => 'Es fin de semana.',
    default => 'Es un día laborable.',
};

echo '<p>' . $mensaje . '</p>';
```

Cada alternativa produce directamente el texto asignado a `$mensaje`. Aquí incluimos `default` para contemplar otros valores. El `match` básico es útil cuando queremos **obtener un resultado a partir de un valor concreto**; para condiciones de intervalo como `$nota >= 5`, `if` suele expresar mejor la intención.

| Característica | `switch` | `match` |
| --- | --- | --- |
| Comparación | No estricta (`==`) | Estricta (`===`) |
| Resultado | Ejecuta instrucciones | Produce una expresión |
| Paso al caso siguiente | Posible si falta `break` | No ocurre |
| Rama restante | `default` | `default` |

`match` está disponible en nuestro entorno PHP 8.4. Lo introducimos para que conozcas también código PHP actual, además del `switch` que aparece en muchos proyectos existentes.

## 3.6. Condición breve: operador ternario

Si una condición sencilla elige entre **dos valores**, podemos utilizar `? :`:

```php
<?php
$edad = 20;
$mensaje = ($edad >= 18) ? 'Mayor de edad' : 'Menor de edad';
echo '<p>' . $mensaje . '</p>';
```

Se lee: «si `$edad >= 18`, utiliza el primer valor; en caso contrario, el segundo». Para varias decisiones o para ejecutar varias instrucciones, un bloque `if` suele ser más claro. Evita encadenar ternarios sin paréntesis.

El operador `??`, usado antes con `$_GET` y `$_POST`, responde a otra pregunta: «¿existe un valor distinto de `null`?». No es equivalente al ternario general. Por ejemplo, una cadena vacía **sí existe**, por lo que `'' ?? 'alternativa'` produce `''`.

## Comprueba lo aprendido

Antes de continuar, deberías poder predecir qué rama se ejecuta para distintos valores, distinguir `==` de `===`, explicar por qué un `switch` necesita `break`, reconocer cuándo `match` produce un valor y aplicar una comprobación sencilla a un dato recibido de un formulario.

## Practica lo aprendido

Realiza los **ejercicios 9 a 13** del [bloque 3: Condiciones](actividades.md#bloque-3-condiciones). Entrega juntos los archivos de este bloque en la tarea correspondiente de Moodle.

## Fuentes de consulta

- [Aitor Medrano: El lenguaje PHP](https://aitor-medrano.github.io/dwes2122/02php.html), sección «Condiciones».
- Manual oficial de PHP: [`if`](https://www.php.net/manual/es/control-structures.if.php), [`switch`](https://www.php.net/manual/es/control-structures.switch.php), [`match`](https://www.php.net/manual/es/control-structures.match.php) y [operadores de comparación](https://www.php.net/manual/es/language.operators.comparison.php).
