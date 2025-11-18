
# Introducción
Máquina temática sobre Pokémon, con enfoque en la enumeración web y acceso vía SSH. Dificultad media. El objetivo es encontrar varias "flags" con nombres de Pokémon y escalar privilegios hasta obtener acceso root.
# Reconocimiento
Como todo vamos a usar nmap para ver los puertos que tiene abiertos esta máquina.
```shell
root@ip-10-10-38-214:~# nmap -sS -sV -sC -Pn 10.10.29.147
Starting Nmap 7.80 ( https://nmap.org ) at 2025-07-26 12:10 BST
Nmap scan report for ip-10-10-29-147.eu-west-1.compute.internal (10.10.29.147)
Host is up (0.00033s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 58:14:75:69:1e:a9:59:5f:b2:3a:69:1c:6c:78:5c:27 (RSA)
|   256 23:f5:fb:e7:57:c2:a5:3e:c2:26:29:0e:74:db:37:c2 (ECDSA)
|_  256 f1:9b:b5:8a:b9:29:aa:b6:aa:a2:52:4a:6e:65:95:c5 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Can You Find Them All?
MAC Address: 02:D9:BA:DB:14:6B (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.06 seconds
```

Vemos que la maquina tiene abierto dos puertos el 22 (ssh) con la versión "- OpenSSH 7.2p2" y el puerto 80 (HTTP) con la versión "Apache 2.4.18" veo que lo mas importante de aquí de momento es el puerto 80 ya que no podemos hacer nada en el ssh sin tener ningún usuario ni contraseña, por lo que dicho esto vamos a ver que hay dentro de la pagina web.

# Enumeración y análisis web

![[Pasted image 20250726121112.png]]

Vemos que al entrar al servicio web lo primero que noto es que el título no es el usual sino simple y llanamente `Apache2` eso no es normal, por lo que primero que nada y para empezar a descartar opciones voy a usar  *gobuster* o *dirsearch* para ver si hay algún directorio oculto más por ahí que no quieren que veamos.

En lo que se dirsearch hacia su trabajo me pongo a curiosear el código fuente en busca de algún detalle y me percato que en el código interno del navegador hay una parte que no es usual de la pagina de inicio, al final del código vi un detalle que puede ser nuestro primer vector de entrada.

![[Pasted image 20250726121705.png]]

>[!Failure] Referente a dirsearch veo que no da ningún resultado por lo que creo que voy a usar mejor gobuster, porque no parece encontrar nada y estoy más familiarizado con el.

>[!Warning] En este momento del reporte, no me había percatado que la primera línea de la foto podía ser un usuario y contraseña de un usuario ssh.

Por lo que siguiendo lo que nos han dicho en el código fuente vamos a ir a la consola para ver que nos encontramos.

![[Pasted image 20250726121822.png]]

Vemos que en la consola lo único que nos encontramos son nombres de Pokémons, voy a ver que puedo hacer con ello y que es lo que más puedo encontrar. Dándole vueltas a la pagina no encontré nada, me quede atascado, y me di cuente que estaba en una calle sin salida me puse a ver un walkthrough y no me percate que en la imagen anterior me daba más información de la que pensaba, ya que el primer `<>` podía ser un nombre de usuario y el segundo podía ser una contraseña, como se percataron por ello, principalmente por el *:* que hay entre las etiquetas. Por lo que vamos a iniciar sesión con ese usuario y vemos que podemos hacer en el ssh

![[Pasted image 20250726123909.png]]

# Post-Explotación

Ahora ya que estamos dentro, vamos a ver que podemos encontrar en este usuario

```SHELL
pokemon@root:~$ ls Desktop/
P0kEmOn.zip
pokemon@root:~$ unzip Desktop/P0kEmOn.zip 
Archive:  Desktop/P0kEmOn.zip
   creating: P0kEmOn/
  inflating: P0kEmOn/grass-type.txt  
pokemon@root:~$ ls Desktop/
P0kEmOn.zip
pokemon@root:~$ ls
Desktop  Documents  Downloads  examples.desktop  Music  P0kEmOn  Pictures  Public  Templates  Videos
pokemon@root:~$ cd P0kEmOn/
pokemon@root:~/P0kEmOn$ ls
grass-type.txt
pokemon@root:~/P0kEmOn$ cat grass-type.txt 
50 6f 4b 65 4d 6f 4e 7b 42 75 6c 62 61 73 61 75 72 7dpokemon@root:~/P0kEmOn$
```

Investigando el usuario entorno del usuario veo que dentro de su escritorio hay un comprimido `.zip`, lo descomprimimos y nos ha dado un archivo `grass-type.txt` y en su interior un texto hexadecimal, lo cual pasamos por cualquier decodificador y tendríamos "PoKeMoN{Bulbasaur}". Esto es solo la primera flag, por lo que vamos a seguir investigando. el usuario.

