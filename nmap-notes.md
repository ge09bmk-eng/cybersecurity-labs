# Nmap Notes

Appunti personali sull’utilizzo di Nmap per scanning, enumeration e preparazione eJPT.

## Host Discovery

Scansione della rete per individuare host attivi:

```bash
nmap -sn 10.10.10.0/24
nmap -sC -sV TARGET
nmap -p- TARGET
nmap -O TARGET
nmap -A TARGET
nmap -sn 10.10.10.0/24
nmap -sC -sV 10.10.10.5
nmap -p- 10.10.10.5
