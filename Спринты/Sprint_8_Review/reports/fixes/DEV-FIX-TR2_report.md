## DEV-FIX-TR2 отчёт — S8R fixes, FIX (с ревью р.1)
Статус: ✅ готово к коммиту — 062, 065, 066 и 063 (источник `stages`; консервативная оценка для CB — вопрос заказчику)

### 1. Что реализовано
- **062:** нереализованный P&L считается общими хелперами `position_unrealized_pnl` и `position_quantity` из `app/trading/unrealized.py`, по исполненным лотам (`RiskMonitor.position_lots`). Пять потребителей:
  - `TradingService._fill_session_card_data` и `get_positions`;
  - `unrealized_pnl_by_session` (через неё — CB: дневной лимит и equity, а также дневной отчёт);
  - дашборд стратегии;
  - Telegram: `/positions` и меню `/close`.

  Объём позиции на карточке (`active_position_lots`), в `get_positions`/`get_all_positions` (`volume_lots`) и в Telegram — тоже исполненные лоты.
- **065:** `trade.closed` и `position.mismatch` добавлены в `OUTBOX_EVENTS`/`OUTBOX_REPLAY_EVENTS`.
  - Строки пишутся в `_settle_close_locked` (`_stage_close_events`) той же транзакцией, что запись закрытия, пометка и пауза (`pause_from` пробрасывается из recovery).
  - Публикация в шину — после commit, с `outbox_id`.
  - Для sandbox/real вся запись — один commit под `run_shielded_from_cancel`, и для полных, и для частичных закрытий.
  - Recovery после commit вызывает `NotificationService.deliver_staged_events`; отменённое досылает повтор outbox.
  - Хуки `on_partial_exit`/`_settle_and_notify`/`_notify_partial_exit`/`_flag_partial_exit` удалены, вместо них колбэк `after_commit`, который вызывается вне защиты.
- **066:** `validation_error_detail` — один проход `jsonable_encoder` без `input` (datetime выходит в ISO), затем неконечные числа на любой глубине заменяются строками `inf`/`-inf`/`nan`. `ctx.error` передаётся текстом исключения.
- **063:** третий источник в `entry_fill_price` — средневзвешенная `stages[].price` по `stages[].quantity`. По контракту `OrderStage`: `price` — за 1 инструмент, `quantity` — в лотах, поэтому лотность на результат не влияет. Значение считает маппер (`OrderStatus.stages_average_price`). Правило общее для входа и дозапроса при закрытии.
  - Операции счёта как источник отвергнуты: в FAQ T-Invest сказано, что идентификатора поручения в операции нет.

### 2. Файлы
- Изменены (app): `trading/{engine,runtime,service,unrealized,session_status}.py`, `common/{exceptions,outbox}.py`, `notification/{service,telegram_webhook}.py`, `strategy/service.py`, `broker/base.py`, `broker/tinvest/mapper.py`.
- Изменены (тесты под новый механизм): `test_fix_048…` (recovery-уведомление — реальный `NotificationService`), `test_reconcile_review_fixes` (`deliver_staged_events`), `test_telegram_webhook_helpers` (у фикстуры `filled_lots=None`).
- Новые тесты: `test_fix_062_*` (2 файла), `test_fix_063_entry_price_from_stages.py`, `test_fix_065_*`, `test_fix_066_*`.

### 3. Тесты
- **RED:**
  - 062 р.1: `assert 10 == 3` (`active_position_lots`), Telegram `10 лот(ов) … +1 000.00 ₽`;
  - 063: `assert None == Decimal('325.66666667')`, `'pending' == 'filled'`;
  - 066 р.1: `'2026-01-02 03:04:00' == '2026-01-02T03:04:00'`.
- **Мутации** (все красные, откачены с проверкой md5):
  - helper по `volume_lots` → `1000.00 == 300.00`;
  - без источника `stages` → `None`;
  - без `_finite_or_text` → `Out of range float values`;
  - без outbox → `CancelledError`/`TimeoutError` в тестах 065;
  - без shield для всех sandbox/real → `['trade.closed'] == [… 'position.mismatch']` и `'filled' == 'closed'`.
