## DEV-FIX-TB отчёт — S8R fixes, FIX
Статус: ✅ готово к коммиту (worktree `wt-s8r-fixes-b` @ 1aaa5ed, не закоммичено)

### 1. Что реализовано
- **043:** `_tax_category` — share/bond/etf либо тип ПФИ как есть (futures/currency/option/…); лоты ПФИ — раздел «…ПФИ — рассчитайте с бухгалтером» под ИТОГО (csv/xlsx), с итогом «справочно»; в базу, прибыль/убыток/комиссии не входят; флаги include_* их не отсекают.
- **043:** `MarketDataService.backfill_instrument_types` — `GetInstrumentBy(FIGI)` токеном владельца для `unknown`/пустых/отсутствующих типов (FIGI справочника, иначе сделки), запись в `instruments.type` вне дедлайна. Константы `TAX_TYPE_BACKFILL_MAX_REQUESTS=20`, `TAX_TYPE_BACKFILL_TIMEOUT_SEC=15.0`; при сбое — `unknown` + пометка. Эвристика даёт только категории ЦБ, в пометке указана категория («X (акция)»).
- **059:** `CandleSeries` — столбцы, словарь бара собирается при обращении; поле `SkipValidation`. Потребитель один — `_run_parity_check` (итерация, `len`, `[0]`/`[-1]`, `bool`), наружу не уходит. Год минуток (205 380 баров): `run` 0,396 → 0,086 с; сборка серии 0,315 → 0,038 с; побайтовое равенство с типами — True.
- **061:** тесты ждали живой gRPC `find_instrument` (канон тикера без строки справочника) до `FIGI_LOOKUP_TIMEOUT_SEC`=10 с. Засеян справочник; autouse-сторож класса запрещает `find_instrument`. Время: 10,32/10,08 → <0,02 с. Код `app/` не менялся.

### 2. Файлы
Новые: `app/backtest/candle_series.py`, `tests/unit/test_tax/test_fix043_pfi_and_type_backfill.py`, `tests/unit/test_backtest/test_fix059_candle_series.py`. Изменённые: `app/tax/service.py`, `app/market_data/service.py`, `app/backtest/{engine,schemas}.py`, `tests/test_routers/test_market_data_router.py`, `tests/unit/test_tax/test_export_escaping.py` (строка лота с типом «@share» ищется по тикеру: она теперь в разделе ПФИ).

### 3. Тесты
RED: 043 `assert Decimal('1500.00') == Decimal('100.00')`; 059 `assert 2000 == 0`; 061 `AssertionError: живой find_instrument в тесте: [call('SBER', broker_account_id=1, purpose='canonical_ticker')]`. GREEN — все.
Мутации: `_tax_category` → `return None` ⇒ `1500.00 == 100.00`; возврат цикла `itertuples` ⇒ `2000 == 0`; убрать засев ⇒ тот же сторож. Откат через бэкап, md5 сверены.
Гейты: pytest **5512 passed / 1 skipped / 0 xfailed / 0 failed**; ruff 0; mypy Success (203); bandit M0/H0; typecheck 0; lint 0; build ok; vitest: фронт не менялся — проверка на уровне пакета.

### 4. Integration points
✅ `app/tax/service.py:502` → `backfill_instrument_types`; ✅ `app/backtest/engine.py:440` → `CandleSeries.from_frame`; `is_pfi_type` — `tax/service.py:592,711,786,913`.

### 5. Контракты
API-схемы не менялись. Миграции нет (по решению). Смысл `total_profit/total_loss/total_commission` отчёта — теперь только ЦБ.

### 6. Проблемы / предложения
- ФТ §18.2, последнее предложение пункта «Единая база» заменить на: «Фьючерсы, опционы, валюта и прочие инструменты вне акций/облигаций/ETF — отдельный раздел отчёта вне базы ЦБ с пометкой „ПФИ — рассчитайте с бухгалтером“ (S8R-FIX-043). Неизвестный тип дозапрашивается у брокера; если не удалось — категория ЦБ по тикеру с пометкой».
- ТЗ §5.13 «База»: добавить «лоты `is_pfi_type` исключены из базы и итогов; `backfill_instrument_types` (≤20 запросов, 15 с, запись `instruments.type`)».
- Решено самостоятельно: `sp`/`clearing_certificate` и прочие типы — тоже в раздел вне базы (консервативно).
- **Новая находка:** сторож сети в conftest ловит только HTTP; gRPC T-Invest из тестов уходит в живую сеть.
- Самопроверка: 1 — сбой записи типа подавляется, отчёт строится на полученном типе; 2 — уведомлений нет; 3 — тест идёт через `generate_report`, путь без `user_id` покрыт старым тестом; 4 — под `asyncio.timeout` только сеть, `scope.expired()`; 5 — `_resolve_instrument_types` вызывается в одном месте; 6 — не применимо.

