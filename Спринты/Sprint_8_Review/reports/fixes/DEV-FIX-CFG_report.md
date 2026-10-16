## DEV-FIX-CFG отчёт — S8R fixes, FIX (031, 033, 034, 046)
Статус: ✅ готово к коммиту (worktree `wt-s8r-fixes-b`, база 5b9e6bd, не закоммичено)

### 1. Что реализовано
- **031** `/health`: `asyncio.timeout(5)` + `scope.expired()` (gotcha-74) → `database:"timeout"`, 503 degraded. После таймаута в БД больше не ходим (gotcha-38). Эндпоинт публичный, поэтому новые поля только в расширенном (авторизованном) ответе: `ai_configured` (true / false / null: null — БД лежит и серверного ключа нет) и `telegram_configured`. Сетевых вызовов нет, на статус не влияют.
- **033** `AI_MODEL="claude-opus-5-5"`; `AI_DAILY/MONTHLY_LIMIT` удалены (`extra="ignore"`, старый `.env` стартует); `start.sh`: `run_uvicorn`, `--reload` только при DEBUG (env > `.env`, истинные значения как у pydantic).
- **034** `object.__subclasses__` → `__subclasses__`; храповик 053: было `object.__subclasses__` → стало `__subclasses__`; пустая параметризация «dotted» заменена тестом «нет записей с точкой».
- **046** `_embedded_ipv4`: mapped, NAT64 `/96` и `64:ff9b:1::/48`, 6to4, Teredo (оба адреса). Сделано ужесточением: с публичным IPv4 разрешены только mapped и NAT64-WKP. 6to4, Teredo и NAT64-local и так запрещены базовым `is_global` (IANA). Ослаблять их не стал: «запрещён встроенный → запрещён весь», а не наоборот.

### 2. Файлы
Изменены: `app/main.py`, `app/config.py`, `app/sandbox/ast_analyzer.py`, `app/ai/url_validator.py`, `scripts/start.sh`, `docs/env_vars.md` (его сверяет тест с `Settings`); тесты: `test_health.py`, `test_config.py`, `test_denylist_is_exhaustive.py`, `test_url_validator_ranges.py`, `test_ai/test_router.py` (2 мёртвые строки мока).

### 3. Тесты
RED: 031 `TimeoutError` (запрос висел 5 с) / `KeyError: 'ai_configured'`; 033 `assert 'claude-sonnet-4-6-20250514' == 'claude-opus-5-5'`, `start.sh: run_uvicorn: command not found`; 034 `мёртвые записи денилиста (имя с точкой): ['object.__subclasses__']`; 046 `DID NOT RAISE InvalidProviderURLError` [nat64-wkp], pinned `_StopAfterConnect`.
Мутации (бэкап + md5, всё откатано): `asyncio.timeout(None)` → TimeoutError; безусловный `--reload` → `assert ('--reload' in […]) is False`; возврат `object.__subclasses__` → 3 падения; `return False` вместо проверки встроенного → DID NOT RAISE.
Гейты: pytest **5392 passed / 0 xfailed / 0 failed** (1 skipped — не мой, был и раньше); ruff 0; mypy Success (201); bandit M0/H0; typecheck 0; lint 0; build ok; vitest: фронт не менялся, прогон на уровне пакета.

### 4. Integration points
✅ `_health_ai_configured` `main.py:729`; `_embedded_ipv4` `url_validator.py:119`: тот же барьер в sync, async и pinned-соединении; `run_uvicorn` `start.sh:129`.

### 5. Контракты
`/health` (расширенный): `database` + `"timeout"`; новые поля `ai_configured: bool|null`, `telegram_configured: bool`. Миграций нет.

