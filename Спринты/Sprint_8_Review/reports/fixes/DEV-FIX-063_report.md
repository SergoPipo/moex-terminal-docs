## DEV-FIX-063 отчёт — S8R fixes, FIX
Статус: ✅ готово к коммиту (worktree `wt-s8r-fixes`, база 48f6773, не закоммичено); ревью р.1 учтено.
### 1. Что реализовано
- `app/circuit_breaker/unpriced.py`: оценка сделки «P&L не определён» для CB. Цена входа: факт → цена сигнала → close бара. Цена выхода: факт → `exit_signal_price` → close бара. На оценённую цену — сдвиг 1 % против позиции, количество — `position_quantity`, в CB идёт только `min(оценка, 0)`.
- Оценка учтена в дневном лимите (класс, собственный лимит сессии, статус CB) и в просадке sandbox/real. Комиссия — по общему правилу: лимит считает валовой P&L, equity уже вычитает Σcommission.
- Если срабатывание учло оценку, в причине пометка «(включая оценку N сделки/сделок без подтверждённой цены)».
### Ревью р.1
1. Уведомление `cb.unpriced_trade` (critical) перенесено в путь закрытия: строка outbox в той же транзакции, что `_settle_close_locked` и сверка после переоткрытия; публикация — `publish_staged_after`. Из проверки CB уведомление и дедупликация удалены.
2. Цена бара берётся из последнего закрытого к моменту бара (`timestamp ≤ t − ТФ`, LIMIT 1, по запросу на момент): `market_data.closed_bar_close_before`. Ограничение «не старше 3 баров» оставил: старый бар — это дыра в кэше, а не цена.
3. Собственный лимит сессии берётся из классового состояния (`DailyLossState.pnl_by_session`) — повторных запросов нет.
4. `cb_unpriced_trade` добавлен в `EMAIL_ALLOWED_EVENTS` и в каналы мастера по умолчанию (Telegram и Email, как `cb_triggered`).
5. `_daily_pnl` удалена, вместо неё `_daily_pnl_by_session`; тесты переведены.
### 2. Файлы
Новые: `circuit_breaker/unpriced.py`, тесты `test_circuit_breaker/test_fix_063_unpriced_cb_estimate.py` и `test_trading/test_fix_063_unpriced_alert_on_close.py`. Изменённые: `circuit_breaker/engine.py`, `trading/engine.py`, `market_data/service.py`, `notification/service.py`, `notification/email.py`, `common/outbox.py`, `NotificationSettingsPage.tsx`, `FirstRunWizard.tsx` и тесты (`FirstRunWizard.test.tsx`, число событий 19→20, `test_fix_048`).
### 3. Тесты
- Первый RED: `event_type='position_size_limit'` вместо daily_loss_limit; `Decimal('0') == Decimal('-139')`.
- Ревью р.1: тесты написаны после правки, поэтому RED снят мутациями:
  - без алерта при закрытии → `assert 0 == 1`;
  - без алерта при сверке → `assert 0 == 1`;
  - бар текущий вместо закрытого → `Decimal('-2150.00') == Decimal('-130')`;
  - собственный лимит пересчитывается → `assert 10 == 7` (число SQL).
- Мутации первого прохода: учёт прибыли → `47000.00 == 0`; без сдвига → `-50000.00 == -53000`.
- GREEN 11/11.
- Гейты: pytest 5643 passed / 0 xfailed / 0 failed; ruff 0; mypy Success (206); bandit 0; typecheck 0; lint 0; build ok. vitest только затронутых файлов (6 файлов) — passed; полный не запускал.
### 4. Integration points
✅ `_settle_close_locked` и `_reconcile_locked` → `unpriced_trade_alert` / `stage_unpriced_trade_alert`; `_daily_pnl_by_session` и `broker_equity_by_session` → `unpriced_cb_losses`; новое событие внесено в EVENT_MAP, EVENT_TYPE_LABELS, OUTBOX_(REPLAY_)EVENTS и EMAIL_ALLOWED_EVENTS.
### 5. Контракты
API не менялось, миграции нет. В настройках уведомлений новая строка `cb_unpriced_trade`, у неё есть email.
### 6. Проблемы / правки документов / находки
- ТЗ §5.4 (FIX-048): «дневной убыток CB её не видит» → «CB учитывает консервативной оценкой; если оценки нет — critical `cb_unpriced_trade` при закрытии». ТЗ §5.8: описать оценку. ФТ §12.4 и §10.1: добавить уведомление в список email-событий.
- Не менял: `session_capital` (это объём ордера, а не проверка убытка); нереализованный P&L открытой позиции без цены входа — новая находка.
- `unrealized_pnl_for_sessions` больше не вызывается в production-коде (только в тестах).
- Под shield записи закрытия добавились до 2 чтений кэша свечей — это только чтения.
- Самопроверка: 1 — если commit закрытия упадёт, не будет ни строки, ни уведомления; 2 — одно уведомление на сделку по построению, critical; 3 — тесты идут через `close_position` и `process_signal`; 4 — новых таймаутов нет; 5 — все вызывающие проверены; 6 — неприменимо.
### 7. Gotchas: 05, 18, 37, 58, 78. 8. Новых нет.
### 9. Плагины: py_compile и mypy, `tsc -b`, скилл tdd; context7 не нужен.

