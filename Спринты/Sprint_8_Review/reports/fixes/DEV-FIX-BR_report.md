## DEV-FIX-BR отчёт — S8R fixes, FIX
Статус: ✅ готово к коммиту — 030, 053, 045, 044 | ⏸ 032: предпосылка уже снята S8R-AUDIT-048, добавлен только охранный тест

### 1. Что реализовано
- **030**: в `tinvest/errors.py` одна таблица `GRPC_CODE_CATEGORIES` (17 `StatusCode` → `GrpcCodeCategory`) плюс наборы категорий для решений. По ней работают `classify`, `is_definitive_rejection`, `is_broker_order_rejection` (из `sandbox_recovery` таблица импортируется внутри функций, иначе получается цикл импортов) и `multiplexer._is_terminal_auth_error` (разбор кода — общими `iter_exception_chain`/`grpc_code_name`). Своих наборов кодов у потребителей больше нет. Поведение не изменилось: решения по всем 17 кодам совпадают с литеральной таблицей, снятой до правки.
- **032 (⏸)**: на сверке `get_real_positions(fetch_instrument_info=False)` вызывается с 01.10 (048, `5325063`), то есть попытка сверки и так равна одному чтению портфеля. Учёт вложенных вызовов в оценке был бы мёртвым кодом. Добавлен только охранный тест на уровне рантайма. **Предложение**: закрыть карточку как дубль 048.
- **053**: параметр `currency` убран из `TInvestAdapter.get_sandbox_balance`, из протокола `SandboxCapableAdapter`, из `top_up_sandbox_to`, из `SandboxTopUpRequest` и из TS-интерфейса `SandboxTopUpRequest`. Валюта задана константой `_SANDBOX_CURRENCY="rub"`. Лишний `currency` в теле или в query игнорируется. Фронт `currency` и раньше не передавал.
- **045**: `_persist_instrument_cache` пишет через `persist_with_retry_fresh` (3 попытки, шаг 0,05 с, суммарные паузы ≤ 0,15 с). Финальная неудача даёт WARNING, исключения нет.
- **044**: снапшот собирается в `_build_snapshot` двумя агрегатами по `IN` вместо 2·N запросов. У `_load_user_sessions` стоит `raiseload("*")`: вся история сделок через `selectin` больше не тянется. На одно подключение — 7 SELECT при любом N (4 из них — авторизация). Раньше было 16 при N=1 и 48 при N=5.

### 2. Файлы
Изменены: `app/broker/{router,schemas,service,sandbox_recovery}.py`, `app/broker/tinvest/{errors,multiplexer,adapter}.py`, `app/common/exceptions.py` (только устаревший комментарий), `app/market_data/service.py`, `app/trading/ws_sessions.py`, `tests/test_trading/test_reconcile_incomplete_portfolio.py` (+1 тест), `frontend/src/api/brokerApi.ts`.
Новые: `tests/unit/test_broker/test_grpc_code_categories.py`, `tests/test_routers/test_sandbox_currency.py`, `tests/test_market_data/test_instrument_cache_persist_retry.py`, `tests/test_trading/test_ws_sessions_snapshot_queries.py`.

### 3. Тесты
| Карточка | RED | GREEN | Мутация → падение |
|---|---|---|---|
| 030 | `AttributeError: module 'app.broker.tinvest.errors' has no attribute 'GRPC_CODE_CATEGORIES'` (19 из 37) | 37 | свой набор в мультиплексоре → `At index 4 diff: True != False` |
| 032 | — (предпосылка снята) | 1 | `fetch_instrument_info=True` в runtime → `assert ['RUB000UTSTO...BBG004730N88'] == []` |
| 053 | `assert 'usd' == 'rub'`; `Failed: DID NOT RAISE <class 'TypeError'>` | 4 | вернуть `currency` у адаптера → `DID NOT RAISE` |
| 045 | `assert 1 == 2` (opened) | 2 | `attempts=1` → `assert 1 == 2` |
| 044 | `AssertionError: (16, 48)` (содержимое снапшота уже совпадало) | 2 | снапшот по одной сессии → `assert 15 == 7` |

