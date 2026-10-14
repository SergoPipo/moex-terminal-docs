# DEV-3 — backend, фиксы сплошной проверки UI-чеклиста S8 (2026-10-14)

`wt-s8r-fix`, ничего не закоммичено. Каждая карточка: RED → GREEN → мутация (все покраснели).

## 1. Что реализовано (RED — одной строкой)
- **114** `BacktestJobManager` не уведомляет за `single` (`RUNNER_NOTIFIES_JOB_TYPES`), Grid — своё уведомление с данными. RED: `['SBER · 5m · P&L -4.26% …', '— (SBER, 5m): P&L —, Sharpe —']`.
- **115** занятость `account_id` — по всем sandbox-записям; лок переоткрытия общий (`sandbox_recovery`, было по `user_id`); отказ — `SANDBOX_ALL_ACCOUNTS_TAKEN_MESSAGE`. RED: `REOPENED is UNAVAILABLE … account_id='f925da17'`.
- **116** «…остановите на странице «Торговля».»; перечень сессий сокращается под потолок 400, подсказки не режутся. RED: `…сессии остановите.'.endswith(…)` False.
- **117** `session_mode_label`: paper / песочница / реальный. RED: `— SBER (1h) — Paper` для sandbox.
- **118** `date_req` = сегодня МСК + отказ курсу «из будущего». RED: `assert None == '14/10/2026'`.
- **119** `BrokerKeyUnreadableError(InvalidTag)` с единым текстом; 409 `broker_key_unreadable`; лог close-all — `close_failure_reason`. RED: `'не расшифровывается' in ''`, лог `reason: ''`.
- **120** `money.format_price`. RED: `достиг 1.00000000 ₽`.
- **121** `_signed_rub`: копейки, без «-0». RED: `real PnL -0 ₽`.
- **122** WARNING `iss_dividends_block_missing` (тикер, блоки); прошедший дивиденд без получателей — `processed=True`, без уведомления. RED: 4 уведомления вместо 1.
- **123** autouse-фикстура `_reset_keyed_locks` + `locks._reset_for_tests`. RED: `Lock … [locked] is bound to a different event loop`.

## 2. Файлы
`app/`: `backtest/jobs.py`, `broker/{crypto_helpers,sandbox_recovery,service}.py`, `common/{exceptions,locks,money}.py`, `corporate_actions/service.py`, `market_data/{cbr_rate,price_alert_monitor}.py`, `notification/telegram_webhook.py`, `scheduler/service.py`, `strategy/router.py`, `trading/engine.py` (строки FIX-097 не тронуты). Тесты: 10 новых `test_fix_1NN_*`; под новое поведение — 11 существующих (в т.ч. `test_sandbox_recovery.py`: тест закреплял дефект 115).

## 3. Тесты
Полный backend (без `test_backup_cli`): **5905 passed, 1 skipped** (baseline 5869 + 36 новых). ruff 0, mypy 0, bandit 0.

## 4. Integration points
Новые функции вызываются в `app/` (grep); `locks._reset_for_tests` — только тесты (как `_reset_reopened_at_registry_for_tests`).

## 5. Контракты
409 `broker_key_unreadable` (`detail` — что делать) — любой endpoint, где ключ не расшифровался (DEV-2: показать `detail`). Grid: заголовок «Grid Search завершён». Тексты Telegram и дневного отчёта изменены → ФТ/чеклист.

## 6. SKIP / TODO
- **122 п.2 SKIP**: бесплатного ISS-источника нет — `dividends.json` отдаёт описание бумаги, `/iss/cci/corp-actions/dividends` — «только подписчикам» (HTTP 200 + HTML). Альтернатива — T-Invest `GetDividends` (отдельная карточка). `/health` статуса источников не имеет.
- **3.11 SKIP**: нет колонки причины в `BacktestTrade`, генератор не пишет причину — миграция + кодоген.
- **3.12 SKIP**: корень — `_fetch_candles` глотает ошибки T-Invest; проброс меняет контракт `get_candles` для графика/runtime/prefetch/backtest — нужно решение.
- **3.13 PASS** — «…или бэктестов» убрано.
- TODO 115: `create_account` по-прежнему заводит второму пользователю запись на тот же sandbox-счёт (UNIQUE по `user_id`) — нужно решение.

## 7. Gotchas: 70, 71, 48, 28, 37. 8. Новые (предложение): ISS CCI отвечает 200 HTML «только подписчикам»; `str(InvalidTag()) == ""`.
## 9. Плагины: pyright-lsp — fallback прогон/mypy; mattpocock-skills:tdd.
