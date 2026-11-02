# Unidad 4. Desarrollo de aplicaciones web con PHP

![Aplicación de una tienda informática con formularios, imágenes de productos, preferencias y acceso de usuarios](images/web-unidad-04.png)

## Presentación

En la unidad 2 aprendimos a utilizar PHP para recoger datos de formularios, tomar decisiones, recorrer arrays y organizar operaciones mediante funciones. En la unidad 3 incorporamos clases y objetos para reunir datos y comportamientos relacionados. Ahora utilizaremos esa base para construir **aplicaciones web que permitan realizar una tarea a través de varias páginas**.

Una tienda informática necesita algo más que mostrar un catálogo: debe comprobar los datos de sus formularios, permitir subir imágenes de productos, recordar preferencias y mantener un carrito mientras el usuario navega. También necesita identificar a quien accede y proteger determinadas páginas.

Para comprender cómo se consigue, estudiaremos el intercambio de **peticiones y respuestas HTTP** entre el navegador y el servidor. Cada petición se procesa por separado: las variables de un script no se conservan automáticamente para la siguiente visita. Aprenderemos a utilizar cookies y sesiones para mantener la información que necesita la aplicación.

Seguiremos trabajando con **PHP y nuestro entorno Docker**. Utilizaremos datos de prueba definidos en el código y sesiones para conservar información temporal. En la siguiente unidad incorporaremos **PDO y MySQL** para almacenar y recuperar datos de forma persistente; más adelante retomaremos estas funcionalidades con Laravel.

!!! question "Pregunta inicial"
    Añades un teclado al carrito de una tienda y después visitas la página de un monitor. ¿Cómo puede recordar la aplicación qué habías añadido si cada página llega al servidor en una petición diferente?

## Objetivos de aprendizaje

Al terminar la unidad serás capaz de:

* Explicar el recorrido de una petición y una respuesta e identificar el método utilizado.
* Consultar información de una petición mediante `$_SERVER` y utilizar redirecciones.
* Procesar formularios con GET y POST, seleccionando el método adecuado para cada operación.
* Validar los datos en el servidor, mostrar errores comprensibles y conservar los valores introducidos.
* Diferenciar la validación de datos del escape de la salida HTML.
* Recibir archivos mediante `$_FILES`, comprobar errores, tamaño y tipo real, y almacenarlos con un nombre seguro.
* Crear, consultar y eliminar cookies para recordar preferencias del usuario.
* Mantener información entre páginas mediante sesiones y distinguirla de la almacenada en cookies.
* Implementar un inicio y un cierre de sesión con usuarios de prueba, contraseñas con hash y páginas protegidas.
* Organizar el procesamiento de datos y la presentación sin concentrar toda la aplicación en un único archivo.
* Comprobar el funcionamiento de la aplicación con datos correctos, valores límite e intentos de acceso no autorizado.

## ¿Qué vamos a aprender?

Avanzaremos desde el funcionamiento de una petición hasta una aplicación que combine formularios, archivos, preferencias y acceso de usuarios. Cada apartado tendrá explicaciones completas y ejemplos comentados para poder consultar los apuntes también fuera del aula.

### 1. Peticiones y respuestas HTTP en PHP

* Navegador y servidor: qué se envía y qué se recibe.
* URL, parámetros y métodos GET y POST.
* Información de la petición con `$_SERVER`.
* Cabeceras, redirecciones y ejecución de PHP antes de enviar la salida.
* Redirección después de procesar un formulario para evitar repetir un envío al actualizar.

### 2. Formularios y validación

* Recuperación de campos simples, selecciones y grupos de valores.
* Diferencias entre la primera visita y el envío del formulario.
* Validación en el navegador y comprobación obligatoria en el servidor.
* Campos requeridos, tipos, rangos y valores permitidos.
* Mensajes de error y conservación de los datos introducidos.
* Escape de los valores al mostrarlos en HTML con `htmlspecialchars`.

### 3. Subida de archivos

* Formularios con `multipart/form-data` y recepción mediante `$_FILES`.
* Códigos de error y límites de tamaño.
* Comprobación del tipo real del archivo: no confiar solo en su extensión ni en el tipo enviado por el navegador.
* Generación de nombres y almacenamiento mediante `move_uploaded_file`.
* Ejemplo guiado: subida de la imagen de un producto.

### 4. Cookies

* Almacenamiento de información en el navegador y envío en peticiones posteriores.
* Creación con `setcookie` y lectura mediante `$_COOKIE`.
* Caducidad, ruta y eliminación.
* Opciones de seguridad y diferencias entre el entorno local y HTTPS.
* Preferencias de visualización de la tienda.

### 5. Sesiones

* Información en el servidor e identificador de sesión.
* Inicio y recuperación con `session_start`.
* Consulta, modificación y eliminación de datos en `$_SESSION`.
* Diferencias y relación entre cookies y sesiones.
* Carrito de productos y mensajes que se muestran una sola vez.
* Introducción a la protección de formularios mediante tokens CSRF.

### 6. Autenticación básica

* Identificación del usuario y diferencia entre autenticación y autorización.
* Usuarios de prueba sin base de datos.
* Contraseñas con hash: `password_hash` y `password_verify`.
* Inicio de sesión y renovación del identificador.
* Comprobación del acceso en las páginas protegidas.
* Cierre de sesión y retirada de la información de autenticación.

### 7. Actividad final integradora