- **Гейты:** pytest 5532 passed / 1 skipped / 0 xfailed / 0 failed; ruff 0; mypy Success (202); bandit M0/H0; typecheck 0; lint 0; build ok. vitest: фронт не менялся — запуск на уровне пакета. Тесты 062/063/065 прогнаны 5 раз подряд — без флейков.

### 4. Integration points
- ✅ `runtime.py:2589` → `deliver_staged_events`
- ✅ `engine.py:7085` → `_stage_close_events`
- ✅ `mapper.py:428` → `stages_average_price`
- ✅ хелперы — 5 вызывающих
- ✅ `main.py:486` → обработчик 422

### 5. Контракты
- `PositionResponse.volume_lots` и `active_position_lots` — теперь объём позиции (исполненные лоты).
- 422: без `input`; нечисла — строкой.
- В payload `trade.closed`/`position.mismatch` появилось поле `outbox_id`.
- Миграции нет.

### 6. Проблемы / правки документов / находки
- **Бюджет «Стопа»:** grace остаётся `ORDER_CANCEL_GRACE_SEC`. Под защитой — один commit (≤ busy_timeout 30 с) плюс чтения и шина, без сети; укладывается в 45 с. Стоп по-прежнему 275 с — это закреплено тестом.
- **Предлагаемые правки документов:**
  - ФТ §7.5: «Нереализованный P&L и объём частично исполненной позиции — по исполненному объёму во всех местах (карточка, позиции, CB, отчёт, Telegram)».
  - ТЗ §2.4: «`trade.closed`, `position.mismatch` — в outbox транзакции закрытия».
  - ТЗ §5.4: «Запись закрытия sandbox/real — один commit под защитой; цена исполнения — средняя → сумма → средневзвешенная `stages`».
  - ТЗ §4: «422 без `input`».
  - `tinvest_api_services.md`: «`OrderStage.price` — за штуку, `quantity` — лоты».
- **Новые находки:**
  - Другие издатели `trade.closed` (`risk_monitor.py:846`, `engine.py:4495/5963`, `runtime.py:4602`) по-прежнему пишут мимо outbox.
  - Paper-закрытие тоже пишет `trade.closed` в outbox: без слушателя шины оно будет дослано при старте, с задержкой до 1 ч.
  - Журнал сделок (`TradesTable`) показывает `filled/volume` — это осознанно, не менялось.
- **Самопроверка:**
  1. outbox откатывается вместе с закрытием;
  2. одно уведомление — строка забирается один раз;
  3. реальные пути (`close_position`, `_recover_orphan_exit_orders`, `_listen_loop`, `replay`);
  4. под защитой нет сети;
  5. вызывающие проверены;
  6. —

### 7. Применённые Stack Gotchas
38, 48, 74, 75, 77 (повторная отмена), 78.

### 8. Новые Stack Gotchas
- Кандидат: обработчик 422 по умолчанию + `allow_nan=False` даёт 500.
- Кандидат: StaticPool в тестах — параллельные транзакции на одном соединении дают «expected to update 1 row».

### 9. Плагины
py_compile, mypy; typecheck; context7 (invest-contracts: `OrderStage`, FAQ operations); WebSearch; tdd.

---

## Ревью р.2
Статус: ✅ готово к коммиту.

### Что сделано
1. **Цена выхода.** Правило цены исполнения теперь одно — `order_fill_price` (было `entry_fill_price`): средняя → сумма/(лоты×лот) → сделки ордера; ноль — не цена. Как и у входа, последним источником остаётся `response.price` > 0. Правило применено к закрытию (ветки `filled` и опроса), к recovery (`finalize`) и к `_extend_partial_entry`.
   - Если у выхода нет цены ни в одном источнике: `_apply_close(exit_price_unconfirmed=True)` → `pnl` и `exit_price` NULL, оценка по кэшу рынка — в `exit_signal_price`.
   - `is_pnl_undetermined(..., exit_price=, exit_signal_price=)` признаёт такую сделку «P&L не определён» (пометка в сделке — пара «`exit_price` NULL + `exit_signal_price` задан»; legacy-строки без обеих цен не попадают).
   - В payload текст «не определён (брокер не сообщил цену выхода)».
   - `exit_turnover` при пустом `exit_price` берёт `exit_signal_price`.
