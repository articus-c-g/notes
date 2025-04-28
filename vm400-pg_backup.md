# Решение проблемы с резервным копированием базы данных

## Исходная проблема
- Периодическое задание в crontab root пользователя не выполнялось
- При ручном запуске pg_dump получали ошибку Permission denied
- Скрипт: /usr/local/bin/backup.sh
- Целевой файл: /var/backups/testdb_backup.sql
- Пользователь БД: backup_user/backup_password

## Решение

### 1. Проверка прав доступа
```bash
ls -la /var/backups
drwxr-xr-x  2 root root   4096 Feb 17 14:10 .
-rw-r--r--  1 root root      0 Apr  2 06:10 testdb_backup.sql
```

### 2. Создание пользователя и настройка прав
```bash
sudo useradd -r -s /bin/false backup_user
sudo chown backup_user:backup_user /var/backups/testdb_backup.sql
sudo chmod 664 /var/backups/testdb_backup.sql
```

### 3. Настройка скрипта backup.sh
```bash
#!/usr/bin/env bash
export PGPASSWORD=backup_password
pg_dump -U backup_user testdb > /var/backups/testdb_backup.sql 2>/var/backups/backup_error.log
if [ $? -ne 0 ]; then
    echo "Backup failed at $(date)" >> /var/backups/backup_error.log
    exit 1
fi
```

### 4. Настройка логирования
```bash
sudo touch /var/backups/backup_error.log
sudo chown backup_user:backup_user /var/backups/backup_error.log
sudo chmod 664 /var/backups/backup_error.log
```

## Результат
- Скрипт успешно выполняется
- Файл бэкапа обновляется
- Настроено логирование ошибок
- Cron-задание работает каждые 5 минут