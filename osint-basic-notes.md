# OSINT Basic Notes - eJPT Lab

Appunti personali sulla ricognizione OSINT di base per il laboratorio eJPT.

> Tutte le attività sono svolte esclusivamente su dominio pubblico di test e con finalità didattiche.

---

## 1. Obiettivo

Allenare la raccolta di informazioni pubbliche e tecniche su un dominio.

Obiettivi eJPT coperti:

```text
Extract company information from public sources
Gather email addresses from public sources
Gather technical information from public sources
Conduct reconnaissance
Evaluate information and criticality
```

---

## 2. Dominio analizzato

```text
Dominio: example.com
Tipo: dominio pubblico di esempio
Uso: laboratorio OSINT controllato
```

Nota:

```text
example.com è un dominio di test, quindi è normale trovare poche informazioni.
```

---

## 3. Creazione cartella di lavoro

```bash
mkdir -p ~/ejpt-osint
cd ~/ejpt-osint
```

---

## 4. Risoluzione DNS con nslookup

Comando:

```bash
nslookup example.com
```

Output osservato:

```text
Name: example.com
Address: 104.20.23.154
Address: 172.66.147.243
Address: 2606:4700:10::ac42:93f3
Address: 2606:4700:10::6814:179a
```

Interpretazione:

```text
Il dominio risolve su due indirizzi IPv4 e due indirizzi IPv6.
Gli IP sono associati a infrastruttura Cloudflare.
```

---

## 5. Record A con dig

Comando:

```bash
dig example.com
```

Output importante:

```text
example.com. IN A 172.66.147.243
example.com. IN A 104.20.23.154
```

Interpretazione:

```text
Il record A indica gli indirizzi IPv4 associati al dominio.
```

---

## 6. Record MX

Comando:

```bash
dig example.com MX
```

Output:

```text
example.com. 300 IN MX 0 .
```

Interpretazione:

```text
Il dominio non accetta email.
MX 0 . indica assenza di server di posta configurati.
```

---

## 7. Record NS

Comando:

```bash
dig example.com NS
```

Output:

```text
hera.ns.cloudflare.com.
elliott.ns.cloudflare.com.
```

Interpretazione:

```text
I nameserver del dominio sono gestiti da Cloudflare.
```

---

## 8. Record TXT

Comando:

```bash
dig example.com TXT
```

Output:

```text
"v=spf1 -all"
"_k2n1y4vw3qtb4skdx9e7dxt97qrmmq9"
```

Interpretazione:

```text
Il record SPF v=spf1 -all indica che nessun server è autorizzato a inviare email per questo dominio.
Questa configurazione è coerente con un dominio che non deve inviare posta.
Il secondo record TXT è un valore tecnico/verifica e non rappresenta una vulnerabilità.
```

---

## 9. WHOIS

Comando:

```bash
whois example.com
```

Output importante:

```text
Domain Name: EXAMPLE.COM
Registrar: RESERVED-Internet Assigned Numbers Authority
Creation Date: 1995-08-14
Registry Expiry Date: 2026-08-13
Name Server: ELLIOTT.NS.CLOUDFLARE.COM
Name Server: HERA.NS.CLOUDFLARE.COM
DNSSEC: signedDelegation
```

Interpretazione:

```text
example.com è un dominio riservato/di esempio.
Il registrar è IANA.
DNSSEC risulta attivo.
I nameserver coincidono con quelli visti nei record NS.
```

---

## 10. WhatWeb

Comando:

```bash
whatweb http://example.com
```

Output:

```text
http://example.com [200 OK] Allow[GET, HEAD], Country[UNITED STATES][US], HTML5, HTTPServer[cloudflare], IP[104.20.23.154], Title[Example Domain], UncommonHeaders[cf-cache-status,cf-ray]
```

Interpretazione:

```text
Il sito risponde con HTTP 200 OK.
Il sito usa HTML5.
Il server rilevato è Cloudflare.
Il titolo della pagina è Example Domain.
Sono presenti header specifici Cloudflare come cf-cache-status e cf-ray.
```

---

## 11. theHarvester

Comando:

```bash
theHarvester -d example.com -b duckduckgo
```

Output:

```text
No IPs found.
No emails found.
No people found.
Hosts found: 0
```

Interpretazione:

```text
Non sono state trovate email pubbliche.
Non sono stati trovati nomi/persona.
Non sono stati trovati host o sottodomini.
Questo risultato è normale per un dominio di esempio come example.com.
```

---

## 12. Comandi per salvare output

```bash
dig example.com A > dig-A.txt
dig example.com MX > dig-MX.txt
dig example.com NS > dig-NS.txt
dig example.com TXT > dig-TXT.txt
whois example.com > whois.txt
whatweb http://example.com > whatweb.txt
theHarvester -d example.com -b duckduckgo > theharvester-duckduckgo.txt
```

---

## 13. Tabella riepilogo

```text
Dominio: example.com

IPv4:
104.20.23.154
172.66.147.243

IPv6:
2606:4700:10::ac42:93f3
2606:4700:10::6814:179a

Nameserver:
hera.ns.cloudflare.com
elliott.ns.cloudflare.com

MX:
0 .
Nessun server email configurato

TXT:
v=spf1 -all
_k2n1y4vw3qtb4skdx9e7dxt97qrmmq9

Tecnologia web:
HTML5
Cloudflare

Email trovate:
Nessuna

Host/sottodomini trovati:
Nessuno

DNSSEC:
signedDelegation
```

---

## 14. Valutazione criticità

Criticità:

```text
Informational / Low
```

Motivo:

```text
Le informazioni raccolte sono pubbliche.
Non sono state trovate email.
Non sono stati trovati sottodomini.
Non sono state identificate vulnerabilità.
L’uso di Cloudflare e DNSSEC non è una vulnerabilità.
```

Impatto:

```text
Confidenzialità: basso
Integrità: basso
Disponibilità: basso
Criticità complessiva: Informational / Low
```

---

## 15. Collegamento con eJPT

Questa esercitazione copre:

```text
Extract company information from public sources
Gather email addresses from public sources
Gather technical information from public sources
Conduct web application reconnaissance
Evaluate information and criticality
```

---

## 16. Lezioni apprese

```text
nslookup e dig aiutano a identificare IP e record DNS.
MX indica la configurazione email del dominio.
NS indica i nameserver.
TXT può contenere SPF, verifiche dominio o informazioni tecniche.
whois fornisce informazioni sul dominio e registrar.
whatweb identifica tecnologie web e header.
theHarvester cerca email, host e informazioni pubbliche.
Non trovare risultati non significa errore: dipende dal dominio analizzato.
Le informazioni OSINT vanno valutate per impatto e criticità.
```

---
## 17. HTTP Headers con curl

Comando:

```bash
curl -I http://example.com
```

Output osservato:

```text
HTTP/1.1 200 OK
Date: Mon, 08 Jun 2026 13:21:50 GMT
Content-Type: text/html
Connection: keep-alive
Server: cloudflare
Last-Modified: Fri, 05 Jun 2026 20:00:44 GMT
Allow: GET, HEAD
Accept-Ranges: bytes
Age: 10185
cf-cache-status: HIT
CF-RAY: a08833b09aeaee8d-MXP
```

Interpretazione:

```text
HTTP/1.1 200 OK indica che il sito risponde correttamente.
Content-Type: text/html indica che il contenuto restituito è HTML.
Server: cloudflare indica che il sito è servito tramite Cloudflare.
Allow: GET, HEAD indica i metodi HTTP consentiti.
cf-cache-status: HIT indica che la risposta è stata servita dalla cache Cloudflare.
CF-RAY è un identificativo Cloudflare della richiesta.
```

Valutazione:

```text
Gli header raccolti sono informazioni pubbliche.
Non indicano da soli una vulnerabilità.
Sono utili per capire tecnologia, caching, server front-end e comportamento HTTP.
```

Criticità:

```text
Informational / Low
```

## 18. Nota etica

Questa attività è stata svolta su un dominio pubblico di esempio.

Non eseguire ricognizione aggressiva o scansioni invasive contro domini reali senza autorizzazione esplicita.
