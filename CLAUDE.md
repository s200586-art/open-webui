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
- **Домен**: `ai.npln.tech` (A-запись → 89.104.65.27, настроена в reg.ru ISPmanager)
- **Nginx**: `/etc/nginx/sites-enabled/open-webui` → proxy_pass 127.0.0.1:3000
- **SSL**: Let's Encrypt, сертификат до 24.06.2026, автообновление через certbot
- **URL**: `https://ai.npln.tech` (основной), `http://89.104.65.27:3000` (прямой)
- **PWA**: поддерживается — можно установить как приложение на телефон
- **Админ**: `s200586@gmail.com` (первый зарегистрированный, роль admin)

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
- Подключается через Docker ENV: `OPENAI_API_KEY` + `OPENAI_API_BASE_URL`
- **11 моделей**: openclaw, default, main, sa-gemini-west, sa-flash3-west, sa-flashlite-west, sa-gpt54-west, sa-codex-west, sa-claude-west, sa-gpt4o-west, sa-openai-sub-west
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
- [x] Подключение OpenClaw как модели (OpenAI-совместимый API)
- [x] DNS: A-запись ai.npln.tech → 89.104.65.27 (reg.ru ISPmanager)
- [x] SSL + nginx (reverse proxy, certbot, HTTPS)
- [x] PWA доступен для установки на мобильные
- [ ] Настройка пользователей и ролей (добавить сотрудников)
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
- **DNS**: A-запись `ai.npln.tech → 89.104.65.27` настроена через reg.ru ISPmanager (server58.hosting.reg.ru:1500). NS-серверы: ns1/ns2.hosting.reg.ru.
- **SSL**: `certbot --nginx -d ai.npln.tech` — сертификат получен 26.03.2026, автообновление настроено.
- **Ollama tunnel**: reverse SSH (`-R 11434:127.0.0.1:11434`) — Mac'овский Ollama виден на VPS как localhost:11434. Launchd с KeepAlive для автопереподключения.
- **Пользователи**: добавляются через Admin Panel → Users. Роли: admin, user, pending. Самостоятельная регистрация — через Admin → Settings → General → Enable Sign Up.