Гейты: pytest 5336 passed / 0 xfailed / 0 failed / 1 skipped (`-o faulthandler_timeout=240`); ruff 0; mypy Success (200); bandit M0/H0; typecheck 0; lint 0; build ok. Полный vitest не запускался, оставлен оркестратору; затронутые файлы — 8 файлов, 32 passed. Маркеров `S8R-FIX-BR` нет.

### 4. Integration points
✅ `errors.py:107–191` (таблица) ← `sandbox_recovery.py` (`is_definitive_rejection`, `is_broker_order_rejection`), `multiplexer.py:49` (`_is_terminal_auth_error`), `classify`
✅ `ws_sessions.py:197` (`_build_snapshot`)
✅ `market_data/service.py:932` (`persist_with_retry_fresh`)
✅ `router.py:138` (`top_up_sandbox_to` без `currency`)

### 5. Контракты
`POST …/sandbox-topup` — поле `currency` удалено, лишнее поле игнорируется. Ответ и WS-снапшот без изменений. Миграции нет.

### 6. Проблемы / TODO / правки документов
- **Риск 045**: каждая попытка может ждать `busy_timeout` 30 с, поэтому худший случай на горячем пути — 90 с вместо прежних 30. Предложение: отдельный короткий `busy_timeout` для сессии кэша (решать оркестратору).
- **044**: сбой агрегата теперь даёт пустой снапшот целиком, а раньше выпадала только одна сессия.
- **ТЗ §9.x, Фаза 1** (строка «Запись вне бэктеста…»), дописать: «Кэш справочника инструментов (`_persist_instrument_cache`) — `persist_with_retry_fresh`, 3 попытки с шагом 0,05 с, best-effort (WARNING без исключения)».
- **ТЗ §11.6 C1**, п. 1, дописать: «снапшот — фиксированным числом запросов (агрегаты по `IN`), независимо от числа сессий».
- **ФТ** (строка Sandbox balance), дописать: «баланс и пополнение песочницы — в рублях; `currency` в запросе пополнения не принимается (игнорируется)».
- Самопроверка: 1 — запись best-effort, статусы не меняются; 2 — уведомлений нет; 3 — тесты идут через SDK-ошибки, HTTP, WS и реальную БД; 4 — повтор 045 не под `wait_for`; 5 — вызывающие `get_sandbox_balance` (4 шт.) и классификаторов проверены; 6 — n/a.

### 7. Применённые Stack Gotchas
37 (свежая сессия на попытку), 48 (файловая БД + NullPool для WS), 56, 74.

### 8. Новые Stack Gotchas
Кандидат: `lazy="selectin"` у `TradingSession`/`LiveTrade` каскадно умножает SELECT при любой выборке сущностей (снапшот WS: 8 запросов на сессию). Правило: в агрегатах выбирать колонки или ставить `raiseload("*")`.

### 9. Плагины
py_compile после каждой правки; typecheck (`tsc -b`); TDD — `mattpocock-skills:tdd`; context7 не понадобился (API SQLAlchemy — `raiseload`/`func.count` — стандартный).

### Ревью р.1 (оркестратор)
- **032** принята как дубль 048, охранный тест оставлен. **044** принята. Сбой агрегата снапшота теперь логируется как ERROR `trading_sessions_ws_serialize_failed` с полями `error_type`, `error` и `session_ids`; раньше это был warning.
- **045, короткий `busy_timeout`**: в `_write` первым выполняется `PRAGMA busy_timeout=2000` (`_CACHE_PERSIST_BUSY_TIMEOUT_MS`), но только если пул движка сессии — `NullPool` (`cache_db.get_bind().engine.pool`; `.engine`, потому что mypy знает тип только как `Engine | Connection`). У StaticPool и других переиспользуемых пулов PRAGMA не выполняется. Худший случай горячего пути — около 3 × 2 с + 0,15 с ≈ 6,2 с; это записано в комментарии к константам и в докстринге.
- **Тесты** (+2 в `test_instrument_cache_persist_retry.py`): событие движка читает `busy_timeout` соединения в момент UPDATE кэша. При NullPool значение `[2000]`, при StaticPool — не 2000.
  - RED без PRAGMA: `assert [5000] == [2000]`.
  - Мутация «без проверки пула» → `assert 2000 != 2000`, после чего откачена (md5 сверен).
