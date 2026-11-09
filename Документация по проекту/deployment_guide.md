# Deployment Guide — MOEX Trading Terminal

> **Версия:** v1.8 (2026-11-09 — блок A-02 «Старая история и дивиденды», в `develop` с 2026-11-09 (PR #121, мерж-коммит `d4223c9`): ревизия `79f3c238dbb8` поверх `b01e7a5c3d92`, head миграций `79f3c238dbb8` (§3.3, §7; пять таблиц — соответствия тикеров и дивиденды, признак «счёт для рыночных данных» у брокерских счетов; data-часть на пути выкатки удаляет 0 строк; откат удаляет ручные поправки и соответствия); **первый фон после старта увеличивает БД в разы** — на копии рабочей БД 114,7 → 1 877 МБ (×16,4) за 2 ч 5 мин, с копиями `backup_job` нужно ≈ 20 ГБ свободного места (§7); фоновая догрузка — задача планировщика (04:00 МСК и сразу после старта), вход пользователя её больше не запускает; T-Invest-часть фона и дивиденды — только «счётом для рыночных данных», который администратор выбирает после выкатки (§7); три новых задания планировщика (§7); новых настроек нет (§3.2); сеть — `smart-lab.ru` раз в ночь по седьмой части бумаг TQBR (§7); на рабочий терминал не выкатывалась); v1.7 (2026-11-06 — блок Б-01 «Новости: сборщик, архив, лента», в `develop` с 2026-11-06 (PR #116): ревизия `b01e7a5c3d92` поверх `0acf47e6607f`, head миграций `b01e7a5c3d92` (§3.3, §7; шесть таблиц `news_*`, семь источников и стартовый словарь; откат удаляет архив новостей); архив растёт ≈ 0,5 МБ в сутки (≈ 175 МБ в год), очистки нет (§7); два новых задания планировщика раз в минуту — сборщик и реакции (§7); новых настроек нет (§3.2), но backend ходит к лентам через прокси окружения, а ЦБ через прокси не отвечает — `www.cbr.ru` в `NO_PROXY`, решение заказчика при выкатке (§7); реакции на торги выходного дня вживую не проверены — с 09.11 на стенде A-02; на рабочий терминал не выкатывалась); v1.6 (2026-11-05 — блок Б-02 «Графики: панели и виды», в `develop` с 2026-11-05 (PR #111): ревизия `0acf47e6607f` поверх `cdb15f21e5ca`, head миграций `0acf47e6607f` (§3.3, §7; таблица раскладок графиков `chart_layouts` и индекс рисунков «пользователь + бумага», строки рисунков не переписываются); раскладка хранится на сервере — переносить из localStorage нечего, ключи рисунков в localStorage переносятся при входе (§7); прежние маршруты рисунков сняты — фронтенд и backend выкатываются вместе (§7, §9.6); новых настроек нет (§3.2); «внебиржевые часы» вживую не проверены — разовая проверка на стенде 07–08.11; на рабочий терминал не выкатывалась); v1.5 (2026-11-04 — блок A-05 «Дефекты этапа 1», в `develop` с 2026-11-04 (PR #113): ревизия `cdb15f21e5ca` поверх `d92b7dd5c939`, head миграций `cdb15f21e5ca` (§3.3, §7; три пустые колонки — отчёт разбора версии, лот бэктеста, недоставленные каналы уведомления); новых настроек нет (§3.2); фоновый повтор Telegram и почты каждые 10 с после старта (§3.3, §7); на рабочий терминал не выкатывалась); v1.4 (2026-11-04 — блок A-03 «Часы торгов, стопы внутри бара, дилер», в `develop` с 2026-11-04 (PR #112): ревизия `d92b7dd5c939` поверх `a858647f24e6`, head миграций `d92b7dd5c939` (§3.3, §7; таблица периодов расписания, алерты на общей модели, список «закрыть при первой возможности», «цена аукциона»); новая настройка `DEALER_STOPS_ENABLED` (по умолчанию выключена) и новое умолчание `MARKET_DATA_STREAM_MODE` = `single_include_weekend` (§3.2); фоновые задачи расписания после старта (§7); на рабочий терминал не выкатывалась); v1.3 (2026-10-24 — блок A-01 «Хранилище баров»: пять ревизий `b9d2f6a14c83` → `a858647f24e6`, head миграций `a858647f24e6` (§3.3, §7; последняя ревизия — история лота `ohlcv_lot_history` и удаление рядов свечей прежнего масштаба, дефект 4 тестового стенда), настройка `MARKET_DATA_STREAM_MODE` (§3.2); сборка сдана, на рабочий терминал не выкатывалась); v1.2 (2026-10-01 — фиксы аудита `S8R-AUDIT-NNN` до 043 и `S8R-FIX-029`: корень НУЦ Минцифры (§6б), привязка ключей `rebind-encrypted-secrets` и права бэкапов (§6, §6а), старт backend — dummy-хеш, очистка `revoked_tokens`, дисклеймер (§3.3), head миграций `fd3492233bc0` (§7, 2026-10-02…10 — S8R-FIX-056 индекс `upper(ticker)`, S8R-FIX-049 комиссия по сторонам, S8R-FIX-039 журнал CB, S8R-FIX-072 догоняющая ревизия `ai_provider_configs`, S8R-FIX-071 `live_trades.close_reason`); v1.1 — 2026-09-24…26, до 037; v1.0 — 2026-05-13, Sprint 8 W3, M4 Production-ready).
> **Целевая платформа:** Mac mini + Docker compose + launchd + Cloudflare Tunnel.
> Утверждено заказчиком 2026-05-12 (arch_design_s8 §7.2, batch 3 п.10).

---

## 1. Целевая платформа

MOEX Trading Terminal — single-user приложение, развёрнутое на личном Mac mini.

| Параметр | Минимум | Рекомендуется |
|---------|---------|---------------|
| Hardware | Mac mini Intel 2018 | Mac mini Apple Silicon (M1/M2/M3) |
| ОЗУ | 8 GB | 16 GB |
| SSD | 256 GB | 512 GB |
| OS | macOS 13 Ventura | macOS 14 Sonoma+ |
| Сеть | Стационарный домашний интернет (≥ 50 Мбит/с) | + UPS на роутер и Mac mini |
| Docker | Docker Desktop ≥ 24.x | Docker Desktop 25+ |

**NB (Apple Silicon):** на M1/M2/M3 контейнеры собираются под `linux/arm64`. Все используемые base-образы (`python:3.11-slim`, `node:24-alpine`, `nginx:alpine`) поддерживают arm64 нативно — Rosetta не требуется.

---

## 2. Предусловия

### 2.1 Установленные локально

- **Docker Desktop for Mac ≥ 24.x** — `docker --version`, `docker compose version`.
- **Git** — для `git pull` обновлений.
- **Homebrew** (рекомендуется) — для `cloudflared`.

### 2.2 Сторонние сервисы

- **Cloudflare аккаунт** — бесплатный, для Tunnel.
- **Домен на Cloudflare DNS** — хотя бы один (можно free `.workers.dev` или купленный).
- **T-Invest API token (production)** — получен в личном кабинете Тинькофф Инвестиций ([T-Invest API tokens](https://www.tinkoff.ru/invest/settings/api/)).
- **Telegram-бот** (опционально) — для notifications: создать через `@BotFather`, получить `BOT_TOKEN` и `chat_id`.

### 2.3 Сетевые требования

- Постоянный IP / DDNS НЕ требуется — Cloudflare Tunnel создаёт исходящий канал.
- Открытые порты НЕ нужны — все соединения исходящие к Cloudflare edge.

---

## 3. Установка

### 3.1 Клонирование репозитория

```bash
mkdir -p ~/Apps && cd ~/Apps
git clone <REPO_URL> moex-terminal && cd moex-terminal
```

> Replace `<REPO_URL>` на актуальный private repo URL.

### 3.2 Production `.env`

```bash
cd Develop
# Шаблон .env.example лежит в корне Develop/; docker-compose (env_file)
# читает production-конфиг из backend/.env.production — туда и копируем.
cp .env.example backend/.env.production
# Откройте `backend/.env.production` и заполните:
```

> **Полный перечень настроек** с дефолтами — `docs/env_vars.md` в репозитории кода (S8R-AUDIT-051; тест сверяет его с `app/config.py` в обе стороны). Ниже — то, что оператор задаёт при установке. Ключ T-Invest в `.env` **не задаётся** — каждый пользователь подключает свой в UI (Настройки → Брокер); часовой пояс и чат Telegram тоже не настраиваются через env.

| Переменная | Назначение | Как получить |
|------------|-----------|--------------|
| `SECRET_KEY` | JWT signing key (**≥ 32 байт**, ≥ 8 различных символов, без `dev-` и без `change-me`/`CHANGE_ME` — плейсхолдер шаблона отбраковывают preflight и валидатор настроек, S8R-AUDIT-034) | `python3 -c "import secrets; print(secrets.token_hex(32))"` |
| `ENCRYPTION_KEY` | AES-256-GCM master key для шифрования broker token'ов (**≥ 32 байт**, те же правила, что у `SECRET_KEY`; проверяется **на старте** backend, а не при первом запросе) | `python3 -c "import secrets; print(secrets.token_urlsafe(32))"` |
| `DATABASE_URL` | SQLite путь внутри контейнера | `sqlite+aiosqlite:////app/data/app.sqlite` (НЕ менять) |
| `TINVEST_UNARY_TIMEOUT_SEC` | (опционально) Дедлайн одного gRPC-вызова T-Invest, сек (S8R-AUDIT-030); стрим котировок не затрагивает | по умолчанию `10` |
| `MARKET_DATA_STREAM_MODE` | (опционально) Источник свечей в подписке потока T-Invest (A-01; умолчание сменено в A-03, ADR 0021): `single_include_weekend` — одна подписка с дилерскими свечами выходного дня и раскладкой по источнику: биржевые — в живую свечу и основной ряд, дилерские — в `ohlcv_dealer`, канал `dealer:{ticker}:{tf}` и стоп у дилера, неизвестные — никуда; `two_subscriptions` — только биржевые свечи, дилерского потока нет (откат). Если в `.env.production` с A-01 осталась строка `two_subscriptions` — удалите её, иначе дилерский поток и стопы у дилера не работают. Менять только по решению разработчика | по умолчанию `single_include_weekend` (до A-03 — `two_subscriptions`) |
| `DEALER_STOPS_ENABLED` | (опционально) Признак «стопы у дилера» (A-03, ADR 0022): `true` — стоп в часы дилера (выходные вне торгов биржи) исполняется по дилерским ценам после проверки статуса бумаги, флагов API и спреда дилерского стакана (≤ 20 б.п.); `false` — стоп в часы дилера ждёт торгов биржи. **Не включать** ни в шаблоне, ни на стендах: включает заказчик после своей ручной заявки у дилера — строкой `DEALER_STOPS_ENABLED=true` и перезапуском backend; переключателя в интерфейсе нет | по умолчанию `false` |
| `GRID_MAX_WORKERS_TOTAL` | (опционально) Общий предел процессов Grid Search на все параллельные job'ы (S8R-AUDIT-078); `0` — авто `cpu−1` | по умолчанию `0`; уменьшить, если live-торговля соседствует с гридами |
| `BACKTEST_EXEC_TIMEOUT_SEC` | (опционально) Предел времени исполнения стратегии в одном бэктесте и в одной комбинации Grid Search (S8R-AUDIT-006); по истечении процесс исполнения убивается, бэктест — `failed` | по умолчанию `900` |
| `BACKTEST_EXEC_MEMORY_LIMIT_MB` | (опционально) Бюджет памяти процесса исполнения стратегии сверх базы (RLIMIT_AS, только Linux/Docker) | по умолчанию `2048` |
| `CORS_ORIGINS` | Публичный origin SPA (что видит браузер за Tunnel), через запятую; без него WS отбиваются 403. Preflight: пусто при `DEBUG≠true` → контейнер не стартует, только localhost → предупреждение (S8R-AUDIT-055). Формат — только `http(s)://host[:port]`; регистр, завершающий `/` и порт по умолчанию не важны, записи без схемы и с пробелом внутри игнорируются (предупреждение), разделитель — только запятая; `*` при `DEBUG≠true` → контейнер не стартует, http-origin не-localhost → предупреждение (S8R-AUDIT-059). С S8R-AUDIT-038 без верного значения 403 получают не только WS, но и вход и все изменяющие запросы. Swagger (`/docs`, `/openapi.json`) в production выключен. | `https://moex.example.com` (+ `http://localhost` для local-only §5.5) |
| `WS_MAX_CONNECTIONS_PER_USER` | (опционально) Предел WebSocket-соединений на пользователя (S8R-AUDIT-058), диапазон 10–50; менять синхронно с `limit_conn` в `nginx.conf` (60 на IP — должен оставаться выше) | по умолчанию `30` |
| `AI_GLOBAL_DAILY_TOKEN_LIMIT`, `AI_GLOBAL_MONTHLY_TOKEN_LIMIT` | (опционально) Потолок токенов общего AI-ключа сервера (`AI_API_KEY`) на инстанс за UTC-сутки/месяц (S8R-AUDIT-014); 0 = без ограничения | по умолчанию `1000000` и `20000000` |
| `AI_ALLOW_PRIVATE_PROVIDER_URLS` | (опционально) Разрешить AI-провайдер на приватном адресе (self-hosted LLM). Задать такой адрес может только администратор (S8R-AUDIT-018); при заданном `HTTPS_PROXY` закреплённый резолв для AI не применяется — имя резолвит прокси. Для известных AI-провайдеров за Proxifier/VPN в режиме fake-IP флаг не нужен (S8R-FIX-130) | по умолчанию `false` |
| `COOKIE_SECURE` | (опционально) Флаг Secure у cookie сессии (S8R-AUDIT-040). Не задано — в production всегда Secure; `auto` — по схеме запроса; `true`/`false` — принудительно (алиасы 1/yes/on, 0/no/off). Доступ без TLS (local-only) — `false`, **не** `DEBUG=true`; preflight предупредит | по умолчанию не задано |
| `POSITION_CHECK_RETRIES`, `POSITION_CHECK_RETRY_DELAY_SEC` | (опционально) Сколько раз и с какой паузой перепрашивать портфель песочницы, пришедший без бумаг (S8R-AUDIT-011). Диапазоны 3–5 и 5–10 с; значение вне диапазона приводится к границе, в лог — warning | по умолчанию `3` и `5` |
| `TELEGRAM_BOT_TOKEN` | (опционально) Telegram уведомления | `@BotFather` |

> ℹ️ **SPA собирается с относительными адресами** (`/api/v1`, `wss://<host>`, S8R-AUDIT-055): `VITE_*` задавать не нужно, nginx проксирует `/api` и `/ws`. `ARG VITE_API_BASE_URL` в `frontend/Dockerfile` — только для нестандартной раскладки. Локально (`scripts/start.sh`) ту же роль играет dev-proxy Vite (`/api`, `/ws` → :8000).

> ℹ️ **Остановка контейнера backend** (S8R-AUDIT-074): graceful shutdown рантайма укладывается в 45 с (дожидается записи id отправленного ордера, останавливает сессии, прогоняет recovery). Поэтому в `docker-compose.yml` для `backend` задан `stop_grace_period: 90s` (S8R-FIX-065: 10 + 45 + 6 + 15 = 76 с) — не уменьшайте его: при значении Docker по умолчанию (10 с) процесс получает SIGKILL посреди записи ответа брокера.

> ⚠️ **`.env.production` НЕ коммитится в git!** Файл уже включён в `.gitignore` (`.env.*`). Перед `git commit` проверьте `git status` — никаких `.env*` в diff не должно быть.

### 3.3 Сборка и запуск Docker compose
> ℹ️ На старте backend повторно доставляет необработанные критические события прошлого запуска (`pending_events`, окно 1 ч, S8R-AUDIT-086); при остановке дожидается фоновых записей outbox (≤ 6 с).
> ℹ️ S8R-FIX-051: на старте backend дозаполняет FIGI ценовых алертов и подписывается на их стрим через T-Invest-счёт владельца; активные алерты с уже пересечённым уровнем сработают на первой свече после старта. Недоставленные Telegram/Email-уведомления прошлого запуска досылаются фоном (без дублей). *С A-03 алерты — таблица `alerts`: у новых направление ставит сервер по цене при создании, «уже пересечённого» уровня у них не бывает; перенесённые из `price_alerts` сохранили прежнее направление и могут сработать на первой минуте после старта.*
> ℹ️ A-03: на старте backend загружает таблицу периодов расписания (`trading_schedule_periods`) в память до планировщика и восстановления сессий (сбой — warning `trading_schedule_startup_load_failed`, работа на запасных часах), затем сразу запускает `sync_trading_schedule` (T-Invest `TradingSchedules`, сегодня + 7 дней; лог `trading_schedule_synced`) и `backfill_trading_schedule_iss` (история из минут ISS с 13.06.2018 — при первом старте ≈ 1 ч фоном). Сделки из списка «закрыть при первой возможности» после старта повторяются раз в минуту (§7).
> ℹ️ A-05: каждое уведомление с включённым Telegram или почтой пишет строку доставки `notification.dispatch` в `pending_events`. Недоставленные каналы прошлого запуска досылаются на старте (это одна из пяти попыток), дальше задача `retry_notification_delivery` раз в 10 с повторяет каналы по сроку (10 с / 1 мин / 5 мин / 30 мин) и после пятой неудачи ставит отметку «не доставлено в Telegram / на почту»; строка старше 1 ч с недоставленным каналом получает отметку без отправки. Работает только при `DEV_MODE=false` и настроенных `TELEGRAM_BOT_TOKEN` / SMTP (§7).
> ℹ️ A-02: на старте backend планировщик сразу запускает фоновую догрузку истории `history_backfill` (дальше — ежедневно в 04:00 МСК): по бумагам сессий, бэктестов, активных алертов, избранного и прежних целей — минуты ISS до 13.06.2018 с декабря 2011 и, если выбран «счёт для рыночных данных», история T-Invest. Первый прогон после выкатки идёт часами и увеличивает БД в разы — §7, блок A-02. Вход пользователя фон больше не запускает.
> ℹ️ S8R-AUDIT-050: backend ставится строго из `backend/requirements.lock` (`--no-deps --require-hashes`, хэши каждого пакета), SDK T-Invest — `scripts/install_tinvest_sdk.sh` по commit-hash из `pyproject.toml`, после установки — `pip check` и `scripts/verify_install.py` (extras); base-образы закреплены по `@sha256` и обновляются PR Dependabot (S8R-AUDIT-049); pnpm — 9.15.9 (`packageManager`). Обновление зависимостей: правка `pyproject.toml` → `backend/scripts/lock_deps.sh` → коммит обоих lock. Лицензии — `docs/licenses.md` репо кода.
> ℹ️ S8R-AUDIT-043: на старте backend один раз считает служебный хеш Argon2 для проверки несуществующих логинов (~50 мс, в потоке) и сразу чистит истёкшие записи `revoked_tokens` (далее — раз в 24 ч). После обновления каждому пользователю при первом входе покажется окно дисклеймера (одно нажатие «Принимаю»); существующие пароли продолжают работать, новая политика (≥ 12 символов, буква и цифра) действует при смене пароля.


```bash
cd ~/Apps/moex-terminal/Develop
docker compose build         # ~5-10 минут на первой сборке (установка python- и node-зависимостей)
docker compose up -d
docker compose ps            # должно показать оба сервиса: Up X seconds (healthy)
```

Ожидаемый вывод `ps`:

```
NAME            STATUS                      PORTS
moex-backend    Up 2 minutes (healthy)
moex-frontend   Up 1 minute  (healthy)      127.0.0.1:80->80/tcp
```

Проверка эндпоинтов:

```bash
curl -fsS http://localhost/                  # HTML с <div id="root">
curl -fsS http://localhost/api/v1/health     # {"status":"ok", "cb_state":"ok", ...}; 503 — degraded
```

**Реальный IP клиента (S8R-AUDIT-037).** Backend запускается командой `uvicorn app.main:app --host 0.0.0.0 --port 8000 --proxy-headers --forwarded-allow-ips="$TRUSTED_PROXY_IPS"`. `TRUSTED_PROXY_IPS` — от кого backend принимает `X-Forwarded-For`; по умолчанию `172.28.0.10` — фиксированный адрес nginx в сети `moex-net` (`172.28.0.0/24`). Переопределяется в окружении или `.env` compose (IP/CIDR через запятую). Та же пара `--proxy-headers`/`TRUSTED_PROXY_IPS` передаёт backend и схему запроса (`X-Forwarded-Proto`, S8R-AUDIT-040). При ошибке `Pool overlaps` сменить одновременно подсеть `networks.moex-net.ipam`, `ipv4_address` сервиса frontend, `TRUSTED_PROXY_IPS` и `set_real_ip_from` в `nginx.conf`.

#### Проверка миграций на чистой установке

`command` backend-контейнера выполняет `alembic upgrade head` до старта uvicorn
(`docker-compose.yml`), поэтому отдельно миграции запускать не нужно. Но на
**чистой** БД схему обязательно надо сверить — молчаливое расхождение миграций
с моделями (forward model drift) уже приводило к неработающему входу:

```bash
docker compose exec backend alembic current   # ожидается: 79f3c238dbb8 (head; с A-02, до него — b01e7a5c3d92, до Б-01 — 0acf47e6607f, до Б-02 — cdb15f21e5ca, до A-05 — d92b7dd5c939, до A-03 — a858647f24e6)
docker compose exec backend alembic heads     # та же ревизия — расхождений нет
```

> **⚠️ Историческая справка (S8R-ALEMBIC-FRESH-DB-DRIFT, 2026-07-28).**
> До ревизии **`d1e2f3a4b5c6`** (`s8r_fresh_db_drift`) сценарий именно из этого
> раздела был сломан: на **чистой** БД после `alembic upgrade head` не
> создавались таблица `user_ai_settings` и восемь колонок
> (`strategies.description`, `instruments.logo_name`, 6 колонок в
> `ai_provider_configs`) — модели ушли вперёд миграций. Первый же вход
> (`POST /api/v1/auth/login`) отдавал **500**
> `no such column: strategies.description`, то есть развёртывание с нуля по
> этому гайду давало неработающую систему. На уже работающих стендах дефект
> не проявлялся — их БД получили недостающие объекты другим путём (`init_db`),
> поэтому он и не всплывал три недели.
> Миграция `d1e2f3a4b5c6` **идемпотентна** (каждый объект добавляется только
> если его ещё нет), поэтому безопасна и для чистой, и для существующей БД.
> Регресс-защита в CI —
> `backend/tests/unit/test_migration.py::test_fresh_db_schema_matches_models`:
> сверяет всю `Base.metadata` (таблицы **и** колонки) с фактической схемой
> чистой БД. Класс ловушки —
> [`Develop/stack_gotchas/gotcha-13-forward-model-drift.md`](../Develop/stack_gotchas/gotcha-13-forward-model-drift.md).

Если `alembic current` пуст или ниже `d1e2f3a4b5c6` — контейнер стартовал без
миграций (см. §9.1), вход работать не будет.

### 3.4 Bootstrap первого администратора

При первом запуске backend применяет миграции (`alembic upgrade head`) — таблицы создаются пустыми.

**Вариант A:** через UI.
- Откройте `http://localhost` в Safari/Chrome → `FirstRunWizard` (4 шага).
- Зарегистрируйте первого пользователя — он автоматически получает `is_admin=true` (см. C-S8-7 DEV-1 W1, эпик Admin role).

**Вариант B:** через CLI (если уже есть users без admin'а).
```bash
docker compose exec backend python -m app.cli.users grant_admin <username>
# снять права (S8R-AUDIT-098): последнего активного администратора — нельзя (exit 1)
docker compose exec backend python -m app.cli.users revoke_admin <username>
```

Ошибки БД в CLI (`users`, `backup`) — строка `Database error: …` и код выхода 1, без traceback.

> **С 2026-09-24 (S8R-AUDIT-033):** `POST /api/v1/auth/setup` работает только на пустой БД —
> после первого пользователя он возвращает **403 «Регистрация закрыта»**. Регистрируйте
> первого пользователя сразу после выкатки, до публикации адреса через Cloudflare Tunnel.

**Дополнительные пользователи** (только admin, UI нет):
```bash
curl -X POST https://<домен>/api/v1/admin/users \
  -H 'Authorization: Bearer <access_token администратора>' -H 'Content-Type: application/json' \
  -d '{"username":"<логин>","password":"<пароль от 8 символов>"}'
# 201 — создан (is_admin=false); 409 — имя занято; 403 — вы не администратор
```

---

## 4. Auto-start через launchd

Чтобы контейнеры поднимались автоматически после перезагрузки Mac mini:

```bash
# 1. Скопируйте plist в ~/Library/LaunchAgents/
cp "~/Apps/moex-terminal/Документация по проекту/launchd/com.moex.terminal.plist" \
   ~/Library/LaunchAgents/

# 2. Откройте в редакторе и замените placeholder'ы:
#    __USER__         → ваш macOS username (например, `sergopipo`)
#    __PATH_TO_REPO__ → путь от ~ (например, `Apps/moex-terminal`)
nano ~/Library/LaunchAgents/com.moex.terminal.plist

# 3. Проверка синтаксиса:
plutil -lint ~/Library/LaunchAgents/com.moex.terminal.plist
# Ожидается: "OK"

# 4. Загрузка:
launchctl load -w ~/Library/LaunchAgents/com.moex.terminal.plist

# 5. Проверка статуса:
launchctl list | grep com.moex.terminal
```

Чтобы протестировать auto-start — перезагрузите Mac mini, через 60 секунд после загрузки проверьте `docker compose ps` (должны быть оба контейнера healthy).

**Известный нюанс:** Docker Desktop стартует не моментально после login. Если launchd запустился до Docker — `docker compose up -d` упадёт. KeepAlive=false означает, что launchd попытку не повторит. Workaround: вручную выполнить `docker compose up -d` либо настроить Docker Desktop «Start Docker Desktop when you log in» в Settings.

---

## 5. SSL через Cloudflare Tunnel

Cloudflare Tunnel = исходящий TLS-туннель от Mac mini до Cloudflare edge. Не требует открытых портов на роутере, бесплатен.

### 5.1 Установка cloudflared

```bash
brew install cloudflare/cloudflare/cloudflared
cloudflared --version       # должно быть ≥ 2024.x
```

### 5.2 Авторизация и создание туннеля

```bash
cloudflared tunnel login                              # откроется браузер для подтверждения домена
cloudflared tunnel create moex-terminal               # создаёт ~/.cloudflared/<UUID>.json
cloudflared tunnel route dns moex-terminal moex.example.com  # автоматически добавляет DNS-запись
```

### 5.3 Конфиг `~/.cloudflared/config.yml`

```yaml
tunnel: <TUNNEL_UUID>             # из вывода `cloudflared tunnel create`
credentials-file: /Users/<USER>/.cloudflared/<TUNNEL_UUID>.json

ingress:
  - hostname: moex.example.com
    service: http://127.0.0.1:80   # не localhost: на macOS он сначала резолвится в ::1, а nginx слушает только IPv4
  - service: http_status:404      # fallback для прочих hostnames
```

nginx опубликован только на `127.0.0.1:80` (S8R-AUDIT-037): извне к нему ходит только cloudflared с этого же Mac mini. Доступ из локальной сети (`"80:80"`) — осознанное изменение: придётся править тест `test_compose_trusts_forwarded_for_only_from_nginx` и доверие `CF-Connecting-IP`, иначе любой клиент сети подделает свой IP для лимитера входа.

**Проверка при первом запуске.** Открыть сайт через Tunnel и выполнить `docker compose logs frontend | tail`. Первое поле строки запроса — публичный IP клиента. Если там `172.x`/`192.168.x` — это адрес cloudflared, которому nginx не поверил: добавить его в `set_real_ip_from` (`nginx.conf`) и выполнить `docker compose restart frontend`. Иначе все пользователи делят один ключ лимитера входа.

**Проверка заголовков (S8R-AUDIT-057).** `curl -sI https://<домен>/ | grep -i -E 'content-security|x-frame|strict-transport|cache-control'` — все есть, по одному; то же для `/assets/<файл>.js` (`immutable`). В конструкторе стратегий — кнопки масштаба на месте, в консоли нет `Refused to…`.

**Проверка Secure (S8R-AUDIT-040).** После входа через `https://<домен>` открыть DevTools → Application → Cookies: у `access_token`, `refresh_token`, `csrf_token` стоит ✓ Secure. В Cloudflare включить *Always Use HTTPS*. nginx пропускает `X-Forwarded-Proto: https` только от адресов cloudflared (те же, что `set_real_ip_from`).

### 5.4 Запуск как macOS service

```bash
sudo cloudflared service install                       # регистрирует launchctl service
sudo launchctl start com.cloudflare.cloudflared        # если не стартует автоматически
```

После этого `https://moex.example.com` будет доступен с любого устройства (TLS термirовался на Cloudflare edge).

### 5.5 Альтернатива: self-signed (local-only сценарий)

Если выход в интернет не нужен — можно ограничиться `http://localhost` или сгенерировать self-signed cert и добавить TLS-секцию в `nginx.conf`. Не рекомендуется для production (предупреждение в браузере, нет Let's Encrypt автоматического обновления).

---

## 6. Backup и restore

### 6.1 Автоматический backup_job

В `app/scheduler/service.py` зарегистрирован `backup_job` (APScheduler, cron daily 03:00 UTC (06:00 МСК) — см. S7 DEV-1). Пишет snapshot SQLite через `sqlite3.Connection.backup` (Online Backup API, WAL-safe, gotcha-19) с проверкой `PRAGMA integrity_check` сразу после создания (битый снимок удаляется) в `BACKUP_DIR/backup_YYYY-MM-DD_HHMMSS.sqlite` (при совпадении секунды — суффикс `_2`). `BACKUP_DIR` — настройка (по умолчанию `backups` относительно `backend/`, в контейнере `/app/backups`). Рядом лежит `.lock` — межпроцессная блокировка `flock` для создания и восстановления (планировщик и CLI не пересекаются); каталог должен быть на локальном томе (не NFS/SMB). S8R-AUDIT-092.

Просмотр:
```bash
docker compose exec backend ls -la /app/backups
```

Файлы переживают `docker compose down` (хранятся в named volume `moex-sqlite-backups`).

Права (S8R-AUDIT-042): файлы снимков, `before_restore_*`, `restore_tmp_*`, дампы и `.lock` — 0600 с момента создания; при старте сервис приводит к 0600 свои старые файлы. Каталог 0700 ставится, только если его создал сервис; у существующего каталога с открытыми правами в логе `backup_dir_permissions_open` — выполните `chmod 700 <BACKUP_DIR>`. Restore переносит на восстановленную БД права и группу прежней.

### 6.2 Cron на хосте (опционально, для off-site copy)

```cron
# Каждый день в 04:00 копировать backup'ы на внешний диск.
0 4 * * * docker run --rm \
    -v moex-sqlite-backups:/backup:ro \
    -v /Volumes/External:/dest \
    alpine sh -c "mkdir -p /dest/$(date +\%Y\%m\%d) && cp -r /backup/* /dest/$(date +\%Y\%m\%d)/"
```

### 6.3 Restore

> ⚠️ **С 2026-09-24 (S8R-AUDIT-092)** — не копируйте файл поверх БД вручную (`docker compose cp … app.sqlite`):
> при непустом `-wal` это теряет подтверждённые транзакции. Восстановление — только через CLI.

```bash
# 1. Остановить backend (restore под живым сервером CLI отклонит, код 3):
docker compose stop backend

# 2. Положить снимок в BACKUP_DIR (CLI отказывает путям вне него) и восстановить:
docker compose run --rm backend python -m app.cli.backup restore --path /app/backups/<файл>.sqlite --yes
#    CLI: проверяет целостность снимка (под локом), снимает текущую БД в before_restore_*
#    (с WAL), копирует снимок во временный файл и атомарно подменяет БД (os.replace).
#    --server-stopped — только если порт 8000 занят ЧУЖИМ сервисом.

# 3. Запустить backend (миграции применятся при старте):
docker compose start backend
```

---

## 6а. Ротация секретов (S8R-AUDIT-041)

При утечке `.env` или по регламенту ключи меняются без повторного ввода токенов брокеров и AI.

```bash
cd ~/Apps/moex-terminal/Develop
# 1. Остановить backend (CLI откажется работать под живым сервером, код 3):
docker compose stop backend

# 2. Сгенерировать новый ключ:
python3 -c 'import secrets; print(secrets.token_urlsafe(48))'

# 3. Перешифровать данные (снимок БД делается автоматически перед изменениями):
docker compose run --rm -e OLD_ENCRYPTION_KEY='<старый>' -e NEW_ENCRYPTION_KEY='<новый>' \
  backend python -m app.cli rotate-encryption-key

# 4. Записать новый ENCRYPTION_KEY в backend/.env.production и запустить:
docker compose start backend
docker compose logs backend | grep encryption_key_mismatch   # должно быть пусто
```

- Снимок до ротации и все старые бэкапы зашифрованы старым ключом: при утечке — увести офлайн или удалить.

**Нативный стенд (`scripts/start.sh`, без Docker).** Тот же порядок, файл окружения — `Develop/.env`
(`backend/app/config.py`, `_ENV_FILE`). Ключи вводятся через `read -rs` — не попадают ни в историю shell,
ни на экран:

```bash
# 1. Остановить стенд: Ctrl+C во вкладке, где запущен scripts/start.sh (CLI под живым сервером откажет, код 3).
# 2. Свой снимок БД (backup API, gotcha-19) — дополнительно к автоматическому снимку CLI:
cd ~/Documents/Claude_Code/Moex-Terminal/Develop/backend
.venv/bin/python -c "import sqlite3; s=sqlite3.connect('data/terminal.db'); d=sqlite3.connect('data/terminal.db.bak-pre-key-rotation'); s.backup(d); d.close(); print('ok')"
# 3. Новый ключ — сгенерировать и сохранить в менеджер паролей:
python3 -c 'import secrets; print(secrets.token_urlsafe(48))'
# 4. Перешифровать (старый ключ — из Develop/.env, новый — из шага 3; ввод скрыт):
read -rs OLD_ENCRYPTION_KEY && read -rs NEW_ENCRYPTION_KEY && export OLD_ENCRYPTION_KEY NEW_ENCRYPTION_KEY
.venv/bin/python -m app.cli rotate-encryption-key
unset OLD_ENCRYPTION_KEY NEW_ENCRYPTION_KEY
# 5. Записать новый ENCRYPTION_KEY в Develop/.env (редактором), запустить стенд:
cd .. && GRPC_DNS_RESOLVER=native bash scripts/start.sh   # GRPC_DNS_RESOLVER не нужен после S8R-FIX-093 в develop
# 6. В ДРУГОЙ вкладке (start.sh занимает свою): в журнале нет encryption_key_mismatch (ожидается 0);
#    «Счёт» и Telegram /balance показывают балансы обоих счетов (ключи брокера расшифровываются);
#    сессии восстановились (session_runtime_restore_complete).
grep -c encryption_key_mismatch ~/Documents/Claude_Code/Moex-Terminal/Develop/backend/logs/dev.log
```

- Если шаг 4 завершился ошибкой — `Develop/.env` не трогать: старый ключ остаётся рабочим; CLI до изменений
  делает снимок в `BACKUP_DIR`, свой снимок — `data/terminal.db.bak-pre-key-rotation`.
- После проверки снимок `terminal.db.bak-pre-key-rotation` и прежние `terminal.db.bak-*` зашифрованы **старым**
  ключом — увести офлайн или удалить.
- Смена `SECRET_KEY` (JWT): `docker compose run --rm backend python -m app.cli rotate-jwt-secret`, затем новый `SECRET_KEY` в `.env.production` и `docker compose restart backend`; все пользователи входят заново.
- **Откат кода после деплоя версии с S8R-AUDIT-041 = повторный ввод ключей брокеров и AI**: новый формат шифртекста старой версией не читается.

**Привязка ключей к записям (S8R-AUDIT-042)** — однократно после обновления на версию с 042:

```bash
docker compose stop backend
docker compose run --rm backend python -m app.cli rebind-encrypted-secrets
#   локально: cd backend && python -m app.cli rebind-encrypted-secrets --server-stopped
docker compose start backend
```

Команда привязывает ключи брокеров и AI, сохранённые до 042, к их записям; перед записью делает снимок в `BACKUP_DIR`; повторный запуск безопасен («nothing to rebind»); `ENCRYPTION_KEY` не меняется. Снимок перед ней и все более старые снимки содержат ключи без привязки: восстановление такого снимка возвращает записи, которые приложение читает через совместимый режим. Чтобы закрыть это полностью — после rebind выполнить `rotate-encryption-key` (старые снимки после этого читаются только старым ключом — храните его вместе с ними или удалите снимки) либо увести старые снимки офлайн. Откат кода ниже 042 после rebind = повторный ввод ключей.

---

## 6б. Сертификат T-Invest — корень НУЦ Минцифры (S8R-FIX-029)

Все gRPC-эндпоинты T-Invest (sandbox и prod) подписаны цепочкой до **Russian Trusted Root CA** Минцифры. Этого корня нет во встроенных корнях `grpcio`, поэтому без него любой вызов брокера падает `Tls handshake failed` (gotcha-55).

- Терминал при старте сам собирает набор «встроенные корни grpc + `backend/app/broker/tinvest/certs/russian_trusted_root_ca.pem`» и выставляет `GRPC_DEFAULT_SSL_ROOTS_FILE_PATH`. Ручная настройка не нужна; в Docker-образ файл попадает вместе с кодом.
- Если оператор задал `GRPC_DEFAULT_SSL_ROOTS_FILE_PATH` сам — терминал её не меняет (тогда в указанном файле должен быть и этот корень).
- HTTPS (ISS, AI, Telegram) и хранилище ОС не затрагиваются.
- Источник: https://www.gosuslugi.ru/crt, SHA-256 `D2:6D:2D:02:31:B7:C3:9F:92:CC:73:85:12:BA:54:10:35:19:E4:40:5D:68:B5:BD:70:3E:97:88:CA:8E:CF:31`, действует до **27.02.2032**.
- Замена: новый PEM в тот же путь, обновить `EXPECTED_SHA256` в `tests/unit/test_broker/test_grpc_trusted_roots.py`, пересобрать образ. Тест падает за 180 дней до истечения.
- В журнале: `grpc_trusted_roots_configured` (набор собран), `grpc_trusted_roots_preset` (переменная задана оператором), `grpc_trusted_roots_setup_failed` (сбой — брокер будет недоступен).

## 6в. Резолвер DNS для gRPC (S8R-FIX-083, S8R-FIX-093)

gRPC разрешает имена **не системным резолвером**, а встроенным c-ares. На машине заказчика (macOS, 2026-10-12) это занимало **10–11 с** при дедлайне unary-вызова `TINVEST_UNARY_TIMEOUT_SEC` = 10 с: каждый вызов к брокеру падал по таймауту, при этом стрим свечей оставался живым (мультиплексор держит один долгий канал, а адаптер создаёт новый на каждый вызов). Снаружи это выглядело как «брокер не отвечает» и держалось пять дней.

- Терминал при старте выставляет `GRPC_DNS_RESOLVER=native` — тот же путь разрешения имён, что у остальных клиентов процесса. Ручная настройка не нужна.
- Если оператор задал `GRPC_DNS_RESOLVER` сам — терминал её не меняет. Так можно вернуть `ares`, если на конкретной машине он быстрее.
- Переменная читается gRPC **при импорте модуля** (grpcio 1.84), а не при первом канале, поэтому терминал выставляет её при импорте пакета `app` — раньше любого кода. До S8R-FIX-093 она ставилась в lifespan, уже после того как шаг корней (§6б) импортировал grpc: в журнале было `resolver=native`, а работал c-ares (замер 2026-10-13: 11,08 с против 0,03 с). Задавать вручную — только в окружении процесса до старта (`environment:` в compose, `export` перед `start.sh`); в `.env` бесполезно, после старта — нужен перезапуск.
- В журнале: `grpc_dns_resolver_configured` (выставлено терминалом), `grpc_dns_resolver_preset` (задано оператором), `grpc_dns_resolver_ineffective` — **warning**: grpc импортирован раньше пакета `app` (сторонний скрипт или инструмент), значение не подействовало — задать переменную в окружении процесса.
- **Диагностика при «брокер не отвечает»:** сравнить время готовности gRPC-канала по имени и по IP. Если по IP быстро, а по имени ~10 с — причина здесь, а не у брокера и не в TLS.

## 7. Обновление до новой версии

```bash
cd ~/Apps/moex-terminal
git pull origin main

cd Develop
docker compose build         # пересобрать образы (только изменившиеся слои)
docker compose down          # без -v — volumes сохраняются
docker compose up -d
docker compose ps            # проверить healthy
```

Миграции `alembic upgrade head` выполняются в entrypoint backend контейнера автоматически — отдельно запускать не нужно.

**Дрейф схемы (S8R-AUDIT-004).** В CI проверяется `alembic check` на чистой БД — модели и миграции обязаны совпадать. Новую ревизию через `alembic revision --autogenerate` ревьюить вручную; непустой `alembic check` после неё — дрейф (gotcha-11).

**Первое обновление на версию с проверкой внешних ключей (S8R-AUDIT-015).** Перед обновлением снимите копию рабочей БД через `sqlite3.Connection.backup` (не `cp` — gotcha-19) и выполните на **копии** `PRAGMA foreign_key_check;` — сохраните вывод. Висячие ссылки не мешают старту (`alembic upgrade head` при применении ревизий пишет WARNING со сводкой), но отказ получат операции, которые записывают висячий внешний ключ или удаляют родителя, на которого ещё ссылаются (например, удаление стратегии с историей сессий — 422). При удалении брокерского счёта у его остановленных сессий `broker_account_id` обнуляется (сделки сохраняются).

После апдейта сверьте, что миграции доехали до головы:

```bash
docker compose exec backend alembic current   # ожидается: 79f3c238dbb8 (head)
```

> **ℹ️ Блок A-02 «Старая история и дивиденды» — ревизия `79f3c238dbb8` (поверх `b01e7a5c3d92`, 2026-11-09).**
> `alembic current` после обновления: `79f3c238dbb8 (head)`. С `v1.0.0` (`a7c1d9e2b4f8`) `alembic upgrade head` проходит
> пять ревизий A-01, ревизии A-03, A-05, Б-02, Б-01 и эту одним запуском (при ветвлении блока родителем была `a858647f24e6`;
> при мержах с `develop` ревизия перевешивалась на `cdb15f21e5ca`, `0acf47e6607f`, затем на `b01e7a5c3d92`; содержимое не
> менялось). Ревизия одна на весь блок:
> * **схема** (ТЗ §3.2, §3.5.6, §3.27): `ticker_aliases` (соответствия тикеров); `dividend_payments`, `dividend_corrections`,
>   `dividend_smartlab_rows`, `dividend_sources` (дивиденды); колонка `broker_accounts.is_market_data_source` (по умолчанию 0)
>   и частичный уникальный индекс `uq_broker_accounts_market_data_source` (`WHERE is_market_data_source = 1`).
>   **Счёт для рыночных данных миграция не выбирает** — ниже;
> * **data-часть** (окна источников, ТЗ §5.6.8; свечи — вместе с покрытием тех же рядов, gotcha-90): фантомные дневки T-Invest
>   раньше 2019-01-01 (O, H, L, C, V равны предыдущей строке; покрытие остаётся), минуты T-Invest и дилерские раньше шва
>   2018-06-13, дневки T-Invest окна 07.03–12.06.2018, ISS-дневки окна с меткой не 00:00 (покрытие вырезается — ряды
>   перезагрузятся). **На пути выкатки поверх A-01 удаляет 0 строк**: ряды свечей уже очистила `a858647f24e6`. На данных
>   v1.0.0 (без A-01) удалила бы 39 фантомов SBER (2014-05-01 … 2018-02-23) и 17 дневок T-Invest SBER (07–30.03.2018).
>
> **Замер миграции** (копии рабочей БД через `sqlite3.Connection.backup` из `mode=ro`; рабочая БД на запись не открывалась):
>
> | Шаг | Время, с | Размер файла, байт |
> |---|---|---|
> | сборка 03.11: A-02 отдельным процессом (копия на `v1.0.0`, после A-01) | 1,03 (≈ старт alembic) | 114 696 192 → без изменений |
> | сборка 03.11: вся цепочка от `v1.0.0` одним `upgrade head` (тогда — A-01 + A-02) | 1,49 | — |
> | стенд 06.11: цепочка до головы Б-01 / затем A-02 отдельным процессом | 1,99 / 1,15 | — (data-часть — 0 строк) |
>
> Файл после миграции не меняется; таблицы A-02 пустые; `broker_accounts` — признак ни у одного счёта.
>
> **Откат** — на коде с этой ревизией, до возврата на прежний тег: `alembic downgrade b01e7a5c3d92` (в коде прежнего тега
> ревизии `79f3c238dbb8` нет, `alembic downgrade -1` там её не найдёт). Снимает индекс и колонку счёта и удаляет пять таблиц:
> историю T-Invest (перезагрузится ночью), строки Смартлаба (перезагрузятся за неделю), выбор счёта (запись аудита остаётся)
> и **ручные поправки дивидендов и соответствия тикеров — их можно вернуть только из копии БД**. Если они уже внесены —
> перед откатом копия БД через `sqlite3.Connection.backup` (§6.1, gotcha-19). Удалённые data-частью свечи не возвращаются.
> Свечи, загруженные фоном A-02, остаются в БД; код прежнего тега читает их только без T-Invest, а при первом запросе
> пользователя с T-Invest по бумаге удаляет ISS-строки её ряда целиком (`_purge_iss_cache` до A-02 чистит без границы по
> дате) — по коду, вживую не проверялось; файл БД при этом не уменьшается. Откат блоков ниже по цепочке (`alembic downgrade
> 0acf47e6607f` для Б-01 и далее) проходит через эту ревизию. `upgrade` → `downgrade b01e7a5c3d92` и вся цепочка туда и
> обратно покрыты тестами.
>
> **⚠️ Рост БД при первом запуске.** Сразу после старта (задание `history_backfill`, `next_run_time` = старт) идёт фоновая
> догрузка по бумагам сессий, бэктестов, активных алертов, избранного и целей A-01: минуты ISS до шва с 2011 года, дневки ISS
> окна, а со счётом для рыночных данных — дневки T-Invest с 2000 года и **минуты T-Invest с 13.06.2018** (раньше — с глубины
> кэша, у SBER с 2023-01-01). Тестовый стенд 06.11 (копия рабочей БД, 7 бумаг: GAZP, LKOH, MTSS, ROSN, SBER, T, ZZZZ):
>
> | Что | Итог |
> |---|---|
> | Время | `history_backfill_started` 16:43:51 → `history_backfill_done` 18:49:02 МСК — **2 ч 5 мин**, ошибок нет |
> | Файл БД | **114 696 192 → 1 877 041 152 Б (114,7 → 1 877 МБ, ×16,4; +1,64 ГиБ)** |
> | Строки | ISS 1m до шва — ≈ 4,08 млн (0,7–0,85 млн на бумагу, 79 месяцев); T-Invest 1m с 13.06.2018 — ≈ 7,56 млн (1,6–1,7 млн на бумагу; у T — 0,83 млн с 28.10.2019); дилерские минуты — 0,98 млн; дневки ISS окна — 68 на бумагу |
> | На бумагу | ≈ 25 мин (ISS 1m — 14–18 мин, ≈ 14 с на месяц; T-Invest — 8–9 мин, из них ≈ 6–7 мин — дни архива 2019–2021 обычными запросами) и ≈ 330 МБ |
>
> На стенде T-Invest-часть GAZP не шла: счёт выбран в 16:45, через 2 мин после старта фона, а GAZP — первая в очереди.
> В ночном прогоне 07.11 (04:00 МСК) T-Invest-часть GAZP прошла за 11 мин: 5 259 дневок и 1,72 млн минут, файл БД
> стенда — 2,17 ГБ (+289 МБ); ночи 08 и 09.11 — 7 бумаг за 1 мин, +2,4 МБ за две ночи.
> **Рабочая БД** — та же вселенная (стенд — её копия), поэтому ожидать **≈ 2 ч 16 мин и ≈ 2,2 ГБ** (стенд: 2 ч 5 мин + T-Invest-часть
> GAZP 11 мин; файл — 2,17 ГБ); каждая бумага, добавленная в сессии, бэктесты, алерты или избранное после 06.11, — ещё
> ≈ 25 мин и ≈ 330 МБ. Дальше каждую ночь догружается только новое. Файл после фона не уменьшается.
>
> **Свободное место — до выкатки:** (1) рост файла БД ≈ 2 ГБ; (2) WAL во время записи — не замерялся (записи идут короткими
> транзакциями по 2000 строк); (3) **копии `backup_job`**: по умолчанию хранятся 7 последних (`BACKUP_KEEP_LAST`), каждая —
> размером с БД, — через неделю ≈ 7 × 2,2 ≈ **15 ГБ** вместо ≈ 0,8 ГБ (по умолчанию `BACKUP_CRON` — 03:00 UTC = 06:00 МСК, после
> ночных заданий A-02; время копии и `integrity_check` на 2 ГБ не замерялось); (4) копия перед обновлением (≈ 115 МБ). Итого —
> **не меньше ≈ 20 ГБ свободного места** на томе с БД и каталогом копий (`BACKUP_DIR`); минимум диска
> терминала — 30 ГБ (ФТ §17.6; решение 2026-11-09); `BACKUP_KEEP_LAST` остаётся 7 — сокращать его или выносить копии, только если места не хватает. Тестовый стенд после выкатки будет копировать уже большую рабочую
> БД: `staging.sh reset` дольше, `staging-data/` — плюс ≈ 2 ГБ.
>
> **Что видно в журнале во время догрузки** (время записей — пояс машины; на стенде CET = МСК − 2 ч):
> * `history_backfill_started tickers=N` — начало прогона (после старта и каждый день в 04:00 МСК);
> * без счёта для рыночных данных — один раз за прогон warning `market_data_account_missing ticker=<бумага>`: T-Invest-часть
>   пропускается, пока счёт не выбран (бумаги после выбора её получают);
> * T-Invest-часть бумаги: `bar_loader_done` и `backfill_target_done ticker=… timeframe=D|1m windows=… loaded=… failed=0`;
>   info `daily_phantom_skipped` (дневки T-Invest до 2019 года в дни без торгов — норма); warning `archive_days_via_getcandles`
>   с длинными списками дат (2019–2021 — почти целые годы; правило A-01, ряд верный, только дольше);
> * ISS-часть: `iss_history_loaded ticker=… timeframe=D loaded=1` и `timeframe=1m loaded=79 failed=0`; единичные
>   `iss_candle_invalid_skipped` — невалидные минуты ISS (на стенде 01.12.2017 12:04 МСК у LKOH, MTSS, ROSN, SBER; 27.02.2015
>   15:00 у SBER и ROSN) — норма, ADR 0030;
> * бумага, которой нет у брокера, — `backfill_target_skipped … не найден у брокера T-Invest` (на стенде ZZZZ) — штатно;
> * у бумаги без минут ISS до шва (на бирже позже 2018: T — с 28.10.2019) ISS-часть — секунды;
> * в конце — `backfill_volume_checked` по бумагам и **`history_backfill_done`**.
> * **Проблема:** `iss_history_window_failed`, `history_backfill_ticker_failed`, `backfill_target_failed`,
>   `backfill_rate_limited` (лимит T-Invest — T-Invest-часть остальных бумаг ждёт следующего прогона),
>   `history_backfill_job_failed`, `Traceback`. На стенде ничего из этого не было.
>
> **Как понять, что догрузка закончилась:** в журнале `history_backfill_done` после `history_backfill_started` (более поздние
> `history_backfill_done` без `started` — штатно: так заканчивается догрузка бумаги, поставленной запросом графика или
> бэктеста); размер файла БД перестал расти; запросом только на чтение (`sqlite3 -readonly`)
> `SELECT ticker, count(*) FROM ohlcv_cache WHERE timeframe='1m' AND source='moex_iss' GROUP BY ticker` — 0,7–0,85 млн у
> бумаг с историей с 2011 года; `GET /api/v1/market-data/candles?ticker=SBER&timeframe=1m&from=2018-06-07T00:00:00Z&to=2018-06-14T00:00:00Z`
> → `history_pending: false`, 07, 08, 11.06 — по 526 минут, 09.06 — 522, 13.06 — 526.
>
> **Счёт для рыночных данных (A8) — сразу после выкатки** (ФТ §7.7):
> * до выбора фон ходит только в ISS, дивиденды T-Invest не грузятся, на «Дивидендах» нет колонки T-Invest и расхождений,
>   дневной график и бэктест глубже года по бумаге без загруженной истории — неполные с пометкой `no_market_data_account`;
> * выбрать: войти администратором → «Настройки → Брокер» → колонка «Рыночные данные» → переключатель у боевого счёта
>   T-Invest. Песочницу выбрать нельзя (422 «Выбрать можно только production-счёт…»). На стенде выбран #2 (production «Только
>   для чтения»); на рабочем терминале — **решить до выкатки** (какой счёт — вопрос заказчику). Выбор — в аудит `broker.market_data_account_selected`;
>   ключ этого счёта сам, по расписанию, ходит к брокеру (свечи ночью, дивиденды по всем акциям TQBR — ≈ 5 мин, не больше
>   150 запросов в минуту из 200) и тратит общий лимит T-Invest;
> * **порядок:** фон стартует вместе с процессом, а бумаги, пройденные до выбора счёта, получат T-Invest-часть только в
>   ночном прогоне 04:00 МСК или по первому запросу графика или бэктеста по бумаге (на стенде — GAZP). Выбрать счёт сразу
>   после входа; если нужно раньше ночи и торговых сессий нет — перезапуск backend: прогон начнётся заново и догрузит только
>   недостающее;
> * дивиденды T-Invest — в первую ночь после выбора, 04:30 МСК; сверка со Смартлабом — с 05:00 МСК, седьмая часть акций за
>   ночь (38 из 261), полный круг — неделя.
>
> **Новые фоновые задания** (ТЗ §5.9): `history_backfill` — 04:00 МСК и сразу после старта; `load_tinvest_dividends` —
> 04:30; `check_smartlab_dividends` — 05:00. В `scheduler_started` на три задания больше (по коду).
> Журнал дивидендов: `dividends_tinvest_loaded tickers=… failed=…`,
> `market_data_account_missing job=load_tinvest_dividends`, `dividends_figi_unresolved`, `dividends_tinvest_key_rejected`, `dividends_tinvest_rate_limited` (прогон встал по лимиту — остальное
> следующей ночью, первыми);
> сверка: `smartlab_check_done checked=… unavailable=… due=…`, `smartlab_check_unavailable ticker=… reason=…`.
>
> **Сетевой доступ.** К ISS сверх прежнего — минуты и дневная история бумаг вселенной (страницы по одной; ночью — только
> недостающее) и список бумаг TQBR (две ночные задачи). К T-Invest — `GetDividends` по всем акциям TQBR раз в сутки и история
> свечей ночью, счётом для рыночных данных. К `smart-lab.ru` — HTTPS, страница `/q/<тикер>/dividend/`, седьмая часть акций
> TQBR за ночь (38), пауза 5 с между страницами; прокси — из окружения, ничего не обходится (с машины разработки Смартлаб
> открывается напрямую — проверено 2026-11-03; через прокси не проверялось). Лимит «страница на бумагу в неделю» каждый
> терминал считает сам: стенд и рабочий терминал вместе — до двух страниц на бумагу в неделю.
>
> **Настройки** (§3.2): новых переменных окружения блок не вводит; окна, лимиты и пороги — константы кода
> (`app/market_data/seam.py`, `app/market_data/service.py`, `app/dividends/service.py`).
>
> **Что увидит пользователь после выкатки** (ФТ §11.4, §21): пункт «Дивиденды» сразу после «Счёта»; на графике и в бэктесте —
> минуты до 13.06.2018 после фона (пока фон идёт, бэктест длинного периода — с жёлтым «История за период догружается…»);
> у администратора — «Выберите счёт для рыночных данных» в «Настройки → Брокер» до выбора; в админке — карточка «Соответствия
> тикеров».
>
> **Итог тестового стенда** — PASS WITH NOTES (2026-11-09; кандидат `104b9e2` 06–09.11, с 09.11 — `088b9e6`; CI
> PR #121 — 4 из 4). Кроме фона и шва (выше) вживую проверены: дивиденды T-Invest — 261 акция TQBR за 5 мин 7 с без
> отказов по лимиту (`088b9e6`), разбор настоящих страниц Смартлаба, поправки (GAZP «Исключить», AQUA «Добавить»,
> AVAN «Исключить» — пережили перезагрузки T-Invest), бэктест SBER Д 2017–2019 без ложных «пропусков» (`acbea74`).
> Замечания на выкатку:
> * приложение хранит 7 полных копий БД — на диске нужно ≈ ×8 размера базы (выше, «Свободное место»);
> * сверка Смартлаба не учитывает дробление акций (BELU 1:8, 2024 — 11 ложных расхождений);
> * узкая гонка остановки: если команда остановки придёт в доли секунды, пока прогон собирает список бумаг, фоновая
>   загрузка не остановится (записано в `PROGRESS.md` блока);
> * сдвиг времени в бэктесте — отдельная ветка `fix/backtest-naive-timestamps`, не A-02;
> * **до выкатки** — решить, какой счёт будет «счётом для рыночных данных» на рабочем терминале (выше).
>
> **Не проверено вживую:** PHOR на Смартлабе (на стенде сверка дойдёт 14.11; 201 ₽ принят по фикстуре Смартлаба и
> данным T-Invest стенда — решение заказчика 09.11); сверка Смартлаба через прокси; время `backup_job` и
> `integrity_check` на БД ≈ 2 ГБ; рост WAL во время первого фона.
>
> **Проверка** (запросы к БД — только на чтение, `sqlite3 -readonly`; время в БД — UTC):
> * `alembic current` → `79f3c238dbb8`; `SELECT count(*) FROM sqlite_master WHERE type='table' AND name IN ('ticker_aliases',
>   'dividend_payments','dividend_corrections','dividend_smartlab_rows','dividend_sources')` → 5;
>   `SELECT count(*) FROM broker_accounts WHERE is_market_data_source = 1` → 0 до выбора, 1 после;
> * журнал: `scheduler_started` с `history_backfill`, `load_tinvest_dividends`, `check_smartlab_dividends`; сразу после
>   старта — `history_backfill_started`; дальше — как в «Что видно в журнале»;
> * после выбора счёта: `SELECT action, entity_id, details FROM audit_log WHERE action='broker.market_data_account_selected'`
>   — одна строка с `previous_id: null`;
> * после первой ночи: `SELECT count(*), count(DISTINCT ticker) FROM dividend_payments`; `SELECT smartlab_status, count(*)
>   FROM dividend_sources GROUP BY 1`; страница «Дивиденды» — список бумаг, у сверенных — расхождения;
> * `GET /api/v1/dividends` без входа → 401.

> **ℹ️ Блок Б-01 «Новости: сборщик, архив, лента» — ревизия `b01e7a5c3d92` (поверх `0acf47e6607f`, 2026-11-06).**
> `alembic current` после обновления: `b01e7a5c3d92` (head до A-02; с A-02 head — `79f3c238dbb8`, выше). С `v1.0.0` (`a7c1d9e2b4f8`) `alembic upgrade head` проходит
> пять ревизий A-01, ревизии A-03, A-05, Б-02 и эту одним запуском (при ветвлении блока родителем была `a7c1d9e2b4f8`; при
> мержах с `develop` ревизия перевешена на `cdb15f21e5ca`, затем на `0acf47e6607f`; после тестового стенда в посев добавлен
> псевдоним «GloraX» → `GLRX`, остальное содержимое не менялось). Ревизия одна на весь блок и только добавляет —
> существующие таблицы не трогает:
> * шесть таблиц архива новостей (ТЗ §3.26): `news_sources` — источники и состояние опросов; `news_items` — заголовок,
>   ссылка, время публикации и время получения (текста статьи нет); `news_labels` и `news_tickers` — разметка словарём и
>   бумаги (поля ИИ-разметки пусты до Б-07); `news_reactions` — реакция «до / после»; `news_dictionary_terms` — псевдонимы
>   и исключения словаря;
> * посев: 7 строк `news_sources` (Интерфакс, Прайм, ТАСС «Экономика и бизнес», Коммерсантъ «Бизнес», Смартлаб, сообщения
>   биржи ISS, Банк России; адреса лент живут здесь, а не в коде) и 217 строк `news_dictionary_terms` (187 псевдонимов,
>   30 исключений).
>
> `downgrade` удаляет все шесть таблиц **вместе с накопленным архивом** — новостями, разметкой, привязками и реакциями.
> Восстановить архив нельзя: ленты держат новости часы и сутки, прошлое у них не скачать; повторный `upgrade` начнёт архив
> с нуля. Если архив нужен — перед откатом копия БД через `sqlite3.Connection.backup` (§6.1, gotcha-19). Откат делать на
> коде с этой ревизией — `alembic downgrade 0acf47e6607f` до возврата на прежний тег (в коде прежнего тега ревизии
> `b01e7a5c3d92` нет, и `alembic downgrade -1` там её не найдёт). Откат блоков ниже по цепочке (`alembic downgrade
> cdb15f21e5ca` для Б-02 и далее) проходит через эту ревизию и тоже удаляет архив новостей. С A-02 откат Б-01 (`alembic downgrade 0acf47e6607f`) сначала проходит ревизию A-02 `79f3c238dbb8` — последствия в блоке A-02 выше. `upgrade` → `downgrade`
> ревизии и вся цепочка туда и обратно покрыты тестами.
>
> **Замер на копии рабочей БД** (2026-10-25, сборка блока; копия через `sqlite3.Connection.backup` из `mode=ro`, рабочая
> БД на запись не открывалась; копия стояла на `a7c1d9e2b4f8` — тогдашнем родителе ревизии; после перевешивания не
> перемерялось, но на тестовом стенде 05.11 цепочка `… → cdb15f21e5ca → 0acf47e6607f → b01e7a5c3d92` применилась к копии
> рабочей БД без ошибок):
>
> | Шаг | Время, с | Размер файла, байт |
> |---|---|---|
> | **Б-01 `upgrade` (`b01e7a5c3d92`)**, процесс целиком | **1,00** (сама ревизия ≈ 0,1; остальное — запуск alembic) | 114 638 848 → 114 720 768 (+81 920, 20 страниц) |
>
> **Объём БД.** Архив не очищается: срока хранения в блоке нет (ADR 0101) — его стоит решить до года работы. Сутки
> тестового стенда (05.11 15:07 → 06.11 15:07 МСК, копия рабочей БД): ≈ 950 новостей (оценка сборки была 300–500),
> объекты `news_*` (`dbstat`) выросли с 311 296 до 757 760 Б — **≈ 508 Б на новость вместе с разметкой, привязками,
> реакциями и индексами, ≈ 0,48 МБ в сутки, ≈ 15 МБ в месяц, ≈ 175 МБ в год**. На фоне свечей это немного (файл БД стенда —
> 489 МБ, почти всё — свечи A-01; новости — 0,16 %), но место на диске и срок хранения стоит учесть.
>
> **Новые фоновые задания** (ТЗ §5.9): `news_collect` (сбор и разметка) и `news_reactions` (реакции) — каждые 60 с; первый
> проход — через минуту после старта (на стенде — через 73 с), в `scheduler_started` на два задания больше (на стенде
> `a4e320c` — 12). Первый проход забирает всё, что в этот момент лежит в лентах (на стенде — 383 новости из шести
> источников), и сразу размечает; реакции по накопившимся парам считаются по 20 в минуту (72 пары — за 4 прохода).
> Перезапуск ничего не теряет и дублей не создаёт: срок реакции хранится в БД (проверено перезапуском стенда).
>
> **Сетевой доступ к источникам.** Backend ходит по HTTPS к `www.interfax.ru`, `1prime.ru`, `tass.ru`, `www.kommersant.ru`,
> `smart-lab.ru`, `iss.moex.com` и `www.cbr.ru` — с подписью `User-Agent: MoexTerminal-NewsCollector/1.0`, раз в минуту на
> источник; страницы статей не скачиваются. К ISS сверх прежнего — список акций TQBR (раз в сутки и после каждого старта) и
> минутные свечи бумаг для реакций (до 20 реакций за проход). Транспорт не подменяется — действуют прокси окружения
> процесса backend (`HTTP_PROXY` / `HTTPS_PROXY` / `NO_PROXY`). Разовые сбои лент следующий проход восстанавливает без
> пропусков (за сутки стенда: Смартлаб 19:46–21:08 — таймауты и `HTTP 504` на стороне источника; Коммерсантъ — `HTTP 500` и
> `ProxyError: 502`; дважды сеть машины — у всех источников сразу).
> * **Банк России через прокси окружения не отвечает.** На тестовом стенде — `ReadTimeout` на каждом проходе: 1446
>   неудачных попыток подряд за сутки; `curl https://www.cbr.ru/rss/eventrss` через прокси — `000` за 16,5 с, напрямую —
>   `200` за 0,15 с. Адрес верный, разбор настоящей ленты ЦБ покрыт тестом; остальные шесть источников через тот же прокси
>   отвечают.
> * **Рекомендация:** добавить `www.cbr.ru` в `NO_PROXY` окружения процесса backend (к уже указанным там хостам, через
>   запятую). Это решение заказчика при выкатке; код не меняется. `~/MoexTerminal/data/stand.sh` прокси не задаёт и не
>   снимает: backend рабочего терминала получает их из окружения, в котором запущен `stand.sh`, — без исключения ЦБ будет
>   падать так же, как на стенде.
> * **Пока не добавлен:** каждый проход ждёт ответа ЦБ ≈ 15 с (таймаут чтения); новости остальных источников записываются
>   после этого, и их время получения сдвинуто на те же ≈ 15 с (норматив «лента не позже 2 мин» звено терминала
>   выдерживает: ≤ 83 с, ФТ §20.6). В журнале каждую минуту warning `news_source_failed source=cbr error=ReadTimeout`
>   (≈ 1440 строк в сутки), у строки `cbr` в `news_sources` растёт `consecutive_failures`; новостей ЦБ в архиве нет.
>
> **Настройки** (§3.2): новых переменных окружения блок не вводит; интервалы и пороги — константы кода
> (`app/news/constants.py`).
>
> **Что увидит пользователь после выкатки** (ФТ §20): пункт меню «Новости» сразу после «Графиков» — лента, общая для всех
> пользователей: время (МСК), источник, заголовок-ссылка, бумаги, «прочее / словарь», реакция по каждой бумаге — «посчитается
> в ЧЧ:ММ», через 1 ч 15 мин после публикации — «до …, после …» или «нет торгов»; фильтры «Бумага», «Источник», «Тип»,
> «Показать ещё», обновление раз в минуту. В первые минуты лента наполняется тем, что лежит в лентах источников, — раньше
> этого архив не идёт.
>
> **Не проверено вживую:** реакции на новости торгов выходного дня — проверка новостей 07–08.11 с 09.11 на стенде A-02
> (`s8r-evidence/008/Б-01/2026-11-05-stand-a4e320c.md` §9.3).
>
> **Проверка** (запросы к БД — только на чтение, `sqlite3 -readonly`; время в БД — UTC, на странице — МСК):
> * `alembic current` → `b01e7a5c3d92` (с A-02 — `79f3c238dbb8`); `SELECT count(*) FROM news_sources` → 7; `SELECT count(*) FROM
>   news_dictionary_terms` → 217;
> * журнал backend: `scheduler_started` с заданиями `news_collect` и `news_reactions`; через 1–2 мин — `news_source_collected`
>   по источникам; нет `news_collect_cycle_failed`, `news_labeling_failed`, `news_reaction_cycle_failed`,
>   `news_source_store_failed` (`news_source_failed source=cbr` — см. выше);
> * `SELECT code, last_polled_at, last_success_at, consecutive_failures FROM news_sources` дважды с паузой 70 с —
>   `last_polled_at` сдвинулся у всех семи, `consecutive_failures` = 0 (кроме `cbr`, пока он не в `NO_PROXY`);
> * `SELECT count(*) FROM news_items` через 2 мин — первый проход: ≈ 320–390 без ЦБ, с ЦБ — ещё ≈ 100; дублей нет —
>   `SELECT count(*) FROM (SELECT source_id, external_id FROM news_items GROUP BY 1, 2 HAVING count(*) > 1)` → 0; новостей без
>   разметки словаря — 0 (или единицы последней минуты);
> * страница «Новости»: строки есть, фильтры сужают ленту, «Показать ещё» добавляет 50; через 1 ч 15 мин после привязанной
>   новости в часы торгов — реакция в процентах; `SELECT status, count(*) FROM news_reactions GROUP BY status` — строк
>   `failed` нет;
> * `GET /api/v1/news` без входа → 401;
> * после решения по ЦБ: у `cbr` `consecutive_failures` = 0, `last_success_at` заполнен, в ленте есть источник «Банк России».

> **ℹ️ Блок Б-02 «Графики: панели и виды» — ревизия `0acf47e6607f` (поверх `cdb15f21e5ca`, 2026-11-05).**
> `alembic current` после обновления: `0acf47e6607f` (head до Б-01; с Б-01 head — `b01e7a5c3d92`, с A-02 — `79f3c238dbb8`, выше). С `v1.0.0` (`a7c1d9e2b4f8`) `alembic upgrade head` проходит
> пять ревизий A-01, ревизии A-03, A-05 и эту одним запуском (при ветвлении блока родителем была `a858647f24e6`; при
> мерже с `develop` ревизия перевешена на `cdb15f21e5ca`, содержимое не менялось). Ревизия одна на весь блок:
> * создаёт таблицу `chart_layouts` — раскладка графиков, один JSON-документ на пользователя (`user_id` — FK на `users`
>   с `ON DELETE CASCADE` и уникальностью `uq_chart_layouts_user`, `layout_json`, `created_at`, `updated_at`; ТЗ §3.25).
>   Таблица пустая, строка появляется при первом изменении раскладки пользователем; создаётся идемпотентно;
> * в `chart_drawings` заменяет индекс `idx_drawing_lookup (user_id, ticker, timeframe)` на
>   `idx_drawing_user_ticker (user_id, ticker)` (`batch_alter_table`, только операции с индексом — таблица не
>   пересоздаётся). **Строки рисунков не переписываются**: таймфрейм уже отдельная колонка, регистр тикера сравнивается
>   через `upper()`. В журнал миграции — строка `Б-02 chart_drawings: рисунков N, ключей «пользователь + бумага» M`.
>
> `downgrade` возвращает прежний индекс и **удаляет `chart_layouts` вместе со всеми раскладками** (после повторного
> обновления пользователи начнут с раскладки по умолчанию); рисунки остаются. Откат делать на коде с этой ревизией —
> `alembic downgrade cdb15f21e5ca` до возврата на прежний тег (в коде прежнего тега ревизии `0acf47e6607f` нет, и
> `alembic downgrade -1` там её не найдёт). Перед обновлением — копия БД через `sqlite3.Connection.backup` (§6.1, gotcha-19).
>
> **Замер на копии рабочей БД** (2026-10-26, копия через `sqlite3.Connection.backup` из `mode=ro`, рабочая БД на запись
> не открывалась; копия стояла на `a7c1d9e2b4f8` — сначала применена цепочка A-01 до `a858647f24e6`, тогдашнего
> родителя, 3,81 с; время включает старт alembic с импортом приложения ≈ 3 с; после перевешивания на `cdb15f21e5ca` не
> перемерялось — содержимое ревизии то же):
>
> | Шаг | Время, с | Размер файла, байт |
> |---|---|---|
> | **Б-02 `upgrade` (`0acf47e6607f`)**, процесс целиком | **3,31** (сама ревизия в одном процессе — 0,02–0,08) | 114 642 944 → 114 642 944 |
> | **Б-02 `downgrade`** / повторный `upgrade` | **3,51** / 3,30 | — |
>
> Рисунков на копии — 3 (один пользователь, одна бумага: `1h` — 2, `5m` — 1), ключей «пользователь + бумага» — 1
> (прежних «пользователь + бумага + ТФ» — 2); бумаг в двух написаниях регистра — 0. Отпечаток всех строк
> `chart_drawings` до обновления, после него и после `downgrade` + `upgrade` совпадает: перенесено на новый ключ 3 рисунка
> из 3 (перенос — смена индекса). `chart_layouts` — пустая. План запроса рисунков — `USING INDEX idx_drawing_user_ticker
> (user_id=?)`; как и раньше, работает только префикс `user_id` (тикер сравнивается через `upper()`).
>
> **Раскладка — на сервере, переносить нечего.** Раскладки до блока не было (один график, таймфрейм при каждом открытии —
> `D`), в localStorage она не хранилась и не хранится. Первое открытие графика после выкатки — раскладка по умолчанию:
> одна панель `D` с бумагой из адреса; первое изменение пишет строку в `chart_layouts` (`PUT /api/v1/chart-layout`
> через 0,5 с после правки).
>
> **Рисунки в браузере.** Ключи localStorage «бумага + ТФ» (`drawings:{user}:{ticker}:{tf}`, версия 1) переносятся в
> ключ «бумага» (`drawings:{user}:{ticker}`, версия 2, таймфрейм — поле рисунка) один раз при входе пользователя, в каждом
> браузере отдельно: рисунок с уже перенесённым id не задваивается, старый ключ удаляется только после записи нового.
> Кэш свечей `candlesCache` прежний (только основной ряд «с учётом дроблений»). **При откате на прежний тег** рисунки с
> сервера видны, как раньше, а рисунки, жившие только в localStorage (не дошедшие до сервера), прежний фронтенд не
> увидит: он читает ключи версии 1, а они уже перенесены (следует из кода `utils/drawingsPersistence.ts`, не проверялось).
>
> **Фронтенд и backend — только вместе.** Прежние маршруты рисунков `/api/v1/charts/{ticker}/{tf}/drawings` сняты без
> переходного периода (ADR 0026): вкладка с прежней сборкой SPA, открытая до выкатки, не загрузит и не сохранит рисунки —
> её нужно обновить (кэш SPA — §9.6).
>
> **Что увидит пользователь после выкатки** (по копии рабочей БД и тестовому стенду 30.10–05.11):
> * страница графика — общая полоса с раскладкой (1 / 2 рядом / 2 друг под другом / 2×2), «Связь символа» (выкл.) и
>   «Перекрестие по времени» (вкл.); у каждой панели заголовок — бумага, таймфрейм с избранными кнопками, вид, шкала
>   «Об / Лог / %», значки «Внебиржевые часы» и «Как было»; на узком экране — одна панель и вкладки (ФТ §5.1, §5.4);
> * на внутридневных таймфреймах видно утро 07:00–10:00 МСК (фильтр часов снят); свеча Н/М больше не сжимается на
>   ~1,6 с при открытии канала (ФТ §5.5);
> * у пользователя без T-Invest «MOEX ISS · задержка 15 мин» — у каждой панели, под легендой OHLC (на 2×2 и «2 рядом»
>   вторую строку легенды не закрывает);
> * три рисунка SBER на месте, на своих таймфреймах; предупреждение «N рисунок(ов) в старом формате» теперь выходит при
>   открытии SBER на любом таймфрейме (раньше — только на 1ч) — не чинится, уходит в Б-04 вместе со старыми рисунками
>   (E10; нужные линии заказчику перерисовать до выкатки Б-04);
> * «Внебиржевые часы» выключены у всех панелей; включаются на 1м–4ч при подключённом T-Invest.
>
> **Настройки** (§3.2): новых переменных окружения блок не вводит. Дилерская подписка графика работает при
> `MARKET_DATA_STREAM_MODE=single_include_weekend` (умолчание с A-03); при `two_subscriptions` сервер отвечает
> `channel: null`, и «Внебиржевые часы» показывают только историю дилерских свечей.
>
> **Известные мелочи** (ФТ §5.4): правка раскладки меньше чем за 0,5 с до перезагрузки или закрытия вкладки на сервер не
> уходит; иногда уходит лишний одинаковый `PUT /api/v1/chart-layout` (на сервере — последнее состояние).
>
> **Не проверено вживую:** «Внебиржевые часы» с живым каналом `dealer:` — в будни дилерских часов нет; заказчик влил блок
> без этой проверки (2026-11-05: «вливать, но на выходных проверить»). Разовая проверка на тестовом стенде 07–08.11 (дилер —
> 02:00–09:40 и 19:00–23:50 МСК; задачи `b02-dealer-hours-sat` 07.11 19:30 МСК и запасная `b02-dealer-hours-sun`, отчёт
> `s8r-evidence/008/Б-02/weekend_2026-11-07.md`): включение — ровно один `POST /candles/subscribe?…&source=dealer` с
> каналом `dealer:SBER:1m` в ответе, второй подписки к T-Invest нет; дилерские свечи — второй серией между барами основного
> ряда; живая дилерская минута растёт и закрывается; перекрестие, рисунки и маркеры — по основному ряду; на Д/Н/М
> переключатель недоступен, выключение снимает подписку `dealer:`, после F5 состояние то же. Замечания — отдельным PR.
>
> **Проверка:** `alembic current` → `0acf47e6607f` (с Б-01 — `b01e7a5c3d92`, с A-02 — `79f3c238dbb8`); `SELECT name FROM sqlite_master WHERE type='index' AND
> tbl_name='chart_drawings'` — есть `idx_drawing_user_ticker`, нет `idx_drawing_lookup`; `SELECT count(*) FROM
> chart_drawings` — то же число, что до обновления; `GET /api/v1/chart-layout` под пользователем — `{"layout": null,
> "updated_at": null}`, после смены раскладки на странице графика — документ; 2×2 SBER 1м / 5м / 1ч / Д — в журнале
> backend четыре `stream_started` (по одному на «тикер:ТФ»), запросов свечей — четыре; к T-Invest на бумагу — одна
> подписка обновлений 1м (плюс закрытые минуты A-01), пятая такая же панель ни запроса, ни подписки не добавляет.

> **ℹ️ Блок A-05 «Дефекты этапа 1» — ревизия `cdb15f21e5ca` (поверх `d92b7dd5c939`, 2026-11-04).**
> `alembic current` после обновления: `cdb15f21e5ca` (head до Б-02; с Б-02 head — `0acf47e6607f`, с Б-01 — `b01e7a5c3d92`, с A-02 — `79f3c238dbb8`, выше). С `v1.0.0` (`a7c1d9e2b4f8`) `alembic upgrade head` проходит
> пять ревизий A-01, ревизию A-03 и эту одним запуском (при ветвлении блока родителем была `a7c1d9e2b4f8`; при мерже с
> `develop` ревизия перевешена на `d92b7dd5c939`, содержимое не менялось). Ревизия одна на весь блок и добавляет три
> **nullable-колонки без `server_default`**, существующие строки не меняет:
> * `strategy_versions.parse_report` (Text) — отчёт разбора описания версии (JSON); NULL — отчёт не составлялся
>   (версия до ревизии или текст не в формате ИИ-чата; в «Истории версий» — «Отчёт не составлялся»);
> * `backtests.lot_size` (Integer) — лот, которым посчитан бэктест; NULL — бэктест до ревизии: по бумаге с лотом в
>   справочнике больше 1 он показывается с пометкой «лоты штуками»;
> * `notifications.channels_failed` (String(100)) — CSV каналов, не доставленных после всех повторов; NULL — неудач нет.
>
> `upgrade` на SQLite — нативный `ADD COLUMN` через `batch_alter_table`. **`downgrade` обратим по схеме и пересоздаёт три
> таблицы** (`strategy_versions`, `backtests`, `notifications`; миграции SQLite идут с `PRAGMA foreign_keys=OFF`, каскада
> нет), строки сохраняются, **содержимое трёх колонок теряется**: после повторного обновления бэктесты, посчитанные в
> лотах, останутся без записанного лота и по бумагам с лотом больше 1 получат пометку «лоты штуками». Откат делать
> на коде с этой ревизией — `alembic downgrade d92b7dd5c939` до возврата на прежний тег (в коде прежнего тега ревизии
> `cdb15f21e5ca` нет, и `alembic downgrade -1` там её не найдёт). Перед обновлением — копия БД через
> `sqlite3.Connection.backup` (§6.1, gotcha-19).
>
> **Замер на копии рабочей БД** (2026-10-25, ревизия копии `a7c1d9e2b4f8` — тогдашний родитель; время включает старт
> alembic с импортом приложения):
>
> | Шаг | Время, с |
> |---|---|
> | **A-05 `upgrade` (`cdb15f21e5ca`)** | **1,0** |
> | **A-05 `downgrade`** | **1,1** |
>
> После отката строки целы и совпадают с числом до миграции: `notifications` — 2 300, `backtests` — 65, `backtest_trades` —
> 2 986, `strategy_versions` — 102.
>
> **Что увидит пользователь после выкатки** (по копии рабочей БД 2026-10-25; все 65 бэктестов посчитаны до ревизии,
> `lot_size` у них NULL):
> * **GAZP** — 10 бэктестов, лот 10 (сверен с T-Invest 2026-10-14) → пометка «лоты штуками» **сразу**: предупреждение в
>   карточке, значок в списке бэктестов и в таблице бэктестов стратегии, объём сделок «шт.», колонка экспорта «Акций»;
> * **MTSS** — 2 бэктеста, в справочнике лот 1, сверки не было → пометка появится **после сверки лота с T-Invest** (лот 10):
>   при первом бэктесте или открытии карточки MTSS пользователем с боевым T-Invest;
> * **T** — 6 бэктестов, лот 1, сверки не было → пометка, только если после сверки лот окажется больше 1;
> * SBER (34) и LKOH (13) — лот 1, без пометки.
>
> Старые бэктесты **не пересчитываются** (пересчёт — вне блока): чтобы посчитать в лотах, их запускают заново — новый
> бэктест хранит лот и считает «N лотов» = N × лот акций, «%» — целое число лотов с комиссией. Пересчёт GAZP и MTSS на
> стенде и в рабочем терминале запускает заказчик.
>
> **Повтор доставки Telegram и почты** (ФТ §10.1): после старта задача `retry_notification_delivery` раз в 10 с (виден в
> `/api/v1/health` → `scheduler_jobs`). В `pending_events` копии — 19 строк, все уже обработаны, строк `notification.dispatch`
> нет: первый проход после выкатки **ничего не пометит** «не доставлено», отметки появятся только у новых уведомлений.
> Журнал: `notification_delivery_retry_scheduled` (попытка не удалась, назначен повтор), `notification_delivery_exhausted`
> (пять неудач — отметка), `notification_delivery_expired` (строка старше 1 ч — отметка без отправки),
> `notification_retry_pass rows=N`. Повтор работает только при `DEV_MODE=false` (в DEV_MODE строка закрывается без
> отправки и без отметки) и для каналов с получателем (привязка Telegram, email пользователя). На тестовом стенде
> Telegram и почта выключены — **вживую повтор проверяется после выкатки** (или на стенде тестовым ботом: токен
> тестового бота, `DEV_MODE=false`, привязка чата, событие — короткий бэктест или алерт, сломанный канал; канал вернули —
> сообщение приходит один раз без рестарта, не вернули — после пятой неудачи в колокольчике «не доставлено в Telegram»).
>
> **Пометка ISS на графике:** у пользователя без активного счёта T-Invest график из ISS или кеша постоянно помечен
> «MOEX ISS · задержка 15 мин»; у пользователя с T-Invest — как раньше.
>
> **Настройки** (§3.2): новых переменных окружения блок не вводит.
>
> **Проверка:** `alembic current` → `cdb15f21e5ca` (с Б-02 — `0acf47e6607f`, с Б-01 — `b01e7a5c3d92`, с A-02 — `79f3c238dbb8`); `PRAGMA table_info(backtests)` содержит `lot_size`; карточка старого
> бэктеста GAZP — предупреждение «Лоты штуками»; новый бэктест GAZP «5 лотов» — объём сделок 5 лотов (50 акций).

> **ℹ️ Блок A-03 «Часы торгов, стопы внутри бара, дилер» — ревизия `d92b7dd5c939` (поверх `a858647f24e6`, 2026-11-04).**
> `alembic current` после обновления: `d92b7dd5c939` (head до A-05; с A-05 head — `cdb15f21e5ca`, с Б-02 — `0acf47e6607f`, с Б-01 — `b01e7a5c3d92`, с A-02 — `79f3c238dbb8`, выше). С `v1.0.0` (`a7c1d9e2b4f8`) `alembic upgrade head` проходит
> пять ревизий A-01 (ниже), эту и ревизию A-05 одним запуском. Ревизия одна на весь блок:
> * создаёт `trading_schedule_periods` — таблица периодов расписания: дата МСК, вид периода (`opening_auction`, `trading`,
>   `closing_auction`, `evening`, `dealer`), начало и конец в наивном UTC (конец исключён), источник `tinvest`/`iss`;
>   уникальность `(trade_date, kind, start_at)`, индекс по дате (ТЗ §3.24);
> * создаёт `trading_schedule_empty_days` — даты, на которые ISS ответил без минут: история их повторно не запрашивает;
> * создаёт `alerts` и `alert_triggers` — общая модель алертов (вид `level`; поля алертов по условиям A-11 — пустые) и
>   история срабатываний (ТЗ §3.13); **переносит активные строки `price_alerts`** в `alerts` видом `level` (уровень,
>   направление, FIGI, время создания — как были; взведены, цены создания нет), сработавшие и удалённые не переносит,
>   и **удаляет `price_alerts`**;
> * добавляет колонки `live_trades.exit_retry_kind / exit_retry_since / exit_retry_notified_at` (список «закрыть при
>   первой возможности») и `backtest_trades.price_note` («цена аукциона») — через `batch_alter_table` (таблицы
>   пересоздаются, строки и индексы сохраняются), новые колонки nullable и пусты.
>
> Таблицы и колонки создаются идемпотентно (через инспектор). **`downgrade` обратим по схеме:** снимает колонки и
> таблицы, создаёт `price_alerts` заново (колонки и индексы ревизии `14c59a1057bf`) с уровнями активных алертов `level`;
> строки сделок и бэктестов сохраняются; периоды расписания теряются (кэш перезаполнится), история срабатываний, цена
> создания и признак «взведён» алертов теряются — сработавшие алерты снова становятся активными (известный дефект).
> Перед обновлением — копия БД через `sqlite3.Connection.backup` (§6.1, gotcha-19).
>
> **Замер на копии рабочей БД** (2026-10-26, ревизия `a7c1d9e2b4f8`, 114,6 МБ; каждый шаг — отдельный `alembic upgrade`,
> время включает ≈ 1,2 с старта alembic с импортом приложения):
>
> | Шаг | Время, с | Файл БД, байт (до → после) |
> |---|---|---|
> | A-01, пять ревизий `a7c1d9e2b4f8 → a858647f24e6` | 2,04 | 114 638 848 → 114 642 944 |
> | **A-03 `a858647f24e6 → d92b7dd5c939`** | **1,28** (сама ревизия ≈ 0,05) | 114 642 944 → 114 642 944 |
>
> После `d92b7dd5c939`: `live_trades` (45 строк) и `backtest_trades` (2 986 строк) на месте, новые колонки пустые;
> `trading_schedule_periods`, `alerts`, `alert_triggers` пусты; `price_alerts` на рабочей БД — 0 строк (переносить
> нечего); `integrity_check` — ok. Файл не растёт: страницы берутся из freelist после A-01 (≈ 18,7 тыс. страниц).
>
> **После старта** фоном: `sync_trading_schedule` — T-Invest `TradingSchedules` на сегодня + 7 дней (сразу, затем в 00:10
> МСК, в выходные и праздники — ежечасно 01:00–23:00; токен — счёт рыночных данных первого активного пользователя;
> лог `trading_schedule_synced` с `days`, `first`, `last`) и `backfill_trading_schedule_iss` — история из минут ISS SBER
> с 13.06.2018. Первый полный проход — ≈ 2 900 запросов к ISS, **≈ 58–60 мин** (замер на копии: 0,83 дня/с, за 40 мин
> записано 1 500 дней с 2021-05-12 по 2026-10-23, 5 615 строк, сбоев 0); дальше — только непроверенные даты, в том
> числе после перезапуска (даты без минут отмечаются в `trading_schedule_empty_days`). До заполнения даты без строки
> отвечают запасные константы текущего расписания (warning `trading_schedule_fallback` раз в сутки) — это не ошибка.
> Нет счёта T-Invest или брокер не ответил — warning `trading_schedule_sync_no_account` / `trading_schedule_sync_failed`,
> таблица не меняется. **Проверка:** `alembic current` → `d92b7dd5c939` (с A-05 — `cdb15f21e5ca`, с Б-02 — `0acf47e6607f`, с Б-01 — `b01e7a5c3d92`, с A-02 — `79f3c238dbb8`); в журнале `trading_schedule_synced`;
> `SELECT source, count(*) FROM trading_schedule_periods GROUP BY source` — строки `tinvest` на ближайшие дни и растущее
> число `iss`; подвал терминала показывает статус по фазе («MOEX Аукцион закрытия — только выходы» в 18:55–19:00).
>
> **Настройки** (§3.2): умолчание `MARKET_DATA_STREAM_MODE` сменилось на `single_include_weekend` — строку
> `two_subscriptions`, если она есть в `.env.production`, удалить; с этим режимом в выходные растёт `ohlcv_dealer`
> (дилерские минуты потока). `DEALER_STOPS_ENABLED` не задавать — включает заказчик после своей ручной заявки у дилера.
> Умолчание окна часов сессии Circuit Breaker — 07:00–23:50 (сохранённые пользователем окна не меняются).
>
> **Тестовый стенд на копии рабочей БД:** `staging_copy.py prepare` снимает список «закрыть при первой возможности» у
> сделок песочницы и реального счёта, `check` не пускает `up`, пока такие пометки есть — иначе повтор раз в минуту
> отправил бы ордер на настоящий счёт.

> **ℹ️ Обновление с версии старше `b9d2f6a14c83` (блок A-01 «Хранилище баров», 2026-10-24).**
> Одна линейная цепочка из пяти ревизий от `a7c1d9e2b4f8` (S8R-FIX-013, снимает `daily_stats.peak_equity`); последняя
> ревизия блока — `a858647f24e6` (head до A-03; с A-03 head — `d92b7dd5c939`, с A-05 — `cdb15f21e5ca`, с Б-02 — `0acf47e6607f`, с Б-01 — `b01e7a5c3d92`, с A-02 — `79f3c238dbb8`, выше). `alembic upgrade head` проходит все пять за один запуск (entrypoint backend). В БД остаются только
> родные биржевые свечи `1m` и `D`, прочие таймфреймы собирает агрегатор (ТЗ §5.6.6), а свечи хранятся «как было» — в
> ценах и штуках своего дня, с пересчётом по дроблениям при чтении (ТЗ §3.5), поэтому кэш свечей T-Invest и ISS
> удаляется и перезагружается:
> * `b9d2f6a14c83` — создаёт `ohlcv_backfill_target` и засевает цели фоновой догрузки из прежнего кэша T-Invest (по бумаге:
>   `1m` с самого раннего внутридневного бара, но не раньше шва 2018-06-13; `D` с самого раннего дневного), затем удаляет
>   из `ohlcv_cache` все таймфреймы, кроме `1m` и `D` (5m/15m/1h/4h — кэш производный);
> * `e5a7c2d90b16` — удаляет строки T-Invest (объём в лотах, дилерские свечи выходных в основном ряду, недоформированные
>   бары — перезагрузка по новым правилам) и последний бар каждого оставшегося ряда ISS (мог быть записан незакрытым);
> * `c4e81f3a9d27` — удаляет индекс `idx_ohlcv_lookup` — дубль индекса уникальности (на полном кэше ≈ 19 МБ);
> * `f1b3d8e62a54` — создаёт `ohlcv_coverage`, `ohlcv_dealer`, `ohlcv_volume_check`; строк покрытия для лежавших свечей нет
>   (они дозагружаются по запросу);
> * `a858647f24e6` (дефект 4 тестового стенда, решение заказчика 2026-10-24) — создаёт `ohlcv_lot_history` (лот бумаги с
>   даты — для объёма минут архива `history-data`, ТЗ §3.5.5; пустая, заполняется фоновой догрузкой) и удаляет все ряды
>   свечей прежнего масштаба — `ohlcv_cache` целиком, `ohlcv_dealer`, `ohlcv_coverage`, `ohlcv_volume_check`. Строки ISS:
>   ISS пересчитывает цену по дроблениям, наступившим к запросу, и у строк, записанных до A-01, масштаб цены неизвестен.
>   Строки T-Invest: на базах, где код A-01 уже работал до этой ревизии (тестовый стенд, локальные стенды), они лежат в
>   смешанном масштабе. На рабочей БД при выкатке с `v1.0.0` строк T-Invest к этой ревизии нет — их удалила
>   `e5a7c2d90b16`, а приложение между ревизиями не запускалось, — поэтому удаляются только 269 дневных строк ISS. Кэш
>   производный: ряды дозагрузит фоновая догрузка, ряд ISS — запрос графика, уже «как было».
>
> **Необратимость.** Удаление данных (первая, вторая и пятая ревизии) необратимо: `downgrade` первых двух — no-op,
> `downgrade` пятой удаляет `ohlcv_lot_history`, но строк свечей не возвращает; удалённые свечи перезагрузятся из источника.
> Схема обратима (`downgrade` первой, третьей, четвёртой и пятой ревизий удаляют таблицы и возвращают индекс; `downgrade` каждой ревизии проходит в `tests/test_market_data/test_bar_storage_migrations.py`). Откат самих
> свечей возможен только из бэкапа — **перед обновлением снимите копию БД** через `sqlite3.Connection.backup`
> (§6.1, gotcha-19), как перед любой data-миграцией.
>
> **Замеры на копии рабочей БД** (снята 2026-10-23 на ревизии `a7c1d9e2b4f8`, файл 114,6 МБ; каждая ревизия — отдельный
> `alembic upgrade`, время включает ≈ 1,2 с на старт alembic с импортом приложения; `a858647f24e6` появилась после
> замеров — её время не замерялось):
>
> | Ревизия | Время, с | `ohlcv_cache`, строк (до → после) | WAL после, МБ |
> |---|---|---|---|
> | `b9d2f6a14c83` | 1,77 | 420 950 → 20 456 | ≈ 29 (пик) |
> | `e5a7c2d90b16` | 1,26 | 20 456 → 269 | 0 |
> | `c4e81f3a9d27` | 1,29 | 269 → 269 | < 0,1 |
> | `f1b3d8e62a54` | 1,24 | 269 → 269 | < 0,1 |
> | `a858647f24e6` | не замерялось (мгновенно: таблица маленькая) | 269 → 0 | — |
>
> Итого строк `ohlcv_cache`: **420 950 → 269** после четырёх ревизий (остались только дневные бары ISS индекса IMOEX;
> строк T-Invest нет — было 420 680: 1m 17 080, 5m 252 207, 15m 120 883, 1h 26 527, 4h 877, D 3 106) и **→ 0** после
> `a858647f24e6` (все 269 — строки ISS `D`; `ohlcv_dealer`, `ohlcv_coverage` и `ohlcv_volume_check` на рабочей БД к этой
> ревизии пусты). Целей фоновой догрузки засеяно **9**. Для пика WAL (≈ 29 МБ) нужно
> столько же свободного места на диске поверх файла БД.
> **Файл БД не уменьшается**: освобождённые страницы уходят в freelist — после четырёх ревизий ≈ 18,7 тыс. страниц ≈ 76 МБ
> свободных до `VACUUM` (после него файл был бы ≈ 38 МБ). `VACUUM` делается в окне обслуживания и в обновление не входит.
>
> **После выкатки.** До первой загрузки кэш свечей пуст (и T-Invest, и ISS): цена из кэша для Circuit Breaker и P&L
> (ФТ §11.4) появляется после загрузки ряда бумаги, график без T-Invest дозагружает ряд ISS по запросу. При первом
> входе пользователя стартует фоновая догрузка по 9 целям: порядка 1,05–1,1 тыс. запросов к T-Invest, 3–7 минут (самая тяжёлая цель — SBER `1m` с 2023-01-01, ≈ 3,8 года минуток; архивы
> `history-data` — не чаще 30 запросов в минуту), в конце — сверка дневного объёма с ISS (`ohlcv_volume_check`). *С A-02 (`79f3c238dbb8`) — не при входе, а сразу после старта и ежедневно в 04:00 МСК, только «счётом для рыночных данных» и по вселенной бумаг, с минутами T-Invest с 13.06.2018 и ISS с 2011 года: первый прогон — часы и ≈ 2 ГБ (блок A-02 выше).* Оценка
> сделана до дефектов 4–6: теперь на каждый год архива запрашивается дневная история ISS основного режима (`history`
> TQBR, страницы по 100 дней), а дни без точного совпадения с ней (дилер в окне биржевых часов, неполный архив дня, бумага
> вне TQBR; ISS недоступен — весь год) уходят в обычные запросы истории, так что запросов может быть больше. До
> окончания догрузки свечи дозагружаются по запросу графика или бэктеста (график 4h за 90 суток при пустом кэше —
> порядка 90 запросов минут). Проверка: `alembic current` → `a858647f24e6` (с A-03 — `d92b7dd5c939`, с A-05 — `cdb15f21e5ca`, с Б-02 — `0acf47e6607f`, с Б-01 — `b01e7a5c3d92`, с A-02 — `79f3c238dbb8`); `SELECT DISTINCT timeframe FROM
> ohlcv_cache` → только `1m`, `D`; `ohlcv_coverage` растёт по мере загрузки, `ohlcv_lot_history` — по мере загрузки
> архивов фоновой догрузкой (строка — смена лота бумаги). Новая настройка `MARKET_DATA_STREAM_MODE`
> (§3.2) обязательной не является (с A-03 её умолчание — `single_include_weekend`, см. блок A-03 выше).

> **ℹ️ Ревизия `fd3492233bc0` (S8R-FIX-071, 2026-10-10).** Добавляет
> `live_trades.close_reason` (nullable, String(40)) — причину закрытия, когда
> закрывал не терминал (`closed_outside_terminal`, `…_by_user`,
> `sandbox_account_reopened`). По ней сделка получает «P&L не определён» вместо
> `pnl = 0`: результат позиции, закрытой в приложении брокера, неизвестен, и
> выдуманный ноль уходил в журнал, дневную статистику, дневной лимит Circuit
> Breaker и налоговый раздел. Существующие строки получают NULL — история не
> меняется. `downgrade` снимает колонку (round-trip тест
> `tests/unit/test_migration.py::test_live_trades_close_reason_round_trip`).

> **ℹ️ Обновление с версии старше `606214dc5ccf` (S8R-FIX-072, 2026-10-08).** Догоняющая ревизия для баз, прошедших старый путь миграций: добавляет в `ai_provider_configs` колонки `daily_limit` (100), `monthly_limit` (3000), `usage_today` (0), `usage_month` (0), **только если их нет**; на базе, созданной миграциями с нуля, ничего не делает. Без неё на такой базе после обновления падают запросы к настройкам AI («no such column»). `downgrade` — без изменений схемы (колонки принадлежат более ранней ревизии `a1b2c3d4e5f6`). Перед обновлением существующей базы — прогон `alembic upgrade head` и `alembic check` на копии (снимок `sqlite3.Connection.backup`).

> **ℹ️ Обновление с версии старше `752ae89f7ec2` (S8R-FIX-039, 2026-10-03).** Колонки журнала Circuit Breaker `trigger_value`/`limit_value` становятся `Numeric(18,4)` и допускают NULL (проверки без числа); старые строки не меняются. Обратима: `downgrade` пишет 0 вместо NULL и возвращает `Numeric(18,2) NOT NULL`.

> **ℹ️ Обновление с версии старше `c04dc4f891a2` (S8R-FIX-049, 2026-10-02).** Комиссия по сторонам в `live_trades` (`entry_/exit_commission`, источник и время каждой стороны, `commission_legacy`, `commission_checked_at`). Открытые сделки: сторона входа = итог, факт `broker` переносится как провизорный `broker_partial` (досверщик перечитает его у брокера один раз). Закрытые до ревизии: `commission_legacy=1` — стороны не раскладываются, досверщик их не трогает, налоговый отчёт делит их комиссию 50/50, как раньше. Обратима (`alembic downgrade -1` снимает колонки, итоговая `commission` сохраняется).

> **ℹ️ Обновление с версии старше `fb26c08f99c8` (S8R-FIX-056, 2026-10-02).** Индекс `ix_instruments_ticker_upper` по выражению `upper(ticker)` — поиск справочника без учёта регистра. Создаётся мгновенно, данные не меняются; `alembic downgrade -1` удаляет индекс. `alembic check` пишет предупреждение, что индекс по выражению на SQLite не читается обратно, — это ожидаемо.

> **ℹ️ Обновление с версии старше `7fe0fae4293c` (S8R-AUDIT-096, 2026-10-01).** `tax_lots.report_id` (FK → `tax_reports`, CASCADE), колонка `remaining_quantity` снята; существующие лоты удаляются (производные данные, пересоздаются следующей генерацией отчёта); прежние отчёты и их файлы сохраняются; `downgrade` возвращает `remaining_quantity=0`.

> **ℹ️ Обновление с версии старше `c7d2e5a19f84` (S8R-AUDIT-087, 2026-09-30).** Добавляет `trading_sessions.consecutive_broker_rejections` (серия отказов брокера для автопаузы); обратима; существующие сессии получают 0. Открытые сделки, записанные до обновления как частично исполненные, будут урегулированы первым проходом recovery (одно чтение ордера у брокера и одно уведомление на сделку).

> **ℹ️ Обновление с версии старше `bbb9a0908e89` (S8R-FIX-027, 2026-09-30).** Снимает колонку `circuit_breaker_configs.block_shorts` (флаг запрета шорта; после S8R-AUDIT-032 штатно не действовал). Строки конфигов сохраняются; `downgrade` возвращает колонку со значением `1`.

> **ℹ️ Обновление с версии старше `6f3a063d1c2e` (S8R-AUDIT-063, 2026-09-30).** Удаляет из `ohlcv_cache` свечи MOEX ISS (`source` = `moex_iss`, `tinvest+iss`), записанные со сдвигом +3 ч (время ISS — московское, раньше помечалось как UTC). Кэш производный — перезагрузится при первом запросе графика; `downgrade` — no-op.

> **ℹ️ Обновление с версии старше `2c7bd0443aa6` (S8R-AUDIT-081, 2026-09-30).** Миграция данных, схема не меняется: «Черновик» с завершённым бэктестом → «Протестирована»; «Протестирована/Бумажная/Боевая» пересчитываются по живым сессиям (sandbox/real → «Боевая», иначе paper → «Бумажная», иначе «Протестирована»); «Пауза»/«Архив» не меняются. **Необратима** (`downgrade` ничего не восстанавливает) — снимите бэкап БД до `upgrade` (gotcha-19). После обновления запуск сессии возможен только из «Протестирована/Бумажная/Боевая».

> **ℹ️ Обновление с версии старше `6128b9c52d8a` (S8R-AUDIT-071, 2026-09-26).** Режим размера позиции стал обязательным. Сессии без режима (созданные до его появления) получают «Фикс. лоты» = 1 — ровно тот объём, которым они торговали; их прежнее значение размера перезаписывается. Сессия с неизвестным режимом или без размера для «Фикс. сумма»/«Процент» на первом сигнале встаёт на паузу с уведомлением — перезапустите её с заданным размером. `downgrade` обратим только по схеме, данные не откатывает; SQLite пересоздаёт таблицу `trading_sessions` — перед обновлением снимите backup (§6.1).

> **ℹ️ Обновление с версии старше `f6a2c8e41d93` (S8R-AUDIT-068, 2026-09-25).**
> Ревизия добавляет nullable-колонку `trading_sessions.peak_equity` (пик капитала
> sandbox/real-сессии для лимита просадки); у существующих сессий пик = стартовый
> капитал. Обратима (пик теряется); SQLite пересоздаёт таблицу — снимите backup (§6.1).

> **ℹ️ Обновление с версии старше `e9e5c919fbbf` (S8R-AUDIT-025, 2026-09-25).**
> Ревизия добавляет nullable-колонку `live_trades.lot_size` (размер лота на момент
> сделки). Старые сделки считаются по прежней деривации, действий не требуется;
> SQLite пересоздаёт таблицу — перед обновлением снимите backup (§6.1).

> **ℹ️ Обновление с версии старше `b8e4d17c9a52` (S8R-AUDIT-101, 2026-09-25).**
> Ревизия добавляет `UNIQUE (user_id, broker_type, account_id)` на `broker_accounts`.
> Перед этим дубли одного счёта сливаются: остаётся запись, на которую ссылаются
> торговые сессии, при равенстве — активная, затем более ранняя; ссылки остальных
> перевешиваются на неё, лишние удаляются; если среди удалённых была активная —
> выжившая включается. Обратима по схеме; слитые дубли `downgrade` не восстанавливает.
> SQLite пересоздаёт таблицу — перед обновлением снимите backup (§6.1).

> **ℹ️ Обновление с версии старше `d4f1a9c2b7e0` (S8R-AUDIT-024, 2026-09-24).**
> Ревизия добавляет в `live_trades` две nullable-колонки `client_order_id` и
> `exit_client_order_id` (ключ ордера для выяснения его судьбы у брокера при
> потерянном ответе). Существующие данные не меняются, миграция обратима; SQLite
> пересоздаёт таблицу (`batch_alter_table`) — перед обновлением снимите backup (§6.1).

> **ℹ️ Обновление с версии старше `c5e8b2a7f913` (S8R-AUDIT-035, 2026-09-24).**
> Ревизия добавляет `users.token_version` (`INTEGER NOT NULL DEFAULT 0`). Смена пароля
> и повторное предъявление уже использованного refresh-токена теперь отзывают **все**
> сессии пользователя на всех устройствах; вкладка, где меняли пароль, получает новую
> пару токенов автоматически. Перелогин после обновления не требуется: токены без
> версии считаются версией 0. Миграция идемпотентна и обратима; SQLite пересоздаёт
> таблицу `users` (`batch_alter_table`) — перед обновлением снимите backup (§6.1).

> **ℹ️ Обновление с версии старше `a7b8c9d0e1f2` (S8R, 2026-08-06).**
> Ревизия `a7b8c9d0e1f2` (`backfill_real_figi_in_broker_trades`) —
> **миграция данных**, схему не меняет (`S8R-SANDBOX-TRADES-FAKE-FIGI`).
>
> Что делает: у сделок сессий `sandbox`/`real` значение `live_trades.figi`
> вида `paper_<ТИКЕР>` заменяется на настоящий FIGI из справочника
> `instruments`, а где справочник его не знает — на `NULL`. Paper-сделки не
> трогаются: брокера в этом режиме нет, и `paper_<ТИКЕР>` там осмысленный
> маркер. Заодно зануляется `paper_*` в самом справочнике `instruments`.
>
> Зачем: FIGI — единственный ключ, по которому терминал сопоставляет свою
> сделку с позицией у брокера. Фиктивный `paper_SBER` хуже пустого значения —
> пустое сверка честно пропускает, а фиктивное выглядит сопоставимым ключом,
> не совпадает ни с одной позицией, и здоровая позиция объявляется
> расхождением с остановкой сессии. Начиная с этой версии FIGI новых сделок
> запрашивается у самого брокера и кэшируется в справочнике.
>
> ⚠️ **`downgrade` прежние значения НЕ восстанавливает** — и не может: какие
> строки были заполнены фиктивно, после обновления неизвестно, а
> восстановление «по формуле» записало бы `paper_*` в сделки, которые его
> никогда не содержали. Схема обратима полностью (менять нечего), данные —
> нет. Откат к `e5f6a7b8c9d0` оставляет данные исправленными.
>
> ⚠️ **Позиции, открытые ДО обновления, настоящий FIGI задним числом не
> получат:** он записывается только при создании сделки. Для закрытых сделок
> это безразлично — сверка смотрит лишь открытые. А вот открытую позицию,
> оставшуюся с `NULL`, сверка будет молча пропускать (в логе —
> `session_runtime_recovery_skip_figi_none`). Перед обновлением такие позиции
> лучше закрыть, а после — открыть заново.
>
> **Что ещё изменилось в поведении этой версией** (миграции не требует):
> * сверка позиций идёт **в фоне раз в 15 минут**, а не только при старте
>   сессии; сессии без открытых сделок и без пометки расхождения не
>   опрашиваются, чтобы не тратить лимиты брокера;
> * сессия ставится на паузу, **только если бумаг у брокера меньше**, чем
>   числится за ней. Ситуация «у брокера больше» штатна для счёта, на котором
>   работают две сессии или где владелец докупил бумагу вручную: расхождение
>   показывается, но торговля продолжается;
> * повторный уход сессии на паузу после «Возобновить» теперь сопровождается
>   уведомлением.

> **ℹ️ Обновление с версии старше `e5f6a7b8c9d0` (S8R, 2026-08-05).**
> Две ревизии одного цикла «сверка с брокером», обе идемпотентны и обратимы.
>
> `d4e5f6a7b8c9` — `live_trades.exit_broker_order_id` и
> `live_trades.exit_order_placed_at` (`S8R-EXIT-ORDER-NOT-TRACKED`).
> Идентификатор **закрывающего** ордера: `broker_order_id` занят ордером входа,
> поэтому до этой версии рестарт backend в момент, когда встречный ордер был в
> полёте, оставлял сделку `filled` навсегда — у брокера позиция закрывалась, а
> в терминале нет. После обновления доучёт делается при старте backend и далее
> раз в минуту: статус ордера **опрашивается**, ордер никогда не отправляется
> повторно. У сделок до этой версии обе колонки `NULL` — это «сведений нет», и
> доучёт их не трогает.
>
> `e5f6a7b8c9d0` — `trading_sessions.position_mismatch_at` и
> `position_mismatch_note` (`S8R-SANDBOX-POSITIONS-NOT-RECONCILED`). При старте
> сессии открытые позиции сверяются с `GetPositions` брокера (теперь и в
> sandbox, не только в `real`). **Расхождение показывается, а не чинится:**
> позиции не закрываются и объём не переписывается, сессия ставится на паузу с
> видимой пометкой в карточке и критическим уведомлением. Автозакрытие вслепую
> здесь запрещено намеренно: песочница умеет вернуть неполный портфель, и
> терминал закрыл бы живую позицию.
>
> ⚠️ После обновления возможна **пауза сессии с пометкой «Расхождение с
> брокером»** — это не сбой обновления, а первая честная сверка. Сверьтесь с
> приложением брокера и закройте лишнее вручную; когда позиции сойдутся,
> пометка снимается сама на следующей сверке.

> **ℹ️ Обновление с версии старше `c3d4e5f6a7b8` (S8R, 2026-08-04).**
> Ревизия `c3d4e5f6a7b8` (`commission_source_and_broker_rate`) добавляет три
> колонки:
> * `live_trades.commission_source` — откуда взялась комиссия: `broker`
>   (удержано брокером, T-Invest `OrderState.executed_commission`) или
>   `estimated` (оценка по тарифу — песочница комиссию обычно не удерживает).
>   У сделок до этой версии — `NULL`;
> * `broker_accounts.commission_pct` — тариф счёта, `server_default = '0'`.
>   **Задайте его вручную** на странице «Настройки → Брокер» для каждого
>   счёта: пока он нулевой, sandbox/real-сделки без фактической комиссии
>   брокера продолжат записываться без удержания, а paper-сессии — считать по
>   ставке из формы запуска (0,05 % по умолчанию);
> * `tax_lots.trade_mode` — режим сделки в отчёте 3-НДФЛ. Sandbox-сделки
>   остаются в отчёте, но помечаются колонкой «Режим» и примечанием.
>
> ⚠️ **Отчёты 3-НДФЛ, сформированные до этой версии, следует
> перегенерировать.** Ревизия идёт в паре с исправлением
> `S8R-TAX-PNL-LOSES-LOT-SIZE`: раньше результат сделки считался по лотам без
> множителя штук/лот, и по инструменту с `lot_size = 10` налоговая база была
> занижена в десять раз. Файлы старых отчётов не пересчитываются
> автоматически — их нужно сформировать заново на странице «Счёт».
>
> Миграция **идемпотентна** (колонка уже есть — пропускается) и **обратима**:
> `downgrade` снимает все три колонки.
> ⚠️ SQLite пересоздаёт таблицы целиком (`batch_alter_table`), затронуты
> `live_trades`, `broker_accounts`, `tax_lots` — снимите backup перед
> обновлением (§6.1).

> **ℹ️ Обновление с версии старше `b7c8d9e0f1a2` (S8R, 2026-08-03).**
> Ревизия `b7c8d9e0f1a2` (`add_trading_sessions_commission_pct`) добавляет
> колонку `trading_sessions.commission_pct` — ставку комиссии, которую
> бумажная торговля теперь удерживает так же, как её вычитает бэктест.
> `server_default = '0'`, поэтому **уже запущенные сессии продолжают считать
> деньги по-прежнему**: ретроактивного удержания нет, история не
> пересчитывается. Новые сессии получают ставку из формы запуска (по
> умолчанию 0,05 %).
> Миграция **идемпотентна** (колонка уже есть — проходит без ошибки) и
> **обратима**: `downgrade` снимает колонку.
> ⚠️ SQLite не умеет нативный `ADD COLUMN` с пересозданием ограничений,
> поэтому Alembic пересоздаёт таблицу `trading_sessions` целиком
> (`batch_alter_table`) — снимите backup перед обновлением (§6.1).

> **ℹ️ Обновление с версии старше `e2f3a4b5c6d7` (S8R, 2026-08-03).**
> Ревизия `e2f3a4b5c6d7` (`drop_daily_stats_unrealized_pnl`) удаляет колонку
> `daily_stats.unrealized_pnl`. Колонка не заполнялась никогда с момента
> создания схемы, поэтому **данные не теряются**: все её значения — нули по
> построению. Нереализованный P&L считается на лету
> (`app/trading/unrealized.py`), в БД не хранится.
> Миграция **идемпотентна** (если колонки уже нет — проходит без ошибки) и
> **обратима**: `downgrade` возвращает колонку с прежним типом и
> `DEFAULT 0`. Отдельных действий администратора не требуется.
> ⚠️ SQLite не умеет нативный `DROP COLUMN`, поэтому Alembic пересоздаёт
> таблицу `daily_stats` целиком (`batch_alter_table`) — как и при любой
> правке схемы на SQLite, снимите backup перед обновлением (§6.1).

> **ℹ️ Обновление с версии старше `d1e2f3a4b5c6` (S8R, 2026-07-28).**
> Ревизия `d1e2f3a4b5c6` (`s8r_fresh_db_drift`) досоздаёт таблицу
> `user_ai_settings` и восемь колонок, которых не хватало на чистых установках
> (подробности — §3.3). Она **идемпотентна**: на стенде, где эти объекты уже
> появились через `init_db`, миграция проходит без ошибки «duplicate column» и
> данные не трогает. Отдельных действий администратора не требуется.
> Ограничение `downgrade`: обратная миграция удаляет добавленные объекты
> **вместе с данными** в `user_ai_settings` — перед откатом снимите backup
> (§6.1).

> **⚠️ Разовая процедура при обновлении до версии с BE-TRAD-06 (денежный учёт paper-портфеля, Model A).**
> До этой версии paper-BUY не списывал `PaperPortfolio.balance` и не блокировал капитал. Позиция, **открытая на старом коде и ещё не закрытая на момент апдейта**, при закрытии на новом коде получит зачисление `proceeds` без парного списания при открытии → `balance` завысится, а `blocked_amount` уйдёт в underflow (безопасно зажимается в 0 с `log.error("paper_blocked_amount_underflow")`). Эффект только на paper, транзиентный (само-исцеляется после одного цикла закрытия), но искажает equity/CB-drawdown для straddle-позиций.
> **Перед обновлением:** закрыть/сбросить открытые paper-сессии (в UI закрыть все позиции активных paper-сессий, либо остановить paper-сессии). Оба пути ведут в один и тот же код: `stop_session` → `OrderManager.close_all_positions` → `close_position` → `PaperPortfolioAccountant.apply_close`, то есть деньги возвращаются в портфель штатно.
> **Закрытие — best-effort, обязательно проверьте результат.** `stop_session` доводит сессию до `stopped` даже если часть позиций закрыть не удалось (нет рыночной цены, брокер недоступен): такие позиции остаются в реальном open-статусе, в лог уходит `stop_session_positions_not_closed`, пользователю — уведомление `positions.close_failed`. Drain считается выполненным только когда открытых paper-позиций не осталось:
> ```bash
> docker compose exec backend python -c "
> import asyncio, sqlalchemy as sa
> import app.common.database as database
> from app.trading.models import LiveTrade, TradingSession
> async def main():
>     # AsyncSessionLocal создаётся в init_db() (lifespan приложения); в разовом
>     # скрипте её надо инициализировать явно, иначе она равна None.
>     await database.init_db()
>     async with database.AsyncSessionLocal() as db:
>         rows = (await db.execute(
>             sa.select(TradingSession.id, sa.func.count(LiveTrade.id))
>             .join(LiveTrade, LiveTrade.session_id == TradingSession.id)
>             .where(TradingSession.mode == 'paper',
>                    LiveTrade.status.in_(['filled', 'pending']),
>                    LiveTrade.closed_at.is_(None))
>             .group_by(TradingSession.id)
>         )).all()
>         print(rows or 'открытых paper-позиций нет — drain выполнен')
> asyncio.run(main())
> "
> ```
> **Альтернатива** — одноразовая реконсиляция БД: `blocked_amount = Σ(volume_rub открытых сделок)`, `balance = initial_capital + Σ(realized pnl)`, **и обязательно** `peak_equity = balance + blocked_amount`. Про `peak_equity` забывать нельзя: по нему Circuit Breaker считает max drawdown (`(peak_equity − equity) / peak_equity`, `app/circuit_breaker/engine.py`), и оставленное завышенное пред-апдейтное значение даст фиктивную просадку на первой же проверке — CB поставит сессии на паузу без причины. `volume_rub` для paper уже включает `lot_size` (пишется как `price × lots × lot_size` при создании сделки), так что множитель в формуле не нужен. Пред-апдейтные значения `balance/blocked_amount` и так были фикцией (см. `Спринты/Code_Review_Full_2026-07/BE_TRAD_06_LOG.md`), поэтому сброс безопасен.

**Rollback** (при провале миграции):
```bash
cd ~/Apps/moex-terminal
git checkout <previous-tag>
cd Develop && docker compose build && docker compose down && docker compose up -d
# Если миграция уже применилась — откатить вручную:
docker compose exec backend alembic downgrade -1
```

---

## 7а. Обновление dev-стенда (голый venv, без Docker) — S8R-FIX-086

§7 выше — про production: там зависимости ставятся **внутрь образа** при
`docker compose build`, и отдельного шага установки нет. На машине разработки Docker
не используется (`Develop/CLAUDE.md`: «Docker на машине разработки отсутствует»),
backend и frontend работают из `backend/.venv` и `frontend/node_modules` напрямую,
поэтому после `git pull` зависимости надо доставить руками — **тем же рецептом**, что
в `Dockerfile.backend` и в CI (`.github/workflows/ci.yml`, шаг «Install dependencies
(lock + хэши, S8R-AUDIT-050)»). Первичная установка окружения с нуля —
`Develop/backend/INSTALL.md`.

### 7а.1 Когда нужен полный путь

Полный путь нужен, только если `git pull` принёс изменения в lock-файлы или в
`pyproject.toml`. Сразу после pull посмотрите, что приехало:

```bash
cd <repo>/Develop            # код — отдельный git, своя ветка (develop)
git pull origin develop
git diff --stat HEAD@{1} HEAD -- backend/requirements.lock backend/requirements-dev.lock \
    backend/pyproject.toml frontend/pnpm-lock.yaml frontend/package.json
```

| Что изменилось в pull | Что делать |
|---|---|
| `backend/requirements.lock`, `backend/requirements-dev.lock` | шаг 7а.3 (п. 1, 3, 4) |
| строка `tinkoff-investments @ git+…@<sha>` в `backend/pyproject.toml` | шаг 7а.3 целиком, включая п. 2 (SDK) |
| прочее в `backend/pyproject.toml` (версия, extras, scripts) | шаг 7а.3 без п. 2 |
| `frontend/pnpm-lock.yaml`, `frontend/package.json` | шаг 7а.3, п. 5 (`pnpm install --frozen-lockfile`) |
| только `.py` / `.ts` / миграции | ничего не ставить — сразу шаг 7а.4 (перезапуск) |

Переустанавливать SDK T-Invest **не нужно**, если его commit-hash в
`backend/pyproject.toml` не менялся: скрипт просто склонирует тот же коммит заново.
Текущий sha, на который указывает репозиторий, — `sh scripts/install_tinvest_sdk.sh --print-sha`.

### 7а.2 ⚠️ Почему `pip install -r backend/requirements.lock` не работает

Очевидная команда падает ещё до скачивания пакетов (проверено 2026-10-12):

```
ERROR: Can't verify hashes for these requirements because we don't have a way to hash
version control repositories:
    tinkoff-investments @ git+https://github.com/RussianInvestments/invest-python.git@<sha>
```

`requirements.lock` сгенерирован **с хэшами** sha256 для каждой строки
(`backend/scripts/lock_deps.sh`), а SDK T-Invest закреплён git-зависимостью по
commit-hash — у VCS-ссылки хэша артефакта нет и быть не может. pip, увидев хэши,
переключается в режим `--require-hashes`, где хэш обязателен у **каждого** требования,
и отказывается от файла целиком, а не только от строки SDK. Поэтому установка
разделена на две части: PyPI-часть lock — с хэшами, SDK — отдельным скриптом по sha
(`backend/scripts/install_tinvest_sdk.sh`, единственное место, где SDK берётся из git).

### 7а.3 Последовательность

```bash
# --- backend (из <repo>/Develop/backend; venv стенда — ./.venv) ---
cd <repo>/Develop/backend

# 1. PyPI-часть lock: вырезаем строку SDK (git, без хэша — п. 2) и строки `-r`.
#    requirements-dev.lock нужен для гейтов (pytest/ruff/mypy/bandit); стенду без
#    прогона тестов достаточно одного requirements.lock.
T="${TMPDIR:-/tmp}"
grep -v -e '^tinkoff-investments @ ' -e '^-r ' requirements.lock > "$T/req.txt"
grep -v -e '^-r ' requirements-dev.lock > "$T/req-dev.txt"
.venv/bin/pip install --no-deps --require-hashes -r "$T/req.txt" -r "$T/req-dev.txt"

# 2. SDK T-Invest — по commit-hash из pyproject.toml, с патчем битой зависимости
#    `tinkoff`. ПРОПУСТИТЬ, если sha не менялся (см. 7а.1).
PYTHON=.venv/bin/python sh scripts/install_tinvest_sdk.sh

# 3. Сам backend — без зависимостей и без изоляции сборки (setuptools из lock).
.venv/bin/pip install --no-deps --no-build-isolation -e .

# 4. Проверки — ОТДЕЛЬНЫМИ строками, НЕ через `&&` (см. 7а.5).
.venv/bin/pip check
.venv/bin/python scripts/verify_install.py requirements.lock requirements-dev.lock

# --- frontend ---
cd ../frontend
# 5. Ровно то, что в pnpm-lock.yaml; расхождение с package.json — ошибка,
#    а не тихое обновление лока.
pnpm install --frozen-lockfile
```

`verify_install.py` — финальный гейт: сверяет, что каждая строка lock установлена именно
той версией, и что доехали пакеты extras (`uvicorn[standard]` → `uvloop`/`httptools`,
`sqlalchemy[asyncio]` → `greenlet`, `fonttools[woff]` → `brotli`), которых `pip check`
не видит. Код возврата 1 и список проблем = окружение собрано не полностью.

Если установка ведёт себя не так, как в CI, первым делом сверьте версию pip:
CI и `Dockerfile.backend` её закрепляют (`python -m pip install pip==26.0.1`),
в venv стенда она может быть другой (`.venv/bin/pip --version`).

### 7а.4 Перезапуск (миграции применяются здесь)

Приложение миграции само не применяет — их выполняет стартовая строка. В Docker это
`command:` backend-контейнера (`alembic upgrade head && exec uvicorn …`), на dev-стенде —
та же пара команд:

```bash
# остановить прежние процессы uvicorn/vite, затем, с теми же переменными окружения,
# с которыми стенд поднимался (DATABASE_URL, DEBUG, SECRET_KEY, ENCRYPTION_KEY, …):
cd <repo>/Develop/backend
.venv/bin/alembic upgrade head
.venv/bin/uvicorn app.main:app --port 8000

cd ../frontend && pnpm dev     # Vite видит новые пакеты только после рестарта
```

Проверка, что миграции доехали до головы: `.venv/bin/alembic current` (ожидается та же
ревизия, что в §7). Про сами ревизии и разовые процедуры — §7, они не зависят от того,
Docker это или venv. Если после обновления frontend показывает старую сборку — кэш Vite
сносить только при остановленном dev-сервере (gotcha-46).

### 7а.5 `pip check`: сообщение про `tinkoff` — штатное

```
tinkoff-investments 0.2.0b117 requires tinkoff, which is not installed
```

Метаданные установленного SDK сохраняют `Requires-Dist: tinkoff (>=0.1.1,<0.2.0)` —
дистрибутива с таким именем на PyPI нет (у upstream это ошибка в его `pyproject.toml`).
Сам SDK работает: его код лежит в пакете `tinkoff/invest/`, поставленном вместе с ним.
Это зафиксировано как норма в `Develop/CLAUDE.md`.

Практическое следствие: из-за этой строки **`pip check` возвращает ненулевой код**.
Если склеить проверки через `&&`, следующая команда (`verify_install.py`) молча не
выполнится — поэтому в 7а.3 они идут отдельными строками, а вывод `pip check` читается
глазами: любая другая строка, кроме этой, — реальная рассогласованность версий.

---

## 8. Мониторинг

### 8.1 Health endpoint

```bash
curl http://localhost/api/v1/health
```

Возвращает (S8R-AUDIT-054):
```json
{"status":"ok","version":"0.1.0","database":"connected","cb_state":"ok","tinvest_connected":true,"scheduler_running":true,"scheduler_jobs":6}
```
`status`: `ok` (200) или `degraded` (**503**, если БД недоступна или планировщик остановлен); тело при 503 то же — видно, что отказало. `tinvest_connected` на статус не влияет. Liveness: `GET /api/v1/health/live` → 200 всегда. Healthcheck контейнера — readiness; `restart: unless-stopped` не перезапускает unhealthy-контейнер (только вышедший процесс), при unhealthy backend frontend не поднимется (`depends_on: service_healthy`). Число `scheduler_jobs` — ориентировочное.

### 8.2 Admin metrics (`/api/v1/admin/metrics`)

Plotly Dash панель — только для admin role (см. C-S8-9 DEV-4 W2):
- Откройте `http://localhost/api/v1/admin/metrics`.
- Авторизуйтесь через JWT (cookie `access_token` после логина в SPA).
- Графики: signal→order latency, оценка стратегии на свече, dashboard LCP,
  Telegram webhook latency, backtest jobs throughput.

**Проверка за nginx (S8R-AUDIT-056).** Раньше regex-локация статики nginx перехватывала JS-ассеты Dash под `/api/`, страница висела на «Loading…»; теперь `location ^~ /api/`. После деплоя: в DevTools → Network все `_dash-component-suites/…js` и `_favicon.ico` → 200, без `Cache-Control: public, immutable` (иначе запрос забрала локация статики).

**Источник данных и его ограничения (S8R, 2026-07-30).** До этой версии все
графики рисовали зашитые в код mock-массивы. Теперь четыре из пяти питаются
живыми замерами `@timed_event` из кольцевого буфера
`app/common/metrics_store.py`; пятый (backtest jobs) пока остаётся mock и так и
подписан на странице. Что из этого следует для эксплуатации:

- **Буфер в памяти процесса.** Рестарт backend (`docker compose restart
  backend`, деплой, падение) обнуляет графики — это не потеря данных, а
  отсутствие персистентности по замыслу: писать в SQLite на каждый тик
  торгового хот-пути дороже, чем сама метрика.
- **Глубина = ёмкость буфера**, последние N замеров на метрику, а не
  фиксированное окно времени. На тихом стенде это сутки, под нагрузкой —
  минуты.
- **Несколько воркеров покажут только свою долю.** Текущая поставка
  однопроцессная (один backend-контейнер), поэтому вопрос не стоит; при
  масштабировании график перестанет быть полным.
- **Пустой график с подписью «нет замеров» — нормальное состояние** сразу
  после старта: метрика ещё не срабатывала. Правдоподобных чисел вместо
  пустоты страница не рисует намеренно.
- **Dashboard LCP приходит из браузера**: `PerformanceObserver` в SPA шлёт
  замер в `POST /api/v1/observability/lcp` при уходе вкладки в фон. Эндпоинт
  требует обычной аутентификации (не admin — замер шлёт пользователь) и
  отбрасывает значения свыше 10 минут. Если график LCP пуст, а остальные
  наполнены — значит SPA не доходит до отправки (пользователь не логинился,
  либо браузер не поддерживает `largest-contentful-paint`).

### 8.3 Логи

```bash
docker compose logs -f backend         # tail backend
docker compose logs -f frontend        # tail nginx
docker compose logs --since 1h         # за последний час, оба сервиса
```

Логи контейнеров — драйвер `json-file` с ротацией **10 МБ × 5 файлов** на сервис (S8R-AUDIT-051, `docker-compose.yml`). Нативный запуск `scripts/start.sh`: при `DEBUG≠true` сначала preflight production-настроек (`scripts/check_production_env.sh`, значения из `.env`), `backend/logs/dev.log` ротируется copytruncate по 50 МБ × 5 (при старте и раз в минуту).

**Остановка backend** (S8R-AUDIT-051, gotcha-80): uvicorn запущен с `--timeout-graceful-shutdown 10` — сначала до 10 с на закрытие соединений, **затем** graceful shutdown торговли (до 45 с); сумма вместе с ожиданием фоновых задач outbox (до 6 с) и внешней доставки Telegram/Email (до 15 с) обязана быть меньше `stop_grace_period: 90s`.

> ℹ️ **Доставка уведомлений о закрытиях (S8R-FIX-065).** Lifespan: после `configure_outbox_factory` подключается доставщик outbox (`configure_outbox_deliverer`). Уведомления о закрытиях пишутся в `pending_events` вместе с закрытием и доставляются фоновой задачей сразу после commit — без ожидания слушателя сессии, в том числе при „Стопе“ и shutdown. Не доставленное к остановке досылает повтор при старте (окно 1 с — 1 ч). Shutdown после рантайма ждёт фоновые задачи outbox (до 6 с), затем внешнюю доставку Telegram/Email (до 15 с). `stop_grace_period` backend — 90 с (10 + 45 + 6 + 15 = 76): не уменьшайте.» Сюда же: в §3.3 и строках 90/677 гайда заменить «60 с» на «90 с

### 8.4 Метрики Performance baseline (Sprint 8 W2)

| Метрика | Цель | Фактический baseline |
|---------|------|----------------------|
| Signal → order place p95 | < 500 мс | измеряется `@timed_event(name="trading.signal_to_order")` |
| Dashboard первый paint (LCP) | < 2 с | измеряется через PerformanceObserver |
| Telegram webhook → reply | < 3 с | `@timed_event(name="telegram.webhook_to_reply")` |

Реальные числа — на странице `/api/v1/admin/metrics` после ≥ 1 дня работы.

---

## 9. Troubleshooting

### 9.1 `docker compose up -d` падает / контейнер не healthy

```bash
docker compose logs backend | tail -100
```

| Симптом | Причина | Решение |
|---------|---------|---------|
| `sqlite3.OperationalError: unable to open database file` | volume не примонтирован / нет write-прав | проверить `docker volume inspect moex-sqlite-data` |
| `alembic.util.exc.CommandError: Can't locate revision` | свежий clone, не применились миграции | `docker compose exec backend alembic upgrade head` |
| `ModuleNotFoundError: tinkoff` | T-Invest SDK не установился в builder | очистить `docker compose build --no-cache backend` |
| `KeyError: 'SECRET_KEY'` | `.env.production` не загружен | проверить `docker compose config` (секция env_file) |
| контейнер backend сразу выходит (`exit 1`), в логах `check_production_env`: обнаружены dev-значения / нет `.env.production` | preflight-проверка секретов (CFG-BE-02, S8R-AUDIT-034): `.env.production` отсутствует, либо `SECRET_KEY`/`ENCRYPTION_KEY` — `dev-*`, плейсхолдер `change-me`, < 32 байт или < 8 различных символов | заполнить `.env.production` реальными значениями (см. §3.2) |
| `RuntimeError: SECRET_KEY is a placeholder copied from .env.example with DEBUG=False` (или `… is too short` / `has too little entropy`) | в `.env.production` осталось значение шаблона или слабый ключ | см. §3.2, сгенерировать ключ |
| `ValueError: master_key too short` / `too little entropy` при старте контейнера (healthcheck не поднимается) | `ENCRYPTION_KEY` короче 32 байт или вырожденный (production, DEBUG=false — проверка в lifespan) | сгенерировать новый ключ ≥ 32 байт; ⚠️ смена ключа делает уже сохранённые broker-токены нерасшифровываемыми — пользователям нужно заново ввести токены |

### 9.2 Cloudflare Tunnel 502 Bad Gateway

```bash
cloudflared tunnel info moex-terminal      # connections должны быть active
docker compose ps                          # frontend должен быть Up healthy
```

Если `frontend` healthy, но Tunnel показывает 502 — проверить, что `cloudflared` смотрит на `http://localhost:80` (не `http://moex-frontend:80` — это имя только внутри Docker сети).

### 9.3 T-Invest connection_lost спамит уведомления

См. C-S8-7 DEV-2 W2 (S7R-CONNECTION-EVENTS-MARKET-CLOSED): фильтр `_is_moex_open_now()` уже подавляет уведомления вне торговой сессии. С A-03 `_is_moex_open_now()` опирается на `trading_hours.is_session_open()` — таблицу периодов расписания: уведомления идут во все биржевые периоды (аукционы, торги, вечер, торги выходного дня) и подавляются ночью и в часы дилера. Если они приходят в рабочее время:
- Проверить валидность API-ключа T-Invest пользователя через UI Settings → Broker.
- После Sandbox/Production токен switch: `docker compose restart backend` (singleton multiplexer кэширует token, см. C-S8-6).

### 9.4 Frontend lint падает локально (для разработчика)

```bash
cd Develop/frontend
pnpm install               # обновить deps до lock'а
pnpm lint                  # max-warnings 0 включён
```

Если падает в CI с Node 24 — проверить, что локально тоже Node 24 (`node -v`), `pnpm` через corepack.

### 9.5 Backup_job не пишет файлы

```bash
docker compose exec backend python -c "from app.scheduler.service import scheduler; print(scheduler.get_jobs())"
```

Если `backup_job` в списке нет — проверить `app/main.py` lifespan startup hooks; убедиться, что `scheduler.start()` вызывается.

### 9.6 Контейнер `frontend` показывает старую версию SPA после `git pull`

Хешированные чанки `/assets/` кешируются `immutable`, а `index.html` отдаётся с `Cache-Control: no-cache` (S8R-AUDIT-057/060) — браузер после обновления сам берёт новый `index.html`. Проверка: `curl -sI https://<домен>/ | grep -i cache-control` → `no-cache`. Если версия всё равно старая — причина в Docker layer cache. С Б-02 (v1.6) старая сборка SPA с новым backend не загружает и не сохраняет рисунки на графике: маршруты `/api/v1/charts/{ticker}/{tf}/drawings` сняты (§7) — вкладки, открытые до выкатки, обновить. Решение:
```bash
docker compose build --no-cache frontend
docker compose up -d frontend
```

---

## Ссылки

- [README.md](../README.md) — общее описание проекта и Quick Start (для разработки).
- [Develop/INSTALL.md](../Develop/INSTALL.md) — локальная установка для разработки (без Docker).
- [functional_requirements.md](functional_requirements.md) v4.12 — функциональные требования (M4 production-ready; изменения — таблица истории версий в шапке).
- [technical_specification.md](technical_specification.md) v4.4 — техническое задание (раздел «Deployment Architecture»; §8 — Alembic и `DATABASE_URL`, drain paper-сессий при выкатке).
- [development_plan.md](development_plan.md) — дорожная карта (M4 ✅, Sprint_8_Review план).
- [Sprint_8/arch_design_s8.md](../Спринты/Sprint_8/arch_design_s8.md) §7 — Deployment target rationale.
- [Sprint_8/security_audit_s8.md](../Спринты/Sprint_8/security_audit_s8.md) — security findings и принятые fix'ы.