2. **Отключение адаптера.** Свой адаптер отключается после записи закрытия для всех позиций (раньше обычная позиция отключала его до записи), под `run_shielded_from_cancel(ADAPTER_RELEASE_GRACE_SEC)`. На ошибочном пути — тоже под защитой.
3. **Доставка без слушателя.** В `app/common/outbox.py` добавлен общий хелпер: `stage_for_delivery` → после commit `publish_staged` (шина/WS) и `deliver_staged` (доставщик `configure_outbox_deliverer` = `NotificationService.deliver_staged_events`, подключён в lifespan, при остановке снимается). Дедупликация — по `processed_at` строки.
   - В закрытии: запись + шина — под защитой, доставка — после неё.
4. **Все издатели `trade.closed`** идут через `stage_trade_closed` + `publish_and_deliver`: paper SL/TP (`RiskMonitor`, `_refresh_and_publish` удалён), отмена pending, сверка после переоткрытия sandbox.
   - Вторая публикация `trade.closed` в `runtime` (выход по сигналу) удалена: она дублировала уведомление движка и шла без `outbox_id`.
5. **`_extend_partial_entry`** — общее правило цены.
6. **`_deliver_rows`** — общий для `replay_pending_events` и `deliver_staged_events`; различается только выборка.
7. **Без лишних запросов:** повторный `refresh` после commit убран; владелец читается в `_apply_close_and_settle` до секции портфеля и вне защиты. Владельцы при сверке — одним запросом (`_session_owner_ids`).
8. Докстринг Telegram исправлен на исполненные лоты.

### Тесты
- Новый файл `test_fix_tr2_review2.py`, 9 тестов:
  - выход: цена из `stages`; без цены → `pnl` NULL, а не −6 000; recovery без цены;
  - отмена в `disconnect` → сделка `closed`, канал отключён;
  - пауза без слушателя → доставлено сразу, со слушателем — одно;
  - paper SL/TP: отмена между commit и publish → доставлено повтором;
  - отмена pending;
  - довыполнение остатка входа с `average=0`.

  Файловая БД + NullPool, 3 прогона подряд — зелёные.
- **Мутации** (все красные, откачены с проверкой md5):
  - `state.average_price` вместо правила → `-6000.00 is None`;
  - без `deliver_staged` → `0 == 1`;
  - без staging SL/TP → `CancelledError`/`TimeoutError`;
  - `_extend` только по средней → `300 == 305`;
  - отключение до записи → `'filled' == 'closed'`.
- **Существующие тесты:** 048 — порядок `settle → disconnect`, доставщик через monkeypatch; reconcile — доставщик через monkeypatch.
- **Гейты:** pytest 5541 passed / 1 skipped / 0 failed (лог `pytest_tr2_r2.log`; 14 мин — машина нагружена); ruff 0; mypy Success (202); bandit M0/H0; typecheck 0; lint 0; build ok. vitest: фронт не менялся — запуск на уровне пакета.

### Integration
✅ Все подключены:
- `main.py` lifespan → `configure_outbox_deliverer`;
- `engine._apply_close_and_settle` → `deliver_staged`;
- `risk_monitor`, `_cancel_pending_trade`, `_reconcile_locked` → `stage_trade_closed` + `publish_and_deliver`.

### Изменения поведения и правки документов
- ФТ §7.5/§12: «выход без цены брокера — P&L не определён».
- ТЗ §2.4: «`trade.closed`/`position.mismatch` — outbox транзакции закрытия у всех издателей; доставка сразу после commit записавшим путём; слушатель/повтор — дедупликация по строке».
- ТЗ §5.4: «цена выхода — то же правило, что вход; свой адаптер — после записи».

### Находки
- Последний запасной источник цены (`response.price`) у входа и выхода в песочнице — сумма исполнения, а не цена за штуку (gotcha-33). Так было и раньше у входа; при пустом `GetOrderState` возможна завышенная цена.

---

## Ревью р.3
Статус: ✅ готово к коммиту.

### Что сделано
1. **Запасная цена из ответа брокера (gotcha-33).** Новая функция `response_fill_price` и константа `RESPONSE_PRICE_TOLERANCE = 0.20` с комментарием.
   - Кандидаты: `p` и `p / (лоты × лот)`. Берётся первый, который отстоит от оценки не больше чем на ±20 %.
   - Оценка: для входа — цена сигнала; для выхода — рыночная цена (новый `_market_price_or_none`, без подмены ценой входа).
   - Нет попадания или нет оценки — источник не используется: вход остаётся `pending`, выход — «P&L не определён».
   - Применено ко входу и к выходу.