- **Гейты**:
  - market_data + figi + pool: 314 passed;
  - полный pytest (`-o faulthandler_timeout=240`, в лог): 5338 passed / 0 xfailed / 0 failed / 1 skipped;
  - ruff 0; mypy Success (200) — одну ошибку union-attr исправил до финального прогона; bandit M0/H0;
  - фронт в этом раунде не менялся.
- **Предлагаемая правка ТЗ** «Фаза 1 (SQLite)», вместо прежнего предложения: «Кэш справочника инструментов (`_persist_instrument_cache`) — `persist_with_retry_fresh`, 3 попытки с шагом 0,05 с; на соединении сессии кэша при `NullPool` — `busy_timeout` 2 с (худший случай ≈ 6,2 с); best-effort (WARNING без исключения)».

### Ревью р.2 (оркестратор)
**1. 045: отложенная запись** (короткий `busy_timeout` р.1 убран вместе с проверкой пула, `_CACHE_PERSIST_BUSY_TIMEOUT_MS` и тестами PRAGMA).
- `_persist_instrument_cache` больше не ждёт записи: она ставится фоновой задачей через `_schedule_instrument_cache_write`.
- На каждый тикер — одна задача-писатель (`_CACHE_WRITERS`). Если запись тикера уже идёт, новые поля сливаются поверх ожидающих (`_CACHE_PENDING`, позднее значение побеждает), и писатель дописывает их следующим проходом.
- Сильные ссылки на задачи — в `_CACHE_WRITE_TASKS`; `add_done_callback` убирает завершённые и пишет ERROR `instrument_cache_writer_failed`, если задача упала.
- Сама запись — `persist_with_retry_fresh`: 3 попытки, шаг 0,2 с, обычный `busy_timeout`. Если все попытки неудачны — WARNING `instrument_cache_persist_failed`.
- `drain_instrument_cache_writes(timeout) -> bool`: True — все записи завершились. По таймауту незавершённым отправляется отмена, ждать её завершения функция не стала: запись может стоять в `busy_timeout` до 30 с. Возвращает False и пишет WARNING `instrument_cache_drain_timeout`.
- Точка подключения в shutdown: `app/main.py`, lifespan, `finally` — после ожидания outbox и перед `close_db()`, таймаут `INSTRUMENT_CACHE_DRAIN_TIMEOUT_SEC = 5`.
- Сброс между тестами: `tests/conftest.py`, autouse `_reset_figi_negative_cache` → `_reset_instrument_cache_writes_for_tests()`.
- Девять существующих тестов проверяют БД сразу после резолва — им добавлен `await drain_instrument_cache_writes(5.0)`.
- Тесты (`test_instrument_cache_persist_retry.py`, 5 шт., реальный SQLite; write-лок держит отдельное соединение `BEGIN IMMEDIATE`):
  - вызывающий возвращается быстрее 0,5 с при занятом локе, запись доезжает после освобождения;
  - порядок двух записей одного тикера (первая медленная) → 20;
  - «locked» на первой попытке → записано со второй;
  - постоянный «locked» → 3 попытки и WARNING, вызывающий не ждал;
  - `drain` по таймауту → False быстрее 0,5 с, задача отменена, отменённая запись не доехала.
- RED: `ImportError: cannot import name 'drain_instrument_cache_writes'`.
- Мутации:
  - синхронная запись → `assert 3.9460677090100944 < 0.5`;
  - отдельная задача на каждый вызов → `assert (10, 'BBG004730N88') == (20, 'BBG004730N88')`.

