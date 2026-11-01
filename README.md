# Лабораторна робота. DNS

| Поле | Значення |
|---|---|
| ПІБ | Сависько Назір Михайлович |
| Група | ІПЗ-2.01 |
| Ідентифікатор (`sNN`) | `se-201-27` |
| Дата виконання | 01.11.2026 |
| Середовище | Ubuntu (локально, не Codespaces), Docker 29.1.3, Docker Compose 2.40.3 |
| Версія `dig` | `DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu` |

## Параметри варіанта

| Параметр | Значення |
|---|---|
| `$TTL`, с | 30 |
| TTL запису SOA, с | 45 |
| Поле `minimum` SOA, с | 20 |
| Псевдонім (CNAME на `www`) | `portal` |
| Другий хост і його TTL, с | `stage`, 50 |
| Пріоритет MX | 15 |
| Токен TXT | `b14794e9` |
| Ім'я для завдання 5 | `new-vhqir.se-201-27.lab` |

## Завдання 1. Складові URI

### 1.1. Розбір

URI з картки:

1. `http://www.se-201-27.lab/items/reports?lang=19&limit=asc#results`
2. `http://portal.se-201-27.lab:80/reports/?lang=%D0%BA%D0%B8%D1%97%D0%B2`
3. `http://user:pass@stage.se-201-27.lab/items#results`
4. `https://www.se-201-27.lab:8443/items/../reports`
5. `mailto:admin@se-201-27.lab?subject=items`

| № | scheme | userinfo | host | port | path | query | fragment |
|---|---|---|---|---|---|---|---|
| 1 | `http` | — | `www.se-201-27.lab` | 80 (за замовч.) | `/items/reports` | `lang=19&limit=asc` | `results` |
| 2 | `http` | — | `portal.se-201-27.lab` | 80 | `/reports/` | `lang=%D0%BA%D0%B8%D1%97%D0%B2` | — |
| 3 | `http` | `user:pass` | `stage.se-201-27.lab` | 80 (за замовч.) | `/items` | — | `results` |
| 4 | `https` | — | `www.se-201-27.lab` | 8443 | `/items/../reports` | — | — |
| 5 | `mailto` | — | — | — | `admin@se-201-27.lab` | `subject=items` | — |

| № | Ім'я, що запитується в DNS | Заголовок `Host` | Ціль запиту (request target) |
|---|---|---|---|
| 1 | `www.se-201-27.lab` | `www.se-201-27.lab` | `/items/reports?lang=19&limit=asc` |
| 2 | `portal.se-201-27.lab` (далі CNAME на `www`) | `portal.se-201-27.lab` | `/reports/?lang=%D0%BA%D0%B8%D1%97%D0%B2` |
| 3 | `stage.se-201-27.lab` | `stage.se-201-27.lab` | `/items` |
| 4 | `www.se-201-27.lab` | `www.se-201-27.lab:8443` | `/reports` |
| 5 | `se-201-27.lab` (запис MX, запитує поштовий сервер, а не HTTP-клієнт) | — | — |

Декодоване значення параметра URI 2: `київ`.

Чому URI 5 не має складової host, хоча містить символ `@`: після `mailto:` немає `//`, тому немає authority (userinfo, host, port). Усе після двокрапки до `?` це path, тобто просто поштова адреса, а `@` тут частина адреси, а не роздільник userinfo.

### 1.2. Перевірка (після завдання 3)

```text
root@ns-se-201-27:/# curl -v 'http://www.se-201-27.lab/items/reports?lang=19&limit=asc#results'
* Host www.se-201-27.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to www.se-201-27.lab (10.53.0.80) port 80
> GET /items/reports?lang=19&limit=asc HTTP/1.1
> Host: www.se-201-27.lab
> User-Agent: curl/8.5.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sun, 01 Nov 2026 12:50:06 GMT
< Content-Type: text/plain
< Content-Length: 148
< Connection: keep-alive
<
server: web-a
Host: www.se-201-27.lab
request URI: /items/reports?lang=19&limit=asc
path: /items/reports
query: lang=19&limit=asc
client: 10.53.1.1
* Connection #0 to host www.se-201-27.lab left intact

root@ns-se-201-27:/# curl -v 'http://portal.se-201-27.lab:80/reports/?lang=%D0%BA%D0%B8%D1%97%D0%B2'
* Host portal.se-201-27.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to portal.se-201-27.lab (10.53.0.80) port 80
> GET /reports/?lang=%D0%BA%D0%B8%D1%97%D0%B2 HTTP/1.1
> Host: portal.se-201-27.lab
> User-Agent: curl/8.5.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sun, 01 Nov 2026 12:50:14 GMT
< Content-Type: text/plain
< Content-Length: 165
< Connection: keep-alive
<
server: web-a
Host: portal.se-201-27.lab
request URI: /reports/?lang=%D0%BA%D0%B8%D1%97%D0%B2
path: /reports/
query: lang=%D0%BA%D0%B8%D1%97%D0%B2
client: 10.53.1.1
* Connection #0 to host portal.se-201-27.lab left intact

root@ns-se-201-27:/# curl -v 'http://user:pass@stage.se-201-27.lab/items#results'
* Host stage.se-201-27.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:80...
* Connected to stage.se-201-27.lab (10.53.0.81) port 80
* Server auth using Basic with user 'user'
> GET /items HTTP/1.1
> Host: stage.se-201-27.lab
> Authorization: Basic dXNlcjpwYXNz
> User-Agent: curl/8.5.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sun, 01 Nov 2026 12:50:20 GMT
< Content-Type: text/plain
< Content-Length: 99
< Connection: keep-alive
<
server: web-b
Host: stage.se-201-27.lab
request URI: /items
path: /items
query:
client: 10.53.1.1
* Connection #0 to host stage.se-201-27.lab left intact
```