2. **Доставка при отмене.** События закрытия после commit складываются в `OrderManager._staged_events` (внешний `staged_sink`, если лок берут снаружи).
   - Контекст `publish_staged_after(sink)` срабатывает при выходе из блока лока. При обычном исходе — шина сразу, доставка — `schedule_delivery`. При исключении или отмене — `schedule_publish_and_deliver`.
   - Фоновые задачи держатся в реестре с сильными ссылками (`_track`); done-callback логирует отмену или сбой задачи; shutdown их дожидается.
   - Путь «без цены входа» ничего не теряет: события попадают в sink внутри защищённой записи.
3. **Порядок.** Публикация и доставка — после снятия лока `close_trade` и после отключения своего адаптера.
   - Покрыто: `close_position`, `finalize_exit_order`, вход (`_submit_order_to_broker`), `settle_entry_remainder`, recovery pending (runtime), сверка после переоткрытия, paper SL/TP.
   - В тесте: `["disconnect", "publish:trade.closed:locked=False"]`.
4. **Налоговый отчёт.** Добавлен раздел «Сделки без подтверждённой цены — уточните в брокерском отчёте»: дата, тикер, направление, режим, кол-во в штуках, известные цены, оценка выхода «≈ X (не подтверждена)», комиссия.
   - Раздел вне базы и итогов; csv и xlsx строятся одной функцией `unpriced_rows`.
   - Сделано в текущей структуре `_export_*`. При cherry-pick вместе с пачкой TB (там общий `_report_rows`) будет конфликт в `_export_xlsx`/`_export_csv`: разделу нужен только вызов `unpriced_rows(unpriced)` в конце файла.
5. **Нет получателя.** Если `owner_id` None — строка outbox не пишется (WARNING `outbox_stage_skipped_no_recipient`), событие уходит в шину без `outbox_id`. Со слушателем — одно уведомление.
6. **Фронт.**
   - `LiveTrade.exit_signal_price`.
   - Маркер выхода без цены брокера — по `exit_signal_price`: серый, стрелка пунктирным контуром, «≈» перед ценой. Подсказку при наведении на canvas не сделать, поэтому неподтверждённость видна по стилю.
   - В журнале — «≈ X (не подтверждена)» с подсказкой; подсказка «P&L не определён» говорит, какой цены нет — входа или выхода.
7. **Остальные `position.mismatch`.** Вход без цены (`_entry_without_price`) и остаток входа (`_settle_entry_remainder_locked`) идут через `_stage_position_mismatch` → outbox той же транзакции, публикация после снятия лока.
8. **Владельцы сессий.** `strategy_version_owner_ids` (набор id) — единственный join; `strategy_version_owner_id` и `_session_owner_ids` используют его.

### Тесты
- **Новые:**
  - `test_fix_tr2_review3.py` (9 тестов): правило запасной цены; FILL 9 050 за 30 шт. → 301.67, long и short; без оценки → не определён; отмена во время записи без слушателя → доставлено; порядок; нет получателя; `position.mismatch` входа и остатка.
  - `test_fix_063_unpriced_section.py` (2 теста): раздел одинаковый в csv и xlsx, база = 100.
  - vitest: `TradeMarkerPrimitive.test.ts` (2 теста); +1 тест в `useTradeMarkers`; +1 тест в `TradesTable`.
- **Мутации** (все красные, откачены с проверкой md5):
  - `positive_price(response.price)` → `9050.00000000 == 301.66666667`;
  - без фоновой задачи в ветке исключения → `0 == 1`;
  - публикация под локом → `locked=True`;
  - строка outbox без получателя → `[PendingEvent] == []`;
  - mismatch входа и остатка не добавлен в sink → `0 == 1`;
  - фронт: без оценки выхода → `expected ['entry-long'] …`, `expected '—' to contain '≈'`.
- **Существующие тесты:**
  - w8g, c6 close_all и stop happy-path — засеян кэш рынка (запасная цена теперь требует оценки);
  - 048 и 065 — `wait_background_tasks()` перед чтениями (StaticPool).