**2. Кто держит write-лок дольше 30 с** (только отчёт, код не менял)
- **Где**: `app/market_data/service.py:1219–1250` `_purge_iss_cache`, вызов на `:1111–1112` в `get_candles` (при наличии T-Invest — в любом режиме). DELETE ISS-строк выполняется на сессии вызывающего, а commit — только если `deleted > 0`.
- **Почему держит**: DELETE без единой подходящей строки всё равно открывает пишущую транзакцию. Проверено на sqlite3 в WAL: после DELETE на 0 строк `in_transaction=True`, второй писатель получает «database is locked», после rollback — ок.
- **Сколько держит в бэктесте**: сессия `_run_backtest_task` (`app/backtest/router.py:343`) держит лок
  - от purge через всю загрузку пропусков с T-Invest (сеть) до первого commit пачки в `_save_to_cache` (`service.py:1700`);
  - при тёплом кэше — через чтение кэша, проверку качества OHLCV, прогон Backtrader в executor (`engine.py:501`), IMOEX и до commit в `_save_result` (`backtest/service.py:217`).
- **Оценка на годе минуток** (~250 дней × ~840 баров ≈ 210 тыс.):
  - холодный кэш: сеть — сотни запросов `GetCandles` (десятки секунд, обычно больше минуты), всё это время под локом; потом пачки по 2000 строк по ~0,5–1 с с освобождением между ними;
  - тёплый кэш: чтение ~210 тыс. строк (секунды) плюс Backtrader (≈ 0,5–2 мин) — лок держится всё это время.
- **Совпадает с живым логом**: `trying_tinvest` пишется в `_fetch_candles` (`:1493`). Резолв FIGI там же писал кэш другой сессией и 30 с ждал лок, который держала сессия самого вызывающего после purge, то есть фактически ждал сам себя.
- **Тот же риск для торговли**: `get_candles(mode="trading")` (история при старте и восстановлении сессии) держит лок на время сетевой догрузки. В эти секунды конкурентно падают commit записей сделок, уведомлений и CB.
- **Предложение**: после DELETE всегда завершать транзакцию (commit при `deleted > 0`, иначе rollback/commit), либо выполнять purge в своей короткой сессии с commit. Попутно (низкий приоритет): `_save_to_cache` пишет по одной строке на statement; пакетный `insert … values` сократит время каждой пачки под локом.

**3. 053**
- `sandbox_pay_in` в адаптере и протоколе `SandboxCapableAdapter` — без `currency`, валюта = `SANDBOX_CURRENCY` (`app/broker/base.py`).
- `currency` ≠ rub → 422 с текстом `SANDBOX_CURRENCY_ERROR` («Песочница T-Invest работает только в рублях…»):
  - в теле — валидатор `SandboxTopUpRequest`;
  - в query у обоих эндпоинтов — `_reject_non_ruble`.
- `rub` в любом регистре или отсутствие параметра — ок.
- Тесты: 15 шт. (параметризованные). RED — `assert 200 == 422` и `'currency' not in {…'currency': 'rub'}`. Мутация «валидатор пропускает» → `assert 200 == 422`.

**4. 030**
- Таблица, категории, `grpc_code_category` и константа `TERMINAL_STREAM_CODE_NAMES` перенесены в `app/broker/grpc_codes.py` — модуль без зависимостей.
- `errors.py`, `sandbox_recovery.py`, `multiplexer.py` импортируют его сверху; ленивые импорты и `codes_in` удалены.
- Тест «потребители следуют таблице» использует ошибку только со структурным кодом; строковый fallback проверяется отдельно — константа равна AUTH-кодам таблицы.
- Мутация (свой набор в мультиплексоре) → `At index 4 diff: False != True`.

**5. 044**
- При сбое агрегата сессии отдаются с `positions: null, last_trade_at: null`, ERROR остаётся. Фронт эти поля не читает: мерж `applySnapshot` не ломается.
- Тест: сбой → строки с null, ровно один ERROR с текстом причины. Мутация `snapshot_payload = []` → `assert [] == [{…}]`.

**6.** Псевдоним `fields = update_fields` удалён: запись вынесена в `_write_instrument_row`.

