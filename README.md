# Portainer + Traefik для управления Docker контейнерами

Простая настройка централизованного управления Docker контейнерами на нескольких виртуальных машинах.

## 🚀 Архитектура

- **Центральная машина**: Portainer + Traefik (управляет всем)
- **Сервисные машины**: Только Docker сервисы (подключаются к центральной)

## 📋 Быстрый старт

### 1. Центральная машина

```bash
# Настройка конфигурации
cp env.example .env
nano .env  # Измените домены и email

# Создание директорий
mkdir -p letsencrypt traefik/dynamic

# Запуск
docker-compose up -d
```

### 2. Сервисные машины

```bash
# Настройка Docker для удаленного доступа
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo tee /etc/systemd/system/docker.service.d/override.conf > /dev/null <<EOF
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker
sudo ufw allow 2375/tcp

# Запуск сервисов
docker-compose -f docker-compose-services.yml up -d
```

### 3. Подключение к Portainer

1. Откройте `https://portainer.yourdomain.com`
2. Settings → Environments → Add environment
3. Docker → Remote
4. URL: `tcp://IP_СЕРВИСНОЙ_МАШИНЫ:2375`

## 🔧 Лейблы Traefik

Добавляйте к контейнерам на сервисных машинах:

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.myapp.rule=Host(`myapp.yourdomain.com`)"
  - "traefik.http.routers.myapp.entrypoints=websecure"
  - "traefik.http.routers.myapp.tls.certresolver=letsencrypt"
  - "traefik.http.services.myapp.loadbalancer.server.port=8080"
```

## 📁 Структура проекта

```
├── docker-compose.yml              # Portainer + Traefik (центральная машина)
├── docker-compose-services.yml     # Пример сервисов (сервисные машины)
├── traefik/                        # Конфигурация Traefik
├── letsencrypt/                    # SSL сертификаты
├── env.example                     # Пример конфигурации
├── INSTRUCTIONS.md                 # Подробные инструкции
└── README.md                       # Этот файл
```

## 🌐 Доступ

- **Portainer**: `https://portainer.yourdomain.com`
- **Traefik Dashboard**: `https://traefik.yourdomain.com`
- **Ваши сервисы**: `https://myapp.yourdomain.com`

## 📖 Подробные инструкции

Смотрите [INSTRUCTIONS.md](INSTRUCTIONS.md) для детальной настройки.

## 🔒 Безопасность

- Все соединения зашифрованы TLS
- Автоматические SSL сертификаты от Let's Encrypt
- Ограничьте доступ к порту 2375 файрволом

## 🆘 Решение проблем

1. Проверьте логи: `docker-compose logs -f`
2. Убедитесь в правильности DNS записей
3. Проверьте доступность портов 80, 443, 2375
