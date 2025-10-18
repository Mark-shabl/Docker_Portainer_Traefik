# Portainer + Traefik для управления Docker контейнерами

Простая настройка централизованного управления Docker контейнерами на нескольких виртуальных машинах.

## 🚀 Архитектура

- **Центральная машина**: Portainer + Traefik (управляет всем)
- **Сервисные машины**: Только Docker сервисы (подключаются к центральной)

## 📋 Быстрый старт

### 1. Центральная машина

```bash
# Клонируйте репозиторий
git clone https://github.com/Mark-shabl/Docker_Portainer_Traefik.git
cd Docker_Portainer_Traefik

# Создание директорий
mkdir -p letsencrypt traefik/dynamic

# Запуск
docker-compose up -d
```

### 2. Доступ к админкам

- **Portainer**: `http://YOUR_IP:9000`
- **Traefik Dashboard**: `http://YOUR_IP:8080`
- **Traefik API**: `http://YOUR_IP:8080/api/rawdata`

### 3. Сервисные машины

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
```

### 4. Подключение к Portainer

1. Откройте `http://YOUR_IP:9000`
2. Создайте администратора
3. Settings → Environments → Add environment
4. Docker → Remote
5. URL: `tcp://IP_СЕРВИСНОЙ_МАШИНЫ:2375`

## 🔧 Лейблы Traefik

Добавляйте к контейнерам на сервисных машинах:

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.myapp.rule=Host(`myapp.yourdomain.com`)"
  - "traefik.http.routers.myapp.entrypoints=websecure"
  - "traefik.http.services.myapp.loadbalancer.server.port=8080"
```

## 📁 Структура проекта

```
├── docker-compose.yml              # Portainer + Traefik
├── docker-compose-services.yml     # Пример сервисов
├── traefik/                        # Конфигурация Traefik
├── letsencrypt/                    # SSL сертификаты
├── env.example                     # Пример конфигурации
├── SSL-SETUP.md                    # Настройка SSL
└── README.md                       # Этот файл
```

## 🌐 Доступ

- **Portainer**: `http://YOUR_IP:9000`
- **Traefik Dashboard**: `http://YOUR_IP:8080`
- **Ваши сервисы**: `http://YOUR_IP` (через Traefik)

## 🔐 SSL сертификаты

Для добавления SSL сертификатов смотрите [SSL-SETUP.md](SSL-SETUP.md)

## 🆘 Решение проблем

1. Проверьте логи: `docker-compose logs -f`
2. Проверьте статус: `docker-compose ps`
3. Проверьте порты: `netstat -tlnp | grep :9000`