**Гейты**: полный pytest (`-o faulthandler_timeout=240`, в лог) — 5349 passed / 0 xfailed / 0 failed / 1 skipped, «Task was destroyed» — 0; ruff 0; mypy Success (201); bandit M0/H0; typecheck/lint/build — ok. Фронт в р.2 не менялся, vitest не нужен.

**Предлагаемые правки документов вместо р.1:**
- **ТЗ «Фаза 1 (SQLite)»**: «Кэш справочника инструментов (`_persist_instrument_cache`) пишется фоновой задачей вне горячего пути (одна на тикер, последнее значение побеждает): `persist_with_retry_fresh`, 3 попытки, best-effort (WARNING). При остановке — ожидание до 5 с (`drain_instrument_cache_writes`)».
- **ФТ/ТЗ, sandbox**: «баланс и пополнение песочницы — только в рублях; `currency` ≠ `rub` в запросе — 422».
- **ТЗ §11.6 C1**: «при сбое сводки по сделкам сессии отдаются с `positions`/`last_trade_at` = null».

### Ревью р.3 (оркестратор)
**1. S8R-FIX-064 (новая): purge ISS-кэша не держит write-лок на сессии вызывающего.**
- `_purge_iss_cache` (`app/market_data/service.py`) теперь выполняет DELETE в своей короткой сессии через `persist_with_retry_fresh`: commit всегда, в том числе при 0 удалённых; 3 попытки с паузами 0,5 и 1 с, финальная ошибка пробрасывается, как и раньше.
- Фабрика — новый `_own_write_session_factory()`: инжектированная, иначе сессия на движке `self.db`, иначе общая. Комментарий в `get_candles` про «commit только если удалили» исправлен.
- Почему этого достаточно: SQLAlchemy/pysqlite выполняет SELECT в режиме автокоммита и снимок чтения не держит. Поэтому сессия вызывающего после чистки в другой сессии видит свежие данные, и её последующий INSERT в `_save_to_cache` открывает новую пишущую транзакцию — конфликта снимков нет.
- **Тест** `tests/test_market_data/test_purge_releases_write_lock.py`: файловый SQLite в WAL, PRAGMA как в `init_db`; параметры — режим (backtest, trading) × purge (0 и 3 строки). После `get_candles`, пока сессия вызывающего открыта, второй писатель `sqlite3` с `timeout=0.5` и `BEGIN IMMEDIATE` пишет сразу; ISS-строки вычищены.
  - RED: `assert 'database is locked' == 'ok'` в случаях с 0 строк.
  - Мутация (purge на сессии вызывающего без commit) → `'database is locked' == 'ok'`.
- Два старых теста повтора чистки (`TestPurgeIssCacheRetry` в `test_service_full.py`) патчили `execute` сессии вызывающего; переписаны на фабрику: «locked» на первой сессии → 2 сессии, прочая ошибка пробрасывается. Повтор теперь по стандарту проекта (`is_database_locked`: `OperationalError`/`PendingRollbackError`), а не по тексту любого исключения.
- **DML на сессии вызывающего в путях `get_candles`** (backtest, trading, prefetch, роутеры):
  - `_purge_iss_cache` — исправлено;
  - `_save_to_cache` (`service.py`, пачки по 2000, commit на каждой и в конце) — сети внутри транзакции нет, лок держится ~0,5–1 с на пачку; не менял. Две оговорки: (а) commit фиксирует и чужие незакоммиченные правки сессии вызывающего — сейчас в бэктесте (`router.py:343`) и `runtime._preload_history` (своя сессия) их нет; (б) при исключении посреди пачки транзакция остаётся открытой до отката вызывающим: бэктест откатывает в except, у runtime сессия закрывается сразу;
  - прочих DML на `self.db` в `MarketDataService` нет: лот, FIGI и логотип пишутся своей сессией (`_write_instrument_row`). Бэктест до `_save_result` на своей сессии не пишет; `_run_parity_check` коммитит отдельно.
