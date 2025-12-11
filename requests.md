# Полное руководство по тестированию API с помощью Postman

## Подготовка

### 1. Запуск сервера
```bash
python manage.py runserver
```
Сервер будет доступен по адресу: `http://127.0.0.1:8000`

### 2. Настройка Postman

Создайте новую коллекцию "Blog API Tests" и настройте переменные окружения:

**Environment Variables:**
- `base_url`: `http://127.0.0.1:8000`
- `access_token`: (будет заполнено автоматически)
- `refresh_token`: (будет заполнено автоматически)

---

## 1. Регистрация пользователя

### Endpoint: POST `/api/v1/auth/register/`

**URL:** `{{base_url}}/api/v1/auth/register/`

**Headers:**
```
Content-Type: application/json
```

**Body (raw JSON):**
```json
{
    "username": "testuser",
    "email": "testuser@example.com",
    "password": "SecurePass123!",
    "password_confirm": "SecurePass123!",
    "first_name": "Test",
    "last_name": "User"
}
```

**Ожидаемый ответ (201 Created):**
```json
{
    "user": {
        "id": 1,
        "username": "testuser",
        "email": "testuser@example.com",
        "first_name": "Test",
        "last_name": "User",
        "full_name": "Test User",
        "avatar": null,
        "bio": "",
        "created_at": "2025-11-28T12:00:00Z",
        "updated_at": "2025-11-28T12:00:00Z",
        "posts_count": 0,
        "comments_count": 0
    },
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc...",
    "access": "eyJ0eXAiOiJKV1QiLCJhbGc...",
    "message": "User regirstered successfully"
}
```

**Tests (вкладка Tests в Postman):**
```javascript
if (pm.response.code === 201) {
    const response = pm.response.json();
    pm.environment.set("access_token", response.access);
    pm.environment.set("refresh_token", response.refresh);
    pm.test("Status code is 201", () => {
        pm.response.to.have.status(201);
    });
    pm.test("User registered successfully", () => {
        pm.expect(response.message).to.include("successfully");
    });
    pm.test("Access token exists", () => {
        pm.expect(response.access).to.exist;
    });
}
```

**Проверка ошибок:**

1. **Дублирование email:**
```json
{
    "username": "testuser2",
    "email": "testuser@example.com",
    "password": "SecurePass123!",
    "password_confirm": "SecurePass123!"
}
```
Ожидается: **400 Bad Request**

2. **Несовпадение паролей:**
```json
{
    "username": "testuser3",
    "email": "testuser3@example.com",
    "password": "SecurePass123!",
    "password_confirm": "DifferentPass123!"
}
```
Ожидается: **400 Bad Request**

3. **Слабый пароль:**
```json
{
    "username": "testuser4",
    "email": "testuser4@example.com",
    "password": "123",
    "password_confirm": "123"
}
```
Ожидается: **400 Bad Request**

---

## 2. Вход пользователя

### Endpoint: POST `/api/v1/auth/login/`

**URL:** `{{base_url}}/api/v1/auth/login/`

**Headers:**
```
Content-Type: application/json
```

**Body (raw JSON):**
```json
{
    "email": "testuser@example.com",
    "password": "SecurePass123!"
}
```

**Ожидаемый ответ (200 OK):**
```json
{
    "user": {
        "id": 1,
        "username": "testuser",
        "email": "testuser@example.com",
        "first_name": "Test",
        "last_name": "User",
        "full_name": "Test User",
        "avatar": null,
        "bio": "",
        "created_at": "2025-11-28T12:00:00Z",
        "updated_at": "2025-11-28T12:00:00Z",
        "posts_count": 0,
        "comments_count": 0
    },
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc...",
    "access": "eyJ0eXAiOiJKV1QiLCJhbGc...",
    "message": "User login successfully"
}
```

**Tests:**
```javascript
if (pm.response.code === 200) {
    const response = pm.response.json();
    pm.environment.set("access_token", response.access);
    pm.environment.set("refresh_token", response.refresh);
    pm.test("Status code is 200", () => {
        pm.response.to.have.status(200);
    });
    pm.test("Login successful", () => {
        pm.expect(response.message).to.include("successfully");
    });
}
```

**Проверка ошибок:**

1. **Неверный email:**
```json
{
    "email": "nonexistent@example.com",
    "password": "SecurePass123!"
}
```
Ожидается: **400 Bad Request** с сообщением "User not found."

2. **Неверный пароль:**
```json
{
    "email": "testuser@example.com",
    "password": "WrongPassword123!"
}
```
Ожидается: **400 Bad Request** с сообщением "User not found."

---

## 3. Просмотр профиля

### Endpoint: GET `/api/v1/auth/profile/`

**URL:** `{{base_url}}/api/v1/auth/profile/`

**Headers:**
```
Authorization: Bearer {{access_token}}
```

