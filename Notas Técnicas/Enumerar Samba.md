

***Debemos de tener los permisos que nos indique, o poder acceder como guest***

 Consultar esta página de ser necesario --> https://0xdf.gitlab.io/cheatsheets/smb-enum#

Enumera archivos compartidos

```bash
smbclient -N -L $IP_victima
```


Accede al recurso compartido


```bash
smbclient //$IP_victima/compartido -N

#-N = no pass
```


Para descargarnos todo lo de un recurso compartido

```shell
mget *
```


### Enumerar samba para AD


```bash
smbclient -U n_dom\\user "//ip_dom/recurso_a_acceder"
```