### 6. Правки документов / находки
- ТЗ §8.5 п.1: «предел 5 с → `database: timeout`, тот же 503»; новый п.6: «`ai_configured`/`telegram_configured` (только расширенный ответ, без сети, на статус не влияют)». Строку «Не реализовано… S8R-FIX-031» удалить.
- ТЗ §2183–2185 и §8.2:2779: `AI_MODEL=claude-opus-5-5`, строки `AI_*_LIMIT` удалить; §8.1:2739 `--reload` — только dev.
- ФТ §12.5. Импорты (анализатор): `backtrader/bt, pandas/pd, numpy/np, ta, datetime, math, decimal, dataclasses, typing, enum, collections, itertools, functools`. При исполнении доступны только `datetime, math, decimal, backtrader`. Атрибуты: `datetime{datetime,date,time,timedelta,timezone,MINYEAR,MAXYEAR}`, `decimal{Decimal,DecimalException,InvalidOperation,DivisionByZero,ROUND_CEILING/DOWN/FLOOR/HALF_DOWN/HALF_EVEN/HALF_UP/UP/05UP}`, `bt{Strategy,indicators,ind,Order,TimeFrame}`, `math` — все публичные. Денилист: `eval, exec, compile, __import__, getattr, setattr, delattr, globals, locals, vars, type, __subclasses__, __builtins__, __loader__, __spec__, breakpoint, exit, quit` + любой dunder (кроме `def __init__`).
- `.env.example` не читал (запрет `.env.*`): оркестратору — `grep AI_DAILY_LIMIT\|AI_MONTHLY_LIMIT\|AI_MODEL .env.example`.
- Новые находки: (1) `HealthWidget.tsx` не знает `"timeout"` и покажет жёлтое «нет данных»; (2) в `AISettingsPage.tsx` несуществующий `claude-sonnet-4-6-20250514`; (3) `Procfile` и `backend/scripts/start_backend.sh` — `--reload` всегда; (4) IPv4-compatible `::a00:1` — `is_global=True`; (5) список импортов анализатора шире, чем пропускает `_safe_import`; (6) `session.close()` после отменённого execute на реально зависшем драйвере может держать ответ.
- Самопроверка: 1 — записей нет; 2 — уведомлений нет; 3 — тесты идут через HTTP-клиент, DNS и pinned-бэкенд, сам `run_uvicorn`; 4 — под таймаутом только SELECT, своё отличаем по `expired()`; 5 — вызывающие `_is_forbidden_ip` (3) ужесточены одинаково; 6 — n/a.

### 7. Gotchas
38, 74, 61, 50 (только worktree).
### 8. Новые — нет.
### 9. Плагины
py_compile после каждого `.py`; `pnpm typecheck` (`tsc -b`); tdd — `mattpocock-skills:tdd`; context7 не понадобился (stdlib `ipaddress`/`asyncio`).

---

## Ревью р.1 — все 8 пунктов закрыты
Статус: ✅ готово к коммиту. Гейты: pytest **5436 passed / 0 xfailed / 0 failed** (1 skipped — был и до пачки); ruff 0; mypy Success (201); bandit M0/H0; typecheck 0; lint 0; build ok; vitest затронутых файлов (HealthWidget, AISettingsPage, SettingsPage.risk) — 18 passed. Полный vitest не запускал (по указанию).

**context7 (aiosqlite, SQLAlchemy 2.0):** у aiosqlite на каждое соединение один поток с общей очередью запросов. Отмена await не прерывает вызов, уже идущий в потоке, а `close()` и rollback ставятся в очередь за ним. SQLAlchemy: конкурентным задачам — отдельная `AsyncSession` на каждую. Вывод ревьюера подтверждён и воспроизведён: прежний код на реальном файловом SQLite ответил через **15,5 с** при пределе 5 с.

1. **/health (031).** Вся работа с БД вынесена в задачу `_health_db_probe` с собственной сессией из `get_db()`: проба, проверка токена, `cb_state`, `ai_configured`. Ответ ждёт её через `asyncio.wait({task}, timeout=5)`.
   - Не дождался → 503, `database:"timeout"`, `ai_configured: null`. `cb_state` оставлен `"unknown"`, а не null: это существующий enum контракта и фронта. Расширенный ответ — по подписи токена, как при лежащей БД.
   - Задача остаётся в фоне: сильная ссылка в `_HEALTH_DB_TASKS`, done-callback забирает исключение и пишет WARNING.
   - Сверх задания: пока прошлая проба висит дольше предела, новая не запускается — ответ сразу `timeout`. Иначе каждый healthcheck добавлял бы ещё один зависший поток.
   - Тест: реальный файл SQLite, соседнее соединение держит `BEGIN EXCLUSIVE`, busy_timeout 15 с. Ответ ≤ 5,5 с, повторный запрос < 1 с без новой пробы.
   - Мутация «вернуть `asyncio.timeout` вокруг execute» → `ответ через 15.5 с`.
