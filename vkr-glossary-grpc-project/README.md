# Глоссарий терминов ВКР на основе gRPC и Protobuf

Проект представляет собой микросервисную архитектуру для управления глоссарием терминов выпускной квалификационной работы, реализованную с использованием gRPC и Protocol Buffers.

## Содержание

- [Описание проекта](#описание-проекта)
- [Структура проекта](#структура-проекта)
- [Быстрый старт](#быстрый-старт)
- [Архитектура](#архитектура)
- [Документация](#документация)

## Описание проекта

Проект состоит из двух основных компонентов:

1. **Backend (glossary-grpc/)** - gRPC микросервисы:
   - `glossary-service` - gRPC сервер для работы с терминами
   - `web-service` - HTTP прокси, преобразующий REST запросы в gRPC вызовы

2. **Frontend (mindmap-vkr/)** - Vue.js приложение:
   - Интерактивный интерфейс для управления терминами
   - Визуализация связей между терминами (MindMap)

### Преимущества использования gRPC

- Высокая производительность за счет бинарного формата Protocol Buffers
- Типобезопасность через строгое определение контрактов в `.proto` файлах
- Межъязыковая поддержка для интеграции сервисов на разных языках
- HTTP/2 с поддержкой мультиплексирования и стриминга

## Структура проекта

```
vkr-glossary-grpc-project/
├── glossary-grpc/              # Backend (gRPC)
│   ├── glossary-service/        # gRPC сервер
│   │   ├── protobufs/
│   │   │   └── glossary.proto   # Определение gRPC сервиса
│   │   ├── data/
│   │   │   └── terms.json       # База данных терминов
│   │   ├── glossary.py          # Реализация gRPC сервера
│   │   ├── Dockerfile
│   │   └── requirements.txt
│   │
│   ├── web-service/             # HTTP → gRPC прокси
│   │   ├── protobufs/
│   │   │   └── glossary.proto
│   │   ├── web.py               # FastAPI приложение
│   │   ├── Dockerfile
│   │   └── requirements.txt
│   │
│   ├── docker-compose.yml       # Оркестрация сервисов
│   └── REPORT.md                # Подробная документация
│
└── mindmap-vkr/                 # Frontend (Vue.js)
    ├── src/
    │   ├── components/          # Vue компоненты
    │   ├── services/
    │   │   └── api.js           # API клиент
    │   └── ...
    ├── package.json
    └── vite.config.js
```

## Быстрый старт

### Предварительные требования

- Docker версии 20.10+
- Docker Compose версии 1.29+
- Node.js версии 16.0+ (для фронтенда)

### 1. Запуск Backend (gRPC сервисы)

```bash
cd glossary-grpc
docker-compose up -d
```

Backend будет доступен на:
- Web Service API: http://localhost:8000
- API Документация: http://localhost:8000/docs
- Health Check: http://localhost:8000/api/health

### 2. Запуск Frontend

```bash
cd mindmap-vkr
npm install
npm run dev
```

Frontend будет доступен на: http://localhost:5173

### 3. Проверка работы

1. Откройте http://localhost:5173 в браузере
2. Проверьте, что термины загружаются
3. Попробуйте создать или отредактировать термин

## Архитектура

```
┌─────────────┐
│  Frontend   │ (Vue.js)
│  (Browser)  │
└──────┬──────┘
       │ HTTP/REST
       │
┌──────▼──────────────────┐
│   Web Service           │ (FastAPI прокси)
│   Port: 8000           │
│   - Принимает HTTP     │
│   - Преобразует в gRPC │
└──────┬──────────────────┘
       │ gRPC (HTTP/2)
       │
┌──────▼──────────────────┐
│  Glossary Service       │ (gRPC Server)
│  Port: 50052            │
│   - Обрабатывает запросы│
│   - Работает с данными  │
└──────┬──────────────────┘
       │
┌──────▼──────┐
│  JSON File  │ (data/terms.json)
│  Database   │
└─────────────┘
```

## Документация

### Подробная документация

Полная документация с инструкциями по развертыванию, примерами использования и описанием архитектуры находится в файле:

[glossary-grpc/REPORT.md](glossary-grpc/REPORT.md)

### API Endpoints

Web Service предоставляет следующие HTTP endpoints:

| Метод | URL | Описание |
|-------|-----|----------|
| GET | `/api/terms` | Получить список терминов с пагинацией |
| GET | `/api/terms/{id}` | Получить конкретный термин |
| POST | `/api/terms` | Создать новый термин |
| PUT | `/api/terms/{id}` | Обновить термин |
| DELETE | `/api/terms/{id}` | Удалить термин |
| GET | `/api/terms/search/{query}` | Поиск терминов |
| GET | `/api/health` | Проверка состояния API |

### gRPC Методы

Glossary Service предоставляет следующие gRPC методы:

- `GetTerm(GetTermRequest) -> Term`
- `GetTerms(GetTermsRequest) -> GetTermsResponse`
- `CreateTerm(CreateTermRequest) -> Term`
- `UpdateTerm(UpdateTermRequest) -> Term`
- `DeleteTerm(DeleteTermRequest) -> DeleteTermResponse`
- `SearchTerms(SearchTermsRequest) -> SearchTermsResponse`
- `HealthCheck(HealthCheckRequest) -> HealthCheckResponse`

## Технологический стек

### Backend

- Python 3.12+
- gRPC
- Protocol Buffers
- FastAPI
- Uvicorn

### Frontend

- Vue.js 3
- Vue Router 4
- D3.js
- Vite

### Инфраструктура

- Docker
- Docker Compose

## Примеры использования

### Проверка Health Check

```bash
curl http://localhost:8000/api/health
```

### Получение списка терминов

```bash
curl "http://localhost:8000/api/terms?page=1&per_page=10"
```

### Создание нового термина

```bash
curl -X POST http://localhost:8000/api/terms \
  -H "Content-Type: application/json" \
  -d '{
    "term": "gRPC",
    "definition": "Высокопроизводительный фреймворк для удаленных вызовов процедур",
    "category": "Технологии",
    "related_terms": ["Protocol Buffers", "HTTP/2"]
  }'
```

## Разработка

### Остановка сервисов

```bash
cd glossary-grpc
docker-compose down
```

### Просмотр логов

```bash
docker-compose logs -f
```

### Пересборка после изменений

```bash
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

