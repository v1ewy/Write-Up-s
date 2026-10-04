# OhSINT

Parent: [[Writeups]]

[[BasicPentesting]], [[PickleRick]], [[Google]], [[GitHub]]

## Суть
Прохождение TryHackMe комнаты OhSINT (Easy) — OSINT-расследование по открытой фотографии. Извлечение EXIF-метаданных (GPS-координаты, автор), поиск по социальным сетям и скрытым данным на веб-страницах.

## Простыми словами
По фотографии нашли, где она сделана, кто автор, его Twitter, GitHub, блог, WiFi-точку — и даже пароль, спрятанный на странице блога белым текстом.

## Аспекты
- Hex Fiend: извлечены GPS-координаты (54°17.687778N, 2°15.022104W) — Yorkshire Dales National Park
- Из метаданных: никнейм автора — OWoodflint
- Google → GitHub: профиль с email и ссылкой на Twitter
- Twitter (X): аватарка (1 флаг), BSSID B4:5D:50:AA:86:41
- WiGLE.net: SSID (3 флаг) и город (2 флаг)
- WordPress блог: место отпуска (6 флаг)
- Пароль скрыт на странице блога белым текстом — выделение Cmd+A раскрыло (7 флаг)

#project #writeup #ohsint #osint #exif