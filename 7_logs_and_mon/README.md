# logs_and_mon

# 📈 Мониторинг и логирование: Полное руководство

## 🔍 Утилиты командной строки для мониторинга

### 1. top - Мониторинг процессов в реальном времени
```bash
top -c -d 2 -u www-data
```
**Пример вывода:**
```
top - 14:30:45 up 45 days,  3:15,  1 user,  load average: 0.12, 0.08, 0.05
Tasks: 125 total,   1 running, 124 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.3 us,  1.2 sy,  0.0 ni, 96.5 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7986.8 total,   1024.5 free,   2048.3 used,   4914.0 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   5678.2 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 1234 www-data  20   0  724768  45832  21564 S   2.3   0.6   5:30.67 nginx: worker process
 5678 mysql     20   0 2567892 1.2g   48764 S   1.2  15.8 120:45.89 /usr/sbin/mysqld
```

**Ключевые комбинации:**
- `P` - сортировка по CPU
- `M` - сортировка по памяти
- `1` - показ нагрузки по ядрам
- `k` - завершение процесса
- `f` - добавление/удаление колонок

### 2. htop - Улучшенный интерактивный мониторинг
```bash
sudo apt install htop
htop -u nginx
```
**Особенности:**
- Цветной интерфейс
- Вертикальное и горизонтальное разделение
- Простое управление процессами (F9 - сигналы)
- Поиск процессов (F3)
- Настраиваемый вид

### 3. iotop - Мониторинг дисковых операций
```bash
sudo iotop -o -P -d 5
```
**Пример вывода:**
```
Total DISK READ: 1.25 M/s | Total DISK WRITE: 4.76 M/s
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND
 7890 be/4 mysql      1.25 M/s    3.15 M/s  0.00 % 25.15 % mysqld
 1234 be/3 backup     0.00 B/s    1.61 M/s  0.00 % 10.45 % rsync
```

**Ключевые опции:**
- `-o` - показ только активных процессов
- `-P` - показ только процессов
- `-a` - аккумулятивный вывод
- `-d` - интервал обновления

### 4. netstat/ss - Сетевые соединения
```bash
# Традиционный netstat
netstat -tulpn

# Современный ss
ss -tunp4
ss -s  # Статистика
```

**Пример вывода ss:**
```
State      Recv-Q     Send-Q         Local Address:Port           Peer Address:Port     
ESTAB      0          0              192.168.1.100:22             192.168.1.50:54322     
LISTEN     0          128                  0.0.0.0:80                  0.0.0.0:*        
```

### 5. iftop - Мониторинг сетевого трафика
```bash
sudo iftop -i eth0 -n -P
```
**Интерфейс:**
```
interface: eth0
IP: 192.168.1.100 => 203.0.113.5
TX:             cum:   3.02MB   peak:   2.23Mb    rates:   1.45Mb  1.23Mb  1.01Mb
RX:                     2.76MB           1.98Mb            1.21Mb  1.05Mb  0.87Mb
TOTAL:                  5.78MB           4.21Mb            2.66Mb  2.28Mb  1.88Mb

203.0.113.5:443 => 192.168.1.100:55678    1.23Mb  1.05Mb  0.98Mb
104.16.118.5:443 => 192.168.1.100:44322   0.98Mb  0.87Mb  0.76Mb
```

**Ключевые комбинации:**
- `P` - показ портов
- `n` - отключение DNS-разрешения
- `t` - переключение режима вывода
- `s` - показ исходного IP
- `d` - показ IP назначения

## 📊 Установка Netdata для комплексного мониторинга

### Установка
```bash
# Автоматическая установка
bash <(curl -Ss https://my-netdata.io/kickstart.sh)

# Ручная установка
sudo apt install netdata
sudo systemctl enable --now netdata
```

### Конфигурация
```bash
# Основной конфиг
sudo nano /etc/netdata/netdata.conf

# Включение HTTPS
[web]
    bind to = 127.0.0.1
    allow connections from = localhost
    enable web responses gzip compression = yes
    ssl key = /etc/netdata/ssl/key.pem
    ssl certificate = /etc/netdata/ssl/cert.pem
```

### Настройка оповещений
```bash
sudo nano /etc/netdata/health.d/cpu.conf

# Пример алерта на высокую загрузку CPU
alarm: cpu_usage
    on: system.cpu
    lookup: average -10s percentage of usage
    every: 1m
    warn: $this > 80
    crit: $this > 90
    info: CPU utilization
```

### Доступ к веб-интерфейсу
```
http://your-server:19999
```

**Ключевые возможности Netdata:**
- Реальный мониторинг с задержкой 1 секунда
- Более 2000 метрик из коробки
- Интеграция с Prometheus, Grafana
- Плагины для Docker, Nginx, MySQL
- Настраиваемые оповещения

## 📖 Работа с системными логами

### 1. journalctl - Просмотр системных логов systemd
```bash
# Просмотр всех логов
journalctl

# Фильтрация по службе
journalctl -u nginx.service

# Логи за последние 2 часа
journalctl --since "2 hours ago"

# Поиск по ключевому слову
journalctl -k --grep="error"

# Слежение за логами в реальном времени
journalctl -f

# Логи конкретного пользователя
journalctl _UID=1000
```

