# 3. Subida de archivos

Una aplicación puede recibir una memoria en PDF, una fotografía o un archivo adjunto. Para entender cómo lo hace PHP, nos quedaremos primero con esta idea:

**El navegador envía el archivo, PHP lo recibe en una ubicación temporal y nuestro código lo mueve a la carpeta donde queremos conservarlo.**

Vamos a probar ese recorrido con un archivo pequeño. Después añadiremos unas comprobaciones y terminaremos con un ejemplo que admite imágenes JPEG o PNG.

## 3.1. Preparar el lugar donde guardaremos los archivos

Antes de programar, necesitamos una carpeta de destino. En clase PHP se ejecuta dentro del contenedor Docker, por lo que las rutas que utiliza son **rutas del contenedor**, aunque trabajemos desde Windows o WSL.

| Ruta dentro del contenedor | Para qué la utilizamos |
| --- | --- |
| `/var/www/html` | Código de nuestras páginas, servido por Apache. |
| `/var/www/almacen-ud04` | Archivos recibidos que queremos conservar. |

El almacén está fuera de la carpeta pública: guardar un archivo ahí no le asigna una URL para abrirlo desde el navegador. De momento comprobaremos su existencia desde el terminal.

### Revisar el montaje del código

Abre tu `compose.yaml` o `docker-compose.yml`. En el servicio **`apache_php`**, puedes tener uno de estos montajes:

```yaml
volumes:
  - ./src:/var/www/html
```

O bien:

```yaml
volumes:
  - ./:/var/www/html
```

**Conserva el que ya utiliza tu proyecto.** La parte izquierda es la carpeta de tu ordenador, relativa al archivo Compose; la derecha es su destino dentro del contenedor.

* Con `./src`, las páginas se guardan en la carpeta `src` del proyecto.
* Con `./`, las páginas se guardan directamente en la carpeta del proyecto.

En ambos casos, Apache las encuentra en `/var/www/html`. Esta diferencia no cambia la ruta del almacén: seguiremos utilizando `/var/www/almacen-ud04`.

### Tres maneras de preparar el almacén

PHP utilizará **la misma ruta en las tres opciones**:

```php
$carpeta = '/var/www/almacen-ud04';
```

Lo que cambia es dónde se conserva realmente el contenido. Elige **una opción cada vez**. Los siguientes ejemplos son fragmentos para integrar en tu Compose: mantén tu `image` o `build`, los puertos y las demás opciones del proyecto. Conserva también tu montaje del código: si usas `./:/var/www/html`, sustituye con él la línea `./src:/var/www/html` de los ejemplos.

#### Opción 1. Dentro del contenedor, sin montar el almacén

```yaml
services:
  apache_php:
    # Mantén tu image/build, ports y demás configuración.
    volumes:
      - ./src:/var/www/html
```

Solo compartimos el código. Como no hay un montaje para `/var/www/almacen-ud04`, esa carpeta se creará en el sistema de archivos propio del contenedor.

Desde la carpeta de tu Compose, arranca el servicio y prepara el almacén:

```bash
docker compose up -d
docker compose exec --user root apache_php mkdir -p /var/www/almacen-ud04
docker compose exec --user root apache_php chown www-data:www-data /var/www/almacen-ud04
docker compose exec --user root apache_php chmod 750 /var/www/almacen-ud04
```

**Parar no es eliminar.** Puedes parar y arrancar el mismo contenedor con:

```bash
docker compose stop apache_php
docker compose start apache_php
```

Los archivos siguen ahí porque el contenedor sigue siendo el mismo. Puedes consultar su estado con:

```bash
docker compose ps -a
```

En cambio, `docker compose down` elimina los contenedores del proyecto. Si después ejecutas `docker compose up -d`, se crean otros: **las subidas que estaban únicamente dentro del anterior se pierden**. Habría que volver a preparar la carpeta y sus permisos. Una recreación del servicio también puede provocar esa pérdida.

#### Opción 2. Volumen persistente administrado por Docker

```yaml
services:
  apache_php:
    # Mantén tu image/build, ports y demás configuración.
    volumes:
      - ./src:/var/www/html
      - almacen_ud04:/var/www/almacen-ud04

volumes:
  almacen_ud04:
```

La línea `almacen_ud04:/var/www/almacen-ud04` conecta un **volumen con nombre** al almacén del contenedor. La declaración final `volumes`, al mismo nivel que `services`, permite que Docker lo prepare.

Si ya tienes esa declaración final, añade `almacen_ud04:` dentro de ella; no dupliques la clave `volumes` en el mismo nivel.

