# OpenClaw — Docker + Telegram Bot

Персональный AI-ассистент в Telegram, запущенный через Docker.
Использует OpenRouter как провайдер моделей (бесплатный тир, карта не нужна).

---

## Предварительные требования

- Docker Engine 20.10+ и Docker Compose v2
- API-ключ OpenRouter — [openrouter.ai](https://openrouter.ai) → Keys
- Telegram-бот — создаётся через @BotFather

Проверить версии:
```bash
docker --version
docker compose version
```

---

## Шаг 1 — Создать Telegram-бота

1. Открой Telegram, найди **@BotFather**
2. Отправь `/newbot`
3. Придумай имя (например: `My AI Assistant`) и username (например: `my_ai_bot`)
4. Скопируй токен бота

---

## Шаг 2 — Настроить переменные окружения

```bash
cp .env.example .env
```

Открой `.env` и заполни:
```env
OPENROUTER_API_KEY=sk-or-v1-...
```

---

## Шаг 3 — Запустить контейнер

```bash
docker compose up -d
```

Проверить статус:
```bash
docker compose ps
docker compose logs -f openclaw
```

Health-check:
```bash
curl http://127.0.0.1:18789/healthz
```

---

## Шаг 4 — Онбординг (первый запуск)

```bash
docker compose exec openclaw openclaw onboard
```

В процессе онбординга:
- **Auth method** → выбери OpenRouter, введи API-ключ
- **Default model** → `openrouter/meta-llama/llama-3.3-70b-instruct:free`
- **Tailscale** → Off
- **DM access** → No
- **Search provider** → Skip for now
- **Skills** → No
- **Plugins** → выбери `@openclaw/openrouter-provider`, Skip остальное
- **Hooks** → выбери `boot-md` (загружает SOUL.md при старте сессии)
- **Hatch** → Hatch in TUI

> Если сессия прервётся — запусти `openclaw onboard` снова, прогресс сохраняется.

---

## Шаг 5 — Подтвердить устройство (если TUI просит pairing)

В новом терминале:
```bash
docker compose exec openclaw openclaw devices list
docker compose exec openclaw openclaw devices approve <requestId>
```

---

## Шаг 6 — SOUL.md (личность ассистента)

`SOUL.md` уже лежит в `.docker/openclaw/workspace/SOUL.md` — это Сарказм-9000.
Шаблон хранится в `config-template/SOUL.md`.

Чтобы обновить личность — отредактируй `.docker/openclaw/workspace/SOUL.md` и перезапусти:
```bash
docker compose restart openclaw
```

---

## Шаг 7 — Проверить

Открой Telegram, найди своего бота и напиши ему. Должен ответить.

Логи в реальном времени:
```bash
docker compose logs -f openclaw
```

---

## Полезные команды

| Действие | Команда |
|---|---|
| Запустить | `docker compose up -d` |
| Остановить | `docker compose down` |
| Перезапустить | `docker compose restart openclaw` |
| Логи | `docker compose logs -f openclaw` |
| Войти в контейнер | `docker compose exec openclaw bash` |
| Сменить модель | `docker compose exec openclaw openclaw config set agents.defaults.model.primary openrouter/<model>` |
| Обновить образ | `docker compose down && docker compose pull && docker compose up -d` |
| Найти токен шлюза | `cat .docker/openclaw/config/openclaw.json \| grep token` |

---

## Структура проекта

```
.
├── docker-compose.yml              # Конфигурация Docker
├── .env.example                    # Шаблон переменных окружения
├── .env                            # Твои секреты (не коммитить!)
├── .gitignore
├── config-template/
│   ├── openclaw.json               # Шаблон конфига OpenClaw (справочно)
│   └── SOUL.md                     # Эталон личности ассистента
└── .docker/                        # Данные контейнера (не коммитить!)
    └── openclaw/
        ├── config/                 # openclaw.json, токены, логи
        └── workspace/
            └── SOUL.md             # Активная личность ассистента
```

---

## Устранение проблем

**Контейнер не запускается (exit code 137)**
Не хватает памяти. Добавь swap или освободи RAM.

**Permission denied на конфиг**
```bash
sudo chown -R 1000:1000 .docker/openclaw/
```

**Бот не отвечает в Telegram**
```bash
docker compose logs --tail 50 openclaw
# Должна быть строка: "starting provider (@username)"
```

**Модель не отвечает (429 rate limit)**
Бесплатные модели на OpenRouter перегружены. Смени модель:
```bash
docker compose exec openclaw openclaw config set agents.defaults.model.primary openrouter/openrouter/free
docker compose restart openclaw
```

**Токен шлюза не совпадает (pairing required)**
```bash
docker compose exec openclaw openclaw devices list
docker compose exec openclaw openclaw devices approve <requestId>
```

**SOUL.md не применяется**
Редактируй файл напрямую на хосте: `.docker/openclaw/workspace/SOUL.md`
После редактирования перезапусти контейнер и начни новую сессию в TUI.
