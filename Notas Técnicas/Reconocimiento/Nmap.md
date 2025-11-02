
Es una herramienta open Source para el descubrimiento y auditoría de seguridad de redes


## Para qué sirve

- Descubrimiento de Hosts
- Escaneo de puertos
- Detección de Servicios/Versiones
- Detección de Sistemas Operativos (OS Figerprinting)

## Sintaxis

nmap {Tipo de Escaneo} {Opciones} {Objetivo}

Ejemplo básico

```bash
nmap IP_Objetivo
```

## Modificaciones de Nmap


#### SYN Scan

**Uso:** Es el escaneo por defecto para usuarios con privilegios

Ejemplo:

```bash
nmap -sS IP_Objetivo
```



#### TCP Connect Scan

**¿Qué es?** Nmap completa el handshake TCP de tres vías con cada puerto (funciona sin privilegios root)

Ejemplo:

```bash
nmap -sT IP_Objetivo
```


#### UDP Scan

**Uso:** Necesario para identificar servicios basados en UDP

Ejemplo:

```bash
nmap -sU IP_Objetivo
```


#### Descubrimiento de hosts y puertos


**Descubrimiento de hosts (-sn & -Pn)**

- **-sn (Ping Scan):** Sólo descubre hosts activos sin escanear puertos

Ejemplo:

```bash
nmap -sn IP_Objetivo
```

- **-Pn (No Ping):** Útil si el objetivo bloquea ICMP ya que salta la fase de ping asumiendo que todos los hosts están activos

Ejemplo:

```bash
nmap -Pn IP_Objetivo
```


**Especificación de Puertos (-p)**

- Escaneo de puertos específicos

```bash
nmap - 80,443,22 IP_Objetivo
```

- Escaneo de rango de puertos

```bash
nmap -p- 1-1024 IP_Objetivo
```

- Escaneo de todos los puertos (65535)

```bash
nmap -p- IP_Objetivo
```

- Puertos comunes (1000 más comunes por defecto)

```bash
nmap IP_Objetivo
```


- Especificar puertos comunes en uso -top-ports=XX


#### Detección de versiones


**Uso** Conocer la versión exacta de un servicio mediante ***banner grabbing***

Ejemplo:

```bash
nmap -sV IP_Objetivo
```

Ejemplo combinado:

```bash
nmap -sS -sV IP_Objetivo --> SYN scan + detección de versiones
```


#### Uso de Scripts NSE

**NSE** es un potente motor que permite a los usuarios escribir scipts en Lua para automatizar una gran variedad de tareas

**Scripts comunes**

- auth: Detección de credenciales por defecto
- vuln: Detección de vulnerabilidades específicas
- enum: Enumeración de información (usuarios, recursos, DNS...)
- exploit: Explotación de vulnerabilidades
- brute: fuerza bruta con credenciales


#### Uso de Scripts

- Ejecutar una categoría:

```bash
nmap --script=vuln Objetivo
```

- Ejecutar un script específico:

```bash
nmap --script=http-enum Objetivo
```

- Ejecutar todos los scripts predeterminados (más lento):

```bash
nmap -sc Objetivo
```

-Ver información del módulo en uso:

```bash
nmap --script-help http-headers
```

- [Directorio de Scripts](https://nmap.org/nsedoc/)


#### Nmap Agresivo, Fragmentado y Evasión


**Escaneo Agresivo (-A)**

- Combina detección de SO (-O) , detección de versiones (-sV), escaneo de scripts predeterminados (-sC) y traceroute. Hay que tener en cuenta que a pesar de proporcionarnos una gran cantidad de información rápidamente, es muy ruidoso y muy fácil de detectar

**Sintaxis**

```bash
nmap -A Objetivo
```


**Escaneo Fragmentado (-f**

- Divide los paquetes TCP en fragmentos más pequeños para intentar evadir algunos firewalls o IDS/IPS

**Sintaxis**

```bash
nmap -f Objetivo
```


**Evasión de Firewalls/IDS**

```bash
--data-length <num> #--> Añade datos aleatorios a los paquetes
```

```bash
--badsum #--> Envía paquetes con checksums TCP/UDP/IP inválidos
```

```bash
--source-port <port> #--> Especifica el puerto de origen
```

```bash
--scan-delay <time> #--> Añade un retraso entre sondas para evitar detecciones por velocidad
```