Con Docker Desktop y el backend WSL2, Docker conserva el volumen en su almacenamiento Linux. **No crea una carpeta `almacen_ud04` junto a tu proyecto en Windows.** Docker administra ese almacenamiento, no el propio contenedor.

Aplica la configuración y prepara los permisos del volumen ya montado:

```bash
docker compose up -d
docker compose exec --user root apache_php mkdir -p /var/www/almacen-ud04
docker compose exec --user root apache_php chown www-data:www-data /var/www/almacen-ud04
docker compose exec --user root apache_php chmod 750 /var/www/almacen-ud04
```

Al eliminar y recrear el contenedor dentro del mismo proyecto Compose, este vuelve a montar el volumen y encuentra los archivos y permisos anteriores. Por ejemplo, un `docker compose down` normal seguido de `docker compose up -d` conserva el almacén.

**`docker compose down -v` elimina también este volumen y sus archivos.** La persistencia no sustituye una copia de seguridad.

#### Opción 3. Carpeta persistente de tu ordenador

```yaml
services:
  apache_php:
    # Mantén tu image/build, ports y demás configuración.
    volumes:
      - ./src:/var/www/html
      - ./almacen-ud04:/var/www/almacen-ud04
```

Este montaje se llama **bind mount**. A la izquierda aparece una carpeta de tu ordenador; a la derecha, la ruta donde PHP la encuentra dentro del contenedor. No necesita una declaración final `volumes: almacen_ud04:` porque no utiliza ese volumen con nombre.

Si el Compose está en `C:\dwes\dwes-profesor`, los archivos se guardan en:

```text
C:\dwes\dwes-profesor\almacen-ud04
```

Si el proyecto está en el sistema de archivos Linux de WSL, la carpeta estará allí. **`./` se refiere a la carpeta del proyecto Compose; no significa siempre Windows.**

Crea primero la carpeta desde el terminal del ordenador. En PowerShell:

```powershell
New-Item -ItemType Directory -Force -Path ./almacen-ud04
```

En un terminal Linux o WSL:

```bash
mkdir -p ./almacen-ud04
```

Después aplica el montaje:

```bash
docker compose up -d
```

PHP sigue necesitando permiso de escritura. Si la carpeta está en un sistema de archivos Linux, puedes preparar propietario y permisos como antes:

```bash
docker compose exec --user root apache_php chown www-data:www-data /var/www/almacen-ud04
docker compose exec --user root apache_php chmod 750 /var/www/almacen-ud04
```

**Si compartes una carpeta de Windows, `chown` y `chmod` pueden no comportarse igual que en Linux.** El acceso depende de los permisos de Windows y de la integración de Docker Desktop. No es necesario que aparezca literalmente `www-data` como propietario: lo decisivo es que PHP pueda escribir. Compruébalo con las órdenes del siguiente apartado. Si falla, revisa que Docker Desktop tenga acceso a esa carpeta y que Windows permita la escritura.

Las subidas quedan visibles en tu ordenador y se conservan al eliminar el contenedor. `docker compose down -v` no borra esta carpeta del ordenador.

Si utilizas `./:/var/www/html`, una carpeta `almacen-ud04` situada dentro del proyecto también puede quedar accesible como `/var/www/html/almacen-ud04`. **En ese caso, sitúa el almacén fuera del proyecto**, por ejemplo en una carpeta hermana, y usa esta línea en lugar de la anterior:

```yaml
- ../almacen-ud04:/var/www/almacen-ud04
```

Crea entonces `../almacen-ud04` en lugar de `./almacen-ud04`. Así evitamos publicar las subidas a través del montaje del código.

### Qué significa www-data:www-data

En nuestro contenedor, **Apache ejecuta PHP con el usuario `www-data`**. Por tanto, cuando PHP intenta guardar una subida, lo hace con los permisos de ese usuario, no con los de tu cuenta de Windows ni con los de root.

Observa esta orden que hemos utilizado para preparar el almacén:

```bash
docker compose exec --user root apache_php chown www-data:www-data /var/www/almacen-ud04
```

La parte inicial ejecuta la orden dentro del servicio `apache_php`, como `root`, para poder cambiar el propietario. Eso no cambia el usuario con el que Apache ejecuta nuestras páginas.

La instrucción que realiza el cambio es:

```bash
chown www-data:www-data /var/www/almacen-ud04
```

| Parte | Significado |
| --- | --- |
| `chown` | Cambia el propietario y, cuando se indica, el grupo de un archivo o carpeta. |
| Primer `www-data` | Usuario que será propietario de la carpeta. |
| `:` | Separa el nombre del usuario del nombre del grupo. |
| Segundo `www-data` | Grupo que será propietario de la carpeta. |
| `/var/www/almacen-ud04` | Carpeta sobre la que se realiza el cambio. |