Reuniremos los contenidos de la unidad en una pequeña aplicación de tienda informática. El enunciado concretará las funcionalidades, la organización de archivos, las pruebas y los criterios de valoración.

## Cómo vamos a trabajar

En clase explicaremos los conceptos, analizaremos ejemplos guiados y resolveremos las dudas de los ejercicios. Antes de pasar a un nuevo bloque revisaremos las dificultades más frecuentes del anterior.

El trabajo autónomo será parte del aprendizaje: **completaremos en casa los ejercicios de consolidación y avanzaremos en la actividad final**. Los enunciados serán progresivos y especificarán los datos de entrada, el comportamiento y el resultado esperado. Las soluciones se facilitarán de forma separada según las indicaciones de clase.

Los apuntes constituirán un recurso completo de consulta. En ellos explicaremos qué hace cada instrucción nueva, cuándo se utiliza y qué errores debemos evitar. La selección de ejemplos que trabajemos presencialmente nos permitirá ajustar el ritmo sin reducir el contenido de referencia.

!!! tip "Observa lo que ocurre entre peticiones"
    No compruebes únicamente el resultado que aparece en pantalla. Observa también la URL y las herramientas de desarrollo del navegador: la pestaña Red permite inspeccionar peticiones y respuestas, y el apartado de almacenamiento permite consultar las cookies. Repite una operación, actualiza la página y abre otra pestaña para comprobar qué información se mantiene y por qué.

## Actividad final

El proyecto propuesto será **«Tienda informática: formularios, carrito y acceso de usuarios»**. Permitirá practicar la validación de formularios, la subida de una imagen, las preferencias mediante cookies, el mantenimiento de un carrito con sesiones y el acceso a páginas protegidas.

Será una actividad individual, desarrollada principalmente en casa y con seguimiento y resolución de dudas en el aula. Se utilizarán datos de prueba y almacenamiento temporal mediante sesiones. En la unidad 5 aprenderemos a trasladar la información que deba conservarse a una base de datos.

El enunciado y la rúbrica se publicarán al desarrollar la actividad. Las condiciones y la fecha de entrega se indicarán en Moodle.

## Relación con los resultados de aprendizaje y criterios de evaluación

Esta unidad trabaja los siguientes resultados de aprendizaje:

**RA3.** Escribe bloques de sentencias embebidos en lenguajes de marcas, seleccionando y utilizando las estructuras de programación.

**RA4.** Desarrolla aplicaciones web embebidas en lenguajes de marcas analizando e incorporando funcionalidades según especificaciones.

Del RA3 retomaremos los criterios relacionados con los formularios y la recuperación de información. Del RA4 trabajaremos el mantenimiento del estado, la autenticación y las herramientas de prueba y depuración.

### Criterios de evaluación del RA3

| Código | Criterio de evaluación |
| --- | --- |
| RA3.e | Se han utilizado formularios web para interactuar con el usuario del navegador web. |
| RA3.f | Se han empleado métodos para recuperar la información introducida en el formulario. |

### Criterios de evaluación del RA4

| Código | Criterio de evaluación |
| --- | --- |
| RA4.a | Se han identificado los mecanismos disponibles para el mantenimiento de la información que concierne a un cliente web concreto y se han señalado sus ventajas. |
| RA4.b | Se han utilizado mecanismos para mantener el estado de las aplicaciones web. |
| RA4.c | Se han utilizado mecanismos para almacenar información en el cliente web y para recuperar su contenido. |
| RA4.d | Se han identificado y caracterizado los mecanismos disponibles para la autentificación de usuarios. |
| RA4.e | Se han escrito aplicaciones que integren mecanismos de autentificación de usuarios. |
| RA4.f | Se han utilizado herramientas y entornos para facilitar la programación, prueba y depuración del código. |

Trabajaremos estos criterios mediante formularios con validación, subida de archivos, preferencias almacenadas en cookies, un carrito mantenido mediante sesiones y un sistema básico de acceso de usuarios. Utilizaremos el navegador y el entorno de desarrollo para comprobar las peticiones, las respuestas y el comportamiento de la aplicación.

Los enunciados de las actividades y pruebas concretarán los criterios de evaluación asociados. La autenticación, la seguridad y la calidad se retomarán y ampliarán en la unidad 7.

## Referencias

* [Aitor Medrano y Luis Alemañ: Programación web](https://aitor-medrano.github.io/dwes2122/04web.html).
* [Manual de PHP: variables predefinidas](https://www.php.net/manual/es/reserved.variables.php).
* [Manual de PHP: subida de archivos](https://www.php.net/manual/es/features.file-upload.php).
* [Manual de PHP: cookies](https://www.php.net/manual/es/features.cookies.php).
* [Manual de PHP: sesiones](https://www.php.net/manual/es/book.session.php).
* [Manual de PHP: funciones de contraseñas](https://www.php.net/manual/es/ref.password.php).

## Autoría y licencia

Material elaborado por **Joaquín García** para el módulo de **Desarrollo Web en Entorno Servidor** de 2.º DAW del **IES Portada Alta**.

Este material adapta y amplía contenidos de *Desarrollo Web en Entorno Servidor*, elaborados por **Aitor Medrano** y **Luis Alemañ** y publicados bajo licencia **Creative Commons Reconocimiento-NoComercial-CompartirIgual**. Los ejemplos se contrastarán con la documentación oficial de PHP al desarrollar cada apartado.

Esta adaptación se distribuye bajo licencia **Creative Commons Reconocimiento-NoComercial-CompartirIgual (CC BY-NC-SA)**.
