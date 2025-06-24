#Linux-usage

В этом разделе я демонстрирую навыки работы с Linux системами на практических примерах.

## 💻 Опыт работы с дистрибутивами

Я лично устанавливал, переустанавливал, ломал и исправлял множество дистрибутивов Linux:
- **Для рабочих станций**: Ubuntu, Fedora, Linux Lite, Kali, Parrot, BlackArch
- **Для серверов**: Debian, CentOS, Alpine

На физических машинах я уверенно:
- Создаю загрузочные носители (Rufus, Etcher, `dd`)
- Управляю разделами диска (GParted, установщики, CLI)
- Выполняю тонкую настройку систем

На виртуальных машинах:
- Оптимизирую под конкретные задачи
- Настраиваю баланс производительности/безопасности
- Использую различные технологии виртуализации

**Мой выбор для серверов - Debian** благодаря:
- Стабильности и предсказуемости
- Широкому выбору ПО в репозиториях
- Богатой документации и сообществу
- Популярности в enterprise-среде

## 🖥️ Практика на VPS (Debian 12)

### 🔌 Подключение и базовые команды
```bash
# Подключаемся по SSH
ssh admin@aka0601-vps-5

# Проверяем текущую директорию
admin@aka0601-vps-5:~$ pwd
/home/admin

# Смотрим содержимое (включая скрытые файлы)
admin@aka0601-vps-5:~$ ls -A
.bash_history  .bash_logout  .bashrc  .cache  .config  .profile  
.sudo_as_admin_successful  myweb  myweb-main.zip
```

### 🔍 Анализ процессов
```bash
# Все запущенные процессы
admin@aka0601-vps-5:~$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  1.1 102096 12068 ?        Ss   04:46   0:00 /sbin/init
www-data     370  0.0  0.6  21300  6868 ?        S    04:46   0:00 nginx: worker process
admin        382  0.0  3.4  41472 35152 ?        S    04:46   0:00 /home/admin/myweb/venv/bin/python3 ...
... [сокращено для читаемости] ...

# Поиск процессов с высокой нагрузкой CPU
admin@aka0601-vps-5:~$ ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | awk '$5 > 10'
    PID    PPID CMD                         %MEM %CPU
    427     368 sshd: admin [priv]           1.0  0.5
    451     450 -bash                        0.4  0.3

# Проверка на zombie-процессы
admin@aka0601-vps-5:~$ ps aux | awk '$8=="Z" {print $2, $11}'
[вывод пуст - хороший знак!]
```

### 📊 Мониторинг ресурсов (top)
```bash
admin@aka0601-vps-5:~$ top
Tasks:  73 total,   2 running,  71 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.3 si,  0.0 st 
MiB Mem :   1000.8 total,    715.5 free,    287.7 used,    132.1 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.    713.1 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
     15 root      20   0       0      0      0 R   0.3   0.0   0:00.97 rcu_preempt
    450 admin     20   0   17680   6576   4680 S   0.3   0.6   0:00.07 sshd
      1 root      20   0  102096  12068   9232 S   0.0   1.2   0:00.43 systemd
... [сокращено] ...
```

### 🌐 Сетевые соединения
```bash
# Просмотр всех слушающих портов
admin@aka0601-vps-5:~$ sudo netstat -tulpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address    Foreign Address State    PID/Program name    
tcp        0      0 0.0.0.0:22      0.0.0.0:*        LISTEN   368/sshd: /usr/sbin 
tcp        0      0 0.0.0.0:80      0.0.0.0:*        LISTEN   369/nginx: master p 
tcp        0      0 0.0.0.0:443     0.0.0.0:*        LISTEN   369/nginx: master p 
... [сокращено] ...

# Проверка активных SSH-соединений
admin@aka0601-vps-5:~$ ss -t -a sport = :22
State    Recv-Q   Send-Q     Local Address:Port     Peer Address:Port   Process                 
LISTEN   0        128             0.0.0.0:ssh           0.0.0.0:*                  
ESTAB    0        52        77.222.54.243:ssh     178.71.122.75:56742               
LISTEN   0        128                [::]:ssh              [::]:*                  
```

### 💾 Дисковое пространство
```bash
# Обзор использования диска
admin@aka0601-vps-5:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1       9.9G  1.7G  7.7G  18% /
tmpfs           501M     0  501M   0% /dev/shm

# Поиск разделов с заполнением >10%
admin@aka0601-vps-5:~$ df -h | awk '$5 >= "10%" {print $6 " " $5}'
/ 18%

# Поиск больших лог-файлов
admin@aka0601-vps-5:~$ sudo find /var/log -type f -size +10M -exec du -h {} \;
41M     /var/log/journal/...journal
17M     /var/log/nginx/access.log
```

### ⚙️ Управление сервисами
```bash
# Активные сервисы
admin@aka0601-vps-5:~$ sudo systemctl list-units --type=service --state=running
  UNIT                      LOAD   ACTIVE SUB     DESCRIPTION             
  nginx.service             loaded active running A high performance web server
  ssh.service               loaded active running OpenBSD Secure Shell server
  kalininantoxa.service     loaded active running Gunicorn сервер для myproject
... [сокращено] ...

# Детальный статус сервиса
admin@aka0601-vps-5:~$ sudo systemctl status kalininantoxa
● kalininantoxa.service - Gunicorn сервер для myproject
     Loaded: loaded (/etc/systemd/system/kalininantoxa.service; enabled; preset: enabled)
     Active: active (running) since Tue 2025-06-24 04:46:22 MSK; 31min ago
   Main PID: 361 (gunicorn)
      Tasks: 4 (limit: 1173)
     Memory: 81.1M
        CPU: 1.075s
... [сокращено] ...

# Просмотр логов сервиса
admin@aka0601-vps-5:~$ sudo journalctl --unit=kalininantoxa.service | grep 'Jun 24 '
Jun 24 04:46:22 aka0601-vps-5 systemd[1]: Started kalininantoxa.service
Jun 24 04:46:23 aka0601-vps-5 gunicorn[361]: [INFO] Starting gunicorn 23.0.0
... [сокращено] ...
```

### 🛠️ Конфигурация сервиса
```bash
admin@aka0601-vps-5:~$ sudo cat /etc/systemd/system/kalininantoxa.service
[Unit]
Description=Gunicorn сервер для myproject
After=network.target

[Service]
User=admin
Group=www-data
WorkingDirectory=/home/admin/myweb
Environment="PATH=/home/admin/myweb/venv/bin"
ExecStart=/home/admin/myweb/venv/bin/gunicorn --workers 3 --bind unix:myweb.sock -m 007 wsgi:app

[Install]
WantedBy=multi-user.target
```

### 🔎 Анализ безопасности
```bash
# Поиск перезагрузок в логах
admin@aka0601-vps-5:~$ sudo journalctl -b -1 --since "10 minutes ago" | grep -i 'reboot\|panic\|error'
[вывод пуст - система стабильна]

# Мониторинг попыток взлома SSH
admin@aka0601-vps-5:~$ sudo journalctl -u sshd | grep 'Failed password'
[вывод пуст - хороший признак безопасности]
```