**El usuario y el grupo se llaman igual, pero son dos conceptos distintos.** Un usuario es una identidad del sistema; un grupo permite reunir usuarios y asignarles permisos comunes. En este caso asignamos la carpeta al usuario de Apache y al grupo del mismo nombre.

Después ejecutamos:

```bash
docker compose exec --user root apache_php chmod 750 /var/www/almacen-ud04
```

`chmod` establece los permisos de la carpeta:

| Cifra | A quién corresponde | Qué permite en esta carpeta |
| --- | --- | --- |
| `7` | Propietario: `www-data`. | Leer, escribir y acceder a la carpeta. |
| `5` | Grupo: `www-data`. | Leer y acceder, sin escribir. |
| `0` | Otros usuarios. | Ningún permiso. |

Para interpretar esas cifras: lectura vale **4**, escritura **2** y acceso **1**. Por eso `7 = 4 + 2 + 1` y `5 = 4 + 1`. En una carpeta, el permiso de acceso permite atravesarla y acceder a sus elementos según los permisos de estos.

Con esta combinación, PHP es el propietario y dispone de escritura y acceso para crear los archivos recibidos. **`chown` decide de quién es la carpeta; `chmod` decide qué puede hacer cada uno con ella.** Dar la propiedad y dar permisos son pasos relacionados, pero distintos.

Esta explicación describe los permisos Linux de las opciones 1 y 2 y de las carpetas Linux de la opción 3. En una carpeta compartida desde Windows se mantiene la necesidad de escritura, pero su gestión puede depender de Windows y Docker Desktop, como hemos indicado antes.

### Permisos y comprobaciones: qué tienen en común

Las tres opciones necesitan una carpeta accesible y escribible por **`www-data`**, el usuario que ejecuta PHP en nuestro entorno.

| Orden utilizada en Linux | Qué hace |
| --- | --- |
| `mkdir -p` | Crea la carpeta si no existe. |
| `chown www-data:www-data` | Asigna la carpeta al usuario y grupo de Apache. |
| `chmod 750` | Permite al propietario leer, escribir y acceder; el grupo puede leer y acceder. No necesitamos `777`. |

En cualquiera de las tres opciones puedes inspeccionar la carpeta:

```bash
docker compose exec apache_php ls -ld /var/www/almacen-ud04
```

Y comprobar el acceso **como el usuario de PHP**, no como root:

```bash
docker compose exec --user www-data apache_php test -d /var/www/almacen-ud04
docker compose exec --user www-data apache_php test -w /var/www/almacen-ud04
```

`test -d` comprueba que existe y es un directorio; `test -w`, que el usuario puede escribir. No imprimen texto: terminar correctamente significa que se cumple la condición. Para obtener un mensaje legible desde PHP puedes ejecutar:

```bash
docker compose exec --user www-data apache_php php -r 'var_dump(is_dir("/var/www/almacen-ud04"), is_writable("/var/www/almacen-ud04"));'
```

Debes obtener dos valores `bool(true)`. Las mismas comprobaciones dentro de una página PHP serían:

```php
$carpeta = '/var/www/almacen-ud04';

if (!is_dir($carpeta)) {
    echo 'La carpeta de destino no existe.';
} elseif (!is_writable($carpeta)) {
    echo 'PHP no tiene permiso para escribir en la carpeta.';
} else {
    echo 'La carpeta está preparada para recibir archivos.';
}
```

**Estas funciones solo comprueban: no crean la carpeta ni conceden permisos.** El resultado de `move_uploaded_file` confirmará después si el guardado se realiza correctamente. La validación de recepción, tamaño y tipo del archivo es la misma en las tres opciones.

### Comparación y elección para las prácticas

| Operación | Dentro del contenedor | Volumen de Docker | Carpeta del ordenador |
| --- | --- | --- | --- |
| Parar y arrancar el mismo contenedor | Se conserva. | Se conserva. | Se conserva. |
| Eliminar y recrear el contenedor | Se pierde. | Se conserva si se reutiliza el volumen. | Se conserva. |
| `docker compose down -v` | Se pierde. | Elimina el volumen del ejemplo. | No elimina la carpeta del ordenador. |
| Ver archivos en una carpeta del ordenador elegida por nosotros | No. | No. | Sí. |
| Ruta utilizada por PHP | `/var/www/almacen-ud04` | `/var/www/almacen-ud04` | `/var/www/almacen-ud04` |

Para seguir los ejemplos puedes mantener la **opción 2**, que conserva las subidas sin mezclarlas con el código. No hay que implementar tres programas PHP: el programa es el mismo y cambia el montaje del almacén.

