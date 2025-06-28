# services

# 📦 Установка и настройка популярных сервисов

## 🌐 Веб-серверы: Nginx vs Apache

### Установка Nginx
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install nginx -y
sudo systemctl enable --now nginx

# CentOS/RHEL
sudo yum install epel-release -y
sudo yum install nginx -y
sudo systemctl enable --now nginx
```

### Установка Apache
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install apache2 -y
sudo systemctl enable --now apache2

# CentOS/RHEL
sudo yum install httpd -y
sudo systemctl enable --now httpd
```

### Конфигурация Nginx (пример /etc/nginx/sites-available/example.com)
```nginx
server {
    listen 80;
    listen [::]:80;
    
    server_name example.com www.example.com;
    root /var/www/example.com/html;
    index index.html index.nginx-debian.html;
    
    location / {
        try_files $uri $uri/ =404;
    }
    
    location ~ /\.ht {
        deny all;
    }
    
    access_log /var/log/nginx/example.com.access.log;
    error_log /var/log/nginx/example.com.error.log;
}
```

### Конфигурация Apache (пример /etc/apache2/sites-available/example.com.conf)
```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    ServerAdmin webmaster@example.com
    DocumentRoot /var/www/example.com/html
    
    <Directory /var/www/example.com/html>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/example.com_error.log
    CustomLog ${APACHE_LOG_DIR}/example.com_access.log combined
</VirtualHost>
```

### Активация сайта
```bash
# Nginx
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

# Apache
sudo a2ensite example.com.conf
sudo apachectl configtest
sudo systemctl reload apache2
```

---

## 🔐 SSH: Безопасная настройка

### Основной конфиг (/etc/ssh/sshd_config)
```bash
# Изменение порта
Port 2222

# Отключение root-входа
PermitRootLogin no

# Использование только ключей
PasswordAuthentication no
PubkeyAuthentication yes

# Ограничение пользователей
AllowUsers deploy admin

# Настройки безопасности
LoginGraceTime 1m
MaxAuthTries 3
MaxSessions 3
ClientAliveInterval 300
ClientAliveCountMax 2

# Отключение устаревших протоколов
Protocol 2
```

### Генерация SSH-ключей
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### Копирование ключа на сервер
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2222 user@example.com
```

### SSH-туннелирование
```bash
# Локальный туннель (доступ к удаленному сервису как к локальному)
ssh -L 3306:localhost:3306 user@example.com

# Удаленный туннель (доступ к локальному сервису с удаленного сервера)
ssh -R 8080:localhost:80 user@example.com

# Динамическое портовое форвардинг (SOCKS proxy)
ssh -D 1080 user@example.com
```

### Настройка Jump-сервера
```bash
# ~/.ssh/config
Host jump-server
    HostName jump.example.com
    Port 2222
    User jump-user
    IdentityFile ~/.ssh/jump-key

Host internal-server
    HostName 10.0.0.5
    User internal-user
    IdentityFile ~/.ssh/internal-key
    ProxyJump jump-server
```

Подключение через Jump-сервер:
```bash
ssh internal-server
```

---

## 📁 FTP: Настройка vsftpd

### Установка
```bash
sudo apt install vsftpd -y
```

### Конфигурация (/etc/vsftpd.conf)
```bash
# Базовые настройки
listen=YES
listen_ipv6=NO
anonymous_enable=NO
local_enable=YES
write_enable=YES
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES

# Безопасность
chroot_local_user=YES
allow_writeable_chroot=YES
ssl_enable=YES
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
force_local_data_ssl=YES
force_local_logins_ssl=YES

# Ограничения
local_root=/srv/ftp
user_sub_token=$USER
local_umask=022
max_clients=10
max_per_ip=3
```

### Создание FTP-пользователя
```bash
sudo useradd -m -d /srv/ftp/user1 -s /bin/bash user1
sudo passwd user1
sudo chown root:root /srv/ftp/user1
sudo mkdir /srv/ftp/user1/files
sudo chown user1:user1 /srv/ftp/user1/files
```

### Применение настроек
```bash
sudo systemctl restart vsftpd
```

---

## 🚀 Прокачка: Развертывание сайта с SSL и защитой

### Шаг 1: Установка Certbot для Let's Encrypt
```bash
sudo apt install certbot python3-certbot-nginx -y
```

### Шаг 2: Получение SSL-сертификата
```bash
sudo certbot --nginx -d example.com -d www.example.com
```

### Шаг 3: Автоматическое продление
```bash
sudo certbot renew --dry-run
```

### Шаг 4: Настройка Nginx для HTTPS
```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    
    server_name example.com www.example.com;
    
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # Параметры безопасности SSL
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    
    # HSTS (HTTP Strict Transport Security)
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # Дополнительные заголовки безопасности
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    root /var/www/example.com/html;
    index index.html;
    
    location / {
        try_files $uri $uri/ =404;
    }
}

# Редирект HTTP -> HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}
```

### Шаг 5: Установка и настройка Fail2ban
```bash
sudo apt install fail2ban -y
```

Конфигурация для SSH (/etc/fail2ban/jail.local):
```ini
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 1h
findtime = 10m
ignoreip = 127.0.0.1/8 ::1 192.168.1.0/24
```

Конфигурация для Nginx (/etc/fail2ban/filter.d/nginx-badbots.conf):
```ini
[Definition]
failregex = ^<HOST> -.*"(GET|POST).*".*(404|444|403|400) .*$
ignoreregex =
```

Активация:
```bash
sudo systemctl enable --now fail2ban
sudo fail2ban-client status
```

### Шаг 6: Защита от ботов и DDoS
```nginx
# /etc/nginx/nginx.conf
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
    
    server {
        location / {
            limit_req zone=one burst=20;
        }
    }
}
```

---

## 🧪 Проверка безопасности

### SSL Labs Test
```bash
curl https://api.ssllabs.com/api/v3/analyze?host=example.com
```

### Security Headers Check
```bash
curl -I https://example.com
```

### Fail2ban Status
```bash
sudo fail2ban-client status sshd
```

### Firewall Rules
```bash
sudo iptables -L -n -v
```

---
## Реальный пример: мой сайт.

У меня был опыт разработки и деплоя сайта с 0. Я его использовал для дипломной работы в университете. Передо мной стояла задача в короткие сроки развернуть более менее дееспособный сайт. Так как специальность связана с брендингом, а тема - с малобюджетным продвижением, сайт может казаться недоделанным и не до конца оптимизированным под все устройства. Однако упор был сделан на то, чтобы охватить как можно больше всего, что связано с брендингом и продвижением в рекордно короткие сроки. Поэтому сделан макет, лишь с самым необходимым и оптимизация под мобильные устройства, как самые используемые.
Сайт расположен по адресу http://77.222.54.243:8888/

На нем не настроен домен и SSL так как в этом не было необходимости. А также, потому что на этом сервере помимо него работают еще два сайта для других целей. Настроено это через конфиг nginx также мной.

---

<div align="center" style="margin-top: 40px;">
  <a href="/3_VM_and_containers/README.md" style="display: inline-block; margin-right: 20px; padding: 12px 24px; background: #555; color: white; border-radius: 6px; text-decoration: none; font-weight: bold; box-shadow: 0 2px 5px rgba(0,0,0,0.2);">
    ← Назад: Виртуализация и контейнеры
  </a>
  <a href="/5_security/README.md" style="display: inline-block; padding: 12px 24px; background: #4CAF50; color: white; border-radius: 6px; text-decoration: none; font-weight: bold; box-shadow: 0 2px 5px rgba(0,0,0,0.2);">
    Далее: Безопасность и Hardening →
  </a>
</div>
