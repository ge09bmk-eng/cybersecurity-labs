# SMB Enumeration Notes

## Basic SMB Enumeration

```bash
smbclient -L //TARGET -N
enum4linux -a TARGET
nmap --script smb-enum-shares,smb-enum-users -p 139,445 TARGET
nmap --script smb-os-discovery,smb-security-mode -p 139,445 TARGET
```

## Comandi utili dentro smbclient

```text
pwd
ls
dir
get filename
put filename
exit
```

## Accesso a una share

```bash
smbclient //TARGET/tmp -N
```

## Esempio Metasploitable2

Target:

```text
192.168.1.100
```

## Risultati importanti

```text
Anonymous SMB login successful
Share tmp accessible
tmp anonymous READ/WRITE
Samba 3.0.20-Debian
Computer name: metasploitable
Domain name: localdomain
FQDN: metasploitable.localdomain
SMB message signing disabled
```

## Utenti interessanti

```text
msfadmin
user
root
www-data
postgres
mysql
tomcat55
service
backup
ftp
```

## Share trovate

```text
print$   - access denied
tmp      - anonymous access allowed
opt      - access denied
IPC$     - IPC service
ADMIN$   - access denied
```

## Analisi share tmp

```text
Share: //192.168.1.100/tmp
Accesso: anonymous
Permessi: READ/WRITE
Contenuto trovato:
- .ICE-unix
- .X11-unix
- .X0-lock
- 4570.jsvc_up
```

Interpretazione:

```text
La share tmp è accessibile anonimamente, ma non contiene file utili come backup, configurazioni o credenziali.
È comunque una misconfigurazione importante da documentare.
```

## Ragionamento eJPT

```text
SMB aperto → enumero share
Share accessibile → controllo file
Enumero utenti → creo lista utenti
Controllo versione Samba
Cerco misconfigurazioni
Solo dopo scelgo exploit o accesso con credenziali
```

## Note

Usare questi comandi solo in ambienti autorizzati e controllati.