**Cambiar de opción no traslada automáticamente las subidas anteriores.** Cada montaje puede mostrar un almacén diferente; copia antes lo que quieras conservar. `docker compose up -d` aplica los cambios del Compose; `docker compose restart` solo reinicia y no aplica un montaje nuevo.

Ejecuta siempre los comandos en el mismo proyecto Compose que atiende tu página. Preparar una carpeta en otro contenedor no prepara la del servidor que estás utilizando.

## 3.2. El formulario que envía el archivo

Hasta ahora enviábamos textos, números y opciones. Para adjuntar un archivo, incorporamos **un selector de archivos** y especificamos **cómo debe preparar el navegador el envío**:

```html
<form action="subir.php" method="post" enctype="multipart/form-data">
    <label for="archivo">Selecciona un archivo:</label>
    <input type="file" id="archivo" name="archivo" required>
    <button type="submit">Subir archivo</button>
</form>
```

### Seleccionar: input type="file"

```html
<input type="file" id="archivo" name="archivo" required>
```

`type="file"` muestra un control con el que el usuario puede **seleccionar un archivo de su ordenador**. Seleccionarlo todavía no lo sube: el navegador lo enviará cuando se envíe el formulario.

* `id="archivo"` relaciona el control con su etiqueta mediante `for="archivo"`.
* `name="archivo"` establece el nombre que utilizaremos para consultar la subida en PHP: `$_FILES['archivo']`.
* `required` pide seleccionar un archivo antes de enviar. Es una ayuda del navegador; también comprobaremos la recepción en PHP.

El navegador no permite que nuestra página rellene este control con una ruta del ordenador para subir archivos automáticamente. Si se vuelve a mostrar el formulario, el usuario deberá seleccionar el archivo otra vez.

### Enviar: qué indica enctype="multipart/form-data"

En nuestro formulario utilizamos **POST**, como ya hacíamos con otros formularios. La novedad es `enctype`, que indica **cómo organiza el navegador los datos en el cuerpo de esa petición**.

Cuando no escribimos `enctype`, la codificación predeterminada es `application/x-www-form-urlencoded`. Un formulario con título y nombre puede enviar sus datos así:

```text
titulo=Mi+practica&nombre=Ana
```

Con `enctype="multipart/form-data"`, el navegador prepara el envío en **partes separadas**: una para cada campo y otra para el archivo, con su contenido. Añade automáticamente los separadores y la información necesaria para distinguirlas; no tenemos que programar esa estructura.

| Formulario | Qué utilizamos |
| --- | --- |
| Solo textos, números, selecciones o casillas. | Podemos omitir `enctype` y utilizar la codificación predeterminada. |
| Incluye archivos. | Utilizamos `method="post"` y `enctype="multipart/form-data"`. |

**También podemos utilizar `multipart/form-data` sin archivos:** los campos normales seguirán llegando a `$_POST`, aunque no es necesario para ese caso. No significa «solo archivos», sino «datos organizados en partes, que pueden incluir archivos».

El selector y la codificación cumplen funciones diferentes: **`type="file"` permite elegir el archivo; `multipart/form-data` permite enviarlo correctamente con el formulario**. Si olvidamos ese `enctype`, PHP no recibirá correctamente la subida en `$_FILES`.

### Combinar texto y archivo

Un mismo formulario puede enviar ambos:

```html
<form action="subir.php" method="post" enctype="multipart/form-data">
    <label for="titulo">Título:</label>
    <input type="text" id="titulo" name="titulo">

    <label for="archivo">Archivo:</label>
    <input type="file" id="archivo" name="archivo" required>

    <button type="submit">Enviar</button>
</form>
```

PHP los recoge por separado:

```php
$titulo = $_POST['titulo'] ?? '';       // El texto.
$archivo = $_FILES['archivo'] ?? null; // La información del archivo.
```

**Los campos normales llegan a `$_POST`; los archivos se consultan en `$_FILES`.**

### Orientar el selector con accept

Si queremos orientar la selección hacia documentos PDF, añadimos `accept`:

```html
<input type="file" name="archivo" accept=".pdf,application/pdf">
```

Podemos indicar extensiones como `.pdf` y tipos MIME como `application/pdf`. Separamos los valores con comas.

Si queremos permitir seleccionar **cualquier tipo de archivo**, omitimos `accept`:

```html
<input type="file" name="archivo">
```

| Qué queremos seleccionar | Ejemplo |
| --- | --- |
| PDF | `accept=".pdf,application/pdf"` |
| JPEG y PNG | `accept="image/jpeg,image/png"` |
| Cualquier imagen | `accept="image/*"` |
| Cualquier tipo de archivo | Sin atributo `accept`. |

