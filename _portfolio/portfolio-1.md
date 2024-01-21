---
title: "Hack The Box - Access"
excerpt: "<img src='/images/portfolio/access/cover.webp' width='500' height=auto><br/>Access is an 'Easy' difficulty machine, that highlights how machines associated with the physical security of an environment may not themselves be secure. Also highlighted is how accessible FTP/file shares can often lead to getting a foothold or lateral movement. It teaches techniques for identifying and exploiting saved credentials."
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

Si trato de conectarme a la máquina a través del puerto 80 desde el navegador obtengo lo siguiente.<br/>
<img src='/images/portfolio/access/http.png' width='700' height=auto><br/>
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
    
<br/>Solo consigo encontrar la ruta /index.html por lo que no parece que haya nada interesante respecto al sitio web.
