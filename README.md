![trrr](https://github.com/user-attachments/assets/1ceaa612-d04e-4610-ab23-edc99e27329e)

# tracer
Утилита для получения возможности обогощать событие в KUMA по значению поля со сторонних систем или внутренних словарей. 

Tracer.py мимикрирует под механизм обогащения аналогично CyberTrace, с обогащенными данными можно работать подобно обогащению Threat Intelligence. Утилита может работать как на Linux (рекомендуется), так и Windows платформах (ОС). 

Необходимые библиотеки для работы Tracer.py:
- import socket
- import requests
- import pickle
- import logging
- from select import select
- from sys import platform, exit
- from re import match, compile, search, error
- from datetime import datetime
- from optparse import OptionParser
- from urllib.parse import unquote
- from os.path import isfile, splitext, getsize
- from csv import DictReader
- from json import load, loads
- from time import time, sleep
- from collections import deque


Для использования TCP_FASTOPEN (рекомендуется) на ОС Linux выполните команду ниже и переиспользование портов:
- echo 3 > /proc/sys/net/ipv4/tcp_fastopen
- echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse

Предварительные правки для Tracer.py:
- SERVER = "127.0.0.1" (строка кода 26) - укажите IP-адрес для прослушивания
- PORT = 16666 (строка кода 27) - укажите порт для прослушивания

## Режимы работы:
- **Custom Mode** (Режим пользовательских функций, по умолчанию, mode == 0): Позволяет использовать собственные функции для обогащения данных. Обогащение данными производится в строках 162-183 (mode == 0), в этой секции можете использовать произвольное обогащение данными, в коде есть примеры двух тестовых обогащений.
- **Feed File Mode** (Режим загрузки файла, mode == 1): Загружает данные из указанного файла(ов) JSON или CSV для обогащения. Пример: `python3 Tracer.py -f /root/tracer/example.csv -k ioc` или `python3 Tracer.py -f /root/tracer/example.json -k mask`. Примеры файлов рядом со скриптом. Для масок URL заполняется отдельный словарь с регулярными выражениями по маске.
- **Dump Feed Mode** (Режим дампа данных, mode == 2): Сохраняет данные в файл с расширением .tracer для последующего использования. Пример: `python3 Tracer.py -d /root/tracer/Phishing_URL_Data_Feed.json -k mask` или `python3 Tracer.py -d Malicious_Hash_Data_Feed.json -k MD5`
- **Load Feed Mode** (Режим загрузки данных, mode == 3): Загружает данные из нескольких файлов с расширением .tracer для обогащения. Пример: `python3 Tracer.py -l IP_Reputation_Data_Feed.json.tracer -l Phishing_URL_Data_Feed.json.tracer -l Malicious_Hash_Data_Feed.json.tracer`
- **MISP Mode** (Режим интеграции с MISP, mode == 4): Интеграция и обогащение фидами из MISP по API с аутентификацией по токену. Пример: `python3 Tracer.py -m`. Предваритетльно необходимо добавить URL адрес MISP в переменной `misp_url` и указать API токен для выполнения запросов в переменной `misp_api_key`.

Все действия сервера логируются в файл `Tracer.log` для отслеживания и анализа работы сервера.

В случае ошибки: *-bash: /root/tracer/Tracer.py: /usr/bin/python3^M: bad interpreter: No such file or directory*

Выполните команду: `sed -i -e 's/\r$//' Tracer.py`

На стороне KUMA нужно прописать следующее обогащение:
![image](https://github.com/borross/tracer/assets/39199196/ecbae16d-638b-4236-a809-fffd06ec7963)

По картинке выше, обогащается значение поля Code и сопоставляется с полем Tracer - url. Производительность скрипта состовляет ~ 50 EPS, при рекомендуемой настройке Enrichment (количество подключений): 50 connections и 50 RPS (запросов в секунду). Если использовать в пропорции 500 / 500, то можно обогащать примерно 500 EPS событий без потерь. Протестировано (mode=3) с ~ 1.5М индикаторов. Максимальное кол-во событий в очереди обогащения рекомендуется указать 10000, а время ожидания - 5 сек.

Возможно использовать только поле url в сопоставлении, но туда можно поместить произвольные данные

При обогащении события получаем следующие обогащенные данные:

![image](https://github.com/borross/tracer/assets/39199196/4935e2c5-b7fc-4c06-a57e-de7920e98085)

При обогащении нескольких индикаторов из события получаем следующее обогащенные данные:

![image](https://github.com/borross/tracer/assets/39199196/c324dec1-8902-4bf0-9663-a8be87bc2187)

Так как используется "нелегальный" механизм обогащения в логах коллектора копятся (периодически очищайте) ошибки следующего вида:

![image](https://github.com/borross/tracer/assets/39199196/783bd530-956f-4634-8a4b-2af4dd41a126)

Пример очистки раз в сутки в CRON:
```bash
sudo tee -a /etc/cron.daily/tracer_log_clear > /dev/null <<EOF
#!/bin/bash
sudo  -- sh -c '>/var/log/Tracer.log; >/opt/kaspersky/kuma/collector/<ВАШ_ID>/log/collector'
EOF
sudo chmod +x /etc/cron.daily/tracer_log_clear
```