**`accept` orienta el selector, pero no valida el archivo en el servidor.** El usuario o un cliente distinto puede enviar otro formato. Después comprobaremos el contenido con PHP.

Permitir cualquier tipo no significa seleccionar varios archivos a la vez. Para varias selecciones existe `multiple`, pero en esta unidad trabajaremos con **un solo archivo por envío**, para entender primero el recorrido.

### Qué es un tipo MIME

MIME es una forma estándar de **indicar el tipo de contenido** de un archivo o de un dato. Se expresa así:

```text
tipo/subtipo
```

| Archivo o contenido | Tipo MIME |
| --- | --- |
| Documento PDF | `application/pdf` |
| Imagen JPEG | `image/jpeg` |
| Imagen PNG | `image/png` |
| Texto sencillo | `text/plain` |
| Página HTML | `text/html` |

Por ejemplo, en `image/png`, `image` indica que es una imagen y `png` concreta el formato. En `application/pdf`, el valor completo identifica un documento PDF.

No confundamos **extensión**, **tipo MIME** y **contenido**:

* `memoria.pdf` es un nombre que termina en la extensión `.pdf`.
* `application/pdf` es el tipo MIME que identifica ese formato.
* El contenido son los datos que componen realmente el archivo.

Renombrar `memoria.pdf` como `memoria.jpg` no transforma el documento en una imagen. Asimismo, el MIME que comunica el navegador puede ser incorrecto: no basta con confiar en esa etiqueta.

Más adelante utilizaremos **Fileinfo** para que PHP examine el contenido del archivo temporal e identifique su tipo. Primero vamos a observar dónde llega la subida.

## 3.3. Qué ha recibido PHP

Cuando comienza a ejecutarse nuestro código tras el envío, PHP ya ha gestionado la recepción. Para verla, guarda este archivo como **`inspeccionar.php`** y ábrelo a través del servidor:

```php
<?php
if (($_SERVER['REQUEST_METHOD'] ?? '') === 'POST') {
    echo '<pre>';
    echo htmlspecialchars(print_r($_FILES, true), ENT_QUOTES, 'UTF-8');
    echo '</pre>';
}
?>
<form action="inspeccionar.php" method="post" enctype="multipart/form-data">
    <input type="file" name="archivo" required>
    <button type="submit">Inspeccionar</button>
</form>
```

`print_r` permite observar la estructura del array. Escapamos su salida porque el nombre del archivo procede del usuario.

Por ejemplo, al seleccionar `memoria.pdf`, la entrada `$_FILES['archivo']` puede contener estos datos:

| Clave | Ejemplo | Qué significa |
| --- | --- | --- |
| `name` | `memoria.pdf` | Nombre original comunicado por el navegador. |
| `tmp_name` | `/tmp/phpA1b2C3` | Ruta temporal donde PHP ha dejado el contenido recibido. |
| `size` | `153600` | Tamaño del archivo en bytes. |
| `type` | `application/pdf` | Tipo comunicado por el navegador; no es una comprobación del servidor. |
| `error` | `0` | Recepción correcta; utilizaremos la constante `UPLOAD_ERR_OK`. |

**`tmp_name` apunta a un archivo del contenedor. No es la ruta del archivo en el ordenador del alumno.**

Si no conservamos ese temporal, PHP lo elimina al terminar la petición. En esta prueba solo lo hemos inspeccionado: todavía no lo hemos guardado en nuestro almacén.

## 3.4. Guardar el archivo: el ejemplo más sencillo

Vamos a centrarnos en la instrucción que conserva la subida. Guarda lo siguiente como **`subir-basico.php`**:

```php
<?php
$mensaje = '';

if (($_SERVER['REQUEST_METHOD'] ?? '') === 'POST') {
    if (($_FILES['archivo']['error'] ?? null) === UPLOAD_ERR_OK) {
        $temporal = $_FILES['archivo']['tmp_name'];

        // Nombre generado por nuestra aplicación, independiente del original.
        $nombre = bin2hex(random_bytes(16)) . '.dat';
        $destino = '/var/www/almacen-ud04/' . $nombre;

        if (move_uploaded_file($temporal, $destino)) {
            $mensaje = 'Archivo guardado correctamente.';
        } else {
            $mensaje = 'No se ha podido guardar. Revisa la carpeta y sus permisos.';
        }
    } else {
        $mensaje = 'Selecciona un archivo pequeño y vuelve a intentarlo.';
    }
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Primera subida</title>
</head>
<body>
    <h1>Primera subida</h1>
    <p><?= htmlspecialchars($mensaje, ENT_QUOTES, 'UTF-8') ?></p>
    <form action="subir-basico.php" method="post" enctype="multipart/form-data">
        <input type="file" name="archivo" required>
        <button type="submit">Subir archivo</button>
    </form>
</body>
</html>
```

