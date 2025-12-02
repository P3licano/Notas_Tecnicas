

### Metodología

1. **Revisar respuestas del servidor**: Errores, headers, mensajes de debug
2. **Analizar código fuente**: Comentarios HTML, JavaScript expuesto
3. **Buscar archivos sensibles**: Backups, configuraciones, robots.txt
4. **Examinar metadata**: En documentos, imágenes subidas
5. **Recopilar información**: Versiones de software, estructura de directorios, usuarios


### Error messages verbose

```
# Stack traces completos
Fatal error: Uncaught Error: Call to undefined function mysql_connect() in /var/www/html/config.php:15

# Información de la base de datos
MySQL error: Table 'users' doesn't exist in database 'webdb'

# Versiones de software
Apache/2.4.41 (Ubuntu)
PHP/7.4.3
```


### Directory traversal en mensajes de error

```
http://vulnerable.com/file?path=../../../etc/passwd

# Error: File "/var/www/html/../../../etc/passwd" not found
# Revela estructura de directorios
```


### Comentarios en código fuente

```html
<!-- TODO: Remover usuario admin:admin123 de testing -->
<!-- API key: sk-1234567890abcdef -->
<!-- Debug: /admin/debug.php -->
<!-- Database: mysql://user:pass@localhost/dbname -->
```


### Robots.txt y sitemap.xml

```ruby
# robots.txt
User-agent: *
Disallow: /admin/
Disallow: /backup/
Disallow: /internal/

# Revela directorios sensibles
```


### Metadatos en archivos

```ruby
# EXIF en imágenes
exiftool image.jpg
# Puede revelar: ubicación GPS, software usado, autor

# Metadata en PDFs
exiftool document.pdf
# Puede revelar: autor, fechas, rutas de archivos

# Metadata en Office
exiftool document.docx
# Puede revelar: autor, empresa, historial de revisiones
```


### Backup files expuestos

```
# Archivos comunes
config.php.bak
config.php~
config.php.old
config.php.backup
.git/
.svn/
database.sql
backup.zip
dump.sql
```


### Sensitive data en URL

```
# Tokens en GET
http://vulnerable.com/reset?token=abc123secret

# Session IDs
http://vulnerable.com/page?PHPSESSID=1234567890abcdef

# Credenciales
http://vulnerable.com/login?user=admin&pass=secret
```