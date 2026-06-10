# eJPT Lab - Host Auditing Metasploitable2

## Informazioni Target

* Target: Metasploitable2
* IP: 192.168.1.100
* Attacker: Kali Linux 192.168.1.101
* Credenziali iniziali: msfadmin / msfadmin

## Obiettivo

Completare gli obiettivi eJPT della sezione Host and Network Auditing:

* Enumerazione sistema
* Enumerazione utenti
* Analisi file sensibili
* Individuazione credenziali
* Accesso database
* Estrazione hash
* Cracking password
* Valutazione criticità

---

## Accesso SSH

Poiché Metasploitable2 utilizza algoritmi SSH obsoleti, da Kali è stato necessario utilizzare:

```bash
ssh -oHostKeyAlgorithms=+ssh-rsa \
-oPubkeyAcceptedKeyTypes=+ssh-rsa \
msfadmin@192.168.1.100
```

Password:

```text
msfadmin
```

---

## Enumerazione Sistema

Comandi utilizzati:

```bash
hostname
whoami
id
uname -a
cat /etc/issue
```

Risultati:

```text
Hostname: metasploitable
Utente: msfadmin
Kernel: Linux 2.6.24-16-server i686
```

---

## Enumerazione Utenti

Comando:

```bash
grep "/bin/bash" /etc/passwd
```

Utenti con shell interattiva:

```text
root
msfadmin
postgres
user
service
```

Verifica file shadow:

```bash
ls -l /etc/shadow
```

Risultato:

```text
-rw-r----- 1 root shadow
```

Valutazione:

Il file contenente gli hash delle password è protetto e richiede privilegi elevati.

---

## Enumerazione File Web

Comando:

```bash
find /var/www -name "*.php" | head -30
```

File interessanti trovati:

```text
/var/www/dvwa/login.php
/var/www/dvwa/setup.php
/var/www/dvwa/security.php
/var/www/dvwa/vulnerabilities/brute/
/var/www/dvwa/vulnerabilities/fi/
/var/www/dvwa/vulnerabilities/sqli/
/var/www/dvwa/vulnerabilities/sqli_blind/
```

Tecniche individuate:

* Brute Force
* File Inclusion
* SQL Injection
* Blind SQL Injection

---

## Ricerca Credenziali

Comandi:

```bash
grep -Ri "password" /var/www 2>/dev/null | head -50
grep -Ri "mysql" /var/www 2>/dev/null | head -50
```

Ricerca file di configurazione:

```bash
find /var/www/dvwa -name "*config*" 2>/dev/null
```

Risultati:

```text
/var/www/dvwa/config/config.inc.php
/var/www/dvwa/config/config.inc.php~
```

---

## Credenziali Database DVWA

Comando:

```bash
cat /var/www/dvwa/config/config.inc.php
```

Configurazione trovata:

```php
$_DVWA['db_server']   = 'localhost';
$_DVWA['db_database'] = 'dvwa';
$_DVWA['db_user']     = 'root';
$_DVWA['db_password'] = '';
```

Credenziali individuate:

```text
Database: dvwa
Utente: root
Password: vuota
```

Criticità: Alta

Impatto:

Un attaccante con accesso locale può autenticarsi a MySQL senza password.

---

## Accesso MySQL

Comando:

```bash
mysql -u root
```

Enumerazione database:

```sql
show databases;
```

Database trovati:

```text
information_schema
dvwa
metasploit
mysql
owasp10
tikiwiki
tikiwiki195
```

Selezione database DVWA:

```sql
use dvwa;
show tables;
```

Tabelle trovate:

```text
guestbook
users
```

Estrazione utenti e hash:

```sql
select user,password from users;
```

Hash ottenuti:

```text
admin   5f4dcc3b5aa765d61d8327deb882cf99
gordonb e99a18c428cb38d5f260853678922e03
1337    8d3533d75ae2c3966d7e0d4fcc69216b
pablo   0d107d09f5bbe40cade3de5c71e9e9b7
smithy  5f4dcc3b5aa765d61d8327deb882cf99
```

---

## Preparazione File Hash

Su Kali:

```bash
cat > dvwa-hashes.txt << 'HASHES'
admin:5f4dcc3b5aa765d61d8327deb882cf99
gordonb:e99a18c428cb38d5f260853678922e03
1337:8d3533d75ae2c3966d7e0d4fcc69216b
pablo:0d107d09f5bbe40cade3de5c71e9e9b7
smithy:5f4dcc3b5aa765d61d8327deb882cf99
HASHES
```

---

## Cracking Password con John

Comando:

```bash
john --format=Raw-MD5 dvwa-hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Visualizzazione risultati:

```bash
john --show --format=Raw-MD5 dvwa-hashes.txt
```

Password recuperate:

```text
admin:password
gordonb:abc123
1337:charley
pablo:letmein
smithy:password
```

Risultato:

```text
5 hash crackati su 5
```

---

## Findings

### Finding 1 - Credenziali Database Deboli

```text
Utente: root
Password: vuota
```

Criticità: Alta

Impatto:

Accesso completo al database senza autenticazione.

### Finding 2 - Credenziali Applicative Esposte

```text
File: /var/www/dvwa/config/config.inc.php
```

Criticità: Alta

Impatto:

Le credenziali del database sono memorizzate in chiaro.

### Finding 3 - Password Deboli

```text
admin:password
gordonb:abc123
1337:charley
pablo:letmein
smithy:password
```

Criticità: Alta

Impatto:

Le password vengono crackate immediatamente con wordlist pubbliche.

---

## Obiettivi eJPT Coperti

* Compile information from files on target
* Enumerate system information on target
* Gather user account information on target
* Gather hash/password information from target
* Conduct hash cracking
* Evaluate information and criticality

---

## Conclusioni

Il laboratorio ha dimostrato un ciclo completo di Host Auditing:

1. Accesso SSH al target
2. Enumerazione del sistema
3. Enumerazione utenti
4. Analisi file web
5. Individuazione credenziali
6. Accesso MySQL
7. Estrazione hash
8. Cracking password
9. Valutazione impatto e criticità

Questo laboratorio completa una parte fondamentale del dominio Host and Network Auditing previsto dall'eJPT.