### 2. Анализ файлов логов
```bash
# Аутентификация
sudo tail -f /var/log/auth.log

# Системные сообщения
sudo less /var/log/syslog

# Ядро
dmesg | grep -i error

# Логи Apache
tail -f /var/log/apache2/access.log
tail -f /var/log/apache2/error.log

# Логи Nginx
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

### 3. Полезные команды для анализа
```bash
# Топ IP в access.log
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head

# Поиск ошибок 500
grep ' 500 ' /var/log/nginx/access.log | awk '{print $7}' | sort | uniq -c | sort -nr

# Статистика HTTP-методов
awk '{print $6}' /var/log/nginx/access.log | cut -d'"' -f2 | sort | uniq -c

# Поиск медленных запросов (>5 сек)
awk '($NF > 5) {print $1, $4, $NF, $7}' /var/log/nginx/access.log
```

## 🔄 Настройка logrotate

### Пример конфигурации для Nginx
```bash
sudo nano /etc/logrotate.d/nginx

/var/log/nginx/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    prerotate
        if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
            run-parts /etc/logrotate.d/httpd-prerotate; \
        fi \
    endscript
    postrotate
        invoke-rc.d nginx rotate >/dev/null 2>&1
    endscript
}
```

**Ключевые директивы:**
- `daily` - ротация ежедневно
- `rotate 30` - хранить 30 архивов
- `compress` - сжатие gzip
- `delaycompress` - сжимать предыдущий лог
- `missingok` - игнорировать отсутствие файлов
- `notifempty` - не ротировать пустые файлы
- `create` - права на новый файл

### Принудительный запуск logrotate
```bash
sudo logrotate -f /etc/logrotate.conf
```

## 🧪 Практические кейсы

### Кейс 1: Анализ атак на SSH
```bash
# Поиск неудачных попыток входа
sudo grep "Failed password" /var/log/auth.log

# Топ атакующих IP
sudo grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr | head

# Блокировка IP с помощью fail2ban
sudo fail2ban-client set sshd banip 123.45.67.89
```

### Кейс 2: Диагностика медленного сайта
```bash
# Поиск медленных запросов в Nginx
awk '($NF > 5) {print $1, $4, $NF, $7}' /var/log/nginx/access.log | sort -k3 -nr | head

# Анализ времени ответа
cat /var/log/nginx/access.log | awk '{print $1, $NF}' | sort -k2 -nr | head

# Мониторинг в реальном времени
tail -f /var/log/nginx/access.log | awk '{if ($NF > 1) print}'
```

### Кейс 3: Обнаружение утечки памяти
```bash
# Мониторинг памяти в htop
htop --sort-key=PERCENT_MEM

# Поиск процессов с растущей памятью
ps aux --sort=-%mem | head

# Анализ использования памяти приложением
sudo pmap -x $(pgrep myapp) | less
```

## 📊 Интеграция мониторинга

### Prometheus + Grafana
```bash
# Установка Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.36.1/prometheus-2.36.1.linux-amd64.tar.gz
tar xvfz prometheus-*.tar.gz
cd prometheus-*

# Конфигурация
cat <<EOF > prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
  - job_name: 'netdata'
    metrics_path: '/api/v1/allmetrics'
    params:
      format: ['prometheus']
    static_configs:
      - targets: ['localhost:19999']
EOF

# Запуск
./prometheus --config.file=prometheus.yml
```

### Дашборд в Grafana
1. Импортируйте дашборд Node Exporter Full (ID: 1860)
2. Настройте источники данных:
   - Prometheus: http://localhost:9090
   - Netdata: http://localhost:19999

## 💡 Лучшие практики

1. **Централизованное логирование**
   - Используйте ELK Stack (Elasticsearch, Logstash, Kibana)
   - Или Loki + Grafana для облачных сред

2. **Ротация логов**
   - Настраивайте разумные периоды хранения
   - Учитывайте требования комплаенса
   - Мониторьте использование диска

3. **Оповещения**
   - Настройка алертов для критичных метрик
   - Интеграция с Slack, Telegram, PagerDuty
   - Регулярный пересмотр правил оповещений

4. **Аудит безопасности**
   - Регулярный анализ логов аутентификации
   - Мониторинг необычной активности
   - Интеграция с SIEM-системами

---

<div align="center" style="margin-top: 40px;">
  <a href="/6_scripts/README.md" style="display: inline-block; margin-right: 20px; padding: 12px 24px; background: #555; color: white; border-radius: 6px; text-decoration: none; font-weight: bold; box-shadow: 0 2px 5px rgba(0,0,0,0.2);">
    ← Назад: Скрипты и автоматизация
  </a>
  <a href="/1_linux_usage/README.md" style="display: inline-block; padding: 12px 24px; background: #4CAF50; color: white; border-radius: 6px; text-decoration: none; font-weight: bold; box-shadow: 0 2px 5px rgba(0,0,0,0.2);">
    В начало: Использование Linux →
  </a>
</div>
