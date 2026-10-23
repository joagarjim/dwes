# Unidad 3. Programación orientada a objetos en PHP

![Clases y objetos: productos informáticos y sus representaciones como plantillas](images/poo-unidad-03.png)

## Presentación

En la unidad anterior aprendimos a trabajar con variables, formularios, condiciones, bucles, arrays y funciones. Con esas herramientas ya podemos resolver problemas y generar páginas dinámicas. Ahora daremos un paso más: aprenderemos a **organizar los datos y las operaciones relacionadas mediante clases y objetos**.

La **programación orientada a objetos (POO)** nos ayudará a representar elementos de una aplicación, como productos, categorías o usuarios, y a definir qué puede hacer cada uno. No sustituye lo que ya sabemos: dentro de los métodos seguiremos utilizando condiciones, bucles, arrays y funciones del lenguaje.

Utilizaremos una tienda informática como hilo conductor. Partiremos de un producto representado mediante un array y construiremos progresivamente clases que permitan consultar datos, realizar cálculos y controlar operaciones. También resolveremos ejercicios con otros contextos para aprender a aplicar estas ideas a problemas diferentes.

Esta base nos servirá para organizar las próximas aplicaciones PHP y comprender el código de Laravel: sus controladores, modelos y otros componentes utilizan clases y objetos. Seguiremos trabajando con nuestro entorno Docker y PHP; en esta unidad no necesitaremos una base de datos.

!!! question "Pregunta inicial"
    En una tienda tenemos productos con nombre, precio y stock. También necesitamos calcular importes y reponer existencias. ¿Cómo podríamos reunir los datos y las operaciones de cada producto para utilizarlos sin repetir su estructura en todo el programa?

## Objetivos de aprendizaje

Al terminar la unidad serás capaz de:

* Diferenciar una clase de un objeto y reconocer sus propiedades y métodos.
* Crear y utilizar varios objetos, comprendiendo qué representa `$this` en cada llamada.
* Inicializar objetos mediante constructores y utilizar tipos en propiedades, parámetros y resultados.
* Aplicar encapsulación para controlar la consulta y la modificación de los datos de un objeto.
* Relacionar objetos mediante composición y trabajar con arrays de objetos.
* Distinguir los datos y las operaciones de cada instancia de los elementos compartidos de la clase.
* Reutilizar y especializar comportamiento mediante herencia, sobrescritura y polimorfismo.
* Comprender la finalidad de las clases abstractas y las interfaces y utilizar contratos sencillos.
* Organizar las clases en archivos y reconocer la función de los espacios de nombres y la autocarga.
* Comunicar operaciones inválidas mediante excepciones, comprobar el comportamiento de las clases y documentar su uso.
* Separar las operaciones de las clases del código que presenta los resultados en una página web.

## ¿Qué vamos a aprender?

Seguiremos una secuencia progresiva: primero crearemos objetos sencillos; después aprenderemos a inicializarlos, proteger sus datos y relacionarlos; finalmente organizaremos el código y comprobaremos su funcionamiento.

### 1. Clases y objetos

* Del array al objeto: datos y operaciones relacionados.
* Clases, propiedades, métodos e instancias.
* Creación con `new`, acceso con `->` y significado de `$this`.
* Estado de los objetos, valores de retorno y organización inicial en archivos.

[Comenzar el apartado 1](01-clases-objetos.md){ .md-button .md-button--primary }

### 2. Constructor y encapsulación

* Inicialización de los objetos con `__construct`.
* Visibilidad de propiedades y métodos.
* Consulta y modificación controlada de los datos.
* Getters, setters y métodos que expresan operaciones concretas.
* Validación y promoción de propiedades del constructor.

### 3. Relaciones entre objetos, constantes y miembros estáticos

* Objetos que contienen o utilizan otros objetos.
* Relación entre productos y categorías.
* Arrays de objetos.
* Constantes y elementos estáticos: diferencia entre instancia y clase.

### 4. Herencia y polimorfismo

* Especialización de clases con `extends`.
* Uso de `protected` y llamadas mediante `parent`.
* Constructores en las clases hijas y sobrescritura de métodos.
* Tratamiento de objetos de distintas clases mediante un tipo común.
* Finalidad de las clases abstractas y del modificador `final`.

### 5. Interfaces y organización del código

* Contratos con `interface` e `implements`.
* Diferencias entre una interfaz y una clase abstracta.
* Una clase por archivo y carga con `require_once`.
* Espacios de nombres con `namespace` y uso de nombres con `use`.
* Introducción a la finalidad de la autocarga.

### 6. Excepciones, pruebas y documentación

* Comunicación de situaciones inválidas con `throw`.
* Tratamiento de excepciones mediante `try` y `catch`.
* Comprobación de casos normales, valores límite y errores.
* Documentación de las clases, sus métodos y sus responsabilidades.

## Cómo vamos a trabajar

En clase estudiaremos los conceptos con ejemplos pequeños y comentados. Analizaremos qué hace el código, qué resultados produce y cómo cambia el estado de los objetos. Después pondremos en común soluciones de los ejercicios, resolveremos dudas y revisaremos errores frecuentes.

Los **ejercicios se trabajarán en casa**, organizados en bloques que siguen los apartados de la unidad. Algunos serán independientes y otros ampliarán una solución anterior para incorporar un concepto nuevo. Se entregarán en Moodle según las indicaciones de clase.

Los apuntes servirán como manual de consulta: incluirán explicaciones de la sintaxis, ejemplos, resultados esperados y preguntas para comprobar la comprensión. Al programar, será importante poder explicar por qué utilizamos una clase, qué responsabilidad tiene y cómo hemos comprobado que funciona.

!!! tip "Para aprender a trabajar con objetos"
    Antes de ejecutar un ejemplo, intenta anticipar el resultado. Después cambia un dato, crea otro objeto o llama de nuevo a un método. Comprueba qué cambia y explica el motivo: esa comparación te ayudará a comprender el código.

## Actividad final

Integraremos lo aprendido en una **pequeña aplicación de gestión de un catálogo de productos en memoria**, sin base de datos. Organizaremos la información mediante clases y objetos y mostraremos sus resultados en una página web.

La actividad permitirá aplicar encapsulación, composición, herencia e interfaces, además de comprobar y documentar las operaciones. Su desarrollo se realizará principalmente en casa y se acompañará de una comprobación individual del aprendizaje. El enunciado detallado se publicará durante la unidad.

## Actividades de la unidad

Consulta la [relación de actividades](actividades.md) para acceder a los enunciados por bloques y a las indicaciones de trabajo. Este documento se irá completando a medida que avancemos en los apartados.

## Autoría y licencia

Material elaborado por **Joaquín García** para el módulo de **Desarrollo Web en Entorno Servidor** de 2.º DAW del **IES Portada Alta**.

Este material adapta y amplía contenidos de *Desarrollo Web en Entorno Servidor*, elaborados por **Aitor Medrano** y **Luis Alemañ** y publicados bajo licencia **Creative Commons Reconocimiento-NoComercial-CompartirIgual**.

Esta adaptación se distribuye bajo licencia **Creative Commons Reconocimiento-NoComercial-CompartirIgual (CC BY-NC-SA)**.
