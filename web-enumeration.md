# Web Enumeration Notes

Appunti personali su web enumeration, fingerprinting e raccolta informazioni per laboratori eJPT.

> Tutti i comandi sono da usare solo in ambienti autorizzati e controllati.

---

## 1. Web Fingerprinting

Obiettivo: identificare tecnologie, server web, versioni, header HTTP e possibili superfici interessanti.

```bash
whatweb http://TARGET
curl -I http://TARGET
```

Esempio su Metasploitable2:

```bash
whatweb http://192.168.1.100
curl -I http://192.168.1.100
```

Informazioni utili da cercare:

* Server web
* Versione Apache/Nginx
* Versione PHP
* Header interessanti
* Cookie
* Redirect
* WebDAV
* Pagine di login
* Directory o applicazioni esposte

---

## 2. HTTP Methods

Obiettivo: verificare quali metodi HTTP sono abilitati sul server web.

```bash
nmap --script http-methods -p 80 TARGET
```

Esempio:

```bash
nmap --script http-methods -p 80 192.168.1.100
```

Risultato esempio:

```text
Supported Methods: GET HEAD POST OPTIONS
```

Interpretazione:

```text
GET/POST/HEAD/OPTIONS = comportamento normale
PUT presente = possibile upload da verificare solo in laboratorio autorizzato
DELETE presente = configurazione pericolosa
```

---

## 3. Directory Enumeration

Obiettivo: trovare directory e file nascosti.

```bash
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt
```

Esempio:

```bash
gobuster dir -u http://192.168.1.100 -w /usr/share/wordlists/dirb/common.txt
```

Directory interessanti:

* `/admin`
* `/login`
* `/backup`
* `/config`
* `/uploads`
* `/dav`
* `/phpMyAdmin`
* `/phpinfo.php`
* `/test`
* `/twiki`
* `/cgi-bin`
* `/server-status`

---

## 4. phpinfo.php Enumeration

Obiettivo: raccogliere informazioni utili dalla pagina `phpinfo.php`.

```bash
curl -s http://TARGET/phpinfo.php | grep -i "document_root"
curl -s http://TARGET/phpinfo.php | grep -i "server_software"
curl -s http://TARGET/phpinfo.php | grep -i "disable_functions"
```

Esempio:

```bash
curl -s http://192.168.1.100/phpinfo.php | grep -i "document_root"
curl -s http://192.168.1.100/phpinfo.php | grep -i "server_software"
curl -s http://192.168.1.100/phpinfo.php | grep -i "disable_functions"
```

Cosa cercare:

* Document root
* Versione PHP
* Server software
* Funzioni PHP disabilitate
* Path locali
* Moduli caricati
* Variabili ambiente

Risultati esempio su Metasploitable2:

```text
DOCUMENT_ROOT: /var/www/
SERVER_SOFTWARE: Apache/2.2.8 (Ubuntu) DAV/2
disable_functions: no value
```

Interpretazione:

```text
DOCUMENT_ROOT /var/www/ = percorso principale dei file web
Apache/2.2.8 = server web vecchio
PHP 5.2.4 = versione PHP vecchia
disable_functions no value = funzioni PHP non disabilitate
```

---

## 5. phpMyAdmin Check

Obiettivo: verificare se phpMyAdmin è raggiungibile.

```bash
curl -I http://TARGET/phpMyAdmin/
```

Esempio:

```bash
curl -I http://192.168.1.100/phpMyAdmin/
```

Se risponde:

```text
HTTP/1.1 200 OK
Set-Cookie: phpMyAdmin=...
Content-Type: text/html
```

Significa:

```text
phpMyAdmin è raggiungibile
la pagina probabilmente presenta un login
sono presenti cookie di sessione
```

Aprire nel browser:

```text
http://TARGET/phpMyAdmin/
```

Esempio:

```text
http://192.168.1.100/phpMyAdmin/
```

Prima di usare tecniche più invasive, provare solo credenziali note, deboli o trovate durante l’enumerazione.

Esempi di controllo manuale in laboratorio:

```text
root / root
root / password
root / vuoto
admin / admin
```

Se l’accesso non funziona:

```text
Non insistere su phpMyAdmin.
Passare a credential hunting tramite altri servizi come SMB, NFS, FTP, database o file web.
```

---

## 6. WebDAV

Se `whatweb`, `curl -I` o gli header mostrano `DAV/2`, verificare i metodi HTTP supportati.

Controllo generale sulla porta 80:

```bash
nmap --script http-methods -p 80 TARGET
```

Controllo specifico sulla directory `/dav/`:

```bash
curl -i -X OPTIONS http://TARGET/dav/
nmap --script http-methods --script-args http-methods.url-path=/dav/ -p 80 TARGET
```

Esempio su Metasploitable2:

```bash
curl -i -X OPTIONS http://192.168.1.100/dav/
nmap --script http-methods --script-args http-methods.url-path=/dav/ -p 80 192.168.1.100
```

Interpretazione:

```text
Se PUT è abilitato → possibile upload da verificare solo in laboratorio autorizzato.
Se PUT non è presente → WebDAV esiste, ma upload diretto non disponibile sulla porta testata.
```

Regola mentale:

```text
Vedo DAV/2 oppure /dav/ → controllo i metodi HTTP.
```

---

## 7. Esempio Metasploitable2

Target:

```text
192.168.1.100
```

Comandi usati:

```bash
whatweb http://192.168.1.100
curl -I http://192.168.1.100
nmap --script http-methods -p 80 192.168.1.100
gobuster dir -u http://192.168.1.100 -w /usr/share/wordlists/dirb/common.txt
curl -I http://192.168.1.100/phpMyAdmin/
curl -s http://192.168.1.100/phpinfo.php | grep -i "document_root"
curl -s http://192.168.1.100/phpinfo.php | grep -i "server_software"
curl -s http://192.168.1.100/phpinfo.php | grep -i "disable_functions"
curl -i -X OPTIONS http://192.168.1.100/dav/
nmap --script http-methods --script-args http-methods.url-path=/dav/ -p 80 192.168.1.100
```

Risultati importanti:

```text
Apache/2.2.8 (Ubuntu) DAV/2
PHP/5.2.4
DOCUMENT_ROOT: /var/www/
phpMyAdmin raggiungibile
disable_functions: no value
```

Directory trovate con Gobuster:

```text
/dav
/phpMyAdmin
/phpinfo.php
/test
/twiki
/cgi-bin
/server-status
```

---

## 8. Ragionamento eJPT

Flusso corretto:

```text
1. Identifico il servizio web
2. Raccolgo header e tecnologie
3. Controllo metodi HTTP
4. Cerco directory e file nascosti
5. Analizzo pagine interessanti
6. Cerco credenziali o configurazioni
7. Se non trovo accesso, passo ad altri servizi
8. Solo dopo scelgo eventuale exploit
```

Regola importante:

```text
Enumeration first, exploitation later.
```

---

## 9. Decisione pratica

Esempio di ragionamento su Metasploitable2:

```text
FTP anonymous → accesso riuscito ma directory vuota
HTTP → Apache/PHP/WebDAV
Gobuster → phpMyAdmin, phpinfo.php, /dav, /twiki, /test
phpMyAdmin → raggiungibile ma login non riuscito
/dav → controllare metodi HTTP
Se non ottengo accesso dal web → passare a SMB enumeration
```

Quando il web non porta accesso diretto:

```text
Passare a SMB, NFS, database, SSH/Telnet o servizi con versioni vulnerabili.
```