Значення заголовка `Authorization` і результат декодування:

```text
Authorization: Basic dXNlcjpwYXNz

root@ns-se-201-27:/# echo 'dXNlcjpwYXNz' | base64 -d
user:pass
```

URI 4: етап, на якому запит не вдається (пошук імені / TCP / TLS), і рядки виводу, що це підтверджують:

Ім'я знайшлося (`was resolved`, IP 10.53.0.80), а впало на **TCP**: `Connection refused`, бо на порту 8443 нічого не слухає. До TLS не дійшло, бо TLS починається тільки після TCP-з'єднання.

```text
root@ns-se-201-27:/# curl -v 'https://www.se-201-27.lab:8443/items/../reports'
* Host www.se-201-27.lab:8443 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:8443...
* connect to 10.53.0.80 port 8443 from 10.53.1.1 port 57294 failed: Connection refused
* Failed to connect to www.se-201-27.lab port 8443 after 0 ms: Couldn't connect to server
* Closing connection
curl: (7) Failed to connect to www.se-201-27.lab port 8443 after 0 ms: Couldn't connect to server
```

```text
root@ns-se-201-27:/# curl 'http://www.se-201-27.lab/items/../reports'
server: web-a
Host: www.se-201-27.lab
request URI: /reports
path: /reports
query:
client: 10.53.1.1
root@ns-se-201-27:/# curl --path-as-is 'http://www.se-201-27.lab/items/../reports'
server: web-a
Host: www.se-201-27.lab
request URI: /items/../reports
path: /reports
query:
client: 10.53.1.1
```

| Запит | `request URI`, отриманий сервером | `path`, отриманий сервером |
|---|---|---|
| звичайний | `/reports` | `/reports` |
| `--path-as-is` | `/items/../reports` | `/reports` |

Розбіжності між URI та тим, що отримав сервер, і їх пояснення:

- `#results` (URI 1 і 3) на сервер не йде, фрагмент лишається у клієнта.
- `user:pass` (URI 3) у `Host` не потрапляє: curl робить з нього `Authorization: Basic dXNlcjpwYXNz`, а це просто base64, без шифрування.
- `:80` (URI 2) curl у `Host` не пише, бо це стандартний порт для `http`.
- `query` (URI 2) сервер отримує закодованим, `%D0%BA…`, розкодовує його вже програма.
- `..` (URI 4) звичайно прибирає curl ще до відправки. З `--path-as-is` шлях іде як є, і тоді його прибирає сервер: у `request URI` ще є `..`, а `path` уже `/reports`.

## Завдання 2. Проходження ієрархії вручну

```text
root@ns-se-201-27:/# dig @10.53.0.2 ns.se-201-27.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.2 ns.se-201-27.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44628
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: cb8f0e6abcb981d0010000006ae731001bd48ee9b59f5a7b (good)
;; QUESTION SECTION:
;ns.se-201-27.lab.		IN	A

;; AUTHORITY SECTION:
lab.			172800	IN	NS	ns1.nic.lab.

;; ADDITIONAL SECTION:
ns1.nic.lab.		172800	IN	A	10.53.0.3

;; Query time: 81 msec
;; SERVER: 10.53.0.2#53(10.53.0.2) (UDP)
;; WHEN: Sun Nov 01 12:30:57 UTC 2026
;; MSG SIZE  rcvd: 117

root@ns-se-201-27:/# dig @10.53.0.3 ns.se-201-27.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.3 ns.se-201-27.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64842
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: d410629802b10c8f010000006ae731076ce57bae7b72aefb (good)
;; QUESTION SECTION:
;ns.se-201-27.lab.		IN	A

;; AUTHORITY SECTION:
se-201-27.lab.		3600	IN	NS	ns.se-201-27.lab.

;; ADDITIONAL SECTION:
ns.se-201-27.lab.	3600	IN	A	10.53.1.1

;; Query time: 79 msec
;; SERVER: 10.53.0.3#53(10.53.0.3) (UDP)
;; WHEN: Sun Nov 01 12:31:03 UTC 2026
;; MSG SIZE  rcvd: 103

root@ns-se-201-27:/# dig @10.53.1.1 ns.se-201-27.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 ns.se-201-27.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2697
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: b809721e2b9f2c61010000006ae731234f117b2ce36cf91f (good)
;; QUESTION SECTION:
;ns.se-201-27.lab.		IN	A

;; ANSWER SECTION:
ns.se-201-27.lab.	30	IN	A	10.53.1.1

;; AUTHORITY SECTION:
se-201-27.lab.		30	IN	NS	ns.se-201-27.lab.

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sun Nov 01 12:31:31 UTC 2026
;; MSG SIZE  rcvd: 103
```

