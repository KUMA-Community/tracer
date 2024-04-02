![image](https://github.com/borross/tracer/assets/39199196/edb2c62c-052c-440c-868e-8ea020b8c58e)

# tracer
Утилита (скрипт) была написана для получения возможности обогощать событие по значению поля со сторонних систем с использованием языка программирования Python3. Tracer.py мимикрирует под механизм обогащения аналогично CyberTrace, с обогащенными данными можно работать подобно обогащению Threat Intelligence. Утилита может работать как на Linux (рекомендуется), так и Windows платформах (ОС).

Необходимые библиотеки для работы Tracer.py:
- import socket
- from select import select
- from signal import signal
- from sys import platform
- from re import match
- from datetime import datetime
- from dateutil.relativedelta import relativedelta

Для использования TCP_FASTOPEN (рекомендуется) на ОС Linux выполните команду ниже:
echo 3 > /proc/sys/net/ipv4/tcp_fastopen

Предварительные правки для Tracer.py:
- SERVER = "127.0.0.1" (строка кода 14) - укажите IP-адрес для прослушивания
- PORT = 16666 (строка кода 15) - укажите порт для прослушивания
- Обогащение данными производится в строках 72-74, в переменную somedata можно добавить произвольные данные полученные любым способом (из файла, БД, GET запросом и т.д.), таже можно использовать и другие поля (см строку 74 кода - extraInfo=KUMA_THE_BEST_SIEM) с разделитетем "|"


На стороне KUMA нужно прописать следующее обогащение:
![image](https://github.com/borross/tracer/assets/39199196/ecbae16d-638b-4236-a809-fffd06ec7963)

По картинке выше, обогащается значение поля Code и сопоставляется с полем Tracer - url. Производительность скрипта состовляет ~ 50 EPS, при рекомендуемой настройке Enrichment: 50 connections и 100 RPS.

Возможно использовать только поле url в сопоставлении, но туда можно поместить произвольные данные

При обогащении события получаем следующие обогащенные данные:

![image](https://github.com/borross/tracer/assets/39199196/4935e2c5-b7fc-4c06-a57e-de7920e98085)

Так как используется "нелегальный" механизм обогащения в логах коллектора копятся (периодически очищайте) ошибки следующего вида:

![image](https://github.com/borross/tracer/assets/39199196/783bd530-956f-4634-8a4b-2af4dd41a126)