2. **`ai_configured`** теперь через `AIService().get_active_config`. Ошибка, в том числе `MultipleResultsFound`, → null + WARNING. RED: `assert True is None`. Inline-импорт убран.
3. **HealthWidget.** `database:"timeout"` → красная строка «БД не отвечает». Для проверки цвета на строку добавлен `data-traffic`. Мутация (жёлтый цвет) → тест красный.
4. **SSRF-барьер (046).** Встроенный IPv4 теперь извлекается и из `::/96` (кроме `::/127`), и из SIIT `::ffff:0:0:0/96`. `::127.0.0.1`, `::10.0.0.1`, `::169.254.169.254`, `::ffff:0:127.0.0.1` запрещены. Мутация → 5 failed.
   - **Для ТЗ (известное ограничение):** «NAT64 с операторским префиксом (RFC 6052 §2.2 / RFC 7050) не распознаётся; целевая платформа — домашняя сеть с IPv4».
5. **Скрипты запуска (033).** В `start.sh` один разборщик `effective_env get|exec` (python-dotenv; окружение важнее `.env`; имена без учёта регистра) и `is_true` (`1/true/t/yes/y/on`, регистр не важен). Им пользуются и `debug_enabled`, и `production_preflight`.
   - `Procfile` и `backend/scripts/start_backend.sh` подключают `start.sh` через `source` и вызывают `run_uvicorn` — третьей копии парсера нет.
   - RED: `debug=true` без reload, `DEBUG=t` → preflight FATAL. Мутация (окружение без перевода имён в верхний регистр) → 2 failed.
6. **AISettingsPage.** Список Claude: `claude-opus-5-5` (по умолчанию; дефолт формы = `MODELS_BY_TYPE.claude[0]`), `claude-sonnet-5`, `claude-haiku-4-5-20251001`. На старом списке тест падает: `expected 'claude-sonnet-4-6-20250514' to be 'claude-opus-5-5'`.
7. **Анализатор: одно обращение — одно сообщение.** Сделано через `report()` с дедупликацией по узлу и порядком правил: функция → имя → атрибут-денилист → dunder → allow-list. Мёртвый `_get_call_name` удалён. Мутация (без дедупликации) → 2 failed.
8. **Белый список импортов** = `IMPORTABLE_MODULES` (`backtrader, datetime, decimal, math`) из `module_proxies.py`.
   - Подмодули, относительные импорты и `bt`/`pandas`/`numpy`/`ta`/`typing`… отклоняются сообщением «Неразрешённый модуль: X (в стратегии доступны только: backtrader, datetime, decimal, math)».
   - Фикстура `tests/strategy_blocks.py::all_block_types_workspace` покрывает все 20 типов `ALL_BLOCKS`, состав сверяет тест. Кодоген (long/up, short/down) проходит анализатор без замечаний и компилируется `_compile_strategy`.
   - Осознанно поправлены `test_safe_code_passes` и `test_allowed_imports` (убраны pandas/numpy). Мутация (вернуть pandas/bt в список) → 9 failed.

**ФТ §12.5 — предлагаемый текст:** «AST-анализ перед исполнением: импорт разрешён только для модулей `backtrader`, `datetime`, `decimal`, `math` — тех, что доступны при исполнении; остальные импорты, подмодули и относительные импорты отклоняются до запуска с перечнем доступных модулей. Атрибуты разрешённых модулей — по белому списку (`datetime`: datetime/date/time/timedelta/timezone/MINYEAR/MAXYEAR; `decimal`: Decimal, исключения, режимы ROUND_*; `backtrader`: Strategy/indicators/Order/TimeFrame; `math` — все публичные функции и константы). Запрещены `eval, exec, compile, __import__, getattr, setattr, delattr, globals, locals, vars, type, __subclasses__, __builtins__, __loader__, __spec__, breakpoint, exit, quit`, обращение к любым dunder-атрибутам и объявление dunder-методов, кроме `__init__`. На одно нарушение — одно сообщение.»

**ТЗ §8.5 (дополнение к разделу 6):** «Вся работа с БД — отдельная задача со своей сессией; ответ ждёт её ≤ 5 с, иначе `database: timeout`, `ai_configured: null`, `cb_state: unknown`; задача доживает в фоне, пока висит дольше предела — новая не запускается.» Гайд (§3.1 / нативный запуск): `start.sh`, `Procfile`, `start_backend.sh` — `--reload` только при DEBUG=true.

**Самопроверка р.1:** записей в БД и уведомлений нет. Таймаут не отменяет работу драйвера. Все вызывающие проверены: `_health_viewer` — только проба; `_is_forbidden_ip` — 3 точки, все ужесточены одинаково; анализатор — только `_compile_strategy` (бэктест и grid). Остаток: при остановке процесса зависшая фоновая проба будет уничтожена вместе с loop'ом — допустимо.

