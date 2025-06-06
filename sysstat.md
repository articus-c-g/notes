# Утилита sysstat (sar)

`sysstat` — это набор инструментов для мониторинга производительности системы Linux. Одним из ключевых инструментов в этом наборе является `sar` (System Activity Reporter), который позволяет собирать, просматривать и анализировать данные о системной активности.

## Установка sysstat

### Debian/Ubuntu

В большинстве дистрибутивов Linux, таких как Ubuntu/Debian, пакет `sysstat` обычно уже установлен. Чтобы проверить, установлен ли он, и узнать его версию, можно использовать следующие команды:

```bash
sudo apt update
apt-cache policy sysstat
```

Если `sysstat` не установлен, его можно установить с помощью менеджера пакетов вашего дистрибутива. Для Ubuntu/Debian команда будет следующей:

```bash
sudo apt install sysstat
```

По умолчанию, сбор данных может быть не включен. Отредактируйте файл `/etc/default/sysstat` и установите `ENABLED="true"`.

### Fedora/RHEL (включая AlmaLinux, CentOS, Rocky Linux)

Для установки `sysstat` в дистрибутивах на базе RHEL (Fedora, AlmaLinux, CentOS, Rocky Linux) используйте следующую команду:

```bash
sudo dnf install sysstat
```
Для более старых версий RHEL/CentOS (до 8) может использоваться `yum`:
```bash
sudo yum install sysstat
```

После установки необходимо включить и запустить службу `sysstat`:

```bash
sudo systemctl enable --now sysstat
```

Проверить статус службы:
```bash
sudo systemctl status sysstat
```

## Настройка автоматического сбора данных

### Debian/Ubuntu

`sysstat` включает скрипты для автоматического сбора данных о производительности через cron. Эти данные сохраняются в `/var/log/sysstat/` (или `/var/log/sa/` в некоторых конфигурациях) и могут быть использованы `sar` для анализа исторических данных.

Задания cron обычно находятся в файлах `/etc/cron.d/sysstat`, `/etc/cron.daily/sysstat`, `/etc/cron.hourly/sysstat`.
Скрипты сбора (`sa1`) и генерации отчетов (`sa2`) вызываются из этих cron-заданий. Основной файл конфигурации для cron-заданий - `/etc/default/sysstat`.

### Fedora/RHEL

В современных RHEL-подобных системах (RHEL 8+, Fedora) для управления сбором данных используется `systemd timer` (`sysstat-collect.timer` и `sysstat-summary.timer`) вместо традиционных cron-заданий для `sa1` и `sa2`.

*   **Конфигурационный файл sysstat:** `/etc/sysconfig/sysstat`. Здесь можно настроить такие параметры, как `HISTORY` (сколько дней хранить логи).
*   **Файлы systemd timer:**
    *   `/usr/lib/systemd/system/sysstat-collect.timer`: Определяет, как часто запускается `sa1` (по умолчанию каждые 10 минут). Для изменения интервала используйте `sudo systemctl edit sysstat-collect.timer` и переопределите параметр `OnCalendar`.
    *   `/usr/lib/systemd/system/sysstat-summary.timer`: Определяет, как часто запускается `sa2` (генерация суммарного отчета за день).
*   **Путь к сборщику данных:** Обычно `/usr/lib64/sa/sa1` и `/usr/lib64/sa/sa2`.
*   **Логи данных:** `/var/log/sa/`.

После изменения конфигурации systemd timers, перезагрузите конфигурацию systemd:
```bash
sudo systemctl daemon-reload
```
И перезапустите таймеры, если необходимо:
```bash
sudo systemctl restart sysstat-collect.timer
sudo systemctl restart sysstat-summary.timer
```

## Варианты использования sar

Утилита `sar` предоставляет множество опций для анализа различных аспектов производительности системы.

### Нагрузка на CPU

1.  **Средняя загрузка процессора за текущий день:**
    ```bash
    sar -u
    ```
    Метрики: `%user`, `%system`, `%iowait`, `%steal`, `%idle`.

2.  **Просмотр загрузки CPU конкретными процессами (используется `pidstat` из пакета `sysstat`):**
    ```bash
    pidstat -u 1 3
    ```
    Покажет загрузку CPU процессами в реальном времени (обновление каждую 1 секунду, 3 раза).

