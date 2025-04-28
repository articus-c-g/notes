# **Решение задач с vmtest-400**

## **1** 
Восстановление работы /opt/scripts/test.sh из под пользователя support
```bash
/opt/scripts/test.sh
# System Error - не видел такой ошибки
mcedit /opt/scripts/test.sh # Смотрю что в файле
bash -x /opt/scripts/test.sh # Запускаю скрипт в режиме отладки - Получаю ту же ошибку System Error
which bash # Смотрю откуда запускается bash, получаю /srv/bad_bin/bash
echo $PATH # /srv/bad_bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# Ищу где папка bad_bin добавляется в PATH
grep -rn bad_bin ~ 2>/dev/null # Ищу в домашней папке пользователя - там нет
grep -rn bad_bin /etc/ 2>/dev/null # Найдено вхождение в /etc/profile
mcedit /etc/profile # В строке 36 убрал /srv/bad_bin/ из PATH
source /etc/profile # Применил изменения
which bash # Проверяю
/opt/scripts/test.sh # Проверяю - скрипт работает.
```

## **2** 
Найти остановить и удалить контейнер, излишне потребляющий CPU
```bash
docker ps # Смотрю список контейнеров
docker stats # Смотрю статистику по контейнерам, вижу загрузку CPU в контейнере 7b7392daa61e
docker stop 7b7392daa61e # Останавливаю этот контейнер
docker rm 7b7392daa61e # Удаляю этот контейнер
```

## **3** 
Разобраться почему скрипт `/usr/local/bin/backup.sh` запускаемый через cron не создает дамп `/var/backups/testdb_backup.sql`
```bash
sudo crontab -l|grep -v -E "^#|^$" # Смотрю какие задачи запланированы у root
sudo /usr/local/bin/backup.sh # Пробую запустить вручную - permission denied
id backup_user # Пользователя backup_user не существует. Создаем системного пользователя backup_user. и даем ему права на Sql файл в который будет выполняться дамп
sudo useradd -r -s /bin/false backup_user
sudo chown backup_user:backup_user /var/backups/testdb_backup.sql
sudo chmod 664 /var/backups/testdb_backup.sql
# Проверяю есть ли файл паролей у root
ls -la /root/.pgpass 
# Создание файла .pgpass с паролем для подключения к БД
sudo echo "localhost:5432:testdb:backup_user:backup_password" | sudo tee /root/.pgpass
# Установка необходимых для такого типа файла прав: чтение запись только для владельца (root)
sudo chmod 600 /root/.pgpass 
sudo /usr/local/bin/backup.sh # Запускаю backup вручную. Проходит без ошибок
less /var/backups/testdb_backup.sql # Проверяю содержимое файла, дамп сделался
```

## **4**
Получение статистики из лога
```bash
# 4.1 Количество запросов от UserAgent curl
grep -c curl ~/huge_access.log
# 4.2 Количество ошибок 500+ в запросах HTTP/1.1
grep "HTTP/1.1\" [5-9][0-9][0-9]" ~/huge_access.log | wc -l
# 4.3 Количество запросов от ботов (Googlebot, YandexBot, Baiduspider)
grep -cE "Googlebot|YandexBot|Baiduspider" ~/huge_access.log
# 4.4 Найти IP-адреса, которые чаще всего запрашивали ресурс.
grep GET ~/huge_access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -n 10
# 4.5 Найти IP-адреса, которые часто запрашивали страницу, но получили 403.
awk '$9 == 403 {print $1}' ~/huge_access.log | sort | uniq -c | sort -nr | head -n 10
# 4.6 Найти IP-адрес, который отправил больше всего данных.
awk '{sum[$1] += $10} END {for (ip in sum) print sum[ip], ip}' ~/huge_access.log | sort -nr | head -n 1
```