- **Гейты:** pytest 5552 passed / 1 skipped / 0 failed (лог `pytest_tr2_r3.log`, 13 мин); ruff 0; mypy Success (202); bandit M0/H0; typecheck 0; lint 0; build ok. vitest затронутых файлов — 15/15. Новые тесты — 3 прогона подряд зелёные.

### Integration
✅ `publish_staged_after` — 7 держателей лока; `schedule_delivery`/`schedule_publish_and_deliver` — через него; `_stage_position_mismatch` — 2 издателя; `unpriced_rows` — оба экспортёра; `response_fill_price` — вход и выход.

### Текст для `deployment_guide.md` §3.3
«Lifespan: после `configure_outbox_factory` подключается доставщик outbox (`configure_outbox_deliverer(notification_service.deliver_staged_events)`). Уведомления о закрытии позиций (`trade.closed`, `position.mismatch`) пишутся в `pending_events` той же транзакцией, что запись закрытия. Сразу после commit и снятия локов их доставляет фоновая задача — без ожидания слушателя сессии и в том числе после отмены («Стоп», дедлайн, shutdown). Не доставленное к остановке процесса досылает повтор при старте (`replay_pending_events`, окно 1 с — 1 ч). Shutdown дожидается фоновых задач outbox (`wait_background_tasks`) в пределах общего дедлайна. Получатель — владелец стратегии; строки без получателя не пишутся.»

### Находки
- Подсказку при наведении на маркер canvas без отдельного слоя hit-test не сделать; неподтверждённость показана стилем.

---

## Ревью р.4 (финальный раунд)
Статус: ✅ готово к коммиту.

### Что сделано
1. **Нет рыночной оценки.** `exit_signal_price` не пишется (NULL — не цена входа и не 0). В `_apply_close` добавлен параметр `exit_estimate`.
   - Пометка «P&L не определён» у выхода без цены теперь ставится по паре «`exit_price` NULL» + («оценка есть» или «встречный ордер известен», `exit_order_id`). Legacy-закрытия paper без цен и ордера под неё по-прежнему не попадают.
   - `TradeResponse.exit_broker_order_id` (поле добавлено).
   - Журнал показывает «—», маркер выхода не рисуется, в налоговом разделе — «—».
2. **Запасная сверка `response.price`.** Без рыночной оценки — сверка с ценой входа сделки; нет ни того, ни другого — источник не используется.
3. **Shutdown.** Фоновые задачи outbox ждём ДО внешней доставки (`SHUTDOWN_OUTBOX_WAIT_SEC = 6`, `SHUTDOWN_DISPATCH_WAIT_SEC = 15`).
   - Бюджет: 10 (drain) + 45 (рантайм) + 6 + 15 = 76 → `stop_grace_period` в `docker-compose.yml` поднят с 60 с до 90 с.
   - Тест compose учитывает хвост lifespan.
   - Добавлен тест порядка через настоящий `lifespan`.
4. **Вход:** `filled_qty or volume_lots`, как на выходе.
5. **Налоговый отчёт:** предупреждение о песочнице (`_has_sandbox`) и пометка эвристики типа учитывают раздел «без подтверждённой цены».
6. **Подсказка «P&L не определён»** называет недостающие цены по факту: входа / выхода / входа и выхода.
7. Мёртвый `publish_and_deliver` удалён; докстринги `stage_for_delivery`/`stage_trade_closed` ссылаются на `publish_staged_after`.
8. `schedule_staged_publish` идёт через `_track`.
9. Повторного `last_cached_bar` больше нет: `_market_exit_price` удалён, на закрытии одно чтение `_market_price_or_none`.

### Тесты
- **Новые:**
  - `test_fix_tr2_review4.py` (3 теста): пустой кэш → `exit_signal_price` NULL и undetermined; real, состояние упало, кэша нет → 310 принята; вход FILL с суммой и `lots_executed=0` → 326.00;
  - tax: песочница и эвристика по разделу;
  - lifespan: порядок outbox → dispatch;
  - compose: бюджет;
  - vitest +4 (подсказка ×3, «—» в журнале, маркер не рисуется).
