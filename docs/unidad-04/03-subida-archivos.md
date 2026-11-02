# 3. Subida de archivos

Hasta ahora nuestros formularios enviaban textos y opciones. Una aplicación también puede necesitar recibir **una imagen de producto, un documento o un archivo adjunto**.

Subir un archivo significa transferir su contenido desde el equipo del usuario al servidor. PHP lo recibe inicialmente en una ubicación temporal; nuestra aplicación debe comprobarlo y decidir si lo conserva.

En este apartado utilizaremos una **imagen de producto**, sin base de datos. Aprenderemos primero las piezas y después las reuniremos en un ejemplo completo.

## 3.1. Preparar el formulario

Un formulario de subida necesita tres elementos:

```html
<form action="subir.php" method="post" enctype="multipart/form-data">
    <label for="imagen">Imagen del producto:</label>
    <input type="file" id="imagen" name="imagen"
           accept="image/jpeg,image/png" required>
    <button type="submit">Subir imagen</button>
</form>
```

| Elemento | Finalidad |
| --- | --- |
| `method="post"` | Envía los datos en el cuerpo de la petición. |
| `enctype="multipart/form-data"` | Permite enviar partes que contienen campos y archivos. |
| `type="file"` | Permite seleccionar un archivo del equipo. |
| `name="imagen"` | Da nombre a la entrada que consultaremos en `$_FILES`. |
| `accept` | Orienta el selector del navegador hacia los tipos indicados. |

**`accept` no valida el archivo en el servidor.** Un cliente puede enviar un archivo diferente. También debemos comprobarlo con PHP.

Si añadimos un campo de texto al mismo formulario, seguiremos recogiéndolo en `$_POST`. El archivo se consulta en **`$_FILES`**.

!!! important "El usuario elige el archivo"
    El navegador no permite rellenar un campo de archivo con una ruta del equipo del usuario para subirlo automáticamente. Si el formulario vuelve a mostrarse por un error, habrá que seleccionar el archivo otra vez. No intentaremos conservarlo mediante `value`.

## 3.2. Qué contiene $_FILES

Para el campo anterior, PHP proporciona información en `$_FILES['imagen']`:

| Clave | Qué contiene | Ejemplo orientativo |
| --- | --- | --- |
| `name` | Nombre original comunicado por el cliente. | `teclado.png` |
| `type` | Tipo MIME comunicado por el cliente. | `image/png` |
| `tmp_name` | Ruta temporal del archivo recibido en el servidor. | `/tmp/phpAb12Cd` |
| `error` | Código del resultado de la recepción. | `UPLOAD_ERR_OK`, cuyo valor es `0`. |
| `size` | Tamaño recibido, expresado en bytes. | `153600` |

Un **tipo MIME** identifica la naturaleza del contenido. Por ejemplo, `image/jpeg` representa una imagen JPEG y `application/pdf` un PDF.

`name` y `type` proceden del cliente: **no utilizaremos el nombre original como destino ni aceptaremos un archivo únicamente porque `type` diga que es una imagen**.

`tmp_name` es una ruta del servidor, no la ruta original del equipo del alumno. El archivo temporal normalmente se elimina al terminar la petición si no lo movemos.

Para observar estos datos durante el desarrollo:

```php
if (($_SERVER['REQUEST_METHOD'] ?? '') === 'POST') {
    echo '<pre>';
    echo htmlspecialchars(
        print_r($_FILES, true),
        ENT_QUOTES | ENT_SUBSTITUTE,
        'UTF-8'
    );
    echo '</pre>';
}
```

`print_r($_FILES, true)` devuelve una representación del array como cadena. Es una herramienta de observación: retiraremos esta salida al terminar las pruebas.

## 3.3. Comprobar si la recepción ha funcionado

Antes de analizar o mover el archivo, consultamos **`error`**:

| Constante | Qué ha ocurrido |
| --- | --- |
| `UPLOAD_ERR_OK` | PHP ha recibido el archivo sin errores de subida. |
| `UPLOAD_ERR_INI_SIZE` | Supera el límite `upload_max_filesize` de PHP. |
| `UPLOAD_ERR_FORM_SIZE` | Supera el límite indicado mediante `MAX_FILE_SIZE` en el formulario. |
| `UPLOAD_ERR_PARTIAL` | Solo se ha recibido una parte. |
| `UPLOAD_ERR_NO_FILE` | No se ha seleccionado un archivo. |
| `UPLOAD_ERR_NO_TMP_DIR` | Falta la carpeta temporal del servidor. |
| `UPLOAD_ERR_CANT_WRITE` | PHP no ha podido escribir el archivo en disco. |
| `UPLOAD_ERR_EXTENSION` | Una extensión de PHP ha detenido la subida. |

No hay que memorizar sus valores numéricos: utilizaremos las constantes por su nombre.

```php
$archivo = $_FILES['imagen'] ?? null;

if (!is_array($archivo) || !isset($archivo['error'])
    || !is_int($archivo['error'])) {
    // No hay una estructura válida para una subida individual.
} elseif ($archivo['error'] === UPLOAD_ERR_NO_FILE) {
    // Pedir que se seleccione un archivo.
} elseif ($archivo['error'] !== UPLOAD_ERR_OK) {
    // La recepción no ha terminado correctamente.
}
```

Comprobamos la estructura porque un envío puede tener una forma inesperada. Por ejemplo, `name="imagen[]"` genera arrays dentro de las claves: es una subida múltiple y no coincide con el formato individual de nuestro ejemplo.

## 3.4. Limitar el tamaño

La aplicación tendrá su propio límite. Por ejemplo, **2 MiB**:

```php
$maximo = 2 * 1024 * 1024; // 2 097 152 bytes.
```

```php
if ($archivo['size'] <= 0 || $archivo['size'] > $maximo) {
    // Rechazar archivos vacíos o demasiado grandes.
}
```

Un MiB equivale a 1024 × 1024 bytes. Indicaremos expresamente esta unidad en los ejemplos para que el límite sea preciso.

También interviene la configuración del servidor:

| Directiva PHP | Finalidad |
| --- | --- |
| `file_uploads` | Habilita la recepción de archivos. |
| `upload_max_filesize` | Limita el tamaño de cada archivo. |
| `post_max_size` | Limita el cuerpo completo del POST, incluidos archivos, campos y estructura del envío. |

Para permitir imágenes de hasta 2 MiB, una configuración posible es:

```ini
file_uploads = On
upload_max_filesize = 2M
post_max_size = 3M
```

`post_max_size` debe dejar margen para el resto del formulario. Si se supera, **`$_POST` y `$_FILES` pueden llegar vacíos**, en lugar de recibir un código de error dentro del archivo. Nuestro ejemplo mostrará un mensaje general cuando no encuentre una estructura válida.

Podemos consultar los límites activos con `ini_get('upload_max_filesize')` e `ini_get('post_max_size')`. La configuración debe aplicarse al PHP que atiende Apache dentro de Docker; cambiar la del equipo anfitrión no modifica automáticamente la del contenedor.

Un campo oculto `MAX_FILE_SIZE` no sustituye el límite de PHP ni la comprobación de nuestra aplicación, porque el cliente puede modificarlo.

## 3.5. Comprobar el tipo del contenido

Cambiar el nombre de `documento.txt` a `foto.jpg` **no convierte el archivo en una imagen**. Tampoco es suficiente consultar `$_FILES['imagen']['type']`.

Utilizaremos **Fileinfo** para detectar el tipo a partir del contenido temporal:

```php
$finfo = new finfo(FILEINFO_MIME_TYPE);
$tipo = $finfo->file($archivo['tmp_name']);
```

Aquí conectamos con la unidad 3: creamos un objeto `finfo` mediante `new` y llamamos a su método `file` utilizando `->`. `FILEINFO_MIME_TYPE` indica la información que queremos obtener. Necesitamos la extensión **fileinfo** habilitada.

Definimos los tipos que admite la aplicación y la extensión que asignaremos:

```php
$permitidos = [
    'image/jpeg' => 'jpg',
    'image/png' => 'png',
];

if ($tipo === false || !array_key_exists($tipo, $permitidos)) {
    // Rechazar el archivo.
}
```

La lista la decide el servidor. En este ejemplo no aceptaremos SVG ni otros formatos. Fileinfo mejora la comprobación del tipo, pero no certifica que cualquier contenido sea inocuo: por eso combinamos límites, tipos permitidos y almacenamiento fuera de la carpeta pública. En aplicaciones publicadas pueden ser necesarias comprobaciones adicionales y reprocesamiento de imágenes.

## 3.6. Elegir un nombre y mover el archivo

No utilizaremos el nombre original para construir la ruta del destino. Podría repetirse o incluir caracteres problemáticos.

```php
$nombre = bin2hex(random_bytes(16)) . '.' . $permitidos[$tipo];
```

* `random_bytes(16)` genera 16 bytes aleatorios.
* `bin2hex` los representa mediante 32 caracteres hexadecimales, cómodos para un nombre de archivo.
* La extensión se obtiene de nuestra lista, según el tipo detectado.

Por ejemplo, el resultado podría ser `a13f…92bc.png`, con 32 caracteres antes del punto. El nombre aleatorio hace extremadamente improbable una coincidencia; no dependemos del nombre del usuario.

Movemos el temporal con:

```php
$destino = $carpeta . '/' . $nombre;

if (!move_uploaded_file($archivo['tmp_name'], $destino)) {
    // No se ha podido guardar.
}
```

**`move_uploaded_file` comprueba que el origen procede del mecanismo de subida HTTP POST de PHP**, además de moverlo. Devuelve `true` o `false`. Si el destino ya existe, puede sobrescribirlo: por eso generamos nombres nuevos y controlamos el destino.

La carpeta debe existir y el proceso de PHP debe tener permiso para escribir en ella.

## 3.7. Preparar la carpeta en Docker

Guardaremos las imágenes **fuera de la raíz pública del servidor**. Si Apache sirve `/var/www/html`, utilizaremos:

```text
/var/www/almacen-ud04
```

Es una ruta dentro del contenedor. No debe estar publicada mediante alias ni configurada como carpeta ejecutable del servidor web.

En una imagen habitual `php:apache`, el usuario de Apache es `www-data`. Desde la carpeta donde tienes el Compose, puedes preparar el directorio con:

```bash
docker compose exec --user root NOMBRE_SERVICIO mkdir -p /var/www/almacen-ud04
docker compose exec --user root NOMBRE_SERVICIO chown www-data:www-data /var/www/almacen-ud04
docker compose exec --user root NOMBRE_SERVICIO chmod 750 /var/www/almacen-ud04
```

Sustituye **`NOMBRE_SERVICIO`** por el servicio que ejecuta Apache/PHP; puedes consultarlo con `docker compose config --services`. Si tu imagen utiliza otro usuario, adapta el propietario a ese usuario. No necesitamos permisos `777`.

Esta preparación se realiza una vez por contenedor. Para conservar el almacén al eliminar y recrear el contenedor, habrá que montarlo mediante un volumen o una carpeta del anfitrión, según nuestro Compose. Reiniciar y recrear un contenedor no son lo mismo.

!!! note "Guardar no es publicar"
    El archivo guardado en esta carpeta no tiene una URL pública directa. En este primer ejemplo comprobaremos su existencia desde el contenedor. Más adelante podremos servir imágenes mediante una ruta controlada; con Laravel retomaremos almacenamiento y publicación.

## 3.8. Ejemplo completo: subir una imagen de producto

Guarda este archivo como **`subir.php`** en tu carpeta de prácticas. Acepta una imagen JPEG o PNG no vacía, de hasta 2 MiB, y la conserva con un nombre generado en `/var/www/almacen-ud04`.

Separamos el procesamiento en una función que devuelve el nombre guardado. Para comunicar errores utilizamos `RuntimeException`, que hereda de `Exception`: aplicamos `throw`, `try` y `catch`, vistos en la unidad 3.

