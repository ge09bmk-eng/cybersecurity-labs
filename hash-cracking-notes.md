# Hash Cracking Notes - eJPT Lab

Appunti personali sul cracking di hash nel laboratorio eJPT.

> Tutte le attività sono svolte esclusivamente in ambiente locale autorizzato.

---

## 1. Obiettivo

Allenare la raccolta e il cracking di hash/password, competenza richiesta nell’esame eJPT.

Obiettivi coperti:

```text
Raccogliere informazioni hash/password dal target
Condurre cracking dell'hash
Usare wordlist
Interpretare il risultato del cracking
```

---

## 2. Hash usato nel laboratorio

Hash MD5 estratto durante il laboratorio DVWA:

```text
5f4dcc3b5aa765d61d8327deb882cf99
```

Questo hash era stato ottenuto dalla tabella `users` del database DVWA.

---

## 3. Creazione file hash

Comando:

```bash
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hashes.txt
```

Verifica:

```bash
cat hashes.txt
```

Output:

```text
5f4dcc3b5aa765d61d8327deb882cf99
```

---

## 4. Primo tentativo con John

Comando:

```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

Errore iniziale:

```text
fopen: /usr/share/wordlists/rockyou.txt: No such file or directory
```

Interpretazione:

```text
John funzionava correttamente.
L'hash era stato caricato correttamente.
Il problema era la wordlist rockyou.txt mancante o non estratta.
```

---

## 5. Controllo wordlist

Comando:

```bash
ls -la /usr/share/wordlists/
```

Se presente `rockyou.txt.gz`, estrarre con:

```bash
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
```

Verifica:

```bash
ls -la /usr/share/wordlists/rockyou.txt
```

Se la wordlist non è presente, installare il pacchetto:

```bash
sudo apt update
sudo apt install wordlists -y
```

---

## 6. Cracking hash MD5 con John

Comando corretto:

```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

Output ottenuto:

```text
Loaded 1 password hash (Raw-MD5 [MD5 128/128 SSE2 4x3])
password         (?)
Session completed.
```

Interpretazione:

```text
John ha identificato l'hash come Raw-MD5.
Ha usato la wordlist rockyou.txt.
La password è stata trovata.
```

---

## 7. Visualizzare password crackata

Comando:

```bash
john --show --format=raw-md5 hashes.txt
```

Output:

```text
?:password

1 password hash cracked, 0 left
```

Risultato finale:

```text
5f4dcc3b5aa765d61d8327deb882cf99 = password
```

---

## 8. Spiegazione

L’hash:

```text
5f4dcc3b5aa765d61d8327deb882cf99
```

è un hash MD5 della stringa:

```text
password
```

John ha trovato la password perché `password` è presente nella wordlist `rockyou.txt`.

---

## 9. Comandi rapidi

Creare file hash:

```bash
echo "HASH" > hashes.txt
```

Crack MD5:

```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

Mostrare risultato:

```bash
john --show --format=raw-md5 hashes.txt
```

Esempio completo:

```bash
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hashes.txt
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --show --format=raw-md5 hashes.txt
```

---

## 10. Errori comuni

### Errore: rockyou.txt non trovato

Errore:

```text
fopen: /usr/share/wordlists/rockyou.txt: No such file or directory
```

Soluzione:

```bash
ls -la /usr/share/wordlists/
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
```

Oppure:

```bash
sudo apt install wordlists -y
```

---

### Errore: formato hash errato

Se John non riconosce il formato o non trova risultati, verificare il tipo di hash.

Per MD5 semplice usare:

```bash
john --format=raw-md5 hashes.txt
```

---

## 11. Collegamento con eJPT

Questa esercitazione copre gli obiettivi:

```text
Condurre cracking dell'hash
Raccogliere informazioni hash/password dal target
Analizzare credenziali deboli
Usare wordlist
Interpretare output di John
```

---

## 12. Lezioni apprese

```text
Un hash non è una password in chiaro.
Il formato dell'hash deve essere identificato correttamente.
John richiede una wordlist valida.
rockyou.txt può essere compressa come rockyou.txt.gz.
john --show serve per visualizzare i risultati già crackati.
Password comuni come "password" vengono trovate molto velocemente.
```

---

## 13. Nota etica

Questo cracking è stato svolto solo su hash ottenuti da un laboratorio locale autorizzato.

Non eseguire cracking di hash appartenenti a sistemi reali o terzi senza autorizzazione esplicita.
