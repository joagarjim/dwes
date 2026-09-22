# 4. Bucles en PHP

En el apartado anterior elegimos qué instrucciones ejecutar. Ahora repetiremos un bloque mientras se cumpla una condición o durante un número determinado de veces. Ya conoces los bucles de otros lenguajes; aquí practicaremos su sintaxis en PHP y cómo utilizarlos para generar HTML.

Al terminar podrás usar `while`, `do...while` y `for`, identificar cuándo termina un bucle y construir elementos HTML repetidos. Dejaremos `foreach` para el apartado de arrays.

## 4.1. Repetir mientras se cumple una condición: `while`

`while` comprueba la condición **antes** de cada vuelta. Si es falsa desde el principio, no ejecuta el bloque ninguna vez:

```php
<?php
$numero = 1;

while ($numero <= 5) {
    echo '<p>Número: ' . $numero . '</p>';
    $numero++;
}
```

Este código genera cinco párrafos, del 1 al 5. La variable `$numero` comienza en 1, aumenta al final de cada vuelta y, cuando llega a 6, la condición deja de cumplirse. Si olvidamos `$numero++`, el bucle no llegará a su fin por sí solo. Antes de ejecutar un bucle, identifica el valor inicial, la condición y qué cambia en cada iteración.

## 4.2. Ejecutar primero y comprobar después: `do...while`

`do...while` comprueba la condición **al final**. Por eso el bloque se ejecuta al menos una vez:

```php
<?php
$numero = 8;

do {
    echo '<p>Número: ' . $numero . '</p>';
    $numero++;
} while ($numero <= 5);
```

El ejemplo muestra `8` una vez aunque la condición ya sea falsa. Con los mismos valores iniciales, un `while` no mostraría nada. Observa el punto y coma después de `while (...)`.

## 4.3. Un número conocido de repeticiones: `for`

`for` reúne en la cabecera el valor inicial, la condición y la actualización:

```php
<?php
for ($numero = 1; $numero <= 5; $numero++) {
    echo '<p>Número: ' . $numero . '</p>';
}
```

La inicialización se realiza una vez. La condición se comprueba antes de cada vuelta y la actualización se ejecuta al terminarla. Si queremos contar hacia atrás, podemos usar `$numero--` y una condición como `$numero >= 1`. En un recorrido de 1 a 5, usar `< 5` en lugar de `<= 5` dejaría fuera el 5.

| Estructura | ¿Cuándo comprueba la condición? | Uso habitual en esta unidad |
| --- | --- | --- |
| `while` | Antes de cada vuelta | Repetir hasta que cambie una condición |
| `do...while` | Después de cada vuelta | Ejecutar al menos una vez |
| `for` | Antes de cada vuelta | Recorrer un número conocido de valores |

## 4.4. Construir HTML con un bucle

PHP puede repetir filas de una tabla sin escribirlas una por una. El HTML fijo queda fuera del bucle y cada iteración produce una fila:

```php
<table>
    <thead><tr><th>Número</th><th>Cuadrado</th></tr></thead>
    <tbody>
        <?php for ($numero = 1; $numero <= 5; $numero++) { ?>
            <tr>
                <td><?= $numero ?></td>
                <td><?= $numero * $numero ?></td>
            </tr>
        <?php } ?>
    </tbody>
</table>
```

El navegador recibe una tabla HTML con cinco filas en el cuerpo: no recibe el `for`. El mismo patrón permite generar listas `<li>`, opciones o párrafos. Utilizamos llaves y abrimos o cerramos PHP donde corresponde para mantener el HTML legible.

Si el límite procede de un formulario, primero hay que comprobarlo en el servidor. Por ejemplo, para aceptar solo enteros del 1 al 20:

```php
<?php
$limiteTexto = $_GET['limite'] ?? '';
$limite = null;

if (is_string($limiteTexto) && ctype_digit($limiteTexto)) {
    $limite = (int) $limiteTexto;
}

if ($limite !== null && $limite >= 1 && $limite <= 20) {
    for ($numero = 1; $numero <= $limite; $numero++) {
        echo '<p>' . $numero . '</p>';
    }
} else {
    echo '<p>Introduce un entero entre 1 y 20.</p>';
}
```

`ctype_digit()` comprueba que la cadena contenga solo dígitos. Después la convertimos a entero y comprobamos el intervalo. Un atributo `max` en el formulario ayuda en el navegador, pero no sustituye estas comprobaciones en el servidor.

## 4.5. Bucles anidados

Un bucle puede contener otro. En una tabla de multiplicar, el bucle exterior elige la fila y el interior genera sus columnas:

```php
<?php
for ($fila = 1; $fila <= 3; $fila++) {
    for ($columna = 1; $columna <= 3; $columna++) {
        echo '<span>' . ($fila * $columna) . ' </span>';
    }
    echo '<br>';
}
```

El bucle interior se ejecuta por completo por cada vuelta del exterior: aquí realiza `3 × 3 = 9` iteraciones. Usamos variables distintas para que cada bucle controle su propio recorrido. En una página real podemos sustituir los `<span>` y `<br>` por una tabla HTML.

## 4.6. Salir o saltar una vuelta: `break` y `continue`

`break` termina el bucle actual. `continue` salta lo que queda de la vuelta actual y pasa a la siguiente:

```php
<?php
for ($numero = 1; $numero <= 10; $numero++) {
    if ($numero === 3) {
        continue; // No mostramos el 3.
    }
    if ($numero === 8) {
        break; // Terminamos antes de mostrar el 8.
    }
    echo '<p>' . $numero . '</p>';
}
```

Se muestran `1, 2, 4, 5, 6, 7`. En bucles anidados, sin indicar otro nivel, ambas instrucciones afectan al bucle más cercano. `break` también se utilizó en `switch`, pero aquí sirve para detener la repetición. Procura que la condición de salida habitual sea visible en la cabecera; reserva `break` y `continue` para casos en los que aclaren el código.

## Comprueba lo aprendido

Antes de continuar, deberías poder predecir cuántas veces se ejecuta cada bucle, explicar por qué `do...while` siempre se ejecuta al menos una vez, detectar un límite mal planteado y distinguir el efecto de `break` y `continue`.

## Practica lo aprendido

Realiza los **ejercicios 14 a 20** del [bloque 4: Bucles](actividades.md#bloque-4-bucles). Entrega juntos los archivos del bloque en la tarea correspondiente de Moodle.

## Fuentes de consulta

- [Aitor Medrano: El lenguaje PHP](https://aitor-medrano.github.io/dwes2122/02php.html), sección «Bucles».
- Manual oficial de PHP: [`while`](https://www.php.net/manual/es/control-structures.while.php), [`do...while`](https://www.php.net/manual/es/control-structures.do.while.php), [`for`](https://www.php.net/manual/es/control-structures.for.php), [`break`](https://www.php.net/manual/es/control-structures.break.php) y [`continue`](https://www.php.net/manual/es/control-structures.continue.php).