| Сервер | status | flags | ANSWER | AUTHORITY | ADDITIONAL |
|---|---|---|---|---|---|
| 10.53.0.2 | NOERROR | qr | 0 | `lab. 172800 NS ns1.nic.lab.` | `ns1.nic.lab. 172800 A 10.53.0.3` (+ OPT) |
| 10.53.0.3 | NOERROR | qr | 0 | `se-201-27.lab. 3600 NS ns.se-201-27.lab.` | `ns.se-201-27.lab. 3600 A 10.53.1.1` (+ OPT) |
| 10.53.1.1 | NOERROR | qr aa | `ns.se-201-27.lab. 30 A 10.53.1.1` | `se-201-27.lab. 30 NS ns.se-201-27.lab.` | лише OPT |

У `dig` лічильник `ADDITIONAL` на один більший, ніж видно записів, бо він рахує ще службовий OPT (EDNS).

Вивід `dig ns.se-201-27.lab A +trace`:

```text
root@ns-se-201-27:/# dig ns.se-201-27.lab A +trace

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> ns.se-201-27.lab A +trace
;; global options: +cmd
.			85839	IN	NS	a.root-servers.net.
;; Received 59 bytes from 10.53.0.53#53(10.53.0.53) in 0 ms

lab.			172800	IN	NS	ns1.nic.lab.
;; Received 117 bytes from 10.53.0.2#53(a.root-servers.net) in 80 ms

se-201-27.lab.		3600	IN	NS	ns.se-201-27.lab.
;; Received 103 bytes from 10.53.0.3#53(ns1.nic.lab) in 39 ms

ns.se-201-27.lab.	30	IN	A	10.53.1.1
se-201-27.lab.		30	IN	NS	ns.se-201-27.lab.
;; Received 103 bytes from 10.53.1.1#53(ns.se-201-27.lab) in 0 ms
```

`+trace` пройшов ті самі три кроки (root, TLD, мій сервер) і отримав ті самі записи й TTL, що й мої запити вручну.

| Запит | TTL запису NS | Прапорець `aa` | Секція, у якій міститься NS |
|---|---|---|---|
| `dig @10.53.0.3 se-201-27.lab NS +norec` | 3600 | немає | AUTHORITY |
| `dig @10.53.1.1 se-201-27.lab NS +norec` | 30 | є | ANSWER |

1. Які відповіді є відсиланнями і за якими ознаками це визначено?

Відсилання дали root (10.53.0.2) і TLD (10.53.0.3). Видно це так: немає `aa`, `ANSWER: 0`, а в AUTHORITY стоїть NS нижчої зони (`lab.` у root, `se-201-27.lab.` у TLD).

2. Для чого у відповіді TLD присутній запис у секції ADDITIONAL і що сталося б, якби його не було?

Це glue, тобто адреса `ns.se-201-27.lab` = 10.53.1.1. Мій сервер сам лежить у моїй зоні, тому без glue, щоб дізнатися його адресу, треба було б спитати його ж. Вийшло б замкнене коло, і резолвер не дійшов би до моєї зони.

3. Яка з двох копій запису NS є авторитативною і чому?

Та, що з мого сервера 10.53.1.1: у неї є `aa`, вона в ANSWER і береться з файлу зони. У TLD це лише делегування, тобто вказівка, куди йти (без `aa`, в AUTHORITY). Тому й TTL різні: 3600 поставив TLD, а 30 моя зона.

## Завдання 3. Заповнення власної зони

Serial змінив з `2026110101` на `2026110102` і додав шість записів з картки.

Вивід `zone-reload`:

```text
zone se-201-27.lab/IN: loaded serial 2026110102
OK
zone reload queued
```

| Запис | status | Прапорець `aa` | Значення в ANSWER | TTL | Відповідає картці (так/ні) |
|---|---|---|---|---|---|
| `www` A | NOERROR | так | `10.53.0.80` | 30 | так |
| псевдонім CNAME (`portal`) | NOERROR | так | `www.se-201-27.lab.` | 30 | так |
| `mail` A | NOERROR | так | `10.53.0.80` | 30 | так |
| `@` MX | NOERROR | так | `15 mail.se-201-27.lab.` | 30 | так |
| `@` TXT | NOERROR | так | `"lab-token=b14794e9"` | 30 | так |
| другий хост A (`stage`) | NOERROR | так | `10.53.0.81` | 50 | так |

Порівняння відповідей резолвера та власного сервера:

```text
root@ns-se-201-27:/# dig portal.se-201-27.lab

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> portal.se-201-27.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8683
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;portal.se-201-27.lab.		IN	A

;; ANSWER SECTION:
portal.se-201-27.lab.	30	IN	CNAME	www.se-201-27.lab.
www.se-201-27.lab.	30	IN	A	10.53.0.80

;; Query time: 40 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Sun Nov 01 12:43:44 UTC 2026
;; MSG SIZE  rcvd: 83

root@ns-se-201-27:/# dig @10.53.1.1 portal.se-201-27.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 portal.se-201-27.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58596
;; flags: qr aa; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 104e68eaa9a48106010000006ae73405cabb963fda745d3b (good)
;; QUESTION SECTION:
;portal.se-201-27.lab.		IN	A

;; ANSWER SECTION:
portal.se-201-27.lab.	30	IN	CNAME	www.se-201-27.lab.
www.se-201-27.lab.	30	IN	A	10.53.0.80

;; AUTHORITY SECTION:
se-201-27.lab.		30	IN	NS	ns.se-201-27.lab.

;; ADDITIONAL SECTION:
ns.se-201-27.lab.	30	IN	A	10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sun Nov 01 12:43:49 UTC 2026
;; MSG SIZE  rcvd: 144

root@ns-se-201-27:/# dig se-201-27.lab MX

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> se-201-27.lab MX
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 60626
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;se-201-27.lab.			IN	MX

;; ANSWER SECTION:
se-201-27.lab.		30	IN	MX	15 mail.se-201-27.lab.

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Sun Nov 01 12:43:54 UTC 2026
;; MSG SIZE  rcvd: 63

root@ns-se-201-27:/# dig @10.53.1.1 se-201-27.lab MX +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 se-201-27.lab MX +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43418
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 080f1a078a43deeb010000006ae734102045bbce1d88f852 (good)
;; QUESTION SECTION:
;se-201-27.lab.			IN	MX

;; ANSWER SECTION:
se-201-27.lab.		30	IN	MX	15 mail.se-201-27.lab.

;; AUTHORITY SECTION:
se-201-27.lab.		30	IN	NS	ns.se-201-27.lab.

;; ADDITIONAL SECTION:
mail.se-201-27.lab.	30	IN	A	10.53.0.80
ns.se-201-27.lab.	30	IN	A	10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sun Nov 01 12:44:00 UTC 2026
;; MSG SIZE  rcvd: 140
```

| Запит | flags | ANSWER (кількість, типи) | AUTHORITY (кількість, типи) | ADDITIONAL (кількість, типи) | TTL |
|---|---|---|---|---|---|
| псевдонім через resolver-a | qr rd ra | 2: CNAME, A | 0 | лише OPT | 30 |
| псевдонім через 10.53.1.1 | qr aa | 2: CNAME, A | 1: NS | 1: A (`ns`) + OPT | 30 |
| MX через resolver-a | qr rd ra | 1: MX | 0 | лише OPT | 30 |
| MX через 10.53.1.1 | qr aa | 1: MX | 1: NS | 2: A (`mail`), A (`ns`) + OPT | 30 |

Пояснення відмінностей:

Мій сервер власник даних, тому відповідає з `aa`, рекурсії не робить і додає довідку: NS зони в AUTHORITY і адреси серверів в ADDITIONAL (для MX ще й адресу `mail`, щоб клієнт не питав окремо). Резолвер посередник: у нього `rd ra` (робить рекурсію), `aa` немає, бо дані не його, і віддає він тільки відповідь, без AUTHORITY і ADDITIONAL. TTL у резолвера тут 30, бо запис щойно потрапив у кеш, а при повторі буде менше. У мого сервера TTL завжди такий, як у файлі. `Query time: 40 msec` у першому запиті через резолвер означає, що він ще ходив до TLD (у журналі TLD о 12:43:44 є запит `portal.se-201-27.lab`), бо делегування моєї зони з TTL 30 у його кеші вже спливло.

Повідомлення `zone-check` після тимчасового додавання `@ IN CNAME www`:

```text
root@ns-se-201-27:/# zone-check
dns_master_load: /lab/zone/se-201-27.lab.zone:23: se-201-27.lab: CNAME and other data
zone se-201-27.lab/IN: loading from master file /lab/zone/se-201-27.lab.zone failed: CNAME and other data
zone se-201-27.lab/IN: not loaded due to errors.
```

Після видалення рядка:

```text
root@ns-se-201-27:/# zone-check
zone se-201-27.lab/IN: loaded serial 2026110102
OK
```

1. Який TTL має кожен запис зони і звідки це значення взято?

`www`, `portal`, `mail`, MX, TXT, а також `ns` і NS мають 30 с з `$TTL 30`, бо в самих записах TTL не вказано. `stage` має 50 с, бо я вказав його явно. SOA має 45 с, бо це записано прямо в рядку SOA.

2. Скільки записів містить секція ANSWER у відповіді на запит псевдоніма і чому?

Два: `portal CNAME www.se-201-27.lab.` і `www A 10.53.0.80`. Я питав тип A, а `portal` лише псевдонім, тому сервер іде за ним і одразу додає адресу `www`. Якщо питати саме CNAME, запис буде один.

3. Чому запис CNAME неможливо розмістити для імені `@`?

CNAME означає, що це ім'я повністю є іншим ім'ям, тому інших записів у нього бути не може. А на `@` обов'язково є SOA і NS (у мене ще MX і TXT). Сервер так і пише: `CNAME and other data`, і зону не завантажує.

## Завдання 4. TTL і видимість змін

Два запити з інтервалом 5 с:

```text
root@ns-se-201-27:/# dig www.se-201-27.lab +noall +answer; sleep 5; dig www.se-201-27.lab +noall +answer
www.se-201-27.lab.	30	IN	A	10.53.0.80
www.se-201-27.lab.	25	IN	A	10.53.0.80
```

Перше значення 30, як `$TTL 30` (запис щойно потрапив у кеш), друге на 5 менше, бо між запитами пройшло 5 с.

**Прогноз** (записати до вимірювання). Формула часу появи нової адреси через залишок R, найбільша можлива затримка:

Поки запис лежить у кеші, резолвер віддає стару адресу. Якщо в момент зміни в кеші лишилось R секунд, нова адреса з'явиться через R секунд: t(нова) = t(зміни) + R. Найбільша затримка 30 с (повний TTL), якщо змінити запис одразу після того, як резолвер поклав його в кеш. Ще до 2 с може додатися через крок циклу.

