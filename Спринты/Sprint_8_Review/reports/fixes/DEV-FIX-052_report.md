## DEV-FIX-052 отчёт — S8R fixes, FIX
Статус: ✅ готово к коммиту (worktree `wt-s8r-fixes-c` @ 48f6773, ничего не закоммичено)

### 1. Что реализовано
- Одна проверка `BrokerService._assert_account_not_in_use` под тем же `keyed_lock("session_start_account")`, что и A4. Перечень общий: живые сессии плюс сессии в любом другом статусе, у которых есть открытые позиции.
- Открытая позиция — сделка в статусе `filled`/`pending`. Новая константа `OPEN_POSITION_TRADE_STATUSES` в `trading/models.py` описывает то, что закрывает «Стоп» (`close_all_positions`) и что UI показывает как открытые позиции (`open_positions_count`, WS-снапшот).
- Один SQL-запрос: outer join + `HAVING`, только скаляры (gotcha-58).
- `BrokerAccountInUseError(ValidationError)` → 409 `{detail, error_code:"broker_account_in_use"}`.
- Текст: «Нельзя удалить счёт «X»: его используют работающие торговые сессии — остановите их; у сессий остались открытые позиции — закройте их на странице «Торговля»… Сессии: №7 (SBER, остановлена, открытых позиций: 1)».
- Деактивация счёта не тронута.

### 2. Файлы
Изменены: `backend/app/broker/service.py`, `backend/app/common/exceptions.py`, `backend/app/trading/models.py`. Новый: `backend/tests/unit/test_broker/test_delete_account_open_positions.py` (14 тестов).

### 3. Тесты
RED: `Failed: DID NOT RAISE <class 'app.common.exceptions.ValidationError'>` (stopped + filled/pending); HTTP `assert 204 == 409`; живая сессия `assert 422 == 409`.
GREEN: 14/14, соседние наборы delete/race/guard/strategy — 529 passed.
Мутации:
- без проверки stopped (`open_positions > 0` убран из HAVING) → `Failed: удаление счёта прошло, ожидался отказ` (5 failed);
- без лока → `assert (False or not [<TradingSession …>])` — активная сессия на удалённом счёте (2 failed).

Обе мутации откачены через бэкап, md5 сверен.
Гейты: pytest 5645 passed / 0 xfailed / 0 failed / 1 skipped; ruff 0; mypy Success (205); bandit M0/H0; typecheck 0; lint 0; build ok. vitest: фронт не менялся — на уровне пакета.

### 4. Integration points
✅ `app/broker/service.py:1000` (`_delete_account_locked` → проверка); ✅ обработчик `app/common/exceptions.py:429`; ✅ роутер `DELETE /broker-accounts/{id}` проверен HTTP-тестом.

### 5. Контракты
DELETE счёта: для живых сессий ответ меняется с **422 на 409** + `error_code`. Предпосылка «409 как раньше» неверна: A4 отдавал 422. Фронт (`BrokerAccountList` → `getApiErrorMessage`) читает `detail` при любом статусе, поэтому новый перечень читается без доработки. Миграции нет.

### 6. Проблемы / ФТ-ТЗ / находки
- ФТ §7.7, новый пункт: «Счёт нельзя удалить, пока его используют работающие сессии или у остановленных сессий на нём остались открытые позиции; отказ перечисляет сессии (номер, тикер, статус, число открытых позиций), сессии и позиции автоматически не закрываются (S8R-FIX-052)».
- ТЗ табл. стр. 955 и блок `DELETE /api/v1/broker-accounts/{id}` (стр. 1279): «Response: 204; 409 `broker_account_in_use` — живые сессии или сессии с открытыми позициями (`filled`/`pending`)».
- Находка 1: список `("filled","pending")` продублирован литералами примерно в 15 местах (engine/service/risk_monitor/ws_sessions). Предлагаю заменить их на константу отдельной карточкой. Местами есть и условие `closed_at IS NULL`, местами нет.
- Находка 2: `TradingService.delete_session` удаляет `stopped`-сессию вместе с открытыми сделками — та же потеря учёта.
- Находка 3: фронт обрезает текст ошибки до 400 символов: при 5+ сессиях перечень обрежется.
- Самопроверка: 1 — в БД не пишем, сбой commit невозможен до отказа; 2 — уведомлений нет; 3 — тесты идут через `delete_account` с локом и через HTTP; 4 — таймаутов нет; 5 — один вызывающий (роутер), `SESSION_STATUS_LABELS` +`stopped` на стратегии не влияет; 6 — неприменимо.

### 7. Применённые Stack Gotchas
48 (файловая БД + NullPool в гонке), 58 (скаляры), 59 (рандеву до лока), 71 (см. п. 8), 78.

### 8. Новые Stack Gotchas
Подтверждение gotcha-71, новый вариант. `pytest.raises(...) as exc` на пути `delete_account` создаёт цикл «кадр теста → исключение → traceback → кадр с локом». Из-за него лок `session_start_account` того же id переживает тест, и следующий гоночный тест падает: `Lock … bound to a different event loop`. Правило: отказ ловить в хелпере (`except … as e: return e.detail`) либо делать `gc.collect()` перед гонкой. Дополнить gotcha-71.

### 9. Плагины
py_compile (вместо pyright, в worktree LSP не резолвит), pnpm typecheck (`tsc -b`), context7 не понадобился (стандартный SQLAlchemy `outerjoin/having`), TDD — `mattpocock-skills:tdd`.

---

## Ревью р.1 (оркестратор, 8 пунктов) — статус: ✅ готово к коммиту

### Что сделано по пунктам
1. **Обход через удаление сессии закрыт.** `TradingService._delete_session_locked`: сессия sandbox/real с открытыми позициями не удаляется. Ответ — 409 `session_has_open_positions` с текстом «Нельзя удалить сессию №N: у сессии остались открытые позиции — сначала закройте их (K) кнопкой «Закрыть позиции» на странице сессии». Проверка под `session_lifecycle`. Определение одно: `open_broker_positions_count()` в `trading/models.py`, коррелированный подзапрос. Им же пользуется проверка счёта.
2. **Выход из тупика.** Бэкенд закрытие у `stopped` уже разрешал: `close-all` и закрытие позиции статус сессии не проверяют. Повторный «Стоп» тоже закрывает остаток, но отвечает 422 «уже остановлена», поэтому для кнопки он не подходит. Что добавлено на фронте:
   - на странице сессии кнопка **«Закрыть позиции»** при `isStopped && positions.length > 0`: вызывает `close-all`, затем перечитывает позиции, потому что закрытие best-effort;
   - в `PositionsTable` кнопка «Закрыть» теперь есть у любой сессии (проп `sessionStopped` удалён);
   - `deleteSession` возвращает `boolean`. При отказе страница остаётся открытой, причина видна в Alert `session-action-error`. Store берёт текст из `detail` (`getApiErrorMessage`), а не сырое сообщение axios; так же у `closePosition`/`closeAllPositions`;
   - текст 409 у счёта и у сессии ссылается на кнопку «Закрыть позиции».