---

## Ревью р.2 — все 8 пунктов закрыты
Статус: ✅ готово к коммиту. Гейты: pytest **5446 passed / 0 xfailed / 0 failed** (1 skipped — был и до пачки; в логе нет «never retrieved» / «Task was destroyed»); ruff 0; mypy Success (201); bandit M0/H0; typecheck 0; lint 0; build ok. В р.2 фронт не менялся, vitest не запускал.

1. **Предел пробы `_READINESS_TIMEOUT_SEC = 3`** (комментарий с обоснованием). Тест `test_readiness_deadline_fits_container_healthcheck` читает `timeout` healthcheck из `docker-compose.yml` и `Dockerfile.backend` (оба 5 с) и требует запас ≥ 1,5 с. RED при 5: `assert 6.5 <= 5`. Тест на реальном SQLite: ответ ≤ предел + 0,5 с.
2. **Потолок зависшей пробы `_HEALTH_PROBE_STUCK_MAX_SEC = 60`.**
   - Висящая проба моложе потолка → сразу `timeout`, новой пробы нет.
   - Старше потолка → одна новая; старая остаётся в реестре.
   - Тесты: «старше потолка → 200, новая сессия открыта, старая жива» и «моложе → 503, сессий 0». Мутация (убрать верхнюю границу) → `assert 503 == 200`.
3. **WARNING `health_db_probe_failed_unretrieved` для любой незабранной пробы.** Реестр хранит запись `_HealthProbe(started, awaited, taken)`, множество `_HEALTH_ABANDONED` убрано.
   - Если обработчик ещё ждёт, исключение забирает он.
   - Если обработчик ушёл (таймаут или отмена), пишет done-callback.
   - Если отмена пришла уже после завершения пробы, пишет `finally` обработчика. Итог — ровно одна запись.
   - Тест: отмена клиентского запроса, затем проба падает `AttributeError` → одна WARNING. Мутация (callback без лога) → `assert 0 == 1`.
4. **Ветка timeout:** `ai_configured = True if settings.AI_API_KEY else None`. RED: `assert None is True`; мутация → то же.
5. **Один источник модулей.** `module_proxies.build_strategy_modules()` собирает прокси исполнителя и падает с `RuntimeError`, если набор ≠ `IMPORTABLE_MODULES`; `_compile_strategy` берёт модули оттуда.
   - Тест сверяет анализатор с реальными ключами исполнителя. Второй тест через подмену доказывает, что `import` в стратегии получает именно эти модули.
   - Мутация (прокси `cmath` в `build_stdlib_proxies` без записи в списке) → `RuntimeError: … расходятся с IMPORTABLE_MODULES: ['cmath']`, 4 failed.
6. **`start.sh`: сбой чтения настроек — явная ошибка.** `effective_env` ловит отсутствие dotenv и нечитаемый `.env` и выходит с ненулевым кодом. `debug_enabled` проверяет код возврата → «FATAL: не удалось прочитать настройки … — запуск остановлен», `exit 1`.
   - Тест: нет python / `.env` с правами 000 × `run_uvicorn` / `production_preflight` → код ≠ 0, uvicorn не запущен.
   - RED: `app.main:app --port 8000`, код 0 — раньше молча стартовал в production. Мутация → 4 failed.
7. **Autouse-фикстура `_clean_health_probe_registry`** в `test_health.py`: очищает реестр до теста; после теста отменяет и дожидается незавершённые пробы. Сделал её локальной для файла, а не в общем conftest: зависшие пробы создают только эти тесты, а async-autouse в корневом conftest задел бы весь набор.
8. **Docstring `_is_forbidden_ip` переписан по факту, поведение не менял. Расхождение с ревью:** в рантайме проекта (venv Python 3.11.15) `2002::/16` (6to4) — `is_global=False`, проверено: `ipaddress.ip_address('2002:808:808::1').is_global → False`; Teredo и `64:ff9b:1::/48` — тоже False.
   - В docstring записано: реестр спецадресов `ipaddress` менялся между патч-версиями 3.11; на 3.11.15 эти префиксы запрещены и со встроенным публичным IPv4. Если где-то префикс окажется глобальным, встроенный IPv4 всё равно проверяется.
   - Тест `test_embedded_public_ipv4[6to4/teredo/nat64-local]` зависит от этой версии; CI (`python-version: '3.11'`, свежий патч) и образ `python:3.11-slim` дают тот же результат.
