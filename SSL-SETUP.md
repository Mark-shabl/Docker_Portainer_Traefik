# 🔐 Настройка SSL сертификатов в Traefik

## 📋 Постоянный доступ по IP + SSL для доменов

### 1. **Запуск без SSL (по IP)**

```bash
# Используйте эту конфигурацию для постоянной работы
sudo docker-compose -f docker-compose-production.yml up -d
```

**Доступ:**
- **Portainer**: `http://YOUR_IP:9000`
- **Traefik API**: `http://YOUR_IP:8080`
- **Traefik Dashboard**: `http://YOUR_IP:8080/dashboard/`

### 2. **Добавление SSL сертификатов для доменов**

#### Вариант A: Автоматические сертификаты Let's Encrypt

1. **Настройте DNS записи** для ваших доменов:
   ```
   yourdomain.com → YOUR_IP
   *.yourdomain.com → YOUR_IP
   ```

2. **Создайте файл конфигурации SSL:**
   ```bash
   nano traefik/dynamic/ssl-config.yml
   ```

3. **Добавьте конфигурацию:**
   ```yaml
   http:
     routers:
       # Portainer с SSL
       portainer-ssl:
         rule: "Host(`portainer.yourdomain.com`)"
         service: portainer
         entryPoints:
           - websecure
         tls:
           certResolver: letsencrypt
       
       # Traefik с SSL
       traefik-ssl:
         rule: "Host(`traefik.yourdomain.com`)"
         service: api@internal
         entryPoints:
           - websecure
         tls:
           certResolver: letsencrypt
   
   tls:
     certificates:
       - certFile: /letsencrypt/certs/yourdomain.com.crt
         keyFile: /letsencrypt/certs/yourdomain.com.key
   ```

4. **Обновите docker-compose-production.yml:**
   ```yaml
   command:
     - --api.dashboard=true
     - --api.insecure=true
     - --providers.docker=true
     - --providers.docker.exposedbydefault=false
     - --providers.file.directory=/etc/traefik/dynamic
     - --providers.file.watch=true
     - --entrypoints.web.address=:80
     - --entrypoints.websecure.address=:443
     - --certificatesresolvers.letsencrypt.acme.tlschallenge=true
     - --certificatesresolvers.letsencrypt.acme.email=your-email@domain.com
     - --certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json
     - --log.level=INFO
     - --accesslog=true
   ```

#### Вариант B: Ручные сертификаты

1. **Получите SSL сертификаты** (от Let's Encrypt, Cloudflare, или другого CA)

2. **Поместите сертификаты в папку:**
   ```bash
   mkdir -p letsencrypt/certs
   # Скопируйте ваши сертификаты:
   # yourdomain.com.crt
   # yourdomain.com.key
   ```

3. **Создайте конфигурацию:**
   ```yaml
   # traefik/dynamic/ssl-config.yml
   tls:
     certificates:
       - certFile: /letsencrypt/certs/yourdomain.com.crt
         keyFile: /letsencrypt/certs/yourdomain.com.key
   
   http:
     routers:
       portainer-ssl:
         rule: "Host(`portainer.yourdomain.com`)"
         service: portainer
         entryPoints:
           - websecure
         tls: {}
   ```

### 3. **Перезапуск с SSL**

```bash
# Остановите текущие контейнеры
sudo docker-compose -f docker-compose-production.yml down

# Запустите с SSL конфигурацией
sudo docker-compose -f docker-compose-production.yml up -d
```

### 4. **Проверка работы**

- **По IP (без SSL)**: `http://YOUR_IP:9000`
- **По домену (с SSL)**: `https://portainer.yourdomain.com`

## 🔧 Полезные команды

### Проверка сертификатов:
```bash
# Проверка SSL сертификата
openssl s_client -connect yourdomain.com:443 -servername yourdomain.com

# Проверка файлов сертификатов
ls -la letsencrypt/certs/
```

### Логи Traefik:
```bash
sudo docker logs traefik -f
```

### Перезапуск только Traefik:
```bash
sudo docker restart traefik
```

## 📝 Структура файлов

```
traefik/
├── traefik.yml              # Основная конфигурация
└── dynamic/
    ├── ssl-config.yml       # SSL конфигурация
    └── remote-hosts.yml     # Удаленные хосты

letsencrypt/
├── acme.json               # Let's Encrypt данные
└── certs/                  # Ручные сертификаты
    ├── yourdomain.com.crt
    └── yourdomain.com.key
```

## 🚨 Важно!

1. **DNS записи** должны указывать на ваш IP
2. **Порты 80 и 443** должны быть открыты
3. **Файрвол** должен разрешать входящие соединения
4. **Сертификаты** должны быть действительными

**Теперь у вас есть постоянный доступ по IP + возможность SSL для доменов!** 🎉
