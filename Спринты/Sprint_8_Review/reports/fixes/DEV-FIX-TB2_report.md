## DEV-FIX-TB2 отчёт — S8R fixes, FIX
Статус: ✅ готово к коммиту (worktree `wt-s8r-fixes-b`, база `5ff0a30`, ничего не закоммичено)

### S8R-FIX-067 — дозапрос типа в скане облигаций
1. `_bond_tickers_with_open_positions` теперь выбирает ещё FIGI справочника и сделки, счёт сессии и владельца сессии (outer join, чтобы ни одна позиция не выпала из выборки); тип берётся через `_backfill_unknown_types`.
2. `_backfill_unknown_types`: тикеры с `unknown` или пустым типом → `ensure_instrument_types` (тот же кэш промахов). Токен: у sandbox/real — счёт сессии, у paper — счёт владельца; без счёта сети нет. Бюджет на прогон общий: `BOND_SCAN_TYPE_BACKFILL_MAX_REQUESTS=20` и дедлайн 15 с, как у налогового отчёта. Сбой → warning, отбор по эвристике, как раньше.
3. Сетевой вызов идёт в `_fetch_iss`, то есть до `persist_with_retry`.
- RED: `AssertionError: assert [] == ['BBGZBOND01']`. GREEN: 6/6 (`test_fix067_bond_type_backfill.py`).
- Мутация: в отборе игнорировать дозапрошенный тип → 3 теста красные (`assert [] == ['ZBOND01']`), откат через бэкап, md5 совпал.

### S8R-FIX-068 — флейк `test_extension_between_read_and_lock_skips_pass`
1. Нашёл две причины, обе дают ту же строку, что в прогоне DEV-FIX-TB: `AssertionError: assert not True … Task finished … result=[]`. Обе воспроизведены временным тестом-харнессом (удалён, копия — `s8r-evidence/fixes/repro_FIX-068_harness.py.txt`):
   - цикл событий стоит дольше `COMMISSION_LOCK_TIMEOUT_SEC`;
   - лок `("close_trade", 1)` от прошлого теста ещё не собран и привязан к чужому event loop (gotcha-71).
2. Правка теста: рандеву через `asyncio.Event` (обёртка над `keyed_lock`) и `asyncio.wait(FIRST_COMPLETED)`. Таймаут лока в тесте `None`, перед захватом `gc.collect()`, итог через `await task`. Код продукта не менялся. После правки харнесс зелёный.
- Мутация: снять сверку снимка в `apply_commission_states` → `assert [{… 'new': '0.30' …}] == []`, откат, md5 совпал.
- 20 прогонов подряд под нагрузкой (параллельно шёл полный pytest): 20/20.

### Гейты
- pytest (`faulthandler_timeout=240`): **5533 passed / 1 skipped / 0 xfailed / 0 failed** (`pytest_TB2_run2.log`).
- Первый прогон, шедший вместе с 20 повторами: 1 failed **вне пачки** — `test_position_sizing_percent_base.py::test_adapter_released_when_cancelled_waiting_trade_lock` (`InvalidRequestError: session is in 'prepared' state`). Изолированно 5/5 и файл 29/29 зелёные.
- ruff 0; mypy Success (204); bandit M0/H0; typecheck 0; lint 0; build ok.
- vitest: фронт не менялся — на уровне пакета.
- Маркеров xfail с `S8R-FIX-TB2` нет.

### Файлы
- Изменены: `backend/app/corporate_actions/service.py`, `backend/tests/test_trading/test_commission_settle.py`.
- Новый: `backend/tests/unit/test_corporate_actions/test_fix067_bond_type_backfill.py`.

### Integration points
✅ `service.py:731` (`_backfill_unknown_types`) ← `:973` (`_fetch_iss`) ← scheduler `detect_corporate_actions` и `POST /detect` (admin). ✅ `:799` → `MarketDataService.ensure_instrument_types`. API, схемы, миграции не менялись.