- Пакетная вставка в `_save_to_cache` не сделана: нетривиальна. Для источника `tinvest` нужен `on_conflict_do_update` через `excluded.*`, а смена семантики конфликта требует отдельного теста. Предложение — отдельная low-карточка: `insert().on_conflict…` с executemany по пачке.

**2. 045 — откат фоновой записи.**
- Удалены задачи-писатели, `_CACHE_*`, `drain_instrument_cache_writes`, шаг в lifespan `main.py`, сброс в conftest, drain в 9 тестах (эти файлы возвращены к исходному состоянию) и тесты фоновой записи.
- Запись синхронная: `_write_instrument_row` → `persist_with_retry_fresh`, attempts=2, шаг 0,2 с, обычный `busy_timeout`; финальная неудача — WARNING.
- Тесты без порогов реального времени: «locked» на первой попытке → записано со второй (`opened == 2`); постоянный «locked» → 2 попытки, ровно один WARNING, без исключения. Мутация `attempts=1` → `assert 1 == 2`.

**3. 053.**
- Query-параметр `currency` убран у обоих эндпоинтов: `?currency=` игнорируется, в OpenAPI его нет.
- В теле `currency` необязательное, `description` «Только «rub»…», `examples=["rub"]`; иное значение → 422 с текстом `SANDBOX_CURRENCY_ERROR`.
- Тесты: `?currency=usd` → 200 и `rub`; в OpenAPI нет query-параметра `currency`, а у поля тела есть описание «rub» и оно необязательное. Мутация валидатора → `assert 200 == 422`.

**4. 044.** Новая `_load_snapshot`: список сессий и сводка по сделкам читаются в одной сессии БД, которая закрывается до пересылки событий. Запасной путь с `null` сохранён. Тест: на подключение 2 сессии фабрики (авторизация + снапшот). Мутация «сводка в отдельной сессии» → `assert 3 == 2`; мутация «пустой запасной путь» → красный.

**Гейты**: полный pytest (`-o faulthandler_timeout=240`, лог в файл) — 5349 passed / 0 xfailed / 0 failed / 1 skipped; ruff 0; mypy Success (201); bandit M0/H0; typecheck/lint/build — ok. Фронт в р.3 не менялся.

**Тексты для документов:**
- **ТЗ §5.6 (рыночные данные, кэш свечей)**: «Перед загрузкой с T-Invest ISS-строки пары чистятся в отдельной короткой сессии с commit всегда (`S8R-FIX-064`): сессия вызывающего (бэктест, торговля) пишущую транзакцию SQLite не получает и write-лок на время сетевой догрузки не держит».
- **ТЗ «Фаза 1 (SQLite)»**, вместо прежних предложений для 045: «Кэш справочника инструментов — синхронно в своей сессии (`persist_with_retry_fresh`, 2 попытки), best-effort (WARNING)».
- **ФТ/ТЗ, sandbox**: «баланс и пополнение — в рублях; поле `currency` в теле пополнения — только `rub`, иное — 422; query-параметра валюты нет».
- **ТЗ §11.6 C1**: «снапшот — одна сессия БД и фиксированное число запросов; при сбое сводки — `positions`/`last_trade_at` = null».
- **Гайд §7**: миграций нет, правка не нужна.

### Ревью р.4 (оркестратор, финальный)
1. **064, порядок фабрик.** `_own_write_session_factory` теперь выбирает так:
   - сначала сессию на движке `self.db.bind` — ту же БД, что читает вызывающий;
   - инжектированную `session_factory` — только если `bind` недоступен;
   - если нет ни того, ни другого — возвращает `None`, и `_purge_iss_cache` пишет WARNING `iss_cache_purge_no_factory` и выполняет DELETE на `self.db` через `persist_with_retry(rollback_first=False)` с commit: транзакция завершается в любом случае.

   Докстринги приведены к факту.
