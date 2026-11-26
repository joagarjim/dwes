# Proyecto final · Unidad 4
# Gestor de incidencias del aula

## 1. Objetivo

Construye una aplicación PHP para registrar incidencias de los equipos del aula. Reunirás los formularios y su validación, la subida de archivos, las cookies, las sesiones y la autenticación trabajados en esta unidad.

No utilices JavaScript, base de datos ni frameworks. Guarda las incidencias en `$_SESSION` y las imágenes en una carpeta `uploads/` del servidor. Los datos pertenecen a cada sesión: no es un listado compartido entre todos los usuarios.

## 2. Archivos

| Archivo | Función |
| --- | --- |
| `login.php` | Mostrar y procesar el acceso. |
| `incidencias.php` | Consultar las incidencias y mostrar las opciones disponibles. |
| `nueva.php` | Mostrar, validar y procesar el formulario de incidencias. |
| `preferencias.php` | Guardar o restablecer el tamaño de lectura. |
| `vaciar.php` | Vaciar las incidencias de la sesión. |
| `logout.php` | Cerrar la sesión. |
| `uploads/` | Almacenar las imágenes. |

Puedes añadir un archivo CSS. No es necesario utilizar clases, funciones auxiliares ni una arquitectura adicional.

## 3. Acceso y protección

Utiliza estas credenciales de prueba:

```text
Usuario: aula
Contraseña: aula123
```

Genera el hash de la contraseña una vez y guárdalo en `login.php` junto al usuario, como hemos trabajado en el apartado 6. No generes un hash nuevo en cada intento de acceso.

El formulario enviará las credenciales mediante POST. Comprueba el usuario y utiliza `password_verify()` para comprobar la contraseña.

Si son correctos, renueva el identificador de sesión, guarda el usuario autenticado en `$_SESSION`, redirige a `incidencias.php` con estado 303 y termina la ejecución con `exit`. Si son incorrectos, muestra «Usuario o contraseña incorrectos».

Utiliza el nombre de sesión `UD04INCIDENCIAS` en todos los archivos. Todas las páginas y acciones privadas deben recuperar la sesión y comprobar el acceso antes de enviar HTML o modificar datos. Sin acceso, redirige a `login.php` y termina.

## 4. Formulario de nueva incidencia

| Campo | Tipo | Validación en PHP |
| --- | --- | --- |
| Equipo | Texto | Entre 2 y 30 caracteres, sin contar espacios iniciales y finales. |
| Categoría | Desplegable | Solo `hardware`, `software` o `red`. |
| Descripción | Área de texto | Entre 10 y 300 caracteres, sin contar espacios iniciales y finales. |
| Impide trabajar | Casilla | Opcional. Guarda verdadero o falso según esté marcada. |
| Imagen | Archivo | Opcional. JPEG o PNG y máximo 50 MiB. |

Aplica el procedimiento de validación del apartado 2: recoge los campos, prepara un array de errores y muestra cada mensaje junto al campo correspondiente. Conserva los valores introducidos y el estado de la casilla cuando haya errores. El archivo seleccionado debe elegirse de nuevo.

Utiliza POST y `enctype="multipart/form-data"`. Los atributos HTML ayudan al usuario, pero las comprobaciones también deben hacerse en PHP.

### Subida de la imagen: solo lo básico

- Permite registrar una incidencia sin imagen.
- Si se selecciona una, comprueba que la subida haya terminado correctamente.
- Rechaza archivos superiores a **50 × 1024 × 1024 bytes (50 MiB)**.
- Comprueba el tipo real del archivo: JPEG o PNG. No basta con la extensión o el atributo `accept`.
- Genera un nombre propio, utiliza la extensión correspondiente al tipo comprobado y guarda el archivo con `move_uploaded_file()` en `uploads/`.
- Si el archivo no se puede guardar, muestra el error y no registres la incidencia.

No se pide redimensionar imágenes, crear miniaturas ni añadir más formatos. Mueve el archivo solo cuando no haya errores en los campos ni en la imagen.

Cuando todo sea válido, añade a `$_SESSION['incidencias']` un array con el equipo, la categoría, la descripción, la casilla y la ruta de la imagen (vacía si no existe). Redirige al listado con estado 303 y termina la ejecución.

## 5. Listado

En `incidencias.php`, muestra el usuario autenticado, el número de incidencias y los datos de cada una. Muestra también la imagen si existe, con un tamaño adecuado.

Si no hay datos, muestra «Todavía no hay incidencias registradas».

