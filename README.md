# Task Manager

Многосервисный backend на Go: управление задачами, авторизация и обработка событий через RabbitMQ.

## Сервисы

- [Task API](https://github.com/mxmrnn/api-task-manager) — создание, чтение, обновление и удаление задач, фильтрация и статистика.
- [Auth](https://github.com/mxmrnn/auth-todo-service) — регистрация, вход, JWT и обновление токенов.
- [Email](https://github.com/mxmrnn/email-todo-service) — обработка событий регистрации. Отправка писем имитируется в логах.
- [Infrastructure](https://github.com/mxmrnn/infra-todo) — Docker Compose для совместного запуска сервисов, баз данных и брокера.

## Стек и взаимодействие

Go, Gin, GORM, pgx, PostgreSQL, RabbitMQ, Docker Compose.

Task API и Auth используют отдельные базы PostgreSQL и обмениваются запросами через RabbitMQ RPC. После регистрации Auth публикует событие `user_registered`, которое обрабатывает Email.

## Запуск

Нужны Git и Docker с Linux-контейнерами и Docker Compose.

Клонировать сервисы в указанные каталоги — их имена используются в Compose:

```sh
git clone https://github.com/mxmrnn/infra-todo.git task-manager
cd task-manager
git clone https://github.com/mxmrnn/api-task-manager.git task-service
git clone https://github.com/mxmrnn/auth-todo-service.git auth-service
git clone https://github.com/mxmrnn/email-todo-service.git email-service

docker compose up --build -d
docker compose ps
```

| Компонент | Адрес |
| --- | --- |
| Task API | http://localhost:8080 |
| Auth API | http://localhost:8081 |
| RabbitMQ Management | http://localhost:15672 |

Миграции баз данных выполняются при запуске. Перед регистрацией пользователя дождаться сообщения `Email consumer started, listening to email.notifications` в логах Email:
```
docker compose logs --tail=100 task-app auth-app email-service
```

Для проверки взаимодействия зарегистрировать пользователя через `POST /api/v1/auth/register`, выполнить вход через `POST /api/v1/auth/login` и создать задачу через `POST /tasks/`. Чтение задачи запрашивает сведения об авторе из Auth; получение профиля через `GET /api/v1/user/info/` запрашивает статистику задач из Task API.

Остановка с сохранением данных:
```sh
docker compose down
```

Task API пока не проверяет JWT и права доступа к задачам.