- **Существующие тесты:** r3 «без оценки» переписан под п.2 (кандидаты вне ±20 % и от цены входа); concurrency-тест гейтит `_market_price_or_none` (единственное чтение цены).
- **Мутации** (все красные, откачены с проверкой md5):
  - оценка = цена входа → `300.00000000 is None`;
  - без сверки с ценой входа → `None == 310`;
  - `filled_qty` без запасного → `None == 326.00`;
  - песочница только по лотам → нет предупреждения;
  - нет ожидания outbox → `outbox_waited` отсутствует;
  - фронт: без ветки «входа и выхода» → подсказка не та.
- **Гейты:**
  - pytest 5558 passed / 1 skipped / 0 failed (лог `pytest_tr2_r4b.log`, 13 мин).
  - Первый прогон (`pytest_tr2_r4.log`) дал 2 падения. Первое — тест r3, переписан. Второе — флейк под нагрузкой `test_commission_settle::test_extension_between_read_and_lock_skips_pass`: `COMMISSION_LOCK_TIMEOUT_SEC = 5` истёк раньше рандеву; код не тронут, отдельно 5/5 зелёный.
  - ruff 0; mypy Success (202); bandit M0/H0; typecheck 0; lint 0; build ok.
  - vitest затронутых файлов: 20/20.

### Финальные тексты для документов
- **ФТ §7.5 / §12 (торговля, журнал сделок):** «Если брокер исполнил ордер, но не сообщил цену (ни средней, ни суммы, ни сделок ордера), цена не выдумывается: вход остаётся в ожидании до подтверждения, выход закрывает позицию с пометкой „P&L не определён“. В журнале — оценка „≈ X (не подтверждена)“ по рынку на момент закрытия или „—“, если оценки нет; на графике — серый маркер с „≈“. Такие сделки не входят в дневной лимит убытка и в налоговую базу; в налоговом отчёте они вынесены в раздел „Сделки без подтверждённой цены — уточните в брокерском отчёте“. Нереализованный P&L и объём частично исполненной позиции считаются по исполненному объёму везде (карточка, позиции, Circuit Breaker, отчёт, Telegram).»
- **ТЗ §2.4 (outbox):** «`trade.closed` и `position.mismatch` (закрытие, paper SL/TP, отмена pending, сверка после переоткрытия, вход без цены, остаток входа) — строка `pending_events` той же транзакцией, что запись. После снятия локов `close_trade` и отключения адаптера (`publish_staged_after`): шина (WS), доставка своих строк фоновой задачей (`deliver_staged` → `NotificationService.deliver_staged_events`); при отмене — обе фоном. Дедупликация — по `processed_at`. Нет получателя — строка не пишется. Повтор при старте — `OUTBOX_REPLAY_EVENTS` (+`trade.closed`, `position.mismatch`).»
- **ТЗ §5.4:** «Цена исполнения ордера — `order_fill_price`: `average_position_price` → `executed_order_price/(лоты×лот)` → средневзвешенная `stages[].price`; ноль — не цена. Последний источник — `response.price`: цена или сумма/(лоты×лот), берётся кандидат в пределах ±20 % от оценки (рынок, иначе цена входа/сигнала, gotcha-33). Запись закрытия sandbox/real — один commit под `run_shielded_from_cancel` (`ORDER_CANCEL_GRACE_SEC`); свой адаптер — после записи.»
- **ТЗ §4:** «422 — `{detail:[{type,loc,msg,ctx?}]}`, без `input`; неконечные числа — строкой. `TradeResponse.exit_broker_order_id` (новое поле).»
- **Гайд §3.3:** «Lifespan: после `configure_outbox_factory` подключается доставщик outbox (`configure_outbox_deliverer`). Уведомления о закрытиях пишутся в `pending_events` вместе с закрытием и доставляются фоновой задачей сразу после commit — без ожидания слушателя сессии, в том числе при „Стопе“ и shutdown. Не доставленное к остановке досылает повтор при старте (окно 1 с — 1 ч). Shutdown после рантайма ждёт фоновые задачи outbox (до 6 с), затем внешнюю доставку Telegram/Email (до 15 с). `stop_grace_period` backend — 90 с (10 + 45 + 6 + 15 = 76): не уменьшайте.» Сюда же: в §3.3 и строках 90/677 гайда заменить «60 с» на «90 с».
