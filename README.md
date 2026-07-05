# Корпоративное приложение - Конструкторское бюро

⚠️ **Внимание: NDA (Соглашение о неразглашении)**

Данный репозиторий является портфолио-визиткой. Исходный код проекта является закрытым в связи с коммерческой тайной работодателя. В этом описании представлена архитектура проекта, используемый стек технологий, а также скриншоты пользовательского интерфейса (с использованием тестовых и обезличенных данных) исключительно для демонстрации моих компетенций как инженера-разработчика.

---

## Стек

![Go](https://img.shields.io/badge/Go-1.25-00ADD8?logo=go) ![SQLite](https://img.shields.io/badge/SQLite-003B57?logo=sqlite) ![JavaScript](https://img.shields.io/badge/JavaScript-ES6-F7DF1E?logo=javascript) ![WebSocket](https://img.shields.io/badge/WebSocket-✔-4A90D9)

- **Бэкенд:** Go 1.25, `net/http`, `gorilla/websocket`
- **База данных:** SQLite (основная), PostgreSQL (статистика)
- **Фронтенд:** Vanilla JS (ES6), Chart.js — SPA без фреймворков
- **Шифрование:** AES-256-GCM + PBKDF2 (хранилище паролей), bcrypt (пароли)
- **Реалтайм:** WebSocket (уведомления, чат)
- **Экспорт:** Excel (excelize), DOCX (шаблоны)

## Возможности

| Модуль | Описание |
|--------|----------|
| **Задачи** | Kanban-доска, подзадачи, приоритеты, дедлайны, исполнители, комментарии, история изменений, вложения |
| **Мессенджер** | Групповые чаты, GIF, пересылка сообщений, ответы, вложения, прочитано |
| **Документы (Акты)** | Шаблоны DOCX, автоматический парсинг, генерация XLSX-отчётов, watcher файловой системы |
| **Расходы** | Учёт по категориям, датам, экспорт в Excel |
| **Хранилище паролей** | Шифрованное AES-256-GCM, привязка к паролю пользователя |
| **Сотрудники** | Справочник с отделами, должностями, контактами |
| **Учёт времени** | Таймер задач, отчёты в JSON/HTML, история |
| **Отчёты эффективности** | Шаблоны KPI, импорт из Excel |
| **Резервное копирование** | VACUUM INTO + SHA256, авто-бэкап при старте |
| **Уведомления** | In-app (WebSocket) + всплывающие Toast-уведомления (SPA) |
| **Telegram** | Интеграция с ботом |
| **Статистика** | PostgreSQL (с фолбэком на SQLite) |

## Быстрый старт

```bat
build.bat            # go build -o build/bin/corp-app.exe .
start.bat            # запуск сервера на 127.0.0.1:8765
```

Откройте `http://localhost:8765` в браузере.

### Переменные окружения

| Переменная | По умолчанию | Назначение |
|------------|-------------|-----------|
| `LISTEN_ADDR` | `127.0.0.1:8765` | Адрес HTTP-сервера |
| `SUPERADMIN_PASSWORD` | — | Пароль суперадминистратора |
| `ALLOWED_ORIGIN` | auto | CORS origin |
| `CORP_APP_AUTO_BACKUP` | `0` | Авто-бэкап при старте |

## Разработка

```bash
go test ./internal/server/...
```

Тесты используют in-memory SQLite.

## Структура проекта

```text
corp-app/
├── main.go                     # Точка входа, //go:embed all:frontend
├── internal/server/            # Весь бэкенд (Go)
│   ├── handlers*.go            # REST-хендлеры по модулям
│   ├── mux.go                  # Маршрутизация
│   ├── middleware.go           # Сессионная аутентификация, CORS
│   ├── database.go             # SQLite миграции
│   ├── websocket.go            # WebSocket hub
│   ├── vault_crypto.go         # AES-256-GCM шифрование
│   ├── backup.go               # Резервное копирование
│   ├── ratelimit.go            # Rate limiter
│   ├── acts_watcher.go         # Watcher DOCX
│   ├── models.go               # Структуры данных
│   ├── bootstrap.go            # Инициализация
│   └── lan.go                  # HTTP-сервер
├── frontend/
│   ├── index.html              # SPA
│   ├── css/style.css           # Тёмная тема
│   └── js/
│       ├── api.js              # HTTP-клиент
│       ├── app-core.js         # Навигация, layout
│       ├── app-tasks.js        # Задачи, чат, GIF
│       ├── app-modules.js      # Расходы, vault, админка
│       ├── app-timetracking.js # Трекер времени
│       └── app-profile-sync.js # Профиль, акты, синхронизация
├── acts/                       # DOCX-файлы актов (watcher)
└── templates/                  # Шаблоны документов
```

## API (REST, /api)

Авторизация: Opaque Bearer-токен в заголовке `Authorization` (сессии хранятся в памяти сервера).

| Раздел | Эндпоинты |
|--------|-----------|
| Auth | `POST /api/login`, `POST /api/register` |
| Users | `GET/POST/PUT/DELETE /api/users` |
| Profile | `GET/PUT /api/profile`, `PUT /api/profile/password` |
| Admin | `GET/PUT /api/admin/*` |
| Tasks | `GET/POST /api/tasks`, `GET/PUT/DELETE /api/tasks/:id`, подзадачи, комментарии |
| Chats | `GET/POST /api/chats`, сообщения `GET/POST /api/chats/:id/messages` |
| Messages | `PUT/DELETE` сообщения, forward |
| Expenses | `GET/POST /api/expenses`, категории |
| Acts | `GET/POST /api/acts`, `GET /api/acts/download/:id` |
| Vault | `GET/POST/PUT/DELETE /api/vault` |
| Departments | `GET/POST /api/departments` |
| Employees | `GET/POST /api/employees` |
| Time Tracking | `GET/POST /api/timetracking` |
| Statistics | `GET /api/statistics` |
| Efficiency | `GET/POST /api/efficiency` |
| Files | `POST /api/upload`, `GET /api/files/:id` |
| Backup | `POST /api/backup`, `GET /api/backup`, `POST /api/restore` |
| Notifications | `GET /api/notifications`, `POST /api/notifications/:id/read` |
| Export/Import | `GET /api/export/*`, `POST /api/import/*` |
| WebSocket | `/api/ws`, `/api/ws2` (аутентификация первым сообщением) |
| GIF | `GET /api/gifs?q=:query` (прокси Giphy) |
| Telegram | `GET/PUT /api/tg-config`, `POST /api/tg-test` |
| Network | `GET /api/network` (LAN-адреса) |

## Пользовательский интерфейс
<img width="1919" height="904" alt="image" src="https://github.com/user-attachments/assets/a44b9dc7-d06a-46ee-bacb-1eb49fc72379" />
<img width="1919" height="909" alt="image" src="https://github.com/user-attachments/assets/2dd6c7b4-a7ca-4aed-9ae0-a0d47cd55405" />