3. **Расхождение (`position_mismatch_at`) — пути урегулирования нет, не блокирую.** Пометку снимает только `_clear_position_mismatch` из `_reconcile_broker_positions`. Её вызывают фоновая сверка (только `active` с listener'ом, `runtime.py:1660`) и restore (`active`/`suspended`). У `stopped` пометку не снимает ничто, и действия в UI нет. Запрет запер бы счёт навсегда. У живой сессии счёт и так держит её статус. **Оркестратору — карточка:** «пометка расхождения у остановленной сессии неснимаема (нужно действие "Сверить/Снять пометку" или сверка `stopped` с позициями)». Тест на выбранную ветку: `stopped` + пометка без позиций → счёт удаляется.
4. **Paper исключён:** `TradingSession.mode.in_(BROKER_MODES)` внутри подзапроса. У paper число открытых позиций у брокера = 0. Удаление paper-сессии — как раньше. Живая paper-сессия со ссылкой на счёт по-прежнему держит счёт (A4).
5. **Сообщение:** «Нельзя удалить счёт: №1 (GAZP, работает, открытых позиций: 0), … и ещё 4. Работающие торговые сессии остановите. У сессий остались открытые позиции — закройте их кнопкой «Закрыть позиции» на странице сессии.» В тексте не больше 5 сессий (`ACCOUNT_IN_USE_LISTED_SESSIONS`). Полный перечень — поле `sessions: [{id, ticker, status, open_positions}]`. Тест: 9 сессий → 5 в тексте, «и ещё 4», длина ≤ 400. Оговорка: при трёх-четырёхзначных номерах и статусе «приостановлена рестартом сервера» хвост может превысить 400 символов и обрежется фронтом.
6. `BrokerAccountListDeleteBlocked.test.tsx` переведён на реальный контракт: 409, `error_code`, `sessions`, новый текст.
7. Константа `OPEN_POSITION_TRADE_STATUSES` теперь используется в:
   - engine: `close_all_positions`, `_stop_remainder`, `_count_open_trades`, `close_position` (проверка статуса);
   - service: `open_positions_count`, карточка сессии, `get_positions`, `get_all_positions`, дашборд;
   - `ws_sessions._POSITION_STATUSES`;
   - risk_monitor — 5 мест;
   - circuit_breaker `_check_duplicate_instrument`.

   **Не тронуты:**
   - места с другим условием (`closed_at IS NULL`): `engine.py:1193` (открытый капитал), `:3263` (выход по сигналу), `:4343` (переоткрытие sandbox);
   - `runtime.py:4511` — публикация `trade.filled`, семантика другая.
8. Добавлен `ConflictError(detail, error_code, extra)` и один обработчик 409 `{**extra, detail, error_code}`. От него наследуют `StrategyStatusConflictError`, `BrokerAccountInUseError`, `SessionHasOpenPositionsError`. Формат ответа стратегии не изменился.

### Файлы
- Изменены: `backend/app/{broker/service.py, common/exceptions.py, trading/models.py, trading/service.py, trading/engine.py, trading/risk_monitor.py, trading/ws_sessions.py, circuit_breaker/engine.py}`, `backend/tests/test_broker/test_audit_s8r_delete_account_race.py` (assert под новый текст), `frontend/src/{stores/tradingStore.ts, components/trading/SessionDashboard.tsx, components/trading/PositionsTable.tsx, components/settings/__tests__/BrokerAccountListDeleteBlocked.test.tsx}`.
- Новые: `backend/tests/test_trading/test_delete_session_open_positions.py` (5), `frontend/src/components/trading/__tests__/{SessionDashboard.closeRemainder.test.tsx (5), tradingStore.deleteSession.test.ts (3)}`. В `test_delete_account_open_positions.py` +4 теста (итого 20).

### Тесты р.1
**RED:**
- удаление сессии: `assert 204 == 409`;
- ссылка на действие в тексте: `assert 'остались открытые позиции — закройте их кнопкой «Закрыть позиции» на странице сессии' in 'Нельзя удалить счёт «Песочница»: …'`;
- поле `sessions`: `KeyError: 'sessions'`;
- paper: `DID NOT RAISE`;
- фронт: `Unable to find an accessible element with the role "button" and name "Закрыть позиции"`, `Unable to find … [data-testid="close-position-btn"]`, `expected undefined to be false`, `expected 'Request failed with status code 502' to be 'Брокер недоступен'`.

**GREEN:** бэкенд 25/25 (карточка + гонка), фронт — затронутые файлы, 7 файлов / 45 тестов.

**Мутации** (бэкап + md5, все откачены):
- проверка в `delete_session` выключена → `assert 204 == 409`;
- фильтр `BROKER_MODES` снят → `assert 409 == 204`;
- потолок перечня = 100 → `assert 9 == 5`;
- кнопка «Закрыть позиции» скрыта (`{false && …}`) → `Unable to find … "Закрыть позиции"`.

**Гейты:** pytest 5654 passed / 0 xfailed / 0 failed / 1 skipped (лог с `-o faulthandler_timeout=240`); ruff 0; mypy Success (205); bandit M0/H0; typecheck 0; lint 0; build ok; vitest — только затронутые файлы, 45 passed.

### Проблемы / находки р.1
- `test_audit_s8r_delete_account_race.py` падает, если перед ним в том же процессе идёт `test_delete_account_guard.py`: `Lock … bound to a different event loop`, gotcha-71. **Воспроизводится и на HEAD** (проверено подменой файлов через бэкап). В порядке полного прогона не проявляется. В своём файле я защитился `gc.collect()` и хелпером `_refusal`. Оркестратору: поправить guard-тесты так же или дополнить gotcha-71.
- Флейк `test_restore_all_timeout.py::test_slow_accounts_not_starved_by_silent_one` под нагрузкой (параллельный исполнитель); отдельно — 11/11.
- Карточка сессии в списке (`SessionCard`) кнопки закрытия остатка не получила: `open_positions_count` есть только в `SessionDetailResponse`. Действие доступно на странице сессии, а отказ удаления с карточки показывается текстом backend в Alert страницы «Торговля».
- `closeAllPositions` в store после успеха обнуляет `positions` целиком, даже если часть не закрылась (best-effort). На странице сессии я перечитываю позиции, у кнопки «Закрыть все» живой сессии поведение прежнее — кандидат в карточку.

### Правки ФТ/ТЗ (вместо текста раздела 6 исходного отчёта)
- **ФТ §7.7**, новый пункт: «**Удаление счёта (S8R-FIX-052).** Счёт нельзя удалить, пока его используют работающие сессии (работает / пауза / приостановлена) или у сессий песочницы/боевого счёта на нём остались открытые позиции. Отказ перечисляет до пяти сессий (номер, тикер, статус, число открытых позиций) и подсказывает действие; сессии и позиции автоматически не останавливаются и не закрываются. Позиции paper-сессий удалению не мешают.»
- **ФТ, раздел торговых сессий** (рядом с правилом удаления сессии): «Остановленную сессию песочницы/боевого счёта с открытыми позициями удалить нельзя — сначала закройте позиции кнопкой «Закрыть позиции» на странице сессии или «Закрыть» в таблице позиций; закрытие остатка доступно и у остановленной сессии (S8R-FIX-052).»
- **ТЗ**, таблица эндпоинтов (стр. 955) и блок REST (стр. 1279): `DELETE /api/v1/broker-accounts/{id}` → `204`; `409 {detail, error_code: "broker_account_in_use", sessions: [{id, ticker, status, open_positions}]}`. `DELETE /api/v1/trading/sessions/{id}` → `204`; `409 {detail, error_code: "session_has_open_positions"}` для sandbox/real с открытыми позициями (`filled`/`pending`).
- **ТЗ, раздел ошибок API:** общий формат 409 — `{detail, error_code, …доп. поля}` (`ConflictError`).
- Отдельный UI-чеклист: «Страница остановленной сессии с позициями: кнопка «Закрыть позиции», «Закрыть» в таблице; отказ удаления — Alert с причиной».

---

## Ревью р.2 (оркестратор, 7 пунктов) — статус: ✅ готово к коммиту

### Что сделано
1. **(блокирующее) Закрытие у остановленной sandbox/real-сессии не шлёт ордер вслепую.** В `OrderManager._close_position_locked` перед встречным ордером при `trade_session.status == "stopped"` вызывается `_broker_holds_position`:
   - портфель читает общий со сверкой после переоткрытия песочницы читатель `_read_live_positions`; из него теперь же собирается `_live_position_figis`;
   - FIGI сделки — `resolve_trade_figi`;
   - учтённое считается по **счёту** и FIGI, в штуках со знаком. Формула вынесена из сверки runtime в `RiskMonitor.broker_position_units`, runtime переведён на неё;
   - у песочницы портфель «без бумаг» (gotcha-56) перечитывается до `POSITION_CHECK_RETRIES` раз.

   Исходы:
   - **позиции нет или у брокера меньше учтённого** → ордер не уходит, сделка закрывается путём «фантома» сверки. Этот путь вынесен из `_reconcile_locked` в `_close_trade_without_order`: `closed` с P&L 0, как в существующей сверке; без цены входа — «P&L не определён»; `pending` без цены — `cancelled`. `trade.closed` уходит через outbox с причиной `closed_outside_terminal`;
   - **позиция есть** → обычное закрытие;
   - **портфель не прочитан, FIGI или адаптер недоступны** → ордер не уходит, 409 `broker_position_unverified`: «Не удалось проверить позицию у брокера — повторите позже. Встречный ордер не отправлен.».

   Живые сессии не затронуты. Первый «Стоп» закрывает позиции, пока сессия ещё `active`, поэтому проверка его не касается; повторный «Стоп» и ручное закрытие у `stopped` её проходят.
2. **Итог close-all:** `POST /close-all` → `{closed_count, results:[{trade_id, result: closed|closed_without_order|not_closed, reason}]}` (`CloseOutcome` → `CloseAllResponse`). Фронт показывает Alert `close-result`: «Закрыто: X из Y», строка по каждой сделке с причиной. Позиции перечитываются, store убирает только закрытые.
3. **Ошибки — один источник:** локальный `rerunError` удалён, все ошибки действий показываются из `error` store в одном Alert `session-action-error`. Все действия store используют `getApiErrorMessage`. Мутирующие действия (пауза, возобновление, стоп, удаление, закрытие) сбрасывают `error` перед запросом; после успеха он остаётся `null`.
4. **paper в перечне 409:** `open_positions_count(broker_only=…)` — одно определение. Держат счёт позиции у брокера (`broker_only=True`), а число в перечне и `sessions[].open_positions` — реальное по всем открытым (`broker_only=False`).
5. **Скриншоты:** vite preview из своего worktree на порту 4317, весь API перехвачен через `page.route`, backend не поднимался. Процесс остановлен, порт свободен.
   - `/Users/sergopipo/Documents/Claude_Code/Test/s8r-evidence/fixes/screens/FIX-052_stopped_session_close_remainder.png` — остановленная сессия, 2 позиции, кнопки «Закрыть позиции» и «Закрыть»;
   - `/Users/sergopipo/Documents/Claude_Code/Test/s8r-evidence/fixes/screens/FIX-052_close_remainder_result.png` — Alert итога: №31 закрыта вне терминала (ордер не отправлен), №32 не закрыта — вне торговых часов.
6. **Остальные 409 переведены на `ConflictError`:** `TInvestRequiredError`, `ParityGateError`, `StrategyCodeStaleError`. Проверка по grep: из 10 блоков `except ValidationError` в `app/` ни один не охватывает места, где они бросаются:
   - `TInvestRequiredError` — `MarketDataService.get_candles`; вызовы `get_candles` под `except ValidationError` не стоят;
   - `ParityGateError` — `TradingService.start_session`, сразу из роутера;
   - `StrategyCodeStaleError` — `_check_code_not_stale` в обработчиках бэктеста без такого `except`.

   Блоки `except Exception` ловили их и раньше. Отдельные обработчики удалены, тело ответа прежнее; это фиксирует новый `tests/unit/test_common/test_conflict_error_format.py` — 9 тестов, включая «обычный `ValidationError` = 422». `test_parity_gate.py` теперь выбирает обработчик по MRO, как Starlette, а не по точному классу.
7. Псевдоним `_POSITION_STATUSES` удалён, в `ws_sessions` напрямую `OPEN_POSITION_TRADE_STATUSES`. Тестов, патчащих псевдоним, не было.

### Файлы р.2
- Изменены:
  - бэкенд: `app/trading/{engine.py, service.py, router.py, schemas.py, models.py, risk_monitor.py, runtime.py, ws_sessions.py}`, `app/common/exceptions.py`, `app/broker/service.py`;
  - тесты: `tests/test_trading/{test_parity_gate.py, test_delete_session_open_positions.py}`;
  - фронт: `src/{api/types.ts, api/tradingApi.ts, stores/tradingStore.ts, components/trading/SessionDashboard.tsx}`, тесты `SessionDashboard.rerun.test.tsx`, `SessionDashboard.closeRemainder.test.tsx`, `tradingStore.deleteSession.test.ts`.
- Новые: `tests/test_trading/test_close_stopped_session_broker_check.py` (10), `tests/unit/test_common/test_conflict_error_format.py` (9).

### Тесты р.2
**RED:**
- бэкенд: `AssertionError: Expected place_order to not have been awaited. Awaited 1 times.`, `assert 200 == 409` (сбой чтения), `KeyError: 'results'`, `assert 0 == 2` (перечитывание песочницы);
- фронт: `Unable to find … [data-testid="close-result"]`, `expected 'Request failed with status code 422' to be 'Пауза невозможна…'`, `expected 'старая ошибка' to be null`, `expected undefined to deeply equal {closed_count…}`;
- п.4 (RED через мутацию, код уже был написан): `assert '№1 (SBER, работает, открытых позиций: 1)' in '… открытых позиций: 0) …'`.

**GREEN:** бэкенд — новые 10 + 9, затронутые наборы; фронт — 9 файлов / 56 тестов.

**Мутации** (бэкап + md5, все откачены):
- проверка выключена (`and False`) → `results … 'closed'` вместо `closed_without_order`;
- сбой чтения превращён в пустой портфель → `assert 200 == 409`;
- без перечитывания песочницы → `assert 1 == 2`, `assert 1 == 3`;
- учтённое только по своей сессии → `Expected place_order to not have been awaited. Awaited 1 times.`;
- число в перечне по `broker_only=True` → `открытых позиций: 0`;
- фронт: без сброса `error` в `stopSession` → `expected 'старая ошибка' to be null`; без `setCloseResult` → `Unable to find … close-result`.

**Гейты:** pytest 5673 passed / 0 xfailed / 0 failed / 1 skipped (`-o faulthandler_timeout=240`, лог в файл); ruff 0; mypy Success (205); bandit M0/H0; typecheck 0; lint 0; build ok; vitest затронутых — 56 passed.

### Решения и оговорки р.2 (для оркестратора)
- **«Меньше объёма → без ордера»** сделано по решению. Цена: при частичном закрытии вне терминала (у брокера 1 лот из 3) остаток у брокера выпадает из учёта, пометки расхождения нет. Альтернатива — отказ с текстом «сверьте вручную», если нужна, — отдельной карточкой.
- **P&L фантома — 0, как в существующей сверке** (решение разрешало выбор). Честнее был бы «P&L не определён», но предикат `is_pnl_undetermined` для сделки с ценой входа требует оценку выхода или номер ордера — это правка общей логики, вне карточки. Пометка «закрыта вне терминала» — в причине `trade.closed` (`closed_outside_terminal`) и в итоге close-all. Отдельного поля пометки у сделки нет (нужна была бы миграция). В Telegram-шаблоне `trade.closed` причина не выводится.
- **Песочница «без бумаг»:** если все `POSITION_CHECK_RETRIES` чтений подряд (паузы по `POSITION_CHECK_RETRY_DELAY_SEC`) вернули портфель без бумаг, позиция считается отсутствующей. Иначе остаток, закрытый вне терминала на счёте без других бумаг, нельзя было бы закрыть никогда. Риск — только песочница. Для real одно чтение.
- `_read_live_positions` и пауза перечитывания выполняются под локом `close_trade` (как у сверки): до 3 × 20 с + 2 × 5 с на сделку. SL/TP этой сделки на это время ждут — у остановленной сессии их нет.

### Правки ФТ/ТЗ — дополнение к р.1
- **ФТ, торговые сессии (закрытие остатка):** «Закрытие позиций у остановленной сессии песочницы или боевого счёта сначала сверяется с брокером. Если позиции у брокера нет (её закрыли в приложении брокера) или там её меньше, чем учтено, встречная заявка не отправляется: сделка закрывается с отметкой «закрыта вне терминала — по данным брокера позиции нет» и P&L 0. Если связаться с брокером не удалось, заявка не отправляется и показывается «Не удалось проверить позицию у брокера — повторите позже». После «Закрыть позиции» показывается итог по каждой позиции: закрыта, закрыта вне терминала или не закрыта с причиной (вне торговых часов, заявка на закрытие уже у брокера, позиция не проверена). Работающих сессий это не касается (S8R-FIX-052).»
- **ТЗ, REST:** `POST /api/v1/trading/sessions/{id}/close-all` → `200 {closed_count, results: [{trade_id, result: "closed"|"closed_without_order"|"not_closed", reason}]}`. `POST …/positions/{trade_id}/close` у остановленной sandbox/real-сессии: `409 {error_code: "broker_position_unverified"}`, если портфель брокера не прочитан.
- **ТЗ, движок (закрытие позиции):** у `stopped`-сессии sandbox/real перед встречным ордером — чтение портфеля (`_read_live_positions`). Сравнение по счёту и FIGI в штуках (`RiskMonitor.broker_position_units`). Нет или меньше → `_close_trade_without_order` + `trade.closed` (`closed_outside_terminal`) через outbox.
- **ТЗ, ошибки API:** все 409 (`tinvest_required`, `parity_*`, `code_stale`, `strategy_*`, `broker_account_in_use`, `session_has_open_positions`, `broker_position_unverified`) — общий формат `ConflictError` `{detail, error_code, …}`.
- **UI-чеклист:** «Остановленная сессия: «Закрыть позиции» → Alert итога по каждой позиции; ошибки действий — один Alert без дублей.»

---

## Ревью р.3 (оркестратор, 1 правка) — статус: ✅ готово к коммиту

### Что сделано
1. **Позиция у брокера меньше учтённой, но не ноль — отказ.** `OrderManager._broker_holds_position`:
   - у брокера не меньше учтённого той же стороны → обычное закрытие ордером;
   - у брокера 0 → закрытие без ордера, как в р.2;
   - меньше, но не ноль, или другая сторона → `BrokerPositionPartialError` (новый наследник `ConflictError`, 409 `broker_position_partial`): ордер не отправляется, сделка не меняется. Текст: «У брокера N шт. из M учтённых — позицию частично закрыли вне терминала. Закройте остаток в приложении брокера и повторите». Числа без хвостовых нулей и экспоненты (`_fmt_units`).
   - В итоге «Закрыть все» та же причина: `not_closed` с этим текстом. Фронт выводит её без изменений (общий вывод итога из р.2).
2. «P&L 0» у закрытых вне терминала оставлен — семантика сверки REV-LQ, по решению оркестратора.

### Файлы р.3
`backend/app/common/exceptions.py` (`BrokerPositionPartialError`), `backend/app/trading/engine.py` (ветка отказа, `_fmt_units`), тесты `backend/tests/test_trading/test_close_stopped_session_broker_check.py`, `backend/tests/unit/test_common/test_conflict_error_format.py`. Фронт в р.3 не менялся.

### Тесты р.3
- **RED:** `assert 200 == 409` (у брокера 1 из 3 — сделка закрылась без ордера); `Failed: закрытие прошло, ожидался отказ` (соседняя живая сессия).
- **GREEN:**
  - у брокера 1 из 3 → `place_order` не вызван, сделка `filled` и P&L не тронут, HTTP 409 с точным текстом, в итоге close-all та же причина;
  - у брокера 0 → закрыта без ордера (тест р.2);
  - тест формата 409 дополнен (`broker_position_partial`), 10/10;
  - затронутые наборы — 15/15.
- **Мутация** (бэкап + md5, откачено): ветку отказа вернул к прежнему «закрыть без ордера» → `assert 200 == 409`, `Failed: закрытие прошло, ожидался отказ`.
- **Гейты:** pytest 5674 passed / 0 xfailed / 0 failed / 1 skipped (повторный прогон; в первом — 1 failed: тайминг-флейк `test_stream_figi_from_ensure_figi.py::test_silent_source_one_network_wait_per_session_start` (таймаут 0.2 с под нагрузкой параллельного исполнителя), модуль не затронут, отдельно — 3/3); ruff 0; mypy Success (205); bandit M0/H0; typecheck 0; lint 0; build ok; vitest — фронт в р.3 не менялся (затронутые в р.2 — 56 passed).

### Оговорка
Сравнение идёт по счёту. Если на том же счёте и инструменте работает живая сессия, у брокера лежат её лоты. Тогда остаток остановленной сессии получит тот же отказ «У брокера 2 шт. из 3 учтённых». Закрыть его можно после закрытия или «Стопа» соседа, либо руками у брокера. Тест `test_broker_lots_of_live_neighbour_do_not_cover_stopped_trade` переведён на отказ.

### Правки ФТ/ТЗ — уточнение к р.2
- **ФТ (закрытие остатка остановленной сессии):** вместо «если позиции у брокера нет … или там её меньше, чем учтено, заявка не отправляется: сделка закрывается…» → «Если позиции у брокера нет (её закрыли в приложении брокера), встречная заявка не отправляется: сделка закрывается с отметкой «закрыта вне терминала — по данным брокера позиции нет» и P&L 0. Если у брокера позиции меньше, чем учтено (часть закрыли вне терминала), заявка не отправляется и сделка не меняется: показывается «У брокера N шт. из M учтённых — позицию частично закрыли вне терминала. Закройте остаток в приложении брокера и повторите».»
- **ТЗ, REST:** `POST …/positions/{trade_id}/close` и итог `close-all` у остановленной sandbox/real-сессии: `409 broker_position_partial` — у брокера меньше учтённого, но не ноль; сделка не меняется.

---

## Ревью р.4 (оркестратор, 11 пунктов) — статус: ✅ готово к коммиту

Принцип р.4: при любой неопределённости позиции у брокера ордер не отправляется и сделка не закрывается (409). Выход из тупика — явное «Позиция закрыта у брокера».

### Что сделано
1. **Песочница «без бумаг» во всех `POSITION_CHECK_RETRIES` чтениях** → `BrokerPositionUnverifiedError` (409), как в сверке S8R-AUDIT-011, а не «позиции нет» (`_read_portfolio_securities` возвращает `None`).
2. **Проверка — для всех статусов, которые рантайм не ведёт:** `UNMANAGED_SESSION_STATUSES = ("stopped", "suspended")`. Первый «Стоп» `active`-сессии не затронут; «Стоп» `suspended`-сессии проверку проходит.
3. **Выход в полёте по той же бумаге счёта** (`exit_broker_order_id`/`exit_order_placed_at` у любой открытой сделки, свою отсекает `OrderInFlightError` раньше) → 409 unverified. Так же — ордер входа без исхода (`pending`) по бумаге.
4. **Учтённое:**
   - считается по `RiskMonitor.position_units` = `position_lots` × лот (откат на `volume_lots`, как у самого закрытия). Сверка рантайма оставлена на `broker_position_units`;
   - учтённое 0 при бумаге у брокера → 409 unverified;
   - разнонаправленные сделки счёта по бумаге → 409 unverified (до чтения портфеля, без сети).
5. **Сделки без FIGI с тем же тикером** (`ticker_eq`) учитываются в учтённом.
6. **«Позиция закрыта у брокера»:** `POST …/positions/{trade_id}/close` с телом `{"confirm_closed_outside": true}` (поле `ClosePositionRequest`, в стиле существующего тела).
   - Только для `stopped`/`suspended` sandbox/real и `filled`. Иначе 422: «…только у остановленной или приостановленной сессии…» / «ордер входа ещё не исполнен».
   - Без ордера и без чтения портфеля, путём «закрыта вне терминала»; `trade.closed` уходит через outbox с причиной `closed_outside_terminal_by_user`. В журнале действий — `confirm_closed_outside`.
   - Тексты 409 unverified/partial ссылаются на действие. На фронте: по отказу `broker_position_unverified`/`_partial` (одиночное закрытие или итог close-all, `CloseOutcome.error_code`) — Alert `mark-closed-offer` с кнопкой «№N: Позиция закрыта у брокера» → Modal: «Ордер брокеру не отправляется. Подтвердите, что позиция уже закрыта в приложении брокера…» → `markPositionClosedOutside`.
7. **«Закрыть позицию и пауза»** (`PauseConfirmModal`): отказ close-all целиком (`null`) или итог `not_closed` → пауза не ставится, модалка показывает причину (`pause-chain-error`). Пауза вызывается только после полного закрытия, поэтому сброс `error` в `pauseSession` причину не стирает: она в локальном состоянии модалки.
8. **Тексты partial:** числа по модулю (`_fmt_units(abs)`); бумага у брокера в другую сторону — отдельный текст: «По этой бумаге у брокера позиция в другую сторону — проверьте вручную…».
9. **«Закрыть все»:** `BrokerPortfolioSnapshot` на вызов — портфель счёта читается один раз, решение по `(счёт, FIGI)` принимается один раз до первого ордера вызова и хранится данными, не исключением (gotcha-71). Это закрывает и риск устаревшего снимка: второй сделке не нужно заново сравнивать портфель с уже изменённым учётом.
10. **Смена `sessionId` на странице** сбрасывает итог закрытия, предложения, диалог и `error` store. Эффект зависит только от `sessionId`, `clearError` — через ref: с зависимостью от ссылки рендер зацикливался (OOM в тесте).
11. **Ответ одиночного закрытия:** `TradeResponse.closed_without_order`; фронт показывает уведомление «Позиция №N закрыта без ордера…» (и после «Позиция закрыта у брокера»).

### Файлы р.4
- Бэкенд:
  - `app/trading/engine.py`: `BrokerPortfolioSnapshot`, `UNMANAGED_SESSION_STATUSES`, `_broker_holds_position` / `_broker_position_verdict` / `_read_portfolio_securities`, ветка `confirm_closed_outside`, `CloseOutcome.error_code`;
  - `app/trading/risk_monitor.py` (`position_units`), `app/trading/schemas.py`, `app/trading/router.py`, `app/trading/service.py`;
  - тест `tests/test_trading/test_close_stopped_session_broker_check.py` (+12, итого 22).
- Фронт:
  - `src/utils/apiError.ts` (`getApiErrorCode`), `src/api/types.ts`, `src/api/tradingApi.ts`, `src/stores/tradingStore.ts` (`ClosePositionResult`, `markPositionClosedOutside`), `src/components/trading/SessionDashboard.tsx`, `src/components/trading/PauseConfirmModal.tsx`;
  - тесты `SessionDashboard.closeRemainder.test.tsx` (+5), `tradingStore.deleteSession.test.ts` (+3), новый `PauseConfirmModal.closeChain.test.tsx` (3).

### Тесты р.4
**RED** (фактический вывод):
- бэкенд: `Failed: закрытие прошло, ожидался отказ` (песочница без бумаг, разнонаправленные сделки, сделки без FIGI, short partial); `Expected get_real_positions to have been awaited` (suspended); `Expected place_order to have been awaited once. Awaited 0 times` (откат на `volume_lots`); `assert 2 == 1` (один портфель на close-all); `KeyError: 'closed_without_order'`; `assert 200 == 422` (подтверждение у живой); `KeyError: 'error_code'`;
- фронт: `Unable to find … mark-closed-31`, `… pause-chain-error`; `expected undefined to deeply equal { ok: true, … }`; `markPositionClosedOutside is not a function`; `expected "vi.fn()" to be called at least once` (уведомление, clearError).

**GREEN:**
- бэкенд: 22/22 в файле карточки; затронутые наборы (close-all C6, stop race, best-effort, удаление сессии, формат 409) — 58/58;
- фронт: 11 файлов / 113 тестов (трейдинг, store, удаление счёта, `apiError`).

**Мутации** (бэкап + md5, все откачены):
- M1 песочница без бумаг = «позиции нет» → `Failed: закрытие прошло`;
- M2 только `stopped` → `Expected get_real_positions to have been awaited`;
- M3 без проверки выхода в полёте → тест краснеет (`'в полёте' in …`);
- M4 `broker_position_units` вместо `position_units` → `BrokerPositionUnverifiedError … учтённая позиция нулевая`;
- M5 без сделок без FIGI → `Failed: закрытие прошло`;
- M6 без общего снимка → `assert 2 == 1`;
- M7 подтверждение и у живой → `assert 200 == 422`;
- F1 пауза при незакрытой → `Unable to find … pause-chain-error`;
- F2 без предложения → `Unable to find … mark-closed-31`;
- F3 без сброса при смене сессии → `expected "vi.fn()" to be called at least once`.

**Гейты:** pytest 5686 passed / 0 xfailed / 0 failed / 1 skipped (`-o faulthandler_timeout=240`, лог в файл; см. «Тест тайминга» ниже); ruff 0; mypy Success (205); bandit M0/H0; typecheck 0; lint 0; build ok; vitest затронутых — 113 passed.

**Скриншоты** (vite preview из своего worktree на 4317, перехват API через `page.route`, процесс остановлен):
- `/Users/sergopipo/Documents/Claude_Code/Test/s8r-evidence/fixes/screens/FIX-052_unverified_mark_closed_offer.png` — 409 unverified в Alert и предложение «№31: Позиция закрыта у брокера»;
- `/Users/sergopipo/Documents/Claude_Code/Test/s8r-evidence/fixes/screens/FIX-052_mark_closed_confirm_dialog.png` — диалог подтверждения.

### Решения вне списка (для оркестратора)
- **Ордер входа без исхода (`pending`)** по той же бумаге счёта тоже считается неопределённостью (409 unverified) — по принципу р.4; в решении назван только выход в полёте.
- **У остановленной сессии `pending` с ценой входа**, не прошедший урегулирование остатка, — тоже 409 (раньше уходил ордер).
- **Порядок проверок:** сначала учёт (выход/вход в полёте, разнонаправленность), без сети; портфель читается, только если учёт однозначен.
- Причина неопределённости дописывается в текст 409 («Причина: …»), `error_code` один — `broker_position_unverified`.

### Правки ФТ/ТЗ — дополнение к р.2–р.3
- **ФТ (закрытие остатка сессии, которую терминал не ведёт — остановленной или приостановленной, песочница/боевой):** «Если позицию не удаётся достоверно проверить у брокера — брокер не ответил, песочница несколько раз подряд вернула портфель без бумаг, по бумаге ещё исполняется заявка, по бумаге открыты сделки в разные стороны, учтённая позиция нулевая, а у брокера бумага есть, — заявка не отправляется и сделка не меняется. Показывается «Не удалось достоверно проверить позицию у брокера — повторите позже…» с причиной. Если позиция уже закрыта в приложении брокера, пользователь может отметить её закрытой кнопкой «Позиция закрыта у брокера» — после подтверждения («Ордер брокеру не отправляется. Подтвердите, что позиция уже закрыта в приложении брокера») сделка закрывается без заявки, приходит уведомление. Кнопка предлагается после такого отказа и после отказа «позицию частично закрыли вне терминала». «Закрыть позицию и пауза» не ставит паузу, если позиция не закрылась, и показывает причину.»
- **ТЗ, REST:** `POST /api/v1/trading/sessions/{id}/positions/{trade_id}/close`, тело `{reason?, confirm_closed_outside?: bool}` → `200 TradeResponse` (+`closed_without_order: bool`). С `confirm_closed_outside=true` закрывает без ордера только у `stopped`/`suspended` sandbox/real и `filled`, иначе 422. `409 broker_position_unverified | broker_position_partial`. `close-all`: `results[].error_code`.
- **ТЗ, движок:** проверка портфеля — для `UNMANAGED_SESSION_STATUSES` (`stopped`, `suspended`). Учтённое — по счёту: сделки с FIGI плюс без FIGI того же тикера, `position_units`. Неопределённость → 409. close-all — один портфель и одно решение на вызов (`BrokerPortfolioSnapshot`). Причины `trade.closed`: `closed_outside_terminal`, `closed_outside_terminal_by_user`.
- **UI-чеклист:** «Отказ «не удалось достоверно проверить позицию» → кнопка «Позиция закрыта у брокера» → диалог подтверждения → уведомление; «Закрыть позицию и пауза» при незакрытой позиции — причина в модалке, пауза не ставится; переход на другую сессию сбрасывает итог и ошибку.»

### Тест тайминга restore (правка вне кода карточки)
После р.4 `tests/test_trading/test_restore_all_timeout.py::TestReconcileTimeoutOnRestore::test_reconcile_timeout_does_not_block_restore` падал в каждом полном прогоне: 3 из 3, «restore шёл 1.18 с» при допуске 0,8 с. Изолированно он укладывается в 0,51 с.

Разбор:
- код р.4 на путь restore не попадает: runtime закрытия не вызывает;
- прогон всех каталогов до `test_trading` включительно, без сбора `tests/unit`, проходит;
- `pytest tests/ --deselect tests/unit` (unit собирается, но не запускается) — падает, 1,18 с;
- тот же прогон с `gc.disable()` — restore-тест проходит. Падает другой, `test_order_path_unknown_outcome…`: он полагается на сборку циклов (gotcha-71), это артефакт эксперимента.

Вывод: на большой куче полного прогона полная сборка поколения 2 приходится на окно замера и добавляет около 0,7 с. Новые тесты карточки идут в `test_trading` раньше по алфавиту и сдвинули этот момент внутрь замера.

Правка — только тест: в `_restore` добавлен `gc.collect()` до `time.monotonic()`, с комментарием.
- `--deselect tests/unit`: 2340 passed;
- полный прогон: 5686 passed / 0 failed.

Кандидат в gotcha: «тест-таймер с допуском < 1 с в большом наборе — сборка мусора до замера».


---

## Ревью р.5 (оркестратор, 8 пунктов) — статус: ✅ готово к коммиту

### Что сделано
1. **(блокирующее) «Сессию не ведёт рантайм» определяется по факту — по живому слушателю.**
   - Добавлены `SessionRuntime.has_listener(session_id)` и `sandbox_reopen.runtime_has_listener(session_id)`; второй берёт рантайм из реестра, который lifespan уже заполняет через `set_reconcile_runtime`.
   - Решение — `engine.runtime_does_not_manage(session)`: sandbox/real, статус не `active` и нет живого слушателя.
     - `paused` после паузы — слушатель снят → проверка портфеля, доступно «Позиция закрыта у брокера».
     - `stopped` → то же.
     - `suspended` до `start()` → то же; `suspended` после `start()` в окне restore → обычное закрытие без чтения портфеля.
   - Исключение `active` — **моё решение**: «Стоп» снимает слушатель **до** закрытия позиций (S8R-AUDIT-029), и первое закрытие при «Стопе» должно остаться обычным, с ордером. `active` без слушателя вне «Стопа» — нарушение инварианта S8R-AUDIT-008.
   - Рантайм не зарегистрирован (тесты без lifespan, CLI) → слушателя нет: решает тот же «не `active`».
   - Константа `UNMANAGED_SESSION_STATUSES` удалена. Текст 422 подтверждения: «…только у сессии песочницы или боевого счёта, которую терминал сейчас не ведёт: остановленной, на паузе или приостановленной».
2. **`confirm_closed_outside` при неурегулированном остатке ордера входа:** сначала тот же `_settle_entry_remainder_locked`, что у обычного закрытия, затем закрытие без ордера. Остаток не снят → 409 `entry_remainder_unsettled` (`ConflictError`): «Остаток частично исполненного ордера входа ещё не снят у брокера (сделка №N) — отметить позицию закрытой можно после его отмены. Повторите позже.». Ветка подтверждения перенесена внутрь `try`: адаптер освобождает общий `finally`.
3. **Своя `pending`:**
   - у сессии, которую рантайм не ведёт, — существующим путём отмены ордера входа с проверкой исхода (`_cancel_pending_trade`), в том числе `pending` с ценой. Не 409, поэтому и пары 409→422 больше нет;
   - «вход без исхода» как причина unverified — только у **других** сделок бумаги;
   - «Позиция закрыта у брокера» — только для `filled`, это уже было в р.4.
4. **Учтённое в штуках:** `RiskMonitor.position_units(trade, lot_size)`; лот — `t.lot_size or _safe_lot_size(тикер, …, сессия, trade=t)`, тем же источником, что количество встречного ордера, без отката на 1 из `derive_lot_size(trade, 1)`. Пример: старая строка без `lot_size` и лот 10 → «У брокера 3 шт. из 30 учтённых», а не ордер на 3 лота.
5. **«Закрыть все» — две фазы:** сначала `pending`-сделки вызова (их отмена), затем остальные; общий дедлайн делится между фазами. Решение «вход без исхода» (`_PENDING_ENTRY_CAUSE`) в снимке не кэшируется.
6. **Фронт:** после «Закрыть все» предложение «Позиция закрыта у брокера» убирается у сделок с итогом `closed`/`closed_without_order`.
7. **Telegram `/close`** (команда и кнопка `close_pos:`): при `closed_without_order` — «позиция закрыта без ордера: у брокера её уже нет», иначе прежнее «ордер на закрытие отправлен» (`_close_result_text`).
8. **`apiError.ts`:** `getApiErrorCode` перенесён после `getApiErrorMessage` со своим JSDoc; JSDoc `getApiErrorMessage` снова стоит над ней.

### Файлы р.5
- Бэкенд: `app/trading/engine.py`, `app/trading/runtime.py` (`has_listener`), `app/trading/sandbox_reopen.py` (`runtime_has_listener`), `app/trading/risk_monitor.py` (`position_units(trade, lot_size)`), `app/notification/telegram_webhook.py`.
- Тесты бэкенда: `tests/test_trading/test_close_stopped_session_broker_check.py` (+8, итого 30), новый `tests/test_notification/test_telegram_close_without_order.py` (3).
- Фронт: `src/components/trading/SessionDashboard.tsx`, `src/utils/apiError.ts`, тест `SessionDashboard.closeRemainder.test.tsx` (+1).

### Тесты р.5
**RED** (фактический вывод):
- бэкенд:
  - paused → `Expected place_order to not have been awaited. Awaited 1 times`;
  - suspended со слушателем → `BrokerPositionUnverifiedError … портфель брокера не прочитан`;
  - остаток → `assert [] == ['settle']` и `assert 200 == 409`;
  - своя pending с ценой → `BrokerPositionUnverifiedError … ордер входа без исхода`;
  - лот → `Failed: закрытие прошло, ожидался отказ`;
  - close-all filled+pending (отмена pending медленнее) → `assert 'not_closed' == 'closed'`;
  - Telegram → `'позиция закрыта без ордера…' in '✅ Позиция #1 (SBER) — ордер на закрытие отправлен'`;
- фронт: `expect(element).not.toBeInTheDocument()`.

**GREEN:** файл карточки 30/30, Telegram 3/3, `test_notification` 223/223, затронутые наборы движка 56/56; фронт — 11 файлов / 114 тестов.

**Мутации** (бэкап + md5, все откачены):
- B1 по статусу вместо слушателя → 2 failed;
- B2 подтверждение без урегулирования остатка (`and not confirm_closed_outside`) → `assert [] == ['settle']`, `assert 200 == 409`;
- B3 своя pending с ценой мимо pending-пути → `BrokerPositionUnverifiedError`;
- B4 лот с откатом на 1 → `Failed: закрытие прошло`;
- B5 без фазы pending → `'not_closed' == 'closed'`;
- B6 Telegram всегда «ордер отправлен» → тесты красные;
- F1 без снятия предложения → `not.toBeInTheDocument`.

**Гейты:** pytest 5697 passed / 0 xfailed / 0 failed / 1 skipped (`-o faulthandler_timeout=240`, лог в файл; см. «Затронутые чужие тесты» ниже); ruff 0; mypy Success (205); bandit M0/H0; typecheck 0; lint 0; build ok; vitest затронутых — 114 passed.

### Правки ФТ/ТЗ — уточнение р.4
- **ФТ:** вместо «остановленной или приостановленной сессии» → «сессии, которую терминал сейчас не ведёт: остановленной, поставленной на паузу или приостановленной до восстановления». Добавить: «Отметить позицию закрытой у брокера при неснятом остатке частично исполненной заявки на вход нельзя — сначала снимается остаток; если не удалось — «повторите позже». Telegram `/close` при закрытии без заявки отвечает «позиция закрыта без ордера: у брокера её уже нет».»
- **ТЗ, движок:** признак «рантайм не ведёт» — `runtime_does_not_manage`: sandbox/real, статус ≠ `active`, нет слушателя (`SessionRuntime.has_listener`). Своя `pending` → `_cancel_pending_trade`. Учтённое — `position_units(trade, trade.lot_size or _safe_lot_size)`. close-all — фаза pending, затем остальные. `409 entry_remainder_unsettled` у `confirm_closed_outside`.

### Затронутые чужие тесты (р.5)
Первый полный прогон после р.5 дал 3 failed:
- `test_fix_tr2_review2.py::test_manual_close_delivered_immediately_exactly_once[paused_no_listener|listener]`;
- `test_accounting_edges.py::TestRecoveryPriceFallback::test_no_price_anywhere_flags_mismatch_and_can_close`.

Все три закрывают позицию sandbox-сессии на **паузе** через адаптер-мок без портфеля. По решению р.5 п.1 такую сессию рантайм не ведёт, и перед ордером читается портфель, а мок его не отдавал → 409 unverified.

Правка — только моки адаптеров (`_close_adapter`, `_patch_adapter`): добавлен `get_real_positions` с бумагой сделки (с запасом, 1000 шт.). Смысл тестов — закрытие встречным ордером — сохранён. Повторный полный прогон: 5697 passed / 0 failed.

**Для оркестратора:** автопауза (CB, расхождение — `pause_session_if_active`) меняет статус в БД. Снимает ли она слушатель в рантайме, не проверял. Если нет — такая сессия остаётся «ведомой», и закрытие идёт ордером без чтения портфеля; признак по факту это корректно отражает.


---

## Ревью р.6 (оркестратор, 1 правка) — статус: ✅ готово к коммиту

### Что сделано
Правило «сессию ведёт рантайм» уточнено по проверке оркестратора: автопауза (CB, расхождение) слушатель не снимает, а слушатель у сессии не в `active` позиции не ведёт. `engine.runtime_does_not_manage(session)` для sandbox/real:
- `paused` (ручная или автопауза) и `stopped` — рантайм не ведёт, **со слушателем или без**: перед ордером читается портфель, доступно «Позиция закрыта у брокера»;
- `suspended` — ведёт, только если слушатель уже поднят (`runtime_has_listener`);
- `active` — ведёт всегда. Это исключение из р.5 сохранено: «Стоп» снимает слушатель до закрытия позиций (S8R-AUDIT-029), и это закрытие штатное, ордером. Без исключения правило «слушатель И статус в (`active`, `suspended`)» перевело бы первое закрытие при «Стопе» на проверку портфеля.

Paper — по-прежнему «ведёт» (позиций у брокера нет).

### Файлы р.6
`backend/app/trading/engine.py` (`runtime_does_not_manage`), тест `backend/tests/test_trading/test_close_stopped_session_broker_check.py` (+2, итого 32).

### Тесты р.6
- **RED:** автопауза со слушателем → `Expected get_real_positions to have been awaited.`; «Позиция закрыта у брокера» у такой сессии → `assert 422 == 200`.
- **GREEN:** 2/2. Файл карточки 32/32; вместе с наборами, задетыми в р.5 (`test_fix_tr2_review2`, `test_accounting_edges`), и форматом 409 — 63/63.
- **Мутация** (бэкап + md5, откачено): `paused` «ведётся», если есть слушатель (правило р.5) → оба теста красные.
- **Гейты:** pytest 5699 passed / 0 xfailed / 0 failed / 1 skipped (`-o faulthandler_timeout=240`, лог в файл); ruff 0; mypy Success (205); bandit M0/H0; typecheck 0; lint 0; build ok. Фронт в р.6 не менялся.

### Правки ФТ/ТЗ — уточнение р.5
- **ФТ:** «сессия, которую терминал сейчас не ведёт» = остановленная, на паузе (в том числе поставленная на паузу автоматически — Circuit Breaker, расхождение с брокером) или приостановленная до восстановления.
- **ТЗ, движок:** `runtime_does_not_manage` — sandbox/real и (`paused`/`stopped`, либо `suspended` без слушателя `SessionRuntime.has_listener`); `active` — ведётся (закрытие при «Стопе» — ордером).

---

## Сведение с S8R-FIX-063 ч.2 (HEAD `fd74766`) — проверка вставки оркестратора

**Вставка** в `_close_outside_terminal_locked`: после `_close_trade_without_order`, если сделка `closed` с `pnl is None`, вызывается `unpriced_trade_alert`, затем `stage_unpriced_trade_alert` в той же транзакции. Проверил по коду, правок не требуется:
- **порядок** — `trade.closed`, затем `cb.unpriced_trade`, затем один `commit`, затем оба события в `_staged_events`; так же устроены обычное закрытие (`_stage_close_events`) и сверка (`_reconcile_locked`);
- **commit** — строки outbox и закрытие в одной транзакции. Если commit упал, в `_staged_events` ничего не попадает, а строки откатываются вместе со сделкой;
- **отмена** — до commit: всё откатывается, сделка остаётся `filled`. После commit: публикует `publish_staged_after` держателя лока, недоставленное поднимает повтор outbox при старте;
- **сеть** — `unpriced_trade_alert` → `estimate_trade_for_cb` читает только кэш баров БД (`closed_bar_close_before`, без ISS), поэтому под локом `close_trade` сетевых вызовов нет;
- **проверка `pnl is None`** перекрывается `is_pnl_undetermined` внутри `unpriced_trade_alert`: она безвредна и избавляет от чтения баров у сделки с ценой входа (там P&L 0).

**Тесты** (новый класс `TestUnpricedCloseOutsideTerminal` в `backend/tests/test_trading/test_close_stopped_session_broker_check.py`, итого 35 в файле):
- остановленная sandbox-сессия, сделка `filled` без цены входа, у брокера позиции нет → ордера нет, сделка `closed` с P&L `None`, в outbox ровно `cb.unpriced_trade` + `trade.closed`;
- то же, но есть закрытый бар T-Invest до момента закрытия (оценка выхода) → в outbox только `trade.closed`;
- «Позиция закрыта у брокера» (`confirm_closed_outside`) по такой сделке → 200, `pnl_undetermined: true`, в outbox `cb.unpriced_trade` + `trade.closed`.

**Мутация** (бэкап + md5, откачено): вставку убрал (`if False:`) → оба теста «без оценки» красные: `assert ['trade.closed'] == ['cb.unpriced...trade.closed']`. Тест «с оценкой» остаётся зелёным, как и положено.

**Гейты:** pytest 5719 passed / 0 failed / 1 skipped (`-o faulthandler_timeout=240`, лог в файл); ruff 0; mypy Success (206); bandit M0/H0. Фронт не менялся.
