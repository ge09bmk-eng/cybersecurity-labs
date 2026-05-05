whoami
id
uname -a
hostname
ip a
ip route
sudo -l
cat /etc/passwd
find / -name "flag*" 2>/dev/null
# Linux Commands

Appunti personali sui comandi Linux utili per enumeration, post-exploitation e preparazione eJPT.

## Identificazione utente e sistema

```bash
whoami
id
hostname
uname -a
```

## Informazioni di rete

```bash
ip a
ip route
ip neigh
cat /etc/hosts
cat /etc/resolv.conf
```

## Utenti e permessi

```bash
cat /etc/passwd
ls -la /home
sudo -l
```

## Ricerca file interessanti

```bash
find / -name "flag*" 2>/dev/null
find / -name "*.txt" 2>/dev/null
find / -name "*pass*" 2>/dev/null
find / -name "*config*" 2>/dev/null
```

## Lettura file

```bash
cat file.txt
less file.txt
head file.txt
tail file.txt
```

## File compressi

```bash
unzip file.zip
tar -xvf file.tar
tar -xvzf file.tar.gz
```

## Stabilizzazione shell

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

## Privilege escalation con sudo

Controllare sempre i privilegi disponibili:

```bash
sudo -l
```

Esempi comuni:

```bash
sudo find . -exec /bin/sh \; -quit
sudo python3 -c 'import os; os.execl("/bin/sh", "sh")'
sudo perl -e 'exec "/bin/sh"'
sudo awk 'BEGIN {system("/bin/sh")}'
sudo vim -c ':!/bin/bash'
```

## Note

Usare questi comandi solo in ambienti autorizzati e controllati.
