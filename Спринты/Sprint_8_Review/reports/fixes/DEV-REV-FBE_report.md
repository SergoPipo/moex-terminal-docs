## DEV-REV-FBE отчёт — S8R fixes, FIX
Статус: ✅ готово к коммиту (worktree `wt-s8r-fixes-b`, база 11d5a1c, не закоммичено)
### 1. Что реализовано
- 1, 3, 7, 12: PEP 8; комментарий источников комиссии (`broker_zero`, `unknown`); `SHUTDOWN_OUTBOX_WAIT_SEC = OUTBOX_STAGE_TIMEOUT_SEC + 1.0`; комментарий про `ticker_eq`.
- 2, 10: пороги времени убраны. В 048 вместо них журнал `cancelled:<id>` и событие `entry_hang_started`, у бюджета 3600 с. В 056 — рандеву после `_has_tinvest`. Предохранитель `_HANG_GUARD_SEC`.
- 4: монитор алертов берётся только из реестра (`registered_monitor`), `app.state.price_alert_monitor` убран.
- 5: `is_pnl_undetermined(trade)` по объекту (Protocol) во всех 5 местах.
- 6: цена выхода без брокера передаётся как `None`, флаг `exit_price_unconfirmed` удалён. В логе `exit_price=None`, оценка — в поле `exit_estimate`.
- 8: в create, run-async и grid период проверяется до `_with_canonical_ticker`.
- 9, 11: резолвер переведён на `TtlCache`. Негативный кэш ключуется `(тикер, есть ли T-Invest)`, как полёт.
- 13: `same_ticker` в `market_data/service.py` (4 места).
- 14: `app/common/validation_text.py` — общий перевод; тексты обоих путей прежние.
- 15–17: рисунок сохраняется с каноническим тикером (только известным, без сети — ревью р.1); кнопка ведёт по тикеру без `.upper()`; дозапрос типа берёт и сделки без цены (вес — |комиссия|).
### 2. Файлы
Новые: `app/common/validation_text.py`, 6 тестов `test_rev_fbe_*`. Изменены 16 файлов `app/` и 4 существующих теста.
### 3. Тесты
- RED (фактические строки):
  - 6: `assert '300.00000000' is None`
  - 8: `AssertionError: резолв тикера до проверки периода`
  - 11: `assert 'SIZ6' == 'SiZ6'`
  - 15: `assert 'siz6' == 'SiZ6'`
  - 16: `assert 'http://localhost:5173/chart/SiZ6' in '…/chart/SIZ6'`
  - 17: `assert [] == ['FIGINOPRICE']`
- GREEN — все шесть. Мутации (откат через бэкап и md5) падают теми же строками. На п.6 мутация — заглушка в `finalize`, падение «0 == 1».
- П.2 и п.10: 20 итераций × 3 параллельных процесса + 4 CPU-петли — 240/240 зелёные. Мутация single-flight проверена.
- pytest: 5614 passed / 0 xfailed / 0 failed. ruff 0; mypy Success (205); bandit 0; typecheck 0; lint 0; build ok.
- vitest: фронт не менялся — на уровне пакета.
### 4. Integration points
✅ `named_value_problem` → `risk_params.py`; `labeled_field_problem` → `backtest/router.py`; `known_canonical` → `chart_drawings/router.py::create_drawing`. NOT CONNECTED нет.
### 5. Контракты
Миграции нет. Контракт API прежний (после ревью р.1 422 на формат тикера рисунка снята).
### 6. Проблемы / правки доков / находки
- ТЗ §5.6 (Канонический тикер): «негативный кэш 60 с по ключу (тикер, есть ли T-Invest), как single-flight; свежие отметки обоих видов — ответ без чтения счёта»; «создание рисунка графика — `known_canonical` (кэш/справочник, без сети; не известен — тикер пути как есть)».
- ТЗ §5.13: «Дозапрос: тикеры лотов и сделок «P&L не определён»; вес — Σ|realized_pnl|, у сделок без цены — |комиссия|».
- ТЗ §11.15, ФТ §19.13: «`open_chart:{ticker}` — канонический тикер, регистр сохраняется; формат — `TICKER_RE` (латиница, цифры, «.», «-», «_», 1–20, без «..»)» вместо `[A-Z0-9_]{1,20}`.
- ТЗ API backtest: «период проверяется до канонизации тикера».
- Развилки: тексты 422 бэктеста без детализации (п.14). У `same_ticker` пустая строка ≠ пустой.
- Новые находки: нет (находка про `isalnum()` закрыта в ревью р.1 п.7).
- Самопроверка: 1–6 пройдены. Новых записей статуса и уведомлений нет. Пути тестов через HTTP/`close_position`/`finalize`/`generate_report`. Вызывающие `_apply_close`, `is_pnl_undetermined`, `_refresh_alert_monitor` разобраны.
### 7. Gotchas
59, 71, 81, 37, 48.
### 8. Новые Gotchas
Нет.
### 9. Плагины
py_compile/mypy вместо pyright (worktree); typecheck `tsc -b`; context7 не потребовался (своё API); tdd — скилл `mattpocock-skills:tdd`.

### Ревью р.1
1. п.15 — `known_canonical` (без сети); `Ticker`/422 в POST рисунка убраны. Тест: `siz6` при известном `SiZ6` → `SiZ6`, `_from_tinvest`/`_from_iss` не вызваны; неизвестный `NewT1` — как есть, 201. Мутации: без канона → `'siz6' == 'SiZ6'`; сетевой `canonical_ticker` → `['NEWT1'] == ['NewT1']`.
2. п.6 — `position_closed` (engine) и `orphan_exit_recovered` (runtime): `exit_price` и `pnl` — null. Тесты на оба лога (recovery через `_recover_orphan_exit_orders`). Мутация `str(...)` → `'exit_price': 'None'`.
3. п.4 — `test_market_data_router`: фикстура `registered_alert_monitor` (`set_default_price_alert_monitor` + восстановление, `spec=PriceAlertMonitor`). Мутация «/candles зовёт монитор из реестра» → `Expected 'check_alerts_for_figi' to not have been called`.
4. п.11 — свежи `(key, False)` и `(key, True)` → фолбэк до `_has_tinvest`. Тест-счётчик `before_cursor_execute`: 0 SELECT. Мутация → `['SELECT brok…'] == []`.
5. п.17 — общий `_add_weight(ticker, amount)` для лотов и сделок без цены.
6. Обёртки `_cached/_remember/_negative_is_fresh/_remember_negative` убраны — методы `TtlCache` напрямую.
7. Кнопка графика — `TICKER_RE` + `is_dot_path_like`. RED: `'Некорректный' in '…/chart/Сбер'`; теперь допустимы и `BRK.B`, `SI-12.26` (формат `Ticker`). Мутация `isalnum()` → 4 failed.
Гейты после р.1: pytest 5623 passed / 0 xfailed / 0 failed; ruff 0; mypy Success (205); bandit 0; typecheck 0; lint 0; build ok; vitest — фронт не менялся.
