
Es la recopilación de información a través de fuentes abiertas. Hay de dos tipos

1. Recopilación activa
	- Puede dejar rastro como logs o solicitudes DNS
	- Busca información directamente del sistema público del objetivo
	- Requiere de herramientas más específicas para la interacción
2. Recopilación pasiva
	- No deja rastro directo en los sistemas del objetivo
	- Generalmente más segura y menos arriesgada en cuanto a protección
	- Basada en información ya publicada o almacenada en terceros



### Técnicas de enumeración pasiva

- ***Whois*** --> podemos ver bastante información sobre dominios
- ***crt.sh / Censys*** --> Bases de datos de certificados SSL/TLS emitidos
- ***DNSSumpster*** --> Herramienta web usada para visualizar la estructura DNS de un dominio
- ***Wayback Machine*** --> Permite ver *snapshots* archivadas de sitios web a lo largo del tiempo
- ***Shodan*** --> Base de datos con todo tipo de información como puertos abiertos y servicios
- ***Google Dorking***

#### Google Dorking


**Operadores lógicos**

- OR |
- AND &

**Sintaxis**

Combinación de operadores: `(site:facebook.con | site twitter.com) & intext:"login"`

Excluir resultados: `site:facebook.* -site:facebook.com`

Incluir resultados: `-site:facebook.com +site:facebook.`

**Dorks útiles**

- `site:objetivo.com filetype:pdf`
- `site:objetivo.com inurl:admin`
- `site:objetivo.com intitle:"Index of"`
- `site:objetivo.com intext:"password" | intext:"contraseña"`
- `inurl:wp-admin site:objetivo.com`
- `inurl:login.php intext:"failed login"`


### Técnicas de enumeración activa


- ***Sublist3r*** --> Listar subdirectorios
- ***dnsrecon*** --> Reconocimiento DNS
- ***nslookup*** --> Conocer la ip del dominio
- ***Gobuster / Wfuzz*** --> Herramientas de Fuzzing mediante fuerza bruta
- ***Nmap*** --> Consultar [[Nmap|Nmap]]
- ***Whatweb*** --> Similar a ***Wappalyzer***, detecta las tecnologías de una página web visitada


### Técnicas de Enumeración de  nicknames

- ***WhatsMyName*** --> Herramienta web para buscar un username
- ***Sherlock*** --> Binario que busca un username en redes sociales
- **Motores de búsqueda** --> Hacer uso de *dorking* en  múltiples navegadores


### Búsqueda de correos electrónicos

- Usar funciones de "Recuperar la contraseña" o "Olvidé mi usuario" en páginas populares para verificar si el email está vinculado a una cuenta
- Usar páginas de filtraciones como ***Dehashed / IntelX***


### Herramientas para búsqueda reversa de imágenes

- ***Google lens***
- ***PimEyes***
- ***Yandex Images***


### Análisis de archivos internamente

- Metadatos:
	- ***ExifTool***
	- ***FotoForensics***

