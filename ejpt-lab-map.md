# eJPT Lab Map

Mappa del laboratorio personale per preparazione eJPT.

> Tutte le attività sono svolte esclusivamente in ambiente locale autorizzato.

---

## 1. Obiettivo del laboratorio

Questo laboratorio è stato creato per allenare le competenze richieste dall’esame eJPT:

```text
Metodologie di assessment
Host discovery
Port scanning
Service enumeration
Host and network penetration testing
Web application penetration testing
Post-exploitation
Privilege escalation
Credential hunting
Hash/password collection
File transfer
Pivoting
Port forwarding
```

---

## 2. Schema generale

```text
Kali Linux
↓
Rete laboratorio 192.168.1.0/24
↓
Target 1: Metasploitable2
Target 2: OWASP-BWA
Target 3: Ubuntu Target3
```

---

## 3. Rete laboratorio

Subnet principale:

```text
192.168.1.0/24
```

Macchine rilevate con:

```bash
nmap -sn 192.168.1.0/24
```

Output importante:

```text
192.168.1.1     Host VirtualBox / gateway laboratorio
192.168.1.100   Metasploitable2
192.168.1.101   Kali Linux
192.168.1.102   OWASP-BWA
192.168.1.104   Target3-Ubuntu
```

---

## 4. Tabella macchine

| IP            | Host               | Ruolo          | Note                                                                     |
| ------------- | ------------------ | -------------- | ------------------------------------------------------------------------ |
| 192.168.1.101 | Kali Linux         | Attaccante     | Macchina usata per scansione, enumeration, exploit e Burp Suite          |
| 192.168.1.100 | Metasploitable2    | Target 1       | Servizi vulnerabili, Metasploit, NFS, SMB, MySQL, privilege escalation   |
| 192.168.1.102 | OWASP-BWA          | Target 2       | Web application testing, RailsGoat, WordPress, Joomla, phpBB, phpMyAdmin |
| 192.168.1.104 | Target3-Ubuntu     | Target 3       | SSH, file transfer, pivoting, port forwarding, routing                   |
| 192.168.1.1   | VirtualBox gateway | Infrastruttura | Host/gateway della rete laboratorio                                      |

---

# Target 1 - Metasploitable2

## 5. Informazioni generali

```text
IP: 192.168.1.100
Hostname: metasploitable
Sistema operativo: Linux 2.6.x
Ruolo: Target host/network principale
```

## 6. Full port scan

Comando:

```bash
nmap -p- -T4 192.168.1.100
```

Porte aperte trovate:

```text
21/tcp    open  ftp
22/tcp    open  ssh
23/tcp    open  telnet
25/tcp    open  smtp
53/tcp    open  domain
80/tcp    open  http
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
512/tcp   open  exec
513/tcp   open  login
514/tcp   open  shell
1099/tcp  open  rmiregistry
1524/tcp  open  ingreslock
2049/tcp  open  nfs
2121/tcp  open  ccproxy-ftp
3306/tcp  open  mysql
3632/tcp  open  distccd
5432/tcp  open  postgresql
5900/tcp  open  vnc
6000/tcp  open  X11
6667/tcp  open  irc
6697/tcp  open  ircs-u
8009/tcp  open  ajp13
8180/tcp  open  unknown
```

## 7. Service enumeration

Comando:

```bash
nmap -sC -sV -A -p 21,22,23,25,53,80,111,139,445,2049,3306,3632,5432,5900,6667,8009,8180 192.168.1.100
```

Servizi principali:

```text
21/tcp    FTP         vsftpd 2.3.4
22/tcp    SSH         OpenSSH 4.7p1 Debian
23/tcp    Telnet      Linux telnetd
25/tcp    SMTP        Postfix smtpd
53/tcp    DNS         ISC BIND 9.4.2
80/tcp    HTTP        Apache 2.2.8 Ubuntu DAV/2
139/tcp   SMB         Samba
445/tcp   SMB         Samba 3.0.20-Debian
2049/tcp  NFS
3306/tcp  MySQL       MySQL 5.0.51a
3632/tcp  distccd     distccd v1
5432/tcp  PostgreSQL
5900/tcp  VNC
6667/tcp  IRC         UnrealIRCd
8009/tcp  AJP13       Apache JServ Protocol
8180/tcp  HTTP        Apache Tomcat/5.5
```

## 8. Target 1 - Priorità di analisi

Priorità alta:

```text
FTP vsftpd 2.3.4
SMB Samba 3.0.20
NFS 2049
MySQL 3306
distccd 3632
HTTP 80
Tomcat 8180
UnrealIRCd 6667
```

## 9. Attività già svolte su Metasploitable2

```text
FTP anonymous login
SMB enumeration
Web enumeration
WebDAV enumeration
NFS mount
MySQL credential disclosure
DVWA SQL Injection
Burp Suite Repeater
Metasploit vsftpd exploitation
Metasploit distccd exploitation
SUID nmap privilege escalation
Post-exploitation enumeration
Lettura /etc/passwd
Lettura /etc/shadow
Credential hunting
```

---

# Target 2 - OWASP-BWA

## 10. Informazioni generali

```text
IP: 192.168.1.102
Hostname: OWASPBWA
Sistema operativo: Linux 2.6.x
Ruolo: Target web application principale
```

## 11. Full port scan

Comando:

```bash
nmap -p- -T4 192.168.1.102
```

Porte aperte:

```text
22/tcp    open  ssh
80/tcp    open  http
139/tcp   open  netbios-ssn
143/tcp   open  imap
443/tcp   open  https
445/tcp   open  microsoft-ds
5001/tcp  open  commplex-link
8080/tcp  open  http-proxy
8081/tcp  open  blackice-icecap
```

## 12. Service enumeration

Comando:

```bash
nmap -sC -sV -A -p 22,80,139,143,443,445,5001,8080,8081 192.168.1.102
```

Servizi principali:

```text
22/tcp    SSH       OpenSSH 5.3p1 Debian 3ubuntu4
80/tcp    HTTP      Apache 2.2.14 Ubuntu
139/tcp   SMB       Samba
143/tcp   IMAP      Courier Imapd
443/tcp   HTTPS     Apache 2.2.14 Ubuntu
445/tcp   SMB       Samba
5001/tcp  Java Object Serialization
8080/tcp  HTTP      Apache Tomcat/Coyote JSP engine 1.1
8081/tcp  HTTP      Jetty 6.1.25
```

## 13. Web fingerprinting

Comandi:

```bash
whatweb http://192.168.1.102
whatweb https://192.168.1.102
whatweb http://192.168.1.102:8080
whatweb http://192.168.1.102:8081
```

Risultati importanti:

```text
Apache 2.2.14
PHP 5.3.2
mod_mono
mod_python
mod_perl
Phusion Passenger 4.0.38
Jetty 6.1.25
Tomcat/Coyote
```

## 14. Directory enumeration porta 80

Comando:

```bash
gobuster dir -u http://192.168.1.102 -w /usr/share/wordlists/dirb/common.txt
```

Risultati interessanti:

```text
.bash_history
cgi-bin/
crossdomain.xml
evil/
gallery2/
joomla/
phpBB2/
phpmyadmin/
test/
wordpress/
```

## 15. Directory enumeration porta 8081

Comando:

```bash
gobuster dir -u http://192.168.1.102:8081 -w /usr/share/wordlists/dirb/common.txt
```

Risultati interessanti:

```text
/admin
/css
/images
/index.html
/js
/ws
```

## 16. SMB enumeration

Comando:

```bash
smbclient -L //192.168.1.102 -N
```

Share trovate:

```text
print$      Printer Drivers
apache      Apache Web Server Root
tomcat      Tomcat6 Root
var         /var
etc         /etc
usr         /usr
owaspbwa    /owaspbwa
IPC$
```

Nota:

```text
Le share sono visibili ma l’accesso anonymous risulta negato.
Se in futuro vengono trovate credenziali, riprovare accesso SMB.
```

## 17. File interessante .bash_history

Comando:

```bash
curl http://192.168.1.102/.bash_history
```

Output:

```text
/usr/local/rvm/bin/rvm user gemsets
/usr/local/rvm/wrappers/ruby-1.9.3-p484@railsgoat/ruby /usr/local/rvm/gems/ruby-1.9.3-p484@railsgoat/gems/passenger-4.0.38/bin/passenger-config --detect-ruby
/usr/local/rvm/gems/ruby-1.9.3-p484@railsgoat/gems/passenger-4.0.38/bin/passenger-config --detect-ruby
exit
```

Interpretazione:

```text
Il file .bash_history è accessibile via web.
Non contiene password, ma contiene un riferimento importante a RailsGoat.
Questo ha portato alla scoperta dell’applicazione /railsgoat/.
```

## 18. RailsGoat