Para esta demostración controlada, utiliza un pequeño archivo de texto. El ejemplo guarda el contenido con extensión `.dat` porque todavía no estamos decidiendo qué formatos admitimos. **Cambiar la extensión no convierte el contenido a otro formato.** Después sustituiremos esta demostración por la versión que comprueba tamaño y tipo.

### La instrucción principal

```php
move_uploaded_file($temporal, $destino);
```

| Argumento | Qué representa |
| --- | --- |
| `$temporal` | Archivo recibido que queremos mover: por ejemplo, `/tmp/phpA1b2C3`. |
| `$destino` | Ruta completa donde lo conservaremos, incluyendo el nombre nuevo. |

La función verifica que el origen procede de una subida HTTP POST de PHP y trata de moverlo. Devuelve `true` si lo consigue o `false` si falla. **No crea la carpeta de destino.**

Comprueba qué se ha guardado:

```bash
docker compose exec apache_php ls -lh /var/www/almacen-ud04
```

### El recorrido de esta primera prueba

1. Abres `subir-basico.php` mediante GET y recibes el formulario.
2. Seleccionas un archivo y pulsas el botón: el navegador envía un POST.
3. PHP recibe el contenido en un archivo temporal y prepara `$_FILES`.
4. Nuestro código obtiene la ruta temporal y prepara la ruta definitiva.
5. `move_uploaded_file` mueve el archivo al almacén.
6. PHP devuelve el HTML con el mensaje de resultado.

El archivo original del ordenador del alumno permanece donde estaba. Lo que movemos es **la copia temporal recibida por el servidor**.

## 3.5. Por qué generamos otro nombre

Hay tres conceptos diferentes:

| Nombre o ruta | Quién lo decide | Cuándo se utiliza |
| --- | --- | --- |
| Original: `memoria.pdf` | El usuario, a través del archivo seleccionado. | Llega como información en `name`. |
| Temporal: `/tmp/phpA1b2C3` | PHP. | Durante el procesamiento de la petición. |
| Definitivo: `a81f92c4….pdf` | Nuestra aplicación. | Al conservar el archivo en el almacén. |

Supón que Ana y Luis entregan un archivo llamado **`memoria.pdf`**. Si construimos siempre este destino:

```php
$destino = '/var/www/almacen-ud04/memoria.pdf';
```

ambas subidas apuntan al mismo archivo. **La segunda puede sobrescribir la primera.** El hecho de que PHP les haya asignado temporales diferentes no evita este problema si después elegimos el mismo destino.

Por eso generamos un nombre definitivo diferente para cada subida:

```php
$nombre = bin2hex(random_bytes(16)) . '.pdf';
```

* `random_bytes(16)` genera 16 bytes aleatorios.
* `bin2hex` los convierte en 32 caracteres que podemos utilizar en el nombre.
* Añadimos `.pdf` si nuestra aplicación ha comprobado que admite ese PDF.

Así, las dos memorias originales pueden llamarse igual y almacenarse con nombres diferentes. La coincidencia de los nombres generados resulta extremadamente improbable.

**No hay que aprender la generación aleatoria como el mecanismo de subida:** su finalidad es elegir un destino que no dependa del nombre original. La instrucción que conserva el archivo sigue siendo `move_uploaded_file`.

## 3.6. Añadir unas comprobaciones básicas

Ahora que sabemos guardar, añadimos tres preguntas antes de mover:

### ¿Se ha recibido correctamente?

Nos bastan estas dos constantes:

| Constante | Significado |
| --- | --- |
| `UPLOAD_ERR_OK` | Se ha recibido correctamente. Su valor es `0`. |
| `UPLOAD_ERR_NO_FILE` | No se ha seleccionado un archivo. |

Para otros errores de recepción mostraremos un mensaje general. No necesitamos memorizar todos los códigos.

### ¿Tiene un tamaño permitido?

`size` se expresa en bytes. Por ejemplo, para admitir hasta **2 MiB**:

```php
$maximo = 2 * 1024 * 1024;

if ($archivo['size'] <= 0 || $archivo['size'] > $maximo) {
    $error = 'La imagen debe ocupar más de 0 bytes y hasta 2 MiB.';
}
```

### ¿El contenido es de un tipo admitido?

Poner `.jpg` al final del nombre no convierte un archivo en una fotografía. Tampoco basta con confiar en `type`, que lo comunica el navegador.