Перевірка, що сервер до `zone-reload` повертає стару адресу (serial змінено на `2026110103`, `www` на `10.53.0.81`, файл збережено):

```text
root@ns-se-201-27:/# dig @10.53.1.1 www.se-201-27.lab +norec +short
10.53.0.80
```

Залишок R і час (крок 4), час `zone-reload` (крок 5), перевірка на сервері (крок 6):

Перша спроба кроку 4 о 12:56:33 дала R = 30, але до `zone-reload` пройшло забагато часу, тому я повторив крок 4. Щоб не було паузи між R і `zone-reload`, кроки з 4 по 7 виконав одним рядком.

```text
root@ns-se-201-27:/# date +%T; dig www.se-201-27.lab +noall +answer
12:56:33
www.se-201-27.lab.	30	IN	A	10.53.0.80
root@ns-se-201-27:/# date +%T; dig www.se-201-27.lab +noall +answer; zone-reload; date +%T; dig @10.53.1.1 www.se-201-27.lab +norec +short; while true; do echo "$(date +%T) $(dig +short www.se-201-27.lab)"; sleep 2; done
12:57:41
www.se-201-27.lab.	30	IN	A	10.53.0.80
zone se-201-27.lab/IN: loaded serial 2026110103
OK
zone reload queued
12:57:41
10.53.0.81
```

R = 30 с, час зміни 12:57:41.

Очікуваний час появи нової адреси (підставити R у формулу): 12:57:41 + 30 с = 12:58:11.

Вивід циклу спостереження (перші два рядки, два останні рядки зі старою адресою, рядок з новою адресою):

```text
12:57:41 10.53.0.80
12:57:43 10.53.0.80
...
12:58:08 10.53.0.80
12:58:10 10.53.0.80
12:58:12 10.53.0.81
```

Рядок `server:` з відповіді `curl`:

```text
root@ns-se-201-27:/# curl http://www.se-201-27.lab/
server: web-b
```

| Час зміни | Очікуваний час | Спостережений час | Виміряна затримка, с | Прогноз, с |
|---|---|---|---|---|
| 12:57:41 | 12:58:11 | 12:58:12 | 31 | 30 |

1. Чим пояснюється розбіжність між прогнозом і виміряним значенням?

Цикл перевіряє раз на 2 с, тому адреса змінилась десь між 12:58:10 і 12:58:12, а я побачив її о 12:58:12. Ще `date` і TTL рахують цілими секундами, і сам `dig` займає час (між 12:58:05 і 12:58:08 вийшло 3 с замість 2).

2. Якою була б найбільша можлива затримка для цього запису і за яких умов?

30 с, тобто повний `$TTL`. Так буде, якщо змінити запис одразу після того, як резолвер поклав його в кеш. У мене так і вийшло: R = 30.

3. Як слід діяти, щоб перенести сервіс на нову адресу, якщо TTL запису становить 86 400 с?

Заздалегідь, хоча б за добу, зменшити TTL до малого (наприклад, 30 або 60 с) і почекати, поки старий TTL спливе в кешах. Потім змінити адресу, і клієнти побачать її майже одразу. Старий сервер не вимикати, поки всі не перейдуть, а після переїзду повернути TTL назад.

## Завдання 5. Негативне кешування

**Прогноз** (записати до вимірювання). Формула за RFC 2308 і розраховане значення:

За RFC 2308 відповідь «імені немає» кешується на min(TTL SOA, `minimum`) = min(45, 20) = 20 с. Тому після створення запису ім'я через resolver-a з'явиться десь через 20 с після запиту, який дав `NXDOMAIN` (плюс до 2 с через крок циклу).

Перевірка, що сервер до `zone-reload` повертає `NXDOMAIN` (serial змінено на `2026110104`, додано `new-vhqir IN A 10.53.0.80`, файл збережено):

```text
root@ns-se-201-27:/# dig @10.53.1.1 new-vhqir.se-201-27.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 new-vhqir.se-201-27.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 37017
;; flags: qr aa; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 75812a24a3837578010000006ae737c6f243d867356b4bfc (good)
;; QUESTION SECTION:
;new-vhqir.se-201-27.lab.	IN	A

;; AUTHORITY SECTION:
se-201-27.lab.		20	IN	SOA	ns.se-201-27.lab. admin.se-201-27.lab. 2026110103 3600 600 86400 20

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sun Nov 01 12:59:50 UTC 2026
;; MSG SIZE  rcvd: 125
```

Запит через resolver-a (крок 3), час `zone-reload` (крок 4), перевірка на сервері (крок 5). Кроки з 3 по 6 виконав одним рядком, щоб між ними не було паузи:

```text
root@ns-se-201-27:/# date +%T; dig new-vhqir.se-201-27.lab; zone-reload; date +%T; dig @10.53.1.1 new-vhqir.se-201-27.lab +norec +short; while true; do echo "$(date +%T) $(dig new-vhqir.se-201-27.lab +noall +comments | grep -o 'status: [A-Z]*')"; sleep 2; done
13:00:19

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> new-vhqir.se-201-27.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 28904
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;new-vhqir.se-201-27.lab.	IN	A

;; AUTHORITY SECTION:
se-201-27.lab.		20	IN	SOA	ns.se-201-27.lab. admin.se-201-27.lab. 2026110103 3600 600 86400 20

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Sun Nov 01 13:00:19 UTC 2026
;; MSG SIZE  rcvd: 97

zone se-201-27.lab/IN: loaded serial 2026110104
OK
zone reload queued
13:00:19
10.53.0.80
```

