# Nmap Notes

Appunti personali sull’utilizzo di Nmap per scanning, enumeration e preparazione eJPT.

## Host Discovery

Scansione della rete per individuare host attivi:

```bash
nmap -sn 10.10.10.0/24
```

## Service Enumeration

```bash
nmap -sC -sV TARGET
```

## Full Port Scan

```bash
nmap -p- TARGET
```

## OS Detection

```bash
nmap -O TARGET
```

## Aggressive Scan

```bash
nmap -A TARGET
```

## Workflow esempio

```bash
nmap -sn 10.10.10.0/24
nmap -sC -sV 10.10.10.5
nmap -p- 10.10.10.5
```

## Note

Usare Nmap solo in ambienti autorizzati e controllati.