Añade un enlace a `nueva.php` y los formularios de preferencias, vaciado y cierre de sesión. Utiliza `htmlspecialchars()` al insertar texto del usuario en el HTML; no necesitas una función auxiliar.

## 6. Preferencia de lectura mediante cookie

Permite elegir `normal` (1rem), `grande` (1.25rem) o `muy-grande` (1.5rem). Envía la elección por POST a `preferencias.php` con la acción `guardar`.

Guarda únicamente valores permitidos en la cookie `ud04_incidencias_tamano`, con duración de 30 días, ruta `/`, `httponly=true`, `samesite=Lax` y `secure=false` en nuestro HTTP local.

Después de procesar el formulario, redirige al listado con estado 303. Una acción o un tamaño desconocidos no deben modificar la cookie.

En cada visita al listado, lee y valida la cookie. Si falta o no es válida, aplica Normal sin crear una cookie automáticamente. Aplica una clase CSS del catálogo permitido y conserva la selección.

Añade un botón «Restablecer tamaño», con acción `restablecer`, que solicite borrar la cookie con una caducidad pasada y el mismo ámbito.

## 7. Vaciar y cerrar

Ambas acciones utilizarán POST:

| Acción | Resultado |
| --- | --- |
| Vaciar incidencias | Elimina solo `$_SESSION['incidencias']`. Mantiene el acceso y vuelve al listado mediante 303. |
| Cerrar sesión | Vacía `$_SESSION`, destruye la sesión y solicita borrar su cookie. Vuelve al acceso mediante 303. |

La cookie del tamaño de lectura se conserva al cerrar sesión. Los archivos de `uploads/` también pueden permanecer: su eliminación no forma parte de este proyecto.

## 8. Solicitudes y respuestas HTTP

Activa Keep log / Conservar registro en Red y documenta estos recorridos:

1. **Acceso correcto:** GET del formulario, POST de credenciales, respuesta 303 con actualización del identificador y GET de la página privada.
2. **Guardar una incidencia:** POST, respuesta 303 y GET del listado con respuesta 200. Explica por qué recargar no duplica la incidencia.
3. **Guardar el tamaño:** identifica `Set-Cookie` en la respuesta y `Cookie` en la solicitud posterior. Comprueba la cookie en Aplicación.

Para cada recorrido, prepara una tabla con solicitud del navegador, procesamiento de PHP y respuesta del servidor. Acompáñala de capturas de Red. No hace falta copiar todas las cabeceras: incluye las relevantes.

## 9. Pruebas

Comprueba al menos:

- Acceso directo a una página privada sin autenticarse y credenciales incorrectas.
- Incidencia válida sin imagen y con imagen.
- Campos incorrectos: aparecen mensajes y se conserva lo introducido.
- Imagen de tipo no permitido y archivo superior a 50 MiB: no se registra la incidencia.
- Recarga del listado después de guardar: no duplica datos.
- Cookie manipulada: utiliza Normal. Restablecer: vuelve a Normal.
- Vaciar: elimina las incidencias y mantiene el acceso.
- Cerrar sesión: una nueva solicitud privada redirige al acceso.
- Volver a entrar: se conserva la preferencia de tamaño, pero no las incidencias anteriores.
- Otra pestaña del mismo perfil: comparte la sesión. Incógnito: sesión independiente.

Para probar archivos grandes, los límites de PHP deben permitir recibirlos antes de que la aplicación los compruebe. Configura `upload_max_filesize=100M` y `post_max_size=120M` en el contenedor. El límite de la aplicación sigue siendo 50 MiB. Utiliza el procedimiento de configuración y permisos trabajado con Docker; `uploads/` debe permitir escribir al usuario de Apache.

## 10. Entrega

Entrega un ZIP con el código, una carpeta `uploads/` sin imágenes de prueba y un `README.md` con la dirección local, las credenciales y las instrucciones de ejecución. Adjunta el documento de trazabilidad HTTP y sus capturas.

## 11. Evaluación

| Aspecto | Puntos |
| --- | ---: |
| Formulario, validación y conservación de valores | 2 |
| Subida y presentación de la imagen | 1,5 |
| Incidencias en sesión y vaciado | 1,5 |
| Cookie de tamaño y restablecimiento | 1,5 |
| Autenticación, protección y cierre | 2 |
| Trazabilidad HTTP y pruebas | 1,5 |
| **Total** | **10** |

Se valora el funcionamiento y la comprensión de lo que ocurre en cada petición. Es suficiente una presentación sencilla y legible.