**Ожидаемый ответ (200 OK):**
```json
{
    "id": 1,
    "username": "testuser",
    "email": "testuser@example.com",
    "first_name": "Test",
    "last_name": "User",
    "full_name": "Test User",
    "avatar": null,
    "bio": "",
    "created_at": "2025-11-28T12:00:00Z",
    "updated_at": "2025-11-28T12:00:00Z",
    "posts_count": 0,
    "comments_count": 0
}
```

**Tests:**
```javascript
pm.test("Status code is 200", () => {
    pm.response.to.have.status(200);
});

pm.test("Profile data exists", () => {
    const response = pm.response.json();
    pm.expect(response.id).to.exist;
    pm.expect(response.email).to.exist;
    pm.expect(response.username).to.exist;
});
```

**Проверка ошибок:**

1. **Без токена:**
   - Удалите заголовок `Authorization`
   - Ожидается: **401 Unauthorized**

2. **Неверный токен:**
```
Authorization: Bearer invalid_token_here
```
Ожидается: **401 Unauthorized**

---

## 4. Обновление профиля (JSON)

### Endpoint: PATCH `/api/v1/auth/profile/`

**URL:** `{{base_url}}/api/v1/auth/profile/`

**Headers:**
```
Authorization: Bearer {{access_token}}
Content-Type: application/json
```

**Body (raw JSON):**
```json
{
    "first_name": "Updated",
    "last_name": "Name",
    "bio": "This is my updated bio. I love coding!"
}
```

**Ожидаемый ответ (200 OK):**
```json
{
    "first_name": "Updated",
    "last_name": "Name",
    "avatar": null,
    "bio": "This is my updated bio. I love coding!"
}
```

**Tests:**
```javascript
pm.test("Status code is 200", () => {
    pm.response.to.have.status(200);
});

pm.test("Profile updated", () => {
    const response = pm.response.json();
    pm.expect(response.first_name).to.eql("Updated");
    pm.expect(response.bio).to.include("coding");
});
```

---

## 5. Обновление профиля с аватаром

### Endpoint: PATCH `/api/v1/auth/profile/`

**URL:** `{{base_url}}/api/v1/auth/profile/`

**Headers:**
```
Authorization: Bearer {{access_token}}
```

**Body (form-data):**
- `first_name`: `Updated`
- `last_name`: `Name`
- `bio`: `This is my updated bio with avatar`
- `avatar`: [Выберите файл изображения: File]

**Инструкция:**
1. В Postman выберите вкладку "Body"
2. Выберите "form-data"
3. Добавьте поля как "Text"
4. Для `avatar` выберите тип "File" и загрузите изображение

**Ожидаемый ответ (200 OK):**
```json
{
    "first_name": "Updated",
    "last_name": "Name",
    "avatar": "http://127.0.0.1:8000/media/avatars/image_name.jpg",
    "bio": "This is my updated bio with avatar"
}
```

---

## 6. Смена пароля

### Endpoint: PUT `/api/v1/auth/change-password/`

**URL:** `{{base_url}}/api/v1/auth/change-password/`

**Headers:**
```
Authorization: Bearer {{access_token}}
Content-Type: application/json
```

**Body (raw JSON):**
```json
{
    "old_password": "SecurePass123!",
    "new_password": "NewSecurePass456!",
    "new_password_confirm": "NewSecurePass456!"
}
```

**Ожидаемый ответ (200 OK):**
```json
{
    "message": "Password changed successfully"
}
```

**Tests:**
```javascript
pm.test("Status code is 200", () => {
    pm.response.to.have.status(200);
});

pm.test("Password changed", () => {
    const response = pm.response.json();
    pm.expect(response.message).to.include("successfully");
});
```

**Проверка ошибок:**

1. **Неверный старый пароль:**
```json
{
    "old_password": "WrongPassword!",
    "new_password": "NewSecurePass456!",
    "new_password_confirm": "NewSecurePass456!"
}
```
Ожидается: **400 Bad Request** с сообщением "Old password is incorrect."

2. **Несовпадение новых паролей:**
```json
{
    "old_password": "SecurePass123!",
    "new_password": "NewSecurePass456!",
    "new_password_confirm": "DifferentPass789!"
}
```
Ожидается: **400 Bad Request**

3. **Слабый новый пароль:**
```json
{
    "old_password": "SecurePass123!",
    "new_password": "123",
    "new_password_confirm": "123"
}
```
Ожидается: **400 Bad Request**

**⚠️ ВАЖНО:** После смены пароля старый токен остается действительным. Для входа с новым паролем используйте endpoint `/api/v1/auth/login/` с новым паролем!

---

## 7. Обновление токена

### Endpoint: POST `/api/v1/auth/token/refresh/`

**URL:** `{{base_url}}/api/v1/auth/token/refresh/`