### 7. Gotchas: 30, 35, 48, 50, 74, 75
### 8. Новые: кандидат — «conftest-сторож сети не покрывает gRPC: тест ждёт таймаут живого вызова» (симптом: тест ровно N с).
### 9. Плагины: py_compile + mypy; typecheck `tsc -b`; context7 не нужен (сигнатура `get_instrument_by` сверена по установленному SDK); tdd — скилл загружен.

---

## Ревью р.1 — DEV-FIX-TB
Статус: ✅ готово к коммиту (все 11 пунктов; не закоммичено)

### Что сделано по пунктам
1. ПФИ = `PFI_TYPES` {futures, option, currency}. `sp`/`clearing_certificate`/прочие известные типы — в базе (`by_type["other"]`), пометка «Прочие ценные бумаги… SPNOTE (sp)». Не определён — эвристика + пометка, в базе.
2. Дозапрос — только тикеры лотов отчёта, по убыванию Σ|realized_pnl|. Негативный кэш процесса по FIGI: ответ источника (NOT_FOUND, пустой тип, иной отказ по запросу) — 24 ч. **Отклонение:** сбой связи (UNAVAILABLE/DEADLINE/INTERNAL/обрыв канала) — 15 мин (`FIGI_NEGATIVE_TTL`); причины — инвариант «сбой связи не кэшируется» и gotcha-56. Промахи бюджет не занимают.
3. `_prefetch_instrument_types` — до `persist_with_retry`, один раз; типы передаются в каждую попытку (`prefetched_types`), внутри единицы сети нет.
4. Итога ПФИ нет. Раздел построчно, заголовок результата: «Результат, в единицах цены инструмента — рассчитайте с бухгалтером».
5. Общая функция `MarketDataService.ensure_instrument_types`. Вызывается из `ensure_figi_lookup` (FIGI в справочнике, тип `unknown`/пусто, бюджет 1, `FIGI_LOOKUP_TIMEOUT_SEC`; тип известен — сети нет) и из налогового отчёта. В `directory_instrument` не встраивал: это чистое чтение БД, его зовут десятки мест.
6. `series_dicts` (список — без копии; `CandleSeries` — одна сборка: интерпретатор обращается к бару многократно) + `series_timestamps` (столбец) в `_run_parity_check`.
7. `_report_rows` — один источник строк для xlsx и csv: лоты ЦБ → ИТОГО → база/прибыль/убыток/комиссии → ПФИ → пометки.
8. `pfi_result`/`pfi_commission`, `_lots_result`, `PFI_TOTAL_LABEL` удалены.
9. `UNTYPED_INSTRUMENT_TYPES` — публичная константа `ticker_resolver`, копии нет.
10. Ответ брокера пишет `type` и `figi`; FIGI — только если тикер ответа совпал с нашим.
11. conftest `_block_live_grpc` (autouse): `grpc[.aio].secure/insecure_channel` → `OutboundNetworkBlocked` + `pytest.fail` на teardown (приложение глотает сбой). Маркер `live_grpc` зарегистрирован. Красных тестов после включения — 0 (полный прогон), сторож включён по умолчанию.

### Тесты
RED: п.1 `Decimal('100.00') == Decimal('300.00')`; п.2 `'FIGIAAA00' == 'FIGIZFUT'`; п.3 `AttributeError … ensure_instrument_types`; п.5/10 `[] == ['BBGOLDBOND']`, `None == 'FIGINEWFUT'`; п.6 `assert 2 == 1`; п.7 различие строк csv↔xlsx; п.11 `Тест пытался открыть настоящий gRPC-канал: ['invest-public-api.tinkoff.ru']`.
Мутации (бэкап+md5, откачены): ПФИ+`sp` → `100.00 == 300.00`; сортировка по алфавиту → `'FIGIAAA00' == 'FIGIZFUT'`; кэш промахов выключен → 3 запроса вместо 1; прекэш внутрь единицы → `assert 2 == 1`; хук `ensure_figi_lookup` снят → `[] == ['BBGOLDBOND']`; без figi → `None == 'FIGINEWFUT'`; csv без итогов → несовпадение строк; итог ПФИ → `assert not True`; ts по словарям → `assert 2 == 1`.
`test_export.py::test_csv_has_correct_content` обновлён под итоговые строки (п.7).
Гейты: pytest **5518 passed / 1 skipped / 0 failed**; ruff 0; mypy Success (203); bandit M0/H0; typecheck 0; lint 0; build ok; vitest: фронт не менялся — проверка на уровне пакета.

