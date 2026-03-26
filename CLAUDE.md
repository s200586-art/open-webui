# Open WebUI (Fork) — NPLN Tech / Идеал-пол

Форк [open-webui/open-webui](https://github.com/open-webui/open-webui) для деплоя на VPS (Tyumen server).

**Цель**: корпоративный AI-интерфейс для бизнеса (Идеал-пол / NPLN Tech).

**Основная задача**: подключить локальную LLM — OpenClaw — через OpenAI-совместимый API.
OpenClaw уже работает на этом же VPS и предоставляет OpenAI-совместимый эндпоинт,
поэтому подключается как кастомный OpenAI endpoint через `OPENAI_API_BASE_URL` и `OPENAI_API_KEY`.

## Инфраструктура

<!-- TODO: заполнить после деплоя -->
<!-- Адрес VPS, порты, домен, nginx конфиг -->

## OpenClaw интеграция

- OpenClaw запущен локально на VPS (тот же сервер, что и Open WebUI)
- Эндпоинт: `http://localhost:ПОРТ/v1` (порт уточним позже)
- Подключается через настройки OpenAI API в Open WebUI (Admin → Settings → Connections → OpenAI API)
- Задача: чтобы сотрудники компании работали с OpenClaw через удобный веб-интерфейс

## Что сделано

- [ ] Деплой Open WebUI в Docker на VPS
- [ ] Подключение OpenClaw как модели (OpenAI-совместимый API)
- [ ] Настройка пользователей и ролей
- [ ] SSL + nginx (reverse proxy, домен)
- [ ] RAG с документами компании

## ENV-переменные

Основные переменные окружения (значения в `.env.local`, не коммитить!):

```
OPENAI_API_KEY=         # API-ключ OpenClaw
OPENAI_API_BASE_URL=    # http://localhost:ПОРТ/v1 (OpenClaw endpoint)
OLLAMA_BASE_URL=        # http://localhost:11434 (если нужен Ollama)
```

## Решения и заметки

<!-- История решений и заметок по проекту -->
