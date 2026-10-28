# 7. Actividad final de la unidad

## Tienda informática: catálogo orientado a objetos

Desarrolla una aplicación PHP que gestione un catálogo de productos físicos y digitales mediante clases y objetos. La página mostrará los datos del catálogo, calculará sus importes con envío y permitirá comprobar operaciones de reserva y reposición de existencias programadas en el código.

Esta actividad es **individual**. Integra clases, constructores, encapsulación, relaciones entre objetos, arrays de objetos, constantes, elementos estáticos, herencia, polimorfismo, interfaces, excepciones, pruebas y documentación.

La explicaremos en clase y se realizará principalmente en casa. Dedicación orientativa: **unas 4 horas**, según el repaso necesario. La fecha de entrega será la indicada en Moodle.

Trabaja en el entorno Apache y PHP de Docker utilizado en clase. Los datos se escribirán en PHP y se conservarán únicamente durante cada petición. No se requieren formularios, sesiones, base de datos, Laravel ni envío de correos. Puedes partir de los ejemplos y ejercicios de la unidad, adaptándolos y comprendiendo su funcionamiento.

## 1. Organización del proyecto

Guarda la aplicación en una carpeta llamada `actividad-final-ud3`:

| Archivo | Responsabilidad |
| --- | --- |
| `Describible.php` | Interfaz común para obtener descripciones. |
| `Categoria.php` | Datos de una categoría. |
| `Producto.php` | Clase abstracta con los datos y operaciones comunes de los productos. |
| `ProductoFisico.php` | Peso, existencias, envío y operaciones de stock. |
| `ProductoDigital.php` | Producto sin gastos de envío. |
| `Catalogo.php` | Colección de productos y cálculo del valor conjunto. |
| `index.php` | Creación del catálogo, presentación y demostración de operaciones. |
| `pruebas.php` | Comprobaciones de funcionamiento y validación. |
| `README.md` | Instrucciones, explicación del diseño y tabla de pruebas. |

Carga los archivos necesarios mediante `require_once` y `__DIR__`. Utiliza `declare(strict_types=1);` al principio de cada archivo PHP y tipos en propiedades, parámetros y resultados. Cada clase o interfaz tendrá su propio archivo.

Las clases no deben imprimir mensajes ni generar HTML. La presentación y la captura de excepciones se realizarán en las páginas. El diseño visual puede ser sencillo: se evalúa principalmente la programación.

## 2. Interfaz y categorías

### Interfaz Describible

Declara el método público `obtenerDescripcion(): string`, sin cuerpo.

### Clase Categoria

Implementa `Describible` y declara una propiedad privada `nombre` de tipo `string`.

Su constructor recibe el nombre, aplica `trim()` y lanza `InvalidArgumentException` si queda vacío. Añade:

| Método | Resultado |
| --- | --- |
| `getNombre(): string` | Nombre de la categoría. |
| `obtenerDescripcion(): string` | `Categoría: ` seguido del nombre. |

Los productos recibirán un objeto `Categoria`, no una cadena con su nombre. Varios productos podrán compartir la misma categoría.

## 3. Clase abstracta Producto

Haz que `Producto` implemente `Describible`. Declara estas propiedades privadas:

| Propiedad | Tipo y significado |
| --- | --- |
| `nombre` | `string`: nombre del producto. |
| `precio` | `float`: precio de una unidad, sin el envío. |
| `categoria` | `Categoria`: objeto relacionado. |
| `totalCreados` | `static int`, inicialmente `0`: construcciones válidas de productos en esa ejecución. |

El constructor recibirá nombre, precio y categoría. Aplica `trim()` al nombre y rechaza el nombre vacío o el precio negativo mediante `InvalidArgumentException`. El precio cero está permitido. Guarda los datos e incrementa el contador después de comprobar los argumentos comunes.

Implementa estas operaciones:

| Método | Comportamiento |
| --- | --- |
| `getNombre(): string` | Devuelve el nombre. |
| `getPrecio(): float` | Devuelve el precio. Debe declararse **público y final**. |
| `getCategoria(): Categoria` | Devuelve el objeto de categoría. |
| `obtenerDescripcion(): string` | Devuelve `Producto: ` seguido del nombre. |
| `calcularTotal(): float` | Devuelve el precio más el resultado de `$this->calcularGastosEnvio()`. |
| `getTotalCreados(): int` | Método público **estático** que consulta el contador. |
| `calcularGastosEnvio(): float` | Método público **abstracto**, sin cuerpo. |

No añadas setters generales: los datos comunes se establecen al construir el producto. Cada tipo concreto aportará su envío.

## 4. Productos físicos y digitales

### ProductoFisico

Hereda de `Producto` y añade propiedades privadas `peso` de tipo `float` y `stock` de tipo `int`. Declara una constante de clase `TARIFA_POR_KG` de valor `2.0`.

El constructor recibe nombre, precio, categoría, peso y stock. **Comprueba primero los datos específicos**: el peso debe ser positivo y el stock no negativo. Si son inválidos, lanza `InvalidArgumentException` antes de llamar al constructor del padre. Después inicializa los datos comunes mediante `parent::__construct()` y guarda peso y stock. Así un producto físico inválido no incrementa el contador.

Implementa:

| Método | Comportamiento |
| --- | --- |
| `getPeso(): float` | Devuelve el peso. |
| `getStock(): int` | Devuelve las unidades disponibles. |
| `obtenerDescripcion(): string` | Reutiliza la descripción del padre con `parent::` y añade el peso en kg. |
| `calcularGastosEnvio(): float` | Peso por la constante `TARIFA_POR_KG`, consultada mediante `self::`. |
| `reservar(int $unidades): void` | Descuenta unidades si la cantidad es positiva y no supera el stock. Si se rechaza, lanza `InvalidArgumentException` y conserva el stock. |
| `reponer(int $unidades): void` | Suma una cantidad positiva. Si recibe cero o un negativo, lanza `InvalidArgumentException` y conserva el stock. |

### ProductoDigital

Declárala **final** y haz que herede de `Producto`.

- Utiliza el constructor heredado: no necesita datos adicionales.
- Amplía la descripción del padre añadiendo ` / Descarga digital`.
- Implementa `calcularGastosEnvio()` devolviendo `0.0`.

El producto digital no tiene stock en esta práctica ni necesita las operaciones de reserva y reposición del físico. La función común de cálculo solo utilizará operaciones disponibles en `Producto`.

## 5. Colección de objetos: Catalogo

Declara una propiedad privada `productos` de tipo `array`, inicialmente vacía. Implementa:

| Método | Comportamiento |
| --- | --- |
| `anadirProducto(Producto $producto): void` | Añade el objeto recibido. No crea otro producto. |
| `getProductos(): array` | Devuelve el array para recorrerlo desde la página. |
| `contarProductos(): int` | Devuelve el número de elementos del catálogo. |
| `calcularValorTotal(): float` | Recorre los objetos y suma el resultado de `calcularTotal()` de cada uno. Si está vacío, devuelve `0.0`. |

No utilices `if`, `switch` ni `instanceof` para elegir la fórmula del envío. Cada producto debe responder mediante su implementación.

Añade cada producto una sola vez. No se requieren identificadores, búsquedas, eliminaciones ni control de duplicados.

## 6. Datos y presentación del catálogo

En `index.php`, crea estas categorías y productos. Reutiliza el mismo objeto `Periféricos` para teclado y ratón:

| Nombre | Tipo | Categoría | Precio | Peso | Stock inicial |
| --- | --- | --- | ---: | ---: | ---: |
| Teclado | Físico | Periféricos | 35,00 € | 0.5 kg | 5 |
| Ratón | Físico | Periféricos | 20,00 € | 0.2 kg | 8 |
| Monitor | Físico | Pantallas | 180,00 € | 4 kg | 2 |
| Curso PHP | Digital | Formación | 40,00 € | No corresponde. | No corresponde. |

Incorpóralos a un objeto `Catalogo` y genera con `foreach` una tabla de descripción, categoría, precio, envío y total. Consulta el nombre de la categoría mediante el objeto relacionado.