### Предлагаемые правки ФТ/ТЗ (вместо раунда 0)
- ФТ §18.2: «Фьючерсы, опционы, валюта (ПФИ) — отдельный раздел отчёта вне базы ЦБ, построчно, результат в единицах цены инструмента, без итога, с пометкой „ПФИ — рассчитайте с бухгалтером“. Прочие ценные бумаги (структурные ноты и т.п.) — в базе с пометкой типа. Неизвестный тип дозапрашивается у брокера; если не удалось — категория ЦБ по тикеру с пометкой. CSV и XLSX содержат одинаковые строки, включая итоги».
- ТЗ §5.13: `_prefetch_instrument_types` до `persist_with_retry`; `ensure_instrument_types` (≤20 запросов на отчёт, 15 с, порядок по |результату|, кэш промахов 24 ч / сбой связи 15 мин, запись type+figi), он же вызывается из `ensure_figi_lookup` для строки без типа; `_report_rows` — общий источник строк файлов.

### Самопроверка
Сбой записи справочника подавляется, отчёт строится на полученном типе. Под `asyncio.timeout` — только сеть. Хук в `ensure_figi_lookup` в `try` — FIGI от сбоя типа не зависит. В горячем пути сеть только при типе `unknown`, не чаще раза в 15 мин / 24 ч на FIGI. Кэш промахов ограничен 512 записями.

---

## Ревью р.2 — DEV-FIX-TB
Статус: ✅ готово к коммиту (все 9 пунктов; не закоммичено)

### Что сделано по пунктам
1. Из `ensure_figi_lookup` вызов `ensure_instrument_types` убран: горячий путь сделки тип не дозапрашивает. Дозапрос вызывает только налоговый отчёт. В фоновый скан облигаций `corporate_actions._bond_tickers_with_open_positions` **не встроен**: в планировщике нет пользователя, а токен по S8R-AUDIT-003 нужен по счёту каждой сессии (`TradingSession.broker_account_id`). Пришлось бы группировать по счетам и выносить сеть из единицы работы детекта (S8R-AUDIT-097) — нетривиально, предлагаю отдельной карточкой.
2. FIGI, запрос по которому оборван дедлайном или отменой (`in_flight`, `finally`), отмечается промахом «сбой связи» на `INSTRUMENT_TYPE_TRANSIENT_TTL` (15 мин).
3. Классификация — через `grpc_code_name` + `grpc_code_category`, свой набор кодов удалён. TRANSIENT/LOST и ошибка без кода gRPC → 15 мин; NOT_FOUND, пустой тип, чужой тикер → 24 ч; AUTH, RATE_LIMITED и прочие → не кэшируются.
4. CSV: `None` → пустая ячейка, колонки совпадают с xlsx (база ИТОГО — в 14-й колонке). Помощник `_csv_data_rows` в `test_audit_s8r_lots.py` и проверка в `test_export.py` теперь отличают лот по дате покупки, а не по числу колонок.
5. Ответ с чужим тикером не используется: тип и FIGI не пишутся, промах 24 ч. В отчёте тикер помечен «определён по тикеру».
6. Вес агрегируется по `ticker.upper()`; в `ensure_instrument_types` один FIGI — один запрос, тип получает только тикер, совпавший с ответом.
7. `_prepare_inputs` вне единицы работы, один раз на генерацию: сделки, `_resolve_lot_size_fallbacks`, типы (вес — с лотностью). Первая попытка единицы берёт готовые сделки (`_ReportInputs.take_trades`); **повтор после «locked» перечитывает их** — rollback экспайрит LiveTrade (gotcha-37), лениво дочитать в async нельзя. Лоты строятся после резолва типов: от типа зависят категория, НКД и флаги include. Предварительный проход для веса — только CPU, без чтения БД. Сеть из единицы ушла полностью, включая лотность.
8. `app/common/ttl_cache.py::TtlCache` (TTL на запись или `default_ttl()` в момент проверки, предел размера, вытеснение старейшей) — для `_figi_negative_cache`, `_type_miss_cache` и `_FALLBACK_FIGI_CACHE`. Внешние функции и поведение прежние; существующие тесты кэшей зелёные, добавлен `tests/unit/test_common/test_ttl_cache.py`.
9. `ensure_instrument_types` → `timed_block("market_data.ensure_instrument_types")`.

### Тесты
Код правился до тестов, поэтому RED снят откатом соответствующего фрагмента фикса (бэкап+md5, откачено):
- п.1 `assert ['BBGOLDBOND'] == []`
- п.2 `AssertionError: в пределах 15 мин сеть не трогается`
- п.3 `assert {} == {'OLDBOND': 'bond'}`
- п.4 строки csv ≠ xlsx поколоночно
- п.5 `{'OLDBOND': 'futures'} == {'OLDBOND': 'unknown'}`
- п.6 `['FIGIOTHER'] == ['FUTSI']` (агрегация); `['FUTSISIZ6',…] == ['FUTSI']` (дедупликация)
- п.7 `['FIGISMALL'] == ['BBGBIG']`

