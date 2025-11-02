

### ¿Qué es?

Es una herramienta usada para el ***debugging*** a través de TCP/UDP pudiendo leer y escribir

### ¿Para qué se usa?

Al ser un tipo de conexiones muy ligera, es usado para conexiones reversas, inversas y para conectarse a servicios ligeros como ***SMTP, Telnet o POP3***, tambien se puede usar como:

- Escáner de puertos básoco
- Cliente/servidor UDP/TCP
- Transferencia de archivos
- Banner grabbing
- Shell inversa/directa
- Servidor de chat


### Sintaxis


Sintaxis básica:

```bash
nc -zv IP_Objetivo P_Inicial-P_Final
```

- -z: Modo de escaneo (zero-I/O,  no envía datos)
- -v: Modo *Verbose* (muestra más info)

Hay que tener en cuenta las **limitaciones de netcat a comparación con nmap**, ya que netcat no nos da la flexibilidad ni la potencia que nos ofrece nmap en cuanto a escaneos complejos


### Netcat como cliente


Lo podemos usar para interactuar manualmente con los servicios web, FTP...

Comando:

```bash
nc IP Puerto
```

Ejemplo:

```bash
nc 192.168.1.15 80 --> conexion por el puerto 80 a la ip 192.168.1.15
```


### Netcat como servidor


Lo podemos usar para recibir conexiones inversas como rev shell o para transferir archivos

Abrimos un puerto en local a la escucha de conexiones entrantes

```bash
nc -lvp Puerto
```

- -l: Modo de escucha (listen)
- -v Modo *Verbose*
- -p: Especifica el puerto

Ejemplo:

```bash
nc -lvp 4444 --> escuha en el puerto 4444
```


### Netcat para Transferencia de archivos


En el receptor (servidor)

```bash
nc -lvp 1234 > archivo_recibido.txt
```

En el emisor (cliente)

```bash
nc IP_Receptor 1234 < archivo_a_enviar.txt
```