Resultados esperados:

| Producto | Envío | Total de una unidad |
| --- | ---: | ---: |
| Teclado | 1,00 € | 36,00 € |
| Ratón | 0,40 € | 20,40 € |
| Monitor | 8,00 € | 188,00 € |
| Curso PHP | 0,00 € | 40,00 € |
| **Valor conjunto** | | **284,40 €** |

El envío es una regla didáctica por producto; no estamos calculando un paquete conjunto. No se aplican impuestos ni descuentos en esta actividad.

Muestra también:

- Cantidad de productos del catálogo: **4**.
- Contador estático de productos creados, consultado antes de otras pruebas: **4**. Crear categorías o añadir productos al catálogo no debe incrementarlo.
- Una sección de descripciones obtenidas mediante una función `describir(Describible $elemento): string`. Llama a esa misma función con la categoría Periféricos y con el teclado: debe aceptar ambos objetos gracias a la interfaz.

Escapa los textos con `htmlspecialchars()` y formatea los importes con dos decimales. Los métodos de cálculo deben seguir devolviendo números.

## 7. Demostración de operaciones y captura de errores

Después de mostrar el catálogo inicial, utiliza el teclado ya creado y realiza estas operaciones **en este orden**, mostrando el resultado y el stock después de cada una:

| Operación | Resultado esperado | Stock posterior |
| --- | --- | ---: |
| Consultar stock inicial | 5 unidades. | 5 |
| Reservar 2 | Se completa. | 3 |
| Reponer 1 | Se completa. | 4 |
| Intentar reservar 5 | Se rechaza mediante excepción. | 4 |

Utiliza bloques `try/catch` para los intentos. Coloca los mensajes de éxito después de las llamadas y muestra el mensaje de la excepción en los rechazos. El último fallo no debe interrumpir el resto de la página.

La reserva afecta al stock, no al precio ni al valor del catálogo: este último sigue representando una unidad de cada producto. No se pide calcular ventas, facturas o presupuestos de cantidades múltiples.

Al recargar, se crean de nuevo los objetos con sus valores iniciales y se repite la secuencia: no se guardan cambios entre peticiones.

## 8. Pruebas del funcionamiento

Prepara `pruebas.php` con comprobaciones separadas del catálogo de presentación. Crea un estado conocido para cada prueba; si una fila comienza con stock 5, no reutilices sin reiniciar el objeto de la anterior.

Incluye al menos estos casos:

| Caso | Resultado esperado |
| --- | --- |
| Catálogo vacío | Cero productos y valor total 0,00 €. |
| Catálogo de los cuatro productos indicados | Cuatro productos y valor total 284,40 €. |
| Producto digital | Envío 0,00 €; total igual al precio. |
| Físico con stock 5: reservar 5 | Se completa y deja stock 0. |
| Físico con stock 5: reservar 6 | Excepción y stock conservado en 5. |
| Físico con stock 5: reservar 0 | Excepción y stock conservado en 5. |
| Físico con stock 5: reponer -1 | Excepción y stock conservado en 5. |
| Intentar construir un producto con precio negativo | Excepción; no se completa la construcción. |
| Intentar construir un físico de peso 0 | Excepción; el contador conserva el valor anterior al intento. |
| Intentar construir una categoría de espacios | Excepción; no se completa la construcción. |

Los constructores que quieras probar deben ejecutarse dentro de sus correspondientes `try`. Un fallo no debe impedir que se ejecuten las demás pruebas. Cuando compruebes un precio o un total decimal, compara los importes redondeados a dos decimales o utiliza una tolerancia pequeña: no dependas de la igualdad exacta de resultados de coma flotante.

Muestra una identificación de cada prueba, su resultado y, cuando corresponda, el estado posterior. No basta con provocar errores: debes comprobar que se produce el rechazo correcto y que los datos permanecen válidos.

No es obligatorio utilizar `Throwable`, `finally`, excepciones propias ni herramientas de pruebas externas.

## 9. Documentación y explicación del diseño

