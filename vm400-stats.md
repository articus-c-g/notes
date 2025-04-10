# Анализ лога huge_access.log

- [Анализ лога huge\_access.log](#анализ-лога-huge_accesslog)
  - [Исходные данные](#исходные-данные)
  - [Команды и результаты анализа](#команды-и-результаты-анализа)
    - [Количество запросов от UserAgent curl](#количество-запросов-от-useragent-curl)
    - [Количество ошибок 500+ в запросах HTTP/1.1](#количество-ошибок-500-в-запросах-http11)
    - [Количество запросов от ботов (Googlebot, YandexBot, Baiduspider)](#количество-запросов-от-ботов-googlebot-yandexbot-baiduspider)
    - [IP-адреса, которые чаще всего запрашивали ресурс (топ-10)](#ip-адреса-которые-чаще-всего-запрашивали-ресурс-топ-10)
    - [IP-адреса с ошибкой 403 (топ-10)](#ip-адреса-с-ошибкой-403-топ-10)
    - [IP-адреса, отправившие больше всего данных (топ-10)](#ip-адреса-отправившие-больше-всего-данных-топ-10)


## Исходные данные
- Формат лога: 
1. `ip_адрес_клиента` 
2. `-`
3. `-`
4. `[07/Apr/2025:01:02:03`
5. '+1000]'
6. `"МЕТОД` 
7. `/uri
8. `HTTP/1.1"`
9. `статус-код_запроса` 
10. `размер` 
11. `-`
12. `"meta-externalagent/1.1`
13. `(+https://developers.facebook.com/docs/sharing/webmasters/crawler)"`

## Команды и результаты анализа

### Количество запросов от UserAgent curl
```bash
grep -c "curl" ajs.su.access.log
```

### Количество ошибок 500+ в запросах HTTP/1.1
```bash
grep "HTTP/1.1\" [5-9][0-9][0-9]" ajs.su.access.log | wc -l
```
### Количество запросов от ботов (Googlebot, YandexBot, Baiduspider)
```bash
grep -E "Googlebot|YandexBot|Baiduspider" ajs.su.access.log | wc -l
```

### IP-адреса, которые чаще всего запрашивали ресурс (топ-10)
```bash
awk '{print $1}' ajs.su.access.log | sort | uniq -c | sort -nr | head -n 10
```

### IP-адреса с ошибкой 403 (топ-10)
```bash
awk '$9 == 403 {print $1}' ajs.su.access.log | sort | uniq -c | sort -nr | head -n 10
```

### IP-адреса, отправившие больше всего данных (топ-10)
```bash
awk '{sum[$1] += $10} END {for (ip in sum) print sum[ip], ip}' ajs.su.access.log | sort -nr | head -n 10
```
