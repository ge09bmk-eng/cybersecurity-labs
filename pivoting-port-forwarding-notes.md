# Pivoting and Port Forwarding Notes - eJPT Lab

Appunti personali sul pivoting e port forwarding nel laboratorio eJPT.

> Tutte le attività sono svolte esclusivamente in ambiente locale autorizzato.

---

## 1. Obiettivo

Allenare il pivoting usando una macchina ponte per raggiungere un target interno non accessibile direttamente da Kali.

Obiettivi eJPT coperti:

```text
Dimostrare di ruotare aggiungendo un percorso
Dimostrare port forwarding
Raggiungere un host interno tramite macchina compromessa/ponte
Usare SSH local port forwarding
```

---

## 2. Schema laboratorio pivoting

```text
Kali Linux
192.168.1.101
        |
        | rete principale 192.168.1.0/24
        |
Metasploitable2
eth0: 192.168.1.100/24
eth1: 10.10.10.1/24
        |
        | rete interna PIVOT-NET 10.10.10.0/24
        |
Target3-Ubuntu
enp0s8: 10.10.10.2/24
```

---

## 3. Ruoli delle macchine

```text
Kali Linux        macchina attaccante
Metasploitable2   macchina ponte
Target3-Ubuntu    target interno
```

---

## 4. Condizione corretta per il pivoting

La condizione desiderata è:

```text
Metasploitable2 deve vedere Target3 sulla rete 10.10.10.0/24
Kali NON deve vedere direttamente Target3 sulla rete 10.10.10.0/24
```

---

## 5. Configurazione rete Metasploitable2

Metasploitable2 ha due interfacce:

```text
eth0: 192.168.1.100/24
eth1: rete interna PIVOT-NET
```

Comando usato per configurare la seconda interfaccia:

```bash
sudo ifconfig eth1 10.10.10.1 netmask 255.255.255.0 up
```

Verifica:

```bash
ip a
ifconfig
```

Risultato atteso:

```text
eth1 = 10.10.10.1/24
```

---

## 6. Configurazione rete Target3-Ubuntu

Target3 ha una seconda interfaccia sulla rete interna:

```text
enp0s8
```

Comandi usati:

```bash
sudo ip addr add 10.10.10.2/24 dev enp0s8
sudo ip link set enp0s8 up
```

Verifica:

```bash
ip a
```

Risultato atteso:

```text
enp0s8 = 10.10.10.2/24
```

---

## 7. Test da Metasploitable2 verso Target3

Da Metasploitable2:

```bash
ping -c 3 10.10.10.2
```

Risultato ottenuto:

```text
3 packets transmitted, 3 received, 0% packet loss
```

Interpretazione:

```text
Metasploitable2 riesce a comunicare con Target3 sulla rete interna.
```

---

## 8. Test da Kali verso Target3 interno

Da Kali:

```bash
ping -c 3 10.10.10.2
```

Risultato ottenuto:

```text
3 packets transmitted, 0 received, 100% packet loss
```

Interpretazione:

```text
Kali non raggiunge direttamente Target3 sulla rete 10.10.10.0/24.
Questa è la condizione corretta per allenare pivoting.
```

---

## 9. Problema SSH con Metasploitable2

Tentativo iniziale:

```bash
ssh msfadmin@192.168.1.100
```

Errore:

```text
Unable to negotiate with 192.168.1.100 port 22: no matching host key type found
```

Motivo:

```text
Metasploitable2 usa algoritmi SSH vecchi.
Kali moderno li rifiuta di default.
```

Comando corretto:

```bash
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa msfadmin@192.168.1.100
```

Credenziali:

```text
Username: msfadmin
Password: msfadmin
```

Accesso riuscito:

```text
msfadmin@metasploitable:~$
```

---

## 10. Creazione tunnel SSH local port forwarding

Da Kali è stato creato un tunnel verso Target3 passando da Metasploitable2:

```bash
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa -L 2222:10.10.10.2:22 msfadmin@192.168.1.100
```

Spiegazione:

```text
-L 2222:10.10.10.2:22
```

Significa:

```text
Porta locale Kali: 2222
Target interno finale: 10.10.10.2
Porta target finale: 22 SSH
Macchina ponte: Metasploitable2 192.168.1.100
```

Schema:

```text
Kali 127.0.0.1:2222
        ↓
Tunnel SSH verso Metasploitable2
        ↓
Metasploitable2 inoltra traffico
        ↓
Target3 10.10.10.2:22
```

Nota:

```text
La sessione SSH verso Metasploitable2 deve rimanere aperta.
Se viene chiusa, il tunnel si interrompe.
```

---

## 11. Accesso a Target3 tramite tunnel