```php
<?php
declare(strict_types=1);

function e(string $texto): string
{
    return htmlspecialchars($texto, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}

function guardarImagen(array $archivo, string $carpeta): string
{
    if (!isset($archivo['error']) || !is_int($archivo['error'])) {
        throw new RuntimeException('El formato de la subida no es válido.');
    }

    if ($archivo['error'] === UPLOAD_ERR_NO_FILE) {
        throw new RuntimeException('Selecciona una imagen.');
    }

    if ($archivo['error'] === UPLOAD_ERR_INI_SIZE
        || $archivo['error'] === UPLOAD_ERR_FORM_SIZE) {
        throw new RuntimeException('El archivo supera el límite de subida.');
    }

    if ($archivo['error'] !== UPLOAD_ERR_OK) {
        throw new RuntimeException('No se ha recibido el archivo completo. Reinténtalo.');
    }

    // A partir de aquí necesitamos un tamaño entero y una ruta temporal.
    if (!isset($archivo['size'], $archivo['tmp_name'])
        || !is_int($archivo['size']) || !is_string($archivo['tmp_name'])) {
        throw new RuntimeException('Los datos del archivo no son válidos.');
    }

    if ($archivo['size'] <= 0 || $archivo['size'] > 2 * 1024 * 1024) {
        throw new RuntimeException('La imagen debe ocupar más de 0 bytes y hasta 2 MiB.');
    }

    if (!is_uploaded_file($archivo['tmp_name'])) {
        throw new RuntimeException('No se ha recibido una subida válida.');
    }

    $permitidos = ['image/jpeg' => 'jpg', 'image/png' => 'png'];
    $finfo = new finfo(FILEINFO_MIME_TYPE);
    $tipo = $finfo->file($archivo['tmp_name']);

    if ($tipo === false || !array_key_exists($tipo, $permitidos)) {
        throw new RuntimeException('Solo se admiten imágenes JPEG o PNG.');
    }

    if (!is_dir($carpeta) || !is_writable($carpeta)) {
        throw new RuntimeException('El almacén no está preparado para guardar archivos.');
    }

    $nombre = bin2hex(random_bytes(16)) . '.' . $permitidos[$tipo];
    $destino = $carpeta . '/' . $nombre;

    if (!move_uploaded_file($archivo['tmp_name'], $destino)) {
        throw new RuntimeException('No se ha podido guardar la imagen.');
    }

    return $nombre;
}

$error = '';
$nombreGuardado = '';

if (($_SERVER['REQUEST_METHOD'] ?? '') === 'POST') {
    try {
        $archivo = $_FILES['imagen'] ?? null;
        if (!is_array($archivo)) {
            throw new RuntimeException(
                'No se ha recibido el archivo. Revisa el formulario y los límites de PHP.'
            );
        }
        $nombreGuardado = guardarImagen($archivo, '/var/www/almacen-ud04');
    } catch (RuntimeException $excepcion) {
        $error = $excepcion->getMessage();
    }
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Subir imagen de producto</title>
</head>
<body>
    <h1>Subir imagen de producto</h1>
    <p>JPEG o PNG, hasta 2 MiB.</p>

    <?php if ($error !== ''): ?>
        <p role="alert"><?= e($error) ?></p>
    <?php endif; ?>

    <?php if ($nombreGuardado !== ''): ?>
        <p role="status">Imagen guardada como: <?= e($nombreGuardado) ?></p>
    <?php endif; ?>

    <form action="subir.php" method="post" enctype="multipart/form-data">
        <label for="imagen">Imagen:</label>
        <input type="file" id="imagen" name="imagen"
               accept="image/jpeg,image/png" required>
        <button type="submit">Subir imagen</button>
    </form>
</body>
</html>
```

### Cómo leer el ejemplo

1. En GET mostramos el formulario sin mensajes.
2. En POST recogemos la entrada de `$_FILES`.
3. La función comprueba recepción, tamaño, origen temporal y tipo del contenido.
4. Genera un nombre, mueve el archivo y devuelve ese nombre.
5. Si se lanza una excepción, `catch` recoge el mensaje y volvemos a mostrar el formulario.