Вивід циклу спостереження (перші два рядки, останній рядок з `NXDOMAIN`, перший рядок з `NOERROR`):

```text
13:00:19 status: NXDOMAIN
13:00:21 status: NXDOMAIN
...
13:00:39 status: NXDOMAIN
13:00:41 status: NOERROR
```

| Негативну відповідь закешовано | Запис створено | Ім'я резолвиться через resolver-a | Виміряно, с | Прогноз, с |
|---|---|---|---|---|
| 13:00:19 | 13:00:19 | 13:00:41 | 22 | 20 |

Різниця 2 с через крок циклу: о 13:00:39, рівно через 20 с, ще був `NXDOMAIN`, а кеш сплив десь між 13:00:39 і 13:00:41.

1. Чому тривалість негативного кешування визначається записом SOA, а не записом, який запитували?

Запис, якого немає, не має свого TTL. Тому сервер кладе у відповідь `NXDOMAIN` запис SOA зони, і резолвер з нього бере, скільки пам'ятати «немає».

2. Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть.

Ні. У файлі TTL SOA 45, а в AUTHORITY 20, і від сервера напряму (12:59:50), і через резолвер (13:00:19). Для негативної відповіді сервер ставить SOA з TTL = min(45, `minimum` 20) = 20. Через резолвер могло б бути ще менше, якби відповідь уже лежала в кеші, але тут 20, бо резолвер щойно взяв її в сервера.

3. Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити?

Ще до 20 с після створення він бачитиме `NXDOMAIN`, бо резолвер закешував «немає» на першій перевірці. Виправити: не перевіряти ім'я до створення, або перевіряти прямо на авторитативному сервері (`dig @10.53.1.1 … +norec`), або після створення почекати, поки сплине негативний TTL.

## Завдання 6. Затримка пошуку імені

**Прогноз** (записати до вимірювання). До яких серверів звернеться resolver-a:

Резолвер іде тільки до тих серверів, яких ще немає в його кеші. Корінь і `lab.` він уже знає з попередніх завдань, а повторний запит іде з кешу. За завданням 2 root відповідав десь за 80 мс, TLD десь за 40 мс, мій сервер майже за 0 мс.

| Ім'я | Перший запит | Повторний запит |
|---|---|---|
| A: нове ім'я у власній зоні | мій сервер, ≈0 мс | кеш |
| B: нове ім'я в зоні `lab.` | TLD, ≈40 мс | кеш |
| C: ім'я в неіснуючому TLD | root, ≈80 мс | кеш |
| D: `www.se-201-27.lab` після перезапуску резолвера | root, TLD, мій сервер, ≈120 мс | кеш |

Вивід вимірювань:

```text
root@ns-se-201-27:/# A=y$RANDOM.se-201-27.lab; dig $A | grep 'Query time'; dig $A | grep 'Query time'
;; Query time: 0 msec
;; Query time: 0 msec
root@ns-se-201-27:/# B=x$RANDOM.lab; dig $B | grep 'Query time'; dig $B | grep 'Query time'
;; Query time: 39 msec
;; Query time: 0 msec
root@ns-se-201-27:/# C=x$RANDOM.zz$RANDOM; dig $C | grep 'Query time'; dig $C | grep 'Query time'
;; Query time: 80 msec
;; Query time: 0 msec

user@nazir-savysko:~/pract/stand-lab$ docker compose -f stand/compose.yaml restart resolver-a
[+] Restarting 1/1
 ✔ Container dnslab-resolver-a-1  Started

root@ns-se-201-27:/# dig www.se-201-27.lab | grep 'Query time'; dig www.se-201-27.lab | grep 'Query time'
;; Query time: 403 msec
;; Query time: 0 msec
```

| Ім'я | Перший запит, мс | Повторний, мс | Сервери, до яких звернувся резолвер (за журналами) |
|---|---|---|---|
| A | 0 | 0 | лише мій сервер (13:03:09, `y21788.se-201-27.lab`) |
| B | 39 | 0 | лише TLD (13:03:13, `x15630.lab`) |
| C | 80 | 0 | лише root (13:03:18, `x29037.zz2482`) |
| D | 403 | 0 | root ×3 (`. NS`, `a.root-servers.net A`, `www A`), TLD ×1, мій сервер ×1 |

Значення `time_namelookup` з `curl` після перезапуску резолвера: `0.404301` с (перед цим ще раз перезапустив резолвер).

Фрагменти журналів запитів root, TLD і власного сервера для рядка D:

```text
root:
01-Nov-2026 13:04:34.395 client @0x7af2b0000e98 10.53.0.53#12226 (.): query: . IN NS -E(0)DC (10.53.0.2)
01-Nov-2026 13:04:34.475 client @0x7af29c001ee8 10.53.0.53#27375 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
01-Nov-2026 13:04:34.556 client @0x7af2a8000e98 10.53.0.53#7888 (www.se-201-27.lab): query: www.se-201-27.lab IN A -E(0)DC (10.53.0.2)

tld:
01-Nov-2026 13:04:34.677 client @0x7f377c000e98 10.53.0.53#42629 (www.se-201-27.lab): query: www.se-201-27.lab IN A -E(0)DC (10.53.0.3)

власний сервер (querylog):
01-Nov-2026 13:04:34.718 client @0x7fa27c002108 10.53.0.53#50312 (www.se-201-27.lab): query: www.se-201-27.lab IN A -E(0)DC (10.53.1.1)
```