Utilizaremos **Fileinfo**, que examina el contenido del temporal:

```php
$finfo = new finfo(FILEINFO_MIME_TYPE);
$tipo = $finfo->file($archivo['tmp_name']);
```

Creamos un objeto `finfo` y le pedimos que examine el archivo. El resultado es un tipo MIME, como `image/jpeg` o `image/png`. Es una identificación del formato, no una garantía de que cualquier archivo sea inocuo.

Definimos qué tipos admitimos y qué extensión asignamos:

```php
$permitidos = [
    'image/jpeg' => 'jpg',
    'image/png' => 'png',
];
```

Esta pequeña tabla permite comprobar el tipo y elegir la extensión del nombre definitivo desde nuestro código.

## 3.7. Ejemplo completo: subir una imagen

Guarda este ejemplo como **`subir.php`**. Mantiene el procesamiento en el mismo archivo, sin funciones de subida ni excepciones: primero comprobamos y, solo si no hay errores, guardamos.

```php
<?php
declare(strict_types=1);

$error = '';
$nombreGuardado = '';
$carpeta = '/var/www/almacen-ud04';
$permitidos = ['image/jpeg' => 'jpg', 'image/png' => 'png'];

if (($_SERVER['REQUEST_METHOD'] ?? '') === 'POST') {
    $archivo = $_FILES['imagen'] ?? null;

    // Esperamos una sola imagen, no un conjunto imagen[].
    if (!is_array($archivo) || !isset($archivo['error'])
        || !is_int($archivo['error'])) {
        $error = 'No se ha recibido una subida válida. Revisa el formulario y los límites de PHP.';
    } elseif ($archivo['error'] === UPLOAD_ERR_NO_FILE) {
        $error = 'Selecciona una imagen.';
    } elseif ($archivo['error'] !== UPLOAD_ERR_OK) {
        $error = 'Ha ocurrido un error al recibir la imagen. Revisa su tamaño y vuelve a intentarlo.';
    } elseif (!isset($archivo['size'], $archivo['tmp_name'])
        || !is_int($archivo['size']) || !is_string($archivo['tmp_name'])) {
        $error = 'Los datos de la subida no son válidos.';
    } elseif ($archivo['size'] <= 0 || $archivo['size'] > 2 * 1024 * 1024) {
        $error = 'La imagen debe ocupar más de 0 bytes y hasta 2 MiB.';
    }

    // Analizamos el contenido solo si la recepción y el tamaño son correctos.
    if ($error === '') {
        $finfo = new finfo(FILEINFO_MIME_TYPE);
        $tipo = $finfo->file($archivo['tmp_name']);

        if ($tipo === false || !array_key_exists($tipo, $permitidos)) {
            $error = 'Solo se admiten imágenes JPEG o PNG.';
        }
    }

    // Guardamos solo después de superar las comprobaciones.
    if ($error === '') {
        $nombre = bin2hex(random_bytes(16)) . '.' . $permitidos[$tipo];
        $destino = $carpeta . '/' . $nombre;

        if (move_uploaded_file($archivo['tmp_name'], $destino)) {
            $nombreGuardado = $nombre;
        } else {
            $error = 'No se ha podido guardar. Revisa la carpeta y sus permisos.';
        }
    }
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Subir imagen</title>
</head>
<body>
    <h1>Subir una imagen</h1>
    <p>Selecciona una imagen JPEG o PNG de hasta 2 MiB.</p>

    <?php if ($error !== ''): ?>
        <p role="alert"><?= htmlspecialchars($error, ENT_QUOTES, 'UTF-8') ?></p>
    <?php endif; ?>

    <?php if ($nombreGuardado !== ''): ?>
        <p>Imagen guardada como:
            <?= htmlspecialchars($nombreGuardado, ENT_QUOTES, 'UTF-8') ?>
        </p>
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

### Cómo leerlo sin perderse

El procesamiento tiene tres bloques separados por comentarios:

1. **Recepción y tamaño:** comprobamos que tenemos una subida individual correcta y que no supera el límite.
2. **Tipo:** Fileinfo examina el contenido y lo contrastamos con el catálogo.
3. **Guardado:** generamos un nombre y movemos el temporal.

`$error === ''` significa «hasta ahora no hemos encontrado ningún error». Si un bloque establece un mensaje, los siguientes ya no se ejecutan. Las comprobaciones de tipo con `is_array`, `is_int` e `is_string` evitan trabajar con una estructura inesperada; no hay que confundirlas con la comprobación del formato de imagen.

El atributo `accept` facilita la selección, pero no sustituye Fileinfo. Si hay un error, el usuario debe seleccionar otra vez el archivo: el navegador no permite rellenar el campo `file` mediante `value`.

Esta versión muestra el resultado en la respuesta al POST. Si recargas y confirmas su reenvío, puedes guardar otra copia. Más adelante podremos aplicar POST/Redirect/GET y conservar el mensaje mediante sesiones.

## 3.8. Límites de PHP y comprobación final

Nuestro código permite hasta 2 MiB, pero PHP también tiene límites de recepción. Para las pruebas de clase, una configuración adecuada es:

```ini
file_uploads = On
upload_max_filesize = 4M
post_max_size = 5M
```

* `upload_max_filesize` limita cada archivo.
* `post_max_size` limita todo el envío, incluidos el archivo y los demás campos.

El margen permite recibir una imagen de más de 2 MiB y menos de 4 MiB para comprobar que **nuestro código** la rechaza. Si se supera `post_max_size`, los arrays del envío pueden llegar vacíos.

Estos valores deben estar en la configuración de PHP que utiliza Apache **dentro del contenedor**. Si ya montáis un archivo `.ini` en `/usr/local/etc/php/conf.d/`, edita ese archivo; después reinicia el servicio:

```bash
docker compose restart apache_php
```

Si todavía no tienes ese montaje, crea `php/subidas.ini` junto al Compose con el contenido anterior y añade esta línea a los `volumes` del servicio, junto a las existentes:

```yaml
- ./php/subidas.ini:/usr/local/etc/php/conf.d/subidas.ini:ro
```

Después aplica el nuevo montaje con `docker compose up -d`. No basta con reiniciar cuando has cambiado los montajes del Compose. Fileinfo también debe estar habilitada en el PHP del contenedor.

### Pruebas para clase

| Prueba | Resultado esperado |
| --- | --- |
| JPEG o PNG pequeño. | Se guarda y aparece el nombre generado. |
| Otro archivo con el mismo nombre original. | Se conserva como otro archivo, con otro nombre generado. |
| Un PDF renombrado a `.jpg`. | Se rechaza por el tipo del contenido. |
| Imagen de más de 2 MiB que PHP permite recibir. | Se rechaza por tamaño. |
| Ningún archivo, quitando temporalmente `required`. | Se pide seleccionar una imagen. |

Tras las subidas, lista el almacén:

```bash
docker compose exec apache_php ls -lh /var/www/almacen-ud04
```

Si no se guarda, vuelve al punto 3.1 y comprueba carpeta, permisos y proyecto Compose. El almacén no está publicado: no podrás abrir sus archivos escribiendo su ruta interna en el navegador.

## 3.9. Qué debemos recordar

* El navegador envía; PHP recibe en un temporal; nuestro código decide si lo conserva.
* El formulario necesita POST, `multipart/form-data` y un campo `file` con `name`.
* `$_FILES` describe la subida; `tmp_name` indica su ubicación temporal.
* El nombre temporal lo asigna PHP; el definitivo lo elegimos nosotros.
* Generar un nombre evita depender del original y hace extremadamente improbable sobrescribir otra subida por coincidencia de nombre.
* `move_uploaded_file` conserva el archivo en un destino cuya carpeta ya debe existir y permitir la escritura.
* Primero entendemos el recorrido; después comprobamos recepción, tamaño y tipo.

## Practica lo aprendido

Realiza el **ejercicio 5 del bloque 3: Subida de archivos** de la [relación de actividades](actividades.md#bloque-3-subida-de-archivos). Aplicarás el mismo recorrido a una memoria PDF y guardarás el archivo solo si también es válido su título.

Comenzaremos el ejercicio en clase y completarás en casa lo pendiente. Revisaremos la solución por separado de los enunciados.

## Referencias

* [Aitor Medrano y Luis Alemañ: Programación web](https://aitor-medrano.github.io/dwes2122/04web.html), referencia didáctica de la unidad.
* [Manual de PHP: subida de archivos mediante POST](https://www.php.net/manual/es/features.file-upload.post-method.php).
* [Manual de PHP: move_uploaded_file](https://www.php.net/manual/es/function.move-uploaded-file.php).
* [Manual de PHP: Fileinfo](https://www.php.net/manual/es/function.finfo-file.php).
* [Docker: volúmenes en Compose](https://docs.docker.com/reference/compose-file/volumes/).

## Autoría y licencia

Material elaborado por **Joaquín García** para DWES de 2.º DAW del **IES Portada Alta**. Adapta y amplía los materiales de **Aitor Medrano** y **Luis Alemañ**, con ejemplos apoyados en el manual de PHP y la documentación de Docker.

Licencia **Creative Commons Reconocimiento-NoComercial-CompartirIgual (CC BY-NC-SA)**.
