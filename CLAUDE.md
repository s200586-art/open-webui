# Open WebUI (Fork) — NPLN Tech / Идеал-пол

Форк [open-webui/open-webui](https://github.com/open-webui/open-webui) для деплоя на VPS (Tyumen server).

**Цель**: корпоративный AI-интерфейс для бизнеса (Идеал-пол / NPLN Tech).

**Основная задача**: подключить локальную LLM — OpenClaw — через OpenAI-совместимый API.
OpenClaw уже работает на этом же VPS и предоставляет OpenAI-совместимый эндпоинт,
поэтому подключается как кастомный OpenAI endpoint через `OPENAI_API_BASE_URL` и `OPENAI_API_KEY`.

## Инфраструктура

- **VPS**: 89.104.65.27 (Ubuntu, Tyumen server)
- **Open WebUI**: Docker контейнер `open-webui`, `--network host`, порт 3000
- **Данные**: `/root/open-webui-data` (volume)
- **Домен**: `ai.npln.tech` (нужна A-запись → 89.104.65.27)
- **Nginx**: `/etc/nginx/sites-enabled/open-webui` → proxy_pass 127.0.0.1:3000
- **SSL**: certbot (после настройки DNS: `certbot --nginx -d ai.npln.tech`)
- **Прямой доступ**: `http://89.104.65.27:3000`

### Ollama (MacBook → VPS)

- **Ollama** установлен на MacBook (24 GB RAM, Apple Silicon)
- **Модель**: `llama3.1:8b` (4.9 GB)
- **Локально**: `http://127.0.0.1:11434` (Mac)
- **SSH reverse tunnel**: Mac:11434 → VPS:11434
  - launchd: `com.ollama.ssh-tunnel` (`~/Library/LaunchAgents/com.ollama.ssh-tunnel.plist`)
  - Автозапуск, KeepAlive, ServerAliveInterval=60
- **На VPS**: Open WebUI видит Ollama по `http://127.0.0.1:11434`

## OpenClaw интеграция

- OpenClaw запущен локально на VPS (тот же сервер, что и Open WebUI)
- Эндпоинт: `http://127.0.0.1:18789/v1` (порт 18789, loopback only)
- Подключается через настройки OpenAI API в Open WebUI (Admin → Settings → Connections → OpenAI API)
- Задача: чтобы сотрудники компании работали с OpenClaw через удобный веб-интерфейс

### API-ключ

**Внешние ключи не нужны.** OpenClaw сам генерирует API-ключ — это просто «пароль»,
по которому Open WebUI авторизуется в OpenClaw. Формат: `sk-napoli-vps-2026` (пример).

Схема подключения:
1. OpenClaw генерирует ключ в своих настройках
2. Ключ прописывается в Open WebUI как `OPENAI_API_KEY`
3. Open WebUI обращается к OpenClaw по `OPENAI_API_BASE_URL` с этим ключом
4. Пользователи заходят в Open WebUI по ссылке (логин/пароль) — и видят OpenClaw в списке моделей

## Что сделано

- [x] Деплой Open WebUI в Docker на VPS (порт 3000, --network host)
- [x] Ollama на MacBook + SSH reverse tunnel на VPS (llama3.1:8b)
- [ ] Подключение OpenClaw как модели (ждём API-ключ от OpenClaw)
- [ ] Настройка пользователей и ролей
- [ ] SSL + nginx (DNS A-запись `ai.npln.tech`, затем certbot)
- [ ] RAG с документами компании

## ENV-переменные

Основные переменные окружения (значения в `.env.local`, не коммитить!):

```
OPENAI_API_KEY=         # API-ключ OpenClaw (генерируется самим OpenClaw)
OPENAI_API_BASE_URL=http://127.0.0.1:18789/v1  # OpenClaw endpoint
OLLAMA_BASE_URL=http://127.0.0.1:11434  # Ollama через SSH tunnel с MacBook
```

## Решения и заметки

- **API-ключ**: OpenClaw генерирует сам, покупать/регистрировать ничего не нужно. Это внутренний пароль между Open WebUI и OpenClaw на одном сервере.
- **Docker**: `--network host` вместо bridge — потому что OpenClaw слушает только на 127.0.0.1:18789 (loopback), `host.docker.internal` не достаёт.
- **DNS**: нужна A-запись `ai.npln.tech → 89.104.65.27`, после этого `certbot --nginx -d ai.npln.tech` для SSL.
- **Ollama tunnel**: reverse SSH (`-R 11434:127.0.0.1:11434`) — Mac'овский Ollama виден на VPS как localhost:11434. Launchd с KeepAlive для автопереподключения.
