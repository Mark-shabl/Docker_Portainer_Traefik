# 🔐 Руководство по безопасности и паролям

## 📋 Пароли и настройки безопасности в проекте

### 1. **Portainer** - Веб-интерфейс управления

#### Где настраивается:
- **При первом входе** в Portainer через веб-интерфейс
- **URL**: `https://portainer.yourdomain.com`

#### Стандартные настройки:
- **Пароль**: Создается при первом входе (НЕТ стандартного пароля)
- **Пользователь**: `admin` (по умолчанию)
- **Длина пароля**: Минимум 8 символов

#### Как изменить:
1. Откройте Portainer
2. Settings → Users
3. Выберите пользователя → Edit
4. Измените пароль

#### Рекомендации:
```bash
# Сгенерируйте безопасный пароль
openssl rand -base64 32

# Или используйте онлайн генератор
# https://passwordsgenerator.net/
```

---

### 2. **Traefik Dashboard** - Мониторинг маршрутов

#### Где настраивается:
- **URL**: `https://traefik.yourdomain.com`
- **Пароль**: НЕТ (только чтение, без аутентификации)

#### Безопасность:
- Доступен только по HTTPS
- Только для мониторинга (не для управления)
- Можно добавить аутентификацию через middleware

---

### 3. **Docker API** - Удаленный доступ

#### Где настраивается:
- **Порт**: `2375` (на сервисных машинах)
- **Аутентификация**: НЕТ (только по IP)

#### Безопасность:
```bash
# Ограничьте доступ только с центральной машины
sudo ufw allow from CENTRAL_MACHINE_IP to any port 2375
sudo ufw deny 2375

# Или используйте VPN
```

---

### 4. **База данных PostgreSQL** (в примере)

#### Где настраивается:
- **Файл**: `docker-compose-services.yml`
- **Строки**: 48-51

#### Стандартные настройки:
```yaml
environment:
  - POSTGRES_DB=myapp
  - POSTGRES_USER=user
  - POSTGRES_PASSWORD=password
```

#### Как изменить:
```yaml
environment:
  - POSTGRES_DB=your_database_name
  - POSTGRES_USER=your_username
  - POSTGRES_PASSWORD=your_secure_password
```

#### Рекомендации:
```bash
# Сгенерируйте безопасный пароль
openssl rand -base64 32

# Пример безопасного пароля
POSTGRES_PASSWORD=Kj8#mN2$pL9@vR4!wX7&qZ1
```

---

### 5. **Let's Encrypt** - SSL сертификаты

#### Где настраивается:
- **Файл**: `.env`
- **Переменная**: `ACME_EMAIL`

#### Стандартные настройки:
```bash
ACME_EMAIL=your-email@example.com
```

#### Как изменить:
```bash
# Отредактируйте .env файл
nano .env

# Измените email
ACME_EMAIL=your-real-email@domain.com
```

---

## 🛡️ Рекомендации по безопасности

### 1. **Обязательные изменения:**

```bash
# 1. Измените пароли в docker-compose-services.yml
POSTGRES_PASSWORD=your_secure_password_here

# 2. Настройте email для SSL
ACME_EMAIL=your-real-email@domain.com

# 3. Ограничьте доступ к Docker API
sudo ufw allow from CENTRAL_MACHINE_IP to any port 2375
sudo ufw deny 2375
```

### 2. **Дополнительная безопасность:**

#### Добавьте аутентификацию к Traefik Dashboard:
```yaml
# В docker-compose.yml добавьте middleware
labels:
  - "traefik.http.middlewares.auth.basicauth.users=admin:$$2y$$10$$..."
  - "traefik.http.routers.traefik.middlewares=auth"
```

#### Используйте переменные окружения для паролей:
```yaml
# В docker-compose-services.yml
environment:
  - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
```

#### Создайте .env файл с паролями:
```bash
# .env
POSTGRES_PASSWORD=your_secure_password
ACME_EMAIL=your-email@domain.com
```

### 3. **Мониторинг безопасности:**

```bash
# Проверьте открытые порты
sudo netstat -tlnp | grep :2375

# Проверьте логи
docker-compose logs -f

# Проверьте подключения
sudo ss -tlnp | grep :2375
```

---

## 🚨 Критически важно!

### ❌ НЕ ДЕЛАЙТЕ:
- Не оставляйте стандартные пароли
- Не открывайте порт 2375 для всех
- Не используйте простые пароли
- Не забывайте обновлять сертификаты

### ✅ ОБЯЗАТЕЛЬНО:
- Измените все пароли по умолчанию
- Ограничьте доступ к Docker API
- Используйте сильные пароли
- Регулярно обновляйте систему

---

## 📝 Чек-лист безопасности

- [ ] Изменен пароль Portainer
- [ ] Изменен пароль PostgreSQL
- [ ] Настроен email для Let's Encrypt
- [ ] Ограничен доступ к порту 2375
- [ ] Настроен файрвол
- [ ] Созданы резервные копии
- [ ] Настроено логирование
- [ ] Регулярные обновления

**Помните: безопасность - это процесс, а не разовое действие!** 🔒