### Предлагаемые правки документов
ТЗ §5.12, новый абзац: «S8R-FIX-067: перед отбором облигаций для `bondization` тикеры открытых позиций с типом `unknown`/пустым дозапрашиваются через `ensure_instrument_types`: токен — счёта сессии (paper — счета владельца), бюджет `BOND_SCAN_TYPE_BACKFILL_MAX_REQUESTS=20` / 15 с, промахи — общий кэш, сеть — в `_fetch_iss` вне единицы записи. Сбой — отбор по эвристике тикера.»

### Новые находки
- `test_adapter_released_when_cancelled_waiting_trade_lock` — тот же класс, что 068: опрос по реальному времени (200×10 мс) через ту же `AsyncSession`, которой пользуется задача. Кандидат в карточку.
- Gotcha-71 подтверждена ещё одним случаем: тесты, где лок берут вручную, должны делать `gc.collect()`.

### Самопроверка
Новых записей статуса и уведомлений нет. Тесты 067 идут через `detect_corporate_actions` до SDK. Под дедлайном только сеть. Бюджет ограничен.

### Плагины
py_compile, tdd (скилл), typecheck. context7 не понадобился: новых API нет.

---

## Ревью р.1 (оркестратор) — все 7 пунктов и флейк закрыты
Статус: ✅ готово к коммиту

### Что сделано
1. **П.1, бюджет по реальным запросам.** Новый `MarketDataService.backfill_instrument_types` возвращает `InstrumentTypeBackfill(types, requests)`. `ensure_instrument_types` стал тонкой обёрткой, поэтому налоговый отчёт работает без изменений. Скан списывает бюджет по `requests`, а FIGI из кэша промахов бюджет не занимают.
2. **П.2, выбор счёта.** Вызов идёт с `user_id` владельца и `preferred_account_id` = счёт сессии. Как и у остальных читателей данных: сначала production (gotcha-57), деактивированный счёт сессии заменяется активным счётом владельца. Сессия без владельца — без сети.
3. **П.3, остаток дедлайна.** Добавлена константа `BOND_SCAN_TYPE_BACKFILL_MIN_SLICE_SEC=1.0`. Её проверяет и скан перед каждым вызовом, и сам метод перед каждым запросом (`min_request_slice_sec`, по умолчанию 0 — в налоговом пути поведение прежнее). Если остатка меньше, FIGI не трогается и не помечается промахом.
   - Запись в справочник: `persist_until` — после общего дедлайна запись не начинается.
   - Все строки пишутся одной транзакцией (`_persist_instrument_rows`) — одно ожидание write-лока вместо ожидания на каждый тикер.
4. **П.4.** Используются `UNTYPED_INSTRUMENT_TYPES` и `broker_figi`, без копий логики.
5. **П.5.** Добавлен тест изоляции токенов: двое владельцев, и чужой счёт в сессии третьего.
6. **П.6.** `_HANG_GUARD_SEC=60` на рандеву и на `await task` — только страховка от зависания, на ход теста не влияет.
7. **П.7.** Autouse-фикстура `TestRaceAndLock._fresh_lock_registry` делает `gc.collect()` до и после каждого теста.
8. **Флейк `test_adapter_released_when_cancelled_waiting_trade_lock`.** Причина того же класса: тест опрашивал `_trades(db_session)` той же сессией, которой пишет задача входа.
   - Воспроизвёл случайными задержками драйвера aiosqlite: 12/40 падений с `InvalidRequestError: This session is in 'prepared' state`, ещё 27/40 с `assert not task.done()`.
   - Починил так же: событие на лок сделки, `gc.collect()`, страховочный таймаут. Код продукта не менялся.
   - После правки 40/40 зелёные. Харнесс удалён, копия: `s8r-evidence/fixes/repro_FIX-068_pspb_harness.py.txt`.

