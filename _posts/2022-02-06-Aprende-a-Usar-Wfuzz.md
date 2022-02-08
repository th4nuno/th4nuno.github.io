---
layout: post
title: "Aprende a Usar Wfuzz"
date: 2022-02-06
excerpt: "El fuzzing es una de las labores más importantes que un pentester debería saber, y Wfuzz pone esta tarea lo más fácil posible con una increíble herramienta que nos trae todo lo necesario para hacerlo"
tags: [fuzzing, pentesting, tools, web]
feature: 'https://th4nuno.github.io/assets/img/posts/wfuzz.jpg'
comments: true
---

# ¿Qué es Wfuzz?

**Wfuzz** es una herramienta que nos permite hacer **fuzzing** a una página web, reemplazando la palabra clave **FUZZ** con un tipo de **payload**, que puede ser una lista de directorios, un rango de números, entre otras más opciones. Tambien nos permite hacer un escaneo recursivo, seguir las redirecciones para obtener el código de estado final y más cosas que las iremos viendo en este post. 

# Como Instalar Wfuzz

#### Muy lindo todo, pero... ¿Como instalo Wfuzz?

Depende de en que sistema operativo te encuentres. Lo que yo recomiendo es que si querés hacer pentesting te instales Kali Linux o Parrot (yo utilizo Parrot), que son sistemas especializados en Hacking Ético que traen por defecto una gran cantidad de herramientas, entre ellas Wfuzz. 

De forma alternativa, se puede descargar fácilmente haciendo uso del siguiente comando (teniendo Python instalado):

{% highlight bash %}
pip install wfuzz
{% endhighlight %}

# Como Usar Wfuzz

## Fuzzing con Diccionarios

Bien, ahora que ya lo tenemos instalado, vamos a ver como se usa:

{% highlight bash %}
wfuzz --hc=404 -u "https://google.com/FUZZ" -w /usr/share/wordlists/wfuzz/general/common.txt
{% endhighlight %}

En el comando de arriba lo que hice fue realizar un escaneo en la página web de google.com, especificando la URL con el parametro *-u*, utilizando un diccionario de directorios que trae Wfuzz por defecto con *-w*. Además con *--hc* escondí el código 404 (Not Found) del output, porque los directorios que no existen no me sirven de nada.

Esto es lo que me devolvió:

{% highlight bash %}
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: https://google.com/FUZZ
Total requests: 951

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000015:   301        6 L      14 W       224 Ch      "2001"
000000019:   301        6 L      14 W       224 Ch      "2005"
000000018:   301        6 L      14 W       224 Ch      "2004"
000000017:   301        6 L      14 W       224 Ch      "2003"
000000016:   301        6 L      14 W       224 Ch      "2002"
000000029:   301        6 L      14 W       227 Ch      "account"
000000025:   301        6 L      14 W       225 Ch      "about"
000000021:   301        6 L      14 W       221 Ch      "a"
000000031:   301        6 L      14 W       226 Ch      "action"
000000066:   301        6 L      14 W       229 Ch      "appliance"
000000070:   301        6 L      14 W       224 Ch      "apps"
000000113:   301        6 L      14 W       224 Ch      "blog"
000000130:   301        6 L      14 W       228 Ch      "business"
000000184:   301        6 L      14 W       228 Ch      "commerce"
000000203:   301        6 L      14 W       227 Ch      "contact"
000000204:   301        6 L      14 W       228 Ch      "contacts"
000000220:   301        6 L      14 W       231 Ch      "creditcards"
000000226:   301        6 L      14 W       229 Ch      "customers"
000000255:   301        6 L      14 W       226 Ch      "design"
000000262:   301        6 L      14 W       230 Ch      "developers"
000000279:   301        6 L      14 W       224 Ch      "docs"
000000265:   301        6 L      14 W       227 Ch      "devices"
000000264:   302        9 L      18 W       216 Ch      "device"
000000286:   301        6 L      14 W       229 Ch      "downloads"
000000314:   301        6 L      14 W       226 Ch      "errors"
000000309:   301        6 L      14 W       230 Ch      "enterprise"
000000342:   301        6 L      14 W       225 Ch      "files"
000000366:   301        6 L      14 W       225 Ch      "games"
000000370:   301        6 L      14 W       223 Ch      "get"
000000400:   301        6 L      14 W       228 Ch      "homepage"
000000399:   301        6 L      14 W       224 Ch      "home"
000000380:   301        6 L      14 W       226 Ch      "groups"
000000416:   301        6 L      14 W       225 Ch      "inbox"
000000413:   301        6 L      14 W       226 Ch      "images"
000000397:   302        0 L      0 W        0 Ch        "history"
000000465:   301        6 L      14 W       224 Ch      "labs"
000000460:   301        6 L      14 W       224 Ch      "keep"
000000456:   301        6 L      14 W       222 Ch      "js"
000000498:   301        6 L      14 W       224 Ch      "mail"
000000565:   301        6 L      14 W       224 Ch      "null"
000000558:   302        0 L      0 W        0 Ch        "news"
000000551:   301        6 L      14 W       230 Ch      "navigation"
000000546:   301        6 L      14 W       225 Ch      "music"
000000576:   301        6 L      14 W       222 Ch      "on"
000000632:   301        6 L      14 W       227 Ch      "preview"
000000631:   301        6 L      14 W       225 Ch      "press"
000000611:   301        6 L      14 W       225 Ch      "phone"
000000605:   301        6 L      14 W       223 Ch      "pdf"
000000602:   301        6 L      14 W       228 Ch      "password"
000000656:   301        6 L      14 W       229 Ch      "publisher"
000000689:   301        6 L      14 W       228 Ch      "research"
000000695:   301        6 L      14 W       226 Ch      "retail"
000000714:   301        6 L      14 W       226 Ch      "script"
000000736:   301        6 L      14 W       228 Ch      "services"
000000724:   301        6 L      14 W       228 Ch      "security"
000000710:   302        0 L      0 W        0 Ch        "saved"
000000717:   301        6 L      14 W       226 Ch      "search"
000000763:   301        6 L      14 W       225 Ch      "sites"
000000799:   301        6 L      14 W       227 Ch      "student"
000000797:   301        6 L      14 W       225 Ch      "story"
000000796:   301        6 L      14 W       225 Ch      "store"
000000808:   301        6 L      14 W       226 Ch      "support"
000000810:   301        6 L      14 W       226 Ch      "survey"
000000616:   400        9 L      14 W       145 Ch      "ping"
000000839:   301        6 L      14 W       227 Ch      "toolbar"
000000826:   301        6 L      14 W       229 Ch      "templates"
000000885:   301        6 L      14 W       225 Ch      "views"
000000913:   301        6 L      14 W       229 Ch      "webmaster"
000000930:   301        6 L      14 W       224 Ch      "work"
000000709:   302        0 L      0 W        0 Ch        "save"
000000840:   302        9 L      18 W       211 Ch      "tools"

Total time: 3.595023
Processed Requests: 951
Filtered Requests: 880
Requests/sec.: 264.5323
{% endhighlight %}

Ademas de *--hc* existen otras formas de esconder algunos resultados, porque a veces hay páginas que no nos sirven que resuelven todas el mismo resultado. Estas son:
{% highlight bash %}
--hl/hw/hh: Esconde las respuestas con un determinado numero de lineas/palabras/caracteres
{% endhighlight %}

Como ven arriba casi todas las respuestas que obtuvimos tienen un total de 6 lineas, por lo tanto, si escondemos este número, obtendremos menos resultados:

{% highlight bash %}
wfuzz --hc=404 --hl=6 -u "https://google.com/FUZZ" -w /usr/share/wordlists/wfuzz/general/common.txt
{% endhighlight %}

{% highlight bash %}
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: https://google.com/FUZZ
Total requests: 951

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                     
=====================================================================

000000264:   302        9 L      18 W       216 Ch      "device"                                                                                                                    
000000397:   302        0 L      0 W        0 Ch        "history"                                                                                                                   
000000558:   302        0 L      0 W        0 Ch        "news"                                                                                                                      
000000616:   400        9 L      14 W       145 Ch      "ping"                                                                                                                      
000000840:   302        9 L      18 W       211 Ch      "tools"                                                                                                                     
000000709:   302        0 L      0 W        0 Ch        "save"                                                                                                                      
000000710:   302        0 L      0 W        0 Ch        "saved"                                                                                                                     

Total time: 0
Processed Requests: 951
Filtered Requests: 944
Requests/sec.: 0
{% endhighlight %}

Si miramos el código de respuesta, solo "ping" nos devuelve un número diferente a *302*. Como sabemos que los códigos 301 y 302 son para las redirecciones, podriamos probar a seguirlas con el parámetro *-L*, para ver en que páginas terminamos.

Como este parámetro se va a ejecutar antes de esconder el número de líneas, nos mostrará todas las páginas que teniamos al principio. Asi que para obtener solo los resultados de las últimas páginas, vamos a jugar un poco con otro payload de Wfuzz.

