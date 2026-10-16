# Сплошная проверка UI-чеклиста S8 — пакет qa4 (брокерский контур)

Дата: 2026-10-13. Стенд 4 (backend :8104, фронт :5277), код `wt-s8r-check` = `develop` `e527a05`,
БД — копия рабочей. Разделы: S8.22, S8.24, S8.25, S8.27, S8.28, S8.29, S8.30, S8.31, S8.32, S8.35,
S8.37, S8.43 — 83 пункта.

Ордера не выставлялись, sandbox/real-сессии не запускались и не возобновлялись, «Закрыть позиции»
у брокерских сессий не нажималось. В рантайме стенда поднимались только paper-сессии
(в логе `session_runtime_started` только для #2 и #7–#13, все paper). У брокера — только чтение.

## Итог

| Статус | Кол-во |
|---|---|
| ✅ PASS | 40 |
| ❌ FAIL | 2 |
| ⚠️ BLOCKED | 1 |
| ⏸ ORDER | 34 |
| ⏸ EVENT | 5 |
| ⏸ TELEGRAM | 1 |

Доказательства: спеки `wt-s8r-check/frontend/e2e-check/qa4/*.spec.ts` (s8_22, s8_24, s8_27, s8_30_paper,
s8_30_sandbox, s8_31, s8_32, s8_37, s8_43), скриншоты и выгрузки — `s8r-evidence/checkrun/results/qa4/`
(`<раздел>-<пункт>-*.png`, `S8.24-report-2026.{xlsx,csv}`, `S8.43-api.json`), лог backend прогона —
`results/qa4/backend_stand4_run.log`, правки копии БД — `results/qa4/db_edits.log`.
Backend-тесты: `results/qa4/pytest_qa4.log` (456 passed, 1 failed — см. «Попутные находки» п.3),
`pytest_qa4_b2.log` (164 passed), отдельно `test_session_uniqueness` + `test_sltp_broker_close` +
`test_broker_position_reconcile` — 46 passed. Frontend unit: `vitest_qa4.log` — 7 файлов, 87 passed.

### Решения, принятые по ходу (развилки)

1. **Правки копии БД.** Чтобы проверить конфликты и отказы без запуска брокерских сессий, статус
   остановленных sandbox-сессий менялся прямо в копии `stand4/terminal.db` (`paused` / `suspended` /
   `active`, пометка расхождения у #3, `timeframe=NULL` у paper #1) и сразу возвращался обратно.
   Безопасно: рантайм поднимает сессии только на старте, фоновая сверка обходит только сессии со своим
   listener'ом (`runtime.py:1684`), backend в это время не перезапускался. Все правки — в `db_edits.log`.
2. **«Возобновить»** у sandbox-сессии не нажималось даже в сценарии с конфликтом: если защита
   сломана, сессия реально поднялась бы. S8.30 п.8 переведён в ⏸ ORDER.
3. У testuser1 в копии не был принят дисклеймер: модалка перехватывала клики (gotcha-47). Он принят
   через `POST /users/me/disclaimer/accept` (только копия).
4. Удалены счета копии #4 (testuser1) и #2 (S8.32 п.5, п.7). **В конце стенд 4 сброшен**
   (`down → reset → up`), копия снова исходная. Удалены и временные файлы отчётов
   `wt-s8r-check/backend/data/tax_reports/*`; их копии лежат в evidence.
5. Пункты, где фронт ничего не показывает без живой брокерской сессии, а логика backend покрыта
   тестами, получили ⏸ со ссылкой на тест. ✅ PASS по одним тестам не ставился, кроме S8.24 п.3 и
   S8.43 п.2: там проверена реальная выгрузка или API, а тест закрывает только недостающий вариант данных.

## Таблица

| Раздел | п. | Статус | Доказательство / диагноз |
|---|---|---|---|
| S8.22 | 1 | ✅ PASS | `s8_22` п.1: в UI «-5» клампится в 0, «150» — в 100. Сохраняются 0 и 0.0375 (API отдаёт то же). `PATCH /broker-accounts/3` с -1 и 101 → 422. `S8.22-1-clamp.png` |
| S8.22 | 2 | ✅ PASS | `s8_22` п.2: 0.123 → «Сохранить» → reload → в поле снова 0.123. `S8.22-2-reload.png` |
| S8.22 | 3 | ✅ PASS | UI, источник «Тариф брокерского счёта» (#3 = 0,3 %): у новой paper #7 `commission_pct=0.3000`. API: ставка 0 + счёт → 0.3000 (тариф), явные 0.2 + счёт → 0.2000. Лог `s8_37` |
| S8.22 | 4 | ⏸ ORDER | Sandbox-сделки в копии — legacy (`commission` NULL). Колонка «Комиссия» живьём заполнена на paper-сделке #54. Логика: `test_sandbox_commission::test_actual_commission_from_order_state`, `test_placed_then_polled_records_commission` (passed) |
| S8.22 | 5 | ⏸ ORDER | Рендер проверен на paper-сделке с `commission_source=estimated`: ячейка «≈ 29,22 ₽», `title` дословно «Оценка по тарифу счёта, а не удержание брокера» (`S8.37-5-commission-column.png`). Sandbox-ветка: `test_zero_from_broker_falls_back_to_account_rate`, `test_sandbox_zero_after_window_stays_estimate`. Нужна sandbox-сделка |
| S8.22 | 6 | ⏸ ORDER | Брокер удерживает комиссию только на **реальном** счёте, песочница не удерживает. Значит, пункт проверяется только на real — ордер даёт заказчик. Ветка без «≈»: vitest `TradesTable.test.tsx`, backend `test_actual_commission_from_order_state` |
| S8.22 | 7 | ⏸ ORDER | `test_sandbox_commission::test_same_commission_and_net_pnl` (passed). Нужна пара sandbox + paper на одних входных |
| S8.24 | 1 | ✅ PASS | Отчёт за 2026 (xlsx и csv): колонка «Режим», в трёх sandbox-строках «песочница». Метка «реальный счёт» — `tax/service.py:43–44`; реальных сделок в копии нет. `S8.24-report-2026.*` |
| S8.24 | 2 | ✅ PASS | Под таблицей обоих форматов: «Внимание: в отчёт включены сделки в режиме «песочница»… не подлежат декларированию» |
| S8.24 | 3 | ✅ PASS | Заголовок «Кол-во, шт», значения 30/32/33. Лот SBER у брокера = 1 (`/market-data/instruments/SBER` → `lot_size:1`), поэтому лоты = штуки. Вариант «лот ≠ 1»: `tests/unit/test_tax/test_audit_s8r_lots.py`, `test_lot_size_convergence.py` (passed); живьём — в п.4 |
| S8.24 | 4 | ⏸ ORDER | В копии нет закрытых сделок по инструменту с лотом 10. GAZP — лот 10 (в логе `lot_size_synced lot_size=10`) — сценарий A1 |
| S8.24 | 5 | ⏸ ORDER | У имеющихся строк комиссия 0+0=0 при NULL в журнале — проверка вырожденная. `test_commission_settle::test_tax_lot_uses_commission_sides` (passed) |
| S8.25 | 1 | ⏸ ORDER | `test_sltp_broker_close::test_stop_loss_places_counter_order`, `test_take_profit_places_counter_order` |
| S8.25 | 2 | ⏸ ORDER | `test_sltp_broker_close` / `test_exit_order_tracking::test_filled_exit_order_finalizes_trade` |
| S8.25 | 3 | ⏸ ORDER | `test_sandbox_commission::test_sandbox_sltp_close_charges_exit_commission` |
| S8.25 | 4 | ⏸ ORDER | `test_sltp_broker_close::test_commission_is_broker_fact_not_doubled` |
| S8.25 | 5 | ⏸ ORDER | Нужна открытая sandbox-позиция вне торговых часов: `test_outside_trading_hours_keeps_trade_filled`, `test_weekday_holiday_skips_without_broker` |
| S8.25 | 6 | ⏸ EVENT | Отказ брокера: `test_failed_close_notifies_user`, `test_rejected_order_keeps_trade_filled`, `test_broker_exception_keeps_trade_filled` |
| S8.25 | 7 | ✅ PASS | В копии paper-закрытия по уровням 2026-10-10: #50 по SL 290.5602, #52 по TP 4702.88. На свежей paper #7 портфель сходится: баланс + блок = 100 000 + P&L − комиссии. `test_paper_close_without_broker`, `test_paper_sltp_close_marks_estimated` |
| S8.27 | 1 | ✅ PASS | Ручное закрытие paper #54 (pnl 1.70, комиссия 58.45) → уведомление «SBER: P&L -56.75 ₽ · комиссия 58.45 ₽ (оценка)», то есть чистый результат. В центре уведомлений — `S8.27-notification-center.png`. В Telegram уходит тот же `notification.body` (`notification/service.py:868`), бот на стенде выключен |
| S8.27 | 2 | ✅ PASS | Там же: «· комиссия 58.45 ₽» |
| S8.27 | 3 | ✅ PASS | Там же: «(оценка)» (`commission_source=estimated`). Без пометки для broker — `test_trade_closed_net_pnl::test_broker_commission_has_no_mark` |
| S8.27 | 4 | ✅ PASS | В тексте нет `{…}` (проверено регэкспом). Все publisher'ы — `test_trade_closed_net_pnl` (8 passed) |
| S8.28 | 1 | ✅ PASS | Пометка расхождения у #3 (paused) выставлена в копии БД. На карточке рядом с «ПАУЗА» — красный бейдж «Расхождение с брокером» (`rgb(224,49,49)`). `S8.28-1-2-badge.png`. Выставление пометки сверкой — `test_mismatch_shown_and_trade_not_closed` |
| S8.28 | 2 | ✅ PASS | `title` бейджа = пометка + «Позиции не изменены — сверьте с приложением брокера». Оба числа и номер сделки собирает `runtime.py:3506–3520` («у брокера N шт, у нас M шт (сделки №…)»). Текст в копию записан в этом формате — сама проверка частично круговая, композиция текста проверена по коду и тестам |
| S8.28 | 3 | ⏸ ORDER | `test_broker_position_reconcile::test_mismatch_shown_and_trade_not_closed` |
| S8.28 | 4 | ⏸ ORDER | `test_reconcile_review_fixes::test_repeated_identical_mismatch_notifies_once` |
| S8.28 | 5 | ⏸ ORDER | `test_broker_position_reconcile::test_note_cleared_on_next_clean_check` |
| S8.28 | 6 | ⏸ ORDER | Текст «(возможно, куплено вручную)», без паузы — `runtime.py:3578–3582`. `test_mismatch_shown_without_pause` |
| S8.28 | 7 | ⏸ ORDER | `test_exit_order_tracking::test_filled_exit_order_finalizes_trade`, `test_close_does_not_place_second_order_after_restart` |
| S8.28 | 8 | ⏸ ORDER | `test_sltp_broker_close::test_daily_stat_counts_trade_once` |
| S8.28 | 9 | ⏸ ORDER | `OrderInFlightError` «Ордер на закрытие уже отправлен брокеру…» — `engine.py:6702`. `test_close_position_concurrency::test_two_closes_send_one_broker_order` |
| S8.28 | 10 | ⏸ EVENT | Сбой песочницы 70001 → `SANDBOX_UNAVAILABLE_MESSAGE` (`sandbox_recovery.py:58`, `tinvest/errors.py:270`). `test_broker/test_sandbox_flaky_70001.py` (passed) |
| S8.28 | 11 | ⏸ EVENT | `tinvest/adapter.py:1521` — «…Проверьте портфель у брокера перед повторной отправкой: ордер мог уйти» |
| S8.29 | 1 | ⏸ ORDER | `test_figi_resolution::test_sandbox_trade_gets_real_figi_from_broker` |
| S8.29 | 2 | ✅ PASS | Paper-сессии #7, #10–#13 живьём открыли и закрыли сделки, сверка их не трогает: `test_periodic_reconcile::test_paper_session_is_not_checked` |
| S8.29 | 3 | ⏸ ORDER | Интервал `POSITION_RECONCILE_INTERVAL_SEC = 900` (`runtime.py:1492`). `test_loop_reconciles_once_per_interval` |
| S8.29 | 4 | ⏸ ORDER | `test_note_cleared_on_next_clean_check`, `test_flagged_session_is_checked_even_without_open_trades` |
| S8.29 | 5 | ⏸ ORDER | `test_reconcile_account_scope::test_sibling_session_units_are_counted` |
| S8.29 | 6 | ⏸ ORDER | `test_mismatch_shown_without_pause` |
| S8.29 | 7 | ⏸ ORDER | `test_reconcile_account_scope::test_broker_short_of_account_total_still_pauses`, `test_own_position_absent_still_pauses` |
| S8.29 | 8 | ⏸ ORDER | `runtime.py:3609–3617`: уведомление при каждой новой остановке (`will_pause`) |
| S8.29 | 9 | ⏸ ORDER | `test_reconcile_lock_divergence::test_in_flight_trade_is_not_touched`, `test_exit_in_flight_log_names_owning_session` |
| S8.29 | 10 | ⏸ EVENT | `test_periodic_reconcile::test_broker_failure_does_not_stop_the_sweep`, бюджет `POSITION_RECONCILE_TIMEOUT_SEC` |
| S8.30 | 1 | ✅ PASS | #3 (sandbox, «Сэндбокс», SBER) переведена в paused в копии. В форме sandbox / «Сэндбокс» / SBER — `duplicate-session-alert`, «Запустить» disabled. `S8.30-1-2-conflict.png` |
| S8.30 | 2 | ✅ PASS | «По инструменту SBER на счёте «Сэндбокс» уже есть сессия (на паузе) №3…». Backend-текст — `test_message_names_session_account_and_reason`. Для счёта без имени см. дефект D-1 |
| S8.30 | 3 | ⚠️ BLOCKED | У каждого пользователя копии один sandbox-счёт, второго счёта для того же тикера нет. `test_session_uniqueness::test_same_ticker_other_account_allowed` (passed) |
| S8.30 | 4 | ✅ PASS | GAZP на том же счёте: предупреждения нет, «Запустить» активна (не нажималась). `S8.30-4-other-ticker.png`. Backend: `test_other_ticker_same_account_allowed` |
| S8.30 | 5 | ✅ PASS | ТФ 1 час вместо 5 минут — по-прежнему предупреждение и disabled. Строчные `sber` запрет тоже не обходят. `S8.30-5-other-tf.png` |
| S8.30 | 6 | ✅ PASS | Через UI запущены paper SBER: #10 и #11 (1m, одна версия 90) и #12 (5m) — все active, конфликта в форме нет. `S8.30-6-paper-dups.png` |
| S8.30 | 7 | ✅ PASS | У каждой сессии своя сделка (#55/#56/#57) и своя строка `paper_portfolios`. Закрытие у #12 не изменило ни stats, ни открытую сделку #11. `s8_30_paper` |
| S8.30 | 8 | ⏸ ORDER | Нужно нажать «Возобновить» у sandbox-сессии — запрещено. `test_resume_blocked_by_active_sibling` (passed). Сценарий A6 |
| S8.30 | 9 | ✅ PASS | #3 в статусе stopped: форма sandbox / «Сэндбокс» / SBER без предупреждения, кнопка активна. `S8.30-9-stopped-no-conflict.png`; `test_stopped_session_does_not_block` |
| S8.30 | 10 | ⏸ ORDER | Нужен рестарт с sandbox-сессиями в статусах active/suspended, рантайм поднял бы их. `test_restore_pair_priority.py` (10 passed), `test_suspended_wins_pair_over_paused_sibling` |
| S8.30 | 11 | ⏸ ORDER | `test_reconcile_account_scope.py` (7 passed) |
| S8.31 | 1 | ⏸ ORDER | У остановленной sandbox #3 кнопка «Запустить заново» активна, причины блокировки нет (`S8.31-1-sandbox-stopped.png`). Нажатие запускает sandbox-сессию. Тело запроса — vitest `SessionDashboard.rerun.test.tsx` |
| S8.31 | 2 | ⏸ ORDER | Real-сессий в копии нет; real — только заказчик |
| S8.31 | 3 | ✅ PASS | Paper #9 (GAZP 1h, 0,2 %) → «Запустить заново» → #13: paper, GAZP, 1h, 0.2000, та же версия. `S8.31-3-4-paper-rerun.png` |
| S8.31 | 4 | ✅ PASS | После клика URL `/trading/sessions/13`, в шапке «#13» |
| S8.31 | 5 | ❌ FAIL | Кнопка disabled, номер №6 и тикер SBER видны, но счёт не назван: «на счёте «»». D-1. `S8.31-5-rerun-conflict.png` |
| S8.31 | 6 | ✅ PASS | У #1 в копии `timeframe=NULL` → кнопка disabled, «У сессии не сохранён таймфрейм… Запустить заново нельзя…». `S8.31-6-no-timeframe.png` |
| S8.31 | 7 | ✅ PASS | Paper запускается без счёта (S8.37 / S8.30-6), sandbox требует выбор счёта (`broker-select`), конфликт подсвечивается до нажатия (S8.30 п.1) |
| S8.32 | 1 | ✅ PASS | #6 переведена в active в копии: `DELETE /broker-accounts/4` → 409 «№6 (SBER, работает…)». В UI — тост отказа, модалка открыта, счёт на месте. `S8.32-1-refusal-active.png` |
| S8.32 | 2 | ✅ PASS | paused — через UI (тост, модалка открыта); suspended — через API, 409 «№6 (SBER, приостановлена рестартом сервера…)». `S8.32-2-3-4-refusal-paused.png` |
| S8.32 | 3 | ❌ FAIL | Номер, тикер и статус перечислены: «Нельзя удалить счёт: №6 (SBER, на паузе, открытых позиций: 0). Работающие торговые сессии остановите.» Нет подсказки «остановите их на странице Торговля». D-2 |
| S8.32 | 4 | ✅ PASS | После отказа диалог «Удалить аккаунт» остаётся видимым |
| S8.32 | 5 | ✅ PASS | #4 только со stopped-сессиями #4/#6 → «Аккаунт удалён», в API счёта нет. `S8.32-5-deleted.png` |
| S8.32 | 6 | ✅ PASS | После отказа у #6 `broker_account_id=4`, статус прежний (paused) |
| S8.32 | 7 | ✅ PASS | #2 без сессий → удалён. `S8.32-7-deleted-no-sessions.png` |
| S8.35 | 1 | ⏸ ORDER | Нужна запущенная sandbox-сессия. `test_figi_negative_cache::test_second_signal_does_not_reach_broker`, `test_miss_is_remembered_per_ticker` |
| S8.35 | 2 | ⏸ ORDER | `test_lookup_repeats_after_ttl`, `test_success_forgets_earlier_miss` |
| S8.35 | 3 | ⏸ EVENT | `test_source_failure_is_not_cached`, `test_transport_error_is_not_an_answer` |
| S8.35 | 4 | ⏸ ORDER | `test_figi_resolution::test_sandbox_trade_gets_real_figi_from_broker` |
| S8.37 | 1 | ✅ PASS | `paper-commission-source`: «Ставка вручную» / «Тариф брокерского счёта». `S8.37-1-switch.png` |
| S8.37 | 2 | ✅ PASS | Выбран «Сэндбокс» → «Тариф счёта: 0,3 % с каждой стороны сделки.» `S8.37-2-tariff-hint.png` |
| S8.37 | 3 | ✅ PASS | Счёт с тарифом 0 → оранжевый текст «тариф не задан (0 %) — комиссия удерживаться не будет…». `S8.37-3-zero-tariff.png` |
| S8.37 | 4 | ✅ PASS | Тариф без счёта → «Выберите счёт, тариф которого применить», модалка открыта, число сессий не изменилось. `S8.37-4-no-account.png` |
| S8.37 | 5 | ✅ PASS | #7 (тариф 0,3 %): вход 9 741,34 ₽ → комиссия 29,22 в колонке. После закрытия 58,45 = вход + выход. `S8.37-5-*.png` |
| S8.37 | 6 | ✅ PASS | Тариф #3 сменён на 0.1 после запуска — у #7 осталось 0.3000 |
| S8.37 | 7 | ✅ PASS | Режим Sandbox: нет ни переключателя, ни полей комиссии. `S8.37-7-sandbox-no-commission.png`. Real в копии недоступен (нет production-счёта с торговыми правами), условие `mode === 'paper'` одно на оба режима (`LaunchSessionModal.tsx:701`) |
| S8.43 | 1 | ✅ PASS | «Счёт», #2 (реальный, с бумагами): Всего 1 390 855,04 = Доступно 8 099,46 + В позициях 1 382 755,58, совпадает с `/broker/accounts/balances`. `S8.43-1-account.png`, `S8.43-api.json` |
| S8.43 | 2 | ✅ PASS | Счёт только с деньгами (#3, рубли): API `in_positions=0`, `total=available`. На экране «Счёт» sandbox не показывается — решение 2026-04-28. Вариант «рубли + валюта» — `tests/unit/test_broker/test_balance_available.py` (passed), формула `mapper.py:239` (портфель − валюты, ≥ 0) |
| S8.43 | 3 | ⏸ TELEGRAM | Бот на стенде выключен. `/balance` берёт те же `available` / `in_positions` из `adapter.get_balance` (`telegram_webhook.py:741–751`) |

## Дефекты

### D-1 (S8.31 п.5; задевает S8.30 п.2) — счёт без имени в тексте конфликта отображается как «» — low
- Шаги: у testuser1 sandbox-счёт #4 с `name=""`. Сессия #6 на нём на паузе, открыть остановленную #4 (SBER, тот же счёт).
- Ожидалось: под шапкой названы номер, тикер и счёт.
- Получено: «По инструменту SBER на счёте «» уже есть сессия (на паузе) №6…». Тот же пустой текст будет в форме запуска и в модалке удаления («удалить аккаунт «»?»).
- Корень: имя sandbox-счёта приходит от брокера пустым, `broker/service.py:535` (`name=info.name`).
  Подстановка на фронте рассчитана только на `undefined`: `account?.name ?? `№${id}`` в
  `components/trading/LaunchSessionModal.tsx:406` и в `SessionDashboard.tsx` (`rerunBlockReason`,
  ~стр. 293). Для пустой строки она не срабатывает.
- Исправление: `account?.name || `№${id}``. Плюс имя по умолчанию при регистрации (например, «Sandbox …») либо подстановка в `BrokerAccountResponse`.

### D-2 (S8.32 п.3) — из отказа удаления счёта пропала подсказка «на странице «Торговля»» — low
- Шаги: счёт с сессией на паузе → «Удалить».
- Получено: «… №6 (SBER, на паузе, открытых позиций: 0). Работающие торговые сессии остановите.»
- Ожидалось по чеклисту и ФТ v3.8 п.(3): подсказка «остановите их на странице Торговля».
- Корень: регрессия формулировки в `cb19c72` (S8R-FIX-052), `backend/app/broker/service.py:98`. До этого
  было «Остановите их на странице «Торговля», затем удалите счёт.». Решения заказчика о смене текста в
  карточке FIX-052 нет. Соседняя подсказка для stopped-сессий место называет («на странице сессии»).
- Исправление: «Работающие торговые сессии остановите на странице «Торговля».»

### D-3 (вне пунктов, брокерский контур) — переоткрытие sandbox-счёта привязывает запись к счёту, уже занятому записью другого пользователя — medium
- Как найдено: testuser1 открыл «Настройки → Брокер». `GET /broker-accounts/4/sandbox-balance` получил
  `NOT_FOUND 50004` по устаревшему `8de9093c…`. В логе `sandbox_account_reopened broker_account_id=4
  new_account_id=f925da17-…`: запись #4 (testuser1) стала указывать на тот же живой sandbox-счёт, что
  и #3 (sergopipo). Изменилась только копия БД, у брокера ничего не открывалось (вызовов
  `open_sandbox_account` нет, список счетов токена был непуст).
- Корень: `backend/app/broker/sandbox_recovery.py:356–384`, `_account_ids_taken_by_siblings`.
  «Занятые» счета ищутся только среди записей того же пользователя (`BrokerAccount.user_id ==
  account.user_id`), а у двух пользователей с одним токеном коллизия остаётся. Это ровно тот случай,
  о котором предупреждает комментарий в `_refresh_locked`: «баланс и позиции задвоятся, закрытие
  позиции одной записи «съест» позицию другой».
- Чем грозит: запрет «один инструмент — одна сессия на счёте» и сверка ключуются по
  `broker_accounts.id`. Две записи на один реальный счёт обходят запрет S8.30 и дают ложные
  расхождения или «возможно, куплено вручную».
- Исправление: проверять занятость `account_id` по всем записям (или по токену), а не по пользователю.
  Альтернатива — UNIQUE(`account_id`, `is_sandbox`) с честным `UNAVAILABLE`.

## Попутные находки (не дефекты пунктов)

1. **Лимитер в dev-режиме.** Одна загрузка «Настроек» под `vite dev` (StrictMode) даёт около 45
   запросов, из них `sandbox-balance` — 5 раз. При лимите `general` 200/мин быстрые переходы ловят 429.
   В спеках qa4 — пауза 35 с между тестами (`qa4/util.ts`). В production двойных запросов нет, на
   пользователя не влияет.
2. **Стенды-копии делят каталог `wt-s8r-check/backend/data`**: `tax_reports/tax_report_<id>.*`
   нумеруются от 1 в каждой копии, `.last_shutdown_at` общий. Параллельные выгрузки 3-НДФЛ на разных
   стендах перезапишут файлы друг друга, а маркер останова одного стенда виден остальным.
3. **Флак теста.** `tests/test_broker/test_audit_s8r_delete_account_race.py::test_start_session_then_delete_account_is_rejected`
   в общем прогоне падает: `Lock … is bound to a different event loop`, `broker/service.py:1017`. Отдельно — 2 passed.
   Лок реестра `keyed_lock` переживает event loop предыдущего теста. Это изоляция тестов, не продукт.
4. **Paper-комиссия показана как «≈ … Оценка по тарифу счёта, а не удержание брокера».** Формально
   верно (`estimated`), но подпись про брокера на бумажной сессии может запутать. Предложение —
   отдельная подсказка для paper.
5. **Запись в каталог `Develop/`.** Прогон vitest в worktree обновил кэш
   `Develop/frontend/node_modules/.vite/vitest/<hash>/results.json` (через симлинк `node_modules`). Это
   кэш порядка тестов, код и данные не затронуты, но запись в `Develop/` формально была.

## Предложения по формулировкам OUTDATED

Пунктов OUTDATED нет. Уточнения:
- S8.22 п.6: «Комиссия, подтверждённая брокером (**только реальный счёт** — песочница комиссию не
  удерживает), показана без знака ≈». Иначе пункт читается как проверяемый на sandbox.
- S8.29 п.4: дописать «…для активной сессии; у сессии на паузе пометку снимает кнопка «Сверить с
  брокером» (S8R-FIX-070)». Фоновая сверка паузы не обходит (`runtime.py:1684`, докстринг).

## Что нужно от оркестратора

Сценарии на **sandbox #3** (`f925da17-…`) — по согласованию с заказчиком. Стратегия — копия «Тестовая»
(v90 покупает на первой свече при RSI>0), но с `stop_loss_pct = take_profit_pct = 0,1`, чтобы уровень
сработал за минуты. ТФ 1m, сайзинг «фикс. лоты = 1», тариф #3 = 0,05 %.

- **A1. GAZP (лот 10), 1 лот, покупка → выход по SL/TP 0,1 %.** Закрывает S8.22 п.4–5, S8.24 п.4–5,
  S8.25 п.1–4, S8.27 повторно на sandbox, S8.29 п.1, S8.35 п.4.
  Ожидается:
  - у сделки настоящий FIGI `BBG004730RP0`, в логе `figi_synced`, бейджа нет;
  - комиссия «≈» по 0,05 % на каждую сторону, начислена один раз;
  - `exit_price` = цена исполнения у брокера, а не уровень;
  - на «Счёт» (#3 через API `/broker/accounts/3/positions`) позиции GAZP нет;
  - уведомление «P&L <net> ₽ · комиссия … (оценка)»;
  - в 3-НДФЛ 2026 строка GAZP: «Кол-во, шт» = 10, «Фин. результат» = P&L − комиссия, вход + выход = комиссия журнала.
- **A1-paper (S8.22 п.7).** Paper-сессия на тех же параметрах: формула чистого P&L совпадает с A1 при равных ценах.
- **A2. Двойное закрытие (S8.28 п.9).** На открытой позиции GAZP дважды быстро «Закрыть». Ожидается один
  ордер; второй ответ — «Ордер на закрытие уже отправлен брокеру…».
- **A3. Позиция продана вне терминала (S8.28 п.1–5, S8.29 п.3–4, п.7–8).** Открыть 1 лот GAZP, затем
  продать его прямым sandbox-ордером вне терминала. В течение 15 мин:
  - бейдж с «у брокера 0 шт, у нас 10 шт (сделки №…)»;
  - сессия на паузе, сделка открыта, уведомление одно.
  Затем докупить 1 лот вручную → «Сверить с брокером» / «Возобновить» → бейдж снят. При повторном
  расхождении — новое уведомление о паузе.
- **A4. Ручная докупка (S8.28 п.6, S8.29 п.6).** При открытой сделке GAZP сессии докупить 1 лот GAZP
  вне терминала. Ожидается «у брокера 20 шт, у нас 10 шт», без паузы. Без открытой сделки — текст «(возможно, куплено вручную)».
- **A5. Две сессии на одном счёте (S8.29 п.5, S8.30 п.11).** Сессии GAZP и SBER на #3 одновременно: обе
  active, после сверки ложного расхождения нет.
- **A6. «Возобновить» при конфликте (S8.30 п.8).** GAZP-сессия X active; остановленную ранее GAZP-сессию Y
  на #3 перевести в копии БД в paused и нажать «Возобновить» у Y. Ожидается 409 с номером X; Y не
  поднимается (если поднимется — сразу «Стоп»).
- **A7. Рестарт с конфликтной парой (S8.30 п.10).** X active, Y suspended (в БД) → рестарт backend.
  Ожидается: поднята X, Y на паузе, уведомление «Пара счёт+тикер занята сессией #X».
- **A8. «Запустить заново» для sandbox (S8.31 п.1).** Остановленная sandbox-сессия → «Запустить
  заново». Ожидается новая sandbox-сессия с тем же счётом, тикером и ТФ; открывается её карточка.
- **A9. FIGI не резолвится (S8.35 п.1–2).** Sandbox-сессия по тикеру, которого нет у T-Invest
  (ордеров не будет — FIGI не найден). Ожидается одна запись `figi_unresolved` на серию свечей и
  повтор через 15 мин.
- **A10. Вне торговых часов (S8.25 п.5).** Позиция открыта к концу торгов, уровень пересечён после
  закрытия — сделка остаётся открытой.

Только заказчик (реальный счёт): S8.22 п.6 (комиссия, удержанная брокером, без «≈») и S8.31 п.2.

События:
- S8.25 п.6 — отказ брокера при автозакрытии;
- S8.28 п.10–11 — сбой песочницы 70001 на отправке;
- S8.29 п.10 — недоступность брокера во время сверки;
- S8.35 п.3 — кратковременный обрыв связи;
- Telegram: S8.43 п.3 (`/balance`); после включения бота — S8.27 п.1 в канале Telegram.

Решение нужно по D-3 (поведение при двух пользователях с одним токеном).