Для A, B і C у журналах по одному запиту до одного сервера (A: querylog 13:03:09, B: tld 13:03:13, C: root 13:03:18), а повторних запитів там немає, бо вони йшли з кешу.

1. Яку затримку вносить кожен із серверів root, TLD і власний сервер? Покажіть, з яких рядків таблиці це випливає.

Root десь 80 мс (рядок C, резолвер ходив тільки до root), TLD десь 40 мс (рядок B, тільки TLD), мій сервер майже 0 мс (рядок A, тільки мій сервер). D вийшов 403 мс, більше ніж 80 + 40 + 0, бо після перезапуску резолвер ходив до root тричі: спершу дізнавався, які є кореневі сервери, і адресу `a.root-servers.net` (priming), а вже потім питав `www`. Між запитами до root у журналі рівно по 80 мс.

2. Чому в рядку A перший запит не проходить через root і TLD, хоча ім'я нове?

Делегування моєї зони (NS і адреса `ns.se-201-27.lab`) вже було в кеші резолвера з попередніх запитів, тому він одразу пішов на 10.53.1.1.

3. Вебсторінка завантажує ресурси з п'яти різних доменів. Як це впливає на час першого відкриття сторінки?

Для кожного нового домену потрібен свій пошук імені. Коли кеш порожній, це десятки або сотні мілісекунд перед завантаженням кожного ресурсу, і перше відкриття сторінки помітно повільніше. Повторне відкриття бере імена вже з кешу.

## Завдання 7. Інцидент: зупинка власного сервера

**Прогноз** (записати до вимірювання). Що поверне кожен резолвер після зупинки сервера та спливання TTL і скільки часу займе відповідь:

Ім'я `stage.se-201-27.lab`, TTL 50 с. resolver-a (звичайний) після спливання TTL піде до вимкненого сервера, не дочекається відповіді і за кілька секунд поверне `SERVFAIL`. resolver-b (serve-expired) майже одразу віддасть старий запис `10.53.0.81` з кешу, з коротким TTL.

Кешування запису в обох резолверах (крок 2) і час зупинки сервера (крок 3). Кроки з 2 по 5 виконав одним рядком:

```text
root@ns-se-201-27:/# dig @10.53.0.53 stage.se-201-27.lab +noall +answer; dig @10.53.0.54 stage.se-201-27.lab +noall +answer; named-stop; date +%T; sleep 60; time dig @10.53.0.53 stage.se-201-27.lab +tries=1 +timeout=30; time dig @10.53.0.53 stage.se-201-27.lab +tries=1 +timeout=30; time dig @10.53.0.54 stage.se-201-27.lab +tries=1 +timeout=30
stage.se-201-27.lab.	50	IN	A	10.53.0.81
stage.se-201-27.lab.	50	IN	A	10.53.0.81
13:07:24
```

Запити із заміром часу (крок 5):

```text
;; communications error to 10.53.0.53#53: timed out

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 stage.se-201-27.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; no servers could be reached

real	0m30.049s
user	0m0.003s
sys	0m0.006s

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 stage.se-201-27.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 32308
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;stage.se-201-27.lab.		IN	A

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Sun Nov 01 13:08:54 UTC 2026
;; MSG SIZE  rcvd: 48


real	0m0.017s
user	0m0.004s
sys	0m0.003s

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.54 stage.se-201-27.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11127
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;stage.se-201-27.lab.		IN	A

;; ANSWER SECTION:
stage.se-201-27.lab.	30	IN	A	10.53.0.81

;; Query time: 1800 msec
;; SERVER: 10.53.0.54#53(10.53.0.54) (UDP)
;; WHEN: Sun Nov 01 13:08:56 UTC 2026
;; MSG SIZE  rcvd: 64


real	0m1.819s
user	0m0.006s
sys	0m0.002s
```

| Запит | status або тайм-аут | ANSWER, TTL | `real` |
|---|---|---|---|
| resolver-a, перший | тайм-аут (`no servers could be reached`) | — | 30.049 с |
| resolver-a, другий | SERVFAIL | — | 0.017 с |
| resolver-b | NOERROR | `10.53.0.81`, TTL 30 | 1.819 с |

Розбіжності з прогнозом: resolver-a спершу 30 с взагалі не відповідав (тайм-аут `dig`), і тільки другий запит дав миттєвий `SERVFAIL`. resolver-b віддав старий запис не одразу, а через 1,8 с: спершу пробував достукатися до сервера, а потім віддав прострочений.

Час `named-start` і вивід циклу спостереження до першої успішної відповіді (крок 6):

```text
root@ns-se-201-27:/# named-start; date +%T; while true; do echo "$(date +%T) $(dig @10.53.0.53 stage.se-201-27.lab +short +tries=1 +timeout=3)"; sleep 2; done
13:10:33
13:10:33 10.53.0.81
```

Перевірка після завдання (`dig @10.53.1.1 se-201-27.lab SOA +norec +short`):

