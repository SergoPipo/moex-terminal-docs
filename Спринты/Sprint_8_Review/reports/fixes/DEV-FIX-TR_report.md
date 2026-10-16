## DEV-FIX-TR отчёт — S8R fixes, FIX (пачка 042, 041, 037, 039, 048)
Статус: ✅ готово к коммиту после ревью р.5 (worktree `wt-s8r-fixes` от `90f0736`, ничего не закоммичено). Разделы 1–9 — первый проход; что изменилось после ревью — в разделах «Ревью р.1» и «Ревью р.2» ниже; «Ревью р.3», «Ревью р.4», «Ревью р.5»; при расхождении главнее последний раунд.

### 1. Что реализовано
- **042**: в `_resume_session_locked` `consecutive_fund_skips = 0` — та же транзакция, что `active` и сброс `consecutive_broker_rejections`.
- **041**: в дашборде стратегии P&L, стоимость и база % считаются в штуках: лоты × `StrategyService._trade_lot_size` (`derive_lot_size` → `ensure_lot_size`, как у карточки сессии). Другие пути без множителя (grep `app/`) не найдены.
- **037**: новый модуль `app/trading/session_status.py` — `set_session_status`, `set_sessions_status_if` (условный UPDATE), `delete_session_row`, `mirror_session_status` (копия статуса только в памяти). Пересчёт статуса стратегии теперь внутри setter'а, только при переходе через границу «живая/неживая». Через него переведены 22 места (CB, старт/пауза/возобновление/стоп/удаление, restore, shutdown, расхождения, вход без цены). Тест-страж на AST запрещает прямые записи статуса, `delete(TradingSession)` и ручной вызов `sync_strategy_status_with_sessions` вне setter'а.
- **039**: миграция `752ae89f7ec2`: колонки → `Numeric(18,4)`, допускают NULL. `_event_value`: HALF_UP до 4 знаков, `None` → NULL/`null`. Payload: `"2500.00"` и `"12.345"`. API-схема: `Decimal | None`.
- **048**: при закрытии sandbox/real делается один `read_order_state` ордера входа + `entry_fill_price`; запрос идёт до пометки «в полёте» и сохраняется тем же commit'ом. Если цену получить не удалось: сделка `closed`, `pnl`/`pnl_pct` = NULL (раньше было `cancelled`), `trades_closed`+1, P&L и комиссия в `DailyStat` не идут, досверщик её не трогает. Добавлен флаг `TradeResponse.pnl_undetermined`, фронт показывает «P&L не определён».

### 2. Файлы
Новые: `app/trading/session_status.py`, `alembic/versions/752ae89f7ec2_…py`, 5 тестов `test_fix_0{42,41,37,39,48}_*.py`.
Изменённые: `circuit_breaker/{engine,models,schemas}.py`, `strategy/{service,schemas}.py`, `trading/{engine,runtime,service,risk_monitor,schemas}.py`, `tests/test_trading/test_accounting_edges.py` (этот тест закреплял старое округление до 2 знаков), `frontend/src/api/types.ts`, `TradesTable.tsx` и его тест.

