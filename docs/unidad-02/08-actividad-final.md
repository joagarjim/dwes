# 8. Actividad final de la unidad

## Tienda informática: catálogo y presupuesto

Desarrolla una aplicación PHP que muestre un catálogo de productos y permita calcular un presupuesto mediante un formulario.

Esta actividad es **individual** y reúne los contenidos trabajados en la unidad: variables, operadores, formularios, condiciones, bucles, arrays, funciones y reutilización de archivos.

La aplicación utilizará un catálogo fijo guardado en un array y funcionará en el entorno Apache y PHP de Docker utilizado en clase.

## 1. Organización del proyecto

Guarda la aplicación en una carpeta llamada `actividad-final-ud2`. Organízala en estos archivos:

| Archivo | Contenido |
|---|---|
| `index.php` | Catálogo, formulario y resultado del presupuesto |
| `productos.php` | Array con los datos de los productos |
| `funciones.php` | Funciones propias para los cálculos y el formato |
| `encabezado.php` | Comienzo del documento HTML y título variable |
| `pie.php` | Pie de página y cierre del documento HTML |

Carga los archivos necesarios mediante `include` o `require_once`, utilizando `__DIR__` para indicar su ubicación. Define el título antes de incluir la cabecera y escápalo al mostrarlo.

Puedes añadir un archivo CSS para mejorar la presentación. No es necesario realizar un diseño complejo: el objetivo principal es aplicar correctamente PHP.

## 2. Catálogo de productos

En `productos.php`, crea un array de productos. Utiliza el identificador como clave y guarda el nombre, la categoría y el precio de cada producto en un array asociativo.

Incluye estos datos:

| Identificador | Nombre | Categoría | Precio unitario |
|---|---|---|---:|
| 1 | Teclado | Periféricos | 35,00 € |
| 2 | Ratón | Periféricos | 20,00 € |
| 3 | Monitor | Pantallas | 180,00 € |
| 4 | Memoria USB | Almacenamiento | 12,50 € |
| 5 | Auriculares | Audio | 45,00 € |

Guarda los precios como números en PHP, utilizando el punto para los decimales. La coma decimal y el símbolo del euro se añadirán al presentarlos.

En `index.php`:

- Genera una tabla de productos mediante `foreach`. No escribas a mano las filas de datos.
- Muestra la cantidad de productos con `count()`.
- Presenta los precios con dos decimales, coma decimal, punto de miles y símbolo del euro.
- Escapa los nombres y las categorías al insertarlos en HTML.

## 3. Formulario de presupuesto

Crea un formulario que envíe los datos mediante **POST a la misma página**. Debe contener:

| Campo | Elemento del formulario |
|---|---|
| Nombre del cliente | Campo de texto |
| Producto | Lista desplegable generada a partir del catálogo; envía su identificador |
| Cantidad | Lista desplegable con valores del 1 al 10, generados mediante un bucle `for` |
| Tipo de cliente | Botones de radio: «Particular» y «Estudiante», con valores `particular` y `estudiante` |
| Observaciones | Área de texto opcional |
| Calcular presupuesto | Botón de envío |

Antes del primer envío se mostrarán el catálogo y el formulario, sin mensajes de error ni presupuesto.

## 4. Validación en el servidor

Después del envío, comprueba en PHP que:

- El nombre sea una cadena y no esté vacío después de aplicar `trim()`.
- El identificador del producto sea válido y corresponda a una clave existente del catálogo.
- La cantidad sea un entero entre 1 y 10.
- El tipo de cliente sea exactamente `particular` o `estudiante`.
- Las observaciones sean una cadena; pueden quedar vacías después de aplicar `trim()`.

!!! important "Validación antes del cálculo"
    No te apoyes únicamente en los atributos HTML del formulario. Si algún dato no es válido, muestra un mensaje claro y no calcules el presupuesto.

Obtén el precio del producto desde el array del servidor utilizando su identificador. El formulario debe enviar el producto elegido, no un precio que el visitante pueda modificar.

## 5. Funciones y cálculo del presupuesto

Define en `funciones.php` estas funciones:

| Función | Resultado que debe devolver |
|---|---|
| `calcularSubtotal(float $precio, int $cantidad): float` | Precio unitario multiplicado por la cantidad |
| `obtenerDescuento(string $tipoCliente): int` | `10` para estudiantes y `0` para particulares |
| `calcularDescuento(float $subtotal, int $porcentaje): float` | Importe del descuento |
| `calcularIVA(float $base, float $porcentaje = 21): float` | Importe del IVA |
| `formatearPrecio(float $importe): string` | Importe con dos decimales, separadores y símbolo del euro |

Las funciones deben **devolver resultados**, sin mostrar HTML dentro de ellas. En `obtenerDescuento()`, utiliza una condición para decidir el porcentaje.

Realiza el cálculo en este orden:

1. Calcula el subtotal: precio unitario × cantidad.
2. Obtén el porcentaje de descuento según el tipo de cliente.
3. Calcula el importe del descuento: subtotal × porcentaje / 100.
4. Resta el descuento al subtotal para obtener la base.
5. Calcula el IVA del 21 % sobre esa base.
6. Suma la base y el IVA para obtener el total.

Utiliza las funciones definidas y guarda sus resultados en variables. Aplica el formato de los importes al mostrarlos: no utilices las cadenas devueltas por `formatearPrecio()` para realizar cálculos.

## 6. Presentación del resultado

Cuando todos los datos sean válidos, muestra:

- Nombre del cliente.
- Producto y categoría.
- Tipo de cliente.
- Precio unitario y cantidad.
- Subtotal.
- Porcentaje e importe del descuento.
- Base después del descuento.
- Importe del IVA.
- Total del presupuesto.

Si se han escrito observaciones, muéstralas conservando los saltos de línea. Primero utiliza `htmlspecialchars()` y después `nl2br()`.

Escapa los demás textos al insertarlos en HTML. Organiza el resultado con títulos y una tabla o una distribución clara.

## 7. Comprobaciones

Prueba, al menos, los siguientes casos:

| Caso | Resultado esperado |
|---|---|
| Particular compra 2 teclados | Subtotal: 70,00 € · Descuento: 0,00 € · IVA: 14,70 € · Total: **84,70 €** |
| Estudiante compra 2 teclados | Subtotal: 70,00 € · Descuento: 7,00 € · Base: 63,00 € · IVA: 13,23 € · Total: **76,23 €** |
| Nombre vacío o compuesto solo por espacios | Mensaje de error y sin presupuesto |
| Producto inexistente | Mensaje de error y sin presupuesto |
| Cantidad fuera del intervalo 1–10 o no entera | Mensaje de error y sin presupuesto |
| Tipo de cliente distinto de las dos opciones permitidas | Mensaje de error y sin presupuesto |
| Observaciones vacías | Presupuesto válido sin sección de observaciones |
| Observaciones con dos líneas y `<strong>Hola</strong>` | Saltos visibles y etiquetas mostradas como texto |

Para probar datos que el navegador no permite seleccionar normalmente, modifica temporalmente los valores del formulario desde las herramientas de desarrollo. El servidor debe detectarlos igualmente. Restaura los valores después de la prueba.

## Entrega

Entrega en Moodle estos dos archivos, sustituyendo `apellido` y `nombre` por tus datos en minúsculas, sin espacios ni tildes:

- `apellido-nombre-ud02.zip`.
- `apellido-nombre-ud02.pdf`.

### Contenido del ZIP

El ZIP contendrá la carpeta `actividad-final-ud2`, con los cinco archivos PHP y los recursos propios necesarios para ejecutar la aplicación. Si has creado un proyecto Docker independiente, incluye también su configuración e indica en el PDF cómo ponerlo en marcha.

Antes de comprimir, comprueba que no falta ningún archivo y que la aplicación funciona en tu entorno de clase. No incluyas archivos ajenos a la actividad.

### Contenido del PDF

Incluye tu nombre y apellidos y estas cuatro capturas, con un breve rótulo para identificar cada caso:

1. Catálogo y formulario antes del primer envío.
2. Presupuesto de un particular que compra dos teclados.
3. Presupuesto de un estudiante que compra dos teclados.
4. Un caso de validación incorrecta que muestre el error y no genere presupuesto.

Incluye comentarios breves en el código que expliquen las validaciones y los cálculos. Las capturas complementan el proyecto: también debes entregar los archivos PHP.

## Criterios de valoración

| Aspecto | Puntuación |
|---|---:|
| Organización de archivos y reutilización de cabecera, pie y funciones | 1 punto |
| Catálogo con arrays, recorrido y formato de los datos | 1,5 puntos |
| Formulario POST, opciones dinámicas y estado inicial | 1,5 puntos |
| Validación de los datos en el servidor | 2 puntos |
| Funciones y cálculo correcto del presupuesto | 2,5 puntos |
| Presentación, escape de textos, observaciones y evidencias | 1,5 puntos |
| **Total** | **10 puntos** |

## Requisitos mínimos

- [ ] El catálogo procede del array y se muestra mediante `foreach`.
- [ ] El formulario se envía mediante POST a la misma página.
- [ ] Las cantidades se generan mediante `for` y los productos desde el catálogo.
- [ ] Los datos se validan en PHP antes de calcular.
- [ ] El precio se obtiene del catálogo del servidor.
- [ ] Las funciones solicitadas devuelven resultados y se utilizan en la aplicación.
- [ ] Los dos presupuestos de comprobación coinciden con los resultados indicados.
- [ ] Los textos se escapan y las observaciones conservan sus saltos de línea.
- [ ] La aplicación reutiliza cabecera y pie.
- [ ] Se entregan el proyecto completo y las cuatro capturas solicitadas.