2. **064, ветка `bind`.** Тест `test_bind_branch_without_injected_factory`: файловый SQLite в WAL, `MarketDataService(db)` без фабрики, чистка 0 и 3 строк. Второй писатель (`timeout=0.5`) пишет сразу, WARNING `no_factory` нет. Мутация «ветка bind → None» → `assert not [{'event': 'iss_cache_purge_no_factory', …}]`.
3. **064, `SELECT EXISTS` перед DELETE.** Проверка идёт чтением на сессии вызывающего; write-лок она не берёт, потому что pysqlite выполняет SELECT в автокоммите. Если удалять нечего — выход, пишущая сессия не открывается. Тест `test_no_iss_rows_no_write_session` считает новые соединения (`NullPool`, событие `connect`) и DELETE: без ISS-строк — 0 и 0, с ISS-строками — 1 и 1. Мутация «без EXISTS» → `assert 1 == 0`. Мутация «purge на `self.db` без commit» остаётся красной в случаях с ISS-строками (`'database is locked' == 'ok'`). Тесты повтора чистки (`TestPurgeIssCacheRetry`) переведены на подмену `_own_write_session_factory` и засев ISS-строки.
4. **045 — код возвращён к базе `90f0736`.** `_persist_instrument_cache`, одна попытка, побайтово как в базе: в diff с `90f0736` этой функции нет. Тесты 045 удалены (`test_instrument_cache_persist_retry.py`). **Карточка S8R-FIX-045 закрывается через корень S8R-FIX-064**: «database is locked» на записи кэша справочника был следствием того, что чистка ISS-кэша оставляла пишущую транзакцию на сессии бэктеста или торговли на всю сетевую догрузку свечей. В пути из лога (`trying_tinvest` → резолв FIGI → запись кэша) запись ждала лок, который держала сессия самого вызывающего. С исправлением 064 длинного держателя лока на этом пути нет. Повтор записи кэша не нужен; предложенные в р.1–р.3 правки ТЗ для 045 снимаются.
5. **044, `_load_snapshot`.** Результат собирается в переменную внутри `async with` и возвращается после закрытия сессии. Если падает закрытие, а снапшот уже готов, — WARNING `trading_sessions_ws_session_close_failed`, снапшот не подменяется. Если сессию не удалось открыть — ERROR и строки с null. Запасной путь с null вынесен в `_null_rows`. Тест `test_session_close_failure_keeps_ready_snapshot`: закрытие сессии снапшота бросает исключение, а снапшот совпадает с ожидаемым; один WARNING, ERROR нет. Мутация «сбой закрытия как раньше (null-строки)» → строки не совпадают.
6. **053.** У `currency` в `SandboxTopUpRequest` убран `max_length=16`: любое значение, отличное от rub, получает текст `SANDBOX_CURRENCY_ERROR`. Тест — значение из 200 символов → 422 «только в рублях». Мутация (вернуть `max_length`) → `string_too_long` вместо нашего текста. Оговорка: длина поля теперь ограничена только общим лимитом размера тела запроса.

**Гейты**: полный pytest (`-o faulthandler_timeout=240`, в лог) — 5353 passed / 0 xfailed / 0 failed / 1 skipped; ruff 0; mypy Success (201); bandit M0/H0; typecheck/lint/build — ok. Фронт в р.4 не менялся.

**Итог по картам пачки**:
- 030 — готова к коммиту;
- 032 — дубль 048, охранный тест;
- 053 — готова к коммиту;
- 045 — закрыта через 064, код записи кэша не изменён;
- 044 — готова к коммиту;
- 064 (новая) — готова к коммиту.

**Тексты для документов (финальные):**
- **ТЗ §5.6**: «Перед загрузкой с T-Invest ISS-строки пары проверяются `SELECT EXISTS`. Если они есть, удаляются в отдельной короткой сессии на том же движке с commit (`S8R-FIX-064`). Сессия вызывающего (бэктест, торговля) пишущую транзакцию SQLite не получает и write-лок на время сетевой догрузки не держит».
- **ФТ/ТЗ, sandbox**: «баланс и пополнение — в рублях; поле `currency` в теле пополнения — только `rub`, иное — 422; query-параметра валюты нет».
- **ТЗ §11.6 C1**: «снапшот — одна сессия БД и фиксированное число запросов; при сбое сводки — `positions`/`last_trade_at` = null».