### 3. Тесты
RED: 042 `assert 3 == 0`; 041 `assert 20.0 == 200.0`; 037 — страж перечислил 18 нарушений (в этот момент модуль setter'а был пустым файлом-заглушкой); 039 `Decimal('12.35') == Decimal('12.345')` и `Decimal('0.00') != None`; 048 `assert 'cancelled' == 'closed'`; фронт `Expected "P&L не определён", Received "—"`.
GREEN: все. Мутации (откат через бэкап, md5 совпал):
- 042: убран сброс → `3 == 0`;
- 041: `qty = lots` → `20.0 == 200.0`;
- 037: убран вызов пересчёта → `'live' == 'tested'`;
- 039: для `None` пишется 0 → `!= (None, None)`;
- 048: `pnl_undetermined = False` → `'cancelled' == 'closed'`; без дозапроса цены → `None == Decimal('300.00')`.

Гейты: pytest 5309 passed / 1 skipped / 0 xfailed / 0 failed; ruff 0; mypy Success (201); bandit M0/H0; typecheck 0; lint 0; build ok. vitest: только `TradesTable.test.tsx` (8 passed), полный — на уровне пакета.

### 4. Integration points
✅ `set_sessions_status_if` — `circuit_breaker/engine.py::_trigger`, `engine._entry_without_price`; ✅ `set_session_status` — engine (старт/пауза/возобновление/стоп/settle), runtime (`_set_status_if`, `_flag_position_mismatch`, shutdown); ✅ `delete_session_row` — `trading/service._delete_session_locked`; ✅ `_recover_entry_price_for_close` — `_close_filled_position_locked`; ✅ `pnl_undetermined` ↔ `types.ts` ↔ `TradesTable`.

### 5. Контракты
Миграция `752ae89f7ec2` (от `c04dc4f891a2`), round-trip на чистой БД в scratch пройден, `alembic heads` = 1. ⚠️ Если второй DEV тоже делает ревизию от `c04dc4f891a2`, при сведении будет две головы. API `/circuit-breaker/events`: значения могут быть `null` и приходят с 4 знаками. У фронта нет потребителя журнала CB — пункт «null → «—»» неприменим.

### 6. Проблемы / правки документов / находки
- ТЗ §2.4 (стр. 389): «(`Numeric(18,2)`, нет числа — 0, S8R-FIX-039)» → «(`Numeric(18,4)`, у проверки без числа — NULL в БД и `null` в payload, S8R-FIX-039)».
- ТЗ рядом с S8R-AUDIT-031 (стр. 1732) — текст обновлён, см. «Ревью р.1».
- ФТ (журнал сделок): «P&L не определён» вместо числа.
- Гайд §7: ожидаемый head — `752ae89f7ec2`; «колонки журнала CB становятся `Numeric(18,4)` и допускают NULL; старые строки не меняются».
- Находки: (1) у такой сделки Telegram покажет «P&L −комиссия ₽», нужен шаблон `trade.closed` с «не определён»; (2) статистика сессии (`service.py:1090`) считает `pnl None` за 0 в win rate; (3) сверка после переоткрытия sandbox переводит `filled` без цены в `cancelled`; (4) `TradingService` считает нереализованный P&L по `volume_lots`, а не `filled_lots`; (5) в коде нет отдельного kill switch, CB только ставит на паузу.
- Самопроверка: 1 — сбой commit откатывает статус и пересчёт вместе; 2 — новых уведомлений нет; 3 — тесты 037/048 проходят через `_trigger` и `close_position`; 4 — сеть 048 вне записи в БД, бюджет адаптера; 5 — все вызывающие изменённых функций проверены; 6 — н/п.

### 7. Применённые Stack Gotchas
12, 27, 33, 37, 53, 60, 79.

### 8. Новые Stack Gotchas
Нет.

### 9. Плагины
py_compile после каждой правки, `pnpm typecheck` (tsc -b), tdd-скилл; context7 не понадобился (API SQLAlchemy/Pydantic уже применялись в этом коде).

### Ревью р.1
Правки:
1. Payload `trade.closed` у сделки с «P&L не определён»: `pnl` и `net_pnl` = null, `pnl_undetermined: true`, новое поле `pnl_text`. Шаблон `EVENT_MAP['trade.closed']` теперь «{ticker}: P&L {pnl_text} · комиссия {commission} ₽…». Для обычной сделки текст не изменился. Все пять мест публикации собирают payload через `trade_closed_payload`.
2. Комиссия такой сделки идёт в `DailyStat`: `realized_pnl = −commission`, `trades_closed +1`. Исключения в досверщике комиссии и в `RiskMonitor._update_daily_stat` откачены. Дневной лимит CB по-прежнему NULL не суммирует.
3. `get_stats`: такие сделки не входят в знаменатель win rate и в среднюю сделку, их комиссия входит в `net_pnl`.
4. Дозапрос цены входа перенесён: теперь он идёт после отправки и исполнения выхода, перед `_apply_close`. Бюджет — `ENTRY_PRICE_LOOKUP_TIMEOUT_SEC = 5`, через `asyncio.timeout` + `scope.expired()`. Под таймаутом только сеть.
5. Из того же `OrderState` входа пишется факт комиссии входа: `_record_side_commission(…, 'entry', …, fill=False)`.
6. Признак вынесен в `is_pnl_undetermined(status, pnl, entry_price)` (`trading/models.py`): `closed`, `pnl` NULL и `entry_price` NULL. Используется в схеме, payload и статистике.
7. Общий хелпер `trading/unrealized.trade_lot_multiplier`, при сбое источника — 1. Копии `_trade_lot_size` удалены из обоих сервисов.
8. `mirror_session_status` пишет всегда через `set_committed_value`, параметр `committed` убран. Тесты restore/runtime (AUDIT-008) — 83 passed. Страж теперь ловит и форму `.values({'status': …})`. Почему каждое имя из `NON_SESSION_RECEIVERS` — не сессия: `trade`/`db_trade` — `LiveTrade` (engine/runtime/risk_monitor), `report` — `TaxReport` (tax/service), `backtest`/`bt` — `Backtest`, `job` — `BacktestJob` (backtest/*).
9. `CLAUDE.md` worktree, раздел инвариантов: добавлена строка-исключение для журнала CB (`Numeric(18,4)`).
10. Сверка после переоткрытия sandbox обходится с двумя случаями ПО-РАЗНОМУ. `filled` с ценой → `closed`, exit = entry, `pnl` 0, `trades_closed +1`, `realized −commission`. `filled` без цены → `cancelled`, без `trades_closed`, `realized −commission`. Это расхождение в статусе и счётчике сделок; код не правил.

RED (до правок): `None == Decimal('0.15')` (п.5); `Decimal('0.00') == -Decimal('0.15')` (п.2); `True is False` (п.6 legacy); `KeyError: 'pnl_undetermined'` (п.1); `no attribute 'ENTRY_PRICE_LOOKUP_TIMEOUT_SEC'` (п.4); `Decimal('50.00') == Decimal('100')` (п.3); `RuntimeError: lot source down` и `hasattr(StrategyService,'_trade_lot_size')` (п.7); `History(added=['suspended']…).has_changes` и `7 == 8` (п.8).

Мутации (каждая через бэкап, md5 сверен): p1 `net_pnl` строкой → `'-0.30' is None`; p2 без `realized_pnl` → `0.00 == -0.30`; p3 знаменатель `total_trades` → `50.00 == 100`; p4 `timeout(None)` → `30.01 < 5`; p5 без записи комиссии → `None == 0.15`; p6 без `entry_price is None` → `True is False`; p8 обычное присваивание → `has_changes`.

Гейты: pytest 5317 passed / 1 skipped / 0 xfailed / 0 failed; ruff 0; mypy Success (201); bandit M0/H0; typecheck 0; lint 0; build ok; vitest `TradesTable.test.tsx` 8 passed (фронт в р.1 не менялся).

Новые/изменённые файлы р.1: `notification/service.py`, `trading/{models,unrealized,service}.py` (+ уже бывшие `engine`, `risk_monitor`, `schemas`, `session_status`, `strategy/service`), `CLAUDE.md`, тесты 037/041/048.

Предлагаемый текст ТЗ (вместо строки про 048): «Закрытие `filled` без цены (S8R-FIX-048): после исполнения выхода — один `GetOrderState` ордера входа (бюджет 5 с) по тому же правилу, плюс факт комиссии входа. Цены нет — `closed`, `pnl` NULL, признак `pnl_undetermined` (closed + pnl NULL + entry NULL). `DailyStat`: `trades_closed +1`, `pnl` 0, `realized_pnl −commission`. Дневной убыток CB её не видит. Статистика сессии: вне win rate и средней сделки, комиссия — в net. Уведомление: «P&L не определён (брокер не сообщил цену входа) · комиссия X ₽»».

### Ревью р.2
1. Кривая капитала (`get_stats`): сделка с «P&L не определён» даёт точку «−комиссия». Последняя точка равна `net_pnl`, просадка видит комиссию. Пример +100/ком.1 и undetermined/ком.0.30 → 99.00, 98.70.
2. Признак `is_priceless_broker_entry(status, entry_price, mode)` в `trading/models.py`: `filled` + `entry_price IS NULL` + sandbox/real. Используется в `_apply_close` (новый параметр `mode` передаёт `_settle_close_locked`; SL/TP paper — без него) и в сверке после переоткрытия. Paper, `entry = 0` и неизвестный режим идут прежним путём FIX-5 (`cancelled`).
3. Дозапрос цены входа — в единой точке `_apply_close_and_settle`, через которую идут и закрытие, и `finalize_exit_order`. Адаптер свой, бюджет 5 с. **Что выбрал при отмене:** дозапрос вместе с записью закрытия выполняется под `run_shielded_from_cancel` (grace = 5 + `ORDER_CANCEL_GRACE_SEC`) — тот же механизм, что у отправки выхода в 074. Отмена ждёт конца дозапроса и записи закрытия (с ценой или с «P&L не определён») и только потом доходит до вызывающего. Почему так: выход у брокера уже исполнен, и сделка `filled` до recovery показывала бы позицию, которой нет; при этом запись идёт в `db` и под локом вызывающего, оба живут до его выхода (gotcha-38). Сеть ограничена бюджетом, поэтому ожидание конечно. Защищённый участок — только для позиций брокера без цены, остальные закрытия не изменились.
4. Цена входа найдена, а факта комиссии нет → сторона входа пишется как при исполнении (`fill=True`): оценка `estimated` от оборота «цена × лоты × лот». Факт брокера её заменит. Цены нет → только факт (`fill=False`).
5. Сверка после переоткрытия: `filled` без цены → `closed`, `pnl`/`pnl_pct` NULL, `trades_closed +1`, `realized −commission`, уведомление без числа. Ветка «с ценой» (REV-LQ) не тронута. Тест `test_accounting_edges` для старой строки (раньше `cancelled`) переписан под новое решение.
6. Страж усилен. В функциях с `TradingSession` (включая `__table__`) теперь запрещены: `.values` с ключом `status` в любой форме (kw, dict-литерал, `**{}`), если цепочка явно не нацелена на другую таблицу; параметры `execute` (dict или список dict) с `'status'` рядом с `update/insert(TradingSession)`. **Чего страж НЕ ловит:** имя таблицы, переданное через переменную или алиас (`T = TradingSession`); ключ `status`, пришедший переменной или собранный динамически (`{k: v}`, `dict(status=…)`, распаковка неконстантного словаря); `setattr` с неконстантным именем; запись другим модулем через `text()` без `UPDATE trading_sessions SET … status` в одном литерале; присваивание `.status` объекту сессии под именем из белого списка (`trade`, `job` …).
7. `set_session_status`: у сохранённого объекта старый статус для решения о пересчёте берётся из БД; после присваивания — `flag_modified`, UPDATE выпускается гарантированно. У нового объекта — по самому объекту.
8. `trade_lot_multiplier`: при подмене на 1 — WARNING `trade_lot_multiplier_fallback` (trade_id, ticker, error). Параметр `cache` — дашборд стратегии делает один `ensure_lot_size` на тикер за вызов. Кэш по тикеру, без учёта режима сессии — как было решено.
9. `_entry_without_price`: лишний SELECT убран. Пауза не поставлена — статус не зеркалится; пометка и пауза по-прежнему в одной транзакции.

RED (до правок): `[99.00] == [99.00, 98.70]` (п.1); `ImportError is_priceless_broker_entry` (п.2); `None == 300.00` для finalize и `'filled' == 'closed'` при отмене (п.3); `None == 3.00` (п.4); `'cancelled' == 'closed'` (п.5); `[] == [3, 6, 9, 11]` (п.6); `'active' == 'suspended'` и `'live' == 'tested'` (п.7); `['SBER','SBER'] == ['SBER']` и `0 == 1` (WARNING) (п.8); SELECT статуса присутствует (п.9).

Мутации (через бэкап, md5 сверен), все красные: r1 фильтр кривой; r2 признак без режима; r3a без защиты от отмены; r3b без дозапроса в общей точке; r4 `fill=False`; r5 без ветки сверки; r7a без `flag_modified`; r7b старый статус из объекта; r8 без кэша; r9 вернуть SELECT.

Гейты: pytest 5332 passed / 1 skipped / 0 xfailed / 0 failed; ruff 0; mypy Success (201); bandit M0/H0; typecheck 0; lint 0; build ok; vitest `TradesTable.test.tsx` 8 passed (фронт в р.2 не менялся).

Предлагаемый текст ТЗ (вместо строки р.1 про 048): «Закрытие и сверка `filled` без цены в sandbox/real (S8R-FIX-048, признак `is_priceless_broker_entry`): в единой точке учёта закрытия — один `GetOrderState` ордера входа (бюджет 5 с; вместе с записью закрытия под защитой от отмены), правило S8R-AUDIT-031. Цена найдена — обычный P&L, комиссия входа — факт или оценка от оборота. Не найдена — `closed`, `pnl` NULL («P&L не определён»: closed + pnl NULL + entry NULL). `DailyStat`: `trades_closed +1`, `realized −commission`. Дневной убыток CB её не видит. Статистика сессии: вне win rate и средней сделки, комиссия — в net и в кривой капитала. Уведомление: «P&L не определён (брокер не сообщил цену входа)». Paper и `entry = 0` — `cancelled` (FIX-5)».

### Ревью р.3
1. **Дозапрос цены входа отменяемый** (gotcha-77 п.2), вне shield, бюджет 5 с. Если отмена пришла во время дозапроса, закрытие записывается с «P&L не определён» под `run_shielded_from_cancel` с прежним `ORDER_CANCEL_GRACE_SEC` (45 с), затем отмена пробрасывается. Защищённый участок не вырос, grace «Стопа» не трогал: запись закрытия — это commit ≤ busy_timeout 30 с плюс чтения, укладывается в 45 с и в `_cancelled_close_wait_sec` (50 с). **Уточнение:** до пачки запись закрытия под shield не шла вообще (защищались только отправка выхода и commit её id). Shield на пути отмены — новый, но с существующей константой и в существующем бюджете. Тест: отмена при зависшем дозапросе → `closed`, P&L NULL, выход за < 2.5 с, что меньше бюджета дозапроса 5 с и grace 45 с.
2. **Время стороны входа** при восстановленной цене — исполнение входа (`executed_at` из stages, иначе `opened_at`), через новый параметр `recorded_at` у `_record_side_commission`. Тест: real, 0 у терминального ордера без stages, вход старше окна → `broker_zero`.
3. **Адаптер вызывающего.** `_apply_close_and_settle(…, adapter_pair)`: в закрытии свой адаптер живёт до конца учёта (при ошибке отпускается сразу); recovery передаёт свой адаптер в `finalize_exit_order(adapter_pair=…)`. Свой адаптер подключается только при `adapter_pair is None` — в коде это отдельная явная ветка. Тест: одно подключение на закрытие, ноль подключений при доучёте.
4. **Синхронизация возвращена** (п.9 р.2 отменён): новый `pause_session_if_active(db, id) -> (поставлена, статус в БД)`. Если пауза не поставлена — один SELECT; зеркало в объект — после commit. Тест: объект `active`, в БД `paused`/`stopped` → после пометки объект отражает БД.
5. **Пауза «по ходу» — условием в БД.** `_settle_close_locked` (частичный выход), `_settle_entry_remainder_locked`, `_entry_without_price` → `pause_session_if_active`. Runtime `_set_status_if`, `_flag_position_mismatch` и пометка `suspended` при shutdown → `set_sessions_status_if`. Инвариант: `set_session_status` возвращает bool; если в БД `stopped`, переход в другой статус не выполняется (WARNING, False, объект зеркалит `stopped`). `set_sessions_status_if` с `stopped` в `from_statuses` и другим целевым статусом — ValueError. Легальных путей из `stopped` нет: возобновление — только из `paused`, старт создаёт новую сессию, restore берёт `active`/`suspended`, сверки ставят паузу только из `active`. Тест на старте сессии переписан на новую сессию — переход `stopped → active` теперь запрещён. Тесты: устаревший `active` при `stopped` в БД → частичный выход не возвращает `paused`, статус стратегии не меняется; setter не выводит из `stopped`.
6. **Страж:** ключ-колонка `{TradingSession.status: …}` и `TradingSession.__table__.c.status` в `.values` — нарушение; добавлено в перечень и в тест-пробу.

RED (до правок): ожидание дозапроса 5.00 с (п.1); `'estimated' == 'broker_zero'` (п.2); `2 == 1` подключения (п.3); `'active' == 'paused'/'stopped'` (п.4); `'paused' == 'stopped'` и `None is False` (п.5); `[3, 6, 9, 11] == [3, 6, 9, 11, 13, 16]` (п.6).

Мутации (бэкап, md5 сверен): q1 дозапрос под shield → «отмена ждала дозапрос»; q1b без записи закрытия при отмене → `'filled' == 'closed'`; q2 `recorded_at=None` → `estimated`; q3 `adapter_pair=None` → `2 == 1`; q4 без SELECT факта → `'active' == 'paused'`; q5a без инварианта `stopped` → `True is False`. q5b (вернуть `set_session_status` в частичный выход) в одиночку тест не роняет — его подстраховывает инвариант setter'а (две линии защиты). Вместе с q5a → `'paused' == 'stopped'`.

Гейты: pytest 5337 passed / 1 skipped / 0 xfailed / 0 failed; ruff 0; mypy Success (201); bandit M0/H0; typecheck 0; lint 0; build ok; фронт в р.3 не менялся.

Ограничения (честно): если отмена прилетит внутрь DB-чтения (`_estimate_commission` при восстановленной цене или resolve собственного адаптера), это отравит сессию (gotcha-38). Запись закрытия под shield тогда упадёт с WARNING, сделка останется `filled` с id выхода — её доучтёт recovery `finalize_exit_order`. Окно узкое, а в основных путях адаптер теперь передаётся, так что resolve не нужен.

### Ревью р.4
1. **(блокирующее)** На пути «отмена во время дозапроса» сбой защищённой записи больше не подменяет отмену: `run_shielded_from_cancel(...)` обёрнут в `except Exception` → WARNING `close_record_failed_after_cancel`, затем всегда `raise` исходной `CancelledError`. Тест: запись падает `OperationalError(database is locked)` → наружу `CancelledError`, WARNING записан.
2. **Уведомление о частичном выходе на пути отмены.** Новый `_settle_and_notify` = запись закрытия + `_notify_partial_exit` (при `notify_partial`). На пути отмены этот участок целиком под shield. Outbox не использовал: `position.mismatch` не входит в `OUTBOX_EVENTS`/`OUTBOX_REPLAY_EVENTS`, поэтому публикация идёт после commit внутри того же участка. Закрытие передаёт `notify_partial=True` (свой отдельный вызов уведомления убран); recovery (`finalize_exit_order`) уведомляет сам, как раньше. Тест: частичный выход + отмена во время дозапроса → ровно одно `position.mismatch`.
3. **Адаптер до конца учёта — только у позиции без цены входа** (`is_priceless_broker_entry`). У обычной позиции свой адаптер отключается сразу, до учёта, как до пачки; при исключении — тоже сразу. Тест: порядок `disconnect` → учёт.
4. **Ветка «подключиться здесь же» убрана.** Без `adapter_pair` — WARNING `close_entry_price_no_adapter` и «P&L не определён» без сети. В `finalize_exit_order` `adapter_pair` стал обязательным keyword-параметром (None допустим явно). Тест r2 на `finalize_exit_order` дополнен передачей адаптера. Тест: без адаптера — ноль подключений и чтений, `pnl` NULL.
5. **Runtime: сначала чтение статуса.** `_set_status_if`: если `row.status` (только что прочитан `db.get`) не в `from_statuses` — без UPDATE. `_flag_position_mismatch`: условие `db_session.status in pause_from` возвращено перед UPDATE. Пометка `suspended` при shutdown: сначала SELECT статуса. Если статус подходит — условный UPDATE через `set_sessions_status_if`. Тест (счётчик через событие SQLAlchemy `before_cursor_execute`): два холостых перехода и один настоящий → ровно один `UPDATE trading_sessions SET status`.

RED (до правок): `OperationalError … database is locked` вместо `CancelledError` (п.1); `0 == 1` уведомлений (п.2); `['settle', 'disconnect'] == ['disconnect', 'settle']` (п.3); `pnl 200.00 is None` — открылось своё подключение (п.4); `3 == 1` UPDATE статуса (п.5).

Мутации (бэкап, md5 сверен), все красные: t1 `except ZeroDivisionError` → `OperationalError` наружу; t2 `notify_partial=False` → `0 == 1`; t3 всегда держать адаптер → порядок; t4 вернуть своё подключение → `pnl 200.00`; t5 без чтения перед UPDATE → `3 == 1`.

Гейты: pytest 5342 passed / 1 skipped / 0 xfailed / 0 failed; ruff 0; mypy Success (201); bandit M0/H0; typecheck 0; lint 0; build ok; фронт в р.4 не менялся.

### Ревью р.5
1. **Recovery: частичный выход не теряется при отмене.** Флаг `notify_partial` заменён колбэком `on_partial_exit: PartialExitHook` (`(закрытая сделка, (исполнено, заявлено))`). Он вызывается в `_settle_and_notify` после записи, на пути отмены — внутри защищённого участка. Колбэки: закрытие → `_partial_exit_notifier` (шина, `_notify_partial_exit`); recovery → `_flag_partial_exit` runtime. Прежний вызов `_flag_partial_exit` после `finalize_exit_order` убран, поэтому уведомление одно в обоих исходах. Тест (runtime `_recover_orphan_exit_orders`, позиция без цены, выход 1 из 2 лотов): и при истечении бюджета дозапроса, и при отмене посреди него — ровно один `_flag_partial_exit`.
2. **Отключение своего адаптера** (только у позиции без цены входа) — под `run_shielded_from_cancel` с новой константой `ADAPTER_RELEASE_GRACE_SEC = 5` (своей константы отключения в коде не было). Тест: повторная отмена посреди `disconnect` — отключение завершено.
3. **Runtime: перечитать статус при `changed=False`.** `_set_status_if` и `_flag_position_mismatch`: если статус подходил, но условный UPDATE ничего не изменил (гонка), — `db.refresh(..., ["status"])`, объект зеркалит факт. Тест: restore прочитал `suspended`, параллельно закоммичен `stopped` → объект `stopped`, `_notify_not_restored` не вызван.
4. **`adapter_pair` — keyword-only без значения по умолчанию** в `finalize_exit_order`, `_finalize_exit_order_locked`, `_apply_close_and_settle`. Вызывающие передают явно (тест р.3 — `adapter_pair=None` осознанно). Тест на сигнатуры.

Находку «обычная позиция: запись закрытия и `_notify_partial_exit` не под shield» (была до пачки) не трогал — её заведёт оркестратор.

RED (до правок): `[] == [(1, 1, 2)]` (п.1, путь отмены); «отключение адаптера оборвано повторной отменой» (п.2); `'suspended' == 'stopped'` (п.3); `POSITIONAL_OR_KEYWORD is KEYWORD_ONLY` (п.4).

Мутации (бэкап, md5 сверен), все красные: u1 runtime без колбэка → `[] == [(1, 1, 2)]`; u2 отключение без shield → «оборвано»; u3 без `refresh` → `'suspended' == 'stopped'`; u4 значение по умолчанию → сигнатура.

Гейты: pytest 5347 passed / 1 skipped / 0 xfailed / 0 failed; ruff 0; mypy Success (201); bandit M0/H0; typecheck 0; lint 0; build ok; фронт в р.5 не менялся.