### Ревью р.2 (финальный раунд)
1. **Каналы у существующих пользователей.** В `notification/service.py` добавлены `SETTINGS_INHERIT_FROM = {cb_unpriced_trade: cb_triggered}`, `pick_event_setting` и `load_event_setting`. Последнюю используют все чтения настроек при доставке (`stage_notification`, `_create_notification_impl`, `_replay_dispatch`). `GET /settings` использует то же правило через `pick_event_setting`. Своя строка главнее родительской. Дополнительно: первый собственный `PUT` берёт непереданные поля у родителя, иначе переключение одного тумблера молча выключало бы остальные. Миграции нет.
2. **Текст уведомления:** «Сделка #N (TICKER) закрыта без цены от брокера; оценка по рынку на момент закрытия недоступна — дневной лимит её пока не учитывает (учтёт, когда появятся котировки). Проверьте сделку у брокера.» Пересчёт оценки на каждой проверке CB не менял.
3. **Окно поиска бара:** последний закрытый бар (`timestamp ≤ t − ТФ`), не старше `unpriced_bar_max_age`: внутридневные ТФ — 24 ч, D и старше — 7 суток. Предел «3 бара» убран.
4. **Комментарии** приведены к факту: `notification/service.py` (EVENT_MAP), `trading/engine.py` (`_daily_pnl_by_session`; закрытие «P&L не определён»), docstring `risk_monitor._apply_close`, `unrealized.py`.
5. **Сверка после переоткрытия:** общий `BarCache` на весь проход.
6. **Общий построитель запроса** `_latest_bar_query` для `last_cached_bar` и `closed_bar_close_before`.
7. **`unrealized_pnl_for_sessions` удалена.** Тесты `test_b1` и `test_fix_062` переведены на `_daily_pnl_by_session` (это потребитель CB).

**Тесты р.2:**
- RED:
  - п.1 — `Expected send to have been awaited once. Awaited 0 times` и `assert False is True` (GET /settings);
  - п.3 — `Decimal('0') == Decimal('-120')` (1m после паузы 18:50–19:05);
  - п.5 — `assert 2 == 1` (число SQL-запросов).
- Мутации:
  - без наследования → `Awaited 0 times`;
  - окно 3 минуты → `Decimal('0') == Decimal('-120')`;
  - без общего кэша → `assert 2 == 1`.
- Попутно: первая версия `load_event_setting` (`IN` + `.all()`) уронила 14 мок-тестов `test_dispatch_all_events`. Переделал на «своя строка через `first()`, родитель вторым запросом только при её отсутствии» — тесты зелёные без правки.
- Гейты: pytest 5648 passed / 0 failed / 0 xfailed (лог `scratchpad/pytest_full_r2b.log`); ruff 0; mypy Success (206); bandit 0; typecheck 0; lint 0; build ok. vitest: FirstRunWizard и NotificationSettingsPage — 29/29 (фронт в р.2 не менялся).

**Итоговые тексты ФТ/ТЗ (для внесения оркестратором):**
- **ФТ §12.4**, пункт «Дневной лимит убытков», добавить: «Сделка, по которой брокер не сообщил цену входа или выхода („P&L не определён“), учитывается в дневном лимите и в просадке (песочница и реальный счёт) консервативной оценкой. Недостающая цена берётся из цены сигнала, рыночной цены момента закрытия или последней закрытой свечи. Оценка ухудшается на 1 % против позиции, засчитывается только убыток. Если цену оценить нечем, при закрытии сделки владельцу уходит критичное уведомление „Circuit Breaker: сделка без цены“; лимит учтёт сделку, когда появятся котировки. Срабатывание, учёвшее оценку, помечается в причине. Журнал сделок, дневная статистика и налоговый отчёт по-прежнему показывают „P&L не определён“.»
- **ФТ §10.1**, таблица, новая строка: «Сделка без цены и без оценки (Circuit Breaker) | ✅ | ✅ | ✅». Под таблицей: «Если строки настройки для этого события нет, каналы берутся из настройки „Circuit Breaker“».
- **ТЗ §5.4**, абзац S8R-FIX-048: «дневной убыток CB её не видит (S8R-FIX-063)» заменить на «CB учитывает её консервативной оценкой (S8R-FIX-063 ч.2, `app/circuit_breaker/unpriced.py`); без оценки — `cb.unpriced_trade` (critical, outbox транзакции закрытия, `stage_unpriced_trade_alert`; пути `_settle_close_locked` и сверка после переоткрытия)».
- **ТЗ §5.8**, «Расчёт дневных убытков»: «+ Σ min(оценка, 0) сделок „P&L не определён“ (`unpriced_cb_losses`). Цены: вход — факт → `entry_signal_price` → close последнего закрытого бара ≤ `opened_at` − ТФ; выход — факт → `exit_signal_price` → бар ≤ `closed_at` − ТФ. Давность бара: ≤ 24 ч для внутридневных ТФ, ≤ 7 суток для D и выше; ISS не берётся. Сдвиг `CB_UNPRICED_ADVERSE_PCT` = 1 % против позиции, количество — `position_quantity`. Комиссия не вычитается — как у валового `pnl`. Собственный лимит сессии — доля из классового расчёта (`pnl_by_session`). Пометка причины — „включая оценку N сделок без подтверждённой цены“». «Max drawdown» (sandbox/real): «equity включает оценённый убыток таких сделок всех дней».
- **ТЗ, раздел уведомлений:** «`SETTINGS_INHERIT_FROM`: `cb_unpriced_trade` без своей строки наследует каналы `cb_triggered` (доставка и `GET /settings`; первый `PUT` дополняет непереданные поля из родителя)».