`is_uploaded_file` comprueba el temporal antes de analizarlo; `move_uploaded_file` vuelve a verificar su origen al moverlo. `is_dir` y `is_writable` comprueban la carpeta. Son funciones de comprobación, no crean el almacén.

La selección del archivo se vacía en la respuesta: es el comportamiento previsto del campo `file`. Para repetir una subida se debe seleccionar otra vez.

Este ejemplo de aula guarda realmente archivos. **Actualizar y reenviar el POST puede crear otra copia**, como ocurre con otras operaciones. Tras comprender la subida podemos aplicar PRG; para mantener el mensaje de éxito después de redirigir utilizaremos sesiones en el bloque correspondiente. No publicaremos este formulario sin control de acceso y protección CSRF.

## 3.9. Comprobar el resultado

Desde el terminal, podemos listar el almacén:

```bash
docker compose exec NOMBRE_SERVICIO ls -lh /var/www/almacen-ud04
```

| Prueba | Resultado esperado |
| --- | --- |
| JPEG o PNG válido de menos de 2 MiB. | Se guarda con un nombre generado y extensión adecuada. |
| Ningún archivo, desactivando temporalmente `required`. | Mensaje para seleccionar imagen. |
| PDF o texto, aunque se renombre a `.jpg`. | Rechazo por tipo de contenido. |
| Archivo vacío. | Rechazo por tamaño. |
| Archivo que supera el límite de la aplicación o de PHP. | Mensaje de error; no se mueve al almacén. |
| Dos imágenes con el mismo nombre original. | Se guardan con nombres nuevos diferentes. |
| Carpeta de destino inexistente o sin permisos de escritura. | Mensaje sobre la preparación del almacén. |

Para comprobar específicamente el límite de la aplicación, el archivo debe llegar a PHP: podemos elevar temporalmente `upload_max_filesize` y `post_max_size` por encima de ese archivo y mantener los 2 MiB en el código. Después restableceremos la configuración.

Si Fileinfo no reconoce como imagen un archivo, comprobaremos que sea realmente JPEG o PNG. Una extensión o un nombre no determinan el contenido.

## 3.10. Qué debemos recordar

* El formulario utiliza POST, `multipart/form-data` y un control `file` con `name`.
* Los archivos se consultan en `$_FILES`; los campos de texto, en `$_POST`.
* Comprobamos primero el error de recepción y después tamaño y tipo.
* `accept`, el nombre y el tipo comunicado por el navegador no sustituyen la validación del servidor.
* Generamos el nombre y utilizamos `move_uploaded_file` para conservar el temporal.
* El almacén se sitúa fuera de la carpeta pública y necesita permisos adecuados.

## Practica lo aprendido

Realiza el **ejercicio 5 del bloque 3: Subida de archivos** de la [relación de actividades](actividades.md#bloque-3-subida-de-archivos). Combinarás un campo de texto con la recepción, validación y almacenamiento de una memoria en PDF.

Comenzaremos el ejercicio en clase y completarás en casa lo pendiente. Revisaremos la solución por separado y mantendremos la del profesor separada del enunciado.

## Referencias

* [Aitor Medrano y Luis Alemañ: Programación web](https://aitor-medrano.github.io/dwes2122/04web.html), apartado de subida de archivos.
* [Manual de PHP: subida mediante POST](https://www.php.net/manual/es/features.file-upload.post-method.php).
* [Manual de PHP: errores de subida](https://www.php.net/manual/es/features.file-upload.errors.php).
* [Manual de PHP: Fileinfo](https://www.php.net/manual/es/function.finfo-file.php).
* [Manual de PHP: move_uploaded_file](https://www.php.net/manual/es/function.move-uploaded-file.php).

## Autoría y licencia

Material elaborado por **Joaquín García** para DWES de 2.º DAW del **IES Portada Alta**. Adapta y amplía los materiales de **Aitor Medrano** y **Luis Alemañ**, con ejemplos apoyados en el manual de PHP.

Licencia **Creative Commons Reconocimiento-NoComercial-CompartirIgual (CC BY-NC-SA)**.
