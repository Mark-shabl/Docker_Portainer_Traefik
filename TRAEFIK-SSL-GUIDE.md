# 🔐 Управление SSL сертификатами в Traefik

## 📋 Как работает SSL в Traefik (как в Nginx Proxy Manager)

Traefik автоматически управляет SSL сертификатами! Вам не нужно вручную загружать сертификаты - Traefik сам их получает и обновляет.

## 🚀 Быстрый старт

### 1. **Запуск с SSL поддержкой**

```bash
# Клонируйте репозиторий
git clone https://github.com/Mark-shabl/Docker_Portainer_Traefik.git
cd Docker_Portainer_Traefik

# Настройте .env файл
cp env.example .env
nano .env

# Измените домены и email
TRAEFIK_DOMAIN=traefik.yourdomain.com
PORTAINER_DOMAIN=portainer.yourdomain.com
ACME_EMAIL=your-email@domain.com

# Запуск
mkdir -p letsencrypt traefik/dynamic
docker-compose up -d
```

### 2. **Доступ к админкам**

- **Portainer**: `http://YOUR_IP:9000` (HTTP) или `https://portainer.yourdomain.com` (HTTPS)
- **Traefik Dashboard**: `http://YOUR_IP:8080`
- **Traefik API**: `http://YOUR_IP:8080/api/rawdata`

## 🔧 Управление SSL сертификатами

### **Автоматические сертификаты (рекомендуется)**

Traefik автоматически получает и обновляет SSL сертификаты от Let's Encrypt:

1. **Настройте DNS записи** для ваших доменов:
   ```
   portainer.yourdomain.com → YOUR_IP
   traefik.yourdomain.com → YOUR_IP
   ```

2. **Добавьте лейблы к контейнерам:**
   ```yaml
   labels:
     - "traefik.enable=true"
     - "traefik.http.routers.myapp.rule=Host(`myapp.yourdomain.com`)"
     - "traefik.http.routers.myapp.entrypoints=websecure"
     - "traefik.http.routers.myapp.tls.certresolver=letsencrypt"
     - "traefik.http.services.myapp.loadbalancer.server.port=8080"
   ```

3. **Traefik автоматически:**
   - Получит SSL сертификат от Let's Encrypt
   - Обновит сертификат перед истечением
   - Настроит HTTPS редирект

### **Ручные сертификаты**

Если у вас есть собственные SSL сертификаты:

1. **Поместите сертификаты в папку:**
   ```bash
   mkdir -p letsencrypt/certs
   # Скопируйте ваши сертификаты:
   # yourdomain.com.crt
   # yourdomain.com.key
   ```

2. **Создайте конфигурацию:**
   ```yaml
   # traefik/dynamic/ssl-config.yml
   tls:
     certificates:
       - certFile: /letsencrypt/certs/yourdomain.com.crt
         keyFile: /letsencrypt/certs/yourdomain.com.key
   ```

## 🌐 Добавление новых сервисов с SSL

### **Через docker-compose.yml**

```yaml
version: '3.8'

services:
  myapp:
    image: nginx:alpine
    container_name: myapp
    restart: unless-stopped
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`myapp.yourdomain.com`)"
      - "traefik.http.routers.myapp.entrypoints=websecure"
      - "traefik.http.routers.myapp.tls.certresolver=letsencrypt"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
    networks:
      - traefik

networks:
  traefik:
    external: true
```

### **Через Portainer UI**

1. Откройте Portainer
2. Создайте новый контейнер
3. В разделе "Labels" добавьте:
   ```
   traefik.enable = true
   traefik.http.routers.myapp.rule = Host(`myapp.yourdomain.com`)
   traefik.http.routers.myapp.entrypoints = websecure
   traefik.http.routers.myapp.tls.certresolver = letsencrypt
   traefik.http.services.myapp.loadbalancer.server.port = 80
   ```

## 📊 Мониторинг SSL сертификатов

### **Через Traefik Dashboard**

1. Откройте `http://YOUR_IP:8080`
2. Перейдите в раздел "HTTP"
3. Посмотрите на "Routers" - там будут все ваши сервисы
4. Зеленый замок = SSL работает

### **Через API**

```bash
# Получить информацию о сертификатах
curl http://YOUR_IP:8080/api/rawdata | jq '.tls.certificates'

# Проверить конкретный домен
curl -I https://yourdomain.com
```

## 🔧 Полезные команды

### **Проверка SSL сертификатов:**
```bash
# Проверка сертификата
openssl s_client -connect yourdomain.com:443 -servername yourdomain.com

# Проверка файлов сертификатов
ls -la letsencrypt/

# Проверка логов Traefik
docker logs traefik -f
```

### **Перезапуск сервисов:**
```bash
# Перезапуск Traefik
docker restart traefik

# Перезапуск всех сервисов
docker-compose restart
```

## 🚨 Решение проблем

### **SSL не работает:**
1. Проверьте DNS записи
2. Убедитесь, что порты 80 и 443 открыты
3. Проверьте логи: `docker logs traefik`
4. Убедитесь, что домен указывает на ваш IP

### **Let's Encrypt ошибки:**
1. Проверьте email в .env файле
2. Убедитесь, что домен доступен по HTTP
3. Проверьте лимиты Let's Encrypt

### **Сертификат не обновляется:**
1. Проверьте права доступа к папке letsencrypt
2. Убедитесь, что Traefik может писать в папку
3. Проверьте логи на ошибки

## 📝 Структура файлов

```
letsencrypt/
├── acme.json              # Данные Let's Encrypt
└── certs/                 # Ручные сертификаты
    ├── yourdomain.com.crt
    └── yourdomain.com.key

traefik/
├── traefik.yml            # Основная конфигурация
└── dynamic/
    └── ssl-config.yml     # Дополнительные SSL настройки
```

## 🎉 Преимущества Traefik перед Nginx Proxy Manager

- ✅ **Автоматическое управление** SSL сертификатами
- ✅ **Автоматическое обнаружение** новых контейнеров
- ✅ **Автоматическое обновление** сертификатов
- ✅ **Простая настройка** через лейблы
- ✅ **Веб-интерфейс** для мониторинга
- ✅ **API** для интеграции

**Traefik работает как Nginx Proxy Manager, но автоматически!** 🚀