3.  **Анализ исторических данных для выявления пиков нагрузки на CPU:**
    ```bash
    sar -u -f /var/log/sysstat/saXX
    ```
    Где `saXX` — файл с данными за определенный день (например, `sa07` для 7-го числа месяца).

4.  **Детальное распределение нагрузки CPU:**
    ```bash
    sar -u
    ```
    Метрики: `%user`, `%system`, `%iowait`, `%steal`, `%idle`.

5.  **Информация по всем CPU, включая отдельные ядра (если применимо):**
    ```bash
    sar -P ALL
    ```

### Использование памяти

1.  **Изменения использования RAM за день:**
    ```bash
    sar -r
    ```
    Метрики: `kbmemfree`, `kbmemused`, `%memused`, `kbbuffers`, `kbcached`, `kbcommit`, `%commit`, `kbactive`, `kbinact`, `kbdirty`.

2.  **Нагрузка на swap:**
    ```bash
    sar -S
    ```
    Метрики: `kbswpfree`, `kbswpused`, `%swpused`, `kbswpcad`, `%swpcad`.

### Диски и ввод-вывод

1.  **Статистика по устройствам блочного ввода-вывода:**
    ```bash
    sar -b
    ```
    Метрики: `tps` (передачи в секунду), `rtps` (чтения в секунду), `wtps` (записи в секунду), `bread/s` (блоки прочитанные в секунду), `bwrtn/s` (блоки записанные в секунду).

2.  **Статистика по отдельным дисковым устройствам:**
    ```bash
    sar -d
    ```
    Показывает активность для каждого блочного устройства. Используйте `sar -dp` для более понятного вывода имен устройств.

### Сетевые метрики

1.  **Объем данных, проходящих через сетевые интерфейсы:**
    ```bash
    sar -n DEV
    ```
    Метрики: `IFACE` (имя интерфейса), `rxpck/s` (получено пакетов/сек), `txpck/s` (отправлено пакетов/сек), `rxkB/s` (получено кБ/сек), `txkB/s` (отправлено кБ/сек), `rxcmp/s` (получено сжатых пакетов/сек), `txcmp/s` (отправлено сжатых пакетов/сек), `rxmcst/s` (получено multicast пакетов/сек).

2.  **Статистика по TCP:**
    ```bash
    sar -n TCP
    ```
    Метрики: `active/s` (активные открытия TCP соединений/сек), `passive/s` (пассивные открытия TCP соединений/сек), `iseg/s` (получено сегментов/сек), `oseg/s` (отправлено сегментов/сек).

3.  **Статистика по UDP:**
    ```bash
    sar -n UDP
    ```
    Метрики: `idgrm/s` (получено UDP датаграмм/сек), `odgrm/s` (отправлено UDP датаграмм/сек), `noport/s` (ошибки "нет порта" для UDP/сек), `idgmerr/s` (ошибки при получении UDP датаграмм/сек).

4.  **Ошибки на сетевых интерфейсах:**
    ```bash
    sar -n EDEV
    ```
    Показывает статистику ошибок на сетевых интерфейсах.

### Контекстные переключения и задержки

1.  **Количество созданных задач (процессов и потоков) и контекстных переключений:**
    ```bash
    sar -w
    ```
    Метрики: `proc/s` (создано задач/сек), `cswch/s` (контекстных переключений/сек).

### Полный анализ метрик

1.  **Полная сводка за день из файла:**
    ```bash
    sar -A -f /var/log/sysstat/saXX
    ```
    Показывает все доступные метрики за указанный день.

2.  **Все метрики в реальном времени (может быть очень много вывода):**
    ```bash
    sar -A
    ```

### Другие полезные команды `sar`

*   `sar -q`: Длина очереди выполнения и средняя загрузка.
*   `sar -v`: Состояние inode, файлов и других таблиц ядра.
*   `sar -B`: Статистика подкачки страниц (пейджинга).
*   `sar -H`: Статистика использования Huge Pages.
*   `sar -m CPU`: Энергопотребление CPU (если поддерживается).

Это лишь часть возможностей утилиты `sar`. Для более подробной информации обратитесь к `man sar`. 