**Headers:**
```
Content-Type: application/json
```

**Body (raw JSON):**
```json
{
    "refresh": "{{refresh_token}}"
}
```

**Ожидаемый ответ (200 OK):**
```json
{
    "access": "eyJ0eXAiOiJKV1QiLCJhbGc...",
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc..."
}
```

**Tests:**
```javascript
if (pm.response.code === 200) {
    const response = pm.response.json();
    pm.environment.set("access_token", response.access);
    
    // Обновляем refresh token, если он пришел (ROTATE_REFRESH_TOKENS=True)
    if (response.refresh) {
        pm.environment.set("refresh_token", response.refresh);
    }
    
    pm.test("Status code is 200", () => {
        pm.response.to.have.status(200);
    });
    
    pm.test("New access token received", () => {
        pm.expect(response.access).to.exist;
    });
}
```

**Когда использовать:**
- Access token истек (время жизни: 60 минут)
- Получаете ошибку 401 при запросе к защищенным endpoints

---

## 8. Выход из системы

### Endpoint: POST `/api/v1/auth/logout/`

**URL:** `{{base_url}}/api/v1/auth/logout/`

**Headers:**
```
Authorization: Bearer {{access_token}}
Content-Type: application/json
```

**Body (raw JSON):**
```json
{
    "refresh_token": "{{refresh_token}}"
}
```

**Ожидаемый ответ (200 OK):**
```json
{
    "message": "Logout successful"
}
```

**Tests:**
```javascript
pm.test("Status code is 200", () => {
    pm.response.to.have.status(200);
});

pm.test("Logout successful", () => {
    const response = pm.response.json();
    pm.expect(response.message).to.include("successful");
});

// Очистка токенов после logout
pm.environment.unset("access_token");
pm.environment.unset("refresh_token");
```

**⚠️ ВАЖНО:** Для работы этого endpoint нужно:

1. Добавить в `INSTALLED_APPS` в `config/settings.py`:
```python
INSTALLED_APPS = [
    ...
    'rest_framework_simplejwt.token_blacklist',
]
```

2. Выполнить миграции:
```bash
python manage.py migrate
```

**Проверка после logout:**
После успешного logout попробуйте использовать старый refresh token для обновления - должна быть ошибка.

---

## Полные сценарии тестирования

### Сценарий 1: Полный жизненный цикл пользователя

1. **Регистрация нового пользователя**
   - POST `{{base_url}}/api/v1/auth/register/`
   - Сохраните токены

2. **Просмотр профиля**
   - GET `{{base_url}}/api/v1/auth/profile/`
   - Проверьте, что данные соответствуют регистрации

3. **Обновление профиля**
   - PATCH `{{base_url}}/api/v1/auth/profile/`
   - Измените `bio` и `first_name`

4. **Проверка изменений**
   - GET `{{base_url}}/api/v1/auth/profile/`
   - Убедитесь, что изменения сохранены

5. **Смена пароля**
   - PUT `{{base_url}}/api/v1/auth/change-password/`
   - Смените пароль на новый

6. **Выход из системы**
   - POST `{{base_url}}/api/v1/auth/logout/`

7. **Вход с новым паролем**
   - POST `{{base_url}}/api/v1/auth/login/`
   - Используйте новый пароль

---

### Сценарий 2: Проверка безопасности

1. **Попытка доступа без авторизации**
   - GET `{{base_url}}/api/v1/auth/profile/` (без токена)
   - Ожидается: 401 Unauthorized

2. **Попытка доступа с неверным токеном**
   - GET `{{base_url}}/api/v1/auth/profile/`
   - Authorization: `Bearer fake_token`
   - Ожидается: 401 Unauthorized

3. **Обновление токена**
   - POST `{{base_url}}/api/v1/auth/token/refresh/`
   - Получите новый access token

4. **Использование нового токена**
   - GET `{{base_url}}/api/v1/auth/profile/`
   - С новым access token
   - Ожидается: 200 OK

5. **Logout и попытка использовать refresh token**
   - POST `{{base_url}}/api/v1/auth/logout/`
   - Попробуйте обновить токен с помощью старого refresh token
   - Ожидается: ошибка (если token blacklist настроен)

---

### Сценарий 3: Проверка валидации

1. **Слабый пароль при регистрации**
```json
{
    "username": "weakuser",
    "email": "weak@example.com",
    "password": "123",
    "password_confirm": "123"
}
```
Ожидается: 400 Bad Request

2. **Дублирование email**
   - Попробуйте зарегистрировать пользователя с существующим email
   - Ожидается: 400 Bad Request

3. **Несовпадение паролей**
```json
{
    "username": "mismatchuser",
    "email": "mismatch@example.com",
    "password": "Password123!",
    "password_confirm": "Different456!"
}
```
Ожидается: 400 Bad Request

