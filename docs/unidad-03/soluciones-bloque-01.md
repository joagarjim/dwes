# Unidad 3. Soluciones del bloque 1

Material del profesor. Las soluciones son orientativas: se admitirán otras que cumplan los requisitos. En los ejemplos siguientes, cada clase se guarda en su archivo y el código de prueba en `index.php`, salvo el ejercicio 4.

## Ejercicio 1. Libro

**Libro.php**

```php
<?php
declare(strict_types=1);

class Libro
{
    public string $titulo = '';
    public string $autor = '';
    public int $paginas = 0;

    public function obtenerFicha(): string
    {
        return $this->titulo . ' — ' . $this->autor
            . ' (' . $this->paginas . ' páginas)';
    }
}
```

**index.php**

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Libro.php';

$primero = new Libro();
$primero->titulo = 'Libro de PHP';
$primero->autor = 'Ana';
$primero->paginas = 200;

$segundo = new Libro();
$segundo->titulo = 'Libro de bases de datos';
$segundo->autor = 'Luis';
$segundo->paginas = 150;

echo '<p>' . htmlspecialchars($primero->obtenerFicha(), ENT_QUOTES, 'UTF-8') . '</p>';
echo '<p>' . htmlspecialchars($segundo->obtenerFicha(), ENT_QUOTES, 'UTF-8') . '</p>';

$primero->paginas = 220;
echo '<p>' . htmlspecialchars($primero->obtenerFicha(), ENT_QUOTES, 'UTF-8') . '</p>';
echo '<p>' . htmlspecialchars($segundo->obtenerFicha(), ENT_QUOTES, 'UTF-8') . '</p>';
```

La clase es `Libro`; hay dos objetos. `$this` representa el objeto sobre el que se hace la llamada. La modificación solo afecta al primer libro. Verificar que no escriben `$titulo` en lugar de `$this->titulo`.

## Ejercicio 2. Rectangulo

**Rectangulo.php**

```php
<?php
declare(strict_types=1);

class Rectangulo
{
    public float $base = 0.0;
    public float $altura = 0.0;

    public function calcularArea(): float
    {
        return $this->base * $this->altura;
    }

    public function calcularPerimetro(): float
    {
        return 2 * ($this->base + $this->altura);
    }

    public function esCuadrado(): bool
    {
        return $this->base === $this->altura;
    }
}
```

**index.php**

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Rectangulo.php';

$r1 = new Rectangulo();
$r1->base = 5.0;
$r1->altura = 3.0;
$r2 = new Rectangulo();
$r2->base = 4.0;
$r2->altura = 4.0;

echo '<p>Área 1: ' . $r1->calcularArea() . '</p>';
echo '<p>Perímetro 1: ' . $r1->calcularPerimetro() . '</p>';
if ($r1->esCuadrado()) {
    echo '<p>El primero es cuadrado.</p>';
} else {
    echo '<p>El primero no es cuadrado.</p>';
}

echo '<p>Área 2: ' . $r2->calcularArea() . '</p>';
echo '<p>Perímetro 2: ' . $r2->calcularPerimetro() . '</p>';
if ($r2->esCuadrado()) {
    echo '<p>El segundo es cuadrado.</p>';
} else {
    echo '<p>El segundo no es cuadrado.</p>';
}
```

Resultados: `15`, `16`, no cuadrado; `16`, `16`, cuadrado. La comparación exacta es suficiente para estos datos asignados directamente; no generalizarla a resultados de cálculos con coma flotante sin considerar tolerancias.

## Ejercicio 3. Marcador

**Marcador.php**

```php
<?php
declare(strict_types=1);

class Marcador
{
    public int $puntos = 0;

    public function anotarPunto(): void
    {
        $this->puntos++;
    }

    public function reiniciar(): void
    {
        $this->puntos = 0;
    }

    public function obtenerPuntos(): int
    {
        return $this->puntos;
    }
}
```

**index.php**

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Marcador.php';

$m1 = new Marcador();
$m2 = new Marcador();
$m1->anotarPunto();
$m1->anotarPunto();
$m1->anotarPunto();
$m2->anotarPunto();

echo '<p>' . $m1->obtenerPuntos() . ' / ' . $m2->obtenerPuntos() . '</p>';
$m1->reiniciar();
echo '<p>' . $m1->obtenerPuntos() . ' / ' . $m2->obtenerPuntos() . '</p>';
```

Se obtiene `3 / 1` y `0 / 1`. `reiniciar()` tiene retorno `void`: modifica el estado y no devuelve un resultado que debamos mostrar. La consulta se realiza después.

## Ejercicio 4. Contador

Salida, en cuatro líneas: **2, 1, 2, 0**.

Antes del último `new` se han creado dos objetos. `$a` y `$c` acceden al primero; `$b`, al segundo. Por ello, las llamadas sobre `$a` y `$c` incrementan el mismo contador.

El último `new` crea un tercer objeto con valor cero y asigna su identificador a `$c`. `$a` continúa accediendo al primero, que conserva el valor dos. En total se han creado tres objetos, y los tres siguen siendo accesibles al terminar.

No describir la asignación habitual de objetos como si se hubiera utilizado el operador de referencia `&`: aquí se copia el identificador que permite acceder al mismo objeto.

## Ejercicio 5. Entrada

Errores:

1. `$precio` es una variable local no definida; necesitamos `$this->precio`.
2. Falta `new`: `Entrada()` intentaría llamar a una función con ese nombre.
3. Se usa sintaxis de array para un objeto que no implementa acceso como array; necesitamos `$entrada->evento`.

**Entrada.php**

```php
<?php
declare(strict_types=1);

class Entrada
{
    public string $evento = 'Concierto';
    public float $precio = 12.0;

    public function calcularTotal(int $cantidad): float
    {
        return $this->precio * $cantidad;
    }
}
```

**index.php**

```php
<?php
declare(strict_types=1);
require_once __DIR__ . '/Entrada.php';

$entrada = new Entrada();
echo '<h1>' . htmlspecialchars($entrada->evento, ENT_QUOTES, 'UTF-8') . '</h1>';
echo '<p>3 entradas: ' . number_format($entrada->calcularTotal(3), 2, ',', '.') . ' €</p>';
echo '<p>1 entrada: ' . number_format($entrada->calcularTotal(1), 2, ',', '.') . ' €</p>';
echo '<p>5 entradas: ' . number_format($entrada->calcularTotal(5), 2, ',', '.') . ' €</p>';
```

Resultados: `36,00 €`, `12,00 €` y `60,00 €`. El precio sigue siendo `12.0`: calcular el total no modifica la propiedad.

## Orientaciones para la puesta en común

Priorizar la explicación de `$this` con dos instancias, el estado frente a las variables locales y la diferencia entre `return` y `echo`. El ejercicio 4 permite comprobar si el alumnado entiende la identidad de los objetos. El 5 permite revisar sintaxis, organización de archivos y casos de prueba.

No exigir todavía constructores ni encapsulación. Las propiedades públicas son un paso didáctico que se revisará en el apartado 2.