Новые тесты: `test_figi_lookup_does_not_fetch_type`, `test_hung_request_is_remembered_as_transient`, `test_auth_error_is_not_remembered`, `test_not_found_is_remembered_for_a_day`, `test_answer_for_other_ticker_is_not_used`, `test_same_figi_in_two_spellings_is_one_request`, `test_weight_is_aggregated_across_spellings`, `test_pairs_with_one_figi_are_one_request`, `test_order_weighs_lot_size`, поколоночный `test_csv_has_same_rows_as_xlsx`, `test_ttl_cache.py`.
Гейты: pytest — см. итог ниже; ruff 0; mypy Success (204); bandit M0/H0; typecheck 0; lint 0; build ok; vitest: фронт не менялся — проверка на уровне пакета.

### Финальный текст для ФТ §18.2 (заменяет предложения раундов 0–1)
«Единая налоговая база по операциям с обращающимися ценными бумагами (ст. 214.1 НК РФ): результаты акций, облигаций, ETF/БПИФ и прочих ценных бумаг (структурные ноты и т.п., с пометкой типа) сальдируются в пределах года. Фьючерсы, опционы и валюта (ПФИ) — отдельный раздел отчёта вне базы и итогов, построчно; результат указан в единицах цены инструмента, итоговой суммы нет; пометка „ПФИ — рассчитайте с бухгалтером“ (S8R-FIX-043). Тип инструмента — из справочника брокера. Если в справочнике его нет, отчёт дозапрашивает тип у T-Invest (не более 20 запросов и 15 с на отчёт, по убыванию влияния на результат). Если тип не получен — категория ценной бумаги по тикеру с пометкой, в базе. CSV и XLSX содержат одинаковые строки и колонки, включая итоги».

### Финальный текст для ТЗ §5.13 (добавить к абзацу «База»)
«S8R-FIX-043: `_prepare_inputs` до `persist_with_retry`, один раз на генерацию: сделки года, лотность (`_resolve_lot_size_fallbacks`) и дозапрос типов (`_prefetch_instrument_types` → `MarketDataService.ensure_instrument_types`). Единица работы сети не делает; первая попытка берёт готовые сделки, повтор перечитывает их (gotcha-37). Дозапрос: тикеры лотов отчёта с типом `unknown`/пустым, вес — Σ|realized_pnl| по `ticker.upper()` с лотностью; один FIGI — один `GetInstrumentBy`; бюджет `TAX_TYPE_BACKFILL_MAX_REQUESTS=20`, общий дедлайн `TAX_TYPE_BACKFILL_TIMEOUT_SEC=15` (только сеть); промахи — `TtlCache` по FIGI (`grpc_codes`: TRANSIENT/LOST и запрос, оборванный дедлайном, — 15 мин; NOT_FOUND, пустой тип, чужой тикер — 24 ч; AUTH/RATE_LIMITED — не кэшируются); ответ пишется в `instruments` (type+figi) после дедлайна отдельной сессией; метрика `market_data.ensure_instrument_types`. Горячий путь `ensure_figi_lookup` тип не дозапрашивает. ПФИ = `PFI_TYPES` {futures, option, currency} — вне базы; прочие типы — `by_type["other"]` в базе. Строки файла — общий `_report_rows` для xlsx/csv.»

### Самопроверка
- Сеть только под `asyncio.timeout`; запись в справочник — после, отдельной сессией, сбой подавляется.
- Отмена отмечает FIGI в полёте и пробрасывается дальше.
- Подготовка отчёта не бросает (`None` → единица всё сделает сама).
- Кэши ограничены 512 записями; горячий путь сделки без новых сетевых вызовов.

### Итог полного pytest р.2
- Прогон до добавления последних тестов: **5523 passed / 1 skipped / 0 failed**.
- Финальный прогон: **5526 passed / 1 failed / 1 skipped**, 13,5 мин против обычных 6 — машина была нагружена параллельным исполнителем.
- Упал `test_commission_settle.py::TestRaceAndLock::test_extension_between_read_and_lock_skips_pass` (`assert not task.done()`). Отдельно файл зелёный (22 passed), класс — 3/3 зелёный. Путь досверки комиссий пачкой не затронут.
- **Вывод:** это флейк зависимого от времени теста под нагрузкой, не регрессия.
- **Новая находка:** тест ждёт рандеву по реальному времени (200×10 мс и 20 холостых шагов цикла событий). Под нагрузкой досверщик успевает завершиться раньше — кандидат в карточку, класс S8R-FIX-005.