4. **Неверные данные при входе**
```json
{
    "email": "nonexistent@example.com",
    "password": "anypassword"
}
```
Ожидается: 400 Bad Request

5. **Неверный старый пароль при смене**
```json
{
    "old_password": "WrongPassword!",
    "new_password": "NewPassword123!",
    "new_password_confirm": "NewPassword123!"
}
```
Ожидается: 400 Bad Request

---

## Автоматизация с Collection Runner

### Создание последовательности

1. В Postman откройте вашу коллекцию
2. Нажмите "Run" (или иконку Runner)
3. Выберите следующие запросы в порядке:

**Последовательность запросов:**
1. Register User
2. Get Profile
3. Update Profile
4. Get Profile (проверка изменений)
5. Change Password
6. Logout
7. Login (с новым паролем)
8. Refresh Token
9. Get Profile (с новым токеном)
10. Logout

### Настройка данных для Runner

Создайте CSV файл `test_data.csv`:
```csv
username,email,password,first_name,last_name
user1,user1@test.com,SecurePass123!,John,Doe
user2,user2@test.com,SecurePass456!,Jane,Smith
user3,user3@test.com,SecurePass789!,Bob,Johnson
```

В Runner выберите этот файл для итераций.

---

## Pre-request Scripts для динамических данных

### Генерация случайных пользователей

Добавьте в Pre-request Script для Register:

```javascript
// Генерация случайного email и username
const timestamp = Date.now();
const random = Math.floor(Math.random() * 10000);

pm.environment.set("random_email", `user${timestamp}${random}@example.com`);
pm.environment.set("random_username", `user${timestamp}${random}`);
pm.environment.set("random_password", "SecurePass123!");
```

Используйте в Body:
```json
{
    "username": "{{random_username}}",
    "email": "{{random_email}}",
    "password": "{{random_password}}",
    "password_confirm": "{{random_password}}",
    "first_name": "Test",
    "last_name": "User"
}
```

---

## Общие Tests для всех запросов

Добавьте в Collection уровень (Tests):

```javascript
// Проверка времени ответа
pm.test("Response time is less than 1000ms", () => {
    pm.expect(pm.response.responseTime).to.be.below(1000);
});

// Проверка заголовка Content-Type
pm.test("Content-Type is application/json", () => {
    pm.expect(pm.response.headers.get("Content-Type")).to.include("application/json");
});

// Логирование для отладки
console.log("Request URL:", pm.request.url.toString());
console.log("Response Status:", pm.response.code);
```

---

## Troubleshooting

### Ошибка 401 "Authentication credentials were not provided"
**Причина:** Отсутствует или неверный токен
**Решение:** Проверьте заголовок `Authorization: Bearer {{access_token}}`

### Ошибка 401 "Token is invalid or expired"
**Причина:** Access token истек
**Решение:** Используйте `/api/v1/auth/token/refresh/` для получения нового

### Ошибка 400 "User not found"
**Причина:** Неверный email или пароль при входе
**Решение:** Проверьте правильность данных

### Ошибка 500 Internal Server Error
**Причина:** Ошибка на сервере (возможно, не применены миграции)
**Решение:** 
```bash
python manage.py migrate
python manage.py runserver
```

### Avatar не загружается
**Причина:** Неверный тип запроса
**Решение:** Используйте `form-data` вместо `raw JSON` для загрузки файлов

---

## Полезные команды Django

```bash
# Создание суперпользователя
python manage.py createsuperuser

# Просмотр всех пользователей в shell
python manage.py shell
>>> from apps.accounts.models import User
>>> User.objects.all()

# Удаление всех пользователей (для тестирования)
>>> User.objects.all().delete()

# Сброс базы данных
python manage.py flush

# Применение миграций
python manage.py migrate

# Создание новых миграций
python manage.py makemigrations
```

---

## Экспорт и импорт коллекции Postman

### Экспорт:
1. Правый клик на коллекции → Export
2. Выберите Collection v2.1
3. Сохраните JSON файл

### Импорт:
1. Import → Upload Files
2. Выберите JSON файл коллекции
3. Импортируйте Environment отдельно

---

## Чеклист перед началом тестирования

- [ ] Сервер Django запущен (`python manage.py runserver`)
- [ ] База данных PostgreSQL запущена и настроена
- [ ] Применены все миграции (`python manage.py migrate`)
- [ ] Установлены переменные окружения в Postman
- [ ] Создана коллекция с правильными endpoints
- [ ] Настроены Tests для автоматического сохранения токенов

---

Удачи в тестировании вашего API! 🚀

**Полезные ресурсы:**
- [Postman Documentation](https://learning.postman.com/)
- [Django REST Framework](https://www.django-rest-framework.org/)
- [Simple JWT Documentation](https://django-rest-framework-simplejwt.readthedocs.io/)