```shell
pokemon@root:~$ ls Videos/
Gotta
pokemon@root:~$ ls Videos/Gotta/Catch/Them/ALL\!/Could_this_be_what_Im_looking_for\?.cplusplus 
Videos/Gotta/Catch/Them/ALL!/Could_this_be_what_Im_looking_for?.cplusplus
pokemon@root:~$ ls Videos/Gotta/
Catch
pokemon@root:~$ ls Videos/Gotta/Catch/
Them
pokemon@root:~$ ls Videos/Gotta/Catch/Them/
ALL!
pokemon@root:~$ ls Videos/Gotta/Catch/Them/ALL\!/
Could_this_be_what_Im_looking_for?.cplusplus
pokemon@root:~$ cat Videos/Gotta/Catch/Them/ALL\!/Could_this_be_what_Im_looking_for\?.cplusplus 
# include <iostream>

int main() {
	std::cout << "ash : pikapika"
	return 0;
```

Analizando las otras carpetas vemos que en la carpeta `Videos` hay una carpeta que no es usual, vamos a ver que hay dentro de ella, vemos que hay mas carpetas vamos a llegar al final viendo que no hay ninguna otra por el camino. Al final de estas carpetas hay un archivo `.cplusplus` que en su interior vemos que se vuelve a repetir la estructura del principio, por lo que sabemos que tenemos otro potencial usuario, vamos a probarlo.

```shell
root@ip-10-10-38-214:~# ssh ash@10.10.29.147
ash@10.10.29.147's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

84 packages can be updated.
0 updates are security updates.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Could not chdir to home directory /home/ash: Permission denied
$
```

Perfecto, vemos que estamos con el usuario `ash`, vamos a ver que puede hacer.

```
$ sudo -l
[sudo] password for ash: 
Matching Defaults entries for ash on root:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User ash may run the following commands on root:
    (ALL : ALL) ALL
```

Espera, tenemos root 😁?, si!. Bueno ya si queremos podemos tener root con el comando `sudo su`  por lo que ya podemos analizar todo sin problemas mayores, no hace falta ni siquiera una escalada de privilegios. Vamos a ver donde están todas las otras flags.

```shell
$ ls home
ash  pokemon  roots-pokemon.txt
$ ls home/ash
ls: cannot open directory 'home/ash': Permission denied
$ ls home/pokemon
Desktop  Documents  Downloads  examples.desktop  Music	P0kEmOn  Pictures  Public  Templates  Videos
$ ls home/roots-pokemon.txt
home/roots-pokemon.txt
$ cat home/roots-pokemon.txt
Pikachu!
```

Viendo el home de ash, vemos que hay una posible flag, vemos lo que hay en el archivo y vemos que es la la última flag, flag es flag! tenemos la ultima bandera xD.

```shell
ash@root:/$ cat /var/www/html/water-type.txt 
Ecgudfxq_EcGmP{Ecgudfxq}
```

>[!tip] Me mejore la shell con "python -c 'import pty; pty.spawn("/bin/bash")'"

Viendo los demas archivo vemos que en en el directorio de la pagina web esta la siguiente bandera, esta esta codificada por lo que vamos a llevarla a un identificador de cifrado y vemos que es rot. Después lo vamos a llevar a ciberchef para que nos lo pase a texto claro, y en root 14 nos da " Squirtle_SqUaD{Squirtle}". Genial la segunda bandera, vamos a por la última.

```shell
ash@root:/$ locate *-type.txt
/etc/why_am_i_here?/fire-type.txt
/var/www/html/water-type.txt
ash@root:/$ cat /etc/why_am_i_here\?/fire-type.txt 
UDBrM20wbntDaGFybWFuZGVyfQ==
```

Ya recordando que como soy root puedo ejecutar cualquier comando ejecute `locate` y como ya sabia que la flag iba a terminar con `-type.txt` lo busque de esta forma y me dio que en `/etc/why_am_i_here?` hay una flag vemos su interior y lo lleve a un decodificador de base64 y me dio "P0k3m0n{Charmander}". Y así ya tendríamos todo listo.

## Flags
- Usuario: Pokemon
	- `P0kEmOn.zip` → `grass-type.txt` → hex → `PoKeMoN{Bulbasaur}`
- Usuario: Ash (Credenciales sacadas del archivo `.cplusplus` → `ash:pikapika`)
	- - `/var/www/html/water-type.txt` → ROT → `Squirtle_SqUaD{Squirtle}`
	- `/etc/why_am_i_here?/fire-type.txt` → Base64 → `P0k3m0n{Charmander}`
	- `/home/roots-pokemon.txt` → `Pikachu!`
# 🧠 Conclusión

- Acceso inicial mediante pistas en el código fuente y consola web.
    
- Explotación de credenciales expuestas.
    
- Buen uso de herramientas de decodificación como CyberChef.
	
- Cosas que aprender
	1. Tengo que recordar mejorar la shell siempre que pueda para mayor comodidad.
	2. Tengo que pensar más a menudo "fuera de la caja" ya que esto me ayuda a salir de un aparente callejón sin salida.