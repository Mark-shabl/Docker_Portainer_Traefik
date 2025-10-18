# 🚀 Portainer + Traefik - Простая настройка

## Архитектура:
- **Центральная машина**: Portainer + Traefik
- **Сервисные машины**: Только Docker сервисы

---

## 1. Центральная машина (Portainer + Traefik)

### Настройка:
```bash
# 1. Скопируйте конфигурацию
cp env.example .env

# 2. Отредактируйте .env (измените домены и email)
nano .env

# 3. Создайте директории
mkdir -p letsencrypt traefik/dynamic

# 4. Запустите
docker-compose up -d
```

### Доступ:
- **Portainer**: `https://portainer.yourdomain.com`
- **Traefik Dashboard**: `https://traefik.yourdomain.com`

---

## 2. Сервисные машины

### Настройка Docker для удаленного доступа:

```bash
# 1. Создайте конфигурацию Docker
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo tee /etc/systemd/system/docker.service.d/override.conf > /dev/null <<EOF
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375
EOF

# 2. Перезапустите Docker
sudo systemctl daemon-reload
sudo systemctl restart docker

# 3. Откройте порт в файрволе
sudo ufw allow 2375/tcp
```

### Подключение к Portainer:
1. Откройте Portainer на центральной машине
2. Settings → Environments → Add environment
3. Docker → Remote
4. URL: `tcp://IP_СЕРВИСНОЙ_МАШИНЫ:2375`
5. Public IP: `IP_СЕРВИСНОЙ_МАШИНЫ`

### Запуск сервисов:
```bash
# Используйте docker-compose-services.yml как пример
docker-compose -f docker-compose-services.yml up -d
```

---

## 3. Лейблы Traefik для сервисов

Добавляйте к контейнерам на сервисных машинах:

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.myapp.rule=Host(`myapp.yourdomain.com`)"
  - "traefik.http.routers.myapp.entrypoints=websecure"
  - "traefik.http.routers.myapp.tls.certresolver=letsencrypt"
  - "traefik.http.services.myapp.loadbalancer.server.port=8080"
```

---

## 4. Полезные команды

### Центральная машина:
```bash
# Статус
docker-compose ps

# Логи
docker-compose logs -f

# Остановка
docker-compose down
```

### Сервисные машины:
```bash
# Статус
docker ps

# Логи
docker logs CONTAINER_NAME

# Перезапуск
docker restart CONTAINER_NAME
```

---

## 5. Структура файлов

```
Центральная машина:
├── docker-compose.yml          # Portainer + Traefik
├── .env                        # Конфигурация
├── traefik/                    # Конфигурация Traefik
└── letsencrypt/                # SSL сертификаты

Сервисные машины:
├── docker-compose-services.yml # Ваши сервисы
└── html/                       # Статические файлы (если нужны)
```

---

## 6. Решение проблем

### Сервисная машина не подключается:
```bash
# Проверьте порт
telnet IP_СЕРВИСНОЙ_МАШИНЫ 2375

# Проверьте файрвол
sudo ufw status
```

### Сервис не появляется в Traefik:
1. Проверьте лейблы: `docker inspect CONTAINER_NAME`
2. Убедитесь, что лейблы правильные
3. Проверьте логи Traefik: `docker-compose logs traefik`

### Не работает SSL:
1. Проверьте DNS записи
2. Убедитесь, что домен указывает на центральную машину
3. Проверьте email в .env файле

---

**Готово! Теперь все управляется через docker-compose up -d** 🎉