Vamos a guardar nuestro escaneo con el nombre "fuzzing" para usarlo luego:
{% highlight bash %}
wfuzz --hc=404 --hl=6 -u "https://google.com/FUZZ" -w /usr/share/wordlists/wfuzz/general/common.txt --oF fuzzing
{% endhighlight %}

Ahora, utilizando el payload **wfuzzp** vamos a seguir las redirecciones del escaneo anterior. Para consultar los payloads es tan fácil como usar el comando *"wfuzz -e payloads"*

{% highlight bash %}
wfuzz -z wfuzzp,fuzzing -L FUZZ
{% endhighlight %}

{% highlight bash %}
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: FUZZ
Total requests: <<unknown>>

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                     
=====================================================================

000000264:   400        0 L      70 W       1681 Ch     "https://accounts.google.com/o/oauth2/legacy/device/usercode?rt="                                                                                         
000000616:   400        9 L      14 W       145 Ch      "https://google.com/ping"                                                                                                   
000000709:   200        56 L     6191 W     453285 Ch   "https://www.google.com/save"                                                                                               
000000558:   200        1826 L   37305 W    1624121 C   "https://news.google.com/topstories?hl=en-US&gl=US&ceid=US:en"                                                              
                                            h                                                                                                                                       
000000840:   200        2316 L   16996 W    301468 Ch   "https://www.google.com/chrome/"                                                                                            
000000397:   200        519 L    4002 W     191032 Ch   "https://myactivity.google.com/?continue=https://myactivity.google.com/myactivity"                                          
000000710:   200        56 L     6191 W     453302 Ch   "https://www.google.com/saved"                                                                                              

Total time: 1.606515
Processed Requests: 7
Filtered Requests: 0
Requests/sec.: 4.357257
{% endhighlight %}

Al especificar la URL da un error, pero con solo colocar la palabra FUZZ esto se arregla. El payload se especifica con el parámetro *-z* acompañado de la ubicación en donde se encuentra el archivo que guardamos de la sesión anterior. Hay que tener en cuenta de que las respuestas no están en el mismo orden que antes.

Este payload es muy situacional, por lo tanto no es necesario indagar en él demasiado, aunque está bueno conocerlos todos porque pueden llegar a ser necesarios en algún momento. Ahora vamos a utilizar dos de ellos, los cuales en mi opinión son los más útiles.

## Payloads

Wfuzz también es muy útil para hacer un bruteforcing simple, cuando no queremos complicarnos demasiado ni la situación lo requiere. Vamos a ver un ejemplo:

{% highlight bash %}
wfuzz -c -z file,users.txt -z file,pass.txt --sc 200 -d "name=FUZZ&password=FUZ2Z&autologin=1&enter=Sign+in" http://zipper.htb/zabbix/index.php
{% endhighlight %}

Utilizando un payload **file**, podemos llamar a un archivo con credenciales y con el parámetro *-d* especificamos la información que mandaremos por POST. Cuando colocamos múltiples payloads el primer dato se nombra como FUZZ y luego al resto se le añade un número a partir de 2. Luego con *--sc* solo obtendremos las respuestas con el código que hayamos ingresado, que en este caso es 200 (Ok). Tambien he añadido el parámetro *-c*, que muestra el output con colores y queda mucho mejor.

Ahora vamos a ver el otro payload:

{% highlight bash %}
wfuzz -c -z range,0-10 --sc 200,301 "http://zipper.htb/zabbix/index.php?id=FUZZ"
{% endhighlight %}

En este payload **range** utilizaremos GET, para esto solo tenemos que colocar la palabra clave **FUZZ** dentro de la URL en el parámetro que corresponda y especificar el rango de números con el cual queremos hacer el fuzzing. He añadido el código de respuesta *301* pero esto es sólo para mostrarles que se pueden añadir varios datos, no es necesario.

### Para terminar...

Vamos a ver otros parámetros muy útiles, los cuales todavía no he nombrado:

*-f*: Guardar el escaneo dentro de un archivo leíble.

*-p*: Especificar un proxy. Esto es muy util por ejemplo cuando queremos ver las peticiones a través de Burpsuite.

*-t*: Especificar la cantidad de threads con los cuales el escaneo se realizará. Hay que tener cuidado con este parámetro ya que un numero muy alto puede llegar a saturar el servidor.

*-R*: Escoger un nivel de recursión. Este número será la profundidad de directorios en los cuales haremos fuzzing. Por ejemplo con un número 2 no solo hará fuzzing al directorio raíz sino tomará los casos encontrados y hará un escaneo dentro de ellos.

*-b*: Añadir una cookie a las peticiones. Podemos colocar varias cookies repitiendo el mismo parámetro varias veces.

*-H*: Ajustar los headers, por ejemplo el User Agent, Host, etc.
