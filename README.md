# Сервер для управления клиентами (тестовое задание Форте 21)

Этот сервер предоставляет функционал для управления клиентами, включая создание, обновление, удаление и получение данных. Он разработан для демонстрации возможностей работы с REST API, аутентификацией и взаимодействием с базой данных MongoDB.

## Основные возможности сервера

1. **Аутентификация**
   - Поддержка JWT-токенов для обеспечения безопасности.
   - Генерация токенов с ограниченным сроком действия (15 минут).

2. **Работа с клиентами**
   - Получение списка клиентов с поддержкой фильтрации, сортировки и пагинации.
   - Получение данных конкретного клиента по ID.
   - Создание, обновление и удаление клиентов с использованием защищённых маршрутов.

3. **Гибкая настройка**
   - Возможность указания переменных окружения для конфигурации базы данных, аутентификации и других параметров.

4. **Разработка и тестирование**
   - Поддержка разработки в режиме live-reload.
   - Включены тестовые скрипты для обеспечения качества кода и функциональности.

---

## Требования к системе для запуска

1. **Node.js**: Версия **16.x** или выше.
2. **npm**: Версия **7.x** или выше.
3. **MongoDB**: Версия **5.x** или выше (локально или удалённый экземпляр).
4. **Переменные окружения**:
   - `USER_NAME`: Имя пользователя для авторизации.
   - `MONGODB_URI`: Ссылка для подключения к базе данных MongoDB.
   - `JWT_KEY`: Секретный ключ для генерации JWT.
   - `HASHED_PASSWORD`: Захэшированный пароль (bcrypt). Пароль можно захешировать, используя скрипт `password-hasher.js`.
   - `NODE_ENV` (не обязательно): при установке в `develop` отключает **CORS-политики**.

---

## Установка и запуск

1. Установите зависимости:
   ```bash
   npm install
   ```

2. Запустите приложение в нужном режиме, используя один из скриптов, указанных ниже.

---

## Скрипты проекта

```json
"scripts": {
    "build": "nest build",
    "format": "prettier --write \"src/**/*.ts\" \"src/tests/**/*.ts\"",
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "node dist/main",
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand"
}
```

### Основные скрипты

1. **`build`**  
   Команда для сборки приложения в продакшн-режиме. Скомпилированные файлы будут помещены в папку `dist`.
   ```bash
   npm run build
   ```

2. **`start`**  
   Запуск приложения в собранном виде (продакшн-режим). Требуется предварительная сборка с помощью команды `build`.
   ```bash
   npm run start
   ```

3. **`start:dev`**  
   Запуск приложения в режиме разработки с отслеживанием изменений.
   ```bash
   npm run start:dev
   ```

4. **`start:prod`**  
   Запуск приложения в продакшн-режиме. Использует скомпилированные файлы из папки `dist`.
   ```bash
   npm run start:prod
   ```

5. **`test`**  
   Запуск тестов.
   ```bash
   npm run test
   ```

6. **`lint`**  
   Проверка кода на ошибки и автоматическое исправление.
   ```bash
   npm run lint
   ```

---

## API сервера

### Аутентификация

#### **Маршрут: POST `/auth`**

Используется для получения JWT-токена.

**Тело запроса (DTO)**:

```json
{
  "username": "string",
  "password": "string"
}
```

**Пример успешного ответа**:

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Требования**:

- Доступен без авторизации.
- После получения `accessToken` его необходимо передавать в заголовке `Authorization` в формате `Bearer <token>` для всех маршрутов, требующих авторизации.
- `accessToken` действует 15 минут (JWT-blacklist в планах).

---

### Работа с клиентами

#### **Маршруты без токена**

1. **GET `/clients`**  
   Возвращает список всех клиентов. Поддерживает фильтрацию, сортировку и пагинацию.

   **Параметры запроса**:
   - `search` — строка для поиска по имени и компании.
   - `sortField` — поле для сортировки (`name`, `company`).
   - `sortOrder` — порядок сортировки (`asc`, `desc`).
   - `page` — номер страницы (по умолчанию: 1).
   - `limit` — количество записей на странице (по умолчанию: 10).

   **Пример ответа**:

   ```json
   {
     "data": [
       {
         "_id": "64b5d6e2f1b4",
         "name": "John Doe",
         "company": "Doe Inc."
       }
     ],
     "total": 42
   }
   ```

2. **GET `/clients/:id`**  
   Возвращает клиента по `id`.

3. **GET `/clients/total`**  
   Возвращает общее количество клиентов.

#### **Маршруты, требующие токен**

1. **POST `/clients`**  
   Создаёт нового клиента.

2. **PUT `/clients/:id`**  
   Обновляет данные клиента по `id`.

3. **DELETE `/clients/:id`**  
   Удаляет клиента по `id`.

---

## DTO

### **UserDto (используется в аутентификации)**

```typescript
export class UserDto {
  @IsString()
  @IsNotEmpty()
  username: string;

  @IsString()
  @IsNotEmpty()
  password: string;
}
```

### **ClientDto (основной DTO для клиента)**

```typescript
export class ClientDto {
  @IsString()
  @IsNotEmpty()
  name: string;

  @IsString()
  @IsNotEmpty()
  company: string;

  details: ClientDetailsDto;
}
```

### **ClientDetailsDto (подробности клиента)**

```typescript
export class ClientDetailsDto {
  @IsString()
  @IsNotEmpty()
  contact: string;

  @IsString()
  @IsOptional()
  about?: string;

  @IsString()
  @IsOptional()
  phoneNumber?: string;
}
```

### **ClientListDto (список клиентов)**

```typescript
import { IsArray, IsNumber } from 'class-validator';
import { ClientDto } from './client.dto';

export class ClientListDto {
  @IsArray()
  data: ClientDto[];

  @IsNumber()
  total: number;
}

```

