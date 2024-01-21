---
title: "Hack The Box - Access"
excerpt: "<img src='/images/portfolio/access/cover.webp' width='500' height=auto><br/><br/>Access is an 'Easy' difficulty machine, that highlights how machines associated with the physical security of an environment may not themselves be secure. Also highlighted is how accessible FTP/file shares can often lead to getting a foothold or lateral movement. It teaches techniques for identifying and exploiting saved credentials."
collection: portfolio
---

NMAP
------

Utilizando la herramienta nmap para escaneo de puertos y servicios.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# nmap -A 10.10.10.98
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-21 13:07 CET
    Nmap scan report for 10.10.10.98
    Host is up (0.045s latency).
    Not shown: 997 filtered tcp ports (no-response)
    PORT   STATE SERVICE VERSION
    21/tcp open  ftp     Microsoft ftpd
    | ftp-anon: Anonymous FTP login allowed (FTP code 230)
    |_Can't get directory listing: PASV failed: 425 Cannot open data connection.
    | ftp-syst: 
    |_  SYST: Windows_NT
    23/tcp open  telnet?
    80/tcp open  http    Microsoft IIS httpd 7.5
    |_http-title: MegaCorp
    | http-methods: 
    |_  Potentially risky methods: TRACE
    |_http-server-header: Microsoft-IIS/7.5
    Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
    Device type: specialized|general purpose|phone
    Running (JUST GUESSING): Microsoft Windows 7|8|Phone|2008|8.1|Vista (89%)
    OS CPE: cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_8.1 cpe:/o:microsoft:windows_vista::- cpe:/o:microsoft:windows_vista::sp1
    Aggressive OS guesses: Microsoft Windows Embedded Standard 7 (89%), Microsoft Windows 8.1 Update 1 (89%), Microsoft Windows Phone 7.5 or 8.0 (89%), Microsoft Windows 7 or Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 (85%), Microsoft Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 or Windows 8.1 (85%), Microsoft Windows Server 2008 R2 SP1 (85%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (85%), Microsoft Windows Server 2008 SP1 or Windows Server 2008 R2 (85%)
    No exact OS matches for host (test conditions non-ideal).
    Network Distance: 2 hops
    Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

    TRACEROUTE (using port 80/tcp)
    HOP RTT      ADDRESS
    1   43.18 ms 10.10.14.1
    2   44.01 ms 10.10.10.98

    OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 59.22 seconds

HTTP
------

Si trato de conectarme a la máquina a través del puerto 80 desde el navegador obtengo lo siguiente.<br/><br/>
<img src='/images/portfolio/access/http.png' width='900' height=auto><br/><br/>
Intento obtener algo interesante a través de la herramienta 'gobuster' la cual me permite enumerar archivos y directorios dentro del sitio web.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# gobuster dir --url http://10.10.10.98/ --wordlist /usr/share/wordlists/dirb/common.txt
    ===============================================================
    Gobuster v3.6
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://10.10.10.98/
    [+] Method:                  GET
    [+] Threads:                 10
    [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.6
    [+] Timeout:                 10s
    ===============================================================
    Starting gobuster in directory enumeration mode
    ===============================================================
    /aspnet_client        (Status: 301) [Size: 156] [--> http://10.10.10.98/aspnet_client/]
    /index.html           (Status: 200) [Size: 391]
    Progress: 4614 / 4615 (99.98%)
    ===============================================================
    Finished
    ===============================================================
    
Solo consigo encontrar la ruta /index.html por lo que no parece que haya nada interesante respecto al sitio web.

FTP
------

Ahora toca centrarse en el puerto 21 el cual alberga el servicio FTP. Voy a tratar de conectarme como usuario 'anonymous' a ver si puedo encontrar algo que me interese.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# ftp 10.10.10.98
    Connected to 10.10.10.98.
    220 Microsoft FTP Service
    Name (10.10.10.98:nico): anonymous
    331 Anonymous access allowed, send identity (e-mail name) as password.
    Password: 
    230 User logged in.
    Remote system type is Windows_NT.
    ftp> ls -la
    425 Cannot open data connection.
    200 PORT command successful.
    125 Data connection already open; Transfer starting.
    08-23-18  08:16PM       <DIR>          Backups
    08-24-18  09:00PM       <DIR>          Engineer
    226 Transfer complete.
    ftp> cd Backups
    250 CWD command successful.
    ftp> ls -la
    200 PORT command successful.
    125 Data connection already open; Transfer starting.
    08-23-18  08:16PM              5652480 backup.mdb
    226 Transfer complete.
    ftp> bin
    200 Type set to I.
    ftp> get backup.mdb
    local: backup.mdb remote: backup.mdb
    200 PORT command successful.
    125 Data connection already open; Transfer starting.
    100% |****************************************************************************************************************|  5520 KiB    1.11 MiB/s    00:00 ETA
    226 Transfer complete.
    5652480 bytes received in 00:04 (1.11 MiB/s)
    ftp> cd ../Engineer
    250 CWD command successful.
    ftp> ls -lah
    200 PORT command successful.
    125 Data connection already open; Transfer starting.
    08-24-18  12:16AM                10870 Access Control.zip
    226 Transfer complete.
    ftp> get Access\ Control.zip
    local: Access Control.zip remote: Access Control.zip
    200 PORT command successful.
    125 Data connection already open; Transfer starting.
    100% |****************************************************************************************************************| 10870       83.76 KiB/s    00:00 ETA
    226 Transfer complete.
    10870 bytes received in 00:00 (83.57 KiB/s)
    ftp> quit
    221 Goodbye.
                                                                                                                                                                
    ┌──(root㉿kali)-[/home/nico/htb]
    └─# ll                       
    total 5536
    -rw-r--r-- 1 root root   10870 ago 24  2018 'Access Control.zip'
    -rw-r--r-- 1 root root 5652480 ago 23  2018  backup.mdb
    drwxr-xr-x 2 nico nico    4096 ene 20 11:55  vpn

Se han podido encontrar dos directorios 'Backups' y 'Engineer' y cada uno tenía un fichero con nombre 'Access Control.zip' y 'backup.mdb' respectivamente.<br/><br/>
Ahora se va a investigar un poco acerca de estos archivos en busqueda de algo que me interese.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# file backup.mdb         
    backup.mdb: Microsoft Access Database
                                                                                                                   
    ┌──(root㉿kali)-[/home/nico/htb]
    └─# file Access\ Control.zip 
    Access Control.zip: Zip archive data, at least v2.0 to extract, compression method=AES Encrypted

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# binwalk Access\ Control.zip

    DECIMAL       HEXADECIMAL     DESCRIPTION
    --------------------------------------------------------------------------------
    0             0x0             Zip archive data, encrypted at least v2.0 to extract, compressed size: 10678, uncompressed size: 271360, name: Access Control.pst
    10848         0x2A60          End of Zip archive, footer length: 22

Analizando la salida se sabe que el fichero 'backup.mdb' se trata de un fichero Microsoft Access Database. En cuanto al fichero 'Access Control.zip' se puede ver que está encriptado por lo que de momento no podemos leer su contenido por completo.