In un secondo terminale Kali:

```bash
ssh spywer@127.0.0.1 -p 2222
```

Password:

```text
password
```

Risultato:

```text
Welcome to Ubuntu 26.04 LTS
spywer@target3:~$
```

Interpretazione:

```text
Kali è riuscita ad accedere a Target3 passando attraverso Metasploitable2.
Il pivoting tramite local port forwarding è riuscito.
```

---

## 12. Errore incontrato: Connection refused

Comando eseguito prima di aprire il tunnel:

```bash
ssh spywer@127.0.0.1 -p 2222
```

Errore:

```text
Connection refused
```

Motivo:

```text
La porta locale 2222 non era ancora in ascolto.
Il tunnel SSH non era stato creato.
```

Correzione:

```bash
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa -L 2222:10.10.10.2:22 msfadmin@192.168.1.100
```

Poi, in un secondo terminale:

```bash
ssh spywer@127.0.0.1 -p 2222
```

---

## 13. Catena completa

```text
Kali identifica Metasploitable2
↓
Metasploitable2 ha accesso alla rete interna 10.10.10.0/24
↓
Target3 ha IP interno 10.10.10.2
↓
Kali non riesce a pingare 10.10.10.2
↓
Metasploitable2 riesce a pingare 10.10.10.2
↓
Kali apre tunnel SSH verso Metasploitable2
↓
Kali usa 127.0.0.1:2222
↓
Il traffico viene inoltrato a 10.10.10.2:22
↓
Accesso SSH riuscito a Target3
```

---

## 14. Comandi rapidi

Configurare Metasploitable2:

```bash
sudo ifconfig eth1 10.10.10.1 netmask 255.255.255.0 up
```

Configurare Target3:

```bash
sudo ip addr add 10.10.10.2/24 dev enp0s8
sudo ip link set enp0s8 up
```

Test da Metasploitable2:

```bash
ping -c 3 10.10.10.2
```

Test da Kali:

```bash
ping -c 3 10.10.10.2
```

SSH verso Metasploitable2 con algoritmi legacy:

```bash
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa msfadmin@192.168.1.100
```

Tunnel SSH:

```bash
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa -L 2222:10.10.10.2:22 msfadmin@192.168.1.100
```

Accesso a Target3 tramite tunnel:

```bash
ssh spywer@127.0.0.1 -p 2222
```

---

## 15. Concetto chiave

```text
Pivoting significa usare una macchina raggiungibile per arrivare a una rete o macchina non raggiungibile direttamente.
```

In questo laboratorio:

```text
Kali non vede Target3 interno
Metasploitable2 vede Target3 interno
Kali usa Metasploitable2 come ponte
```

---

## 16. Collegamento con eJPT

Questa esercitazione copre:

```text
Pivoting
Port forwarding
Uso di una macchina ponte
Accesso a target interno
Gestione tunnel SSH
Verifica raggiungibilità di rete
```

---

## 17. Lezioni apprese

```text
Prima di fare pivoting bisogna capire bene la topologia.
Un ping fallito da Kali verso la rete interna è corretto se la rete è nascosta.
La macchina ponte deve vedere sia Kali sia il target interno.
Il tunnel SSH deve rimanere aperto.
127.0.0.1:2222 su Kali può rappresentare un servizio remoto interno.
Connection refused indica spesso che il tunnel non è attivo.
```

---

## 18. Enumeration attraverso il tunnel

Dopo aver creato il tunnel SSH:

```bash
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa -L 2222:10.10.10.2:22 msfadmin@192.168.1.100
```

è stata eseguita una scansione locale da Kali sulla porta 2222:

```bash
nmap -sV -p 2222 127.0.0.1
```

Output ottenuto:

```text
PORT     STATE SERVICE VERSION
2222/tcp open  ssh     OpenSSH 10.2p1 Ubuntu 2ubuntu3
```

Interpretazione:

```text
La porta 2222 su Kali è un forward locale.
Il servizio rilevato non appartiene realmente a Kali.
Il traffico viene inoltrato verso Target3 sulla rete interna.
```

Schema:

```text
Kali 127.0.0.1:2222
        ↓
Tunnel SSH
        ↓
Metasploitable2 192.168.1.100 / 10.10.10.1
        ↓
Target3 10.10.10.2:22
```

Conclusione:

```text
L’enumeration attraverso il tunnel è riuscita.
Nmap vede il servizio SSH interno di Target3 passando dal port forwarding.
```
## 19. Nota etica
Questa attività è stata svolta solo in laboratorio locale autorizzato.

Non usare tecniche di pivoting, tunneling o port forwarding su reti reali senza autorizzazione esplicita.