Documenta con bloques PHPDoc los métodos `reservar()`, `reponer()` y `calcularValorTotal()`. Explica su propósito, parámetros cuando existan, resultado y excepciones que puedan comunicar. No sustituyas el código por comentarios ni comentes cada instrucción de forma mecánica.

En `README.md`, incluye:

1. Nombre y apellidos, finalidad del proyecto e instrucciones para ejecutarlo en el entorno de clase.
2. Una frase sobre la responsabilidad de cada clase.
3. Una explicación breve de por qué `Producto` es abstracta y `Describible` es una interfaz. Indica qué dos clases implementan directamente la interfaz.
4. Qué objeto representa `$this` cuando se ejecuta `calcularTotal()` sobre el teclado y qué método calcula su envío.
5. El significado de los dos usos de `final`, la constante de envío y el contador estático.
6. Una tabla con las pruebas del punto 8: caso, resultado esperado, resultado observado y si coincide.

Esta actividad recoge la práctica del apartado 6: **no habrá un bloque adicional de ejercicios de excepciones, pruebas y documentación**.

## Entrega

Entrega en Moodle, sustituyendo apellido y nombre por tus datos en minúsculas, sin espacios ni tildes:

- `apellido-nombre-ud03.zip`.
- `apellido-nombre-ud03.pdf`.

### Contenido del ZIP

La carpeta `actividad-final-ud3`, con los ocho archivos PHP indicados en el punto 1, `README.md` y los recursos propios necesarios. Si añades configuración Docker propia, incluye las instrucciones para ponerla en marcha.

Comprueba que no falte ningún archivo. No incluyas soluciones ajenas a la actividad ni carpetas de otras unidades.

### Contenido del PDF

Incluye nombre y apellidos y tres capturas con un rótulo breve:

1. Catálogo con los cuatro productos, sus importes y los contadores.
2. Secuencia de stock del teclado, incluyendo la reserva rechazada y el stock final de 4.
3. Resultados de las pruebas, donde se vea al menos un constructor rechazado y una operación que conserva el estado tras fallar.

La documentación y la tabla completa de pruebas van en el README; no es necesario duplicarlas en el PDF. Las capturas complementan el código y no lo sustituyen.

## Criterios de valoración

| Aspecto | Puntuación |
| --- | ---: |
| Organización, tipos, encapsulación y constructores con validación | 1,5 puntos |
| Categorías, relaciones entre objetos y colección Catalogo | 1,5 puntos |
| Herencia, clase abstracta, sobrescritura, polimorfismo y final | 2 puntos |
| Interfaz Describible, constante y contador estático | 1 punto |
| Cálculos, operaciones de stock y tratamiento de excepciones | 2 puntos |
| Pruebas, PHPDoc, README, salida segura y evidencias | 2 puntos |
| **Total** | **10 puntos** |

La actividad permite trabajar el uso de POO y la prueba y documentación del código, vinculados a los criterios RA5.g y RA5.h. La puntuación de esta tarea se integrará en la evaluación conforme a la programación del módulo.

## Requisitos mínimos

- [ ] Cada clase e interfaz está en su archivo y se carga correctamente.
- [ ] Las propiedades están encapsuladas y los constructores rechazan datos inválidos.
- [ ] Los productos contienen objetos Categoria y el catálogo contiene objetos Producto.
- [ ] Producto es abstracta y las hijas implementan su cálculo de envío.
- [ ] Los dos usos de final están presentes y funcionan según su propósito.
- [ ] La interfaz permite describir una categoría y un producto mediante la misma función.
- [ ] Se utilizan la constante de envío y el contador estático.
- [ ] El valor conjunto del catálogo es 284,40 €.
- [ ] La secuencia del teclado termina con stock 4 y el rechazo no lo modifica.
- [ ] Se comprueban constructores, límites y operaciones rechazadas desde pruebas.php.
- [ ] Se entregan código, documentación y las tres capturas solicitadas.

Material elaborado por **Joaquín García** para DWES de 2.º DAW del **IES Portada Alta**. Licencia **CC BY-NC-SA**.
