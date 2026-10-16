# Ревью пакета FINDA (S8R-FIX), ось «требование»

Код: `wt-s8r-fixes-c`, ветка `s8r/fix-findings`, диапазон `2d460cf..11d5a1c`. Только чтение.

Прогон: 19 тест-файлов пакета (030/044/053/064, 037/039/041/042/048, 049, 051, 062/063/065 + раунды TR2) — **205 passed**. Отдельные пробы вызовами функций: 039 (`12.345` → `12.3450`, None → NULL), 062 (3 из 10 лотов, P&L 300, а не 1000), 063 (цена из `stages`), 030 (терминальны только AUTH-коды), 053 (`usd` → ValidationError).

| Карточка | Вердикт | На чём проверено |
|---|---|---|
| 030 | OK | Одна таблица `grpc_codes.py`. По ней работают `classify`, `is_definitive_rejection`, `is_broker_order_rejection` и мультиплексор. Своих наборов кодов в `app/broker` не осталось. |
| 032 | OK (дубль 048) | Все пути сверки (runtime, `sandbox_reopen`, engine) вызывают `get_real_positions(fetch_instrument_info=False)`. Вложенных сетевых вызовов нет. |
| 044 | OK | На подключение: одна сессия БД, два агрегата по `IN`, `raiseload("*")`. Других запросов на каждую сессию в WS нет. |
| 045 | OK (через 064) | Длинного держателя write-лока на пути «бэктест → резолв FIGI → запись кэша» больше нет. |
| 064 | OK | Если ISS-строк нет, срабатывает `SELECT EXISTS` и выход. Если есть, DELETE выполняется в своей сессии с commit. Проверены все вызывающие `get_candles` (backtest, grid, runtime, prefetch, роутеры): ни у одного на этой сессии нет незакоммиченного DML. |
| 053 | OK | У адаптера и протокола параметра валюты нет. `currency`≠rub в теле запроса — 422. Query-параметра нет. |
| 037 | OK | Прямых записей `status` у `TradingSession` и `delete(TradingSession)` вне `session_status.py` нет. CB, restore, shutdown, пометки и удаление идут через setter. |
| 039 | OK | Миграция `752ae89f7ec2`: `Numeric(18,4)`, nullable. Проверки без числа (`duplicate_instrument`, sizing) пишут NULL. |
| 041 | OK | Дашборд стратегии использует `trade_lot_multiplier` + `position_quantity`. Других расчётов P&L в `strategy/service.py` нет. |
| 042 | OK | Сброс счётчика — в `_resume_session_locked`, это единственный путь «Возобновить» (`paused→active`). |
| 048 | OK | Через `_apply_close_and_settle` идут закрытие, `finalize_exit_order`, сверка после переоткрытия. Если цена не найдена — `closed`, `pnl` NULL, пометка в UI. |
| 049 | OK | Правило «0 ≠ факт до окна» одно для входа, выхода и досверщика (`_broker_commission_fact`). Досверщик подключён в периодический цикл runtime. |
| 051 | OK | В lifespan вызывается `set_default_price_alert_monitor` + `start_background`. Монитор сам подписывается через счёт владельца. Пересчёт срабатывает на create/update/delete алерта и счёта. |
| 062 | OK | Все потребители (карточка, позиции, `unrealized_pnl_by_session` → CB и дневной отчёт, дашборд, Telegram) используют `position_lots`. |
| 063 | OK | `stages[].price` — третий источник в `order_fill_price`, и на входе, и при дозапросе на закрытии. Консервативная оценка для CB ждёт решения заказчика, дефектом не считается. |
| 065 | OK | `trade.closed`/`position.mismatch` пишутся в outbox той же транзакцией, что закрытие. Для sandbox/real запись под shield. Есть доставка при отмене (`publish_staged_after`) и повтор при старте. |

## Итог
Находок: high 0, medium 0, low 0. Все 15 карточек пакета (с 045/064 — 16 позиций) по оси «требование» закрыты.