### Тесты
- RED на коде раунда 0: п.1 `assert [] == [('token-2', 'BBGZBOND01')]`; п.2 `assert ['token-1'] == ['token-2']`; п.3 и тест `requests` — `AttributeError` (API ещё не было).
- GREEN: `test_fix067_bond_type_backfill.py` 13/13, файлы рандеву зелёные.
- Мутации (каждая откачена, md5 сверен):
  - бюджет по числу FIGI → `assert [] == [('token-2', …)]`;
  - без проверки остатка дедлайна в скане → `'BBGZBOND01' not in [...]` не выполнено;
  - без проверки внутри метода → `assert 2 == 1` (`requests`);
  - без `persist_until` → `assert 'bond' == 'unknown'`;
  - общий список пар для всех владельцев → `[('token-1', …)] == [('token-2', …)]`;
  - без `gc.collect` в фикстуре класса (харнесс с устаревшим локом) → `assert not True`;
  - `_AdapterHolder.release` без disconnect → `Expected disconnect to have been awaited once. Awaited 0 times`.
- Оговорка: мутация «вход без лока сделки» тест `test_adapter_released…` НЕ роняет — вход всё равно останавливается на следующем ожидании. Так было и в исходном тесте; в карточку не выношу.
- 20 повторов `TestRaceAndLock` и `test_adapter_released…` под нагрузкой полного прогона: 20/20.

### Гейты
- pytest (`faulthandler_timeout=240`): **5540 passed / 1 skipped / 0 failed** (`s8r-evidence/fixes/pytest_TB2_r1.log`).
- ruff 0; mypy Success (204); bandit 0 замечаний; typecheck 0; lint 0; build ok.
- vitest: фронт не менялся — на уровне пакета.

### Файлы
- Изменены: `app/market_data/service.py`, `app/corporate_actions/service.py`, `tests/test_trading/test_commission_settle.py`, `tests/test_trading/test_position_sizing_percent_base.py`.
- Новый: `tests/unit/test_corporate_actions/test_fix067_bond_type_backfill.py`.

### Integration points
- ✅ `corporate_actions/service.py:735` → `:812` `backfill_instrument_types`.
- ✅ `market_data/service.py:2187` — `ensure_instrument_types` ← `tax/service.py` (без изменений).
- ✅ `:939` и `:2401` → `_persist_instrument_rows`.

### Текст для ТЗ §5.12 (заменяет предложенный в р.0)
«S8R-FIX-067: перед отбором облигаций для `bondization` тикеры открытых позиций с типом `unknown`/пустым (`UNTYPED_INSTRUMENT_TYPES`) дозапрашиваются по FIGI (`broker_figi` справочника, иначе сделки) через `MarketDataService.backfill_instrument_types`. Токен — счета владельца сессии: production вперёд, среди счетов контура — счёт сессии, неактивный заменяется другим активным счётом владельца; пары разных владельцев не смешиваются (S8R-AUDIT-003). Бюджет на прогон — `BOND_SCAN_TYPE_BACKFILL_MAX_REQUESTS=20` реально начатых запросов (FIGI из кэша промахов не считаются) и общий дедлайн 15 с. При остатке меньше `BOND_SCAN_TYPE_BACKFILL_MIN_SLICE_SEC=1` новые вызовы и запросы не начинаются, FIGI не помечается. Найденные типы пишутся в `instruments` одной транзакцией и не позже дедлайна. Сеть — в `_fetch_iss`, вне единицы записи. Сбой — отбор по эвристике тикера.»
ТЗ §5.13 (налоги), дописать: «Дозапрос — через `backfill_instrument_types` (обёртка `ensure_instrument_types`); строки справочника — одной транзакцией.»

### Остаточный риск и находки
- При `persist_until` одна запись, начатая до дедлайна, может ждать write-лок SQLite до `busy_timeout` (30 с). Это одно ожидание на вызов, а не по одному на тикер.
- Событие журнала `instrument_cache_persist_failed`: поле `ticker` стало `tickers` (список).
- В налоговом пути `min_request_slice_sec` не включён, чтобы не менять поведение и тесты S8R-FIX-043. Можно включить отдельной правкой.