URL:

```text
http://192.168.1.102/railsgoat/
```

Credenziali tutorial trovate:

```text
admin@metacorp.com / admin1234
jack@metacorp.com  / yankeesSuck
jim@metacorp.com   / alohaowasp
mike@metacorp.com  / motocross1445
ken@metacorp.com   / citrusblend
```

A1 Injection - codice vulnerabile individuato:

```ruby
user = User.find(:first, :conditions => "user_id = '#{params[:user][:user_id]}'")
```

Parametro interessante:

```text
params[:user][:user_id]
```

Nota:

```text
Il prossimo step su RailsGoat sarà intercettare con Burp una richiesta di update profilo/account e analizzare il parametro user[user_id].
```

## 19. Target 2 - Priorità di analisi

Priorità alta:

```text
/railsgoat/
/wordpress/
/joomla/
/phpBB2/
/phpmyadmin/
/cgi-bin/
porta 8081 /admin
porta 8080 Tomcat
SMB con credenziali se trovate
```

---

# Target 3 - Ubuntu

## 20. Informazioni generali

```text
IP: 192.168.1.104
Hostname: target3
Utente: spywer
Sistema operativo: Ubuntu
Ruolo: Target per SSH, file transfer, pivoting e port forwarding
```

## 21. Port scan

Comando:

```bash
nmap -p- -T4 192.168.1.104
```

Risultato:

```text
22/tcp open ssh
```

## 22. Service enumeration

Comando:

```bash
nmap -sC -sV -p 22 192.168.1.104
```

Risultato:

```text
22/tcp open ssh OpenSSH 10.2p1 Ubuntu 2ubuntu3
```

## 23. Verifica SSH

Comando:

```bash
ssh spywer@192.168.1.104
```

Risultato:

```text
Accesso SSH riuscito.
Prompt ottenuto: spywer@target3:~$
```

## 24. Snapshot Target3

Snapshot creato in VirtualBox:

```text
Clean install - SSH working
```

Descrizione:

```text
Ubuntu Target3 installata.
Utente spywer creato.
SSH attivo e accesso funzionante su 192.168.1.104.
```

## 25. Target 3 - Utilizzo previsto

Target3 verrà usata per:

```text
SSH practice
file transfer
post-exploitation base
pivoting
port forwarding
routing
test rete interna
```

---

# 26. Allineamento con domini eJPT

## Test di penetrazione host e rete

Coperto da:

```text
Metasploitable2
Metasploit
vsftpd
distccd
Samba
NFS
MySQL
PostgreSQL
Tomcat
UnrealIRCd
```

## Metodologie di valutazione

Coperto da:

```text
nmap -sn
nmap -p-
nmap -sC -sV -A
whatweb
curl
gobuster
smbclient
enum4linux
```

## Auditing host e rete

Coperto da:

```text
whoami
id
hostname
uname -a
ip a
cat /etc/passwd
cat /etc/shadow
find SUID
lettura file di configurazione
credential hunting
hash/password collection
```

## Web application penetration testing

Coperto da:

```text
DVWA
Burp Suite
SQL Injection
RailsGoat
OWASP-BWA
WordPress
Joomla
phpBB2
phpMyAdmin
directory enumeration
```

---

# 27. Cose ancora da allenare

```text
Pivoting
Port forwarding
Aggiunta route
Brute force controllato
Hash cracking
File transfer ordinato
Modifica exploit
Simulazione a tempo
```

---

# 28. Prossima strategia

Ordine consigliato:

```text
1. Consolidare Metasploitable2 come target host/network
2. Consolidare OWASP-BWA come target web
3. Configurare Target3 per pivoting
4. Allenare file transfer
5. Allenare hash cracking
6. Allenare brute force controllato
7. Fare simulazione finale stile eJPT
```

---

# 29. Regola metodologica

```text
Non partire dall’exploit.

Prima:
1. Discovery
2. Port scanning
3. Service enumeration
4. Web enumeration
5. Vulnerability identification
6. Exploitation
7. Post-exploitation
8. Privilege escalation
9. Evidence collection
10. Documentation
```

---

# 30. Nota etica

Questo laboratorio è locale e autorizzato.

Le tecniche documentate devono essere usate solo:

```text
in laboratorio personale
in CTF
in ambienti autorizzati
in esami pratici
in attività professionali con permesso scritto
```

Non utilizzare questi comandi contro sistemi reali senza autorizzazione esplicita.
