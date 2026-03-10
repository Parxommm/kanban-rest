# Деплой Kanban REST API на Render.com

## Вариант 1: Через Blueprint (render.yaml)

Если ваш репозиторий уже подключён к Render:

1. Зайдите в [dashboard.render.com](https://dashboard.render.com).
2. **New** → **Blueprint**.
3. Подключите репозиторий с этим проектом (если ещё не подключён).
4. Render подхватит `render.yaml` из корня и создаст:
   - **PostgreSQL** (база `kanban-db`, план `free`);
   - **Web Service** (API), который получит `DATABASE_URL` и сгенерированный `JWT_SECRET_KEY`.
5. После первого деплоя сохраните **JWT_SECRET_KEY** из Environment — он понадобится фронтенду для авторизации.

Сервис будет доступен по адресу вида: `https://kanban-api.onrender.com`.

---

## Вариант 2: Ручная настройка (без Blueprint)

### 1. База данных PostgreSQL

1. **New** → **PostgreSQL**.
2. Имя: например `kanban-db`.
3. Регион: тот же, что будет у Web Service (например Oregon).
4. Plan: **Free** (или другой по необходимости).
5. Создайте базу и скопируйте **Internal Database URL** (нужен для Web Service).

### 2. Web Service (API)

1. **New** → **Web Service**.
2. Подключите репозиторий и выберите этот проект.
3. Настройки:
   - **Name:** `kanban-api` (или любое).
   - **Region:** тот же, что у БД.
   - **Branch:** `main` (или ваша основная ветка).
   - **Runtime:** Node.
   - **Build Command:**
     ```bash
     npm install && npm run build
     ```
   - **Start Command:**
     ```bash
     npm start
     ```
   - **Pre-Deploy Command** (миграции перед каждым деплоем):
     ```bash
     npm run typeorm:migration
     ```

### 3. Переменные окружения (Environment)

В разделе **Environment** добавьте:

| Key              | Value / источник |
|------------------|-------------------|
| `PORT`           | Оставьте пустым — Render подставит сам, или укажите `10000` |
| `DATABASE_URL`   | **Internal Database URL** из созданной PostgreSQL (из шага 1) |
| `JWT_SECRET_KEY`| Секрет для JWT (например сгенерируйте: `openssl rand -base64 32`) |
| `USE_FASTIFY`    | `true` (опционально) |
| `LOG_CONSOLE`    | `true` (опционально) |

Сохраните и запустите деплой.

---

## Проверка API

После успешного деплоя:

- Основной URL: `https://<your-service-name>.onrender.com`
- Swagger: `https://<your-service-name>.onrender.com/docs`
- Пример здоровья: `GET https://<your-service-name>.onrender.com/` (если есть корневой маршрут)

На бесплатном плане сервис может «засыпать» после неактивности; первый запрос после этого может идти дольше (cold start).

---

## Важно

1. **Node.js:** В `package.json` указано `"node": ">=16.0.0 <17"`. На Render по умолчанию часто используется Node 18+. При ошибках сборки задайте в Environment переменную `NODE_VERSION` (например `18` или `20`), если Render это поддерживает в вашем регионе.
2. **JWT_SECRET_KEY:** Должен совпадать с тем, что использует фронтенд для проверки токенов. После генерации через Blueprint скопируйте его в настройки фронта.
3. **Миграции:** Выполняются при каждом деплое через `preDeployCommand`. Для первого деплоя база должна быть уже создана (PostgreSQL из шага 1 или из Blueprint).