```text
root@ns-se-201-27:/# dig @10.53.1.1 se-201-27.lab SOA +norec +short
ns.se-201-27.lab. admin.se-201-27.lab. 2026110104 3600 600 86400 20
```

### Постмортем

1. Що сталося: о 13:07:24 я зупинив свій сервер 10.53.1.1 (`named-stop`), і зона `se-201-27.lab` стала недоступна.
2. Вплив на клієнтів resolver-a і resolver-b: у resolver-a після спливання TTL `stage` спершу 30 с не було відповіді, потім `SERVFAIL`, тобто ім'я не працювало. resolver-b віддавав стару адресу `10.53.0.81` за 1,8 с, і клієнти працювали далі.
3. Хронологія з виміряним часом: 13:07:24 сервер зупинено; десь 13:08:14 сплив TTL 50 с; десь 13:08:24 запит до resolver-a, тайм-аут 30,049 с; 13:08:54 resolver-a `SERVFAIL` за 0,017 с; 13:08:56 resolver-b `NOERROR` за 1,819 с, TTL 30; 13:10:33 `named-start`, resolver-a одразу віддав адресу.
4. Чому кеш допоміг або не допоміг: у resolver-b допоміг, бо serve-expired віддав прострочений запис. У resolver-a ні: після спливання TTL запису в кеші вже не було, а помилку він ще й закешував.
5. Що слід змінити: поставити другий авторитативний сервер в іншому місці (другий NS), увімкнути serve-stale на resolver-a і зробити моніторинг DNS-сервера.

1. Який TTL мав запис у відповіді resolver-b і навіщо такий TTL?

30 с, хоча в зоні 50. Прострочений запис віддають з коротким TTL (RFC 8767 радить 30 с), щоб клієнт скоро перепитав і, коли сервер повернеться, отримав свіжі дані, а не тримав старі довго.

2. Порівняйте результат і тривалість двох запитів до resolver-a. Якщо вони різняться, сформулюйте гіпотезу щодо причини; якщо ні, поясніть, що робить резолвер протягом цього часу.

Різняться: перший тайм-аут за 30,049 с, другий `SERVFAIL` за 0,017 с. Гіпотеза: на першому запиті резолвер пробував достукатися до вимкненого сервера довше, ніж чекав `dig`, а потім закешував помилку (запам'ятав, що сервер недоступний). Тому другий раз відмовив одразу, навіть не питаючи сервер.

3. Чому після `named-start` ім'я через resolver-a зарезолвилося не одразу?

У мене ім'я зарезолвилось одразу, вже в першому рядку о 13:10:33. Думаю, тому що між `SERVFAIL` (13:08:54) і запуском сервера пройшло десь 100 с, і закешована помилка вже сплила, тож резолвер одразу пішов до робочого сервера. Якби запустити сервер одразу після `SERVFAIL`, резолвер ще якийсь час віддавав би помилку з кешу.

4. У якій ситуації serve-stale шкодить, а не допомагає? Пов'яжіть відповідь із завданням 4.

Коли адресу змінили навмисно, як `www` у завданні 4 (з .80 на .81), а резолвер не може достукатися до сервера. Тоді він віддаватиме стару, вже неправильну адресу, наприклад на вимкнений старий сервер, і клієнти ходитимуть не туди.

## Фінальний файл зони

```text
; Zone se-201-27.lab — starting version.
; After every change: increase the serial, then run  zone-reload
$TTL 30
@     45 IN SOA ns.se-201-27.lab. admin.se-201-27.lab. (
            2026110104 ; serial
            3600       ; refresh
            600        ; retry
            86400      ; expire
            20         ; minimum
        )
@     IN NS ns.se-201-27.lab.
ns    IN A  10.53.1.1

; ---- add records from your variant card below this line ----

www              IN A      10.53.0.81
portal           IN CNAME  www
mail             IN A      10.53.0.80
@                IN MX     15 mail
@                IN TXT    "lab-token=b14794e9"
stage     50     IN A      10.53.0.81

new-vhqir        IN A      10.53.0.80
```

## Висновки

**Що виявилося несподіваним.** Результат, який не збігся з очікуванням, і рядок виводу, де це видно:

Я думав, що після перезапуску резолвера пошук `www` займе десь 80 + 40 + 0 = 120 мс, а вийшло `;; Query time: 403 msec`. У журналі root видно, що резолвер звертався до кореня тричі (`. IN NS`, `a.root-servers.net IN A`, `www.se-201-27.lab IN A`). Тобто після очищення кешу він спершу знову шукає, де корінь, а вже потім саме ім'я.

**Яке питання залишилося без відповіді.** Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено (RFC, документація, додатковий експеримент на стенді) і що з'ясувати не вдалося:

Не зміг пояснити всі 403 мс. У журналах від першого запиту до root (13:04:34.395) до запиту на мій сервер (13:04:34.718) проходить десь 320 мс. Я подивився журнали root, TLD і свого сервера, інших запитів там немає. Куди ділися ще приблизно 80 мс, з виводу не зрозуміло.

## Декларація використання ШІ

Використовувалися інструменти ШІ: так

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | Claude (Cowork), claude-opus-5-5 | `Дай мені покрокову інструкцію, що мені треба зробити` (+ бланк, посилання на стенд і методичка) | інструкція з командами і пояснення виводів; усі команди виконував сам, усі виводи мої, відповіді на питання і висновки писав сам |
