---

# ТЕХНИЧЕСКОЕ ЗАДАНИЕ (ТЗ)
## Торговый терминал для рынка ценных бумаг РФ (MOEX)

**Версия:** 4.11
**Дата:** 2026-11-18
**Основание:** ФТ v4.19 от 2026-11-18
**Статус:** ✅ M4 Production-ready. Gate Sprint_8_Review пройден (PASS WITH NOTES), остаток приёмки (8 замечаний) и весь backlog доведения закрыты. Актуализировано по итогам код-ревью P0–P1, финальной приёмки и closeout-цикла S8R. С 2026-10-23 идёт развитие по плану 008; v4.0 — блок A-01 «Хранилище баров» (сборка сдана 2026-10-24, в рабочем терминале ещё нет; дефект 4 тестового стенда — ряды «как было» с пересчётом по дроблениям при чтении, `ohlcv_lot_history`, head `a858647f24e6`; дефекты 5–6 — день архива в основном ряду только при точном совпадении с итогом дня ISS `history` TQBR; дефект 7 — итог дня ISS по календарным дням без торгов выходного дня, `get_daily_volumes`). v4.1 — блок A-03 «Часы торгов, стопы внутри бара, дилер» (в `develop` с 2026-11-04, PR #112, `dd6d8fa`; в рабочем терминале ещё нет): таблица периодов расписания `trading_schedule_periods`, правила `entry_verdict`/`exit_verdict`, стоп и тейк внутри бара, список «закрыть при первой возможности», поток одной подпиской с каналом `dealer:`, стопы у дилера за `DEALER_STOPS_ENABLED`, уровневые алерты на модели `Alert`; head `d92b7dd5c939`. v4.2 — блок A-05 «Дефекты этапа 1» (в `develop` с 2026-11-04, PR #113, `185b9be`; в рабочем терминале ещё нет): отчёт разбора описания `ParseReport` в `strategy_versions.parse_report` и 422 `description_has_problems`, размер позиции бэктеста в лотах одной формулой (`strategy/sizing.py`, `backtests.lot_size`, пометка «лоты штуками»), фоновый повтор внешней доставки по строкам `notification.dispatch` (`retry_notification_delivery`, `notifications.channels_failed`), пометка «MOEX ISS · задержка 15 мин» (`marketDataStore.lastSource`); T+1 приведено к Model A (ФТ §7.4); head `cdb15f21e5ca`. v4.3 — блок Б-02 «Графики: панели и виды» (в `develop` с 2026-11-05, PR #111, `4c70407`; в рабочем терминале ещё нет): раскладка графиков одним JSON-документом на пользователя (`chart_layouts`, `GET/PUT /api/v1/chart-layout`, ADR 0024), ряды свечей по ключу «источник : тикер : ТФ : adjust» и один `CandleStream` на «источник : тикер : ТФ» (ADR 0025, 0029), рисунки по «пользователь + бумага» (`/api/v1/charts/{ticker}/drawings`, индекс `idx_drawing_user_ticker`, ADR 0026), дилерские свечи второй серией с дробным временем (ADR 0027) по `GET /candles/dealer?adjust` и каналу `dealer:`, виды и шкалы панели, слияние кадра `bar_pending`; head `0acf47e6607f`. v4.4 — блок Б-01 «Новости: сборщик, архив, лента» (в `develop` с 2026-11-06, PR #116, `3c65d27`; в рабочем терминале ещё нет): модуль `app/news/` — шесть таблиц `news_*` (архив только с заголовком и ссылкой, ADR 0101; разметка — запись на разметчика с полями ИИ D8, ADR 0102), сборщик семи источников раз в 60 с (`news_collect`), словарь акций TQBR из ISS + `news_dictionary_terms` (ADR 0104), реакция «до / после» по 1m ISS от точного времени публикации (`news_reactions`, ADR 0103), лента `GET /api/v1/news` с курсором и страница `/news`; новых настроек в `config.py` нет; head `b01e7a5c3d92`. v4.5 — блок A-02 «Старая история и дивиденды» (в `develop` с 2026-11-09, PR #121, мерж-коммит `d4223c9`; в рабочем терминале ещё нет): окна источников `market_data/seam.py` — 1m до шва 13.06.2018 из ISS с 01.12.2011, D 07.03–12.06.2018 из ISS с `LEGALCLOSEPRICE` и меткой 00:00 UTC (ADR 0031, 0032), писатель старой истории `IssHistoryLoader`, фильтр фантомных дневок T-Invest до 2019 года по дням торгов ISS (ADR 0033), синхронно из T-Invest ≤ 365 суток и признак `history_pending` (ADR 0034), фон `history_backfill` по расписанию по вселенной бумаг, счёт для рыночных данных `broker_accounts.is_market_data_source` (ADR 0035), `ticker_aliases`, модуль `app/dividends/` — история `GetDividends`, поправки, сверка со Смартлабом, расхождения при чтении (ADR 0036), страница `/dividends`; невалидная свеча вырезает из покрытия только свой период (ADR 0030), `AGGREGATE_CACHE_MAX_BARS` = 80 000; новых настроек в `config.py` нет; head `79f3c238dbb8`. v4.6 — блок Б-04 «Рисунки» (в `develop` с 2026-11-09, PR #122, мерж-коммит `0a5927d`; в рабочем терминале ещё нет): рисунки бумаги на всех ТФ её панелей — `selectDrawings(state, ticker)`, номер бара точки — только `drawingBarIndex` (бар, в который попадает время, вне баров — первый после; D/W/M — `regularBarIndexForTime`), `logical` для позиции не читается (ADR 0050); «Очистить» — `DELETE /charts/{ticker}/drawings` без `timeframe`, предел 100 на бумагу (ADR 0051); типы `ray`, `extended`, `hray`, `channel` (`p3`), `fib` (`levels`, ADR 0053); линейка — `GET /market-data/trading-hours`, сумма биржевых периодов таблицы расписания (ADR 0052); миграция данных `b6b5064c0be6` удаляет рисунки с точкой года < 2000 и браузер чистит их из localStorage (ADR 0054); новых таблиц и настроек нет; head `b6b5064c0be6`. v4.7 — блок A-04 «„Стоп всему“ и Circuit Breaker» (в `develop` с 2026-11-10, PR #125, мерж-коммит `92eb2ee`; в рабочем терминале ещё нет): журнал-флаг `trading_stops` и `StopAllService` (`app/trading/stop_all.py`) — флаг первым, пауза всех сессий с `pause_reason='stop_all'`, аварийное закрытие `ExitKind.FORCED`, отчёт «N из M», снятие только из веб-сессии (ADR 0040); лок `trading_stop` — самый внешний; `/api/v1/trading/stop-all` GET/POST/DELETE, `/stopall` в Telegram; 409 `trading_stopped` у старта и «Возобновить»; лимиты сессии «как в общих» и свой охват (ADR 0041), `GET /circuit-breaker/config/inherited`, `GET|DELETE /config/session/{id}`; причина паузы `trading_sessions.pause_reason`, связи `circuit_breaker_event_sessions`, история серий `GET /circuit-breaker/history` (ADR 0042, 0045); отчёт закрытия `circuit_breaker_events.close_report`; живые обновления в `system:{user_id}` без нового типа уведомления (ADR 0043, 0044); новых настроек нет; head `5613e921bcc1`. v4.8 — починка времени в результатах бэктеста (в `develop` с 2026-11-10, PR #126, мерж-коммит `7a0e137`; в рабочем терминале ещё нет): `GET /backtest/{id}` — `equity_curve[].time`, `candles[].time`, `trade_markers[].time` — unix-секунды UTC при любом поясе процесса (`app.common.datetime_utils.unix_utc`), выгрузка CSV/PDF — время по МСК; фронтенд — правило «naive из БД = UTC» (`utils/mskTime.ts`: `utcIsoToUnix`, `parseUtcTimestamp`), графики бэктеста со сдвигом `MSK_OFFSET_SEC`, как основной график; схема БД и миграции не менялись; head `5613e921bcc1`. v4.9 — блок Б-07 «ИИ-разметка и алерт новостей»: влит в `develop` 2026-11-12 (PR #128, мерж-коммит `9b2dbc9`), доработка 2026-11-13 (PR #130, `402e7e0`); в рабочем терминале ещё нет (выкатка — тегом после приёмки на тестовом стенде); фоновый вызов ИИ `app/ai/background.py` — провайдер `ai_provider_configs.is_background` первого администратора, `complete_json` с таймаутом и ответом по схеме JSON, учёт `ai_background_usage` по суткам МСК (цена — `usage.cost` OpenRouter, иначе по ценам строки), дневной лимит `ai_background_settings.daily_token_limit` = 2 000 000 (ADR 0061, 0069); шаг ИИ-разметки `news/ai_labeling.py` в проходе `news_collect` — пачки по 10, `PROMPT_VERSION` `b07-v3` с кандидатами словаря, пометки `news_items.ai_note` (ADR 0062); проверка на эталоне `news/ai_benchmark.py` и `news_ai_benchmarks` — ворота включения по связке (ADR 0063); действующая разметка — ИИ, иначе словарь (ADR 0060); алерт `news` на общей модели `Alert` с двумя стадиями и доставкой через outbox `news.alert` (ADR 0064, 0067, 0068); выключатели `news_settings` читаются в начале прохода (ADR 0065); метки `NewsMarkersLayer` — одна на бар, номер бара `drawingBarIndex` на 1m–4h, на D/W/M — дата МСК (ADR 0066); словарь — кавычки спасают имя только от исключения из самого имени (ADR 0105); новых настроек в `config.py` и заданий планировщика нет; ревизии `b07a1e5c0d42` (поверх `5613e921bcc1`) и `b07d34cd8897` (только данные), head `b07d34cd8897`; ИИ-разметка по умолчанию выключена. v4.10 — блок A-06 «Договор исполнения» (в `develop` с 2026-11-17, PR #131, мерж-коммит `2927e39`; в рабочем терминале ещё нет): пакет `app/execution/` — чистые правила договора (`contract.py`: проскальзывание, уровни с шагом цены, касание, гэп; `dividends.py`: последний день покупки T+2/T+1, бар гэпа; `warmup.py`: прогрев max(200, 5 × период)); бэктест и Grid — одна настройка `cerebro_setup.build_cerebro` (`cheat_on_open`, `slip_out`), намерение входа из кодогена исполняет `WrappedStrategy.next_open` (Market + Stop + Limit `oco`), защитный ордер без позиции снимает `AuctionAwareBroker`, тень `parity` по тем же правилам (ADR 0070); paper — ожидающее исполнение `_PendingExecution` на open первой минуты нового бара, минутный наблюдатель paper со стопом и тейком каждой минутой, текущая цена ручных и аварийных закрытий `_paper_current_price` (ADR 0071, 0073); журнал исполнений `execution_journal` (ADR 0072); проскальзывание и налог сессии, умолчание `session_slippage_default`, проверка спреда `GetOrderBook` (ADR 0073); дивиденды — деньги на баре гэпа, `read_dividends` для бэктеста, paper и учёта, `dividend_accruals`, MCFTRR (ADR 0074); прогрев в бэктесте, Grid и live (ADR 0075); годовой Sharpe `metrics.annual_sharpe` по барам теста и денежный рынок `money_market` (LQDT ISS, RUONIA ЦБ, кэш `money_market_quotes`; ADR 0078); окно стратегии в МСК, версия правил `backtests.execution_rules_version` и `DELETE /backtest/legacy` (ADR 0079); сценарии договора данными и эталон B14 (ADR 0076, 0077); новых настроек в `config.py`, заданий планировщика и зависимостей нет; ревизия `8bddf72586cf` поверх `b07d34cd8897`, head `8bddf72586cf`. v4.11 — блок Б-03 «Скринер: снимок и карта» (в `develop` с 2026-11-18, PR #142, мерж-коммит `98d8f23`; в рабочем терминале ещё нет): модуль `app/screener/` — справочник вселенной в колонках `instruments` (акции TQBR `EQIN`, статус у брокера `trading` / `break` / `not_on_tqbr` по одному `Shares` счётом для рыночных данных; ADR 0090), итоги дня ISS `screener_iss_day_totals` и статистика процесса — медиана оборота за 60 дат и шкала P90 за 250 (ADR 0091), снимок `GET /screener/snapshot` — доска ISS (кэш 15 с) и `GetLastPrices` ключом счёта пользователя (кэш 5 с, адаптер на счёт в памяти), давняя цена брокера — по ISS (`broker_quote_is_current`; ADR 0092), поправки секторов `screener_sector_overrides` и `/admin/sector-overrides` (ADR 0093), порог ликвидности пользователя `screener_user_settings` (ADR 0095); задание `screener_nightly` (01:30 МСК и через минуту после старта); фронтенд — «Скринер», «Карта рынка» на `<canvas>` с `d3-hierarchy` 3.1.3 (ADR 0094), карточка «Поправки секторов»; новых настроек в `config.py` нет; ревизия `b03e5c71a9d4` поверх `8bddf72586cf`, head `b03e5c71a9d4`.

### История версий

| Версия | Дата | Спринт | Ключевые изменения |
|--------|------|--------|-------------------|
| 4.11 | 2026-11-18 | План 008 — блок Б-03 «Скринер: снимок и карта»: E1 часть 1, E2, E3, E4 и A9 в части оборота (бриф и восемь развилок — решения заказчика 2026-11-16; макеты «Скринер Б», «Карта А» — 16.11; ремонт R24 — решение 2026-11-17) | Основание — ФТ v4.19. Код — ветка `feat/008-b03-screener-snapshot` от `14d76e3` (голова PR #131; Autopilot semi/strict; план `0092d5a`, таски T01–T06, ремонты по слепой приёмке F2 и по ревью F1, сдача `ee31ed9` 16.11; `5510746` — слияние `develop` `ede13c3`, кандидат стенда 17.11; `c4cf969` — давняя цена брокера по ISS с пометкой (R24); `2ce61d7` — слияние `develop` `da73abf`; `d1457ec` — запись о перепроверке), влита в `develop` 2026-11-18 (PR #142, мерж-коммит `98d8f23`) по локальным гейтам: pytest в режиме CI — 8835 passed / 1 skipped, покрытие 93 %; ruff, mypy, bandit, `alembic check` — чисто; vitest 2665; E2E 322 passed / 1 skipped (сборка). В рабочем терминале ещё нет (выкатка — тегом `v1.1.0` после приёмки на тестовом стенде). **Глоссарий, §1.2, §2.1:** термины скринера, `d3-hierarchy` 3.1.3, модуль `app/screener/`. **§3.0, §3.31 (новый):** ревизия `b03e5c71a9d4` поверх `8bddf72586cf` — десять колонок `instruments` (`short_name`, `list_level`, `in_screener_universe`, `universe_synced_at`, `broker_status`, `broker_qual_only`, `broker_short_enabled`, `broker_uid`, `broker_synced_at`, `iss_history_from`), таблицы `screener_sector_overrides` (четыре стартовые поправки), `screener_iss_day_totals`, `screener_user_settings`; head `b03e5c71a9d4`. **§4.0, §4.19 (новый):** `GET /screener/snapshot`, `GET/PUT /screener/settings`, `GET /screener/sectors`, `GET/POST /admin/sector-overrides`, `PUT/DELETE /admin/sector-overrides/{id}`. **§5.9, §5.18 (новый):** задание `screener_nightly`; проход, синхронизация справочника, история и статистика, снимок с правилом `broker_quote_is_current`, поправки и аудит, скрипты `record_b03_fixtures.py` и `check_b03_sector_indices.py`; новые методы `MOEXISSClient` (`get_board_snapshot`, `get_board_day_totals`, `get_ticker_day_totals`, `get_index_weights`) и `TInvestAdapter` (`get_shares`, `get_last_prices`). **§6.1, §6.9, §6.12 (новый):** пункты меню, `screenerStore`, страницы «Скринер» и «Карта рынка», `SectorOverridesCard`. Решения — ADR 0090–0095. **Тестовый стенд 17–18.11** (кандидаты `5510746` и `2ce61d7`, копия рабочей БД) — **PASS WITH NOTES**: `trading=260 break=1 not_on_tqbr=0`, коды секторов — только 10 словаря, без сектора — GAZC, GAZS, GAZT; ночной статус в `break` не превращается; первое заполнение — 780 страниц без сбоев; 107 ликвидных, шкала 3,80 %; 89 из 96 по отраслевым индексам; `GetLastPrices` ≈ 6 в минуту. Главное замечание R24 (давняя цена брокера шла `live`) — исправлено `c4cf969` и перепроверено 18.11: `live` 243, `iss` 18 из 261. Известные ограничения — §5.18.7, §6.12. |
| 4.10 | 2026-11-17 | План 008 — блок A-06 «Договор исполнения»: B3, B4, B2, годовой Sharpe и денежный рынок A5, прогрев B5, сценарии договора и эталон B14, дефекты 10, 11, 12, 14, окно стратегии в МСК (бриф и пять развилок — решения заказчика 2026-11-10, вопрос 2 брифинга — 2026-11-10; замечания стенда — решения 2026-11-16) | Основание — ФТ v4.18. Код — ветка `feat/008-a06-execution-contract` от `develop` `92eb2ee` (Autopilot semi/strict, 128 требований, слепая приёмка из чистого клона — 36 из 41 пункта «реализовано», 4 — «только на стенде», 1 — «частично»; план `b5502ec`, таски `800ebc3` T01, `2249f60` T03, `3e5f0b0` T02, `77e9349` T04, `121a0cf` T06, `0c7a813` T05, `14ad81d` T07, `b18b2e6` T08, `8918182` T10, `b2a4999` T11, `8351cf7` T09, ремонт по ревью `b698a60` F1, сдача `2914164`; `362cc44` слияние `develop` `7a0e137`; `b168a2d` слияние `develop` `402e7e0` — миграция перевешена с `5613e921bcc1` на `b07d34cd8897`; `ff4e86d` слияние `b3ce7c4` (#129) и `06a231f` — тесты A-06 на `file_db` (кандидат стенда); `14d76e3` — календарь скрипта эталона по таблице периодов), влита в `develop` 2026-11-17 (PR #131, мерж-коммит `2927e39`) по локальным гейтам: pytest в режиме CI — 8623 passed / 1 skipped, покрытие 93,05 %; ruff, mypy, bandit, `alembic check` — чисто; vitest 2544; E2E 298 passed / 1 skipped. В рабочем терминале ещё нет (выкатка — тегом после приёмки на тестовом стенде). **Глоссарий, §2.1:** пакет `app/execution/`, новые модули `backtest/` и `trading/`. **§3:** ревизия `8bddf72586cf` поверх `b07d34cd8897` — таблицы `execution_journal`, `dividend_accruals`, `money_market_quotes`; `trading_sessions.slippage_value` / `slippage_mode` / `dividend_tax_pct` (0 / `pct` / 0 у прежних); `backtests.execution_rules_version` (NULL у прежних — «до договора исполнения»), `dividend_tax_pct`, `price_pnl`, `dividend_pnl`, `bars_per_year`, `money_market_pct`, `money_market_source`, `report_notes`; `backtest_trades.row_kind`; писатель `live_trades.slippage`; head `8bddf72586cf` (§3.6–§3.9, §3.19, §3.30). **§4.0, §4.3, §4.18 (новый):** `GET /backtest/spread-check`, `GET /backtest/session-defaults`, `GET/DELETE /backtest/legacy`, `GET /trading/sessions/{session_id}/trades/{trade_id}/executions`; `dividend_tax_pct` и умолчание проскальзывания 0,1 % у запуска бэктеста и Grid; поля проскальзывания и налога у запуска сессии; ответ бэктеста — `sharpe_basis`, `legacy_execution_rules`, `money_market`, `price_pnl` / `dividend_pnl`, `benchmark.index_code`, `report_notes`, `row_kind`. **§5.3, §5.4, §5.12, §5.17 (новый):** договор исполнения в backtrader и тени, Grid с правилами аукционов, paper по минутам, журнал, дивиденды, прогрев, Sharpe и денежный рынок, окно МСК, сценарии и эталон. **§6.5, §6.6:** `ExecutionReport`, `LegacyExecutionBadge`, `SpreadWarning`, «Удалить все старые», строки начислений, поля окон запуска, `TradeExecutionJournal`. Решения — ADR 0070–0079. **Заменено:** §5.3.1 — `bt.Cerebro()` на запуск, `bt.analyzers.SharpeRatio`, эталон IMOEX → `build_cerebro`, `annual_sharpe`, MCFTRR; §5.3.1а — `TRADING_MINUTES_PER_DAY = 805` и ограничения «Grid без `AuctionAwareBroker`», «окно в UTC» сняты; §5.3.1б — `PERCENT_LOT_EXPR` и размер ордера в кодогене → намерение и `sizing.entry_lots`, шорт проверяется деньгами, как лонг; §5.4.1 — окно истории N=200 → `warmup_bars`; §5.4.4 — «исполнение по текущей цене + slippage», ветка T+1 → договор, ветка удалена; §5.4.6 — наблюдатель «только sandbox/real», `_DeferredEntry`, «paper внутри бара не проверяется» → и paper, `_PendingExecution`; §5.12 — источник `GetDividends` ключом владельца → `read_dividends`, начисление днём гэпа; §6.5 — Sharpe `sharpe_params`, IMOEX → по барам, MCFTRR; §3.9 — комментарий `slippage` «entry_price − entry_signal_price» (писателя не было) → проскальзывание сессии paper. **Тестовый стенд 15–17.11** (кандидат `06a231f`, копия рабочей БД) — **PASS WITH NOTES**: миграция — 65 из 65 бэктестов с версией NULL, 10 сессий 0 / `pct` / 0; paper против бэктеста за 16.11 — 1211 из 1213 сделок совпали побайтно, журнал — по две строки на сделку; sandbox #3 — 8 ордеров, журнал `signal` и цена брокера; Sharpe 1h ↔ D — 3,1 % и 1,6 %; «купить и держать» SBER — +37,94 %; денежный рынок, спред, эталон B14 (99 из 106 версий, повтор совпал) ✔. Известные ограничения — §5.17.11. Дополнено 17.11 после мержей PR #140 (`ede13c3`, ADR 0110, миграций нет): §5.9 — торговый день из таблицы периодов (`stored_trading_day`, `stored_weekend_session_day`, `trading_day_by_periods`), §5.17.11 п. 4 — исправлено; PR #139 (`7ed702c`, дополнения ADR 0072/0073, миграций нет): §5.17 — `_market_price_chain`, `_broker_market_estimate`, `_cached_close`, §5.17.11 п. 2 — исправлено. |
| 4.9 | 2026-11-16 | План 008 — блок Б-07 «ИИ-разметка и алерт новостей»: пункт D8, алерт «новость по бумаге» и метки новостей на графике из D7 (бриф и 13 решений заказчика 2026-11-10, выключатели — 2026-11-10, «не старше суток» — 2026-11-11, доработка — решения 2026-11-12) | Основание — ФТ v4.17. Код — ветка `feat/008-b07-news-ai` от `develop` `3c65d27`, подтянута до `92eb2ee` (Autopilot semi/strict, 87 требований, слепая приёмка 27 из 28 — расхождение закрыто ремонтом F2; план `840e73e`, таски `1367332` T01, `c908f6c` и `0322549` T02, `159d757` T03, `0fc1332` T05, `2d580c7` T04, `a6aca11` T06, ремонты `c702531` F1 и `810dec8` F2, сдача `ea9e8bb`; по тестовому стенду — `9a56d9d` «не старше суток» (ADR 0067), `cc6322d` слияние `develop` `7a0e137`, `5a49ea6` пачки по 10 и 3 попытки на эталоне (кандидат стенда 11–12.11)); влита в `develop` 2026-11-12 (PR #128, мерж-коммит `9b2dbc9`, CI 4 из 4: backend 8269 passed / 12 skipped, покрытие 92,92 %). Доработка — ветка `feat/008-b07-dict-hints` от `9b2dbc9` (`ae9d885` словарь и ревизия `b07d34cd8897`, ADR 0105; `0644eb6` фильтр слов, ADR 0068; `227df39` метки D/W/M и «ещё N», дополнение ADR 0066; `4085804` починка «view_chart»; `c8ba789` подсказка `b07-v3` и цена из ответа OpenRouter, ADR 0069; `9626344` docs; `9fc2613` слияние `develop` `cdea400` — PR #127 с зависимостями, к блоку не относится), влита 2026-11-13 (PR #130, мерж-коммит `402e7e0`) по локальным гейтам: pytest в режиме CI — 8316 passed / 1 skipped, покрытие 92,91 %; bandit — Medium 0 / High 0; vitest 2495 (273 файла); E2E 289 passed / 1 skipped. В рабочем терминале ещё нет (выкатка — тегом после приёмки на тестовом стенде). **§3:** ревизия `b07a1e5c0d42` поверх `5613e921bcc1` — `ai_provider_configs.is_background` + частичный уникальный индекс, таблицы `ai_background_settings` (засев: лимит 2 000 000), `ai_background_usage`, `news_settings` (засев: ИИ-разметка выключена), `news_ai_benchmarks`, колонки `news_items.ai_attempts` / `ai_note` / `alert_stage` (архив — `done`), `alerts.ticker` nullable + `news_filter` / `paused_at` / `active_since`, `alert_triggers.price` nullable + `news_id` / `labeler` / `status` / `note` и уникальность `(alert_id, news_id)`; ревизия `b07d34cd8897` поверх неё — только данные, 11 исключений `news_dictionary_terms`; head `b07d34cd8897` (§3.13, §3.21, §3.26, §3.29). **§4.0, §4.13, §4.14, §4.17 (новый):** `GET /news` — `sentiment`, `from`, `to`, `active`, `ai_note`; `GET /news/by-ticker`, `/news/securities`; `GET/PUT /news/settings`; `POST/GET /news/ai-benchmark`; `GET/POST /news/alerts`, `PATCH/DELETE /news/alerts/{id}`, `GET /news/alerts/{id}/triggers`; `GET /settings/ai/background`, `PUT /settings/ai/background/provider`, `PUT /settings/ai/background/limit`; поле `news` панели раскладки. **§5.9, §5.10, §5.14, §5.16 (новый):** новых заданий нет — `news_collect` читает выключатели, после словаря запускает шаг ИИ задачей, шаг алертов — по сроку ~10 с и после ответа модели; `news_reactions` читает главный выключатель; `complete_json` у провайдеров (`json_schema` со `strict`, у OpenRouter — `require_parameters` и `reasoning.enabled = false`, `max_retries=0`, срок 20 / 40 / 60 с); `EVENT_MAP["news.alert"]` (тип `news_alert`, каналы от `alert`), `news.alert` в `OUTBOX_REPLAY_EVENTS`; словарь — правило кавычек ADR 0105. **§2.1, §6.4, §6.7, §6.9, §6.10:** `NewsMarkersLayer` + `NewsMarkerPrimitive` + `newsMarkers.ts`, значок `panel-news-toggle-<id>`, вкладка `/settings?tab=news` (`NewsSettingsTab`), карточка `AIBackgroundCard`, `newsSettingsStore`, `newsAlertStore`, фильтры `sentiment` / `from` / `to` в `newsStore`, `NewsAlertModal`, `NewsAlertsPanel`. Новых настроек в `config.py`, заданий планировщика и зависимостей блок не добавил. Решения — ADR 0060–0069 и 0105. **Тестовый стенд 11–12.11** (кандидат `5a49ea6`, копия рабочей БД, ИИ выключен) — сутки без ИИ **PASS WITH NOTES** (1197 новостей, все `ai_note = disabled`; 117 срабатываний `sent` и 117 уведомлений `news_alert`, задержка ≤ 0,1 с; 73 строки `unchecked` / `ai_disabled`; выключатели и метки ✔; 0 Traceback); четыре замечания закрыты доработкой. Эталон: `deepseek/deepseek-v4-pro` — в сборке пройден (`b07-v2`, пачка 20), на стенде 1 из 3 (пачка 10), с `b07-v3` — 0 из 3 (12.11, отдельная копия рабочей БД). Решение заказчика 2026-11-12 «Влить как есть (а)»: ИИ-разметка выключена, модель и подсказка — после `v1.1.0`. Известные ограничения — §5.16.10. |
| 4.8 | 2026-11-10 | Починка времени в результатах бэктеста (ветка `fix/backtest-naive-timestamps`; тестовый стенд и мерж — решения заказчика 2026-11-10) | Основание — ФТ v4.16. Код — ветка `fix/backtest-naive-timestamps`; перед стендом в неё влит `develop` `92eb2ee` (A-04) мерж-коммитом `2e9ea4b` без конфликтов (новых `.timestamp()` от naive-дат в backend и `new Date(naive)` в компонентах бэктеста `develop` не принёс; кандидат тестового стенда 10.11); влита в `develop` 2026-11-10 (PR #126, мерж-коммит `7a0e137`); в рабочий терминал — тегом после приёмки на стенде. **§4.3:** `GET /api/v1/backtest/{id}` — `equity_curve[].time`, `candles[].time`, `trade_markers[].time` — unix-секунды UTC при любом поясе процесса (раньше зависели от `TZ` сервера; формат и поля не менялись; `trades[].entry_date/exit_date` — по-прежнему naive ISO, UTC); `GET /api/v1/backtest/{id}/export` (csv, pdf) — `entry_date`, `exit_date`, «Запущен», «Завершён» по МСК (было UTC; формат `ГГГГ-ММ-ДД ЧЧ:ММ:СС` и заголовки прежние); правило «naive из БД = UTC» на backend — `app.common.datetime_utils.unix_utc`. **§6 (общие правила), §6.5:** фронт — `utcIsoToUnix` и `parseUtcTimestamp` (`utils/mskTime.ts`), графики бэктеста сдвигают время на `MSK_OFFSET_SEC`, как основной график; таблица сделок, панель сделки и CSV из таблицы — по МСК. Схема БД и миграции (head `5613e921bcc1`) не менялись; новых настроек и зависимостей нет; `deployment_guide.md` не меняется. Гейты на `2e9ea4b`: pytest `DEBUG=true` — 8051 passed / 1 skipped; ruff, mypy (258 файлов), bandit — чисто; typecheck, lint, build — чисто; vitest 2370 passed (263 файла); E2E бэктеста на моках (10 спек) — 37 passed, полный набор Playwright не гонялся. CI PR #126 — 4 из 4 (backend 8040 passed / 12 skipped, покрытие 93 %); первый прогон дал ошибку teardown чужого теста `test_aggregator_gap.py::test_weekly_prefix_with_tinvest_down_is_incomplete` (фоновый `history_backfill` против `engine.dispose()` фикстуры, к правке не относится), упавший job перезапущен. **Тестовый стенд 10.11** (кандидат `2e9ea4b`, копия рабочей БД) — PASS: ответ бэктеста сверен с `market-data/candles` той же бумаги и периода на новом бэктесте (SBER 1h) и старых (SBER 1h, SBER D, GAZP 1h) — свечи и OHLC без расхождений, маркеры на датах сделок, CSV по МСК, PDF 200; перекрестие, таблица, панель, CSV и PDF дают одно время. Известные ограничения — §6.5. |
| 4.7 | 2026-11-10 | План 008 — блок A-04 «„Стоп всему“ и Circuit Breaker»: пункты C2 и C5 (бриф и решения заказчика 2026-11-05) | Основание — ФТ v4.15. Код — ветка `feat/008-a04-stop-all-cb` от `develop` `d26e5c5` (Autopilot semi/strict, 83 требования манифеста + G01, G02; план `e3c1587`, таски `8d94cbc` T01, `895a3e9` T02 (+ `3e322ae`), `e5ce0c1` T04, `8388bd2` T03 (+ `c452388`: флаг перед INSERT сделки входа, `trading_stop` отпускается до `runtime.start`), `b1dc0cb` T05, `c02ec72` T06, `23d7ea0` F1, сдача `28b8926`; дополнение G02 — `dffb816`, `2e97c98` T07 (`close_report`, одно правило причины), сдача `0535d2f`; `52573f6` — `CLAUDE.md`); перед мержем — `01c2984` (влит `develop` `0a5927d` с Б-01, A-02, #120, Б-04: миграция перевешена на `b6b5064c0be6`; кандидат тестового стенда 09–10.11), `9bfdce3` (E2E: поле стопа по роли), `31cd81a` (влит `develop` `2d93cbe`, #123, #124, без кода); влита в `develop` 2026-11-10 (PR #125, мерж-коммит `92eb2ee`); в рабочий терминал — тегом после приёмки на стенде. **§3:** `trading_stops` (журнал-флаг, частичный уникальный индекс `uq_trading_stops_user_active`), `circuit_breaker_event_sessions`, колонки `circuit_breaker_events.session_limit`, `close_report` (§3.28), `trading_sessions.pause_reason` (§3.8); действия журнала `trading.stop_all.enabled\|disabled` (§3.11); одна ревизия `5613e921bcc1` поверх `b6b5064c0be6` (head; данные не переносит; замер — `deployment_guide.md` §7). **§4:** `GET\|POST\|DELETE /api/v1/trading/stop-all` (снятие — `require_browser_session`, 409 `trading_not_stopped`), `SessionStartRequest.cb_limits`, 409 `trading_stopped` у `POST /trading/sessions` и `PATCH …/resume`, `SessionResponse.pause_reason`, `has_session_limits` (§4.4); `GET /circuit-breaker/config/inherited`, `GET\|DELETE /circuit-breaker/config/session/{id}`, `GET /circuit-breaker/history` (§4.16); кадры `stop_all.changed`, `cb.triggered`, `cb.close_report` в `system:{user_id}`, `session.limits_changed` (§4.12, §11.6). **§5:** `StopAllService` и отсечка входа (`check_before_order`, `OrderManager._open_entry`), лок `trading_stop`, `restore_all` при стопе, подписка уведомлений в повторе списка (§5.4.7); `CBConfigSnapshot` — общие (`general_limit`) и собственные поля, свой охват `session_limit`, `cb_pause_reason`, связи и копия `cb.triggered` в `_trigger`, изоляция сбоев `_force_close_open_positions`, `build_close_report`, `history.py`, `get_inherited_values` (§5.8); `close_failure_reason` — ошибка БД текстом; `/stopall` в `telegram_webhook.py`. **§6:** `StopAllButton`, `StopAllBanner`, `LiftStopModal`, `stopAllStore`, `CbHistoryTable`, `CbLimitsFields`, `SessionLimitsModal`, раздел лимитов `LaunchSessionModal`, ⚠ причины паузы (`utils/pauseReason.ts`). Новых настроек, зависимостей, заданий планировщика и `event_type` нет. Решения — ADR 0040–0045. Гейты на `01c2984`: pytest `DEBUG=true` и как в CI — 8048 passed / 1 skipped, покрытие 93 %; ruff, mypy (258 файлов), bandit — чисто; `alembic upgrade head` + `alembic check` на чистой БД — чисто; typecheck, lint, build — чисто; vitest 2364 passed (260 файлов); E2E полный (живой backend на чистой БД) — 269 passed, 1 skipped (после `9bfdce3`; спеки блока — `a04-stop-all.spec.ts`, `a04-cb-history-limits.spec.ts`). Известные ограничения — §5.4.7, §5.8. |
| 4.6 | 2026-11-09 | План 008 — блок Б-04 «Рисунки»: пункты E8, E9 и E10 (бриф и решения заказчика 2026-11-05; решение 2026-11-09 о старых рисунках) | Основание — ФТ v4.14. Код — ветка `feat/008-b04-drawings` от `develop` `45a67bd` (Autopilot semi/strict, 66 требований манифеста; план `1db29d6`, таски `1a56a67` T02, `233917a` T01, `256e361` T03, `a8a4e23` F1 (ремонт по ревью ветки: проверка данных рисунка при смене типа, общие якоря, кнопки тулбара из одной таблицы), сдача `63a072f`; `e64ca49` — `CLAUDE.md`); перед мержем — `cb825d2` (влит `develop` `d4223c9` с Б-01 и A-02: миграция перевешена с `0acf47e6607f` на `79f3c238dbb8`; кандидат тестового стенда 09.11), `1229ebe` (итог стенда), `975b536` (влит `develop` `9adbc34`, #120, без конфликтов в коде); влита в `develop` 2026-11-09 (PR #122, мерж-коммит `0a5927d`); `a6798da` (PR #123) — ADR 0054: старые линии SBER не перерисовываются (решение заказчика 2026-11-09), без кода; в рабочий терминал — тегом после приёмки на стенде. **§3:** схема не меняется; `chart_drawings` — новые типы и поля `data_json` (`p3`, `levels`), одна ревизия только данных `b6b5064c0be6` поверх `79f3c238dbb8` (head; удаляет рисунки с точкой года < 2000; замер — `deployment_guide.md` §7) (§3.14). **§4:** `DELETE /api/v1/charts/{ticker}/drawings` — `timeframe` необязателен, без него — все ТФ бумаги (§4.13); типы `ray`, `extended`, `hray`, `channel`, `fib`, поля `p3` и `levels` с проверкой 422 (§4.13); `GET /api/v1/market-data/trading-hours` (§4.5, реестр §4.0). **§5:** маршрут часов торгов читает `get_trading_schedule()` (§5.9.1). **§6:** правило точки `coords.drawingBarIndex` / `regularBarIndexForTime` / `sequentialIndex.seqIndexForTime`, `selectDrawings(state, ticker)`, `MAX_DRAWINGS_PER_TICKER`, `clearAll(ticker)`, `purgePre2000LocalDrawings`, удалены `utils/drawingsMigration.ts` и ветки по `logical`; тулбар из таблицы (`TOOL_BUTTON_META`, `LINE_TOOL_META`), примитивы `ChannelPrimitive`, `FibPrimitive`, `RulerPrimitive`, луч и продлённая в `TrendlinePrimitive`, горизонтальный луч в `HlinePrimitive`; `FibPropertiesModal`; линейка — состояние `DrawingsLayer`, подпись `rulerLabel.ts`, `marketDataApi.getTradingHours` (§6.4, §6.9). Новых настроек, зависимостей, заданий планировщика и `event_type` нет. Решения — ADR 0050–0054. Гейты на `cb825d2`: pytest `DEBUG=true` и как в CI — 7913 passed / 1 skipped, покрытие 92,62 %; ruff, mypy, bandit — чисто; `alembic upgrade head` + `alembic check` на чистой БД — чисто; typecheck, lint, build — чисто; vitest 2243 passed (247 файлов); E2E полный (живой backend на чистой БД) — 252 passed, 1 skipped. После `975b536`: vitest 2247, E2E графика (`b04-drawings`, `b02-chart-panels`, `s5-*`) — 60 passed; CI PR #122 — 4 из 4. Известные ограничения — §6.4. |
| 4.5 | 2026-11-09 | План 008 — блок A-02 «Старая история и дивиденды»: решение 5 «Шов источников», пункты A2, A4 и сквозное правило A8 в объёме блока (бриф и решения заказчика 2026-11-03) | Основание — ФТ v4.13. Код — ветка `feat/008-a02-old-history-dividends` от `develop` `917d9dc` (Autopilot semi/strict, 74 требования манифеста; план `715d0e5`, таски `110c73c` T01, `60b9a5e` T02, `162fb30` T04, `2943174` (ремонт T02: покрытие дневок окна ISS по суткам МСК), `5b1fb9d` T03, `5faa0b5` T05, `3d0bd8c` F2 (пометка «отменена на Смартлабе»), ремонт `ab07e45` F1 и `3726120` (повторное ревью), сдача `f13c01b`, `7d8bc10` — `CLAUDE.md`; перед мержем — `30cc14a` (влит `develop` `185b9be` с A-03 и A-05: миграция перевешена на `cdb15f21e5ca`, ADR 0012–0018 перенумерованы в 0030–0036, вселенная фона берёт алерты из общей модели `Alert`), `a829843` (влит `develop` `45a67bd` с Б-02, #114, #115: миграция перевешена на `0acf47e6607f`, в `_read_stored` — `exclude_iss` A-02 и `only_iss` #115), `104b9e2` (влит `develop` `3c65d27` с Б-01: миграция перевешена на `b01e7a5c3d92`; кандидат тестового стенда с 06.11); по тестовому стенду — `acbea74` (праздники раньше 09.05.2018 — не «пропуски» дневок: `ohlcv_quality_report` + `iss_trading_days`, кэш `_ISS_GAP_DAYS`, предел 15 с); по находке 3 стенда — `088b9e6` (загрузка дивидендов T-Invest не упирается в лимит: только акции TQBR `SHARE_SECTYPES`, 261 бумага; пауза 0,8 с; ожидание и повтор при отказе по лимиту; первыми — давно не загружавшиеся); по CI — `1bf985b` (тесты фоновой загрузки ждут исполнителя по событию — правка флака CI, код приложения не менялся)); влита в `develop` 2026-11-09 (PR #121, мерж-коммит `d4223c9`); в рабочий терминал — тегом после приёмки на стенде. **§3:** колонка `broker_accounts.is_market_data_source` и частичный уникальный индекс `uq_broker_accounts_market_data_source` (§3.2); новые таблицы `ticker_aliases` (§3.5.6) и `dividend_payments`, `dividend_corrections`, `dividend_smartlab_rows`, `dividend_sources` (§3.27, §3.23); в `ohlcv_coverage` — отметки `tinvest_invalid` / `moex_iss_invalid` (§3.5.2); `ohlcv_backfill_target.depth_from` больше не читается (§3.5.4); одна ревизия `79f3c238dbb8` поверх `b01e7a5c3d92` (head; data-часть — минутки T-Invest до шва, дневки T-Invest и ISS-дневки окна с чужой меткой, фантомы; на пути выкатки — 0 строк; замер — `deployment_guide.md` §7). **§4.0, §4.3, §4.5, §4.6, §4.15 (новый):** `PUT /api/v1/broker-accounts/{id}/market-data-source` (администратор; 422 с причиной), `GET\|POST /api/v1/admin/ticker-aliases`, `PUT\|DELETE /api/v1/admin/ticker-aliases/{id}`, `GET /api/v1/dividends`, `GET /api/v1/dividends/{ticker}`, `POST /api/v1/dividends/{ticker}/corrections`, `DELETE /api/v1/dividends/{ticker}/corrections/{id}`; `CandlesResponse.history_pending`, `history_pending_reason`; строки «догружается» в `warning` `GET /backtest/{id}`; `BrokerAccountResponse.is_market_data_source`. **§5.3.3, §5.6.6, §5.6.8 (новый), §5.9, §5.15 (новый):** `seam.py`, `IssHistoryLoader`, `backfill.py` переписан (вселенная, очередь, один исполнитель; запуск при входе убран из `auth/router.py`), `market_data_account.py`, `ticker_aliases.py`, `SYNC_TINVEST_SPAN` = 365 сут, `_series_chunks` по 31 сутки, `AGGREGATE_CACHE_MAX_BARS` 500 000 → 80 000; ISS-клиент — `get_minute_candles`, `get_history_days`, `get_board_tickers`; `MOEXCalendarService.known_from`; задания `history_backfill` (04:00 MSK + старт), `load_tinvest_dividends` (04:30), `check_smartlab_dividends` (05:00); модуль `app/dividends/` (`html.parser`, без новых зависимостей). **§2.1, §6.1, §6.5, §6.7, §6.9, §6.11 (новый):** страница `DividendsPage` (`/dividends/:ticker?`), `dividendsApi`, пункт меню «Дивиденды» после «Счёта», переключатель «Рыночные данные» в `BrokerAccountList`, пометка `market-data-source-missing`, `settingsStore.setMarketDataSource` / `accountsLoaded`, карточка `TickerAliasesCard` в админке. Новых настроек в `config.py` нет — константы в `market_data/seam.py`, `market_data/service.py`, `backfill.py`, `dividends/service.py`, `dividends/smartlab.py`. Решения — ADR 0030–0036. Гейты на `acbea74`: pytest как в CI 7829 passed / 1 skipped, покрытие 92,59 %; ruff, mypy (256 файлов), bandit — чисто; vitest 236 файлов / 2064 теста; E2E на моках 48 passed (`104b9e2`); CI PR #121 — 4 из 4. Тестовый стенд — PASS WITH NOTES (2026-11-09; `104b9e2` с 06.11, `088b9e6` с 09.11; `acbea74` принят живым бэктестом SBER D 2017–2019 — §5.3.3). Замечания на выкатку: 7 полных копий БД (≈ ×8 размера базы, `deployment_guide.md` §7); сверка Смартлаба не учитывает дробление акций (§5.15); узкая гонка остановки фона (§5.6.8); сдвиг времени в бэктесте — ветка `fix/backtest-naive-timestamps`, не A-02. Известные ограничения — §5.6.8, §5.15, §6.11. |
| 4.4 | 2026-11-06 | План 008 — блок Б-01 «Новости: сборщик, архив, лента»: пункт D7 в объёме блока (бриф и решения заказчика 2026-10-25) | Основание — ФТ v4.12. Код — ветка `feat/008-b01-news` от `develop` `76480cc` (Autopilot semi/strict, 68 требований манифеста, из них R62–R68 отложены по брифу — другие блоки и перевешивание миграции по команде заказчика; таски `34c0d1c` T01, `0cbdda5` T03, `c5d48f4` T02, `0c2005b` T04, ремонт `3f97db7` F1 (кодировка лент, настоящий ответ ЦБ) и `10d9057` F2 (подпись типа целиком), сдача `8309a80`; `0fe5542` — замер словаря вынесен в `backend/scripts/news_benchmark.py`; перед мержем — `d7dc5a8` (влит `develop` с Б-0, A-01, A-03, A-05: миграция перевешена на `cdb15f21e5ca`), `a4e320c` (влит `develop` `4c70407` с Б-02 и #114: миграция перевешена на `0acf47e6607f`; кандидат тестового стенда 05–06.11), `35bf917` (влит `develop` `45a67bd`, PR #115); по тестовому стенду — `e769c1e` (псевдоним «GloraX» → `GLRX` в посеве ревизии)); влита в `develop` 2026-11-06 (PR #116, мерж `3c65d27`); в рабочий терминал — тегом после приёмки на стенде. **§3:** шесть новых таблиц `news_sources`, `news_items`, `news_labels`, `news_tickers`, `news_reactions`, `news_dictionary_terms` (§3.26, §3.23); одна ревизия `b01e7a5c3d92` поверх `0acf47e6607f` (head; обратима; посев — 7 источников и 217 терминов словаря; замер — `deployment_guide.md` §7). **§4.0, §4.14 (новый):** `GET /api/v1/news` (фильтры `ticker`/`source`/`type`, курсор `published_at|id`, `limit` 1..200), `GET /news/sources`, `GET /news/tickers` — только вошедшему, без входа 401; проценты реакции — строкой. **§5.9, §5.14 (новый):** задания `news_collect` и `news_reactions` (`IntervalTrigger(seconds=60)`, `misfire_grace_time=600`, `max_instances=1`); `FeedFetcher` (таймауты 5/15 с, срок 30 с на источник, тело ≤ 2 МБ, отказ от DTD/ENTITY и UTF-16/32, `ElementTree` с `# nosec B405/B314`, прокси окружения), `NewsCollector` (`INSERT … ON CONFLICT DO NOTHING`, транзакция на источник, сбой — в строку источника), `DictionaryProvider`/`TickerDictionary` (TQBR из ISS, кэш 24 ч, `_ALIASES`/`_EXCLUSIONS` в БД), `label_unlabeled`, `ReactionWorker`/`compute_reaction` (1m ISS, срок P + 75 мин, 20 за проход, 10 попыток; OZON — −0,2118 / −0,8207 %). **§2.1, §6.1, §6.9, §6.10 (новый):** модуль `news/` в дереве; страница `NewsPage` (`/news`), `newsStore` (курсор, «поколение» ответов, `refresh()` раз в 60 с), `newsApi`, пункт меню «Новости». Новых настроек в `config.py` нет — интервалы и пороги в `app/news/constants.py`. Решения — ADR 0101–0104. Известные ограничения — §3.26, §5.14, §6.10. |
| 4.3 | 2026-11-05 | План 008 — блок Б-02 «Графики: панели и виды»: G1 без дефекта 9, G2, правило A4, переключатели «внебиржевые часы» (A1) и «как было» (бриф и решения заказчика 2026-10-26) | Основание — ФТ v4.11. Код — ветка `feat/008-b02-chart-panels` от `develop` `917d9dc` (Autopilot semi/strict, 57 требований манифеста + D01, D02; таски `2985df8` T01, `d36ed8a`+`bc08eb6`+`53f8a10` T02, `194d079` T03, `a466be6`+`bff1c71` T04, `12c7cb1` T05, `521cddd` T06, ремонт `2c7c944` F1 и `bcb7ece` F2, сдача `366c36a`; по тестовому стенду — `8e4db57` (избранные ТФ в заголовке каждой панели), `1b7803e` (перекрестие не залипает на живых свечах), `7461bb6` (легенда после снимка из кэша, связь символа при том же тикере), `ad20e04` (очередь `PUT` раскладки, предел документа 64 КиБ, без повтора подписки до `auth_ok`), `dad3c68` (пометка ISS и подпись источника — под легендой); перед мержем — `1670a33` (влит `develop` с A-03 и A-05: миграция перевешена на `cdb15f21e5ca`, ADR 0012–0017 перенумерованы в 0024–0029, пометка ISS A-05 перенесена в ряды панелей) и `71852b2` (влит `develop` с #114)); влита в `develop` 2026-11-05 (PR #111, `4c70407`; CI 4 из 4, backend 7393 passed, покрытие 92,20 %, vitest 1976); в рабочий терминал — тегом после приёмки на стенде. **§3:** новая таблица `chart_layouts` (§3.25, §3.23); `chart_drawings` — индекс `idx_drawing_user_ticker (user_id, ticker)` вместо `idx_drawing_lookup` (§3.14), строки не переписываются; одна ревизия `0acf47e6607f` поверх `cdb15f21e5ca` (head; обратима; замер — `deployment_guide.md` §7). **§4.0, §4.13 (новый), §11.10:** `GET/PUT /api/v1/chart-layout` (`ChartLayoutDoc` версии 1; 422 на неверный документ и сверх 64 КиБ); рисунки — `GET|POST /api/v1/charts/{ticker}/drawings` (ТФ в теле, `ChartDrawingCreate.timeframe: Timeframe`), `DELETE /api/v1/charts/{ticker}/drawings?timeframe=` (обязателен), прежние `/{ticker}/{tf}/drawings` сняты без переходного периода; `GET /market-data/candles/dealer` — query `adjust`; у `/candles?adjust`, `/candles/dealer` и `/candles/subscribe?source=dealer` появился потребитель. **§2.3, §3.5.1, §5.6:** дилерский канал и `get_dealer_candles(…, adjust)` читает панель графика. **§6.4, §6.9:** страница графика из панелей (`ChartPage`, `ChartLayoutBar`, `ChartPanel`, `ChartPanelHeader`, макет А), `chartLayoutStore` (дебаунс 500 мс, очередь `PUT`, D01 — ADR 0028), `chartCrosshairStore`, ряды `marketDataStore.series` / `streamChannels` (`seriesKey`, `streamKey`, `ensureSeries`, D02 — ADR 0029), `useCandleStream` / `CandleStream` вместо `useChartLiveData`, `mergeLiveCandle` (кадр `bar_pending`), `priceSeries.ts` (5 видов, 3 шкалы, `LOG_HINT_RATIO = 3`), `sequentialIndex.barIndexForTime` / `dealerSequentialData`, `coords.ts` по номеру бара основного ряда, рисунки `byTicker` + ключ localStorage `drawings:{user}:{ticker}` версии 2 (`migrateLocalDrawingKeys`), пометка ISS — `SeriesState.lastSource` + `useIssDelay`; фильтр 07–20 UTC удалён. Известные ограничения — §4.13, §6.4. |
| 4.2 | 2026-11-04 | План 008 — блок A-05 «Дефекты этапа 1»: B12, C4, D5, E3, текст дефекта 14 (бриф и решения заказчика 2026-10-25) | Основание — ФТ v4.10. Код — ветка `feat/008-a05-stage1-defects` от `develop` `76480cc` (Autopilot semi/strict, 42 требования манифеста; таски `4b38508` T01, `7c35de5` T05, `e1d4909` T02, `9543a5d`+`dc255f0` T04, `bfb3f0c` T03, `2ed27a3` T06, ремонт `284f9de` F1, сдача `2481f7a`; перед мержем — `891970f` (влит `develop` с Б-0, A-01, A-03: миграция перевешена на `d92b7dd5c939`; тень `parity.interpreter_trades` совмещает правила аукционов A-03 с проверкой денег A-05) и `d2508bf` (тесты уведомлений — файловая БД + `NullPool` вместо `:memory:` + `StaticPool`, gotcha-95: CI ловил откат чужой незакоммиченной отметки «доставлено»)); влита в `develop` 2026-11-04 (PR #113, `185b9be`; CI на `d2508bf` 4 из 4, backend 7341 passed / 12 skipped, покрытие 92,12 %); в рабочий терминал — тегом после приёмки на стенде. **§3:** колонки `strategy_versions.parse_report` (§3.4), `backtests.lot_size` (§3.6), `notifications.channels_failed` (§3.12); `backtest_trades.volume_lots` — лоты (§3.7); одна ревизия `cdb15f21e5ca` поверх `d92b7dd5c939` (head; обратима; замер — `deployment_guide.md` §7). **§3.20** (и строка §3.0) приведены к коду: колонка `payload` (не `payload_json`), индекса нет, строк не чистят; payload `notification.dispatch`; `notifications.channels_sent` — CSV, а не JSON. **§2.4, §4.8, §5.7:** «гарантированная доставка не реализована» снято — outbox работает с S8R-AUDIT-086/FIX-065; с A-05 строку `notification.dispatch` пишет каждое уведомление с запланированным Telegram/почтой (`_create_notification_impl`, `_plan_external_channels`), повтор по сроку — `NotificationService.retry_due_deliveries` (`DELIVERY_RETRY_DELAYS_SEC = (10, 60, 300, 1800)`, `DELIVERY_MAX_ATTEMPTS = 5`), сторож `_dispatch_in_flight`, отметка `channels_failed` → `NotificationResponse.undelivered_channels`. **§5.9:** задача `retry_notification_delivery` (`IntervalTrigger(seconds=10)`, `misfire_grace_time=600`, `max_instances=1`). **§4.2, §5.2.2а (новый):** `block_parser` — `ParseIssue`, `ParseReport`, `is_reference_format`, построчный разбор формата ИИ-чата (приоритет И/ИЛИ/НЕ, правила уровней); `POST /strategy/parse-template` → `report`; `POST /strategy/{id}/versions` → 422 `{detail, error_code: "description_has_problems", report}`; `VersionResponse.parse_report`; копии версии переносят отчёт; идемпотентность версии сравнивает и `text_description`. **§4.3, §5.3.1б (новый):** `strategy/sizing.py` — `PERCENT_LOT_EXPR` с лотом и комиссией, `percent_lot_size`, `shares_for_lots`, `position_cost`, `SERVICE_STRATEGY_PARAMS`; кодоген объявляет `lot_size`/`commission_pct`, `engine.service_strategy_kwargs`; `backtest/lot.py` — `resolve_backtest_lot`, `is_legacy_share_lots`, `lot_unverified_warning`, `directory_lots`; ответы `GET /backtest/{id}` и списков — `lot_size`, `legacy_share_lots`; `strategy-params` без служебных параметров; экспорт — «Акций». **§6.3–6.5, §6.8:** `ParseReportView`, `strategyStore.parseReport`, тост отказа; `CandlestickChart` — `chart-source-delay` по `marketDataStore.lastSource`; `LegacyShareLotsBadge`, предупреждение «Лоты штуками», «шт.» в сделках; `UndeliveredChannels`. **Дефект 14 (T+1):** §3.10, §5.4.4, таблица «Дополнения v1.6» и реестр Спринта 5 приведены к Model A — paper T+1 по деньгам не моделирует (ФТ §7.4); мёртвая ветка `PaperBrokerAdapter.place_order` — A-06 с B4. Известные ограничения — §3.20, §5.2.2а, §5.3.1б, §5.7, §6.4. |
| 4.1 | 2026-11-04 | План 008 — блок A-03 «Часы торгов, стопы внутри бара, дилер» (бриф и решения заказчика 2026-10-26) | Основание — ФТ v4.9. Код — ветка `feat/008-a03-trading-hours-stops` от `develop` `917d9dc` (Autopilot semi/strict, 86 требований манифеста; таски `bced169` T01, `9795d3f` T03, `7358e3c` T02, `b6382f4` T07, `920226e`+`e43d8e9` T04, `1359e55` T05, `92c8fd6` T06, ремонт `c5cbc38` F1, `4147030` F3, `9a08236` F2, сдача `87b923a`; после сдачи — исправления: `bb8799e` (найдено на тестовом стенде 02–03.11: имя площадки `TradingSchedules` без регистра — живой T-Invest отвечает заглавными, и стенд работал на запасных часах; ночной вход при полном лимите позиций отбрасывается, а не откладывается), `bbb632d` (известный баг сборки, по слову заказчика: стоп внутри бара — сделка вне снимка наблюдателя по текущей цене, а не по low минуты с ценами до входа), `0952d9c` (только тест «Закрыть все» в Telegram с фиксированным «сейчас» — падал ночью и в праздник 04.11, найдено CI-прогоном)); влита в `develop` 2026-11-04 (PR #112, `dd6d8fa`; CI 4 из 4, backend 7123 passed / 12 skipped; локально 7134 passed / 1 skipped, покрытие 92,63 %); в рабочий терминал — тегом после приёмки на стенде. Архитектура и решения в репозитории кода: `docs/architecture.md`, `docs/adr/0012…0023`, `CONTEXT.md`. **§3:** новые таблицы `trading_schedule_periods`, `trading_schedule_empty_days` (§3.24), `alerts`, `alert_triggers` (§3.13 — вместо `price_alerts`, активные строки переносятся); колонки `live_trades.exit_retry_kind/exit_retry_since/exit_retry_notified_at` (§3.9), `backtest_trades.price_note` (§3.7); одна ревизия `d92b7dd5c939` поверх `a858647f24e6` (head; обратима; замер — `deployment_guide.md` §7). **§2.3, §4.0, §4.12:** публичный WS-канал `dealer:{ticker}:{tf}` (1m–4h; контракт с Б-02), `POST /market-data/candles/subscribe?source=exchange\|dealer`, `GET /market-data/price-alerts/quote`, `POST /price-alerts` без `condition` (направление ставит сервер; 409/422), `/market-status` — статусы `closing_auction`, `dealer`. **§5.5:** чтения T-Invest `get_trading_schedules`, `get_last_price`, `get_price_step`, `get_trading_status`, `get_dealer_order_book`; в очередь алертов мультиплексора — только биржевые свечи. **§5.9, §5.9.1 (новый):** `scheduler/trading_schedule.py` — `TradingSchedule` (кэш процесса, синхронное чтение), `fallback_day`, история из минут ISS SBER с 13.06.2018 (`backfill_from_iss`), T-Invest `TradingSchedules` площадки `moex_mrng_evng_e_wknd_dlr` на сегодня + 7 (`sync_from_tinvest`); задачи `sync_trading_schedule`, `refresh_weekend_trading_schedule`, `backfill_trading_schedule_iss`; `get_market_status` по фазе таблицы. **§5.4.6 (новый):** `common/trading_hours.py` — `entry_verdict`, `exit_verdict(ExitKind)` → `ALLOW`/`DEFER`/`DEALER_CHECK`/`DENY` (ADR 0013, 0019, 0020); `trading/pending_exits.py` (ADR 0014); `_IntrabarRiskWatcher` + `RiskMonitor.check_sl_tp_intrabar` (ADR 0015); `_SessionListener` закрывает бар по `bar.closed` / страховочному таймеру / следующему бару, `_DeferredEntry` (ADR 0016); `trading/dealer_exit.py` (ADR 0022). **§5.3:** `AuctionAwareBroker`, цена Д/W/M из сохранённых минут (`_StoredMinuteOpens`, ADR 0017, 0018), теневой прогон `parity` по тем же правилам; `TRADING_MINUTES_PER_DAY = 805` — число. **§5.4.5, §5.6.6:** `MARKET_DATA_STREAM_MODE` по умолчанию `single_include_weekend` (ADR 0021): дилерские свечи потока — `ohlcv_dealer` (`BarLoader.store_stream_dealer_minutes`, без покрытия), канал `dealer:`, риск-монитор; фильтр O=H=L=C в `get_candles` удалён; `bar_close.py` и CB читают таблицу периодов; `bar.closed` читают сессии. **§5.6.7 (новый):** уровневые алерты — `Alert`/`AlertTrigger`, `PriceAlertMonitor` (high/low минуты, гэп, повторное взведение), `common/price_step.py`, шаг цены `instruments.min_price_increment` (`ensure_lot_size`, раз в сутки), ADR 0023. **§5.7:** `system.alert` → `event_type` `alert` («Алерт», настройки наследует от `price_alert`). **§5.8:** окно часов CB по умолчанию 07:00–23:50, аукционы блокируют вход во всех режимах, `_get_trading_day_start` — `trading_day_start` по таблице, аварийное закрытие — `ExitKind.FORCED` + список, свежесть цены D — в торговых днях (G01). **§6.1, §6.4, §6.7:** `StatusFooter` (статусы `closing_auction`, `dealer`), `PriceAlertModal` (без «Выше/Ниже», текущая цена, подсказка направления, точность по шагу цены, `D04`), `riskLimits.ts` — 07:00–23:50, `EVENT_TYPE_LABELS.alert`. Известные ограничения — §5.3, §5.4.6, §5.6.6, §5.6.7, §5.8, §5.9.1. |
| 4.0 | 2026-10-24 | План 008 — блок A-01 «Хранилище баров» (решения заказчика 2026-10-23/24) | Основание — ФТ v4.8. Код — ветка `feat/008-a01-bar-storage` (Autopilot, сборка сдана 2026-10-24, 11 коммитов от `e07044b`: `9c6915f`…`d8260b9`; после сдачи — исправления по тестовому стенду: `71d5a5f` (TLS архива, скрипт сверки), `ccf0302` (архив по uid), `11bd9f0` (дефект 4: «как было», дробления, лот на дату), `9f43de4` (дефекты 5–6: день архива — только при точном совпадении с ISS `history`, устойчивая смена лота, эталонные дни в фикстурах, правки по двум ревью), `2b7c97a` (дефект 7: объём дня ISS — по календарным дням, торги выходного и праздничного дня вычитаются из итога следующего рабочего дня)); в `develop` и рабочий терминал не попал. Архитектура и решения в репозитории кода: `docs/architecture.md`, `docs/adr/0001…0009`, `CONTEXT.md`. **§3.5:** `ohlcv_cache` — только `1m` и `D`, индекс `idx_ohlcv_lookup` удалён (остался `uq_ohlcv_lookup`); новые таблицы `ohlcv_coverage`, `ohlcv_dealer`, `ohlcv_volume_check`, `ohlcv_backfill_target`, `ohlcv_lot_history` (§3.5.1–3.5.5); цепочка миграций `b9d2f6a14c83` → `e5a7c2d90b16` → `c4e81f3a9d27` → `f1b3d8e62a54` → `a858647f24e6` (head; data-миграции необратимы, схема обратима; замеры на копии рабочей БД — `deployment_guide.md` §7). **§4.0:** `GET /api/v1/market-data/candles/dealer` (задел для Б-02). **§5.4.5:** подписка T-Invest только на `1m`, остальные ТФ (в том числе D/W/M) — агрегатор поверх потока минут; второе соединение закрытых минут (`waiting_close`); `MARKET_DATA_STREAM_MODE`. **§5.6 и новый §5.6.6:** таблица покрытия вместо `_find_gaps`/`_tail_tolerance`/`check_cache_coverage`; `BarLoader` — единственный писатель свечей T-Invest; `aggregation.py` — один агрегатор для истории и потока (LRU `AggregateCache`); `bar_close.py` — одно правило «бар закрыт» (конец периода или конец торгов + 2 с); `ClosedBarTracker` и событие шины `bar.closed` (канал `bars:{ticker}:{tf}`, `complete`/`revised`; не уведомление, вне `EVENT_MAP`, клиентам по WS не отдаётся); дилерский ряд `ohlcv_dealer`; объём в штуках (лоты × лот), сверка с ISS (`ohlcv_volume_check`, порог 1 %); шов `TINVEST_MINUTE_SEAM` 2018-06-13; цена из кэша (`last_cached_bar`, `closed_bar_close_before`) — из хранимого ряда по правилу «бар закрыт»; фоновая догрузка при входе (`auth/router.py` → `backfill.run_backfill`, архивы `history-data` ≤ 30 запросов/мин); живая D/W/M и сессии на D/W/M из потока минут. **§6.4, §6.6:** таймфрейм `30m` (`TimeframeSelector`, `LaunchSessionModal`: 9 значений). Годовой Sharpe `30m` — по прежней формуле (A-06). **Дефект 4 тестового стенда (решение заказчика 2026-10-24) — §3.5, §3.5.3, §3.5.5, §4.0, §5.6.6, §5.9, §5.12:** ряды `ohlcv_cache`/`ohlcv_dealer` хранятся «как было» (цена и штуки своего дня): свечи `GetCandles` (1m, D, дилерские) раньше границы наступившего дробления перед записью — `splits.to_as_was` (цена × rt/rf, объём × rf/rt), ISS — только цена, архив `history-data` — как есть, объём = лоты × лот на дату (`lot_history.ArchiveLots`, новая таблица `ohlcv_lot_history`); дробления — `corporate_actions` (`action_type='split'`), граница — `split_opened_before(ex_date)`; чтение по умолчанию с учётом дроблений — единственное место пересчёта `MarketDataService._read_stored` и `_latest_bar`, ключ LRU включает наступившие дробления; `closed_bar_close_before` (оценка CB закрытых неоценённых сделок) — в масштабе момента сделки, `last_cached_bar` — в текущем; загрузчик перечитывает дробления на каждое окно `GetCandles`, момент пересчёта — время ответа окна; новое дробление с наступившей границей — `forget_bars_before_new_splits` **до** записи дробления, в той же сессии (строки до границы вместе с покрытием и сверками удаляются; сбой — ERROR, дробления в этот прогон не пишутся); планировщик и без живых сессий вызывает `detect_corporate_actions([])` — синхронизирует дробления; лот на дату: ISS не знает бумагу — лот истории или текущий (WARNING `archive_lot_iss_empty`), сбой записи истории лота загрузку не останавливает; сверка объёма — «как было» с «как было»; `GET /market-data/candles?adjust=splits\|none` (по умолчанию `splits`; задел Б-02); текст предупреждения бэктеста о сплите — «… цены и объёмы до него пересчитаны в текущий масштаб»; ревизия `a858647f24e6` — `ohlcv_lot_history` и удаление всех рядов свечей прежнего масштаба (`ohlcv_cache`, `ohlcv_dealer`, `ohlcv_coverage`, `ohlcv_volume_check`; на рабочей БД — только 269 строк ISS D); тест миграций голову больше не закрепляет; известные ограничения пересчёта — §5.6.6. **Дефекты 5 и 6 тестового стенда (четвёртый заход 2026-10-24; ADR 0011) — §3.5, §3.5.3, §3.5.5, §5.5.3, §5.6.6:** эталон дня — ISS `history` TQBR (`MOEXISSClient.get_daily_volumes`), дневка `candles.json` бывает неверна (10.04.2025, SBER 01.08.2025); день архива `history-data` идёт в основной ряд, только если Σ лотов минут окна биржевых часов (`EXCHANGE_WINDOWS`, UTC 03:50–20:50 / 06:50–20:50 / 06:50–15:50, первое совпавшее) × лот из `ALLOWED_LOTS` ∪ {текущий} точно равна итогу дня (`ArchiveLots.admit` вместо `lots_for`); пишутся только минуты окна; решение — по всем минутам суток; иначе `GetCandles` (биржа не торговала, нет точного совпадения, лот не подтверждён, ISS недоступен), без минут и без торгов — «сделок не было» без запроса; допуски `VOLUME_TOLERANCE`/`LOT_KEEP_TOLERANCE` сняты; смена лота — только устойчивая (`_drop_unconfirmed`), в `ohlcv_lot_history` — только подтверждённые; `history` пуст — `candles.json` есть → `GetCandles`, нет → окно с утренней сессией и лот истории/текущий (WARNING `archive_lot_iss_empty`); сверка объёма — по `history`, `candles.json` — только для дней без него; эталонные дни — `test_reference_days.py`, `fixtures/reference_days/`, `scripts/record_bar_reference_days.py`; ограничения — архив не используется для бумаг вне TQBR, «как было» дилерского ряда T 15–16.04.2026 ×10 (Б-02). |
| 3.9 | 2026-10-19 | S8R — сводный бамп Dependabot #77–#89 (решения заказчика 2026-10-19) | Основание — ФТ v4.7 (требования не меняются). Код — PR #91 `deps/dependabot-2026-10-19`, смержен → `develop` `c55bbbf`. **§1.1:** FastAPI 0.143+ (с 0.137 `app.routes` — дерево с `_IncludedRouter`, обход маршрутов в тестах — `iter_route_contexts`, gotcha-86); AI SDK anthropic 1.15+ построен на `httpx2` и отвергает клиент `httpx` — `ClaudeProvider` создаёт `httpx2.AsyncClient(follow_redirects=False)`, `httpx2` — прямая зависимость (gotcha-85); **TA-Lib удалён** (не импортировался нигде, индикаторы — `bt.indicators` Backtrader); админ-метрики — plotly 7.1 (plotly.js 4): у всех графиков `showSendToCloud: False`, кнопки выгрузки в облако Plotly нет. **§1.2:** React 19.3+, Zustand 5.0+, React Router 7.18+ (декларативный режим). **§8.1:** Node.js 24 LTS (jsdom 30 и vitest 5 не работают на Node 20), TA-Lib C library не нужна. Образы: nginx 1.31.7 (CVE-2026-93479 и др.), python:3.11-slim и node:24-alpine — новые digest. |
| 3.8 | 2026-10-19 | S8R — `S8R-FIX-139` (решение заказчика 2026-10-19, вариант «а») | Основание — ФТ v4.7 (требование не меняется: «позиция, о которой знает брокер, но не знает терминал, показывается, но сессию не останавливает» — ФТ v3.6). **§5.4, сверка позиций** (`SessionRuntime._reconcile_broker_positions`, предзагрузка `_prefetch_restore_portfolios`): портфель читается без справочника брокера (`fetch_instrument_info=False`, S8R-AUDIT-048), поэтому тикер позиции = FIGI; метка `source="strategy"` теперь ставится и по FIGI тикеров сессий счёта из локального справочника `instruments` (`_directory_figis`, БД, без сети, вне сетевого бюджета), а фильтр «бумага этой сессии» узнаёт инструмент и по FIGI. Обязательные бумаги для правила gotcha-56 (`_required_figis`) не меняются; сбой чтения справочника — прежнее поведение. Найдено ордерным сценарием 2026-10-19 (M2). Код — ветка `s8r/fix-139-manual-position-figi` `473951b` → PR #90, смержен → `develop` `479a429` (2026-10-19). |
| 3.7 | 2026-10-17 | S8R — `S8R-FIX-137`, `S8R-FIX-138` (решения заказчика 2026-10-17, вариант «а»); косметика `S8R-FIX-135`/`136` | Основание — ФТ v4.7. Код — в `develop` `70848d8` (PR #75, #76, смержены 2026-10-17 по локальным гейтам); рабочий стенд — по команде заказчика. **`/health.tinvest_connected`** (S8R-FIX-137): задача стрима жива **и** у мультиплексора нет опубликованной потери связи без восстановления (`TInvestStreamMultiplexer.connection_lost_reported`) — порог 30 с и фильтр рынка те же, что у уведомления. **`connection.lost.reason`** (S8R-FIX-138): `stream_loss_reason(exc, terminal=…)` в `app/broker/tinvest/errors.py` — текст и код в квадратных скобках (`_code_tag`), repr исключения — только в логах `multiplexer_disconnected` / `multiplexer_terminal_error`. Косметика (PR #75, `develop` `7d9bbbe`): `_format_timedelta` — «6ч 28м»; `roundForDisplay` во фронте — знак и цвет P&L от показанного числа. |
| 3.6 | 2026-10-15 | S8R — `S8R-FIX-133` (решение заказчика 2026-10-15) | Основание — ФТ v4.6. Код — в `develop` `4028d83` (PR #72, смержен 2026-10-15 по локальным гейтам); рабочий стенд — после детекта корп. действий. Scheduler: уведомление о дивиденде/купоне пользователю из `CorporateActionService.accrual_short_user_ids(action)` (владельцы позиций по условиям начисления с `not is_long_direction`) — с суффиксом `; по короткой позиции сумма спишется`; `accrual_recipient_user_ids` и он — из одного запроса `_accrual_owner_directions`. |
| 3.5 | 2026-10-15 | S8R — `S8R-FIX-131`, `S8R-FIX-132` (решения заказчика 2026-10-15) | Основание — ФТ v4.5. Код — в `develop` `585e297` (PR #70, #71, смержены 2026-10-15 по локальным гейтам); рабочий стенд — после детекта корп. действий. Модуль корп. действий: `process_dividend` и `process_coupon` — сумма со знаком `is_long_direction(trade.direction)` (шорт — минус, та же семантика, что у P&L/SL/TP, S8R-AUDIT-032); в логах `dividend_processed.total` / `coupon_processed.paper_total` — суммы со знаком. Scheduler: адресаты уведомления о сплите — плюс `split_position_user_ids(action)` (условия `process_split`: `filled`/`open`, открыта до 00:00 МСК дня сплита). Миграции нет. |
| 3.4 | 2026-10-15 | S8R — `S8R-FIX-129` (решение заказчика 2026-10-15) | Основание — ФТ v4.4. Код — в `develop` `833ac34` (PR #69, смержен 2026-10-15 по локальным гейтам), рабочий стенд обновлён. §8 (scheduler) и модуль корп. действий: тикеры джобы `detect_corporate_actions` — `CorporateActionService.detection_tickers()`: сессии `active`/`paused` и сессии любого статуса со сделками в начисляемых статусах (`filled`/`open`/`closed`); адресаты уведомления о дивиденде/купоне — владельцы живых сессий тикера плюс `accrual_recipient_user_ids(action)` (те же условия, что у начисления, `accrual_position_clauses`), без дублей. Начисление (`process_dividend`/`process_coupon`) не менялось — статус сессии оно и раньше не смотрело. Миграции нет. |
| 3.3 | 2026-10-15 | S8R — `S8R-FIX-130` (решение заказчика 2026-10-15) | Основание — ФТ v4.3. Код — в `develop` `8f5ce74` (PR #68, смержен 2026-10-15 по локальным гейтам), рабочий стенд обновлён. §3.21 — SSRF-барьер `app/ai/url_validator.py`: адрес-заглушка прокси/VPN (`_PROXY_PLACEHOLDER_NETS`: Proxifier macOS `0.0.0.0/8` без `0.0.0.0` и `fd00:696e::/32`, fake-IP `198.18.0.0/15` и `fc00::/18`) принимается только для хостов `KNOWN_AI_PROVIDER_HOSTS` (точное совпадение, регистр и точка в конце имени не важны) и только по https — и при проверке в фабрике, и при закреплённом резолве в момент соединения (`_is_forbidden_for_host`); IP-литерал и любое другое имя — прежнее правило, отказ по заглушке называет прокси/VPN. §API — `POST /settings/ai/providers/{id}/verify` при отказе проверки адреса отвечает 200 `{verified: false, error}` и снимает `is_verified` (раньше 422 `provider_url_blocked`, который форма показывала без причины). Миграции нет. |
| 3.2 | 2026-10-14 | S8R — решения заказчика 2026-10-14 (поздний вечер): `S8R-FIX-127`, `S8R-FIX-122` | Основание — ФТ v4.2. Код — в `develop` `834a05b` (PR #65, PR #66, смержены 2026-10-15 по локальным гейтам), рабочий стенд обновлён. `S8R-FIX-127`: §11.4 — `POST /backtest` и `/rerun` — job `single` менеджера (`_submit_single_backtest_job`), ответ + `job_id`, отмена `POST /backtest/jobs/{job_id}/cancel` (прерывает `RateLimitWait`), бэктест → `cancelled`; общий cap=3, `BacktestJobLimitError` → 422 `backtest_job_limit`, предпроверка до строки `Backtest`; защита записи `done` от поздней отмены, самовосстановление строк без задачи, shield постановки/уборки, останов → `failed` «Прервано: перезапуск сервера или сбой записи статуса под нагрузкой БД», `DELETE` снимает job; фронт — `cancelJob`, фактический итог вместо локального «Отменён», `backtestApi.cancel` удалён; `S8R-FIX-128` (2026-10-15) — лимит фонового запуска только сервером, значки статусов в таблице стратегии. `S8R-FIX-122`: §5.12 — дивиденды `TInvestAdapter.get_dividends` → `DividendInfo` (`app/broker/base.py`), `_fetch_tinvest_dividends`/`_read_dividends` (токен владельца, перебор владельцев, вывод недоступного счёта из прогона), окно от первой сделки до +366 дней, `_sync_pending_dividends` (сумма/отмена/перенос для непроведённых), ISS-путь дивидендов удалён; отсечка T+1 — `accrual_last_buy_date` вместо `accrual_ex_date`, порог — 00:00 МСК дня после последнего дня покупки; купон без `recorddate` — реестр за торговый день до выплаты; `_DetectedAction.record_date`, тексты уведомлений. Миграций нет. |
| 3.1 | 2026-10-14 | S8R — сплошная проверка UI-чеклиста S8.1–S8.58 (цикл 2026-10-13/14, `S8R-FIX-093…125`) | Основание — ФТ v4.1. Код — в ветках `s8r/fix-checklist-run` и `s8r/fix-093-grpc-resolver-import`, в `develop` попадёт после мержа. `S8R-FIX-093`: §5.5.2 — `GRPC_DNS_RESOLVER` grpc читает при импорте модуля; значение по умолчанию `native` ставит `app/__init__.py` (`app/grpc_env.py`, перенесён из `app/broker/tinvest/`), lifespan только логирует, `grpc_dns_resolver_ineffective` при раннем импорте grpc — фикс 083 не действовал. `S8R-FIX-115`: §5.5.2, §4.6 — занятость `account_id` песочницы по всем sandbox-записям, регистрация не заводит запись на счёт другого пользователя и на закрытый (422), свой счёт не «занят»; лок `sandbox_recovery` — один на процесс (было `sandbox_recovery_user` по пользователю), берётся внутри `create_account`: порядок `create_account` → `sandbox_recovery`. `S8R-FIX-119`: §7.2, §4.6 — `BrokerKeyUnreadableError(InvalidTag)`, 409 `broker_key_unreadable`, `close_failure_reason` без «брокер не вернул текст». `S8R-FIX-124`: §8.6 — периодическая сверка считает неполные проходы в предзагрузке (`count_incomplete_passes`), ручная (`manual=True`) счётчик не двигает и не подтверждает; пауза сверкой публикует `session.paused`. `S8R-FIX-097`: §11.6 — `status` в `session.paused/resumed/stopped` у всех издателей. `S8R-FIX-098`: §3.11 — `admin.granted/revoked`, `source=cli`. `S8R-FIX-100`: §4.8, §5.7 — `/notifications/` со слэшем. `S8R-FIX-101`: §4.1 — `/login` на 429 `setup-status` — «Слишком много запросов», без порта. `S8R-FIX-103`: §4.2 — смена статуса стратегии из меню одним PUT, `patchStrategyLocal`; `S8R-FIX-104` — подписи статусов. `S8R-FIX-114`: §11.4 — `RUNNER_NOTIFIES_JOB_TYPES`, одно уведомление на одиночный бэктест, Grid — «Grid Search завершён» с данными. `S8R-FIX-116`: §4.6 — подсказка «на странице «Торговля»», потолок `detail` 400 за счёт перечня. `S8R-FIX-117`: §5.7 — `session_mode_label` во всех ответах бота. `S8R-FIX-118`: §5.6 — `date_req` = сегодня МСК, отказ курсу из будущего, кэш до полуночи МСК. `S8R-FIX-120`/`125`: §5.7 — `money.format_price`; `trade.filled` — цена исполнения. `S8R-FIX-121`: §5.9 — `_signed_rub`. `S8R-FIX-122`: §5.12 — `iss_dividends_block_missing`, прошедшие дивиденды без получателей `processed=True`; источник — открытый вопрос. `S8R-FIX-102`: §6.4 — `customValues.barTime`, точка рисунка по времени свечи; смена набора графика — `autoScale` заново (qa2 Н-6). `S8R-FIX-107`: §6.7 — `brokerAccountName`. `S8R-FIX-108`/`099`: §6.1 — `navbar.collapsed.mobile`, мобильный бургер, `aria-label` пунктов. `S8R-FIX-109`: §6.1 — форматтеры дат в `Europe/Moscow`. `S8R-FIX-096`/`123`: §9.1 — окружение подпроцессов без полей `Settings`, сброс реестра `keyed_lock` между тестами, `test_backup_cli` с `--server-stopped`. Без отражения в разделах (UI-мелочи): `S8R-FIX-094` (`aria-label` колокольчика), `095` (e2e: мастер в `beforeAll`), `105` (список фоновых бэктестов не закрывается сам, ФТ §19.10), `106` (testid строк настроек уведомлений), `110`–`113` (просадка CB 2 знака, ширина поля имени, описание `/admin/metrics`, двойной DELETE). **Решения заказчика 2026-10-14:** §5.5.2 — `RateLimitWait` (ожидание лимита в бэктесте), кадр `notification.read` при снятии расхождения, уровни журнала watchdog. |
| 3.0 | 2026-09-24 | S8R — фиксы аудита 2026-09 (`S8R-AUDIT-NNN`) | Изменения по карточкам аудита `audit_2026-09.md`, пополняется по мере закрытия. **§5.2.4** — `S8R-AUDIT-012`: раунд обрыва стрима, фоновая дозагрузка минуток, флаги `bar_complete`/`bar_pending`, неполный бар не торгуется (§5.6). `S8R-AUDIT-011`: «только деньги» в портфеле песочницы — сбой, перезапрос; фон паузит после 3 подряд таких проходов; real — без повторов (§8.6). `S8R-AUDIT-010`: тиры пары при restore active → suspended → paused, проигравший дубликат — paused с уведомлением только если победитель поднят (§8.6). `S8R-AUDIT-009`: сверка при restore — параллельная предзагрузка портфелей под общим дедлайном, несверенные позиции → `paused` + `recovery_mismatch` critical (§8.6). `S8R-AUDIT-041`: CLI `rotate-encryption-key`/`rotate-jwt-secret`, key_id в шифртексте, сверка ключа с данными на старте (§7.2, §8.8). `S8R-AUDIT-040`: `COOKIE_SECURE` (не задан → prod всегда Secure), `app/common/cookies.py`, nginx X-Forwarded-Proto только от cloudflared (§7.4). `S8R-AUDIT-039`: logout без валидного access (всегда 204, отзыв best-effort одной транзакцией), refresh-cookie на path `/api/v1/auth`, фронт — повтор refresh только на 429 (§7.3–7.5). `S8R-AUDIT-038`: CSRF без fail-open при cookie-сессии, проверка Origin/Referer (login-CSRF), общий нормализованный allowlist `app/common/origin.py` (§7.4). `S8R-AUDIT-037`: lockout только на login, сброс счётчика после истечения; лимитер auth по (IP, username) + потолок на IP, refresh по (sub, IP), IPv6 /64, реальный IP — uvicorn `--proxy-headers` от nginx 172.28.0.10, nginx на 127.0.0.1:80; `app/auth/request_tokens.py` (§7.5). `S8R-AUDIT-008`: `restore_all` — `active` только после успешного `start()`, временный сбой → `suspended` с сохранением пары, постоянная причина → `paused`, `RESTORE_PAUSE_FROM` (§8.6). `S8R-AUDIT-014`: `ai_usage_daily` + глобальный потолок токенов серверного AI-ключа, учёт `/ai/explain` (§5.10.4). `S8R-AUDIT-072`: `app/common/trading_hours` — входы `market_closed_reason`/`is_market_open` (календарь `MOEXCalendarService.is_trading_day`, 10:00–23:50, кроме 18:40–19:05), закрытия `close_blocked_reason`/`is_within_trading_hours` (ночь и будничный праздник); одна точка «сейчас» `_now_msk`; CB — календарь только sandbox/real; мультиплексор `_maybe_publish_lost`. `S8R-AUDIT-004`: `alembic/env.py` — metadata из `app.main`; колонки `ai_provider_configs.daily_limit/monthly_limit/usage_today/usage_month` возвращены в модель; CI — шаг `alembic upgrade head && alembic check` на чистой БД (§8). `S8R-AUDIT-003`: `select_market_data_account` (токен только своего пользователя, prod-first), лот песочницы не authoritative для real/paper и не кэшируется, контекст у всех вызывающих (§5.6). `S8R-AUDIT-071`: миграция `6128b9c52d8a` — `position_sizing_mode` NOT NULL (NULL → `fixed_lots`/1); `sizing_config_error` — неизвестный режим или fixed_sum/percent без размера → CB постоянная блокировка (пауза), `calculate_position_lots` → `ValueError`; режимы — `schemas.VALID_SIZING_MODES` (§3.8, §5.8). `S8R-AUDIT-070`: CB дневной лимит — `money_class` (с S8R-FIX-028 = mode: paper / sandbox / real; было paper / broker = sandbox+real), `class_daily_loss_state` (P&L дня по сессиям класса, включая остановленные сегодня; % от Σ `initial_capital`), override сессии — на её P&L и капитал (`scope=own`); `CB_PAUSE_SCOPE` (all / money_class / own) вместо `CB_SCOPE_ALL_SESSIONS`; `get_status` — `daily_pnl_{paper,sandbox,real}`, `daily_loss_limit_{paper,sandbox,real}` (S8R-FIX-028) (§5.8). `S8R-AUDIT-015`: `PRAGMA foreign_keys=ON` на каждом соединении приложения (`database.py`), миграции — с FK OFF до транзакции (`alembic/env.py`, gotcha-79), `foreign_key_check` при применённых ревизиях; удаление стратегии с историей сессий/бэктестом в работе → 422; эндпоинты CB по `session_id` — проверка владельца (404); реально срабатывают `ondelete` SET NULL/CASCADE моделей (§3, §8.9). `S8R-AUDIT-027`: `session_capital` (free/equity, только БД); база `percent` — paper `balance` в секции входа, sandbox/real `min(free, GetPositions.money RUB)`, не больше `Signal.cb_percent_base`; кэш рублей 30 с на счёт (`app/broker/cash_cache.py`: поколения, резерв после сайзинга, сброс на `place_order`/`cancel_order`/`sandbox_pay_in`, проверка до построения адаптера, `_AdapterLease`); переоткрытие sandbox-счёта при 50004; CB без сети: проекция от `free`, лимит `risk_max_position_pct` от `equity` только для `percent` (§5.4, §5.8). `S8R-AUDIT-057`: security-заголовки двумя слоями (backend + nginx на все ответы), CSP SPA, кэш-политика, Blockly media локально, CI nginx -t (§7.6). `S8R-AUDIT-076`: лок `paper_portfolio` до первой записи на всех денежных путях paper, свежее чтение баланса (§5.4, `locks.py`). `S8R-AUDIT-058`: реестр WS (`app/common/ws_registry.py`), лимиты соединений/подписок, закрытие WS по сессии при logout/смене пароля/краже refresh, атомарный слот AI SSE, nginx `location ^~ /ws` (§4.12, §7.5). `S8R-AUDIT-029`: «Стоп» под `session_lifecycle` с защитой от повторной отмены, параллельное закрытие брокерских позиций с дедлайном, DELETE под локом (§5.4). `S8R-AUDIT-021`: код привязки Telegram `token_urlsafe(9)`, лимиты неудач 5/15 мин на отправителя и 100/15 мин глобально (§5 Telegram). `S8R-AUDIT-013`: единый предикат `authenticate_access_payload` (sub, jti, `is_active`, `ver`) — HTTP, три WS, dash-mount; login деактивированного — 401; Telegram: неактивный — отказ во всех командах и привязке, привязка гасит прежние активные привязки чата; исходящие Telegram/email — только активному (§7.3). `S8R-AUDIT-032`: `Signal.entry_direction` (из `StrategyIR.entry_direction`) + `opens_position`; `SignalProcessor` — вход long = BUY, short = SELL, выход — противоположная сторона; `process_signal` — сигнал против направления без позиции для выхода → HOLD (лог DEBUG: штатное состояние после закрытия); CB только для сигнала, открывающего позицию и не являющегося выходом (`OrderManager` — по тому же выбору встречных сделок filled+pending, что и exit-ветка; рантайм — `engine.is_exit_signal`); `_check_short_block` — short стратегии не блокируется, `block_shorts` сохранён для совместимости API (штатно не достижим); `PaperPortfolioAccountant.apply_close(direction=)` — short: в кэш `cost_basis + (cost_basis − proceeds)`, вход — `apply_open` (обеспечение 1:1); сверка с брокером — штуки со знаком (short < 0), нетто 0 по счёту — расхождение только при открытой позиции живой сессии; `models.is_long_direction` — единая проверка для учёта денег paper и сверки (§5.4, §5.8, §5.10.2). `S8R-AUDIT-080`: `app/strategy/risk_params.py` — `RiskParams` (SL (0,50] %, TP (0,100] %, размер) — единая точка для сохранения блоков (`validate_block_fields`, `parse_executable_blocks`), `/versions/from-params`, `params_sync`, `backtest/strategy_params`, Grid Search; `calc_sl_tp_prices` → `ValueError` при pct ≤ 0; `extract_risk_params(blocks_json)` — только IR; фронт `RISK_LIMITS` синхронен со схемой (тест) (§5.2, §5.4). `S8R-AUDIT-026`: `engine.calculate_position_lots` — единая формула лотов движка и CB без `max(1, …)`; 0 лотов → `process_signal` → `None` + `order.error` «Бюджет меньше одного лота» раз на серию (сброс на pause/resume/stop); CB `_check_position_size_limit` при 0 лотов не блокирует (§5.4, §5.8). `S8R-AUDIT-068`: CB `max_drawdown` для sandbox/real — equity = initial_capital + Σ pnl − Σ commission (включая вход открытых) + unrealized по свежей цене (close живой свечи сигнала или бар `ohlcv_cache` не старше 3 × таймфрейм; иначе unrealized не учитывается); пик в `trading_sessions.peak_equity` (миграция `f6a2c8e41d93`) фиксируется собственным commit при новом максимуме; `get_status` — тот же источник; paper-путь без изменений (§5.8). `S8R-AUDIT-069/100`: `StrategyService.delete` — 422 с перечнем при сессиях active/paused/suspended на любой версии, тело под `keyed_lock("strategy_delete", id)` (перечитывание, `StaleDataError` → 404); старт сессии берёт `strategy_delete` до `session_start_account`, запись аудита parity-override — под обоими локами (§5.4, `locks.py`). `S8R-AUDIT-025`: миграция `e9e5c919fbbf` — `live_trades.lot_size`; `derive_lot_size` — колонка, деривация только для старых строк; `RiskMonitor.position_lots` — одно количество для ордера, P&L и комиссии; `partially_filled` не терминал на входе и выходе; частичный выход (терминальный rejected/cancelled, 0 < исполнено < заявлено) → P&L по исполненному, пометка + пауза в той же транзакции, `position.mismatch` → `recovery_mismatch` (critical) (§5.4, §5.7). `S8R-AUDIT-061`: мультиплексор T-Invest — очередь команд на каждый стрим + sentinel в старую (gotcha-76), переподписка одним запросом, проверка ack `subscribe_candles_response` (отказ → маршрут снят, `on_error` подписчику), `UNAUTHENTICATED`/`PERMISSION_DENIED` → терминальное состояние по sha256-отпечатку токена (переживает eviction, сброс при переподключении счёта), `connection.lost` без повторов; eviction при нуле подписчиков; `stream_manager` — именованные держатели (`session:{id}`, `ws:{conn}`), лок на ключ, снятие сирот через 60 с (§5.6). `S8R-AUDIT-075`: реестр `_EXIT_IN_FLIGHT` удалён — «ордер в полёте» определяют колонки `LiveTrade`; `OrderInFlightError` → `order.error` один раз на эпизод; перед «не выполнено» статус перечитывается (§5.4). `S8R-AUDIT-074`: отправка ордера + commit id под `asyncio.shield` (`run_shielded_from_cancel`, grace 45 с), shutdown: quiesce → suspended → параллельный stop → recovery без порога → ожидание pending → отмена фоновых задач под общим дедлайном 45 с; `docker-compose.yml` `stop_grace_period: 60s` (§8.6). `S8R-AUDIT-099`: `BrokerService.delete_account` — проверка живых сессий и удаление под `keyed_lock("session_start_account", account_id)` (ключ старта sandbox/real-сессии, тело `_delete_account_locked`); `_create_session_locked` перепроверяет существование счёта под локом → `ValidationError`; порядок захвата не меняется (§5.4, `locks.py`). `S8R-AUDIT-030`: `TInvestAdapter._unary` — `asyncio.timeout(TINVEST_UNARY_TIMEOUT_SEC=10)` на всех унарных вызовах SDK; истечение → `BrokerTimeoutError(BrokerError)` from `TimeoutError` (`scope.expired()`), классификаторами 024 отказом не считается, на отправке — «исход неизвестен»; `_read_with_retry` повторяет только `70001`; пробы типа токена, прав и инструмента таймаут не глушат (§5.5). `S8R-AUDIT-101`: `create_account` — `keyed_lock("create_account", user_id)` вокруг SELECT+INSERT, `IntegrityError` → существующая запись, sandbox-пополнение только созданных этим вызовом; `UniqueConstraint uq_broker_accounts_user_type_account (user_id, broker_type, account_id)`, миграция `b8e4d17c9a52` со слиянием дублей (§3.2, `locks.py`). `S8R-AUDIT-024`: миграция `d4f1a9c2b7e0` — `live_trades.client_order_id`, `exit_client_order_id` (UUID4, пишутся до отправки, `PostOrder.order_id`); адаптер `get_order_status_by_client_id` (`GetOrderState`, `order_id_type=REQUEST`, ретраи только на чтении); единое правило `resolve_entry_order`/`order_state_verdict`/`read_order_state` для engine и runtime: состояние → отмена активного → перечитка, `lots_executed>0` = позиция на факт, отсутствие — подтверждённый `NOT_FOUND` не раньше `UNKNOWN_OUTCOME_CONFIRM_AGE_SEC=120` с от последней отправки, детерминированный gRPC-отказ → `failed` сразу, иначе `pending` + `order.error`; recovery входа под `close_trade`, exit-recovery с порогом, сверка пропускает «ответ потерян»; счёт удалён/деактивирован → `failed` (§5.4, §5.5). `S8R-AUDIT-078`: Grid Search — `GridWorkerSlots` (общий лимит воркеров `GRID_MAX_WORKERS_TOTAL`, 0 = cpu−1), отмена job: `threading.Event` → `terminate()+join()` Pool ≤ 2 с, исключение воркера — тоже `terminate()`; запись `_tasks` снимается до публикаций (§5.3). `S8R-AUDIT-055`: базовые URL SPA — модуль `frontend/src/api/baseUrl.ts` (`/api/v1`, `ws(s)://location.host`; `VITE_*` только override), dev-proxy Vite `/api` и `/ws` → :8000, `ARG VITE_API_BASE_URL` в Dockerfile, preflight проверяет `CORS_ORIGINS` (публичный origin обязателен — иначе `ws_origin_allowed` отбивает WS 403), CI-smoke «бандл без localhost:8000» (§8.10). `S8R-AUDIT-092`: `BackupService` — `Connection.backup` + `integrity_check`, restore temp → before_restore (Backup API) → checkpoint → `os.replace`, `fcntl.flock` на `BACKUP_DIR/.lock`, `settings.BACKUP_DIR`, CLI restore отказывает при живом сервере (код 3) (§8, backup). `S8R-AUDIT-091`: `detect_corporate_actions` — сплиты `GET /iss/statistics/engines/stock/splits.json` (все secid, `before/after` → `ratio_from/ratio_to`, `ex_date = record_date = tradedate`), купоны `GET /iss/securities/{secid}/bondization.json?iss.only=coupons` (облигации с открытой сделкой, горизонт 366 дн., `record_date = recorddate`); `process_split` — только позиции с `opened_at < 00:00 МСК tradedate`; `backtest/service.corporate_action_warning` → поле `warning` ответа бэктеста (§5.12, §5.3). `S8R-AUDIT-090`: `process_pending(today)` — `coalesce(record_date, ex_date) <= today_msk`; ex-date = предыдущий торговый день от даты реестра (календарь MOEX); право на выплату — `opened_at < ex-date(00:00 МСК) ≤ closed_at|NULL`; `rollback()` на сбое действия, скаляры новых действий снимаются до обработки (§5.12). `S8R-AUDIT-089`: `CorporateActionService._position_units` — дивиденд и купон на штуки (лоты × `lot_size`, `RiskMonitor.derive_lot_size` → fallback `ensure_lot_size`), НКД на бумагу (§5.12). `S8R-AUDIT-007`: `SessionRuntime._recover_orphan_pending_trades` — сбой `get_order_status`/`cancel_order` → сделка остаётся `pending` (`still_pending`), `failed` только по подтверждённым `rejected`/`cancelled` или после успешной отмены — зеркало `_recover_orphan_exit_orders` (§5.4). `S8R-AUDIT-093`: календарь MOEX — ISS `dailytable`, единый экземпляр сервиса, Сб/Вс неторговые до решения заказчика (§5.9). `S8R-AUDIT-001`: исполняется только код из IR валидированных блоков — fallback `generated_code`, legacy live-путь и эндпоинты `/sandbox/*` удалены; прокси с allow-list атрибутов вместо модулей в песочнице и бэктесте; сужены builtins (§5.11). `S8R-AUDIT-036`: traceback без локалов (`plain_traceback`), процессор `mask_secrets` → `***` (§7.7). `S8R-AUDIT-035`: `users.token_version` + claim `ver`, отзыв всех сессий при смене пароля и reuse refresh; миграция `c5e8b2a7f913` (§7.3). `S8R-AUDIT-034`: при `DEBUG=False` слабые `SECRET_KEY`/`ENCRYPTION_KEY` (dev-, плейсхолдер change-me, < 32 байт, < 8 различных символов) роняют старт; мастер-ключ проверяется в lifespan; preflight-скрипт синхронен (§7.3, §7.8). `S8R-AUDIT-033`: `POST /auth/setup` закрыт после первого пользователя (403, атомарный bootstrap `INSERT … WHERE NOT EXISTS`), категория лимитера `auth`; новый `POST /api/v1/admin/users` (admin); `SetupPage` проверяет `setup-status`, ссылка «Создать аккаунт» со страницы входа убрана (§4.1, п. о первом запуске). `S8R-AUDIT-002`: значения полей блоков валидируются по схеме `BLOCK_FIELD_SCHEMA` на входе API и в `parse_blocks`; кодоген берёт выражение `SOURCE` из словаря, пользовательские строки в код стратегии не интерполируются; бэктест и Grid Search отклоняют версию с невалидными блоками (422). `S8R-AUDIT-054`: `/api/v1/health` — readiness (`ok`/200 при БД `connected` и работающем планировщике, иначе `degraded`/503 с тем же телом), `/api/v1/health/live` — liveness (200 всегда); `tinvest_connected` — по живости `_stream_task` мультиплексора (gotcha-34), на статус не влияет; `version` — `app.__version__` (метаданные пакета); healthcheck compose/Dockerfile — readiness, `start_period` 60 с (§8.5). `S8R-AUDIT-044`: `app/broker/tinvest/errors.py` — `classify` → `FailureKind` (TRANSIENT / AUTH / NOT_FOUND / LINK «ответ не получен» / LOCAL «наш код или локальный отказ grpc»), `to_broker_error(order_action=…)` — текст по категории, `details` gRPC только в лог; классы `TransientBrokerError`/`AuthBrokerError`/`NotFoundBrokerError` (подклассы `BrokerError`); `_read_with_retry` — только чтения и только transient (UNAVAILABLE/DEADLINE — экспонента + джиттер, 70001 — 2 × 4 с, RESOURCE_EXHAUSTED — по `ratelimit_reset` + 1 с в пределах потолка), `OpenSandboxAccount`/`PostOrder`/`CancelOrder` без повторов; общий разбор кода `sandbox_recovery.grpc_code_name` для адаптера и движка; оценка попытки сверки — `read_worst_case_sec()` (§5.5.5). `S8R-AUDIT-051`: `docs/env_vars.md` — перечень всех настроек, тест сверки с `Settings` и подстановками compose; compose — `logging` json-file 10m×5, uvicorn `--timeout-graceful-shutdown 10` (drain + lifespan 45 с < `stop_grace_period` 60 с, gotcha-80); `scripts/start.sh` — preflight при `DEBUG≠true`, ротация `dev.log`; §8.2 приведён к фактическим именам (`SECRET_KEY`, `SMTP_FROM`, `BACKUP_KEEP_LAST`/`BACKUP_CRON`) (§8.2, §8.3). `S8R-AUDIT-065`: `app/common/ticker.py` — `Ticker` (`^[A-Z0-9._-]{1,20}$`, upper-case `BeforeValidator`, `TICKER_RE`) во всех схемах с тикером и GET-параметрах market_data (`Annotated[Ticker, Query()]`, gotcha-81); `app/common/export_safety.py` — `csv_safe_cell/row`, `xlsx_text`, `attachment_disposition` (RFC 5987) для CSV/PDF бэктеста и 3-НДФЛ (§4.11, §5.13). `S8R-AUDIT-066`: `telegram_webhook.py` — пользовательские поля в ответах с `parse_mode=HTML` (/start, /positions, /status, /balance, /close) через `_esc` = `_safe_format_event_text`; подписи `InlineKeyboardButton.text` не экранируются (не HTML). `S8R-AUDIT-097`: файловая SQLite — `NullPool`; `persist_with_retry` — sync/async единица работы, повтор при «locked» (OperationalError/PendingRollbackError, `is_database_locked`), откат до паузы; tax `generate_report`, corporate_actions detect (ISS вне повтора) / `_process_one` (идемпотентно), user_favorites add/remove — 3 попытки (§8.9). `S8R-AUDIT-045`: адаптеры `place_order/get_historical_candles/subscribe_candles(figi=)`; классовый `_figi_cache` TInvest удалён; движок — FIGI до отправки в `trade.figi` (запасной резолв, кэш `(счёт, тикер)` 15 мин), нерезолв → ордер не уходит; стрим и дозагрузка — FIGI из `ensure_figi`; `select_tradable_instrument` — приоритет режимов MOEX (§5.6). `S8R-AUDIT-046`: `get_balance` — `GetPortfolio` ‖ `GetPositions` (`asyncio.gather`, `_read_with_retry`), `available` = `TInvestMapper.rub_money_sum` (общий разбор с сайзингом 027, свежий запрос без кэша/резерва), `in_positions` = `portfolio_in_positions` (бумаги, ≥ 0), `blocked` — рубли под заявками; `GET /broker/accounts/balances` — суммы строками Decimal, `available/blocked/in_positions` могут быть `null` (сбой GetPositions). `S8R-AUDIT-081`: `app/strategy/status.py` (статусы, граф, STARTABLE, подсказки), 409 `strategy_status_not_startable` / `strategy_has_live_sessions` / `strategy_status_changed`, автоперевод статуса по сессиям и после бэктеста, миграция данных `2c7bd0443aa6` (§4.2). `S8R-AUDIT-047`: `TInvestAdapter._prepare_order` — отказы до PostOrder (`direction` не buy/sell, LIMIT без шага/цены, FIGI, подключение) → `OrderNotSentError` (окончательный отказ: вход `failed`, выход без пометки «в полёте»); LIMIT — `price` по шагу (buy вниз / sell вверх, кэш шага по FIGI 24 ч); статус `UNSPECIFIED`/неизвестный → `unknown` → опрос по 024 (`order_response_status`); `closing_order_side` (short закрывается BUY). `S8R-AUDIT-082`: фронт — `getApiErrorMessage` во всех catch стратегии, `saveVersion`/`updateStrategy` пробрасывают ошибку, уведомление в `doSave`; `block_parser` — предупреждение для неизвестного заголовка (формат с заголовками, `:` на конце допускается) и для `[I{N}]` на необъявленный индикатор. `S8R-AUDIT-062`: `create_account` — ротация ключа (перешифровка, пересчёт прав, `registration: created|key_updated|already_connected`, 422 при смене режима), после commit best-effort сброс кэшей 027/045 по счёту и `retire_token` (терминальная пометка отпечатка, если токен не держит ни одна активная запись любого пользователя) + `forget_rate_limiter`; `stream_manager.open_stream` переоткрывает нездоровую запись; `get_accounts(open_sandbox_if_empty=False)` — discover/test-connection без побочных эффектов. `S8R-AUDIT-073`: CB — одно событие `cb.triggered` (EVENT_MAP), перечитывание статуса под per-user CB-локом, пауза условным UPDATE, `session.paused` по каждой сессии, подписка уведомлений при resume, `trigger_value/limit_value` в журнале; тест сверки EVENT_MAP ↔ `event_bus.publish` (§2.4). `S8R-FIX-028`: `money_class(mode) = mode` — дневной лимит CB по трём классам paper / sandbox / real, `get_status` — поля по трём классам, `cb.triggered.money_class` = sandbox / real (§5.8). `S8R-AUDIT-083`: доставка уведомлений — изоляция каналов, повторы транзиентных ошибок с таймаутами, переполнение EventBus — error-лог + метрика (§5.7). `S8R-AUDIT-084`: `POST /notifications/telegram/test` без тела (`extra=forbid` → 422) — `send_test_telegram` серверным ботом в `TelegramLink` (409 без привязки, 503 без бота); DEV_MODE — `external_delivery_suppressed()` для `dispatch_external` и тестов; категория лимитера `notifications_test` 3/мин (`NOTIFICATIONS_TEST_RATE_LIMIT_PER_MINUTE`). `S8R-AUDIT-085`: мастер — готовность Telegram по `telegram/status.linked`; `notificationStore.updateSetting` — откат оптимистичного патча + `settingsError`; «Очистить все» с подтверждением; `TelegramNotifier.send(retry=False)` для теста. `S8R-AUDIT-063`: время ISS MSK → UTC, ISS-бары не подмешиваются к T-Invest, лот sandbox/real без ISS, миграция `6f3a063d1c2e` (§5.6). `S8R-FIX-027`: флаг `block_shorts`, проверка `_check_short_block` и событие `short_block` удалены, миграция `bbb9a0908e89`; поле в теле запроса CB-конфига игнорируется (§5.8). `S8R-AUDIT-086`: журнал аудита (`app/common/audit.py`) и outbox `pending_events` (`app/common/outbox.py`) с идемпотентным повтором при старте; runtime больше не публикует второй `order.placed` (§2.4, §3.11). `S8R-AUDIT-006`: исполнение стратегии бэктеста и Grid Search — в отдельном процессе с дедлайном и лимитом памяти (§5.11). `S8R-AUDIT-087`: `validate_ohlcv`/`OHLCVQualityReport` (warning бэктеста); CB `broker_rejection_streak` — `trading_sessions.consecutive_broker_rejections` (миграция `c7d2e5a19f84`), `is_broker_order_rejection`, `CB_PAUSE_WITHOUT_CLOSE`; частичное исполнение входа — `ENTRY_PARTIAL_FILL_TIMEOUT_SEC`, признак `entry_remainder_unsettled` (0 < filled_lots < volume_lots) в БД, `_settle_entry_remainder_locked` — одна функция для фоновой задачи, recovery и пути закрытия; `order.partial_fill` severity warning (§5.4, §5.8). `S8R-AUDIT-077`: `SharpeRatio` настраивается `metrics.sharpe_params(timeframe)` (единый для бэктеста и Grid Search; D/252, W/52, M/12, внутридневные — Minutes, factor = 805 × 252 / мин ТФ), `convertrate=True`, `annualize=True` (§5.3). `S8R-FIX-026`: `DELETE /strategy/{id}` — 204 (удалена) | 200 `{archived, status, detail}` (история сессий / FK-ссылки → `archived` условным UPDATE по правилам `status.py`) | 422 (живые сессии, бэктест в работе, уже в архиве с историей) (§4.2). `S8R-AUDIT-094`: `job_defaults` планировщика — `misfire_grace_time` 3600 с (5-минутная `refresh_backtest_jobs_rate` — 600 с), `coalesce=True`; jobstore memory (одноразовых задач нет); `schedule_t1_unlock`/`_execute_t1_unlock`/`unblock_settled_funds` удалены. Таблица джоб §5.9 расходится с кодом (есть `update_moex_sessions`, `cleanup_revoked_tokens`, которых нет) — сверить при следующем проходе ТЗ. `S8R-AUDIT-096`: `tax_lots.report_id` (миграция `7fe0fae4293c`), регенерация заменяет лоты года, файл отчёта по id, `_build_lots` (§5.13). `S8R-FIX-025`: `MOEXCalendarService.is_weekend_session_day/is_weekend_session_open` ([10:00, 19:00) MSK) — единая точка; `get_market_status` → `weekend_session`, `time_until_close` до 19:00, `/calendar.is_weekend_session_day`; входы — `trading_hours.entry_verdict` (маркер `WEEKEND_FLAG_REQUIRED`) + T-Invest `weekend_flag` (кэш 12 ч, сбой — 60 с, single-flight, `_find_tradable_instrument`); мультиплексор — `is_session_open` (§5.9). `S8R-AUDIT-095`: единая налоговая база обращающихся ЦБ, тип инструмента из `instruments.type` (пишется при резолве FIGI), эвристика — fallback с пометкой (§5.13). `S8R-FIX-029`: `app/broker/tinvest/tls_roots.ensure_grpc_trusted_roots()` — первый шаг lifespan, bundle «встроенные корни grpc + certs/russian_trusted_root_ca.pem» → `GRPC_DEFAULT_SSL_ROOTS_FILE_PATH` (если не задан оператором); SDK своих credentials не принимает (§5.5, гайд §6б). `S8R-AUDIT-005`: downgrade `a1b2c3d4e5f6` удаляет `idx_ai_user` только если он есть — полный `downgrade base` проходит. `S8R-AUDIT-016`: `app/common/ownership.py` — владелец в SQL, чужой id = 404 (§7.9). `S8R-AUDIT-017`: `POST /backtest/grid` — стратегия джобы = `strategy_id` проверенной версии, поле тела `strategy_id` необязательно и игнорируется (`grid_strategy_id_mismatch` в лог); CB PUT по чужой/несуществующей сессии → 404. `S8R-AUDIT-018`: `url_validator` — `not is_global` (+ multicast), `build_pinned_http_client` (резолв и проверка IP на `connect_tcp`, SNI/Host по имени; при прокси окружения — без подмены), `describe_provider_error` в verify, приватный `api_base_url` — только admin. `S8R-AUDIT-019`: удалены `strategy/code_generator.py` (генерация кода — только `ir_codegen.ir_to_backtrader_code`), `sandbox/executor.py`/`router.py`/`schemas.py`, `notification/dispatchers.py` (`dispatch_external` — `NotificationService`), `restore_sessions`, мёртвые методы `BacktestService`, персистентность `rate_limiter`. `S8R-AUDIT-023`: `POST /corporate-actions/detect` — только администратор (`require_admin`), ≤ 50 тикеров формата `Ticker`, категория лимитера `trading`. `S8R-AUDIT-031`: цена исполнения входа — три источника, ноль — не цена; вход без цены дольше окна подтверждения — `filled` без цены + расхождение и пауза; реконсиляция: без входа → `cancelled` вне `trades_closed`, комиссия — в `realized_pnl`; `quantize_money` (§5.4). Ревью MEDIUM (`S8R-REV-MQ`): комиссия при довыполнении частичного входа — итогом по кумулятивному факту ордера, без задвоения оценки и факта (§5.4). `S8R-AUDIT-042`: привязка шифртекста к записи (AAD-контекст, CLI `rebind-encrypted-secrets`), маскирование bot-token на итоговом тексте (structlog + фабрика LogRecord), отпечаток вместо префикса токена, реестры по отпечатку, права бэкапов 0600/0700 (§7.2, §7.7). `S8R-AUDIT-043`: политика нового пароля ≥ 12 + буква + цифра (`setup-status.password_policy`), вход без enumeration (dummy-хеш, единый 401, 423 удалён, атомарный счётчик неудач, сбой записи — единый 503), `cleanup_revoked_tokens` при старте и каждые 24 ч, `POST /users/me/disclaimer/accept` + гейт `disclaimer_required` на старт/resume (§4.1, §7.1, §5.9, §11.11). `S8R-AUDIT-059`: Swagger/OpenAPI только при DEBUG, `/health` — аноним `{status}`, отвергнутый токен 401, при недоступной БД — по подписи, readiness по таблице `users`; CORS — явные методы/заголовки, `*` в production — отказ старта (§7.6, §8.5). `S8R-AUDIT-048`: квоты T-Invest по сервисам, кэш справочника FIGI, «нет времени» = эпоха SDK, `UNKNOWN`-состояние операции, переоткрытие песочницы без гонок со сверкой позиций (§5.5.2). `S8R-AUDIT-064`: ISS — классификация ошибок и повторов, пагинация по cursor с пределом, графики при сбое ISS из кэша; стрим — битая свеча без переподключения, алерты вне цикла чтения (§5.5.2, §5.5.3). `S8R-AUDIT-088`: §4.0 — реестр всех 142 маршрутов из приложения (источник истины) и таблица соответствия устаревших путей подразделов 4.2–4.11. `S8R-AUDIT-088` (2): §3.0 — сверка схемы с моделями (`users`, `broker_accounts` приведены к моделям, прочие расхождения — таблицей); ссылки ФТ→ТЗ и гайд→ФТ/ТЗ на актуальные версии. `S8R-AUDIT-060`: коды закрытия WS доходят до браузера (accept → close; reason `logout`/`revoked`; 1013 при сбое БД), единый контроллер переподключения с баннером, эпоха клиентской сессии и порядок logout без reuse-detection, nginx — gzip и keepalive_timeout 4s (§7, §7.6, §4.12). `S8R-AUDIT-067`: диапазоны конфигурации CB, `invalid_fields`, нейтрализация опасных сохранённых значений, один снимок конфига на проверку, часы override, статус в рублях (§5.8). `S8R-FIX-054`: `GET /broker/accounts/balances` — при сбое опроса брокера `total`/`available`/`blocked`/`in_positions` = null (`total` стал nullable); экран «Счёт» — карточка `blocked`. `S8R-AUDIT-079`: прореживание кривой equity — равномерная сетка + экстремумы и бар максимальной просадки, Decimal только для отобранных точек. `S8R-AUDIT-098`: CLI `revoke_admin`, ошибки БД в CLI без traceback/SQL, избранное — инструменты ≤ 100 (условный INSERT), таймфрейм — значение enum, `/balance/history/full` — капитал активных сессий числом, `iss_path()` для всех путей ISS с тикером, запрет `.`/`..` в `Ticker`. `S8R-AUDIT-050`: SDK T-Invest по commit-hash, `backend/requirements.lock` (с хэшами) — источник установки в Docker и CI, base-образы по `@sha256`, RestrictedPython удалён из стека (§1 стек, §5.11), лицензии — `docs/licenses.md` репо кода. `S8R-AUDIT-049`: CI — `permissions: contents: read`, concurrency, actions по SHA, Dependabot. `S8R-REV-LQ`: сверка позиций после любой смены sandbox-счёта — единая функция, фантом только до момента переоткрытия, фоновая задача с повтором и паузой при неудаче (§5.5.2). `S8R-FIX-051`: монитор ценовых алертов подключён к стриму и сам подписывается на FIGI с алертами; срабатывание + уведомление + outbox одной транзакцией, доставка по каналам (§5.5.2, §5.7). `S8R-FIX-056`: канонический тикер (резолвер, сравнения без учёта регистра, дедупликация кэша свечей, канал стрима по канону, ISS futures), миграция `fb26c08f99c8` — индекс `upper(ticker)` (§5.6). `S8R-FIX-063` ч.2: CB учитывает сделки «P&L не определён» консервативной оценкой, `cb.unpriced_trade`, `SETTINGS_INHERIT_FROM` (§5.4, §5.8, §10). `S8R-FIX-052`: запрет удаления счёта/сессии с открытыми позициями, сверка с брокером перед закрытием у неуправляемой сессии, `confirm_closed_outside`, общий `ConflictError` (§4.0, §5.4). `S8R-FIX-069`: `POST /strategy/{id}/duplicate` (§4.0). `S8R-FIX-005`: флейк vitest (тесты). `S8R-REV-FBE` (находки ревью по трём осям): None вместо заглушек цены, период бэктеста до резолва тикера, ключ негативного кэша резолвера, рисунок графика — `known_canonical` без сети, `open_chart` с регистром, один модуль текстов pydantic (§5.6, §5.13, §11.15). `S8R-FIX-062/063/065/066`: P&L по исполненным лотам, правило цены исполнения без подмен (gotcha-33), уведомления закрытия через outbox у всех издателей, `stop_grace_period` 90 с, 422 без `input` (§2.4, §4, §5.4, §5.13). `S8R-FIX-067/068`: дозапрос типа в скане облигаций (§5.12), тесты гонок без порогов времени (§9). `S8R-FIX-043/059/061`: налоговый отчёт — ПФИ вне базы, дозапрос типов вне горячего пути (§5.13); `CandleSeries` столбцами (§5.3); сторож живого gRPC в тестах (§9). `S8R-FIX-038`: фронт редактора стратегии — откат пустой стратегии только при однозначном отказе, русские тексты 422 (§4 контракт ошибок). `S8R-FIX-031/033/034/046`: `/health` с пределом 3 с на реальном драйвере (§8.5), дефолт `AI_MODEL=claude-opus-5-5`, мёртвые лимиты AI удалены, `--reload` только при DEBUG (§8.1, §8.2), белый список импортов песочницы = модули исполнителя (§5.11), SSRF-барьер по встроенному IPv4 (§7). `S8R-FIX-042/041/037/039/048`: сброс серии нехватки средств на resume, P&L позиции дашборда × лот (общий `trade_lot_multiplier`), единый setter статуса сессии, журнал CB `Numeric(18,4)` NULL (миграция `752ae89f7ec2`), закрытие позиции без цены входа (§5.4, §5.10). `S8R-FIX-030/032/044/045/053/064`: единая таблица кодов gRPC (§5.5.5), песочница только в рублях (§4), снапшот WS без N+1 (§11.6 C1), чистка ISS-кэша без write-лока на сессии вызывающего — корень 045 (§5.6). `S8R-FIX-049`: комиссия по сторонам сделки, ноль брокера — не факт до окна начисления, досверщик в цикле восстановления, коррекция DailyStat, миграция `c04dc4f891a2` (§5.4, §5.13). `S8R-FIX-035/036/057`: rerun бэктеста проверяет параметры до записи и канонизирует тикер, правило периода (конец позже начала, ≥ 30 дней) во всех точках входа, повтор при `database is locked` — своя сессия на попытку (`persist_with_retry_fresh`), векторная сборка кривой и бенчмарка (§4, §5.6). `S8R-FIX-055`: фронт — `RiskSettingsPage` (`/settings?tab=risk`), проверка `utils/riskLimits.ts` (зеркало `risk_limits.py`), PUT — весь объект (§5.8). `S8R-FIX-074`: фронт — `stores/accountStore.ts` переведён на посекционные `{balances,positions,operations,tax}Loading/Error` (был один `error` на шесть загрузок) и на `utils/apiError.ts::getApiErrorMessage`; `pages/AccountPage.tsx` держит полноэкранный Alert только за `balancesError`, ошибки операций и налоговых отчётов рисуются над своими таблицами. Тем же проходом на хелпер переведены оставшиеся 9 файлов с `err.message` (§4, §5.8). `S8R-FIX-073`: `BalanceHistoryPoint.active_capital` (бэкенд `app/account/service.py`) + ветка «Нет активных сессий» в `BalanceWidget.tsx` (§4). `S8R-FIX-075`: `components/layout/StatusFooter.tsx` опрашивает `/health` в своём 60-секундном цикле и ведёт три состояния плашки брокера (`не подключён` / `на связи` при `tinvest_connected === true` / `настроен`); `hasBrokerConnection` из `settingsStore` остался признаком «счёт настроен», но перестал быть источником зелёного (§6). `S8R-FIX-076`: проп `loadFailed` у `components/account/OperationsTable.tsx`, `AccountPage` передаёт `operationsError !== null` (§4). **S8R-FIX-071:** `live_trades.close_reason` (миграция `fd3492233bc0`, nullable String(40)) — причина закрытия не терминалом (`closed_outside_terminal`, `closed_outside_terminal_by_user`, `sandbox_account_reopened`); входит в `UNDETERMINED_CLOSE_REASONS` и предикат `is_pnl_undetermined`, поэтому `_close_trade_without_order` пишет `pnl = NULL` без подстановки `exit_price = entry_price` и без нулевого `pnl` в `DailyStat` (остаётся `trades_closed +1` и `realized_pnl = −commission`); выборка `unpriced_cb_losses` читает колонку, иначе оценка CB её не видит; в payload `trade.closed` — `PNL_UNDETERMINED_OUTSIDE_TEXT`; `TradeResponse.close_reason` и `LiveTrade.close_reason` на фронте (подпись журнала). **S8R-FIX-070:** `POST /trading/sessions/{id}/reconcile` → `SessionRuntime.reconcile_session_with_broker` (тот же `_reconcile_broker_positions` с `pause_from=()` и явным исходом `ReconcileOutcome`, бюджет `POSITION_RECONCILE_TIMEOUT_SEC`); `BrokerService._assert_account_not_in_use` держит счёт при `position_mismatch_at IS NOT NULL` у sandbox/real-сессии, в `sessions[]` — `position_mismatch`. **Фронтенд переведён на правила React-компилятора** (S8R-FIX-080): `eslint-plugin-react-hooks` поднят до `^7.1.1`, `eslint-plugin-react-refresh` — до `^0.5.7`, версии больше не прибиты точно, гейт `pnpm lint` зелёный. 20 мест в 12 файлах приведены к трём формам: выборка данных отделена от состояния (загрузчик чистый, присваивания — в теле эффекта после `await`, добавлена отмена по размонтированию), сброс состояния при смене пропа делается сравнением с предыдущим значением во время рендера (по СОДЕРЖИМОМУ, если источник не обещает стабильной ссылки — например массив из селектора стора), обращение к действиям сторов из эффектов — через `useEffectEvent`, а не через зависимости (нестабильная ссылка иначе перезапускает эффект на каждый рендер). `strategyStore.fetchStrategy` теперь возвращает `Promise<StrategyDetail | null>` — редактор наполняет форму ответом вызова, с откатом на кэш стора при сбое и сверкой `id`. В `CandlestickChart` ref'ы «последнего значения» (`candlesRef`, `sequentialModeRef`, `onChartReadyRef`, `rebuildMarkersRef`) пишутся в `useLayoutEffect` вместо рендера, а вызов `useTradeMarkers` поднят выше эффектов (требование `react-hooks/immutability`) — из-за смены порядка эффектов первые итерации ожидания серии в `useTradeMarkers` сделаны короткими, иначе маркеры сделок опаздывали на 250 мс. **Хвосты аудита, 2026-10-13.** `S8R-FIX-019`: §8.6 — `restore_all` под общим дедлайном `RESTORE_TOTAL_BUDGET_SEC = 45` (сумма частных дедлайнов превышала `start_period` healthcheck). `S8R-FIX-020`: §8.6 — периодическая сверка греет портфели параллельно через `_prefetch_restore_portfolios` с `log_prefix`, отбор сессий — `_sessions_to_reconcile()` без сети. `S8R-FIX-018`: §8.6 — `_run_shutdown_recovery` передаёт `RESTORE_PAUSE_FROM`; постоянный отказ старта без владельца → `stopped` (`paused` держал пару «счёт+тикер» навсегда); уведомления о сессии сведены к одному `SessionRuntime._notify_session` (резолв владельца, запись, обработка сбоя доставки) — три копии расходились в обработке ошибок, и у самого важного пути, расхождения позиций, её не было вовсе. `S8R-FIX-021`: §7.2 — привязка шифртекста к записи закрыта ещё S8R-AUDIT-042; инвариант «ни одного `encrypt()` без `context=` в `app/`» закреплён тестом, состояние рабочей БД проверено на копии (`rebind-encrypted-secrets` → nothing to rebind). `S8R-FIX-013`/`S8R-FIX-015`: §5.8 — пик equity поднимается на каждой закрытой свече (`update_peak_on_bar`, без сети и без CB-лока), состав `current_equity` для paper включает нереализованный P&L; §3.17 — `daily_stats.peak_equity` удалена миграцией `a7c1d9e2b4f8`. `S8R-FIX-009`: §11.4 — `SHUTDOWN_WAIT_SEC = 10`; §11.5 — grid без слотов остаётся `queued` с событием `waiting`. `S8R-FIX-022`: §5.2.4 — периоды, целиком накрытые обрывом, собираются из дозагруженных минут, предел `GAP_REPUBLISH_MAX_BARS = 60`. `S8R-FIX-008`: §5.9 — `detect_corporate_actions` каждые 6 часов, текст уведомления с датой реестра и ex-date. `S8R-FIX-006`: §3 — `live_trades.nkd_entry/nkd_exit` — НКД на ОДНУ облигацию, ₽. `S8R-FIX-003`: §8.4 — описание логирования приведено к факту (один поток в stdout, файл и ротация снаружи процесса, права 0600). `S8R-FIX-017`: §4.12/§7.4 — WS-аутентификация cookie при upgrade, один декодер access-токена на три входа. `S8R-FIX-021`: `alembic/env.py` нормализует `DATABASE_URL` той же функцией, что настройки. `S8R-FIX-023`/`S8R-FIX-024`/`S8R-FIX-002`: §9.1 — окружение тестового движка (FK, `GRPC_ENABLE_FORK_SUPPORT`, `faulthandler_timeout`, CLI по эффекту). `S8R-FIX-079`: §1.2 — TanStack Table 9.3+ (явные фичи, фабрики моделей строк, `table.state`, `getAllCells`). Находка прогона: §11.4 — провал бэктеста доходит до job'а возвращаемым значением `_run_backtest_task`, а не перечитыванием `backtests.status` (в зазоре записи провал превращался в `done` с ложным «Бэктест завершён»). |
| 2.9 | 2026-08-10 | S8R — гигиена перед деплоем | **§5.2.2 (парсер шаблона), §5.6 (рыночные данные), §5.4 (движок), §9.** (1) `S8R-FIGI-NO-NEGATIVE-CACHE` (Q4=a): `_fetch_figi_from_tinvest` возвращает `FigiLookup(figi, source_answered)` вместо `str \| None`. Флаг разделяет «провайдер ответил, инструмента нет» и «источник молчит» (нет счёта с токеном, `ImportError` SDK, ошибка расшифровки, `asyncio.TimeoutError`, любое исключение gRPC) — кэшируется **только первое**. Отметки промахов — модульный `_figi_negative_cache: dict[str, datetime]` с `FIGI_NEGATIVE_TTL = 15 мин`; кэш процессный, а не на экземпляре, потому что `process_signal` создаёт `MarketDataService(self.db)` заново на каждый сигнал. TTL читается при проверке (как `CbrRateService._is_fresh`), протухшие записи вычищаются при записи новой, успешный резолв снимает отметку. Публичная `reset_figi_negative_cache()` + autouse-фикстура в `tests/conftest.py`: без сброса состояние текло бы между тестами и падение зависело бы от их порядка. В БД отрицательный результат не пишется — колонка `instruments.figi` хранит либо настоящий идентификатор, либо ничего; миграций нет (Q8=a). (2) `S8R-PAPER-TARIFF-UNREACHABLE-FROM-UI` (Q5=a): наследование тарифа существовало в `start_session` с 2026-08-04, но включалось условием «ставка сессии == 0 И передан `broker_account_id`», а `buildSessionStartRequest` слал счёт только для `sandbox`/`real`. В черновике появилось поле `commission_source: 'manual' \| 'account'`; при `account` тело запроса содержит `broker_account_id` **и** `commission_pct: 0` — ровно та комбинация, которую backend распознаёт. Backend не менялся. Инварианты, закреплённые тестами (`test_paper_tariff_from_ui.py`): `broker_account_id` у paper-сессии не делает её брокерской — `PaperPortfolio` создаётся, запрет `S8R-DUPLICATE-SESSION-PER-INSTRUMENT` смотрит на `mode.in_(('sandbox','real'))`, ставка фиксируется в момент запуска. Для `sandbox`/`real` `commission_pct` по-прежнему принудительно 0. (3) `S7R-BG-BACKTEST-AUTOCOLLAPSE` (Q6=b): `AUTOCOLLAPSE_DELAY_MS = 30_000` в `BackgroundBacktestsBadge`; таймер на каждую терминальную карточку, задержка считается от `finished_at` (не от монтирования: `items` меняется на каждом тике прогресса соседнего джоба, и отсчёт «с нуля» не дошёл бы до конца; попутно карточка, завершившаяся до перезагрузки страницы, убирается сразу). `error` исключён намеренно — текст причины больше нигде не виден; при открытом popover отсчёт не идёт. (4) `S5R-BLOCKLY-MODE-B-MODAL` / `-CHECK` (Q2=a, Q3=a) — см. §5.2.2: кнопка в редакторе, шаблон приведён к семи секциям парсера, добавлен парсящийся пример, контракт закреплён тестом синхронизации `.tsx` ↔ `template_format_rules.py`; два `test.skip` в `e2e/blockly.spec.ts` заменены рабочими спеками. (5) `S7R-E2E-7.9-MISSING` (Q7=b): сценарий 7.9 закрыт pytest-интеграционным тестом (`tests/test_backup/test_cli_restore_roundtrip.py`), а не Playwright-спеком — сценарий целиком серверный (запуск `python -m app.cli.backup` через `subprocess` и чтение файла БД), браузер в нём не участвует. Проверяется результат, а не код возврата: round-trip восстановления потерянной строки, `PRAGMA integrity_check`, содержимое pre-restore-снимка, попадание в бэкап транзакции, оставшейся в WAL (gotcha-19), ротация 10 → `--keep 7` по `mtime` и неприкосновенность рабочей `data/terminal.db`. |
| 2.8 | 2026-08-10 | S8R — доведение до деплоя | **(1) `S8R-FRONTEND-BUILD-BROKEN` — production-сборка фронтенда была сломана, а гейт этого не видел.** `frontend/Dockerfile` собирает образ через `pnpm build` (= `tsc -b && vite build`); `tsc -b` падал со 102 ошибками типов в 45 файлах. Гейт «tsc 0» (в том числе шаг CI `pnpm tsc --noEmit`) не проверял ни одного файла: корневой `tsconfig.json` — solution-файл с `"files": []`, и без `-b` команда завершается мгновенно с кодом 0. Все ошибки устранены; `package.json` получил скрипт `typecheck: tsc -b`, в CI шаг заменён на `pnpm typecheck` и добавлен `pnpm build`. `fancy-canvas` объявлена прямой зависимостью (импортируется в примитивах графика). **(2) Новый endpoint `GET /api/v1/market-data/usd-rate`** → `{rate, as_of, stale}`. Источник — `https://www.cbr.ru/scripts/XML_daily.asp`, кэш 6 ч в процессе + `asyncio.Lock` (один запрос к ЦБ на всех). Недоступность ЦБ: при наличии кэша — `stale=true`, иначе `503`. Разбор XML — `xml.etree` с лимитом тела 1 МБ и инлайн-`nosec B314` (фиксированный HTTPS-хост, внешние сущности CPython не резолвит). **(3) Порядок захвата локов дополнен** (`app/common/locks.py`): `session_lifecycle` (по `session_id`) — **самый внешний**, под ним `close_trade` (по возрастанию `trade_id`) → `sandbox_reconcile_account`; `session_start_account` — отдельная ветвь. `pause/stop/resume_session` сериализованы по сессии, `_submit_order_to_broker` берёт `close_trade` своей сделки. **(4) `BrokerService.delete_account`** отклоняет удаление счёта при сессиях в статусах `active`/`paused`/`suspended` (FK `ondelete='SET NULL'` иначе обнулял `broker_account_id` у работающих сессий). **(5) Единая сборка тела запроса на запуск сессии** — `frontend/src/components/trading/sessionRequest.ts`; ручная сборка в «Запустить заново» была источником расхождения контракта. Приведены к реальности типы `LiveTrade.direction` (`buy`/`sell`), `Backtest` (поля списочного ответа) и `tradingApi.getSession` (`SessionDetailResponse`). Миграций цикл не добавляет, alembic head прежний — `a7b8c9d0e1f2`. |
| 2.7 | 2026-08-06 | Sprint_8_Review — один инструмент, одна сессия на счёте | **§5.4 (движок), §9.** (1) `S8R-DUPLICATE-SESSION-PER-INSTRUMENT`: `TradingSessionManager._assert_no_conflicting_session` — единая точка запрета для `start_session` и `resume_session`. Единица конфликта — пара «`broker_account_id` + `ticker`» (Q2=a): именно счёт определяет общий пул бумаг у брокера, а таймфрейм на пул не влияет. Область — режимы `sandbox`/`real` (Q1=a): механика песочницы идентична бою, отличается только цена ошибки. Блокирующие статусы — `BLOCKING_SESSION_STATUSES = (active, paused, suspended)` (Q3=a); `suspended` включён потому, что `restore_all` вернёт такую сессию в `active` сам. Выбор виновника — `ORDER BY CASE(status) , id LIMIT 1` + `.scalars().first()`, а не `scalar_one_or_none()`: конфликтующих кандидатов может быть несколько (исходное состояние БД заказчика — сессии #4/#6), и `MultipleResultsFound` дал бы 500 вместо понятного отказа. Схема БД **не** меняется: `UniqueConstraint` на «счёт + тикер» запретил бы и закрытые исторические сессии, которых у заказчика уже две (Q5=a). (2) **Атомарность** — новый ключ `keyed_lock("session_start_account", broker_account_id)`. Без него проверка была `SELECT`-затем-`INSERT` без сериализации: между ними есть `await` на реальном I/O, и два одновременных «Запустить» проходили оба (воспроизведено тестом: в БД оказывались две сессии). Место в документированном порядке захвата (`app/common/locks.py`) — **самый внешний**: путь запуска берёт только этот ключ, а `close_trade` и `sandbox_reconcile_account` внутри секции не запрашиваются, потому что под локом идёт только работа с БД. Подписка на стрим (`SessionRuntime.start`) вынесена из-под лока в `_announce_started_session`: `locks.py` разрешает сеть под локом лишь с таймаутом на самом вызове, а зависший стрим запер бы запуск всех сессий счёта. (3) `SessionRuntime.restore_all` дедуплицирует пару «счёт + тикер» за проход: пред-проход закрепляет её за уже `active`-сессиями, чтобы `suspended`, попавшаяся раньше по порядку, не забрала пару у сессии, которая в БД уже числится работающей. Пропуск логируется (`session_restore_skip_duplicate_instrument`), статус **не** переписывается — восстановление не место для ревизии пользовательских данных. (4) `S8R-RECONCILE-NO-ACCOUNT-DEDUP`: `_reconcile_broker_positions` считает `our_units` по **всем** sandbox/real-сессиям счёта (JOIN `LiveTrade` → `TradingSession` по `broker_account_id`), независимо от статуса сессии — брокеру всё равно, `stopped` у нас сессия или нет, незакрытая сделка занимает бумаги. Сравниваются только FIGI **этой** сессии; обратная проверка ограничена её тикером, иначе карточка сессии по SBER показывала бы расхождение соседа по GAZP. Портфель кэшируется на **один проход** фоновой сверки (`portfolio_cache`, ключ `broker_account_id`) — 10 сессий счёта давали 10 адаптеров и 10 одинаковых запросов; кэш дольше прохода означал бы сверку с устаревшим состоянием счёта. (5) **§9** — тест гонки на файловой БД с `NullPool` (gotcha-48), рандеву **до** захвата лока (gotcha-59); защита проверена мутацией: снятие `async with lock` даёт две сессии и роняет тест. (6) Фронт: `TradingSession.status` в `api/types.ts` дополнен `suspended` — backend отдавал его и раньше, в типе его не было. (7) `S8R-PAPER-CONFLICT-BY-VERSION-NOT-TIMEFRAME` (решение заказчика, вечер 2026-08-06): из `_create_session_locked` удалена унаследованная проверка «одна активная сессия на пару (`strategy_version_id`, `ticker`)» — она не учитывала `timeframe`, из-за чего SBER на 1h и SBER на D от одной версии не запускались вместе. Paper-режим ограничений на повтор инструмента больше не несёт **вообще**: у каждой сессии свой `PaperPortfolio`, брокерских ордеров нет, общего пула бумаг нет — ни одно из оснований `S8R-DUPLICATE-SESSION-PER-INSTRUMENT` к нему не применимо. Запрет для `sandbox`/`real` не затронут и покрыт отдельным классом тестов (`TestSandboxRestrictionUnaffected`). |
| 2.6 | 2026-08-06 | Sprint_8_Review — FIGI сделок, гонка реконсиляций и хвост сверки | **§5.4 (движок), §5.6 (рыночные данные), §8.10 (миграции), §9.** (1) `S8R-SANDBOX-TRADES-FAKE-FIGI`: `MarketDataService.ensure_figi(ticker)` — единая точка резолва FIGI: кэш `instruments.figi` → T-Invest `find_instrument` (отбор через `select_tradable_instrument`, E2) → апсерт в кэш через `_persist_instrument_cache` (BE-MKT-02: injected-сессия не коммитится). `is_paper_figi` трактует `paper_*` как отсутствующее значение и в кэше, и в ответе источника. Резолв не бросает на недоступности источника, никогда не возвращает `paper_*` и ограничен `FIGI_LOOKUP_TIMEOUT_SEC = 10` (в отличие от лотности он не закрыт TTL, и при неудачном резолве в сеть уходит каждый сигнал по тикеру — из обработчика свечи). `OrderManager.process_signal` для `sandbox`/`real` использует `ensure_figi` вместо прямого чтения `Instrument.figi`; paper-ветка не изменилась. Миграция данных `a7b8c9d0e1f2`: `paper_%` → FIGI справочника либо `NULL` у сделок сессий `LOWER(mode) IN ('sandbox','real')`, paper-сделки не тронуты, `paper_*` в `instruments.figi` зануляется; схема не меняется, `downgrade` данные намеренно не восстанавливает. (2) `S8R-RECONCILE-LOCK-DIVERGENCE` (Q2=b): в `app/common/locks.py` зафиксирован **глобальный порядок захвата** — `close_trade` (по возрастанию `trade_id`) → `sandbox_reconcile_account`. `OrderManager._reconcile_positions_after_reopen` выбирает кандидатов (`_reconcile_candidate_ids`, только идентификаторы — чтобы не засорять identity map), захватывает их локи под единым дедлайном `RECONCILE_LOCK_TIMEOUT_SEC = 30` (иначе одна сделка, застрявшая на вызове брокера, заперла бы все сделки счёта вместе с SL/TP и force-close CB), берёт лок счёта и перечитывает набор; появление новой сделки — перезаход (`RECONCILE_LOCK_ATTEMPTS = 3`), на последней попытке обрабатывается залоченное подмножество. `_reconcile_locked` принимает `allowed_ids` и перечитывает статус точечным `db.refresh` (не `populate_existing` — gotcha-58), пропуская исчезнувшие строки по `InvalidRequestError`. (3) `S8R-RECONCILE-NOT-PERIODIC` (Q3=a): `SessionRuntime._reconcile_active_sessions` в `_periodic_recovery_loop` на собственном интервале `POSITION_RECONCILE_INTERVAL_SEC = 900` (счётчик, а не вторая задача) с бюджетом `POSITION_RECONCILE_TIMEOUT_SEC = 60` на сессию — цикл несёт также watchdog стримов и доучёт ордеров, и зависший gRPC подвесил бы их молча. Гейт обхода: только `status='filled'` **или** непогашенная пометка расхождения. (4) `_reconcile_broker_positions`: пауза сессии только при `broker_units < our_units` (портфель брокера суммарный по счёту, `our_units` — по одной сессии; «у брокера больше» штатно при нескольких сессиях счёта или ручной покупке); сделки с заполненным `exit_broker_order_id` из сверки исключены (встречный ордер в полёте — не «позиция исчезла»); остановка **работавшей** сессии уведомляет всегда, дедупликация текста сохранена для уже приостановленной. (5) **§9.** Stack Gotchas 57 → **58** (`populate_existing` затягивает `lazy="selectin"`-связи и замораживает их в identity map). |
| 2.5 | 2026-08-05 | Sprint_8_Review — сверка с брокером как источником правды | **§5.4 (движок), §8.10 (миграции).** (1) `S8R-EXIT-ORDER-NOT-TRACKED`: колонки `live_trades.exit_broker_order_id` и `exit_order_placed_at` (ревизия `d4e5f6a7b8c9`). `OrderManager` пишет идентификатор встречного ордера **до** опроса статуса, отдельным commit'ом; `SessionRuntime._recover_orphan_exit_orders` при старте и в периодическом цикле опрашивает статус и доводит сделку до `closed` через общий `_apply_close_and_settle` (единая точка учёта P&L, комиссии, `DailyStat` и события `trade.closed`). Заполненная колонка при `status='filled'` блокирует повторную отправку встречного ордера между запусками процесса — реестр `_EXIT_IN_FLIGHT` остаётся защитой внутри процесса. Пометка снимается только по подтверждению брокера (`rejected`/`cancelled`/успешная отмена/`NOT_FOUND` ордера) под локом сделки и со сверкой идентификатора. (2) `S8R-SANDBOX-POSITIONS-NOT-RECONCILED`: колонки `trading_sessions.position_mismatch_at` и `position_mismatch_note` (ревизия `e5f6a7b8c9d0`), `SessionRuntime._reconcile_broker_positions` для `sandbox` и `real`. Сравнение — в **штуках** (`filled_lots × derive_lot_size`) и в `Decimal`, с агрегацией по FIGI и исключением валютных позиций (gotcha-57). Пустой/неполный ответ портфеля перепрашивается по признаку «нашей бумаги нет в ответе» (gotcha-56); ошибка брокера расхождением не считается. Расхождение не изменяет данные сделок. (3) `S8R-TINVEST-SANDBOX-FLAKY-70001`: `is_sandbox_internal_error` + `TInvestAdapter._read_with_retry` для чтений (портфель, счета, статус ордера — короткий повтор в горячем пути); размещение ордера не повторяется, но отдаёт человекочитаемый текст. |
| 2.4 | 2026-08-05 | Sprint_8_Review — SL/TP у брокера, гонки и уведомления | **§5.4 (движок).** `RiskMonitor.check_sl_tp` перестал быть единственным исполнителем закрытия: в режимах `sandbox`/`real` он только принимает решение «уровень пробит», а исполнение делегирует `OrderManager.close_position` (`_close_via_broker`) — встречный market-ордер, опрос статуса, фактические цена и комиссия. До этого модуль не отправлял брокеру ничего ни в одном режиме, хотя выборку сделок делал без фильтра по режиму: терминал считал позицию закрытой, а у брокера она оставалась открытой (`S8R-SLTP-SANDBOX-NO-BROKER-ORDER`). Инвариант: **сделка помечается закрытой только если встречный ордер ушёл**; при отказе брокера или вне торговых часов состояние не меняется, публикуется `order.error`, а пользователю уходит уведомление. Повторной отправки ордера нет намеренно — после `PLACED` повтор создал бы вторую позицию в обратную сторону. Вместе с этим из `check_sl_tp` удалён расчёт **оценочной** комиссии выхода для sandbox/real: её теперь пишет `_record_broker_commission` фактом брокера, и прежний расчёт удерживал бы комиссию дважды. Учёт закрытия (`DailyStat`, `realized_pnl`, событие) в этих режимах ведёт только `close_position` — второй копии нет. Paper-путь не изменился. **§5.4 (гонки).** `OrderManager.close_position` сериализован `keyed_lock("close_trade", trade_id)`; тело вынесено в `_close_position_locked`. Ключ — сделка, а не сессия: закрытие ждёт ответа брокера, и лок на сессию застопорил бы её остальные позиции. Сетевые вызовы под локом ограничены `MARKET_PLACED_POLL_RETRIES × MARKET_PLACED_POLL_INTERVAL_SEC`. Закрыт `S8R-CLOSE-POSITION-NO-LOCK`: двойной клик «Закрыть», совпадение ручного закрытия с SL/TP или с exit-сигналом отправляли брокеру два встречных ордера, дважды кредитовали `PaperPortfolio` и дважды удерживали комиссию. **§3 (общее).** Реестр `app/common/locks.py` переведён на `weakref.WeakValueDictionary`: пространство ключей `trade_id` неограниченно (номера не переиспользуются), и вечный dict копил бы по `asyncio.Lock` на каждую закрытую за всё время сделку. **§4 (события).** Payload `trade.closed` собирает единый `trading.engine.trade_closed_payload` — все четыре publisher'а (`close_position`, `RiskMonitor`, реконсиляция sandbox, рантайм) отдают `net_pnl`, `commission`, `commission_note` наравне с валовым `pnl`; шаблон `EVENT_MAP["trade.closed"]` рендерит чистый результат и комиссию отдельно (`S8R-TELEGRAM-TRADE-CLOSED-GROSS-PNL`). Единая точка нужна потому, что `_SafeDict` в диспетчере не роняет рендер на забытом ключе — он подставил бы буквальное `{net_pnl}` прямо в сообщение пользователю. **§9.** Stack Gotchas 54 → **56**: TLS-корень «Russian Trusted Root CA» отсутствует у `grpcio` (55) и спорадический `INTERNAL 70001` песочницы T-Invest (56). |
| 2.3 | 2026-08-04 | Sprint_8_Review — хвост комиссии, налоговый модуль и стабильность E2E | **§5.13 (налоговый модуль)** — `TaxReportService._build_fifo_queue` считает результат в **штуках**: множитель штук/лот деривируется из `volume_rub` входа через `RiskMonitor.derive_lot_size` — тем же способом, что в торговом движке, и без обращения к сети. Fallback для записей с пустым `volume_rub` — карта «тикер → lot_size», заполняемая до построения очереди (`MarketDataService.ensure_lot_size`). Контракт корректности — **сходимость с движком**: `TaxLot.realized_pnl == LiveTrade.pnl − LiveTrade.commission` (для облигаций плюс разница НКД, которую движок не считает). Комиссия делится на половинки так, что их сумма точно равна удержанной (округляется одна сторона, вторая — вычитанием); симметричное `round(total/2)` завышало колонку «Комиссии» на копейку при нечётном числе копеек. **§5.4 (движок)** — комиссия sandbox/real: источник — `OrderState.executed_commission` из того же `GetOrderState`, которым уточняется цена исполнения (разбор `GetOperations` не требуется — комиссия там уже привязана к ордеру); при отсутствии факта считается оценка по тарифу, источник фиксируется в `live_trades.commission_source` (`broker` / `estimated`, NULL — удержаний нет). Денежный учёт sandbox/real **не затрагивается**: баланс ведёт брокер, `PaperPortfolio` для таких сессий не существует — комиссия только записывается в сделку. Ошибка разбора комиссии не роняет весь `OrderStatus` (иначе терялись бы `average_price` и `lots_executed`). **§5.4/§3** — ставка комиссии резолвится `resolve_commission_rate(session, broker_account)`: явная ставка сессии главнее, иначе тариф счёта (`broker_accounts.commission_pct`); значение фиксируется на сессии в момент запуска, чтобы смена тарифа не переписывала деньги идущей сессии. **§7 (rate limit)** — ключ лимитера читает JWT **и из cookie**, и из заголовка: после перехода на Model A заголовка нет ни у одного браузерного запроса, ключ деградировал до IP, и лимит категории (200 req/min) стал общим на всех пользователей; за обратным прокси, где адрес один, он выбирался бы ещё быстрее. Ключ пользователя префиксуется (`user:<id>`), чтобы не пересекаться с пространством IP. **§8** — миграция `c3d4e5f6a7b8`: `live_trades.commission_source`, `broker_accounts.commission_pct` (`server_default='0'`), `tax_lots.trade_mode`; идемпотентна, обратима, round-trip покрыт тестом, `deployment_guide.md` §7 обновлён синхронно. **§9** — E2E-спека `s6-notifications` больше не проверяет флаг сразу после `click()` (гонка с уходом запроса; на CI маскировалась `retries: 2`), а ждёт сам запрос; спека `auth-hardening` проверяет статус `/auth/me` до использования тела. |
| 2.2 | 2026-08-03 | Sprint_8_Review — зелёный CI, наблюдаемость и остаток находок | **§4** — заведена вторая метрика `trading.signal_to_order_full`: полное окно «сигнал → ордер», включающее ожидание за per-user локом Circuit Breaker. Старая `trading.signal_to_order` оставлена как есть, чтобы исторические замеры блока F оставались сопоставимыми; на `/admin/metrics` полное окно выводится **первым** — судить о бюджете «p95 < 500 мс» следует по нему. Замер: при 40 сессиях старая метрика показывает 7,2 / 9,0 мс, новая — 407,2 / 764,4 мс при фактических 406,9 / 764,4 мс. Инструмент замера — контекстный менеджер `app/common/observability.py::timed_block` (декоратором такую границу задать нельзя: окно начинается в середине `SessionRuntime._handle_candle`). **§3/§5** — бумажная торговля удерживает комиссию по ставке сессии (`trading_sessions.commission_pct`, миграция `b7c8d9e0f1a2`, `server_default='0'`): формула та же, что у `cerebro.broker.setcommission` — процент от оборота каждой стороны. Инвариант денежного учёта уточнён: `Δequity == trade.pnl − trade.commission` (при нулевой ставке вырождается в прежний `Δequity == trade.pnl`); `trade.pnl` остаётся **gross**, как `trade.pnl` у backtrader. Колонка `live_trades.commission` впервые получила писателя. **§5** — дневной отчёт включает сессии с открытой позицией, даже если сделок за день не было (отдельный запрос по открытым сделкам, `setdefault` по `session.id` защищает от задваивания). **§9** — исходящий HTTP из unit-тестов перекрыт autouse-фикстурой `tests/conftest.py::_block_outbound_http`: 22 теста в четырёх файлах ходили в живой MOEX ISS через резолв лотности, что давало плавающие падения CI без изменений кода. |
| 2.1 | 2026-08-03 | Sprint_8_Review — сведение веток и остаток замечаний | §5.4 — состав Trading Engine приведён к фактическому (`TradingSessionManager`, `SessionRuntime`, `SignalProcessor`, `OrderManager`, `RiskMonitor`, `PaperTradingEngine`); `PositionTracker` убран из описания компонентов и из схемы потока сигнала — класс удалён 2026-07-31, но §5.4 об этом ещё не знал. §5 — нереализованный P&L **нигде не хранится**: считается на лету `app/trading/unrealized.py::unrealized_pnl_by_session`, и все три потребителя (Circuit Breaker, `TradingService.get_positions`, `SchedulerService.send_daily_stats`) обязаны давать одно число на одних данных; дневной отчёт больше не считает его своей копией формулы без множителя штук/лот. §8 — колонка `daily_stats.unrealized_pnl` удалена миграцией `e2f3a4b5c6d7` (не заполнялась никогда; `deployment_guide.md` §7 обновлён синхронно). §4 — фронтовый тип `TradingDashboard` приведён к `DashboardResponse` (`active_sessions`, `paused_sessions`, `total_sessions`, `total_pnl`, `total_open_positions`), состав полей закреплён сверкой `.ts` ↔ Pydantic-схемы. §3 — дефолты риск-лимитов `User` заданы `Decimal`, а не `float` (конвенция «деньги и проценты — Decimal»; float ронял арифметику Circuit Breaker на ещё не перезагруженном из БД пользователе). §9 — нагрузочный стенд `signal→order` гоняет Circuit Breaker (раньше звал `process_signal` без `user_id`, и девять проверок риска в замеры не входили); потолок ~20 одновременных сессий подтверждён **на боевом пути**. |
| 2.0 | 2026-07-31 | Sprint_8_Review — починка находок части 2 | §5 — `DailyStat.realized_pnl` ведут **оба** пути закрытия (`OrderManager.close_position` наравне с `RiskMonitor`), поле — источник кривой капитала в `AccountService.get_balance_history`; ключ строки `daily_stats.date` и «сегодня» в `SchedulerService.send_daily_stats` считаются **по UTC** (писатели и читатели сведены к одной зоне, конвенция «все даты в UTC»); у синхронизации календаря MOEX год **биржевой** (`Europe/Moscow`) — на UTC 1 января загрузился бы уходящий год. §5/§12 — Circuit Breaker считает нереализованный P&L открытых позиций на лету (`app/trading/unrealized.py`) вместо колонки `daily_stats.unrealized_pnl`, которую никто не заполнял; лишних записей в горячем пути не добавилось. §4 — `PositionTracker` удалён (мёртвый класс, 0 вызовов из `app/`), состав движка: `TradingSessionManager`, `SignalProcessor`, `OrderManager`, `RiskMonitor`. §5 — lot_size из T-Invest берётся у **торгуемого** инструмента (`api_trade_available_flag` + доска `TQBR`), а не у первого совпадения по тикеру. §6 — `StatusFooter` показывает фактическое число активных сессий; WS-хук `/ws/trading-sessions` игнорирует события вытесненного сокета (StrictMode-дубль больше не даёт ни ложного «переподключения», ни утечки соединения). §4 — Email-канал ограничен `EMAIL_ALLOWED_EVENTS`, синхронизировано с ФТ v3.0. |
| 1.9 | 2026-07-30 | Sprint_8_Review — хвост | §4 Performance metrics — метрики стали **живыми**: заведён кольцевой буфер `app/common/metrics_store.py`, куда пишет `@timed_event`; `/admin/metrics` читает его вместо зашитых mock-массивов; появился недостающий декоратор `trading.signal_to_order` на `OrderManager.process_signal` и клиентский `PerformanceObserver` → `POST /api/v1/observability/lcp`. Зафиксированы измеренные числа и предел масштабирования (p95 держится до ~20 одновременных сессий, при 40 бюджет 500 мс превышен). |
| 1.8 | 2026-07-28 | Sprint_8_Review — closeout | §9 — контракт окружения backend-job в CI (`DEBUG=false` + не-дефолтные фиктивные `SECRET_KEY`/`ENCRYPTION_KEY`, чтобы гейт `check_production_secrets` продолжал работать); запрет `waitForLoadState('networkidle')` в E2E-спеках при живом WebSocket; требования к стенду `auth-hardening.spec.ts` (`PW_API_URL`/`PW_WS_URL`/`PW_ORIGIN`, `PW_ORIGIN` == `CORS_ORIGINS`, `LOGIN_RATE_LIMIT_PER_MINUTE` ≥ 10). Регресс-сверка схемы БД с `Base.metadata` на чистой установке (закрыт S8R-ALEMBIC-FRESH-DB-DRIFT: не хватало таблицы `user_ai_settings` и 8 колонок, login отдавал 500). Baseline: 2223 pytest / 832 vitest, E2E 163 passed одним прогоном, Stack Gotchas 46 → 47. |
| 1.6 | 2026-07-27 | Code Review P0–P1 + S8R | §7 — auth Model A (3 HttpOnly-cookie, CSRF double-submit, WS cookie-auth на upgrade, single-flight refresh на `navigator.locks`) + правило «клиент мимо axios обязан слать `X-CSRF-Token`»; §3/§5 — `PaperPortfolioAccountant` как единая точка мутации денег paper-портфеля и инвариант `Δequity == trade.pnl`; §8 — Alembic обязан уважать `DATABASE_URL`, разовый drain paper-сессий при выкатке; §6 — контракт слоя разметки графика (`requestUpdate` в `attached()`, единая выборка фигуры под курсором); §9 — актуальные baseline-цифры тестов |
| 1.5 | 2026-05-13 | S8 W3 | Добавлен §8.4 «Deployment Architecture (Mac mini + Docker)»; §7 расширен под admin role + security headers + ASGI mount auth; §9 — coverage gate 80% + bandit/safety CI gates; §4 — performance baseline числа из W2 |
| 1.4 | 2026-04-26 | S7 R | Phase 1 feature-complete |
| 1.3 | 2026-04-15 | S6 R | Notifications S6, paper SL/TP |
| ... | | | (см. git log) |

### S8 Production-ready дополнения (v1.5)

| Секция ТЗ | Дополнение | Источник |
|-----------|-----------|----------|
| §1 Стек | Python 3.11 (закреплено в CI), Node 24 LTS (миграция S7R-CI-NODE24-MIGRATION в S8 W3), Docker Compose v2, nginx-alpine, Cloudflare Tunnel | S8 W3 |
| §2.1 Архитектура | Admin role + admin panel (Plotly Dash под `/api/v1/admin/metrics` через `AdminAuthASGIMiddleware`) | S8 W2 |
| §4 API | `/api/v1/health` extended: `cb_state`, `tinvest_connected`, `scheduler_running`, `scheduler_jobs[]`; `/api/v1/market-data/sparkline?ticker=X&hours=N`; `/api/v1/account/balance/history?since_first_activity=true`; `POST /api/v1/notifications/telegram/test`; admin namespace `/api/v1/admin/*` под `require_admin` | S8 W1+W2 |
| §4 Performance metrics (реальные, baseline W2) | signal→order p95: измеряется `@timed_event(name="trading.signal_to_order")` (цель < 500мс — мониторинг через `/admin/metrics`); dashboard LCP (PerformanceObserver) < 2с; Telegram webhook handle p95 (`@timed_event(name="telegram.handle")`) < 3с | S8 W2 §4.3 |
| §4 Performance metrics — **живой источник** (S8R хвост, 2026-07-30) | До этой правки метрики существовали только на бумаге: `/admin/metrics` рисовал **зашитые в код** mock-массивы, `@timed_event` писал длительность в structlog на уровне DEBUG и никуда её не агрегировал, декоратора `trading.signal_to_order` в коде **не было вовсе** (были только `signal.process` и `order.place`), а `PerformanceObserver` не был реализован нигде. Теперь: кольцевой буфер `app/common/metrics_store.py` (в памяти процесса, последние N замеров на метрику) наполняется из `@timed_event`; `OrderManager.process_signal` обёрнут `@timed_event("trading.signal_to_order")`; браузер шлёт LCP в `POST /api/v1/observability/lcp` (обычная аутентификация, **не** `require_admin` — замер шлёт пользователь, а не администратор; значение санитизируется потолком 10 мин); Dash-layout собран как callable, иначе страница застыла бы на состоянии при импорте модуля. Ограничения буфера: рестарт backend обнуляет графики, при нескольких воркерах каждый видит только свою долю. Backtest jobs остаётся mock. | S8R хвост |
| §4 Performance — измеренные значения (2026-07-30, залповый ритм баров, paper) | `signal.process` (оценка стратегии) p95 ≤ **5,6 мс** на всех уровнях нагрузки. `trading.signal_to_order` (полный путь сигнал → ордер) p50/p95: 1 сессия — 18,9 / **21,1 мс**; 5 сессий — 15,0 / **33,4 мс**; 20 сессий — 17,2 / **172,3 мс**; 40 сессий — 27,1 / **827,7 мс**. Цель ТЗ **< 500 мс p95 держится до ~20 одновременных сессий и нарушается при 40** (в 1,7×). Узкое место — не стратегия, а обвязка ордера (последовательные записи в SQLite: sizing, Circuit Breaker, денежный учёт, DailyStat, шина событий). Замеры сняты на paper-режиме; gRPC к T-Invest (`order.place`) в цифры не входит, то есть приведённые числа — **пол**, вклад собственного кода. **Решение заказчика 2026-07-30: потолок разделён надвое.** Причина архитектурная — у SQLite один писатель, БД в WAL с `busy_timeout=30000`, поэтому конкурирующие транзакции ждут в очереди, а не падают (деградация тихая, в виде растущей задержки, а не ошибок); число конкурирующих транзакций = `сессии × commit'ов на сигнал`. **Переход на более производительную СУБД → Sprint 9.** **Сокращение числа пишущих транзакций внутри текущей архитектуры → цикл доведения:** схлопывание двух `commit` в один на paper-пути, `DailyStat` через атомарный upsert вместо read-then-write, снятие лишних `refresh`. Обязательный порядок — замер «до» → профилирование → фиксы → замер «после»; мера без подтверждённого замером выигрыша откатывается. | S8R хвост, B1 |
| §4 Performance — результат трёх мер (2026-07-30, вечер) | **Профилирование выполнено** (`backend/scripts/load_signal_to_order.py --profile`, раскладка по фактическим SQL-операциям): на сигнал приходилось ~20 запросов и **2,2 пишущих транзакции**; время при 40 сессиях доминировал `INSERT INTO live_trades` — 281 мс на вызов, то есть ожидание лока писателя, а не стоимость самой вставки. После мер F1–F3: **1,1 пишущей транзакции** на сигнал, `UPDATE daily_stats` устранён (слился в `INSERT … ON CONFLICT`), минус один `SELECT` (`refresh`). Замеры «до» сняты на отдельном worktree с базовым коммитом, по 3 прогона на уровень, медианы: 1 сессия p95 18,9 → **13,1 мс** (−31%), 5 сессий p50 26,6 → **20,3 мс** (−24%), 20 сессий p50 93,8 → 90,7 мс, 40 сессий p50 190,8 → **170,1 мс** (−11%). ⚠️ **p95 при 20 и 40 сессиях не изменился**: разброс между прогонами на одном коде достигает ±20% (949…1174 мс), и разница «до/после» укладывается в него. Половина транзакций ушла, но очередь к единственному писателю осталась — **потолок снимает только смена СУБД (Sprint 9)**, что подтверждает разделение решения. Меры оставлены: выигрыш измерим на p50 всех уровней, на p95 одной сессии и детерминированно — по числу пишущих транзакций. | S8R хвост, F |
| §7 Security | SecurityHeadersMiddleware: CSP / HSTS / X-Frame-Options / X-Content-Type-Options / Referrer-Policy / Permissions-Policy. XSS-protection (html.escape) в Telegram + Email диспетчерах. bandit + safety security-scan job в CI (medium+ блокирует PR). 0 high findings (см. `Спринты/Sprint_8/security_audit_s8.md`) | S8 W1+W2 |
| §8 Deployment | **Новый §8.4:** Docker compose (backend uvicorn + frontend nginx + sqlite volume) на Mac mini + launchd auto-start + Cloudflare Tunnel SSL. Гайд установки: [deployment_guide.md](deployment_guide.md) | S8 W3 |
| §9 Тестирование | Coverage gate `--cov-fail-under=80` активен в CI. Baseline 2026-05-13: 1490 backend pytest passed, 544 frontend vitest, 158 Playwright nightly. Bandit 0 medium+, safety 1 documented CVE | S8 W1+W2+W3 |
| §5 Backend модули | `app/admin/` (router + dash_mount + metrics_dash), `app/common/observability.py` (`@timed_event`), `app/middleware/security_headers.py` (CSP/HSTS), `EVENT_MAP` 17 ключей в `notification/event_bus.py` | S8 W1+W2 |
| §6 Frontend модули | `components/dashboard/{Health,Sparkline,Balance,ActivePositions}Widget.tsx`, `components/common/ErrorBoundary.tsx`, `components/admin/AdminLayout.tsx` + `ProtectedAdminRoute`, `api/types.ts::PaginatedResponse<T>` + `unwrapPaginated()` | S8 W1+W2 |

### Дополнения по итогам код-ревью P0–P1 и Sprint_8_Review (v1.6)

| Секция ТЗ | Дополнение | Источник |
|-----------|-----------|----------|
| **§7 Auth (Model A)** | Оба JWT — в cookie, не в `localStorage`. Три cookie: `access_token` (`path=/`), `refresh_token` (`path=/api/v1/auth/refresh`), `csrf_token` (TTL = refresh), все HttpOnly+Secure кроме csrf (double-submit). `login/setup/refresh` возвращают тело **без токенов** (`AuthMetaResponse`), `logout` стирает все три. Refresh читается из cookie и защищён CSRF double-submit (убран из exempt-списка). Бутстрап клиента — `GET /auth/me` под loading-гейтом. Cross-tab single-flight refresh на `navigator.locks` (grace-окно сознательно не вводилось). | P1 auth-hardening |
| **§7 CSRF: правило для не-axios клиентов** | Interceptor двойной отправки CSRF живёт в axios-инстансе (`api/client.ts`). **Любой клиент, который ходит мимо него** (SSE через `fetch`, `EventSource`, ручные `fetch`), обязан сам подставлять `X-CSRF-Token` из cookie — иначе `CSRFMiddleware` видит «cookie есть, заголовка нет» и отбивает запрос 403. Единый источник токена — экспортируемый `getCSRFToken()`; дублировать регулярку разбора cookie запрещено. Каждый такой клиент покрывается тестом на заголовок. | S8R BUG-32 (класс BUG-21) |
| **§7 WebSocket** | Авторизация по cookie **на upgrade** (`common/ws_auth.py`, применяется к трём точкам), первым кадром сервер шлёт `auth_ok`, проверка `Origin`. Токен в query-string не используется. Отказ — accept → close (S8R-AUDIT-060: close до accept uvicorn превращал в HTTP 403, браузер видел 1006): 4401 без reason — истёк JWT (клиент делает один общий refresh и переподключается), 4401 `revoked`/`logout` — сессию закрыл сервер (тихий стоп без refresh), 4403 — нет доступа, 1013 — сбой БД при проверке (обычный обрыв). Клиент — `WsReconnectController` во всех WS-хуках: backoff ≤ 30 с, после 10 обрывов подряд — баннер «Нет связи с сервером» с «Повторить». | P1 W3 + auth-hardening |
| **§3/§5 Денежный учёт paper** | Единая точка мутации — `PaperPortfolioAccountant`. Семантика полей: `balance` = свободный кэш, `blocked_amount` = капитал открытых позиций по цене входа, `equity = balance + blocked_amount`. Buy-fill списывает и **отклоняет** при нехватке; ручное/all-close и SL/TP-close кредитуют с учётом `lot_size`. Расчёты T+1 по деньгам paper не моделирует: `available = balance`, функций разблокировки (`unblock_settled_funds`, `_auto_unblock_if_settled`) в коде нет; остаток прежней ветки T+1 в `PaperBrokerAdapter.place_order` помечен `DEAD PATH` и уходит в A-06 с B4 (уточнено в v4.2, ФТ §7.4). Инвариант, закреплённый тестом: `Δequity == trade.pnl − trade.commission` (с `quantize` — точно и для облигаций; при нулевой ставке комиссии вырождается в прежний `Δequity == trade.pnl`). Комиссия удерживается из `balance` и **не** попадает в `blocked_amount` — иначе закрытие вернуло бы её как часть cost_basis. Следствие: CB-drawdown работает на paper. | BE-TRAD-06 + Q4 (`S8R-PAPER-NO-COMMISSION`) |
| **§6 Слой разметки графика** | `series.attachPrimitive()` не запрашивает перерисовку — примитив обязан звать `param.requestUpdate()` в `attached()`, иначе фигура не рисуется до постороннего redraw. Выбор фигуры под курсором — одна функция `primitives/pick.ts::pickTopHit` (перебор от верхнего слоя к нижнему) для **всех** обработчиков: click, hover, контекстное меню, drag; иначе захват фигуры проваливается в панорамирование графика. Анкоры (`p1`/`p2`, `corner-*`) отдаются hit-тестом только у выделенной фигуры → после выделения hit-тест пере-опрашивается. Подробности: `Develop/stack_gotchas/gotcha-36-attachprimitive-no-redraw.md`. | S8R BUG-33 / BUG-34 |
| **§8 Deployment: миграции** | `alembic/env.py` обязан уважать `DATABASE_URL` из окружения. Приоритет URL: программный override (`config.set_main_option`, тесты) > `DATABASE_URL` > значение из `alembic.ini`; async-драйверы (`+aiosqlite`/`+asyncpg`) срезаются, т.к. Alembic ходит синхронно. Без этого миграции молча уходят в БД из ini — при отдельном пути к данным (Docker/прод) обновится не тот файл. | S8R FIND-06 |
| **§8 Deployment: выкатка** | Разовый **drain открытых paper-сессий** перед обновлением на релиз с BE-TRAD-06: позиции, открытые старой схемой учёта, на границе деплоя дают расхождение `blocked_amount`. Процедура — `deployment_guide.md` §7. | BE-TRAD-06 |
| **§9 Тестирование (baseline 2026-07-27)** | backend pytest **2186 passed / 1 xfailed / 0 failed**; frontend vitest **772 passed** (111 файлов); `tsc --noEmit` 0; `eslint --max-warnings 0` 0; bandit 0 medium+; Playwright: `auth-hardening.spec.ts` **7/7** (реальный cookie-flow), mock-suite **161 passed / 3 skipped**. | S8R + P1 |

### Дополнения по итогам цикла доведения S8R — остаток приёмки (v1.7)

| Секция ТЗ | Дополнение | Источник |
|-----------|-----------|----------|
| **§4 API: настройки уведомлений** | `GET /api/v1/notifications/settings` отдаёт **канонический** список всех `event_type` из `EVENT_MAP` (сохранённые строки поверх дефолтов), а не только строки из БД: для нового пользователя это 18 объектов, а не `[]`. Каждый элемент несёт производное поле **`email_supported: bool`** — единый источник `EMAIL_ALLOWED_EVENTS` (`app/notification/email.py`). Фронт список email-поддерживаемых типов **не дублирует**, а гасит по нему Email-тумблер. `PUT /settings/{event_type}` возвращает тот же объект; `email_supported` всегда производное, входное значение игнорируется. ⚠️ `PUT` пока **не отклоняет** `email_enabled=true` для неподдерживаемого типа — защита только на UI (осознанно: 422 сломал бы wizard-flow). | S9-EMAIL-TOGGLE-MISMATCH |
| **§4/§5 Бэктест: постановка job'а** | Строка `Backtest` создаётся до постановки job (UI нужен `backtest_id` в ответе 202), но **любой** отказ `job_manager.submit()` (per-user cap, лок БД, `CancelledError` при обрыве клиента) обязан её удалить — записей `status='queued'` без соответствующего job'а не остаётся ни при каком исходе. INSERT в `backtest_jobs` ретраится на «database is locked». Контракт отказов: занятость БД → **503** + `Retry-After: 3` + `{"detail": "<русский текст>"}`; превышение лимита → **422** + `{"detail": "Превышен лимит фоновых бэктестов (макс. 3)"}`. | S9-BG-BACKTEST-503-UX |
| **§5 Бэктест: таймаут данных** | Выборка свечей ограничена таймаутом, **соразмерным объёму работы**: `resolve_data_timeout(timeframe, date_from, date_to)` = `clamp(⌈период / чанк(tf)⌉ × PER_CHUNK, MIN, MAX)`, где чанк — сколько календарных дней провайдер отдаёт за один запрос (~7 дней для 1m, ~1 год для 1h и старше). Настройки: `BACKTEST_DATA_TIMEOUT_PER_CHUNK_SEC` (15 с), `BACKTEST_DATA_TIMEOUT_MIN_SEC` (60 с), `BACKTEST_DATA_TIMEOUT_MAX_SEC` (1800 с); `BACKTEST_DATA_TIMEOUT_SEC > 0` — жёсткий override для диагностики, `0` (по умолчанию) — авторасчёт. Фиксированные 180 с одинаково душили и 3 месяца часовок (лишнее ожидание мёртвого источника), и 10 лет минуток (обрыв живой загрузки). Исчерпание → `BacktestDataTimeoutError` с русским текстом → job `status='error'`, запись `backtests` → `status='failed'` + `error_message` с той же причиной. Запись терминального статуса обязана иметь fallback на свежей сессии из фабрики: отмена по `asyncio.wait_for` отравляет текущую `AsyncSession` (`gotcha-38`). | S9-BACKTEST-DATA-TIMEOUT + доведение |
| **§4 Настройки уведомлений: серверная нормализация** | `PUT /notifications/settings/{event_type}` приводит `email_enabled` к `false` для типов вне `EMAIL_ALLOWED_EVENTS` и отвечает **200** с нормализованным значением. 422 сознательно не используется: жёсткий отказ ломает wizard-flow, который отправляет настройки для всех типов разом. UI-гейт по `email_supported` остаётся, серверная нормализация закрывает обход мимо интерфейса. | S9-EMAIL-TOGGLE-MISMATCH (доведение) |
| **§6 Фоновые бэктесты: реконсиляция после перезагрузки** | Состояние фоновых job'ов персистится в localStorage, а WS-канал сервер закрывает через 30 с после терминального события — job, завершившийся при закрытой вкладке, после F5 «висел» в `running` без результата и причины. На mount выполняется разовая сверка незавершённых job'ов с `GET /backtest/jobs/{id}`; терминальный статус и причина подтягиваются в store, недоступная сеть состояние не меняет. **Контракт ответа задаёт `_job_to_dict` (`backtest/router.py`): идентификатор бэктеста приходит ВЛОЖЕННЫМ — `result.backtest_id`, плоского `result_id` в ответе нет.** Все потребители (оба WS-обработчика и REST-реконсиляция) обязаны читать его одинаково: расхождение даёт `status='done'` без `result_id`, и бейдж не покажет кнопку «Открыть результат». Тип клиента `getJob()` обязан соответствовать этой схеме (`progress`, `error_message`, `result`). | S8R доведение + код-ревью |
| **§9 Тесты не ходят в сеть** | Юнит-тесты `app/trading` обязаны подменять `MarketDataService.ensure_lot_size_strict`. По контракту FIX-1 он идёт в T-Invest → MOEX ISS и **бросает** `LotSizeUnavailableError`, когда лот не определить авторитетно, из-за чего `process_signal` пропускает сигнал. Без подмены `test_max_positions_limit` зависел от доступности ISS: пропуск одного из трёх наполняющих сигналов оставлял 2 открытые позиции, и четвёртый сигнал проходил по лимиту. Ветка «лот недоступен → сигнал пропущен» покрыта явным тестом. | S8R доведение |
| **§4 Контракт ошибок для фронта** | Любая ошибка запроса показывается пользователю текстом из `detail` ответа сервера. Сырой текст axios (`Request failed with status code N`, `Network Error`, `timeout of Nms exceeded`) выводить запрещено. Единая точка разбора — `frontend/src/utils/apiError.ts::getApiErrorMessage(err, fallback)`: понимает `{detail: str}`, `{detail: [{msg}]}` (422), `{detail: {message}}`, `{message}`; HTML-тело прокси и blob отсекает; при непарсибельном ответе отдаёт fallback вызывающей стороны. Причина падения фонового job'а: `error_message` (REST) / `message` (WS-кадр). | S9-BG-BACKTEST-503-UX 422 pydantic (S8R-FIX-038): типовые типы ошибок переводятся на русский (`frontend/src/utils/apiError.ts`, словарь согласован с `_CONSTRAINT_TEXT` бэкенда) с подписью поля из `loc`; `value_error` — текстом валидатора без префикса «Value error,», неизвестный тип — исходным `msg`; формы с ошибками по полям используют `getValidationIssues`. S8R-FIX-066: 422 — `{detail:[{type,loc,msg,ctx?}]}`, без `input`; неконечные числа — строкой. `TradeResponse.exit_broker_order_id` (новое поле). |
| **§6 Тема и доступность** | Тема вынесена в `frontend/src/theme.ts`. `--mantine-color-dimmed` переопределяется **только** для dark-схемы через `cssVariablesResolver` (`light: {}` не трогается): контраст приглушённого текста к фону поднят с **4.04 : 1** (ниже порога WCAG AA) до **7.83 : 1** (AAA). Хардкод цветов в компонентах запрещён — правка на уровне токена. Подробности: `gotcha-42`. | S9-DARK-DIMMED-CONTRAST |
| **§6 SPA-роутинг** | Каждый вложенный `<Routes>` обязан иметь собственный `<Route path="*">`: внешний splat-роут (`admin/*`) уже перехватил путь, и глобальный catch-all не срабатывает — вместо 404 рендерится пустой `main`. `/admin/metrics` внутри SPA — страница-заглушка со ссылкой на Dash (дашборд смонтирован в backend через `app.mount` и живёт на другом origin), остальные `/admin/*` → 404. Подробности: `gotcha-43`. | S9-ADMIN-METRICS-SPA-404 |
| **§7 Поля секретов в UI** | Поле ввода стороннего секрета (Telegram bot token и аналогичные) обязано нести `autoComplete="new-password"` + `data-lpignore="true"`. `autocomplete="off"` для `type=password` браузерами игнорируется намеренно, а Mantine `PasswordInput` ставит именно `off` — без явного переопределения Chrome подставляет в поле секрета сохранённый пароль пользователя. Проверяется тестом на **отрендеренный DOM**, не на пропсы. Подробности: `gotcha-41`. | S9-BOT-TOKEN-AUTOFILL |
| **§5 Локализация парсера** | Все пользовательские предупреждения `block_parser.py` (13 мест `warnings.append`) — на русском; внутренние ключи секций отображаются через `SECTION_LABELS_RU`/`_section_label()`. Технические строки логов и текст самих исключений (`{e}`) не локализуются. | S9-BLOCK-PARSER-I18N |
| **§9 Тестирование (после доведения, 2026-07-27)** | backend pytest **2213 passed / 1 xfailed / 0 failed** (+27 к baseline); frontend vitest **828 passed** (119 файлов, +56); `tsc --noEmit` 0; `eslint --max-warnings 0` 0; mypy 0; bandit 0 medium+. Stack Gotchas: 36 → **45**. ⚠️ Известная нестабильность: `test_trading/test_order_manager.py::test_max_positions_limit` упал 1 раз из 3 полных прогонов (изолированно и в остальных прогонах зелёный; `app/trading/` в цикле не менялся) — требует отдельного разбора. | S8R доведение |

---

## СОДЕРЖАНИЕ

- Глоссарий
1. Стек технологий
2. Системная архитектура
3. Схема базы данных
4. API-контракты
5. Backend-модули
6. Frontend-модули
7. Реализация безопасности
8. Развёртывание и эксплуатация
9. Стратегия тестирования
10. Фазы разработки

---

## ГЛОССАРИЙ

| Термин | Определение |
|---|---|
| Circuit Breaker | Механизм автоматического ограничения торговли при превышении лимитов риска |
| Paper Trading | Виртуальная торговля на реальных рыночных данных без реальных денежных средств |
| Equity Curve | Кривая изменения капитала стратегии во времени |
| Slippage (проскальзывание) | Разница между ожидаемой и фактической ценой исполнения ордера |
| НКД | Накопленный купонный доход по облигации |
| OHLCV | Open, High, Low, Close, Volume — стандартный формат свечных данных |
| Cooldown | Минимальный интервал между двумя сигналами входа одной стратегии |
| Drawdown (просадка) | Снижение капитала от пикового значения (в % или абсолютном выражении) |
| T+1 | Режим расчётов на MOEX: поставка ценных бумаг на следующий торговый день |
| Profit Factor | Отношение суммы прибылей к сумме убытков по всем сделкам |
| Sharpe Ratio | Коэффициент Шарпа — мера доходности с поправкой на риск; с v4.10 — годовой по барам теста (§5.17.8) |
| Договор исполнения | Одни правила исполнения бэктеста, Grid Search, теневой сверки и paper: первая цена следующего бара плюс проскальзывание, уровни от цены входа с шагом цены, оба уровня в баре — стоп (v4.10, A-06, §5.17) |
| Журнал исполнений | Строка на каждое исполнение сделки paper, sandbox, real: правило, цена правила, цена по договору, фактическая, разница (§3.30) |
| Бар гэпа | Первый бар ряда с датой МСК позже последнего дня покупки — на нём позиции зачисляется дивиденд (§5.17.6) |
| Прогрев | Бары истории до даты начала бэктеста (и история сессии live): max(200, 5 × наибольший период индикатора) (§5.17.7) |
| Вселенная скринера | Акции доски TQBR по списку биржи (ISS, `INSTRID = EQIN`; 261 на 16.11.2026) — бумаги с `instruments.in_screener_universe = 1` (v4.11, Б-03, §3.31) |
| Ликвидная бумага | Медианный оборот за последние 60 дат торгов ISS не ниже порога пользователя (по умолчанию 10 млн ₽) (§5.18.3) |
| Снимок рынка | Одна строка на бумагу вселенной — цена, изменение за день, оборот, капитализация, сектор, статус у брокера, «ликвидная»; общий для «Скринера» и «Карты рынка» (§4.19, §5.18.4) |
| Поправка сектора | «Тикер → сектор» администратора, важнее сектора брокера, действует сразу (§5.18.5) |
| Win Rate | Доля прибыльных сделок от общего числа сделок (в %) |
| MOEX ISS | Информационно-статистический сервер Московской биржи (публичное API) |

---

## 1. СТЕК ТЕХНОЛОГИЙ

**Ответственный: OPS + все специалисты (ревью)**

### 1.1 Backend

| Компонент | Технология | Версия | Обоснование |
|---|---|---|---|
| Язык | Python | 3.11+ | Минимум 3.10 (ФТ), рекомендуется 3.11+ для `asyncio.TaskGroup` и улучшенной производительности. CI тестирует на 3.10 и 3.11+ |
| Web-фреймворк | FastAPI | 0.143+ | Async-native, авто-генерация OpenAPI, WebSocket поддержка, высокая производительность. С 0.137 `app.routes` — дерево (`_IncludedRouter`): обход маршрутов — через `iter_route_contexts`, иначе проверки маршрутов проходят впустую (gotcha-86) |
| ASGI-сервер | Uvicorn | 0.29+ | Стандарт для FastAPI, поддержка HTTP/WS |
| ORM | SQLAlchemy | 2.0+ | Зрелая ORM с поддержкой SQLite и PostgreSQL; async через asyncio extension |
| Миграции | Alembic | 1.13+ | Стандарт для SQLAlchemy, обратимые миграции, авто-генерация |
| СУБД (фаза 1) | SQLite | 3.40+ | Нулевая конфигурация для локального deploy; WAL-режим для конкурентного чтения |
| Бэктестинг | Backtrader | 1.9.78+ | Требование ФТ; зрелая библиотека стратегий |
| Технические индикаторы | Backtrader (`bt.indicators`) | — | TA-Lib удалён 2026-10-19: нигде не импортировался, песочница пускает только `datetime`/`math`/`decimal`/`backtrader` |
| Брокер gRPC | grpcio + tinkoff-investments | 0.2.0+ (tinkoff SDK) | Официальный SDK T-Invest API |
| Задачи по расписанию | APScheduler | 3.10+ | In-process планировщик, не требует внешних зависимостей |
| Хеширование паролей | Argon2id (argon2-cffi) | 23.1+ | Рекомендация OWASP, winner PHC |
| Шифрование | cryptography (Fernet / AES-256-GCM) | 42.0+ | AES-256-GCM для API-ключей |
| JWT | PyJWT | 2.8+ | Компактный, проверенный |
| Sandbox | — (RestrictedPython удалён, S8R-AUDIT-050/S8R-FIX-047) | — | Исполняется только код, сгенерированный из IR блоков (`ir_codegen`), в дочернем процессе с лимитами (S8R-AUDIT-001/006, §5.11) |
| Telegram | python-telegram-bot | 21.0+ | Async, поддержка inline-кнопок, webhook |
| HTTP-клиент | httpx | 0.27+ | Async, для MOEX ISS и OpenAI-совместимых AI-провайдеров |
| AI SDK | anthropic + httpx2, openai | 1.15+ / 2.13+, 3.31+ | anthropic ≥ 1.0 построен на `httpx2` и отвергает клиент `httpx`: `ClaudeProvider` создаёт `httpx2.AsyncClient(follow_redirects=False)`, `httpx2` — прямая зависимость (gotcha-85) |
| Валидация | Pydantic | 2.6+ | Нативная интеграция с FastAPI |
| Логирование | structlog | 24.1+ | Структурированные логи (JSON), контекстное логирование |
| Данные | pandas | 2.2+ | Манипуляция OHLCV, интеграция с Backtrader |
| Научные вычисления | numpy | 1.26+ | Зависимость pandas/Backtrader |
| PDF-отчёты | WeasyPrint | 62.0+ | Генерация PDF из HTML-шаблонов (бэктест-отчёт) |
| Excel-экспорт | openpyxl | 3.1+ | Генерация xlsx (налоговый отчёт, журнал сделок) |
| Админ-метрики | Plotly Dash (plotly 7.1+, plotly.js 4) | — | Графики `/api/v1/admin/metrics`; в plotly.js 4 кнопка выгрузки в облако Plotly включена по умолчанию — у всех графиков `showSendToCloud: False` |

### 1.2 Frontend

| Компонент | Технология | Версия | Обоснование |
|---|---|---|---|
| Фреймворк | React | 19.3+ | Обширная экосистема, совместимость с Blockly |
| Язык | TypeScript | 5.4+ | Типизация снижает баги в сложном UI |
| Сборщик | Vite | 5.4+ | Скорость HMR, ESM-native |
| UI-библиотека | Mantine | 7.8+ | Готовые dark-theme компоненты, таблицы, формы; русская локализация |
| Графики | TradingView Lightweight Charts | 4.1+ | Требование ФТ; MIT, canvas, производительность |
| Раскладка «Карты рынка» | `d3-hierarchy` (+ `@types/d3-hierarchy` 3.1.7, dev) | 3.1.3 | v4.11 (Б-03): treemap для холста `<canvas>`; без зависимостей и install-скриптов, в отдельном чанке страницы (§6.12) |
| Блочный редактор | Google Blockly | 11.0+ | Требование ФТ; drag-and-drop, кастомные блоки |
| Состояние | Zustand | 5.0+ | Минималистичный, без boilerplate. В v5 селектор, возвращающий новый объект/массив, даёт бесконечный ререндер — такие селекторы не писать (или `useShallow`) |
| Маршрутизация | React Router (`react-router-dom`) | 7.18+ | Стандарт для React SPA; декларативный режим (`BrowserRouter`/`Routes`), навигация идёт в `startTransition` |
| HTTP-клиент | Axios | 1.7+ | Интерцепторы для JWT, отмена запросов |
| WebSocket | Нативный WebSocket + reconnecting-websocket | 4.4+ | Авто-переподключение |
| Таблицы | TanStack Table | 9.3+ | Сортировка, фильтрация, пагинация. С 2026-10-13 (S8R-FIX-079) — v9: фичи перечисляются явно (`tableFeatures({...})`), модели строк создаются фабриками `create*RowModel` вместо геттеров `get*RowModel`, состояние читается `table.state`, ячейки — `getAllCells()`, сортировщик колонки — `sortFn`. Набор фич входит в типы (`ColumnDef<TFeatures, TData, TValue>`), поэтому объявляется один раз на таблицу и переиспользуется хелпером колонок. Правило `react-hooks/incompatible-library` v9-таблицу не трогает — подавление снято |
| Код (отображение) | Monaco Editor (read-only) или react-syntax-highlighter | latest | Подсветка Python-кода стратегии |
| Иконки | Tabler Icons | 3.1+ | Бесплатные, SVG |
| Уведомления (toast) | Mantine Notifications | 7.8+ | Интеграция с Mantine |

### 1.3 Инфраструктура

| Компонент | Технология | Версия | Обоснование |
|---|---|---|---|
| Менеджер зависимостей (Python) | pip + venv (или Poetry) | — | Стандартный Python workflow, без внешних зависимостей |
| Менеджер зависимостей (Frontend) | pnpm | 9.0+ | Быстрый, экономит дисковое пространство |
| Запуск приложения | Скрипт `start.sh` / `start.bat` | — | Единый скрипт запуска backend + frontend (build → serve) |
| Process manager (опционально) | Honcho / Foreman | — | Запуск backend + frontend в одном терминале через Procfile |
| CI (рекомендация) | GitHub Actions | — | Стандарт для open-source workflow |

> **Примечание:** Docker **не используется** в фазе 1. Приложение устанавливается и запускается нативно на машине пользователя через Python venv + pip и pnpm. Это упрощает установку для нетехнического пользователя и снижает системные требования. При необходимости Docker может быть добавлен в фазе 2 для упрощения развёртывания в продакшн.

---

## 2. СИСТЕМНАЯ АРХИТЕКТУРА

**Ответственный: ARCH (определяет), все специалисты (реализация)**

### 2.1 Общая архитектура — монолит с модульным разделением

Фаза 1 реализуется как монолитное приложение с чётким модульным разделением. Каждый модуль — отдельный Python-пакет с определённым public API. Границы модулей проектируются так, чтобы в фазе 2 при необходимости можно было выделить их в микросервисы без переписывания бизнес-логики.

```
moex-terminal/
├── backend/
│   ├── app/
│   │   ├── main.py                  # FastAPI application factory
│   │   ├── config.py                # Pydantic Settings (env)
│   │   ├── dependencies.py          # FastAPI dependency injection
│   │   ├── middleware/
│   │   │   ├── auth.py              # JWT verification middleware
│   │   │   ├── rate_limit.py        # Rate limiter
│   │   │   └── csrf.py              # CSRF protection
│   │   ├── auth/                    # Модуль аутентификации
│   │   │   ├── router.py
│   │   │   ├── service.py
│   │   │   ├── schemas.py
│   │   │   └── models.py
│   │   ├── admin/                   # Админ-зона /api/v1/admin/* (S8); с A-02 — соответствия тикеров
│   │   │   ├── router.py
│   │   │   ├── ticker_aliases.py    # A-02: CRUD соответствий тикеров, проверка периодов, аудит (§3.5.6)
│   │   │   └── schemas.py           # A-02: TickerAliasWrite / TickerAliasResponse
│   │   ├── strategy/                # Модуль стратегий
│   │   │   ├── router.py
│   │   │   ├── service.py
│   │   │   ├── schemas.py
│   │   │   ├── models.py
│   │   │   ├── block_parser.py      # Шаблон → блоки (режим B)
│   │   │   └── ir_codegen.py        # IR → Python/Backtrader (code_generator.py удалён, S8R-AUDIT-019)
│   │   ├── backtest/                # Модуль бэктестинга
│   │   │   ├── router.py
│   │   │   ├── service.py
│   │   │   ├── schemas.py
│   │   │   ├── models.py
│   │   │   ├── cerebro_setup.py     # v4.10 (A-06): build_cerebro — единственная настройка Cerebro и брокера бэктеста и Grid (§5.17.2)
│   │   │   ├── money_market.py      # v4.10: денежный рынок за период — LQDT (ISS), RUONIA, кэш money_market_quotes (§5.17.8)
│   │   │   ├── cbr_ruonia.py        # v4.10: RUONIA — SOAP DailyInfo ЦБ
│   │   │   ├── spread_check.py      # v4.10: проскальзывание против спреда стакана T-Invest (§5.17.4)
│   │   │   ├── session_defaults.py  # v4.10: умолчание проскальзывания сессии paper (§5.17.4)
│   │   │   └── engine.py            # Backtrader wrapper
│   │   ├── trading/                 # Модуль торговли
│   │   │   ├── router.py
│   │   │   ├── service.py
│   │   │   ├── schemas.py
│   │   │   ├── models.py
│   │   │   ├── order_manager.py
│   │   │   ├── position_tracker.py
│   │   │   ├── execution_journal.py # v4.10 (A-06): журнал исполнений — ExecutionPrice, record_execution, rule_for_close (§5.17.5)
│   │   │   ├── minute_prices.py     # v4.10: последняя минута бумаги — «текущая цена» ручных и аварийных закрытий paper
│   │   │   └── paper_engine.py      # Paper trading
│   │   ├── execution/               # v4.10 (A-06): договор исполнения — чистые правила цены, уровней, дней дивиденда, прогрева (§5.17.1)
│   │   │   ├── contract.py          # Slippage, DEFAULT_SLIPPAGE, EXECUTION_RULES_VERSION, apply_slippage, protective_levels, level_exit
│   │   │   ├── dividends.py         # last_buy_day (T+2/T+1), gap_bar_index, next_session_date, dividend_cash, load_dividend_schedule
│   │   │   └── warmup.py            # warmup_bars = max(200, 5 × период), warmup_request_start
│   │   ├── broker/                  # Брокерский адаптер
│   │   │   ├── base.py              # Абстрактный интерфейс
│   │   │   ├── factory.py           # Фабрика адаптеров
│   │   │   ├── tinvest/             # T-Invest реализация
│   │   │   │   ├── adapter.py
│   │   │   │   ├── grpc_client.py
│   │   │   │   ├── stream_manager.py
│   │   │   │   └── mappers.py
│   │   │   └── moex_iss/            # MOEX ISS fallback
│   │   │       ├── client.py
│   │   │       └── mappers.py
│   │   ├── market_data/             # Сервис рыночных данных
│   │   │   ├── router.py
│   │   │   ├── service.py
│   │   │   ├── schemas.py
│   │   │   ├── models.py
│   │   │   ├── cache.py             # OHLCV cache logic
│   │   │   ├── bond_service.py      # Расчёт НКД, параметры облигационных выпусков
│   │   │   ├── seam.py              # A-02: шов и окна источников (1m до 13.06.2018 — ISS, D 07.03–12.06.2018 — ISS)
│   │   │   ├── iss_history.py       # A-02: IssHistoryLoader — старая история ISS, дни торгов (§5.6.8)
│   │   │   ├── backfill.py          # фоновая догрузка (A-01; с A-02 — по расписанию, вселенная, очередь)
│   │   │   ├── market_data_account.py  # A-02: счёт для рыночных данных
│   │   │   ├── ticker_aliases.py    # A-02: соответствия тикеров (тикер ISS за период)
│   │   │   └── validator.py         # Валидация OHLCV
│   │   ├── notification/            # Сервис уведомлений
│   │   │   ├── router.py
│   │   │   ├── service.py
│   │   │   ├── schemas.py
│   │   │   ├── models.py
│   │   │   ├── telegram_bot.py
│   │   │   └── email_sender.py
│   │   ├── corporate_actions/       # Обработка корпоративных действий
│   │   │   ├── service.py           # CorporateActionHandler
│   │   │   ├── models.py
│   │   │   └── schemas.py
│   │   ├── circuit_breaker/         # Circuit Breaker
│   │   │   ├── service.py
│   │   │   ├── schemas.py
│   │   │   └── models.py
│   │   ├── scheduler/               # Планировщик
│   │   │   ├── service.py
│   │   │   ├── moex_calendar.py
│   │   │   └── jobs.py
│   │   ├── news/                    # Новости (v4.4, Б-01): сборщик, архив, словарь, реакция, лента (§5.14)
│   │   │   ├── router.py            # /api/v1/news* (§4.14)
│   │   │   ├── service.py           # NewsService — задания планировщика
│   │   │   ├── collector.py         # проход по источникам → news_items
│   │   │   ├── feeds.py             # HTTP к лентам, разбор RSS и ISS sitenews
│   │   │   ├── dictionary.py        # словарь акций TQBR, сопоставление заголовка
│   │   │   ├── labeling.py          # разметка словарём
│   │   │   ├── reactions.py         # реакция «до / после» по 1m ISS
│   │   │   ├── ai_labeling.py       # v4.9 (Б-07): шаг ИИ-разметки, подсказка PROMPT_VERSION, проверка ответа (§5.16.2)
│   │   │   ├── ai_benchmark.py      # v4.9: проверка модели на эталоне, ворота «ИИ-разметки» (§5.16.3)
│   │   │   ├── alerts.py            # v4.9: алерт `news` — условие, две стадии, доставка через outbox (§5.16.5)
│   │   │   ├── settings.py          # v4.9: выключатели раздела «Новости» (§5.16.6)
│   │   │   ├── data/d8_news_labels.json  # v4.9: эталон D8 (108 заголовков)
│   │   │   ├── queries.py, schemas.py, models.py
│   │   │   └── constants.py         # интервалы, пороги, лимиты (без адресов лент)
│   │   ├── dividends/               # Дивиденды (v4.5, A-02): история T-Invest, поправки, сверка со Смартлабом (§5.15)
│   │   │   ├── router.py            # /api/v1/dividends* (§4.15)
│   │   │   ├── service.py           # загрузка T-Invest, сверка, поправки, расхождения, read_dividends
│   │   │   ├── smartlab.py          # страница smart-lab.ru и её разбор (html.parser)
│   │   │   ├── schemas.py, models.py
│   │   ├── screener/                # Скринер (v4.11, Б-03): вселенная, история дня ISS, снимок, поправки секторов (§5.18)
│   │   │   ├── router.py            # /api/v1/screener/* — снимок, порог, секторы (§4.19)
│   │   │   ├── admin_router.py      # /api/v1/admin/sector-overrides* (подключён в admin/router.py)
│   │   │   ├── universe.py          # вселенная TQBR, статус у брокера (Shares), кэш IMOEX, sync_universe
│   │   │   ├── history.py           # итоги дня ISS: первое заполнение, дозагрузка, медиана и шкала
│   │   │   ├── snapshot.py          # build_snapshot: доска ISS, GetLastPrices, broker_quote_is_current
│   │   │   ├── jobs.py              # run_screener_pass — задание screener_nightly
│   │   │   ├── iss_gate.py          # замок ISS модуля, ожидание фона A-02
│   │   │   ├── sectors.py, overrides.py, user_settings.py
│   │   │   ├── schemas.py, models.py
│   │   ├── ai/                      # AI-интеграция
│   │   │   ├── router.py
│   │   │   ├── service.py
│   │   │   ├── background.py        # v4.9 (Б-07): фоновые задачи — провайдер, complete_json, учёт, дневной лимит (§5.16.1)
│   │   │   ├── schemas.py
│   │   │   ├── providers/
│   │   │   │   ├── base.py          # Абстрактный LLM provider
│   │   │   │   ├── claude.py
│   │   │   │   └── openai.py
│   │   │   └── prompts.py           # Системные промпты
│   │   ├── sandbox/                 # Песочница кода
│   │   │   ├── executor.py
│   │   │   ├── ast_analyzer.py
│   │   │   └── whitelist.py
│   │   └── common/                  # Общие утилиты
│   │       ├── database.py          # SQLAlchemy engine/session
│   │       ├── exceptions.py
│   │       ├── pagination.py
│   │       └── crypto.py            # AES-256-GCM helpers
│   ├── alembic/                     # Миграции
│   │   ├── alembic.ini
│   │   └── versions/
│   ├── tests/
│   │   ├── unit/
│   │   ├── integration/
│   │   └── e2e/
│   ├── pyproject.toml
│   └── scripts/
│       └── start_backend.sh       # Запуск uvicorn + alembic migrate
├── frontend/
│   ├── src/
│   │   ├── main.tsx
│   │   ├── App.tsx
│   │   ├── api/                     # API-клиент, типы
│   │   ├── stores/                  # Zustand stores
│   │   ├── hooks/                   # Кастомные React-хуки
│   │   ├── components/
│   │   │   ├── layout/              # Shell, Sidebar, Header
│   │   │   ├── dashboard/
│   │   │   ├── strategy/
│   │   │   ├── backtest/
│   │   │   ├── trading/
│   │   │   ├── charts/
│   │   │   ├── account/
│   │   │   ├── settings/
│   │   │   └── notifications/
│   │   ├── pages/                   # Route pages
│   │   └── utils/
│   ├── public/
│   ├── index.html
│   ├── vite.config.ts
│   ├── tsconfig.json
│   └── package.json
├── scripts/
│   ├── start.sh                   # Запуск backend + frontend (dev)
│   ├── start.bat                  # Windows-версия
│   ├── install.sh                 # Установка зависимостей (venv + pip + pnpm)
│   └── install.bat                # Windows-версия
├── Procfile                        # Для Honcho/Foreman (опционально)
├── .env.example
└── README.md
```

### 2.2 Диаграмма потоков данных

```
                     ┌──────────────┐
                     │   Frontend   │
                     │   (React)    │
                     └──────┬───────┘
                            │ REST + WebSocket
                            ▼
                     ┌──────────────┐
                     │   FastAPI    │
                     │  (Backend)   │
                     └──┬───┬───┬───┘
                        │   │   │
          ┌─────────────┘   │   └─────────────┐
          ▼                 ▼                  ▼
   ┌─────────────┐  ┌─────────────┐   ┌──────────────┐
   │   SQLite    │  │  T-Invest   │   │  LLM API     │
   │   (ORM)     │  │  (gRPC)     │   │  (Claude/    │
   │             │  │             │   │   GPT-4)      │
   └─────────────┘  └──────┬──────┘   └──────────────┘
                           │
                    ┌──────┴──────┐
                    │  MOEX ISS   │
                    │  (fallback) │
                    └─────────────┘
                           │
                    ┌──────┴──────┐
                    │  Telegram   │
                    │  Bot API    │
                    └─────────────┘
```

### 2.3 WebSocket-каналы

Backend поддерживает мультиплексированное WebSocket-соединение на одном endpoint `/ws`. Клиент подписывается на каналы через JSON-сообщения:

```
Канал "market:{ticker}:{timeframe}" — свечи и цены в реальном времени
Канал "trades:{session_id}"        — сделки стратегии
Канал "notifications"              — in-app уведомления
Канал "health"                     — статус компонентов системы
Канал "backtest:{backtest_id}"     — прогресс бэктеста
```

> **Внутренний канал шины (A-01, v4.0):** `bars:{ticker}:{timeframe}` (событие `bar.closed`, §5.6.6) — только `event_bus` внутри процесса; подписка клиента на него отклоняется (`forbidden`, как на любой неизвестный канал). Клиентам по-прежнему отдаётся `market:{ticker}:{timeframe}` (`candle.update`). С A-03 (v4.1) событие читают торговые сессии (`_SessionListener.listen_bar_events`, §5.4.6).

> **Дилерский канал (A-03, v4.1; ADR 0021):** `dealer:{ticker}:{timeframe}` — обновления дилерских свечей T-Invest (часы дилера), ТФ только `DEALER_TIMEFRAMES` = 1m, 5m, 15m, 30m, 1h, 4h (иначе `invalid_channel`). Публичный, как `market:` (`backtest/ws.py::_authorize_channel`, разбор — `market_data/schemas.py::parse_stream_channel`, общий с REST `/candles/subscribe`). События и состав данных — как у основного канала: `{channel, event: "candle.update", data: {timestamp, open, high, low, close, volume, stream_gen, bar_complete, bar_pending}}`. Держит ту же подписку T-Invest на 1m, что основной ключ того же ТФ. Контракт общий с блоком Б-02 (показ на графике) — меняется только после вопроса заказчику; ~~потребителя на фронте пока нет~~. С v4.3 (Б-02) потребитель — переключатель «Внебиржевые часы» панели графика: `useCandleStream(…, 'dealer')` шлёт `POST /candles/subscribe?…&source=dealer` и принимает только канал с префиксом `dealer:` (иначе `null` и `console.warn`); кадры — в дилерский ряд через то же слияние, что основные (§6.4, ADR 0027).

### 2.4 Принцип внутреннего взаимодействия модулей

Модули взаимодействуют через dependency injection (FastAPI `Depends`) и вызов сервисных слоёв напрямую (не через HTTP). Каждый модуль экспортирует `service` — единственную точку входа для других модулей. Модели (SQLAlchemy) могут иметь foreign keys на модели других модулей — это допустимо в монолите.

Для асинхронного взаимодействия (уведомления, срабатывания circuit breaker) используется in-process event bus на основе Python `asyncio.Queue` с pub/sub паттерном:

```python
# common/event_bus.py
class EventBus:
    async def publish(self, event_type: str, payload: dict) -> None: ...
    async def subscribe(self, event_type: str, handler: Callable) -> None: ...
```

События:
- `order.placed`, `order.filled`, `order.rejected`
- `position.opened`, `position.closed`
- `cb.triggered` — срабатывание Circuit Breaker (S8R-AUDIT-073): одно событие в `trades:{session_id}` сработавшей сессии, публикует `CircuitBreakerEngine._trigger` сразу после commit паузы, до принудительного закрытия позиций; payload — `event_type`, `reason`, `signal`, `scope`, `money_class`, `trigger_value`/`limit_value` (строкой). По каждой сессии, реально поставленной на паузу (условный UPDATE только из `active`), — `session.paused` `{session_id, status, reason}`. `CircuitBreakerEvent.trigger_value/limit_value` — факт и порог проверки (`Numeric(18,4)` — колонка смешанных единиц ₽/%/счётчики, деньги в payload — 2 знака; у проверки без числа — NULL в БД и `null` в payload; миграция `752ae89f7ec2`, S8R-FIX-039). **A-04 (ТЗ v4.7, ADR 0043):** после commit `_trigger` публикует копию `cb.triggered` (тот же payload — с A-04 в нём есть и `session_limit`, — но без `outbox_id`) в `system:{user_id}` — для живой истории на «Риск-менеджмент»; `NotificationService` этот канал не слушает (подписан только на `trades:{sid}` и `broker:status`), уведомление остаётся одно. Там же новые события без уведомления: `stop_all.changed` (тело — `StopAllState`, §4.4), `cb.close_report` (`{event_id, closed_count, total_count}`); в `trades:{sid}` — `session.limits_changed` (`{session_id, has_session_limits}`), у `session.paused` / `session.resumed` — поле `pause_reason`. В `EVENT_MAP` ничего не добавлено.
- `connection.lost`, `connection.restored`
- `strategy.started`, `strategy.stopped`, `strategy.paused`
- `backtest.completed`, `backtest.failed`

**Гарантия доставки критических событий:**
Реализовано S8R-AUDIT-086 (`app/common/outbox.py`): для `order.placed`, `trade.filled` (исполнение), `cb.triggered`, `connection.lost` строка `pending_events` (§3.20) пишется в транзакции состояния до commit; publish — после, с `outbox_id`. `processed_at` ставится в транзакции in-app уведомления (идемпотентно); фан-аут `connection.lost` — идемпотентно по `(outbox_id, user_id)` (`Notification.related_entity_type="pending_event"`), `processed_at` после доставки всем. При старте повторяются необработанные `order.placed` / `trade.filled` / `cb.triggered` в окне 1 с…1 ч, ≤ 200 строк; `connection.lost` снимается с повтора (`stale_state` — это состояние, а не факт). Запись `connection.lost` мультиплексором — фоновой задачей ≤ 5 с, стрим её не ждёт.

S8R-FIX-063 ч.2: `cb_unpriced_trade` без своей строки настроек наследует каналы `cb_triggered` (`SETTINGS_INHERIT_FROM`; доставка и `GET /notifications/settings`; первый `PUT` дополняет непереданные поля из родителя); событие в `EMAIL_ALLOWED_EVENTS`.

S8R-FIX-065: `trade.closed` и `position.mismatch` (закрытие, paper SL/TP, отмена pending, сверка после переоткрытия, вход без цены, остаток входа) — строка `pending_events` той же транзакцией, что запись. После снятия локов `close_trade` и отключения адаптера (`publish_staged_after`): шина (WS), доставка своих строк фоновой задачей (`deliver_staged` → `NotificationService.deliver_staged_events`); при отмене — обе фоном. Дедупликация — по `processed_at`. Нет получателя — строка не пишется. Повтор при старте — `OUTBOX_REPLAY_EVENTS` (+`trade.closed`, `position.mismatch`).

A-05 (ТЗ v4.2, ФТ §10.1): строки фактов выше гарантируют in-app уведомление; внешнюю доставку (Telegram/почта) гарантирует вторая строка — `notification.dispatch`. Её пишет **каждое** уведомление, у которого запланирован хотя бы один внешний канал, в транзакции INSERT уведомления (`NotificationService._create_notification_impl`, как раньше `stage_notification` ценового алерта); повтор по сроку — задача планировщика `retry_notification_delivery` раз в 10 с (§5.7, §5.9).

---

## 3. СХЕМА БАЗЫ ДАННЫХ

**Ответственный: BACK2 (схема БД, модели, миграции)**

Все таблицы ниже определяются как SQLAlchemy ORM-модели. Типы указаны в нотации SQLAlchemy. Все временные метки хранятся в UTC.

> **Важное примечание о типах для финансовых данных:** Для всех денежных сумм, цен и процентов используется `Numeric` вместо `Float` для предотвращения ошибок округления IEEE 754. Типы: `Numeric(18, 8)` — цены, `Numeric(18, 2)` — денежные суммы, `Numeric(10, 4)` — проценты. В Python-коде для финансовых расчётов использовать `decimal.Decimal`.

### 3.0 Сверка схемы с моделями (S8R-AUDIT-088, 2026-10-02)

Источник истины — модели SQLAlchemy (`app/*/models.py`) и миграции Alembic (head `7fe0fae4293c`). Подразделы 3.1–3.22 сверены программно с `Base.metadata`; таблицы `users` и `broker_accounts` приведены к моделям, остальные расхождения — ниже (поле в модели есть, в описании нет — описание неполное; наоборот — описание устарело):

| Таблица | Только в модели | Только в ТЗ (устарело) |
|---|---|---|
| `ai_provider_configs` | `prompt_tokens_limit`, `prompt_tokens_used`, `completion_tokens_limit`, `completion_tokens_used`, `price_per_1m_prompt`, `price_per_1m_completion` | — |
| `backtests` | `parity_checked`, `parity_signals_match`, `parity_divergences`, `parity_metrics_diff` | — |
| `chart_drawings` | `display_order` | — |
| `corporate_actions` | `amount`, `description`, `record_date`, `created_at` | `detected_at` |
| `daily_stats` | — | `unrealized_pnl` (удалено миграцией `e2f3a4b5c6d7`), `peak_equity` (удалено миграцией `a7c1d9e2b4f8`, S8R-FIX-013) |
| `live_trades` | `figi`, `lot_size`, `client_order_id`, `commission_source`, `exit_broker_order_id`, `exit_client_order_id`, `exit_order_placed_at` | — |
| `pending_events` | `payload` | `payload_json` (§3.20 приведён к модели в v4.2: `payload`, индекса `idx_pending_unprocessed` нет) |
| `price_alerts` (удалена ревизией `d92b7dd5c939`, A-03; данные — в `alerts`, §3.13) | `figi`, `condition`, `target_price` | `direction`, `price_level` |
| `strategies` | `description` | — |
| `strategy_versions` | `comment`, `created_by` | — |
| `trading_sessions` | `timeframe`, `strategy_name`, `commission_pct`, `peak_equity`, `consecutive_broker_rejections`, `position_mismatch_at`, `position_mismatch_note` | — |
| `user_notification_settings` | `in_app_enabled` | `inapp_enabled` |

Таблицы моделей без подраздела 3.x: `ai_usage_daily` (§5.10.4), `backtest_jobs` (§11.3), `circuit_breaker_configs`, `circuit_breaker_events`, `instruments`, `tax_lots`, `tax_reports` (§18.2 ФТ), `user_ai_settings`, `user_favorites` — описание по моделям. С v4.7 (A-04) `circuit_breaker_*` и `trading_stops` описаны в §3.28. С v4.11 (Б-03) колонки `instruments` вселенной скринера и таблицы скринера описаны в §3.31.

### 3.1 Таблица `users`

```
users
├── id: Integer, PK, autoincrement
├── username: String(50), UNIQUE, NOT NULL
├── password_hash: String(255), NOT NULL          -- Argon2id hash
├── email: String(255), NULLABLE
├── risk_daily_loss_pct: Numeric(10, 4), DEFAULT 5.0       -- % от баланса
├── risk_max_position_pct: Numeric(10, 4), NOT NULL, DEFAULT 25.0  -- % капитала на позицию
├── risk_max_drawdown_pct: Numeric(10, 4), DEFAULT 20.0    -- % от пика equity
├── disclaimer_accepted_at: DateTime, NULLABLE              -- S8R-AUDIT-043: пишется POST /users/me/disclaimer/accept
├── failed_login_count: Integer, DEFAULT 0       -- Счётчик неудачных попыток входа (атомарный UPDATE, S8R-AUDIT-043)
├── locked_until: DateTime, NULLABLE              -- Блокировка до (при brute-force)
├── is_active: Boolean, DEFAULT True
├── is_admin: Boolean, NOT NULL, DEFAULT False
├── wizard_completed_at: DateTime, NULLABLE       -- мастер первого запуска
├── token_version: Integer, NOT NULL, DEFAULT 0   -- S8R-AUDIT-035: рост — отзыв всех токенов
├── created_at: DateTime, NOT NULL, DEFAULT utcnow
└── updated_at: DateTime, NOT NULL, DEFAULT utcnow, onupdate utcnow
```

> S8R-AUDIT-088: полей `risk_daily_loss_fixed`, `risk_max_position_size`, `risk_daily_trade_limit` в модели нет (лимиты — в `circuit_breaker_configs`).

### 3.2 Таблица `broker_accounts`

```
broker_accounts
├── id: Integer, PK, autoincrement
├── user_id: Integer, FK(users.id), NOT NULL
├── broker_type: String(20), NOT NULL             -- 'tinvest', 'alor', ...
├── name: String(100), NOT NULL                   -- Человекочитаемое имя
├── encrypted_api_key: LargeBinary, NOT NULL      -- AES-256-GCM
├── encrypted_api_secret: LargeBinary, NULLABLE   -- Если требуется
├── encryption_iv: LargeBinary, NOT NULL          -- IV для AES-GCM
├── account_id: String(50), NOT NULL              -- ID счёта у брокера
├── account_type: String(20), NULLABLE            -- 'broker', 'iis'
├── is_sandbox: Boolean, NOT NULL, DEFAULT False  -- эндпоинт песочницы T-Invest
├── is_active: Boolean, DEFAULT True
├── has_withdrawal_rights: Boolean, DEFAULT False  -- Предупреждение
├── has_trading_rights: Boolean, NOT NULL, DEFAULT False
├── commission_pct: Numeric(10, 4), NOT NULL, DEFAULT 0  -- тариф счёта (оценка комиссии)
├── is_market_data_source: Boolean, NOT NULL, server_default '0'  -- v4.5 (A-02): счёт для рыночных данных (ADR 0035)
├── created_at: DateTime, NOT NULL
├── updated_at: DateTime, NOT NULL
├── INDEX: idx_broker_user (user_id)
└── UNIQUE INDEX: uq_broker_accounts_market_data_source (is_market_data_source) WHERE is_market_data_source = 1  -- не больше одной отмеченной строки
```

**Счёт для рыночных данных (v4.5, A-02; ADR 0035; ФТ §7.7).** Признак ставит только `market_data/market_data_account.py::select_market_data_source` (маршрут `PUT /api/v1/broker-accounts/{id}/market-data-source`, §4.6): свой счёт администратора, T-Invest, `is_active`, с `encrypted_api_key` и `encryption_iv`, `is_sandbox = false` (иначе `MarketDataSourceRejected` → 422 с текстом; чужой или несуществующий — 404); у прежнего счёта (чей бы он ни был) признак снимается в той же транзакции, затем — аудит `broker.market_data_account_selected` (`entity_id` — id строки, `details`: `account_id` брокера, `previous_id`). Снять признак без выбора другого нельзя (маршрута нет). Читает — `market_data_account(db)`: отмеченный счёт, если он T-Invest, активен, не sandbox, с ключом и IV, а владелец активен; иначе `None` — другой счёт не подставляется (S8R-AUDIT-003). Миграция `79f3c238dbb8` признак не ставит. Запросы пользователя идут по-прежнему через `service.select_market_data_account(db, user_id)`.

### 3.3 Таблица `strategies`

```
strategies
├── id: Integer, PK, autoincrement
├── user_id: Integer, FK(users.id), NOT NULL
├── name: String(200), NOT NULL
├── status: String(20), NOT NULL, DEFAULT 'draft'
│         -- 'draft','tested','paper','live','paused','archived'
├── current_version_id: Integer, FK(strategy_versions.id), NULLABLE
├── created_at: DateTime, NOT NULL
├── updated_at: DateTime, NOT NULL
└── INDEX: idx_strategy_user (user_id)
```

### 3.4 Таблица `strategy_versions`

```
strategy_versions
├── id: Integer, PK, autoincrement
├── strategy_id: Integer, FK(strategies.id), NOT NULL
├── version_number: Integer, NOT NULL
├── text_description: Text, NULLABLE
├── blocks_json: Text, NOT NULL                   -- JSON блочной схемы (Blockly XML/JSON)
├── generated_code: Text, NOT NULL                -- Python-код для Backtrader
├── parameters_json: Text, NULLABLE               -- JSON с параметрами стратегии
├── ai_chat_history: Text, NULLABLE               -- JSON истории диалога с AI
├── parse_report: Text, NULLABLE                  -- A-05 (v4.2): JSON `ParseReport.to_dict()` — отчёт разбора описания (§5.2.2а); NULL — не составлялся (версия до `cdb15f21e5ca` или текст не в формате ИИ-чата)
├── created_at: DateTime, NOT NULL
├── UNIQUE: uq_strategy_version (strategy_id, version_number)
└── INDEX: idx_sv_strategy (strategy_id)
```

### 3.5 Таблица `ohlcv_cache`

```
ohlcv_cache
├── id: Integer, PK, autoincrement
├── ticker: String(20), NOT NULL
├── timeframe: String(10), NOT NULL               -- только '1m' и 'D' (v4.0, A-01): остальные ТФ собирает агрегатор, в БД не пишутся
├── timestamp: DateTime, NOT NULL                 -- Время открытия свечи (наивное UTC); метка D: T-Invest 00:00 UTC, ISS 21:00 UTC предыдущих суток; с v4.5 (A-02) ISS-дневка окна 07.03–12.06.2018 — 00:00 UTC даты МСК (ADR 0032)
├── open: Numeric(18, 8), NOT NULL
├── high: Numeric(18, 8), NOT NULL
├── low: Numeric(18, 8), NOT NULL
├── close: Numeric(18, 8), NOT NULL
├── volume: BigInteger, NOT NULL                  -- штуки своего дня («как было»), у всех источников одна единица
├── source: String(20), NULL                      -- 'tinvest', 'moex_iss' (NULL — строки до введения поля; в `_not_iss_source` считаются T-Invest); с v4.5 строки 'moex_iss' в окне ISS (1m до шва, D 07.03–12.06.2018) — часть ряда пользователя с T-Invest (§5.6.8)
└── UNIQUE: uq_ohlcv_lookup (ticker, timeframe, timestamp)   -- единственный индекс таблицы
```

Запись любого ТФ, кроме `1m` и `D`, в `_save_to_cache` — `ValueError`. Ряд — `(ticker, timeframe, source)`; в основном ряду T-Invest только биржевые свечи (источник свечи `EXCHANGE`), дилерские — в `ohlcv_dealer` (§3.5.1).

**Окна источников (v4.5, A-02; ADR 0031, 0032).** Ряд пользователя с T-Invest (`exclude_iss=True`) читает строки `moex_iss` только в «окне ISS» своего ТФ (`seam.iss_window`: 1m — раньше `TINVEST_MINUTE_SEAM` 2018-06-13; D — `[ISS_DAILY_FROM 2018-03-07, шов)` по дате МСК метки); вне окна — только T-Invest. T-Invest в окно не пишет. Старую историю ISS пишет только `IssHistoryLoader` (`ON CONFLICT DO NOTHING`: строка T-Invest не затирается). ISS-дневки в БД лежат на двух сетках: в окне — 00:00 UTC, вне окна — 21:00 UTC предыдущих суток (инвариант A-01); код, берущий дату ISS-дневки из метки, обязан это учитывать.

**Масштаб строк — «как было» (v4.0, дефект 4 тестового стенда, решение заказчика 2026-10-24; `app/market_data/splits.py`).** OHLC и `volume` строки — в ценах и штуках её дня; то же в `ohlcv_dealer`. Источники отдают разную базу: архив `history-data` и ISS `history` — «как было»; `GetCandles` (1m, D, дилерские) — пересчитан к текущему состоянию бумаги (T, дробление 1→10 с 17.04.2026, 22.10.2025: архив O 3021,8 и 3 615 777 шт, `GetCandles` — 302,18 и 36 157 770); ISS `candles.json` пересчитывает по дроблению только цену. Поэтому перед записью:
- свеча `GetCandles` раньше границы дробления, наступившего к моменту ответа окна, — `to_as_was`: цена × rt/rf, объём × rf/rt (`BarLoader._store_window`; дробления перечитываются на каждое окно);
- свеча ISS (1m/D, viewer без T-Invest) — `to_as_was(…, scale_volume=False)`: только цена × rt/rf (`_iss_stored_series`);
- минута архива — как есть, только биржевые минуты дня, принятого `ArchiveLots.admit` (окно биржевых часов); объём = лоты × лот на дату (§3.5.5), а не × текущий лот.

Дробления — `corporate_actions` с `action_type='split'` (§3.19; ISS `splits.json`, `ratio_from/ratio_to` — та же семантика, что у `process_split`: после дробления цена × rf/rt, объём × rt/rf), `load_splits(db, ticker)`. Граница — `split_opened_before(ex_date)`: 00:00 МСК `ex_date` (первый торговый день в новом масштабе), наивное UTC; строка раньше границы — в старом масштабе. Дробление, граница которого ещё не наступила, ряд не меняет. Цена округляется до 8 знаков (`Numeric(18, 8)`), объём — до целых штук (половина — вверх).

Чтение по умолчанию — с учётом наступивших дроблений: строки раньше границы — цена × rf/rt, объём × rt/rf (`adjust_for_splits`; цена из кэша — `adjust_price`). Пересчёт хранимых строк — только в `MarketDataService._read_stored` (ряды) и `_latest_bar` (цена из кэша: `last_cached_bar` — в текущем масштабе, `closed_bar_close_before` — в масштабе момента сделки); производный ТФ ISS, который не кэшируется, — в `_iss_derived_series` (§5.6.6). Новое дробление БД не переписывает; исключение — дробление, которое детект узнал уже после его границы (§5.12).

**Примечание по производительности:** Для OHLCV-кеша при 5 годах дневных данных ~1250 записей на инструмент — вполне подъёмно для SQLite. При минутном таймфрейме за 5 лет — ~600 000 записей на инструмент. Поиск идёт по индексу уникальности `uq_ohlcv_lookup` (`sqlite_autoindex_ohlcv_cache_1`; `EXPLAIN QUERY PLAN` — `COVERING INDEX`, тест `test_bar_storage_migrations.py`) — O(log n). Отдельный `idx_ohlcv_lookup` по тем же колонкам с v4.0 удалён (ревизия `c4e81f3a9d27`): дубль занимал ≈ 19 МБ и обновлялся при каждой записи свечи. При переходе на PostgreSQL рекомендуется партиционирование по `ticker`.

#### 3.5.1 Таблица `ohlcv_dealer` (v4.0, A-01)

```
ohlcv_dealer                                      -- дилерские 1m T-Invest (торги выходного дня Т-Банка)
├── id: Integer, PK, autoincrement
├── ticker: String(20), NOT NULL
├── timeframe: String(10), NOT NULL               -- всегда '1m'
├── timestamp: DateTime, NOT NULL                 -- наивное UTC
├── open / high / low / close: Numeric(18, 8), NOT NULL
├── volume: BigInteger, NOT NULL                  -- штуки
├── source: String(20), NULL                      -- 'tinvest_dealer'
└── UNIQUE: uq_ohlcv_dealer_lookup (ticker, timeframe, timestamp)
```

Колонки — как у `ohlcv_cache`, масштаб строк — тоже «как было», чтение `get_dealer_candles` — с учётом дроблений (§3.5); с v4.3 (Б-02) — по параметру `adjust` (`splits` по умолчанию, `none` — «как было»), как у основного ряда. Основной ряд, `/candles`, живая свеча и номер бара графика дилерских свечей не содержат.

#### 3.5.2 Таблица `ohlcv_coverage` (v4.0, A-01)

```
ohlcv_coverage                                    -- что загружено из источника полностью
├── id: Integer, PK, autoincrement
├── ticker: String(20), NOT NULL                  -- канонический тикер
├── timeframe: String(10), NOT NULL               -- хранимый ТФ: '1m' | 'D'
├── source: String(20), NOT NULL                  -- 'tinvest' | 'tinvest_dealer' | 'moex_iss'; с v4.5 (A-02) — отметки 'tinvest_invalid' | 'moex_iss_invalid'
├── range_from: DateTime, NOT NULL                -- наивное UTC, включительно
├── range_to: DateTime, NOT NULL                  -- наивное UTC, исключительно: диапазон [range_from, range_to)
├── loaded_at: DateTime, NOT NULL
└── INDEX: idx_ohlcv_coverage_lookup (ticker, timeframe, source, range_from)
```

Смежные и пересекающиеся диапазоны одного `(ticker, timeframe, source)` сливает `coverage.add_coverage`. Бар, которого нет внутри покрытого диапазона, — «сделок не было». Для свечей, лежавших при миграции, строк нет (непокрыты, дозагружаются по запросу). Инвариант: свечи и их покрытие удаляются одной транзакцией (`coverage.delete_bars_with_coverage`; data-миграция делает то же SQL-ом, gotcha-90).

**Отметки невалидной свечи (v4.5, A-02; ADR 0030, дополняет 0008).** Невалидная свеча по-прежнему не пишется, но из покрытия окна вырезается только её период `[начало свечи, начало следующего бара ТФ)`; сам период записывается строкой с источником `tinvest_invalid` (`bar_loader`) или `moex_iss_invalid` (`iss_history`). Это не покрытие: отметку читает только признак «догружается» (`MarketDataService._history_pending` не считает период недостающим), загрузка её не читает — дыру по-прежнему перезапрашивают синхронный запрос и ночной фон. Отметку не снимает `forget_bars_before_new_splits`; `forget_alias_history` вырезает и отметки ISS своего периода. Покрытие `moex_iss` окна D лежит на сетке суток МСК (`[2018-03-06 21:00, 2018-06-12 21:00)` UTC), а строки — на 00:00 UTC (ADR 0032).

#### 3.5.3 Таблица `ohlcv_volume_check` (v4.0, A-01)

```
ohlcv_volume_check                                -- сверка дневного объёма с дневкой ISS
├── id: Integer, PK, autoincrement
├── ticker: String(20), NOT NULL
├── trade_date: Date, NOT NULL
├── our_volume: BigInteger, NOT NULL              -- сумма биржевых 1m за день, штуки
├── iss_volume: BigInteger, NOT NULL              -- объём календарного дня по ISS `history` TQBR (`get_daily_volumes`; дня там нет — дневка `candles.json`), штуки
├── ratio: Numeric(18, 8), NULL                   -- our / iss; NULL при iss = 0
├── verdict: String(16), NOT NULL                 -- 'ok' | 'mismatch' | 'lot_change'
├── checked_at: DateTime, NOT NULL
└── UNIQUE: uq_ohlcv_volume_check (ticker, trade_date)
```

Читателя, кроме самой сверки (`volume_check.reconcile_volumes`), нет; повторная сверка дня перезаписывает строку. Данные свечей сверка не меняет. Обе стороны — «как было» (минуты в штуках своего дня, объём ISS `candles.json` дробления не пересчитывает), поэтому дробление или смена лота в истории бумаги расхождения не дают; `lot_change` остаётся только для настоящей аномалии. Строки дней раньше границы дробления, которое детект узнал уже после этой границы, удаляет `forget_bars_before_new_splits` (§5.12). Эталон — итог дня ISS `history` TQBR (`get_daily_volumes`); дневка `candles.json` запрашивается только для дней, которых в `history` нет или там объём 0 (бумага не из TQBR; субботы и воскресенья с сессией — объём их сессии выходного дня, §5.5.3); нет и её, а строка `history` с объёмом 0 есть (приостановка: T 13–16.04.2026) — эталон 0: день сверяется, а не запрашивается заново каждым прогоном. Дни из архива попадают в ряд только при точном совпадении с тем же `history` (§3.5.5), поэтому их сверка всегда `ok`; `mismatch` и `lot_change` остаются для дней `GetCandles`.

#### 3.5.4 Таблица `ohlcv_backfill_target` (v4.0, A-01)

```
ohlcv_backfill_target                             -- цели фоновой догрузки
├── id: Integer, PK, autoincrement
├── ticker: String(20), NOT NULL
├── timeframe: String(10), NOT NULL               -- '1m' | 'D'
├── depth_from: DateTime, NOT NULL                -- до какой даты ряд должен быть загружен
├── created_at / updated_at: DateTime
└── UNIQUE: uq_ohlcv_backfill_target (ticker, timeframe)
```

В `app/` таблица только читается (`backfill._collect_targets`); записала её миграция `b9d2f6a14c83` из прежней глубины кэша (до удаления строк): по бумаге — цель `1m` с глубиной `max(самый ранний внутридневной бар, шов 2018-06-13)` и цель `D` с самым ранним дневным баром. Новая бумага попадает в догрузку через активную сессию. **С v4.5 (A-02)** таблица — одна из пяти частей вселенной фона (`backfill.collect_universe`, берётся только `ticker`); `depth_from` не читается: глубина у всех бумаг одна — D T-Invest с 2000-01-01, 1m T-Invest с шва, ISS 1m с 2011-12-01, ISS D — окно. Код в таблицу по-прежнему не пишет; новая бумага попадает в фон через сессию, бэктест, активный алерт или избранное.

#### 3.5.5 Таблица `ohlcv_lot_history` (v4.0, A-01, дефект 4 тестового стенда)

```
ohlcv_lot_history                                 -- лот бумаги с даты — для объёма минут архива history-data
├── id: Integer, PK, autoincrement
├── ticker: String(20), NOT NULL                  -- тикер загрузчика; читается по ticker_variants
├── valid_from: Date, NOT NULL                    -- с этой даты действует lot_size (до следующей строки бумаги)
├── lot_size: Integer, NOT NULL                   -- штук в лоте
├── created_at: DateTime, NOT NULL
└── UNIQUE: uq_ohlcv_lot_history (ticker, valid_from)
```

Архив `history-data` хранит объём в лотах своего дня (SBER: лот 10 по 30.07.2025, лот 1 с 31.07.2025) и не размечает источник — биржевые и дилерские минуты вместе. Какие дни архива идут в основной ряд и с каким лотом, решает `lot_history.ArchiveLots.admit(ticker, {день: все минуты архива за сутки UTC}, *, current_lot) -> {день: ArchiveDay | None}` (дефекты 5 и 6 тестового стенда, ADR 0011 в репозитории кода; `ArchiveDay(lot, window)`, `window=None` — минут нет).

- **Эталон дня** — объём календарного дня по ISS `history` TQBR (`MOEXISSClient.get_daily_volumes`, §5.5.3): итог торгового дня ISS включает торги выходного и праздничного дня (понедельник = сб + вс + пн, вторник 24.02.2026 — и 21–23.02; дефект 7, gotcha-94), клиент вычитает их строками сессии выходного дня. Дневка `candles.json` бывает неверна: 10.04.2025 неполна у всех бумаг (SBER 49 360 530 при 62 862 840 — отсюда «×1,23–1,33»), 01.08.2025 у SBER — с лишним (25 320 049 при 18 473 506 — отсюда «0,73»); `history` на эталонных днях равен сумме минуток.
- **Совпадение** (`match_day`): первое окно `EXCHANGE_WINDOWS` (UTC: с утренней сессией 03:50–20:50, без неё 06:50–20:50, без вечерней 06:50–15:50; МСК = UTC + 3), где Σ лотов минут окна × лот **точно** (`exact_lot`: без остатка) равна итогу дня, а лот входит в `ALLOWED_LOTS` (1, 10, 100, 1 000, 10 000, 100 000) ∪ {`current_lot`}. Эпоха расписания определяется по данным дня. Решение — по всем минутам суток, а не по куску окна загрузки (неполный первый день окна: LKOH 30.12.2024 с 23:00 UTC). Пишутся только минуты окна (`ArchiveDay.takes`), дилер вне окна в основной ряд не попадает.
- **Биржа не торговала** (в `history` объём 0 или дня нет): минуты в архиве есть — `None` (дилер), нет — `ArchiveDay(current_lot, None)`: «сделок не было», день покрывается без запроса.
- **Смена лота — только устойчивая** (`_drop_unconfirmed`): выброс — день, чей лот отличается от лотов предыдущего оставленного и следующего совпавшего дней, когда те равны, — снимается и соседом не служит; остальной день подтверждают лот соседнего оставленного дня, лот истории на этот день или — у дня без соседей — `current_lot`, если истории на день нет или день позже её последней строки (ежедневная догрузка первого дня после настоящей смены лота). Неподтверждённый — `None`.
- **`history` TQBR пуст за весь период, а минуты есть** — знает ли ISS бумагу (`ArchiveLots._iss_knows`): акции и фонды по справочнику (`instrument_kind` ∈ `ISS_SHARES_KINDS`) — всегда (бумага не из TQBR, смена тикера, дни без торгов биржи); прочие — если `candles.json` D непуст за год до конца периода (`ISS_KNOWN_PROBE`, 366 сут; не по дням вызова: ежедневная догрузка одного дня без торгов биржи — LKOH 31.12.2024, только дилер — пуста и у знакомой ISS бумаги) или его запрос упал (WARNING `archive_lot_iss_probe_failed`). Знает — дни идут `GetCandles`; не знает (облигации, фьючерсы: у клиента ISS нет их рынка) — окно с утренней сессией, лот истории на день или `current_lot`, WARNING `archive_lot_iss_empty` (проверить нечем).
- **ISS недоступен** (любое исключение `get_daily_volumes`, WARNING `archive_lot_iss_unavailable`) — `None` на все дни года.

`None` — день берёт `GetCandles` (`BarLoader._load_archive` переносит его окно в fallback; один WARNING `archive_days_via_getcandles` со списками `no_exchange_trades`, `no_exact_match`, `lot_change_unconfirmed`); там источник свечи размечен, дилерские минуты уходят в `ohlcv_dealer`. Прежние допуски (`VOLUME_TOLERANCE` 1 % к дневке `candles.json`, `LOT_KEEP_TOLERANCE` 10 % к лоту соседа) сняты: они пропускали дни с дилерскими минутами (LKOH 30.12.2024 +0,39 %, суббота с сессией +6 %).

Пишет таблицу только `ArchiveLots._remember`: по подтверждённым совпавшим дням — дни, с которых лот отличается от истории, — upsert по `(ticker, valid_from)` через `persist_with_retry_fresh` (INFO `archive_lot_history_updated`); сбой записи загрузку не останавливает (WARNING `archive_lot_history_write_failed`) — история лишь запасной лот. Читает только `ArchiveLots` (`_history`, по `ticker_variants`): лот на день — строка с наибольшим `valid_from <= день`. Цены от лота не зависят. Таблицу создаёт ревизия `a858647f24e6`; посева нет — история копится фоновой догрузкой.

**Миграции хранилища баров** (одна линейная цепочка от `a7c1d9e2b4f8`, head — `a858647f24e6`): `b9d2f6a14c83` — таблица целей, посев целей, удаление из `ohlcv_cache` всех ТФ, кроме `1m` и `D`; `e5a7c2d90b16` — удаление строк T-Invest (перезагрузка по новым правилам) и последнего бара каждого оставшегося ряда ISS; `c4e81f3a9d27` — удаление `idx_ohlcv_lookup`; `f1b3d8e62a54` — `ohlcv_coverage`, `ohlcv_dealer`, `ohlcv_volume_check`; `a858647f24e6` (дефект 4) — `ohlcv_lot_history` и удаление всех рядов свечей прежнего масштаба: `DELETE` из `ohlcv_cache` (целиком), `ohlcv_dealer`, `ohlcv_coverage`, `ohlcv_volume_check`. У строк ISS, записанных до A-01, масштаб цены неизвестен (ISS `candles.json` пересчитывает цену по дроблениям, наступившим к запросу); строки T-Invest на базах, где код A-01 работал до этой ревизии (стенд, локальные), лежат в смешанном масштабе. На рабочей БД при выкатке с `v1.0.0` строк T-Invest к этой ревизии нет (их удалила `e5a7c2d90b16`, приложение между ревизиями не запускалось) — удаляются только 269 строк ISS D. Кэш производный: ряды дозагрузит фоновая догрузка, ряд ISS — запрос графика, уже «как было». Data-миграции — чистый SQL без импорта моделей, `downgrade` — no-op (удалённые свечи перезагружаются из источника; `downgrade` `a858647f24e6` удаляет `ohlcv_lot_history`, строки свечей не возвращает); схема обратима. Замеры и порядок обновления — `deployment_guide.md` §7.

#### 3.5.6 Таблица `ticker_aliases` (v4.5, A-02)

```
ticker_aliases                                    -- соответствие тикеров: под каким тикером ISS знает бумагу в периоде
├── id: Integer, PK, autoincrement
├── ticker: String(20), NOT NULL                  -- канонический тикер (верхний регистр; под ним пишутся строки и покрытие)
├── iss_ticker: String(20), NOT NULL              -- тикер запроса к ISS за даты периода (верхний регистр)
├── valid_from: Date, NULLABLE                    -- дата МСК, включительно; NULL — без начала
├── valid_to: Date, NULLABLE                      -- дата МСК, включительно; NULL — без конца
├── created_at / updated_at: DateTime             -- TimestampMixin
└── UNIQUE: uq_ticker_aliases (ticker, iss_ticker, valid_from)
```

Модель — `market_data/models.py::TickerAlias`. Ведётся вручную в админке (`admin/ticker_aliases.py`, маршруты §4.15); периоды одной бумаги не пересекаются — проверяет приложение (`ValidationError` → 422 «Период пересекается с существующим соответствием …»), не БД. Читатель — `market_data/ticker_aliases.py::iss_ticker_periods`: любая загрузка свечей и дней торгов из ISS (`_fetch_via_iss`, `IssHistoryLoader`) за даты периода идёт по `iss_ticker`, строки и покрытие — под `ticker`; без строки — по каноническому. Добавление, правка и удаление строки в той же транзакции удаляют ISS-строки и покрытие `moex_iss` канонического тикера за её старый и новый период до шва (`forget_alias_history`, gotcha-90 — свечи вместе с покрытием), затем аудит `ticker_alias.saved` / `ticker_alias.deleted`; после коммита вытесняется `AggregateCache` затронутых бумаг; следующий фон перезагружает период по новому правилу. Граница суток МСК в UTC — фиксированное +3.

### 3.6 Таблица `backtests`

```
backtests
├── id: Integer, PK, autoincrement
├── strategy_version_id: Integer, FK(strategy_versions.id), NOT NULL
├── ticker: String(20), NOT NULL
├── timeframe: String(5), NOT NULL
├── date_from: DateTime, NOT NULL
├── date_to: DateTime, NOT NULL
├── initial_capital: Numeric(18, 2), NOT NULL
├── commission_pct: Numeric(10, 4), NOT NULL
├── slippage_value: Numeric(10, 4), NOT NULL      -- Значение проскальзывания
├── slippage_mode: String(10), NOT NULL, DEFAULT 'pct'  -- 'pct' | 'points'
├── params_json: Text, NULLABLE                   -- Доп. параметры запуска
├── status: String(20), NOT NULL                  -- 'running','completed','failed'
├── net_profit: Numeric(18, 2), NULLABLE
├── net_profit_pct: Numeric(10, 4), NULLABLE
├── total_trades: Integer, NULLABLE
├── winning_trades: Integer, NULLABLE
├── losing_trades: Integer, NULLABLE
├── win_rate: Numeric(10, 4), NULLABLE
├── profit_factor: Numeric(10, 4), NULLABLE
├── max_drawdown_pct: Numeric(10, 4), NULLABLE
├── sharpe_ratio: Numeric(10, 4), NULLABLE
├── avg_profit_per_trade: Numeric(18, 2), NULLABLE
├── avg_loss_per_trade: Numeric(18, 2), NULLABLE
├── avg_trade_duration_bars: Numeric(10, 2), NULLABLE
├── avg_trade_duration_seconds: Numeric(12, 2), NULLABLE
├── benchmark_buy_hold_pct: Numeric(10, 4), NULLABLE
├── benchmark_imoex_pct: Numeric(10, 4), NULLABLE
├── equity_curve_json: Text, NULLABLE             -- JSON массив точек equity
├── error_message: Text, NULLABLE
├── lot_size: Integer, NULLABLE                   -- A-05 (v4.2): лот (штук в лоте), которым посчитан бэктест (`resolve_backtest_lot`, §5.3.1б); NULL — бэктест до `cdb15f21e5ca` («лоты штуками» при лоте справочника > 1)
├── execution_rules_version: Integer, NULLABLE    -- v4.10 (A-06): версия правил исполнения (EXECUTION_RULES_VERSION = 1 у новых); NULL — «до договора исполнения» (ADR 0079)
├── dividend_tax_pct: Numeric(10, 4), NULLABLE    -- v4.10: налог на дивиденды лонга, % (параметр запуска, по умолчанию 13)
├── price_pnl: Numeric(18, 2), NULLABLE           -- v4.10: прибыль от цены; price_pnl + dividend_pnl = net_profit
├── dividend_pnl: Numeric(18, 2), NULLABLE        -- v4.10: прибыль от дивидендов (зачисленные деньги)
├── bars_per_year: Numeric(12, 2), NULLABLE       -- v4.10: баров в году по барам теста (ADR 0078)
├── money_market_pct: Numeric(10, 4), NULLABLE    -- v4.10: доходность денежного рынка за период, %; NULL — нет данных
├── money_market_source: String(20), NULLABLE     -- v4.10: 'LQDT' | 'RUONIA' | 'RUONIA+LQDT'
├── report_notes: JSON, NULLABLE                  -- v4.10: строки отчёта — шаг цены неизвестен, прогрев неполный, расхождения или сбой выплат
├── started_at: DateTime, NOT NULL
├── completed_at: DateTime, NULLABLE
├── INDEX: idx_bt_sv (strategy_version_id)
└── INDEX: idx_bt_ticker (ticker, timeframe)
```

**С v4.10 (A-06).** Колонки добавила ревизия `8bddf72586cf` без `server_default` — у всех строк до неё NULL (§3.30). `benchmark_imoex_pct` хранит доходность эталон-индекса: MCFTRR при `execution_rules_version` ≥ 1, IMOEX у старых (код индекса — в ответе `benchmark.index_code`, отдельной колонки нет). `sharpe_ratio` новых — годовой по барам теста (§5.17.8), у старых — по константам часов (`sharpe_basis = legacy_constants`). `slippage_value` по умолчанию — 0,1 %.

### 3.7 Таблица `backtest_trades`

```
backtest_trades
├── id: Integer, PK, autoincrement
├── backtest_id: Integer, FK(backtests.id, ondelete=CASCADE), NOT NULL
├── direction: String(5), NOT NULL                -- 'long'
├── entry_date: DateTime, NOT NULL
├── entry_price: Numeric(18, 8), NOT NULL
├── exit_date: DateTime, NULLABLE
├── exit_price: Numeric(18, 8), NULLABLE
├── volume_lots: Integer, NOT NULL                -- A-05 (v4.2): лоты = |размер, шт.| // `backtests.lot_size`; у бэктестов до `cdb15f21e5ca` — штуки
├── volume_rub: Numeric(18, 2), NOT NULL
├── pnl: Numeric(18, 2), NULLABLE
├── pnl_pct: Numeric(10, 4), NULLABLE
├── commission: Numeric(18, 2), NULLABLE
├── duration_bars: Integer, NULLABLE
├── duration_seconds: Integer, NULLABLE
├── price_note: String(40), NULLABLE              -- A-03 (v4.1): 'цена аукциона' — вход или выход D/W/M исполнен по open бара (минут торгов дня нет, §5.3); NULL — обычное исполнение
├── row_kind: String(20), NOT NULL, DEFAULT 'trade'   -- v4.10 (A-06): 'trade' | 'dividend' — начисление на баре гэпа: direction позиции, entry_date — бар гэпа, entry_price — сумма на акцию брутто, volume_lots — лоты позиции, volume_rub = 0, pnl — зачисленные деньги со знаком, остальное NULL
└── INDEX: idx_btt_backtest (backtest_id)
```

`price_note` отдаётся в ответе API сделок бэктеста (`TradeRecord.price_note`, `backtest/schemas.py`); показ в интерфейсе — вне A-03.

### 3.8 Таблица `trading_sessions`

```
trading_sessions
├── id: Integer, PK, autoincrement
├── strategy_version_id: Integer, FK(strategy_versions.id), NOT NULL
├── broker_account_id: Integer, FK(broker_accounts.id), NOT NULL
├── ticker: String(20), NOT NULL
├── mode: String(10), NOT NULL                    -- 'paper', 'real'
├── status: String(20), NOT NULL
│         -- 'active','paused','stopped','suspended','error'
├── position_sizing_mode: String(20), NOT NULL, DEFAULT 'fixed_lots'    -- 'fixed_sum' | 'fixed_lots' | 'percent' (S8R-AUDIT-071, миграция 6128b9c52d8a)
├── position_sizing_value: Numeric(18, 2), NOT NULL
├── max_concurrent_positions: Integer, DEFAULT 1
├── cooldown_seconds: Integer, DEFAULT 0
├── initial_capital: Numeric(18, 2), NULLABLE     -- Для paper trading
├── consecutive_fund_skips: Integer, DEFAULT 0    -- Пропуски из-за нехватки средств (пауза при ≥3)
├── pause_reason: String(40), NULLABLE           -- v4.7 (A-04): 'stop_all' | тип события CB | 'session:<тип>' (свой лимит сессии); пишет только trading/session_status.py
├── slippage_value: Numeric(10, 4), NOT NULL, DEFAULT 0    -- v4.10 (A-06): проскальзывание сессии (paper — из формы или session_slippage_default; sandbox/real — 0); у сессий до 8bddf72586cf — 0
├── slippage_mode: String(10), NOT NULL, DEFAULT 'pct'     -- v4.10: 'pct' | 'points'
├── dividend_tax_pct: Numeric(10, 4), NOT NULL, DEFAULT 0  -- v4.10: налог на дивиденды лонга, % (новые — 13; до ревизии — 0)
├── started_at: DateTime, NOT NULL
├── stopped_at: DateTime, NULLABLE
├── last_signal_at: DateTime, NULLABLE
├── UNIQUE: uq_active_session (broker_account_id, ticker, mode, status)
│         -- Partial unique: WHERE status = 'active'
└── INDEX: idx_ts_active (status)
```

**Причина паузы (v4.7, A-04; ADR 0042).** Пишут только `CircuitBreakerEngine._trigger` (`cb_pause_reason(event_type, session_limit=)`) тем же условным UPDATE, что ставит паузу (`set_sessions_status_if(..., "paused", pause_reason=)`), и «Стоп всему» (`PAUSE_REASON_STOP_ALL`; сессиям, уже стоявшим на паузе, — `set_pause_reason`, `WHERE status='paused'`). Любой переход в `active` (`set_session_status`, `set_sessions_status_if`) обнуляет; `paused → stopped` не снимает — фронт показывает причину только при `status == 'paused'`. Ручная пауза и автопаузы (расхождение, остаток входа, вход без цены) её не пишут. AST-страж `test_fix_037_session_status_setter.py` причину не ловит — записи только через `session_status.py`.

### 3.9 Таблица `live_trades`

```
live_trades
├── id: Integer, PK, autoincrement
├── session_id: Integer, FK(trading_sessions.id), NOT NULL
├── broker_order_id: String(100), NULLABLE        -- ID ордера у брокера
├── direction: String(5), NOT NULL                -- 'long'
├── order_type: String(20), NOT NULL              -- 'market','limit','stop_market','stop_limit'
├── status: String(20), NOT NULL
│         -- 'pending','partial','filled','cancelled','rejected'
├── entry_signal_price: Numeric(18, 8), NOT NULL  -- Цена на момент сигнала
├── entry_price: Numeric(18, 8), NULLABLE         -- Фактическая цена исполнения
├── exit_signal_price: Numeric(18, 8), NULLABLE
├── exit_price: Numeric(18, 8), NULLABLE
├── volume_lots: Integer, NOT NULL
├── filled_lots: Integer, DEFAULT 0
├── volume_rub: Numeric(18, 2), NOT NULL
├── stop_loss: Numeric(18, 8), NULLABLE
├── take_profit: Numeric(18, 8), NULLABLE
├── pnl: Numeric(18, 2), NULLABLE
├── pnl_pct: Numeric(10, 4), NULLABLE
├── commission: Numeric(18, 2), NULLABLE
├── slippage: Numeric(18, 8), NULLABLE            -- v4.10 (A-06): у paper — проскальзывание сессии в её единицах (trading_sessions.slippage_mode); у sandbox/real — NULL (до v4.10 в схеме стояло «entry_price - entry_signal_price», писателя не было)
├── nkd_entry: Numeric(18, 2), NULLABLE           -- НКД при покупке, на ОДНУ облигацию, ₽ (S8R-FIX-006)
├── nkd_exit: Numeric(18, 2), NULLABLE            -- НКД при продаже, на ОДНУ облигацию, ₽ (S8R-FIX-006: писателя пока нет; налоговый расчёт умножает разницу на количество штук, как `process_coupon`)
├── opened_at: DateTime, NOT NULL
├── closed_at: DateTime, NULLABLE
├── exit_retry_kind: String(10), NULLABLE         -- A-03 (v4.1): список «закрыть при первой возможности» — вид выхода (`ExitKind`: stop|take|signal|forced); NULL — не в списке
├── exit_retry_since: DateTime, NULLABLE          -- с какого момента выход ждёт (UTC)
├── exit_retry_notified_at: DateTime, NULLABLE    -- последнее уведомление `order.error` (не чаще раза в 15 мин)
├── INDEX: idx_lt_session (session_id)
├── INDEX: idx_lt_status (status)
└── INDEX: idx_lt_opened (opened_at)
```

Колонки `exit_retry_*` (ревизия `d92b7dd5c939`, `batch_alter_table`, nullable без значения по умолчанию) пишет и снимает только `trading/pending_exits.py` (`enqueue`, `clear`, `clear_fields`), §5.4.6; закрытие сделки любым путём снимает пометку. Тестовый стенд: `staging_copy.py prepare` обнуляет их у сделок не-paper сессий копии, `check` не пускает `up`, пока такие пометки есть (повтор отправил бы ордер на настоящий счёт).

### 3.10 Таблица `paper_portfolios`

```
paper_portfolios
├── id: Integer, PK, autoincrement
├── session_id: Integer, FK(trading_sessions.id), UNIQUE, NOT NULL
├── balance: Numeric(18, 2), NOT NULL
├── blocked_amount: Numeric(18, 2), DEFAULT 0.0   -- Model A (BE-TRAD-06): капитал открытых позиций по цене входа; T+1 по деньгам не моделируется (ФТ §7.4, v4.2)
├── peak_equity: Numeric(18, 2), NOT NULL
├── updated_at: DateTime, NOT NULL
```

### 3.11 Таблица `audit_log`

Наполнение (S8R-AUDIT-086): `app/common/audit.py` — `audit()`/`build_audit()`, перечень действий `AUDIT_ACTIONS` (auth.*, admin.user_created, admin.granted/admin.revoked, broker.*, session.*, position.closed_manual, order.placed, cb.triggered, parity_override; с v4.5 (A-02) — `broker.market_data_account_selected`, `dividend.correction_saved`, `dividend.correction_deleted`, `ticker_alias.saved`, `ticker_alias.deleted`; с v4.7 (A-04) — `trading.stop_all.enabled` (`details.source` — `web` | `telegram`, `source` записи — тот же) и `trading.stop_all.disabled` (`source=web`), обе с `entity_type="user"`, пишет `StopAllService`); `source` — `web` (с ip и user-agent) | `system` | `cli`. `S8R-FIX-098`: CLI `python -m app.cli.users grant_admin|revoke_admin <username>` пишет `admin.granted`/`admin.revoked` (`entity_type="user"`, `entity_id`, `details={"username"}`, `user_id=NULL`, `source=cli`) в той же транзакции, что смена `is_admin`, плюс structlog `admin_granted`/`admin_revoked`; идемпотентный повтор («already admin» / «not admin») записи не создаёт. Запись — в транзакции действия; неудачные входы — отдельной короткой транзакцией. Детали — только идентификаторы (фильтр секретов, предел длины); неизвестный логин — `sha256[:12]` и длина. Ротация по `created_at` — не реализована.

```
audit_log
├── id: Integer, PK, autoincrement
├── user_id: Integer, FK(users.id), NULLABLE      -- NULLABLE для системных событий
├── action: String(50), NOT NULL                  -- 'login','order.placed','cb.triggered',...
├── entity_type: String(30), NULLABLE             -- 'strategy','order','session',...
├── entity_id: Integer, NULLABLE
├── details: Text, NULLABLE                       -- JSON с деталями
├── source: String(20), NOT NULL                  -- 'web','telegram','scheduler','system'
├── ip_address: String(45), NULLABLE
├── created_at: DateTime, NOT NULL, DEFAULT utcnow
├── INDEX: idx_audit_created (created_at)
└── INDEX: idx_audit_user (user_id, created_at)
```

**Ограничение:** Эта таблица — append-only. На уровне приложения `AuditLogService` предоставляет только метод `create()`. ORM-модель не имеет методов `update` / `delete`. Дополнительно: SQLite trigger на DELETE для защиты:

```sql
CREATE TRIGGER prevent_audit_delete BEFORE DELETE ON audit_log
BEGIN SELECT RAISE(ABORT, 'Deletion from audit_log is prohibited'); END;

CREATE TRIGGER prevent_audit_update BEFORE UPDATE ON audit_log
BEGIN SELECT RAISE(ABORT, 'Update of audit_log is prohibited'); END;
```

### 3.12 Таблица `notifications`

```
notifications
├── id: Integer, PK, autoincrement
├── user_id: Integer, FK(users.id), NOT NULL
├── event_type: String(50), NOT NULL              -- 'trade.opened','cb.triggered',...
├── title: String(200), NOT NULL
├── body: Text, NOT NULL
├── severity: String(10), NOT NULL                -- 'info','warning','critical'
├── is_read: Boolean, DEFAULT False
├── channels_sent: String(100), NULLABLE          -- CSV: 'in_app,telegram,email' (дописывается по мере доставки)
├── channels_failed: String(100), NULLABLE        -- A-05 (v4.2): CSV внешних каналов, не доставленных после всех повторов (§5.7); NULL — неудач нет или уведомление до `cdb15f21e5ca`. В API — `undelivered_channels: list[str]`
├── related_entity_type: String(30), NULLABLE
├── related_entity_id: Integer, NULLABLE
├── created_at: DateTime, NOT NULL
├── INDEX: idx_notif_user_unread (user_id, is_read, created_at)
```

### 3.13 Таблицы `alerts` и `alert_triggers` (v4.1, A-03; до v4.1 — `price_alerts`)

> **С ревизии `d92b7dd5c939` (A-03, ADR 0023) таблицы `price_alerts` нет.** Активные строки (`is_active = 1`) перенесены в `alerts` видом `level` (уровень `target_price`, `condition`, `figi`, `created_at` — как были; `armed = 1`, `price_at_creation` пуст), сработавшие и удалённые не переносятся. `downgrade` создаёт `price_alerts` заново (колонки и индексы ревизии `14c59a1057bf`) с уровнями активных алертов `level`; история срабатываний, цена создания и `armed` теряются (известный дефект: сработавшие алерты снова становятся активными). Прежняя схема (до v4.1; фактические колонки — `figi`, `condition`, `target_price`, см. §3.0): `id, user_id, ticker, figi, condition, target_price, is_active, triggered_at, created_at`, индексы `idx_price_alerts_figi_active`, `idx_price_alerts_user`.

```
alerts                                            -- общая модель алертов (`market_data/models.py::Alert`)
├── id: Integer, PK, autoincrement
├── user_id: Integer, FK(users.id, ondelete=CASCADE), NOT NULL
├── kind: String(20), NOT NULL, DEFAULT 'level'   -- 'level'; с v4.9 — 'news' (Б-07); 'condition' (A-11), 'line' (D4) — без переноса данных
├── ticker: String(20), NULLABLE                  -- канонический тикер; с v4.9 NULL допустим — у вида 'news' пусто (до v4.9 — NOT NULL)
├── figi: String(20), NULLABLE                    -- дозаполняет монитор
├── level_price: Numeric(18, 8), NULLABLE         -- уровень, округлён к шагу цены (`round_to_price_step`)
├── condition: String(10), NULLABLE               -- 'above' | 'below' — ставит сервер по цене при создании
├── price_at_creation: Numeric(18, 8), NULLABLE   -- цена, по которой определено направление
├── price_source: String(10), NULLABLE            -- 'tinvest' | 'iss'
├── armed: Boolean, NOT NULL, DEFAULT 1           -- взведён (после срабатывания — 0 до повторного взведения)
├── is_active: Boolean, NOT NULL, DEFAULT 1       -- не удалён
├── created_at: DateTime, NOT NULL, DEFAULT CURRENT_TIMESTAMP
├── last_triggered_at: DateTime, NULLABLE
├── trigger_count: Integer, NOT NULL, DEFAULT 0
├── timeframe: String(10), NULLABLE               -- поля D2 — задел A-11, пустые
├── condition_tree: JSON, NULLABLE
├── repeat_mode: String(20), NULLABLE
├── bar_mode: String(20), NULLABLE
├── state: JSON, NULLABLE
├── news_filter: JSON, NULLABLE                    -- v4.9 (Б-07): {"tickers": [..] | null, "words", "types", "sentiments"}; tickers null — весь рынок
├── paused_at: DateTime, NULLABLE                 -- v4.9: пауза алерта 'news'
├── active_since: DateTime, NULLABLE              -- v4.9: с какого момента алерт 'news' проверяет новости (создание, снятие паузы)
├── INDEX: idx_alerts_user_active (user_id, is_active)
└── INDEX: idx_alerts_figi_active (figi, is_active)

alert_triggers                                    -- история срабатываний (`AlertTrigger`)
├── id: Integer, PK, autoincrement
├── alert_id: Integer, FK(alerts.id, ondelete=CASCADE), NOT NULL
├── triggered_at: DateTime, NOT NULL
├── price: Numeric(18, 8), NULLABLE               -- цена срабатывания (при гэпе — open минуты); с v4.9 NULL — у срабатывания 'news' (до v4.9 — NOT NULL)
├── gap: Boolean, NOT NULL, DEFAULT 0             -- уровень пройден гэпом
├── news_id: Integer, FK(news_items.id, ondelete=SET NULL), NULLABLE   -- v4.9: новость срабатывания (`fk_alert_triggers_news_id`)
├── labeler: String(20), NULLABLE                 -- v4.9: по чьей разметке решено — 'ai' | 'dictionary'
├── status: String(12), NULLABLE                  -- v4.9: 'sent' (уведомление ушло) | 'unchecked' (не проверено)
├── note: String(20), NULLABLE                    -- v4.9: 'by_dictionary' | 'ai_unavailable' | 'ai_disabled'
├── UNIQUE: uq_alert_triggers_alert_news (alert_id, news_id)   -- v4.9: одна новость — одно срабатывание на алерт (NULL у уровневых не мешает)
└── INDEX: idx_alert_triggers_alert (alert_id)
```

Срабатывание — условный `UPDATE alerts SET armed = 0, trigger_count = trigger_count + 1, last_triggered_at = … WHERE armed AND is_active`, строка `alert_triggers` и уведомление — одна транзакция (§5.6.7). Поля D2 — разрешённый задел без читателя до A-11.

**Вид `news` (v4.9, Б-07; ADR 0064).** Колонки добавила и NULL у `alerts.ticker` и `alert_triggers.price` разрешила ревизия `b07a1e5c0d42` (`batch_alter_table` пересоздаёт обе таблицы с прежними индексами и FK). Тип `Alert.ticker` в Python остался `str`: поле читает только код уровневых алертов. Удаление алерта `news` — `is_active = false`, история остаётся; уровневые маршруты (`/market-data/price-alerts`) и монитор вид `news` не видят (`GET` фильтрует `kind == 'level'`), но `DELETE /market-data/price-alerts/{id}` вид не проверяет. `downgrade` ревизии удаляет алерты `news` и все срабатывания без цены (иначе NOT NULL не вернуть). Проверка и доставка — §5.16.5.

### 3.14 Таблица `chart_drawings`

```
chart_drawings
├── id: Integer, PK, autoincrement
├── user_id: Integer, FK(users.id), NOT NULL
├── ticker: String(20), NOT NULL
├── timeframe: String(5), NOT NULL
├── type: String(20), NOT NULL                    -- trendline, hline, vline, rect, label, long_position, short_position; с v4.6 — ray, extended, hray, channel, fib
├── data_json: Text, NOT NULL                     -- JSON с координатами
├── style_json: Text, NULLABLE                    -- JSON с цветом, толщиной
├── created_at: DateTime, NOT NULL
├── updated_at: DateTime, NOT NULL
└── INDEX: idx_drawing_lookup (user_id, ticker, timeframe)   -- до v4.3; с v4.3 — idx_drawing_user_ticker (user_id, ticker)
```

> **Рисунки по «пользователь + бумага» (v4.3, Б-02; ADR 0026).** Ревизия `0acf47e6607f` снимает `idx_drawing_lookup` и создаёт `idx_drawing_user_ticker (user_id, ticker)` (`batch_alter_table`, только операции с индексами; в лог — число рисунков и ключей «пользователь + бумага»). Строки не переписываются: `timeframe` уже колонка, регистр тикера сравнивается `ticker_eq` (`upper(ticker)`, записи до S8R-FIX-056 — `SIZ6`), поэтому, как и раньше, индекс работает только префиксом `user_id`. Серверного предела числа рисунков нет; 100 — предел фронтенда на пару «бумага + ТФ». `downgrade` возвращает прежний индекс.

> **Рисунки на всех ТФ и новые типы (v4.6, Б-04; ADR 0050–0054).** Схема не меняется. `timeframe` — «где нарисован»: фронтенд показывает рисунок на панелях любого ТФ его бумаги, колонка и поле API не переименовывались. Новые типы — `ray`, `extended`, `hray`, `channel`, `fib`; в `data_json` новые необязательные поля — `p3` (третья точка канала) и `levels` (уровни Фибоначчи, числа). Список типов — `DrawingType` и `ALLOWED_TYPES` в `app/chart_drawings/schemas.py`, их проверки — §4.13. Предел 100 — **на бумагу** (все ТФ вместе) и только во фронтенде, при создании (`MAX_DRAWINGS_PER_TICKER`); серверного предела по-прежнему нет.
>
> **Ревизия `b6b5064c0be6` (Б-04, только данные; поверх `79f3c238dbb8`).** Удаляет рисунки, у которых хотя бы одна точка с годом < 2000 — номер бара, записанный в `t` как unix-секунды (`1969-12-31T21:10:55Z`), до S8R-FIX-102. Смотрятся `t` у `p1`, `p2`, `anchor`, `entry`, `end` и `t` верхнего уровня (`vline`). Год — `datetime.fromisoformat`, при неудаче — первые четыре цифры строки вида `YYYY-`; значение, которое не разбирается (не строка, не ISO, пустой или битый `data_json`), рисунок не удаляет. Удаление — пачками по 500, чистый SQL и Python без моделей приложения. Журнал: `Б-04 chart_drawings: рисунки с точкой до 2000 года — удалено N`. `downgrade` ничего не восстанавливает и пишет `Б-04 chart_drawings: откат не восстанавливает удалённые рисунки`. На копии рабочей БД — 3 из 3 (линии SBER: `trendline` 1h — 2, 5m — 1), таблица после миграции пуста. Старые линии не перерисовываются — решение заказчика 2026-11-09 (ADR 0054).

### 3.15 Таблица `telegram_links`

```
telegram_links
├── id: Integer, PK, autoincrement
├── user_id: Integer, FK(users.id), UNIQUE, NOT NULL
├── chat_id: BigInteger, NOT NULL
├── linked_at: DateTime, NOT NULL
├── is_active: Boolean, DEFAULT True
```

### 3.16 Таблица `user_notification_settings`

```
user_notification_settings
├── id: Integer, PK, autoincrement
├── user_id: Integer, FK(users.id), NOT NULL
├── event_type: String(50), NOT NULL
├── telegram_enabled: Boolean, DEFAULT True
├── email_enabled: Boolean, DEFAULT False
├── inapp_enabled: Boolean, DEFAULT True
├── UNIQUE: uq_notif_setting (user_id, event_type)
```

### 3.17 Таблица `daily_stats`

```
daily_stats
├── id: Integer, PK, autoincrement
├── session_id: Integer, FK(trading_sessions.id), NOT NULL
├── date: Date, NOT NULL
├── trades_opened: Integer, DEFAULT 0
├── trades_closed: Integer, DEFAULT 0
├── pnl: Numeric(18, 2), DEFAULT 0.0
├── realized_pnl: Numeric(18, 2), DEFAULT 0.0
├── UNIQUE: uq_daily_stat (session_id, date)
```

`unrealized_pnl` удалена миграцией `e2f3a4b5c6d7`, `peak_equity` — миграцией
`a7c1d9e2b4f8` (S8R-FIX-013): обе не заполнялись никогда. Пик капитала живёт в
`trading_sessions.peak_equity` и `paper_portfolios.peak_equity`; «пик за день»
дал бы третье, более мягкое определение просадки под тем же словом. Приём
миграции — `batch_alter_table` (SQLite без native DROP COLUMN, gotcha-12),
идемпотентно и с обратимым `downgrade`.

### 3.18 Таблица `bond_info_cache`

```
bond_info_cache
├── id: Integer, PK, autoincrement
├── ticker: String(20), UNIQUE, NOT NULL
├── isin: String(20), NOT NULL
├── nominal: Numeric(18, 2), NOT NULL            -- Номинал облигации
├── coupon_rate: Numeric(10, 4), NULLABLE        -- Ставка купона (% годовых)
├── coupon_frequency: Integer, NULLABLE           -- Количество выплат в год
├── maturity_date: Date, NULLABLE                 -- Дата погашения
├── coupon_dates_json: Text, NULLABLE             -- JSON массив дат ближайших купонов
├── updated_at: DateTime, NOT NULL
```

### 3.19 Таблица `corporate_actions`

```
corporate_actions
├── id: Integer, PK, autoincrement
├── ticker: String(20), NOT NULL
├── action_type: String(20), NOT NULL             -- 'split', 'dividend', 'coupon'
├── ex_date: Date, NOT NULL                       -- Дата отсечки
├── ratio_from: Integer, NULLABLE                 -- Для сплита: было
├── ratio_to: Integer, NULLABLE                   -- Для сплита: стало
├── amount_per_share: Numeric(18, 8), NULLABLE    -- Дивиденд/купон на акцию/облигацию
├── currency: String(3), DEFAULT 'RUB'
├── detected_at: DateTime, NOT NULL
├── processed: Boolean, DEFAULT False
├── UNIQUE: uq_corp_action (ticker, action_type, ex_date)
└── INDEX: idx_corp_ticker (ticker, ex_date)
```

A-02 (v4.5) таблицу и журнал начислений не менял; история выплат по бумаге — отдельные таблицы §3.27. **С v4.10 (A-06)** строки дивидендов создаёт детект по `read_dividends` (§3.27), а не `GetDividends` ключом владельца сессии; зачисление по сделке отмечается в `dividend_accruals` (§3.30), `processed` ставится, когда незачисленных позиций по действию не осталось.

### 3.20 Таблица `pending_events`

```
pending_events
├── id: Integer, PK, autoincrement
├── event_type: String(50), NOT NULL              -- 'order.placed', 'trade.filled', 'trade.closed', 'cb.triggered', 'connection.lost', ..., 'notification.dispatch'
├── payload: Text, NULLABLE                       -- JSON {"channel", "data", "user_id"} (`common/outbox.stage_event`)
├── created_at: DateTime, NOT NULL, server_default now()  -- naive UTC
└── processed_at: DateTime, NULLABLE              -- «забрана»: условный UPDATE … WHERE processed_at IS NULL (`claim_pending_event`)
```

Приведено к модели `app/common/models.py::PendingEvent` и миграции `3d3e4e3036a6` (ТЗ v4.2): колонка — `payload`, не `payload_json` (см. §3.0); **индекса нет** — прежнее описание `idx_pending_unprocessed` не было реализовано ни одной ревизией. Строки после обработки не удаляются (ротации нет). Outbox — §2.4, §5.7.

- **`notification.dispatch`** (S8R-FIX-051; с A-05 — у каждого уведомления с запланированным Telegram/почтой): `data` = `{"notification_id": N, "delivered": [каналы], "failed_attempts": K, "next_attempt_at": ISO UTC | null}`. `delivered` — каналы, отмеченные сразу после отправки; `failed_attempts` — число неудачных попыток; `next_attempt_at` — срок следующей. Строки без новых полей (до A-05) читаются как `failed_attempts = 0`, срок — `created_at`. Строка забирается, когда доставлены все запланированные каналы, после последней попытки (отметка `notifications.channels_failed`) или по возрасту > 1 ч.
- **Чтение:** повтор при старте — `load_replayable` (окно возраста 1 с…1 ч, ≤ 200 строк, все типы); фоновый повтор — `load_unprocessed(db, event_type="notification.dispatch")` — все необработанные строки типа без окна, старые первыми, то есть полный просмотр таблицы раз в 10 с. **Известное ограничение:** таблица растёт без чистки и без индекса; на объёмах одного пользователя (на копии рабочей БД 2026-10-25 — 19 строк) проход занимает миллисекунды; индекс и чистка старых строк — вне A-05.

### 3.21 Таблица `ai_provider_configs`

```
ai_provider_configs
├── id: Integer, PK, autoincrement
├── user_id: Integer, FK(users.id), NOT NULL
├── provider_type: String(20), NOT NULL           -- 'claude', 'openai', 'custom'
├── display_name: String(100), NOT NULL           -- Человекочитаемое имя ("Claude Sonnet", "GPT-4o")
├── encrypted_api_key: LargeBinary, NOT NULL      -- AES-256-GCM (тот же CryptoService, что и для брокеров)
├── encryption_iv: LargeBinary, NOT NULL          -- IV для AES-GCM
├── model_id: String(100), NOT NULL               -- 'claude-sonnet-4-6-20250514', 'gpt-4o', ...
├── api_base_url: String(255), NULLABLE           -- Для custom-провайдеров (Ollama, LM Studio и др.)
│                                                  -- SSRF-барьер (S8R-AUDIT-018, S8R-FIX-046, `app/ai/url_validator.py`): адрес и все DNS-ответы — не приватные/служебные; у IPv6 проверяется встроенный IPv4 (mapped, compatible ::/96, SIIT, NAT64 64:ff9b::/96 и 64:ff9b:1::/48, 6to4, Teredo). Ограничение: NAT64 с операторским префиксом (RFC 6052 §2.2) не распознаётся. S8R-FIX-130: для хостов `KNOWN_AI_PROVIDER_HOSTS` (openrouter.ai, api.openai.com, api.anthropic.com, api.deepseek.com, api.groq.com, api.mistral.ai, generativelanguage.googleapis.com, dashscope[-intl].aliyuncs.com; точное совпадение) допустим адрес-заглушка прокси/VPN — Proxifier macOS `0.0.0.0/8` (кроме `0.0.0.0`) и `fd00:696e::/32`, fake-IP `198.18.0.0/15` и `fc00::/18`, только по https (по http при подменённом DNS запрос с ключом ушёл бы на чужой адрес — сертификат не защищает): соединение на заглушку перехватывает прокси. Proxifier для Windows (`127.8.*.*`, loopback) не поддержан.
├── daily_limit: Integer, DEFAULT 100             -- Лимит запросов в сутки
├── monthly_limit: Integer, DEFAULT 3000          -- Лимит запросов в месяц
├── usage_today: Integer, DEFAULT 0               -- Счётчик использования (сбрасывается ежедневно)
├── usage_month: Integer, DEFAULT 0               -- Счётчик использования (сбрасывается ежемесячно)
├── is_active: Boolean, DEFAULT True              -- Активный провайдер (только один может быть активным)
├── is_verified: Boolean, DEFAULT False           -- Ключ проверен тестовым запросом
├── is_background: Boolean, NOT NULL, DEFAULT 0   -- v4.9 (Б-07): «провайдер для фоновых задач» (§3.29, §5.16.1)
├── last_used_at: DateTime, NULLABLE
├── created_at: DateTime, NOT NULL
├── updated_at: DateTime, NOT NULL
├── INDEX: idx_ai_provider_user (user_id, is_active)
└── UNIQUE INDEX: uq_ai_provider_configs_background (is_background) WHERE is_background = 1   -- v4.9: не больше одной строки в системе
```

Цены `price_per_1m_prompt` / `price_per_1m_completion` (за 1 млн токенов; в схеме выше не показаны) с v4.9 читает учёт фоновых задач — когда провайдер не сообщил цену вызова сам (§5.16.1).

### 3.22 Таблица `revoked_tokens`

```
revoked_tokens
├── id: Integer, PK, autoincrement
├── jti: String(64), UNIQUE, NOT NULL            -- JWT ID из payload
├── user_id: Integer, FK(users.id), NOT NULL
├── revoked_at: DateTime, NOT NULL, DEFAULT utcnow
├── expires_at: DateTime, NOT NULL               -- Для cleanup: удаляется после истечения
└── INDEX: idx_revoked_jti (jti)
```

### 3.23 ER-диаграмма (текстовое описание ключевых связей)

```
users 1──* broker_accounts
users 1──* strategies
users 1──1 telegram_links
users 1──* notifications
users 1──* alerts                 -- v4.1 (A-03), §3.13; до v4.1 — price_alerts
alerts 1──* alert_triggers        -- v4.1 (A-03), §3.13
users 1──* audit_log
users 1──* chart_drawings
users 1──1 chart_layouts          -- v4.3 (Б-02), §3.25; ON DELETE CASCADE
users 1──* user_notification_settings
users 1──* revoked_tokens
users 1──* ai_provider_configs
users 1──* dividend_corrections   -- v4.5 (A-02), §3.27; created_by, ON DELETE SET NULL
users 1──* trading_stops          -- v4.7 (A-04), §3.28; не больше одной строки с disabled_at IS NULL; ON DELETE CASCADE
users 1──* circuit_breaker_events
circuit_breaker_events *──* trading_sessions  -- через circuit_breaker_event_sessions (v4.7, A-04, §3.28): сессии на паузе

strategies 1──* strategy_versions
strategies *──1 strategy_versions (current_version_id)

strategy_versions 1──* backtests
strategy_versions 1──* trading_sessions

backtests 1──* backtest_trades

trading_sessions 1──* live_trades
trading_sessions 1──1 paper_portfolios
trading_sessions 1──* daily_stats
trading_sessions *──1 broker_accounts
trading_sessions 1──* circuit_breaker_configs  -- лимиты сессии (session_id), ON DELETE CASCADE; глобальная строка — session_id NULL

(standalone — без FK к user)
ohlcv_cache
ohlcv_coverage          -- v4.0 (A-01), §3.5.2
ohlcv_dealer            -- v4.0 (A-01), §3.5.1
ohlcv_volume_check      -- v4.0 (A-01), §3.5.3
ohlcv_backfill_target   -- v4.0 (A-01), §3.5.4
ohlcv_lot_history       -- v4.0 (A-01, дефект 4), §3.5.5
trading_schedule_periods     -- v4.1 (A-03), §3.24
trading_schedule_empty_days  -- v4.1 (A-03), §3.24
bond_info_cache
corporate_actions
pending_events
news_sources 1──* news_items        -- v4.4 (Б-01), §3.26; лента общая, без FK к user
news_items 1──* news_labels         -- не больше одной на разметчика; ON DELETE CASCADE
news_labels 1──* news_tickers       -- бумаги разметки; ON DELETE CASCADE
news_items 1──* news_reactions      -- одна на пару «новость — тикер»; ON DELETE CASCADE
news_dictionary_terms               -- v4.4 (Б-01), §3.26
ticker_aliases          -- v4.5 (A-02), §3.5.6
dividend_payments       -- v4.5 (A-02), §3.27; общая для терминала
dividend_smartlab_rows  -- v4.5 (A-02), §3.27
dividend_sources        -- v4.5 (A-02), §3.27; одна строка на бумагу
```

### 3.24 Таблицы расписания торгов `trading_schedule_periods`, `trading_schedule_empty_days` (v4.1, A-03)

```
trading_schedule_periods                          -- таблица периодов расписания (`scheduler/models.py::TradingSchedulePeriod`)
├── id: Integer, PK, autoincrement
├── trade_date: Date, NOT NULL                    -- дата МСК
├── kind: String(20), NOT NULL                    -- `PeriodKind`: opening_auction | trading | closing_auction | evening | dealer
├── start_at: DateTime, NOT NULL                  -- наивное UTC, включительно
├── end_at: DateTime, NOT NULL                    -- наивное UTC, исключительно (конец T-Invest включителен до секунды — +1 с)
├── source: String(10), NOT NULL                  -- 'tinvest' | 'iss'
├── updated_at: DateTime, NOT NULL
├── UNIQUE: uq_trading_schedule_period (trade_date, kind, start_at)
└── INDEX: idx_trading_schedule_date (trade_date)

trading_schedule_empty_days                       -- даты, где ISS ответил без ошибки, но без минут
├── id: Integer, PK, autoincrement
├── trade_date: Date, NOT NULL
├── source: String(10), NOT NULL                  -- 'iss'
├── checked_at: DateTime, NOT NULL
└── UNIQUE: uq_trading_schedule_empty_day (trade_date)
```

Строка — один период дня; в дне от одного до пяти периодов (в выходной с торгами — два дилерских). Пишут только загрузчики `scheduler/trading_schedule.py` (`save_days(…, keep_tinvest=)`: строки T-Invest замещают строки ISS той же даты, ISS строки T-Invest не трогает); читает только `TradingSchedule.load` при старте и после каждой синхронизации (§5.9.1). `trading_schedule_empty_days` — около 700 выходных до 2025 года и будни без торгов (перерыв 25.02–23.03.2022): без неё каждый старт заново запрашивал бы их у ISS. Таблицы создаёт ревизия `d92b7dd5c939`; посева нет — заполняют фоновые задачи после старта. `downgrade` удаляет обе (кэш расписания перезаполнится).

### 3.25 Таблица `chart_layouts` (v4.3, Б-02)

Раскладка страницы графика пользователя — одним JSON-документом (ADR 0024; модель `app/chart_layouts/models.py::ChartLayout`).

```
chart_layouts
├── id: Integer, PK, autoincrement
├── user_id: Integer, FK(users.id) ON DELETE CASCADE, NOT NULL
├── layout_json: Text, NOT NULL      -- ChartLayoutDoc (§4.13), model_dump_json после валидации
├── created_at: DateTime, NOT NULL, server_default CURRENT_TIMESTAMP
├── updated_at: DateTime, NULLABLE, server_default CURRENT_TIMESTAMP
└── UNIQUE: uq_chart_layouts_user (user_id)
```

Одна строка на пользователя; запись — upsert `INSERT … ON CONFLICT(user_id) DO UPDATE` (`ChartLayoutService.save`, `persist_with_retry`, 3 попытки на «database is locked»): побеждает последний `PUT`, слияния нет. Строку, которую нынешняя схема не принимает (битый JSON, версия ≠ 1), `ChartLayoutService.get` отдаёт как «раскладки нет» с WARNING `chart_layout_invalid_stored`; первый `PUT` её заменит. Таблицу создаёт ревизия `0acf47e6607f` (идемпотентно, вместе со сменой индекса рисунков, §3.14); посева нет — строка появляется при первом изменении раскладки. `downgrade` удаляет таблицу вместе со всеми раскладками.

### 3.26 Таблицы новостей `news_*` (v4.4, Б-01)

Архив новостей, разметка, связи с бумагами, реакции цены и словарь (модели `app/news/models.py`; ADR 0101, 0102, 0104; требования — ФТ §20). Даты — `DateTime` без зоны, значение naive UTC (соглашение проекта); в JSON — с «Z», показ — МСК. Таблицы не привязаны к пользователю: лента общая для всех вошедших.

```
news_sources                                  -- источник: адрес ленты и состояние опросов
├── id: Integer, PK, autoincrement
├── code: String(32), NOT NULL                -- interfax | prime | tass_econ | kommersant_biz | smartlab | moex_iss | cbr
├── name: String(100), NOT NULL               -- подпись в фильтре ленты
├── kind: String(20), NOT NULL                -- 'rss' | 'iss_sitenews' — каким разбором читать ответ
├── url: Text, NOT NULL                       -- адрес ленты; в коде адресов нет
├── enabled: Boolean, NOT NULL, server_default '1'
├── last_polled_at: DateTime, NULLABLE
├── last_success_at: DateTime, NULLABLE
├── last_error: Text, NULLABLE                -- текст ПОСЛЕДНЕГО сбоя (≤ 1000 символов); успех его не стирает
├── last_error_at: DateTime, NULLABLE
├── consecutive_failures: Integer, NOT NULL, server_default '0'   -- 0 — источник отвечает
└── UNIQUE: uq_news_sources_code (code)

news_items                                    -- новость: только заголовок, ссылка и два времени
├── id: Integer, PK, autoincrement
├── source_id: Integer, FK(news_sources.id), NOT NULL
├── external_id: String(512), NOT NULL        -- guid записи RSS (нет guid — ссылка); ISS — id сообщения
├── title: Text, NOT NULL                     -- HTML-сущности раскрыты
├── url: Text, NOT NULL                       -- только http(s); ISS — https://www.moex.com/n<id>
├── published_at: DateTime, NOT NULL          -- из ленты: RFC 822 → UTC; ISS — МСК → UTC
├── received_at: DateTime, NOT NULL           -- время прохода сборщика
├── UNIQUE: uq_news_items_source_external (source_id, external_id)
└── INDEX: idx_news_items_published (published_at)

news_labels                                   -- разметка: не больше одной на разметчика
├── id: Integer, PK, autoincrement
├── news_id: Integer, FK(news_items.id) ON DELETE CASCADE, NOT NULL
├── labeler: String(20), NOT NULL             -- 'dictionary' | 'ai'
├── model: String(100), NULLABLE              -- ИИ (Б-07); в Б-01 — NULL
├── prompt_version: String(50), NULLABLE      -- ИИ; NULL
├── scope: String(20), NULLABLE               -- company | sector | market; NULL
├── news_type: String(20), NOT NULL           -- report | dividends | deal | sanctions | regulator | management | exchange | other; словарь — other
├── sentiment: String(20), NULLABLE           -- positive | negative | neutral | mixed; NULL
├── importance: SmallInteger, NULLABLE        -- 1–3; NULL
├── labeled_at: DateTime, NOT NULL, server_default CURRENT_TIMESTAMP
└── UNIQUE: uq_news_labels_news_labeler (news_id, labeler)

news_tickers                                  -- связь «новость — бумага» в рамках разметки
├── id: Integer, PK, autoincrement
├── news_id: Integer, FK(news_items.id) ON DELETE CASCADE, NOT NULL
├── label_id: Integer, FK(news_labels.id) ON DELETE CASCADE, NOT NULL
├── ticker: String(20), NOT NULL
├── UNIQUE: uq_news_tickers_label_ticker (label_id, ticker)
└── INDEX: idx_news_tickers_ticker_news (ticker, news_id)

news_reactions                                -- реакция пары «новость — бумага»
├── id: Integer, PK, autoincrement
├── news_id: Integer, FK(news_items.id) ON DELETE CASCADE, NOT NULL
├── ticker: String(20), NOT NULL
├── due_at: DateTime, NOT NULL                -- published_at + 75 мин (час «после» + 15 мин)
├── status: String(20), NOT NULL              -- 'pending' | 'done' | 'failed'
├── before_pct: Numeric(10,4), NULLABLE       -- «до», %; NULL — нет торгов или посчитать нечего
├── after_pct: Numeric(10,4), NULLABLE        -- «после», %
├── before_no_trades: Boolean, NOT NULL, server_default '0'
├── after_no_trades: Boolean, NOT NULL, server_default '0'
├── attempts: Integer, NOT NULL, server_default '0'   -- неудачные обращения к ISS
├── last_error: Text, NULLABLE
├── computed_at: DateTime, NULLABLE
├── UNIQUE: uq_news_reactions_news_ticker (news_id, ticker)
└── INDEX: idx_news_reactions_status_due (status, due_at)

news_dictionary_terms                         -- псевдонимы и исключения словаря
├── id: Integer, PK, autoincrement
├── kind: String(20), NOT NULL                -- 'alias' | 'exclusion'
├── term: String(200), NOT NULL               -- фраза
├── ticker: String(20), NULLABLE              -- alias — бумага; exclusion — бумага или NULL (гасит любую)
├── note: Text, NULLABLE
└── UNIQUE: uq_news_dictionary_terms_kind_term_ticker (kind, term, ticker)
```

Пишут: сборщик — `news_items` (`INSERT … ON CONFLICT DO NOTHING`) и строку своего источника (§5.14.1); разметка словарём — `news_labels` и `news_tickers` (§5.14.2); `ReactionWorker` — `news_reactions` (§5.14.3). Сборщик ничего не удаляет: новость, ушедшая из ленты, остаётся. Разметка без строк `news_tickers` — «размечено, бумаг нет»; «ещё не размечено» — нет строки с `labeler = 'dictionary'`. Бумаги новости — объединение бумаг её разметок; реакция держится за пару (`news_id`, `ticker`) и не знает, чья разметка назвала бумагу (ADR 0102). Поля ИИ (`model`, `prompt_version`, `scope`, `sentiment`, `importance`) заполнит Б-07 — единственное исключение из правила «всё новое вызывается в рабочем коде».

Таблицы создаёт ревизия `b01e7a5c3d92` (поверх `0acf47e6607f`; при ветвлении блока родителем была `a7c1d9e2b4f8`, при мержах с `develop` ревизия перевешивалась). Посев: семь строк `news_sources` — порядок строк = порядок фильтра ленты; адреса: `https://www.interfax.ru/rss`, `https://1prime.ru/export/rss2/index.xml`, `https://tass.ru/rss/v2.xml?sections=MjU%3D` (раздел «Экономика и бизнес»), `https://www.kommersant.ru/rss/section-business.xml`, `https://smart-lab.ru/news/rss/`, `https://iss.moex.com/iss/sitenews.json?iss.meta=off`, `https://www.cbr.ru/rss/eventrss`; 217 строк `news_dictionary_terms` — 187 `alias` и 30 `exclusion` (`_ALIASES` / `_EXCLUSIONS` ревизии, в том числе `("glorax", "GLRX")`, добавленный по тестовому стенду). `downgrade` удаляет все шесть таблиц вместе с архивом. Модель и миграция совпадают буквально (`alembic check`); `upgrade` → `downgrade 0acf47e6607f` — `tests/test_news/test_collector.py::test_migration_creates_and_drops_news_tables`, вся цепочка туда и обратно — `tests/unit/test_audit_s8r_alembic.py::test_full_round_trip_upgrade_downgrade_upgrade`, одна голова — `tests/unit/test_migration.py::test_b01_revision_is_single_head`.

Ограничения: `uq_news_dictionary_terms_kind_term_ticker` не ловит дубль общего исключения (`ticker` NULL — в SQLite NULL различны), перед вставкой проверять `SELECT`; правка ревизии `b01e7a5c3d92` не дойдёт до БД, где она уже применена, — новый источник или термин — новой ревизией или записью в БД. Очистки архива нет (срока хранения в блоке нет); объём — ≈ 508 Б на новость вместе с разметкой, связями, реакциями и индексами, ≈ 0,48 МБ в сутки при ≈ 950 новостях (тестовый стенд 05–06.11; `deployment_guide.md` §7).

**С v4.9 (Б-07).** У `news_items` три новые колонки (`ai_attempts`, `ai_note`, `alert_stage`), разметку `labeler = 'ai'` пишет шаг ИИ-разметки (§5.16.2); бумаги новости — по **действующей** разметке: `ai`, если есть, иначе `dictionary` (ADR 0060 заменяет правило «объединение» ADR 0102 в части бумаг новости; реакции по-прежнему заводятся для пар всех разметок). Ревизия `b07d34cd8897` добавляет 11 исключений `news_dictionary_terms`; на БД с посевом Б-01 без ручных правок — 228 строк. Схема — §3.29.

### 3.27 Таблицы дивидендов `dividend_*` (v4.5, A-02)

История выплат по бумаге, ручные поправки, строки Смартлаба для сверки и состояние источников (модели `app/dividends/models.py`; ADR 0036; требования — ФТ §21). Деньги на акцию — `Numeric(18,8)` (в JSON — строкой, gotcha-01); даты — `Date`; моменты загрузки — `DateTime` без зоны, naive UTC. Тикер — `SECID` режима TQBR в верхнем регистре. Журнал начислений `corporate_actions` (§3.19) не меняется.

```
dividend_payments                             -- выплаты T-Invest `GetDividends`
├── id: Integer, PK, autoincrement
├── ticker: String(20), NOT NULL
├── record_date: Date, NOT NULL               -- дата реестра
├── last_buy_date: Date, NULLABLE             -- последний день покупки (как у T-Invest)
├── payment_date: Date, NULLABLE
├── declared_date: Date, NULLABLE
├── amount: Numeric(18,8), NOT NULL           -- на акцию; выплаты одной даты в одной валюте — сумма
├── currency: String(10), NOT NULL
├── dividend_type: String(30), NULLABLE       -- как у T-Invest (≤ 30 символов); 'Cancelled' не хранится
├── regularity: String(30), NULLABLE
├── loaded_at: DateTime, NOT NULL
└── UNIQUE: uq_dividend_payments (ticker, record_date)

dividend_corrections                          -- ручные поправки администратора
├── id: Integer, PK, autoincrement
├── ticker: String(20), NOT NULL
├── record_date: Date, NOT NULL               -- ключ поправки: исходная дата реестра
├── action: String(10), NOT NULL              -- 'add' | 'exclude' | 'fix'
├── amount: Numeric(18,8), NULLABLE           -- add — обязательна; fix — новая сумма; exclude — NULL
├── new_record_date: Date, NULLABLE           -- только fix
├── last_buy_date: Date, NULLABLE             -- add / fix
├── comment: Text, NOT NULL                   -- обязателен, ≤ 1000 символов (схема API)
├── created_by: Integer, FK(users.id) ON DELETE SET NULL, NULLABLE
├── created_at / updated_at: DateTime         -- TimestampMixin; дата поправки
└── UNIQUE: uq_dividend_corrections (ticker, record_date)

dividend_smartlab_rows                        -- таблица «Выплаченные дивиденды» Смартлаба (только для сверки)
├── id: Integer, PK, autoincrement
├── ticker: String(20), NOT NULL
├── record_date: Date, NOT NULL               -- «дата отсечки»
├── last_buy_date: Date, NULLABLE             -- «дата T-1»
├── period: String(50), NULLABLE              -- «Период»; строки одной даты — через «; »
├── amount: Numeric(18,8), NOT NULL           -- строки одной даты реестра — сумма
├── currency: String(10), NOT NULL
├── canceled: Boolean, NOT NULL, server_default '0'   -- Смартлаб сам отметил отменённой (все строки даты)
└── UNIQUE: uq_dividend_smartlab_rows (ticker, record_date, amount)

dividend_sources                              -- состояние источников по бумаге
├── id: Integer, PK, autoincrement
├── ticker: String(20), NOT NULL
├── tinvest_loaded_at: DateTime, NULLABLE     -- последняя удачная загрузка T-Invest
├── smartlab_checked_at: DateTime, NULLABLE   -- последняя попытка сверки (и неудачная)
├── smartlab_status: String(20), NULLABLE     -- 'ok' | 'unavailable'
├── smartlab_ok_at: DateTime, NULLABLE        -- последняя удачная сверка
├── smartlab_error: String(200), NULLABLE     -- причина «сверка недоступна»
└── UNIQUE: uq_dividend_sources_ticker (ticker)
```

Пишут: `load_tinvest_dividends` — `dividend_payments` (удаление строк бумаги и вставка ответа одной транзакцией, `persist_with_retry_fresh`) и `dividend_sources.tinvest_loaded_at`; `check_smartlab` — `dividend_smartlab_rows` (замена строк бумаги) или только `dividend_sources` (`unavailable`, строки не трогаются); `save_correction` / `delete_correction` — `dividend_corrections` и `audit_log` в одной транзакции (§5.15). Расхождения не хранятся — считаются при чтении (ADR 0036).

Таблицы создаёт ревизия `79f3c238dbb8` (поверх `b01e7a5c3d92`; при ветвлении блока родителем была `a858647f24e6`, при мержах с `develop` ревизия перевешивалась на `cdb15f21e5ca`, `0acf47e6607f`, `b01e7a5c3d92`) вместе с `ticker_aliases` (§3.5.6) и признаком счёта (§3.2). Колонку `dividend_smartlab_rows.canceled` в уже созданную ранней сборкой той же ревизии таблицу добавляет `batch_alter_table` (gotcha-12, 79). Ревизия идемпотентна; data-часть — §5.6.8 и `deployment_guide.md` §7. `downgrade` снимает индекс и колонку счёта и удаляет пять таблиц вместе с поправками; удалённые data-частью свечи не возвращаются. Модель и миграция совпадают (`alembic check`); схема, data-часть, `downgrade` к `b01e7a5c3d92` и одна голова — `tests/test_market_data/test_a02_migration.py`, колонка отмены — `tests/unit/test_dividends/test_canceled_migration.py`, вся цепочка туда и обратно — `tests/unit/test_audit_s8r_alembic.py::test_full_round_trip_upgrade_downgrade_upgrade`.

### 3.28 Журнал «Стоп всему» и таблицы Circuit Breaker (v4.7, A-04)

Модели — `app/trading/models.py::TradingStop`, `app/circuit_breaker/models.py` (`CircuitBreakerEvent`, `CircuitBreakerEventSession`, `CircuitBreakerConfig`); ревизия `5613e921bcc1` (поверх `b6b5064c0be6`); ADR 0040, 0041, 0042.

```
trading_stops                                    -- журнал «Стоп всему»: строка — эпизод
├── id: Integer, PK, autoincrement
├── user_id: Integer, FK(users.id) ON DELETE CASCADE, NOT NULL
├── enabled_at: DateTime, NOT NULL               -- наивное UTC; ставит StopAllService
├── enabled_source: String(10), NOT NULL         -- 'web' | 'telegram'
├── disabled_at: DateTime, NULLABLE              -- NULL — стоп включён (флаг «торговля остановлена»)
├── closed_count: Integer, NULLABLE              -- N отчёта включения; NULL — закрытие идёт или не дошло
├── total_count: Integer, NULLABLE               -- M
├── report: Text, NULLABLE                       -- JSON StopAllReport (§4.4)
├── INDEX: idx_trading_stops_user (user_id)
└── UNIQUE INDEX: uq_trading_stops_user_active (user_id) WHERE disabled_at IS NULL

circuit_breaker_event_sessions                   -- сессии, которые событие CB (и «Стоп всему») поставило на паузу
├── event_id: Integer, FK(circuit_breaker_events.id) ON DELETE CASCADE, PK
├── session_id: Integer, FK(trading_sessions.id) ON DELETE CASCADE, PK
└── INDEX: idx_cb_event_sessions_session (session_id)

circuit_breaker_events
├── id: Integer, PK, autoincrement
├── user_id: Integer, FK(users.id), NOT NULL
├── event_type: String(50), NOT NULL             -- тип проверки; 'stop_all' — «Стоп всему» (A-04)
├── trigger_value, limit_value: Numeric(18,4), NULLABLE   -- смешанные единицы (S8R-FIX-039); у 'stop_all' — N и M
├── action_taken: String(50), NOT NULL           -- all_sessions_paused | money_class_sessions_paused | session_paused
├── session_id: Integer, FK(trading_sessions.id) ON DELETE SET NULL, NULLABLE  -- сессия-источник; у 'stop_all' — NULL
├── details: Text, NULLABLE
├── created_at: DateTime, NOT NULL, DEFAULT now
├── session_limit: Boolean, NOT NULL, server_default '0'   -- A-04: сработал собственный лимит сессии
└── close_report: Text, NULLABLE                 -- A-04 (G02): JSON CBCloseReport {closed_count, total_count, items[]}

circuit_breaker_configs                          -- схема не менялась
├── id: Integer, PK; user_id: FK(users.id), NOT NULL
├── session_id: Integer, FK(trading_sessions.id) ON DELETE CASCADE, NULLABLE   -- NULL — общие; id — лимиты сессии
├── daily_loss_limit_pct, max_drawdown_pct: Numeric(10,4); daily_loss_limit_fixed, max_position_size: Numeric(18,2)
├── daily_trade_limit, cooldown_seconds: Integer; trading_hours_start, trading_hours_end: String(5)   -- все NULLABLE
├── created_at, updated_at
├── UNIQUE: uq_cb_config_user_session (user_id, session_id)
└── INDEX: idx_cb_config_user (user_id)
```

- **Флаг** — строка `trading_stops` с `disabled_at IS NULL`; читают `stop_all.get_active_stop` / `is_trading_stopped` из БД при каждой проверке, кэша нет. Пишет только `StopAllService` (§5.4.7); гонка двух процессов отсекается частичным уникальным индексом (`IntegrityError` → ответ состоянием).
- **`circuit_breaker_events` у «Стоп всему»:** `event_type='stop_all'`, `action_taken='all_sessions_paused'`, `session_id` NULL, `details` «„Стоп всему“ включён из интерфейса / из Telegram», после закрытия `trigger_value`/`limit_value` = N/M и `close_report`. Связи пишут `_trigger` (по `affected_session_ids`) и «Стоп всему» тем же commit, что пауза.
- **`close_report`** — только незакрытые позиции: `items[] = {trade_id, session_id, ticker, reason, queued}` (`reason` — `close_failure_reason`, `queued` — `exit_retry_kind IS NOT NULL`). Пишут `_trigger` после принудительного закрытия и «Стоп всему» на шаге отчёта; у `broker_rejection_streak` (`CB_PAUSE_WITHOUT_CLOSE`) отчёта нет.
- **Старые события** (до миграции) связей не получают, `session_limit = false`, `close_report` NULL. У событий до S8R-FIX-039 `trigger_value`/`limit_value` = 0: история отдаёт их как `null` (порог CB нулём не бывает; у `stop_all` 0 — честное «позиций 0»).
- **`circuit_breaker_configs` с `session_id`** с A-04 — собственные лимиты сессии: пустое поле = значение общих (§5.8, ADR 0041; до A-04 строка сессии заменяла общие целиком). Пишут `PUT /circuit-breaker/config/session/{id}` и запуск сессии с `cb_limits` (тем же commit, что сессия); `DELETE /config/session/{id}` удаляет все строки сессии, включая дубли. В копии рабочей БД (05.11) таких строк 0.

### 3.29 Таблицы Б-07: фоновые задачи ИИ, настройки «Новости», проверка на эталоне (v4.9, Б-07)

Модели — `app/common/models.py` (`AIBackgroundSettings`, `AIBackgroundUsage`, `AIProviderConfig.is_background`), `app/news/models.py` (`NewsSettings`, `NewsAIBenchmark`, новые колонки `NewsItem`), `app/market_data/models.py` (`Alert`, `AlertTrigger` — §3.13). Требования — ФТ §20.8–§20.14; решения — ADR 0061–0065. Даты — naive UTC (соглашение проекта); сутки учёта — МСК.

```
ai_background_settings                        -- одна строка id = 1 (засев миграцией)
├── id: Integer, PK, autoincrement
├── daily_token_limit: BigInteger, NOT NULL, server_default 2000000   -- токены (вход + выход) всех фоновых задач за сутки МСК
├── updated_at: DateTime, NULLABLE
└── updated_by: Integer, FK(users.id) ON DELETE SET NULL, NULLABLE

ai_background_usage                           -- расход фоновых задач по суткам МСК
├── id: Integer, PK, autoincrement
├── usage_date: Date, NOT NULL                -- сутки МСК
├── config_id: Integer, FK(ai_provider_configs.id) ON DELETE SET NULL, NULLABLE   -- строку удалили — расход прошлых суток остаётся
├── model_id: String(100), NOT NULL
├── task: String(30), NOT NULL                -- 'news_labeling' | 'news_benchmark'
├── calls: Integer, NOT NULL, server_default 0
├── prompt_tokens: BigInteger, NOT NULL, server_default 0
├── completion_tokens: BigInteger, NOT NULL, server_default 0
├── cost_usd: Numeric(18,8), NOT NULL, server_default 0   -- исключение из «деньги Numeric(18,2)»: доли цента на вызов (ADR 0061)
└── UNIQUE: uq_ai_background_usage_day (usage_date, config_id, model_id, task)

news_settings                                 -- одна строка id = 1 (засев миграцией): выключатели раздела «Новости»
├── id: Integer, PK, autoincrement
├── collect_enabled: Boolean, NOT NULL, server_default 1       -- «Загрузка и обработка новостей»
├── ai_labeling_enabled: Boolean, NOT NULL, server_default 0   -- «ИИ-разметка»; выключена до проверки на эталоне
├── alerts_enabled: Boolean, NOT NULL, server_default 1        -- «Алерты по новостям»
├── chart_marks_enabled: Boolean, NOT NULL, server_default 1   -- «Метки новостей на графике»
├── alerts_enabled_since: DateTime, NULLABLE  -- последнее включение алертов; засев — время миграции
├── updated_at: DateTime, NULLABLE
└── updated_by: Integer, FK(users.id) ON DELETE SET NULL, NULLABLE

news_ai_benchmarks                            -- прогон проверки модели на эталоне
├── id: Integer, PK, autoincrement
├── config_id: Integer, FK(ai_provider_configs.id) ON DELETE SET NULL, NULLABLE
├── provider_type: String(20), NOT NULL
├── model_id: String(100), NOT NULL
├── prompt_version: String(50), NOT NULL      -- связка (config_id, model_id, prompt_version)
├── status: String(10), NOT NULL              -- 'running' | 'done' | 'error'
├── passed: Boolean, NULLABLE
├── precision_pct: Numeric(10,4), NULLABLE
├── recall_hits: Integer, NULLABLE
├── recall_total: Integer, NULLABLE
├── out_of_list: Integer, NULLABLE            -- тикеры вне списка по всем ответам, в том числе отброшенным
├── controls_ok: Boolean, NULLABLE            -- № 19, 40, 71
├── result: JSON, NULLABLE                    -- ошибки по заголовкам, таблицы типа и тональности, контрольные, вызовы с задержками
├── calls: Integer, NOT NULL, server_default 0
├── prompt_tokens / completion_tokens: BigInteger, NOT NULL, server_default 0
├── cost_usd: Numeric(18,8), NOT NULL, server_default 0
├── error: Text, NULLABLE
├── started_by: Integer, FK(users.id) ON DELETE SET NULL, NULLABLE
├── started_at: DateTime, NOT NULL
├── finished_at: DateTime, NULLABLE
└── INDEX: idx_news_ai_benchmarks_binding (config_id, model_id, prompt_version, started_at)

news_items (+ v4.9)
├── ai_attempts: SmallInteger, NOT NULL, server_default 0      -- неудачные попытки ИИ-разметки
├── ai_note: String(20), NULLABLE             -- почему действует словарь: disabled | no_provider | not_verified | limit | unavailable | invalid; NULL — ИИ разметил или не пробовал
├── alert_stage: String(10), NOT NULL, server_default 'new'    -- new → partial → done; skipped — алерты выключены
└── INDEX: idx_news_items_alert_stage (alert_stage)
```

- **Разметка ИИ** пишется в существующие `news_labels` / `news_tickers` (§3.26): строка `labeler = 'ai'` с `model`, `prompt_version`, `scope` (NULL — «не про рынок»), `news_type`, `sentiment` (только при `scope = 'company'`), `importance` 1–3; новая разметка ИИ новости перезаписывает прежнюю. Схема `news_labels` не менялась.
- **`is_background`** — не больше одной строки в системе (частичный уникальный индекс, как `broker_accounts.is_market_data_source`, ADR 0035); действует только на строке первого администратора (§5.16.1).
- **`alerts_enabled_since`** засевается временем миграции, а все существующие `news_items` получают `alert_stage = 'done'` — архив до выкатки алерты не проверяют.

**Ревизии.** `b07a1e5c0d42` (поверх `5613e921bcc1`, A-04) — вся схема блока: колонка и индекс `ai_provider_configs`, четыре таблицы выше с засевом двух строк, три колонки и индекс `news_items` (UPDATE архива в `done`), колонки `alerts` и `alert_triggers` (§3.13). Таблицы и колонки — идемпотентно, через инспектор; колонки — `batch_alter_table` (gotcha-12), FK на время миграций выключены (`env.py`, gotcha-79). `downgrade` — в обратном порядке: удаляет алерты `news` и срабатывания без цены, возвращает NOT NULL, удаляет четыре таблицы (учёт расхода, лимит, выключатели, прогоны проверки), колонки `news_items` и `is_background`; уровневые алерты и их срабатывания целы, строки `news_labels` с `labeler = 'ai'` откат не трогает. Замер на копии рабочей БД (2026-11-10): `upgrade` 1,42 с, `downgrade` 1,39 с, повторный `upgrade` 1,39 с (≈ 1,22 с из них — импорт приложения, сама ревизия ≈ 0,2 с), размер файла не изменился (114 696 192 Б); `alembic check` — ok. `b07d34cd8897` (поверх `b07a1e5c0d42`, доработка 2026-11-12) — **только данные**: 11 строк `news_dictionary_terms` `kind = 'exclusion'` (AKRN — «фк акрон», «акрон тольятти» и падежи «акрона / акрону / акроном / акроне тольятти», «обыграл / обыграла / обыграло / обыграли акрон»; GAZP — «газпром медиа»), вставка `INSERT … WHERE NOT EXISTS` (строка, заведённая вручную, не дублируется), `downgrade` удаляет только строки с пометкой `note` ревизии. Отдельно не замерялась. **Head — `b07d34cd8897`.** Round-trip обеих — `tests/unit/test_migration.py` (в том числе `test_b07_rows_survive_round_trip_and_news_alerts_dropped_on_downgrade`, `test_b07_dictionary_exclusions_round_trip`).

### 3.30 Таблицы A-06: журнал исполнений, начисления дивидендов, денежный рынок (v4.10, A-06)

Модели — `app/trading/models.py` (`ExecutionJournalEntry`, `DividendAccrual`, колонки `TradingSession`), `app/backtest/models.py` (`MoneyMarketQuote`, колонки `Backtest` и `BacktestTrade`); требования — ФТ §22; решения — ADR 0072 (журнал), 0074 (начисления), 0078 (денежный рынок), 0079 (версия правил).

```
execution_journal                             -- журнал исполнений: строка на каждое исполнение paper, sandbox, real
├── id: Integer, PK, autoincrement
├── session_id: Integer, FK(trading_sessions.id, ondelete=CASCADE), NOT NULL
├── trade_id: Integer, FK(live_trades.id, ondelete=CASCADE), NOT NULL
├── mode: String(10), NOT NULL                -- режим сессии: paper | sandbox | real
├── side: String(10), NOT NULL                -- 'entry' | 'exit'
├── lots: Integer, NOT NULL                   -- исполненные лоты (частичный выход — исполненная часть)
├── executed_at: DateTime, NOT NULL           -- naive UTC: начало минуты исполнения paper / OrderStatus.executed_at брокера / момент записи
├── rule: String(20), NOT NULL                -- ExecutionRule: bar_open | signal | stop_level | take_level | gap | manual | forced
├── rule_price: Numeric(18, 8), NULLABLE      -- цена правила: open минуты, уровень, open гэпа, текущая цена / цена сигнала, оценка рынка
├── slippage_value: Numeric(10, 4), NULLABLE  -- проскальзывание сессии на момент исполнения (sandbox/real — 0)
├── slippage_mode: String(10), NULLABLE       -- 'pct' | 'points'
├── contract_price: Numeric(18, 8), NULLABLE  -- цена по договору: paper — цена правила со сдвигом; sandbox/real — = rule_price
├── actual_price: Numeric(18, 8), NULLABLE    -- фактическая: paper — = contract_price; sandbox/real — цена брокера (нет — NULL)
├── price_diff: Numeric(18, 8), NULLABLE      -- actual − contract (paper — 0)
├── partial: Boolean, NOT NULL, DEFAULT 0     -- довыполнение остатка входа (первая строка частичного входа — false)
├── created_at: DateTime, NOT NULL, DEFAULT CURRENT_TIMESTAMP
├── INDEX: idx_ej_trade (trade_id)
└── INDEX: idx_ej_session (session_id, executed_at)

dividend_accruals                             -- «дивиденд зачислен по сделке»: идемпотентность начислений paper и учёта
├── id: Integer, PK, autoincrement
├── corporate_action_id: Integer, FK(corporate_actions.id, ondelete=CASCADE), NOT NULL
├── trade_id: Integer, FK(live_trades.id, ondelete=CASCADE), NOT NULL
├── session_id: Integer, FK(trading_sessions.id, ondelete=CASCADE), NOT NULL
├── amount: Numeric(18, 2), NOT NULL          -- зачислено, со знаком, после налога (шорт — минус полная сумма)
├── tax_pct: Numeric(10, 4), NOT NULL         -- налог сессии на момент зачисления
├── gap_date: Date, NOT NULL                  -- день гэпа (МСК)
├── credited_at: DateTime, NOT NULL
├── UNIQUE: uq_dividend_accrual_action_trade (corporate_action_id, trade_id)
└── INDEX: idx_da_session (session_id)

money_market_quotes                           -- кэш денежного рынка: строка на источник и календарный день
├── id: Integer, PK, autoincrement
├── source: String(10), NOT NULL              -- 'LQDT' (цена закрытия дня ISS) | 'RUONIA' (ставка ЦБ, % годовых)
├── day: Date, NOT NULL                       -- у дня без торгов или публикации — значение последнего дня до него
├── value: Numeric(18, 8), NOT NULL
├── loaded_at: DateTime, NOT NULL
└── UNIQUE: uq_mmq_source_day (source, day)
```

- **Колонки существующих таблиц** — §3.6 (`backtests`: `execution_rules_version`, `dividend_tax_pct`, `price_pnl`, `dividend_pnl`, `bars_per_year`, `money_market_pct`, `money_market_source`, `report_notes`), §3.7 (`backtest_trades.row_kind`), §3.8 (`trading_sessions.slippage_value`, `slippage_mode`, `dividend_tax_pct`); `live_trades.slippage` получил писателя (§3.9).
- **Журнал** пишет только `trading/execution_journal.py` (`record_execution`, `record_broker_entry`, `fill_entry_actual_price`) — `db.add` в транзакции сделки, без своего commit (§5.17.5). Прошлые сделки не заполняются.
- **Начисления**: строку пишет `CorporateActionService` при зачислении (рантайм paper или задача корпоративных действий); UNIQUE исключает повтор; `corporate_actions.processed` ставится, когда незачисленных позиций по действию не осталось (§5.17.6).
- **Денежный рынок**: «день загружен» = «строка есть», поэтому повторный расчёт периода в сеть не ходит; свежие дни без данных (младше 7 дней после последнего дня с данными) в кэш не пишутся (§5.17.8).

**Ревизия.** `8bddf72586cf` (поверх `b07d34cd8897` — доработка Б-07; при сборке 2026-11-10 стояла на `5613e921bcc1`, перевешена при мерже `develop` 13.11, содержимое не менялось) — одна на блок: три таблицы, колонки `trading_sessions` (NOT NULL с константным DEFAULT: `0` / `pct` / `0`), `backtests` (все NULL, без `server_default` — иначе `ADD COLUMN` выставил бы версию правил старым строкам), `backtest_trades.row_kind` (NOT NULL DEFAULT `trade`). Таблицы и колонки — идемпотентно через инспектор, колонки — `batch_alter_table` (gotcha-12); на SQLite `upgrade` — нативный `ADD COLUMN`, `downgrade` пересоздаёт `backtest_trades`, `backtests`, `trading_sessions` (FK на время миграций выключены, `env.py`, gotcha-79 — каскада на сделки, портфели и журнал нет). Данные не переносит и не удаляет: бэктесты до ревизии — версия правил NULL («до договора исполнения»), сессии — 0 / `pct` / 0, сделки бэктестов — `trade`. `downgrade` — в обратном порядке, теряет журнал, начисления и кэш; **строки `row_kind = 'dividend'` в `backtest_trades` откат не удаляет** — код до A-06 покажет их сделками. Замер на копии рабочей БД (2026-11-10, сборка): процесс `upgrade` — 1,3 с (столько же занимает `alembic current`, сама ревизия — доли секунды), `downgrade` — 1,5 с, размер файла не изменился (114 696 192 Б); помечено 65 бэктестов из 65, сделок бэктестов 2 986 — все `trade`, сессий 10 — 0 / `pct` / 0, строк журнала 0; `alembic check` и `PRAGMA integrity_check` — ok. Round-trip — `tests/unit/test_migration.py` (раздел A-06).

### 3.31 Справочник вселенной скринера и таблицы Б-03 (v4.11, Б-03)

Модели — `app/market_data/models.py` (новые колонки `Instrument`), `app/screener/models.py` (`SectorOverride`, `IssDayTotal`, `ScreenerUserSettings`, константы `BROKER_*`); требования — ФТ §23; решения — ADR 0090 (вселенная и статус у брокера — колонки `instruments`), 0091 (итоги дня ISS), 0093 (поправки секторов), 0095 (порог пользователя). Даты — naive UTC (соглашение проекта); `trade_date` — дата торгов ISS.

```
instruments (+ v4.11)                         -- справочник лотов и FIGI — он же справочник вселенной (ADR 0090)
├── short_name: String(100), NULLABLE         -- SHORTNAME ISS
├── list_level: SmallInteger, NULLABLE        -- LISTLEVEL ISS (1–3)
├── in_screener_universe: Boolean, NOT NULL, server_default 0   -- акция доски TQBR, INSTRID = EQIN; ушла с доски — 0, строка остаётся
├── universe_synced_at: DateTime, NULLABLE    -- синхронизация, подтвердившая или снявшая признак
├── broker_status: String(16), NULLABLE       -- 'trading' | 'break' (только break_in_trading) | 'not_on_tqbr'; NULL — не сверялось / нет счёта для рыночных данных
├── broker_qual_only: Boolean, NULLABLE       -- запись TQBR: только для квалифицированных
├── broker_short_enabled: Boolean, NULLABLE   -- запись TQBR: шорт доступен
├── broker_uid: String(64), NULLABLE          -- uid записи TQBR для GetLastPrices
├── broker_synced_at: DateTime, NULLABLE
└── iss_history_from: Date, NULLABLE          -- первое заполнение итогов дня ISS по бумаге сделано, с даты
    -- sector (была, не использовалась): код сектора T-Invest как пришёл (energy, financial, …, other; пусто → NULL)

screener_sector_overrides                     -- поправки секторов: общие для всех, важнее брокера (ADR 0093)
├── id: Integer, PK, autoincrement
├── ticker: String(20), NOT NULL              -- верхний регистр; UNIQUE uq_screener_sector_overrides_ticker
├── sector_code: String(32), NOT NULL         -- один из 10 кодов sectors.SECTOR_NAMES (проверяет маршрут)
├── updated_by_id: Integer, FK(users.id) ON DELETE SET NULL, NULLABLE
├── created_at: DateTime, NOT NULL, DEFAULT CURRENT_TIMESTAMP
└── updated_at: DateTime, NOT NULL, DEFAULT CURRENT_TIMESTAMP

screener_iss_day_totals                       -- итоги дня ISS history (TQBR) по акциям вселенной (ADR 0091)
├── ticker: String(20), PK
├── trade_date: Date, PK                      -- TRADEDATE ISS: торги выходного дня ISS относит к следующему рабочему дню (gotcha-94)
├── value: Numeric(18, 2), NULLABLE           -- VALUE, ₽
├── close: Numeric(18, 8), NULLABLE           -- CLOSE
└── INDEX: ix_screener_iss_day_totals_trade_date (trade_date)

screener_user_settings                        -- порог ликвидности пользователя (ADR 0095)
├── user_id: Integer, PK, FK(users.id) ON DELETE CASCADE
├── liquidity_threshold: Numeric(18, 2), NOT NULL   -- ₽; строки нет — DEFAULT_LIQUIDITY_THRESHOLD = 10 000 000
└── updated_at: DateTime, NOT NULL, DEFAULT CURRENT_TIMESTAMP
```

- **Пишет `instruments`** только синхронизация (`universe.sync_universe`): ISS-часть (имя, уровень, признак) — всегда; колонки `broker_*` и `sector` — только при выбранном счёте для рыночных данных. Лот, шаг цены, `lot_size_synced_at` она не пишет никогда; FIGI — только в пустое поле или при равенстве, расхождение — warning `screener_figi_mismatch` (только тикер). Новая строка: `type='share'`, `name` — SECNAME ISS, лот и шаг не пишутся.
- **Итоги дня** — только бумаги вселенной, upsert пачками по 500 строк (`UPSERT_CHUNK`); урезания нет (≈ 261 строка за дату торгов, ≈ 65 тыс. в год). Медиана и шкала в БД не хранятся — статистика процесса (§5.18.3).
- **Поправки** читает каждый запрос снимка; синхронизация и кэши их не хранят — поправка действует сразу (§5.18.5).
- **Состав и веса IMOEX** колонок не получили — кэш процесса (§5.18.2).

**Ревизия.** `b03e5c71a9d4` (`b03_screener`, поверх `8bddf72586cf` — A-06; перевешивать не пришлось) — одна на блок: десять колонок `instruments` (нативный `ADD COLUMN` через `batch_alter_table`, gotcha-12; NOT NULL — с константным DEFAULT) и три таблицы; засев четырёх стартовых поправок (PIKK → `real_estate`, ZVEZ → `industrials`, HIMCP → `materials`, KRKOP → `consumer`) — только когда таблица создана этой ревизией. Таблицы и колонки — идемпотентно через инспектор; FK на время миграций выключены (`env.py`, gotcha-79). Строки справочника ревизия не меняет. `downgrade` удаляет три таблицы целиком (поправки, в том числе заведённые администратором, итоги дня и пороги теряются) и колонки; пересоздание `instruments` теряет индекс по выражению `ix_instruments_ticker_upper` (`fb26c08f99c8`), поэтому `downgrade` снимает его до и ставит заново после. Замер на копии рабочей БД (схема `v1.0.0`, `a7c1d9e2b4f8`, 114,7 МБ): вся цепочка до `b03e5c71a9d4` — 3,7 с, ревизия Б-03 — 1,3 с (оба числа — с запуском процесса ≈ 1 с); прирост файла — 0 Б. `alembic check` — без дрейфа, одна голова. **Head — `b03e5c71a9d4`.**

---

## 4. API-КОНТРАКТЫ

**Ответственный: BACK1 (определяет), FRONT1 + FRONT2 (ревью), UX (ревью UI-контрактов)**

Все endpoints имеют префикс `/api/v1`. Формат ответов — JSON. Аутентификация — через Bearer JWT в заголовке `Authorization`. CSRF-токен передаётся в заголовке `X-CSRF-Token` для мутирующих запросов.

### 4.0 Реестр эндпоинтов (S8R-AUDIT-088)

**Источник истины по путям, методам и доступу — это реестр** (выгружен 2026-10-02 из приложения, ветка `s8r/fix-low`: 139 HTTP-маршрутов + 3 WebSocket; с S8R-FIX-069 — 140 HTTP; с A-01 (ТЗ v4.0) добавлен `GET /market-data/candles/dealer`; с A-03 (ТЗ v4.1) — `GET /market-data/price-alerts/quote`; с Б-02 (ТЗ v4.3) — `GET` и `PUT /chart-layout`, маршруты рисунков `/charts/{ticker}/drawings` вместо `/charts/{ticker}/{tf}/drawings`; с Б-01 (ТЗ v4.4) — `GET /news`, `/news/sources`, `/news/tickers`; с A-02 (ТЗ v4.5) — `PUT /broker-accounts/{id}/market-data-source`, `GET|POST /admin/ticker-aliases`, `PUT|DELETE /admin/ticker-aliases/{alias_id}`, `GET /dividends`, `GET /dividends/{ticker}`, `POST /dividends/{ticker}/corrections`, `DELETE /dividends/{ticker}/corrections/{correction_id}`; с Б-04 (ТЗ v4.6) — `GET /market-data/trading-hours`, а `DELETE /charts/{ticker}/drawings` без `timeframe` чистит все ТФ бумаги; с A-04 (ТЗ v4.7) — `GET|POST|DELETE /trading/stop-all`, `GET /circuit-breaker/config/inherited`, `GET|DELETE /circuit-breaker/config/session/{session_id}`, `GET /circuit-breaker/history`; с Б-07 (ТЗ v4.9) — `GET /news/by-ticker`, `GET /news/securities`, `GET|PUT /news/settings`, `GET|POST /news/ai-benchmark`, `GET|POST /news/alerts`, `PATCH|DELETE /news/alerts/{alert_id}`, `GET /news/alerts/{alert_id}/triggers`, `GET /settings/ai/background`, `PUT /settings/ai/background/provider`, `PUT /settings/ai/background/limit`, а `GET /news` принимает `sentiment`, `from`, `to`; с A-06 (ТЗ v4.10) — `GET /backtest/spread-check`, `GET /backtest/session-defaults`, `GET|DELETE /backtest/legacy`, `GET /trading/sessions/{session_id}/trades/{trade_id}/executions`, а запуск бэктеста и Grid принимает `dividend_tax_pct`, запуск сессии — `slippage_value`, `slippage_mode`, `dividend_tax_pct`; с Б-03 (ТЗ v4.11) — `GET /screener/snapshot`, `GET|PUT /screener/settings`, `GET /screener/sectors`, `GET|POST /admin/sector-overrides`, `PUT|DELETE /admin/sector-overrides/{id}`) и OpenAPI самого приложения (`/openapi.json` — только при `DEBUG=true`, S8R-AUDIT-059). Подразделы 4.1–4.19 ниже описывают формы запросов/ответов и правила; где их пути расходятся с реестром, верен реестр — соответствие:

| В подразделах ниже (устарело) | Фактически |
|---|---|
| `/api/v1/strategies…` (PATCH) | `/api/v1/strategy…` (PUT); версии — `/versions/list`, `/versions/by-id/{id}`, `/versions/{number}`, `/versions/snapshot`, `/versions/from-params`, `/versions/{id}/restore` |
| `/api/v1/backtests…`, `/backtests/optimize` | `/api/v1/backtest…`; grid-search — `POST /backtest/grid`, фоновые — `/backtest/run-async`, `/backtest/jobs…` |
| `/api/v1/market/…` | `/api/v1/market-data/…` (`/instruments`, `/candles` с query `ticker`, `/calendar`, `/market-status`, `/sparkline`, `/bonds/…`, `/price-alerts`) |
| `/api/v1/broker-accounts/{id}/positions`, `/operations` | `/api/v1/broker/accounts/{id}/positions`, `/operations`; балансы — `/broker/accounts/balances` |
| `PATCH /trading/sessions/{id}` | `PATCH /trading/sessions/{id}/pause`, `/resume`, `/stop`; закрытие позиции — `POST /trading/sessions/{id}/positions/{trade_id}/close` |
| `/api/v1/settings/profile`, `/settings/telegram/*` | профиль — `GET /api/v1/users/me`; Telegram — `/api/v1/notifications/telegram/*` |
| `PATCH /notifications/settings`, `POST /notifications/read-all` | `PUT /notifications/settings/{event_type}`, `PUT /notifications/read-all` |
| Export API (`/api/v1/export/…`) | экспорт бэктеста — `GET /backtest/{id}/export`; налоговый отчёт — `/api/v1/tax/reports…`; отдельного namespace нет |

Перегенерация реестра — скриптом по `app.routes` (метод, путь, зависимости `get_current_user`/`require_admin`, первая строка docstring) при изменении роутеров.

| Метод | Путь | Доступ | Модуль | Назначение (docstring) |
|---|---|---|---|---|
| GET | `/api/v1/account/balance/history` | пользователь | `account.router` | S7 7.7 — `[{ts, total_value, trading_pnl, active_capital, currency:'RUB'}]` за последние ``days`` дней. `active_capital` (S8R-FIX-073) — сумма `initial_capital` сессий, активных на дату точки; `total_value = active_capital + trading_pnl`, ноль в `active_capital` = активных сессий на эту дату не было. |
| GET | `/api/v1/account/balance/history/full` | пользователь | `account.router` | Расширенная версия — полезна для Dashboard widget, |
| GET | `/api/v1/admin/ping` | администратор | `admin.router` | Health-эндпоинт admin-зоны. |
| GET | `/api/v1/admin/sector-overrides` | администратор | `screener.admin_router` | Б-03 (ТЗ v4.11): поправки секторов «тикер → сектор» по тикеру (§3.31, §4.19). |
| POST | `/api/v1/admin/sector-overrides` | администратор | `screener.admin_router` | Б-03: добавить поправку; действует в следующем снимке; 422 — неизвестный сектор или тикер не из вселенной, 409 — поправка для тикера уже есть; 201. |
| DELETE | `/api/v1/admin/sector-overrides/{override_id}` | администратор | `screener.admin_router` | Б-03: удалить поправку — бумага снова берёт сектор брокера; 204 / 404. |
| PUT | `/api/v1/admin/sector-overrides/{override_id}` | администратор | `screener.admin_router` | Б-03: изменить поправку (тикер или сектор); аудит с прежними значениями. |
| GET | `/api/v1/admin/ticker-aliases` | администратор | `admin.router` | A-02 (ТЗ v4.5): соответствия тикеров — канонический · тикер ISS · с · по (§3.5.6, §4.15). |
| POST | `/api/v1/admin/ticker-aliases` | администратор | `admin.router` | A-02: добавить строку; история ISS бумаги за её период перезагрузится по новому правилу; 422 — пересечение периодов, «с» позже «по». |
| DELETE | `/api/v1/admin/ticker-aliases/{alias_id}` | администратор | `admin.router` | A-02: удалить строку; ISS-история за её период отменяется. 204. |
| PUT | `/api/v1/admin/ticker-aliases/{alias_id}` | администратор | `admin.router` | A-02: изменить строку; ISS-история за старый и новый период отменяется. |
| POST | `/api/v1/admin/users` | администратор | `admin.router` | S8R-AUDIT-033: создание учётной записи администратором. |
| POST | `/api/v1/ai/chat` | пользователь | `ai.chat_router` | AI-чат (non-streaming). |
| POST | `/api/v1/ai/chat/clear` | пользователь | `ai.chat_router` | Очистить историю чата стратегии. |
| GET | `/api/v1/ai/chat/history/{strategy_id}` | пользователь | `ai.chat_router` | Получить историю чата стратегии. |
| POST | `/api/v1/ai/chat/stream` | пользователь | `ai.chat_router` | AI-чат с SSE streaming. |
| POST | `/api/v1/ai/explain` | пользователь | `ai.chat_router` | Объяснение фрагмента Python/Backtrader кода. |
| GET | `/api/v1/ai/status` | пользователь | `ai.router` | Статус AI-сервиса. |
| POST | `/api/v1/auth/login` | без входа | `auth.router` | login |
| POST | `/api/v1/auth/logout` | без входа | `auth.router` | Выход: ВСЕГДА 204 + стирание cookie сессии (S8R-AUDIT-039). |
| GET | `/api/v1/auth/me` | пользователь | `auth.router` | get_me |
| PATCH | `/api/v1/auth/password` | пользователь | `auth.router` | change_password |
| PATCH | `/api/v1/auth/profile` | пользователь | `auth.router` | Обновить email и другие данные профиля. |
| POST | `/api/v1/auth/refresh` | без входа | `auth.router` | refresh |
| POST | `/api/v1/auth/setup` | без входа | `auth.router` | Мастер первого запуска: создание ЕДИНСТВЕННОГО пользователя (ФТ §2.1). |
| GET | `/api/v1/auth/setup-status` | без входа | `auth.router` | setup_status |
| POST | `/api/v1/backtest` | пользователь | `backtest.router` | Launch a new backtest asynchronously. |
| GET | `/api/v1/backtest/` | пользователь | `backtest.router` | Список бэктестов пользователя. Опционально фильтр по strategy_id. |
| POST | `/api/v1/backtest/grid` | пользователь | `backtest.router` | S7 7.2-be / C5 — запустить Grid Search. |
| GET | `/api/v1/backtest/jobs` | пользователь | `backtest.router` | S7 7.17 — список фоновых jobs пользователя (для бейджа в шапке). |
| GET | `/api/v1/backtest/jobs/{job_id}` | пользователь | `backtest.router` | S7 7.17 — снапшот одной job (REST-альтернатива WS). |
| POST | `/api/v1/backtest/jobs/{job_id}/cancel` | пользователь | `backtest.router` | S7 7.17 — отменить фоновый бэктест. |
| DELETE | `/api/v1/backtest/legacy` | пользователь | `backtest.router` | A-06 (ТЗ v4.10, ADR 0079): удалить свои бэктесты «до договора исполнения» со сделками → `{deleted}` (§4.18). |
| GET | `/api/v1/backtest/legacy` | пользователь | `backtest.router` | A-06: число своих бэктестов «до договора исполнения» → `{count}` (§4.18). |
| POST | `/api/v1/backtest/run-async` | пользователь | `backtest.router` | S7 7.17 — поставить бэктест в фоновую очередь. |
| GET | `/api/v1/backtest/session-defaults` | пользователь | `backtest.router` | A-06 (ТЗ v4.10, ADR 0073): `?strategy_version_id&ticker&timeframe` → умолчание проскальзывания и налога окна paper; чужая версия — 404 (§4.18). |
| GET | `/api/v1/backtest/spread-check` | пользователь | `backtest.router` | A-06 (ADR 0073): `?ticker&slippage_value&slippage_mode` → `{status, half_spread_pct, recommended_min_pct, reason}` по стакану T-Invest (§4.18). |
| GET | `/api/v1/backtest/strategy-params/{strategy_version_id}` | пользователь | `backtest.router` | Возвращает имена и default-значения параметров стратегии. |
| DELETE | `/api/v1/backtest/{backtest_id}` | пользователь | `backtest.router` | Удалить бэктест. |
| GET | `/api/v1/backtest/{backtest_id}` | пользователь | `backtest.router` | Get a backtest record with all metrics. |
| GET | `/api/v1/backtest/{backtest_id}/equity-curve` | пользователь | `backtest.router` | Get equity curve points for a backtest. |
| GET | `/api/v1/backtest/{backtest_id}/export` | пользователь | `backtest.router` | Экспорт бэктеста в CSV или PDF (S7 7.3). |
| POST | `/api/v1/backtest/{backtest_id}/rerun` | пользователь | `backtest.router` | Перезапустить бэктест с теми же параметрами, но на текущей версии стратегии. Параметры исходного бэктеста проверяются до создания записи (S8R-FIX-035): недопустимый тикер или период → 422 «Перезапуск невозможен…», запись не создаётся; тикер канонизируется (`canonical_ticker`, S8R-FIX-056), не найденный канон — остаётся сохранённый. |
| GET | `/api/v1/backtest/{backtest_id}/trades` | пользователь | `backtest.router` | Get paginated trades list for a backtest. |
| GET | `/api/v1/broker-accounts` | пользователь | `broker.router` | Получить список брокерских аккаунтов. |
| POST | `/api/v1/broker-accounts` | пользователь | `broker.router` | Создать брокерские аккаунты T-Invest. |
| POST | `/api/v1/broker-accounts/discover` | пользователь | `broker.router` | Обнаружить счета у брокера по API-ключу. |
| DELETE | `/api/v1/broker-accounts/{account_id}` | пользователь | `broker.router` | Удалить брокерский аккаунт. 409 `broker_account_in_use` — живые сессии или нежилые sandbox/real-сессии с открытыми позициями (`sessions[]` в ответе; S8R-FIX-052). |
| GET | `/api/v1/broker-accounts/{account_id}` | пользователь | `broker.router` | Получить брокерский аккаунт с балансом. |
| PATCH | `/api/v1/broker-accounts/{account_id}` | пользователь | `broker.router` | Обновить брокерский аккаунт. |
| PUT | `/api/v1/broker-accounts/{account_id}/market-data-source` | администратор | `broker.router` | A-02 (ТЗ v4.5): назначить свой production-счёт T-Invest счётом для рыночных данных; прежний признак снимается, выбор — в аудит; 422 с причиной, чужой счёт — 404 (§3.2, §4.6). |
| POST | `/api/v1/broker-accounts/{account_id}/reopen-sandbox` | пользователь | `broker.router` | Переоткрыть протухший sandbox-счёт (S8R-SANDBOX-ACCOUNT-STALE-REOPEN). |
| GET | `/api/v1/broker-accounts/{account_id}/sandbox-balance` | пользователь | `broker.router` | Текущий баланс sandbox-счёта T-Invest (S8 W8a). Только в рублях (S8R-FIX-053); query-параметра валюты нет. |
| POST | `/api/v1/broker-accounts/{account_id}/sandbox-topup` | пользователь | `broker.router` | Поднять баланс sandbox-счёта до target_balance (S8 W8a). Только рубли (S8R-FIX-053): `currency` в теле необязателен, допускается только `rub`, иное → 422. |
| POST | `/api/v1/broker-accounts/{account_id}/test-connection` | пользователь | `broker.router` | Проверить, что API-ключ активен у T-Invest (S5R). |
| GET | `/api/v1/broker/accounts/balances` | пользователь | `broker.router` | Получить актуальные балансы всех подключённых брокерских счетов. |
| GET | `/api/v1/broker/accounts/{account_id}/operations` | пользователь | `broker.router` | Получить реальные операции брокерского счёта через T-Invest (C9 S5R). |
| GET | `/api/v1/broker/accounts/{account_id}/positions` | пользователь | `broker.router` | Получить реальные позиции брокерского счёта через T-Invest (C8 S5R). |
| GET | `/api/v1/chart-layout` | пользователь | `chart_layouts.router` | Б-02 (ТЗ v4.3): раскладка графиков пользователя — `{layout: ChartLayoutDoc \| null, updated_at}`; раскладки нет (или в БД неверный документ) — `{"layout": null, "updated_at": null}` (§4.13). |
| PUT | `/api/v1/chart-layout` | пользователь | `chart_layouts.router` | Б-02 (ТЗ v4.3): сохранить раскладку целиком (upsert, побеждает последний); 422 — неверный документ или больше 64 КиБ (§4.13). |
| DELETE | `/api/v1/charts/drawings/{drawing_id}` | пользователь | `chart_drawings.router` | delete_drawing |
| PATCH | `/api/v1/charts/drawings/{drawing_id}` | пользователь | `chart_drawings.router` | update_drawing |
| DELETE | `/api/v1/charts/{ticker}/drawings` | пользователь | `chart_drawings.router` | clear_drawings. С Б-02 (ТЗ v4.3; до неё — `/charts/{ticker}/{tf}/drawings`): query `timeframe` обязателен, удаляются рисунки бумаги только этого ТФ. С Б-04 (ТЗ v4.6, ADR 0051): `timeframe` необязателен — без него удаляются рисунки пользователя этой бумаги на всех ТФ («Очистить»), с ним — только этот ТФ; неизвестный ТФ — 422. |
| GET | `/api/v1/charts/{ticker}/drawings` | пользователь | `chart_drawings.router` | list_drawings. С Б-02 (ТЗ v4.3; до неё — `/charts/{ticker}/{tf}/drawings`): все ТФ бумаги одним запросом, фильтр по ТФ — на фронтенде. |
| POST | `/api/v1/charts/{ticker}/drawings` | пользователь | `chart_drawings.router` | create_drawing. С Б-02 (ТЗ v4.3; до неё — `/charts/{ticker}/{tf}/drawings`): ТФ — обязательное поле тела `timeframe`. |
| GET | `/api/v1/circuit-breaker/config` | пользователь | `circuit_breaker.router` | Текущая конфигурация CB пользователя. |
| PUT | `/api/v1/circuit-breaker/config` | пользователь | `circuit_breaker.router` | Обновить глобальную конфигурацию CB. |
| GET | `/api/v1/circuit-breaker/config/inherited` | пользователь | `circuit_breaker.router` | **A-04.** Значения общих лимитов — что применит движок при пустом поле лимита сессии (`InheritedCBValues`, §4.16). |
| DELETE | `/api/v1/circuit-breaker/config/session/{session_id}` | пользователь | `circuit_breaker.router` | **A-04.** «Сбросить к общим»: 204 (строки нет — тоже 204); чужая — 404; после commit — `session.limits_changed`. |
| GET | `/api/v1/circuit-breaker/config/session/{session_id}` | пользователь | `circuit_breaker.router` | **A-04.** Лимиты сессии (`override` или `null`) и значения общих; чужая или несуществующая сессия — 404. |
| PUT | `/api/v1/circuit-breaker/config/session/{session_id}` | пользователь | `circuit_breaker.router` | Override конфигурации CB для конкретной сессии. С A-04 — лимиты сессии: пустое поле = «как в общих»; после commit — `session.limits_changed`. |
| GET | `/api/v1/circuit-breaker/events` | пользователь | `circuit_breaker.router` | Лог срабатываний CB (пагинация). |
| GET | `/api/v1/circuit-breaker/events/session/{session_id}` | пользователь | `circuit_breaker.router` | События CB по конкретной сессии. |
| GET | `/api/v1/circuit-breaker/history` | пользователь | `circuit_breaker.router` | **A-04.** История срабатываний строками-сериями: `?session_id=&offset=&limit=` (`CBHistoryPage`, §4.16). |
| GET | `/api/v1/circuit-breaker/status` | пользователь | `circuit_breaker.router` | Текущий статус CB: сделки сегодня, drawdown, etc. |
| GET | `/api/v1/corporate-actions/` | пользователь | `corporate_actions.router` | Список корп. действий с фильтрами. |
| POST | `/api/v1/corporate-actions/detect` | администратор | `corporate_actions.router` | Принудительный запуск детекции корп. действий — только администратор. |
| GET | `/api/v1/corporate-actions/{action_id}` | пользователь | `corporate_actions.router` | Детали корп. действия. |
| GET | `/api/v1/dividends` | пользователь | `dividends.router` | A-02 (ТЗ v4.5): бумаги — число неразобранных расхождений и итог последней сверки со Смартлабом (§4.15). |
| GET | `/api/v1/dividends/{ticker}` | пользователь | `dividends.router` | A-02: строки по датам реестра — T-Invest · Смартлаб · поправка · итог · вид расхождения; `has_unresolved_discrepancies`. |
| POST | `/api/v1/dividends/{ticker}/corrections` | администратор | `dividends.router` | A-02: добавить, исключить или исправить выплату; повтор ключа (тикер + дата реестра) заменяет поправку; 201. |
| DELETE | `/api/v1/dividends/{ticker}/corrections/{correction_id}` | администратор | `dividends.router` | A-02: снять поправку — выплата возвращается к значению T-Invest; 204 / 404. |
| GET | `/api/v1/health` | без входа | `main` | S8 W2 BACK2 (C-S8-1): extended health-check для Dashboard виджета. |
| GET | `/api/v1/health/live` | без входа | `main` | S8R-AUDIT-054: liveness — процесс жив и отвечает на HTTP. 200 всегда. |
| GET | `/api/v1/market-data/bonds/{ticker}/coupons` | пользователь | `market_data.router` | Расписание купонов облигации. |
| GET | `/api/v1/market-data/bonds/{ticker}/info` | пользователь | `market_data.router` | Информация об облигации. |
| GET | `/api/v1/market-data/bonds/{ticker}/nkd` | пользователь | `market_data.router` | Текущий НКД облигации. |
| GET | `/api/v1/market-data/calendar` | без входа | `market_data.router` | Get trading calendar info for today. |
| GET | `/api/v1/market-data/candles` | пользователь | `market_data.router` | Get OHLCV candles for a ticker with caching. С A-01: только биржевые свечи; ТФ `1m`…`M`, включая `30m`; производные ТФ собирает агрегатор (§5.6.6). Query `adjust=splits\|none` (A-01, дефект 4; необязательный): `splits` (по умолчанию) — бары до наступивших дроблений в текущем масштабе, `none` — «как было» (§3.5); ~~задел переключателя графика (Б-02), фронтенд параметр пока не передаёт~~ — с v4.3 (Б-02) фронтенд передаёт `adjust` панели («как было» — `none`). С A-02 (ТЗ v4.5): в ответе `history_pending: bool`, `history_pending_reason: "loading" \| "no_market_data_account" \| null`; режимы `viewer`/`backtest` грузят из T-Invest не больше 365 суток недостающего, остальное — фоном (§5.6.8). |
| GET | `/api/v1/market-data/candles/dealer` | пользователь | `market_data.router` | A-01: дилерские 1m T-Invest (внебиржевые часы) из `ohlcv_dealer`. Query как у `/candles`, без `mode`; только внутридневные ТФ (D/W/M — 422), без T-Invest — 409 `tinvest_required`. ~~Потребителя на фронте нет (задел переключателя Б-02).~~ С v4.3 (Б-02): query `adjust=splits\|none` (по умолчанию `splits`, неизвестное значение — 422) — масштаб, как у `/candles`; потребитель — переключатель «Внебиржевые часы» панели графика. |
| POST | `/api/v1/market-data/candles/subscribe` | пользователь | `market_data.router` | Подписаться на стриминг свечей в реальном времени через T-Invest. С A-03 (ТЗ v4.1): query `source=exchange\|dealer` (по умолчанию `exchange` → канал `market:{ticker}:{tf}`); `source=dealer` → `{"channel": "dealer:{ticker}:{tf}"}`, ТФ только 1m–4h (D/W/M — 422 «Дилерский поток — только внутридневные ТФ»); держит ту же подписку T-Invest на 1m, что основной канал, второй подписки нет; в режиме `MARKET_DATA_STREAM_MODE=two_subscriptions` — `{"channel": null, "source": "none", "message": "дилерский поток выключен режимом MARKET_DATA_STREAM_MODE"}`. Контракт общий с Б-02 (§2.3). С v4.3 `source=dealer` шлёт переключатель «Внебиржевые часы» панели; `POST` — один на ключ «источник : тикер : ТФ» при открытии страницы, повтор — только после переподключения WS. |
| GET | `/api/v1/market-data/instruments` | пользователь | `market_data.router` | Search instruments via MOEX ISS. |
| GET | `/api/v1/market-data/instruments/{ticker}` | пользователь | `market_data.router` | Get instrument details by ticker. |
| GET | `/api/v1/market-data/instruments/{ticker}/logo` | пользователь | `market_data.router` | Получить URL логотипа инструмента (T-Invest CDN). |
| GET | `/api/v1/market-data/market-status` | без входа | `market_data.router` | Get current MOEX market status (public endpoint). С A-03 — по фазе таблицы периодов (§5.9.1): `status` ∈ `auction` (аукцион открытия), `online`, `weekend_session`, `closing_auction`, `evening`, `dealer`, `offline`, `weekend`; `time_until_close` — до конца текущего биржевого периода. |
| GET | `/api/v1/market-data/price-alerts` | пользователь | `market_data.price_alert_router` | Список активных оповещений пользователя. С A-03 — алерты вида `level` из `alerts` (`is_active`, новые первыми), прежний формат ответа (`target_price` = `level_price`, `triggered_at` = последнее срабатывание) плюс `price_at_creation`, `price_source`, `armed`, `trigger_count` (§5.6.7). |
| POST | `/api/v1/market-data/price-alerts` | пользователь | `market_data.price_alert_router` | Создать ценовое оповещение. С A-03 тело `{ticker, target_price}` без `condition`: сервер берёт текущую цену (`GetLastPrices`, иначе ISS — `warning: "задержка 15 мин"` в ответе), округляет уровень к шагу цены и ставит `condition`; цены нет — 409 «нет текущей цены — попробуйте позже», уровень равен цене — 422 «уровень равен текущей цене — сдвиньте уровень». |
| GET | `/api/v1/market-data/price-alerts/quote` | пользователь | `market_data.price_alert_router` | A-03: `?ticker=` → `{price, price_source, warning, price_step, price_decimals}` — текущая цена, её источник и шаг цены для окна алерта; цены нет — `price: null`; `price_decimals` — знаки шага, без шага — знаки текущей цены, 2..8 (`D04`). |
| DELETE | `/api/v1/market-data/price-alerts/{alert_id}` | пользователь | `market_data.price_alert_router` | Удалить своё оповещение; чужое и несуществующее — один 404 (S8R-AUDIT-016). С A-03 удаление мягкое — `alerts.is_active = False`, история срабатываний остаётся; уже удалённый — тот же 404. |
| GET | `/api/v1/market-data/sparkline` | пользователь | `market_data.router` | S8 W2 BACK2 (C-S8-2): sparkline для виджета MarketSparkline. |
| GET | `/api/v1/market-data/trading-hours` | пользователь | `market_data.router` | Б-04 (ТЗ v4.6, ADR 0052): `?start=&end=` (ISO; без пояса — UTC) → `{start, end, trading_seconds}` — секунды биржевых периодов таблицы расписания внутри `[start, end)`, для линейки графика. 422 — `end < start`, интервал длиннее 14 610 суток, время у края календаря. |
| GET | `/api/v1/market-data/usd-rate` | пользователь | `market_data.router` | Курс USD/RUB по фиксингу ЦБ РФ (`S9-MULTICURRENCY-CBR-RATE`, Q2=b). |
| GET | `/api/v1/news` | пользователь | `news.router` | Б-01 (ТЗ v4.4): лента новостей — `NewsPage {items, next_cursor}` по убыванию `published_at`; query `ticker`, `source`, `type`, `cursor`, `limit` 1..200 (50); неизвестный `type` и битый курсор — 422, неизвестный `source` и бумага без новостей — пустая лента (§4.14). С Б-07 (ТЗ v4.9): `sentiment`, `from`, `to` (`from > to` — 422); `ticker`, `type`, `sentiment` — по действующей разметке; в элементе — `active`, `ai_note` (§4.17). |
| GET | `/api/v1/news/ai-benchmark` | администратор | `news.router` | Б-07 (ТЗ v4.9): текущая связка провайдера фоновых задач и её последний прогон проверки на эталоне — `{current, latest}` (§4.17). |
| POST | `/api/v1/news/ai-benchmark` | администратор | `news.router` | Б-07: запустить проверку на эталоне в фоне → 202 `{id, status}`; нет провайдера — 409 `no_provider`, прогон идёт — 409 `running` (§4.17). |
| GET | `/api/v1/news/alerts` | пользователь | `news.router` | Б-07: «Мои алерты по новостям» — активные алерты `news` пользователя (§4.17). |
| POST | `/api/v1/news/alerts` | пользователь | `news.router` | Б-07: создать алерт `news` → 201; неверное условие — 422 с текстом для окна, список TQBR недоступен при выбранных бумагах — 503 (§4.17). |
| DELETE | `/api/v1/news/alerts/{alert_id}` | пользователь | `news.router` | Б-07: удалить свой алерт `news` (мягко, история остаётся) → 204; чужой — 404 (§4.17). |
| PATCH | `/api/v1/news/alerts/{alert_id}` | пользователь | `news.router` | Б-07: пауза / снятие паузы `{paused}`; чужой — 404 (§4.17). |
| GET | `/api/v1/news/alerts/{alert_id}/triggers` | пользователь | `news.router` | Б-07: история срабатываний алерта `news` (§4.17). |
| GET | `/api/v1/news/by-ticker` | пользователь | `news.router` | Б-07: новости бумаги за интервал для меток графика — `NewsItemOut[]` по возрастанию времени; `ticker`, `from`, `to` обязательны, `from > to` и > 400 суток — 422 (§4.17). |
| GET | `/api/v1/news/securities` | пользователь | `news.router` | Б-07: акции TQBR из списка словаря `[{ticker, shortname}]` — для окна алерта; списка нет — 503 (§4.17). |
| GET | `/api/v1/news/settings` | пользователь | `news.router` | Б-07: выключатели раздела «Новости» и `ai_verified` (§4.17). |
| PUT | `/api/v1/news/settings` | администратор | `news.router` | Б-07: изменить выключатели (любые из четырёх; аудит `news.settings_changed`); «ИИ-разметка» без пройденной проверки — 409 `benchmark_required` (§4.17). |
| GET | `/api/v1/news/sources` | пользователь | `news.router` | Б-01 (ТЗ v4.4): источники для фильтра — `[{code, name}]` в порядке строк миграции. |
| GET | `/api/v1/news/tickers` | пользователь | `news.router` | Б-01 (ТЗ v4.4): бумаги, у которых есть новости, по алфавиту. |
| DELETE | `/api/v1/notifications/` | пользователь | `notification.router` | Удалить все уведомления пользователя. |
| GET | `/api/v1/notifications/` | пользователь | `notification.router` | Список уведомлений пользователя, отсортированных по created_at DESC. |
| PUT | `/api/v1/notifications/read-all` | пользователь | `notification.router` | Пометить все уведомления как прочитанные. |
| GET | `/api/v1/notifications/settings` | пользователь | `notification.router` | Настройки уведомлений пользователя по event_type. |
| PUT | `/api/v1/notifications/settings/{event_type}` | пользователь | `notification.router` | Обновить настройку для конкретного event_type (атомарный upsert). |
| POST | `/api/v1/notifications/telegram/link-token` | пользователь | `notification.router` | Генерирует одноразовый токен для привязки Telegram (base64url, 12 символов). |
| GET | `/api/v1/notifications/telegram/status` | пользователь | `notification.router` | Проверить, привязан ли Telegram. |
| POST | `/api/v1/notifications/telegram/test` | пользователь | `notification.router` | Тестовое сообщение в привязанный Telegram-чат пользователя. |
| DELETE | `/api/v1/notifications/telegram/unlink` | пользователь | `notification.router` | Отвязать Telegram. |
| POST | `/api/v1/notifications/telegram/webhook` | без входа | `notification.router` | Endpoint для Telegram webhook. |
| POST | `/api/v1/notifications/test-email` | пользователь | `notification.router` | Отправить тестовое email-уведомление на почту пользователя. |
| GET | `/api/v1/notifications/unread-count` | пользователь | `notification.router` | Количество непрочитанных уведомлений. |
| PATCH | `/api/v1/notifications/{notification_id}/read` | пользователь | `notification.router` | Пометить уведомление как прочитанное. |
| POST | `/api/v1/observability/lcp` | пользователь | `common.observability_router` | Положить клиентский LCP в тот же буфер, что и серверные `@timed_event`. |
| GET | `/api/v1/screener/sectors` | пользователь | `screener.router` | Б-03 (ТЗ v4.11): 10 секторов (код, название) в порядке таблицы развилки 8 (§4.19). |
| GET | `/api/v1/screener/settings` | пользователь | `screener.router` | Б-03: порог ликвидности пользователя и умолчание (10 млн ₽). |
| PUT | `/api/v1/screener/settings` | пользователь | `screener.router` | Б-03: сохранить порог 0…10¹² ₽; иначе 422 «Порог — число от 0 до 1 000 000 000 000 ₽». |
| GET | `/api/v1/screener/snapshot` | пользователь | `screener.router` | Б-03: снимок рынка — вся вселенная одной строкой на бумагу; доски ISS не было ни разу — 503 (§4.19). |
| GET | `/api/v1/settings/ai/background` | администратор | `ai.router` | Б-07 (ТЗ v4.9): провайдер фоновых задач, кандидаты (строки владельца), дневной лимит токенов, расход за сегодня (§4.17). |
| PUT | `/api/v1/settings/ai/background/limit` | администратор | `ai.router` | Б-07: дневной лимит токенов фоновых задач `{daily_token_limit}` 0…1 000 000 000, иначе 422; аудит `ai.background_limit_changed` (§4.17). |
| PUT | `/api/v1/settings/ai/background/provider` | администратор | `ai.router` | Б-07: отметить строку «для фоновых задач» `{config_id}` (`null` — снять); чужая строка — 404; аудит `ai.background_provider_selected` (§4.17). |
| GET | `/api/v1/settings/ai/instructions` | пользователь | `ai.router` | Получить пользовательские инструкции для AI. |
| PUT | `/api/v1/settings/ai/instructions` | пользователь | `ai.router` | Обновить пользовательские инструкции для AI. |
| GET | `/api/v1/settings/ai/providers` | пользователь | `ai.router` | Список AI-провайдеров пользователя. |
| POST | `/api/v1/settings/ai/providers` | пользователь | `ai.router` | Добавить AI-провайдер. |
| POST | `/api/v1/settings/ai/providers/verify-credentials` | пользователь | `ai.router` | Проверить credentials без сохранения. |
| DELETE | `/api/v1/settings/ai/providers/{provider_id}` | пользователь | `ai.router` | Удалить AI-провайдер. |
| PATCH | `/api/v1/settings/ai/providers/{provider_id}` | пользователь | `ai.router` | Обновить AI-провайдер. |
| POST | `/api/v1/settings/ai/providers/{provider_id}/activate` | пользователь | `ai.router` | Сделать провайдер активным. |
| POST | `/api/v1/settings/ai/providers/{provider_id}/reset-usage` | пользователь | `ai.router` | Сбросить счётчики токенов провайдера. |
| POST | `/api/v1/settings/ai/providers/{provider_id}/verify` | пользователь | `ai.router` | Проверить API-ключ провайдера. |
| GET | `/api/v1/settings/ai/usage` | пользователь | `ai.router` | Статистика использования AI. |
| GET | `/api/v1/strategy` | пользователь | `strategy.router` | Получить список стратегий текущего пользователя. |
| POST | `/api/v1/strategy` | пользователь | `strategy.router` | Создать новую стратегию. |
| POST | `/api/v1/strategy/parse-template` | пользователь | `strategy.router` | Parse text template into block diagram JSON. |
| DELETE | `/api/v1/strategy/{strategy_id}` | пользователь | `strategy.router` | Удалить стратегию: 204 — удалена; 200 `{archived: true}` — при истории |
| GET | `/api/v1/strategy/{strategy_id}` | пользователь | `strategy.router` | Получить детали стратегии с версиями. |
| PUT | `/api/v1/strategy/{strategy_id}` | пользователь | `strategy.router` | Обновить стратегию. |
| POST | `/api/v1/strategy/{strategy_id}/generate-code` | пользователь | `strategy.router` | Generate Python code from block diagram JSON. |
| POST | `/api/v1/strategy/{strategy_id}/duplicate` | пользователь | `strategy.router` | Дублировать стратегию: «<имя> (копия)» (черновик) с копией последней версии одним commit; чужая → 404 (S8R-FIX-069). |
| POST | `/api/v1/strategy/{strategy_id}/versions` | пользователь | `strategy.router` | Создать новую версию стратегии. |
| GET | `/api/v1/strategy/{strategy_id}/versions/by-id/{version_id}` | пользователь | `strategy.router` | Полная версия по её id (для history-detail UI). |
| POST | `/api/v1/strategy/{strategy_id}/versions/from-params` | пользователь | `strategy.router` | S7R-GRID-APPLY-PARAMS — создать новую версию с переписанными default-ами. |
| GET | `/api/v1/strategy/{strategy_id}/versions/list` | пользователь | `strategy.router` | История версий стратегии (DESC по version_number). |
| POST | `/api/v1/strategy/{strategy_id}/versions/snapshot` | пользователь | `strategy.router` | Создать именованный snapshot текущего состояния стратегии. |
| POST | `/api/v1/strategy/{strategy_id}/versions/{version_id}/restore` | пользователь | `strategy.router` | History-preserving откат: создаёт новую версию-копию + переключает current. |
| GET | `/api/v1/strategy/{strategy_id}/versions/{version_number}` | пользователь | `strategy.router` | Получить конкретную версию стратегии (по version_number). |
| GET | `/api/v1/tax/reports` | пользователь | `tax.router` | Список отчётов пользователя. |
| POST | `/api/v1/tax/reports` | пользователь | `tax.router` | Сгенерировать налоговый отчёт. |
| GET | `/api/v1/tax/reports/{report_id}` | пользователь | `tax.router` | Детали отчёта. |
| GET | `/api/v1/tax/reports/{report_id}/download` | пользователь | `tax.router` | Скачать файл отчёта. |
| GET | `/api/v1/trading/dashboard` | пользователь | `trading.router` | Сводка: active sessions count, total P&L. |
| GET | `/api/v1/trading/positions` | пользователь | `trading.router` | Все открытые позиции по всем сессиям. |
| GET | `/api/v1/trading/sessions` | пользователь | `trading.router` | Список торговых сессий. |
| POST | `/api/v1/trading/sessions` | пользователь | `trading.router` | Запустить новую торговую сессию. С A-04 — необязательное `cb_limits`; 409 `trading_stopped`. |
| DELETE | `/api/v1/trading/sessions/{session_id}` | пользователь | `trading.router` | Удалить торговую сессию (только остановленные). 409 `session_has_open_positions` — sandbox/real с открытыми позициями (S8R-FIX-052). Счёт также держит неснятая пометка расхождения sandbox/real-сессии (S8R-FIX-070): в `sessions[]` ответа — признак `position_mismatch`. |
| GET | `/api/v1/trading/sessions/{session_id}` | пользователь | `trading.router` | Детали торговой сессии. |
| POST | `/api/v1/trading/sessions/{session_id}/close-all` | пользователь | `trading.router` | Закрыть все позиции сессии. |
| PATCH | `/api/v1/trading/sessions/{session_id}/pause` | пользователь | `trading.router` | Поставить сессию на паузу. |
| GET | `/api/v1/trading/sessions/{session_id}/positions` | пользователь | `trading.router` | Открытые позиции сессии. |
| POST | `/api/v1/trading/sessions/{session_id}/positions/{trade_id}/close` | пользователь | `trading.router` | Закрыть позицию. Тело `{reason?, confirm_closed_outside?}`; ответ +`closed_without_order`. У сессии, которую рантайм не ведёт (`runtime_does_not_manage`: sandbox/real и `paused`/`stopped` или `suspended` без слушателя), — сверка с портфелем брокера перед ордером: позиции нет → закрытие без ордера; 409 `broker_position_partial` / `broker_position_unverified`; `confirm_closed_outside=true` — закрыть без ордера по подтверждению пользователя (filled, неуправляемая сессия). `close-all` — `results[]` по сделкам (S8R-FIX-052). |
| POST | `/api/v1/trading/sessions/{session_id}/reconcile` | пользователь | `trading.router` | **S8R-FIX-070.** Ручная сверка позиций с брокером для сессии, которую рантайм не ведёт: ответ `{status: cleared|mismatch, note}` — пометка расхождения снята либо подтверждена. 422 — paper или сессия без брокерского счёта; 409 `session_is_live` — живую сессию сверяет фоновый цикл; 409 `reconcile_not_done` — портфель не прочитан (пометка остаётся). Бюджет — `POSITION_RECONCILE_TIMEOUT_SEC`. |
| PATCH | `/api/v1/trading/sessions/{session_id}/resume` | пользователь | `trading.router` | Возобновить сессию. С A-04 — 409 `trading_stopped`. |
| GET | `/api/v1/trading/sessions/{session_id}/stats` | пользователь | `trading.router` | Агрегированная статистика сессии (формат `SessionStats` для PnLSummary). |
| PATCH | `/api/v1/trading/sessions/{session_id}/stop` | пользователь | `trading.router` | Остановить сессию. |
| GET | `/api/v1/trading/sessions/{session_id}/trades` | пользователь | `trading.router` | Сделки сессии с пагинацией. |
| GET | `/api/v1/trading/sessions/{session_id}/trades/{trade_id}/executions` | пользователь | `trading.router` | A-06 (ТЗ v4.10, ADR 0072): журнал исполнений сделки — `ExecutionResponse[]` по времени; чужая сессия или сделка не этой сессии — 404 (§4.18). |
| DELETE | `/api/v1/trading/stop-all` | веб-сессия (cookie `access_token` без `Authorization`) | `trading.router` | **A-04.** Снять стоп; не из веба — 403, не включён — 409 `trading_not_stopped`. |
| GET | `/api/v1/trading/stop-all` | пользователь | `trading.router` | **A-04.** Состояние «Стоп всему» (`StopAllState`, §4.4). |
| POST | `/api/v1/trading/stop-all` | пользователь | `trading.router` | **A-04.** Включить «Стоп всему»: флаг, пауза, аварийное закрытие, отчёт «N из M»; уже включён — состояние и `already_active: true`. |
| GET | `/api/v1/user-favorites` | пользователь | `user_favorites.router` | list_favorites |
| POST | `/api/v1/user-favorites/{kind}` | пользователь | `user_favorites.router` | add_favorite |
| DELETE | `/api/v1/user-favorites/{kind}/{value}` | пользователь | `user_favorites.router` | remove_favorite |
| GET | `/api/v1/users/me` | пользователь | `users.router` | Профиль текущего пользователя (прокси, точная копия `GET /auth/me`). |
| POST | `/api/v1/users/me/disclaimer/accept` | пользователь | `users.router` | Зафиксировать принятие дисклеймера (ФТ §18.1, S8R-AUDIT-043). |
| POST | `/api/v1/users/me/wizard/complete` | пользователь | `users.router` | Отметить, что пользователь прошёл onboarding-визард. |
| WS | `/ws` | токен (cookie/Bearer, проверка до accept) | `backtest.ws` | Multiplexed WebSocket endpoint с cookie-JWT аутентификацией. |
| WS | `/ws/backtest/{job_id}` | токен (cookie/Bearer, проверка до accept) | `backtest.ws_backtest` | WS канал прогресса конкретной job (контракт C2). |
| WS | `/ws/trading-sessions/{user_id}` | токен (cookie/Bearer, проверка до accept) | `trading.ws_sessions` | WebSocket endpoint трейдинг-сессий пользователя (контракт C1). |

### 4.1 Auth API

```
POST   /api/v1/auth/setup          -- Первоначальная настройка (создание ЕДИНСТВЕННОГО пользователя)
       Body: { username, password }
       Response: 201 (токены — в cookie, Model A)
       Error 403: { detail: "Регистрация закрыта: …" } -- в БД уже есть пользователь (S8R-AUDIT-033);
                  атомарно: INSERT … WHERE NOT EXISTS, из двух параллельных setup проходит один
       -- Лимитер: категория auth (как login/refresh); /auth/setup-status — general

POST   /api/v1/auth/login
       Body: { username, password }
       Response: 200 { access_token, refresh_token, expires_in }
       Error 401: { detail: "Неверный логин или пароль" }
                  -- S8R-AUDIT-043: один ответ для «нет пользователя», «неверный пароль»,
                  -- «учётка временно заблокирована» (423 удалён; причина — только в журнал/лог).
                  -- Нет пользователя → verify против dummy-хеша (считается в lifespan);
                  -- Argon2 — в потоке; выборка без eager-связей; счётчик неудач — один
                  -- атомарный UPDATE … WHERE не заблокирован RETURNING; успех — сброс
                  -- тем же условием (0 строк → 401) + rehash устаревшего хеша.
       Error 429: лимитер auth, заголовок Retry-After
       Error 503: { detail: "База данных временно занята…" } + Retry-After
                  -- сбой записи БД в ЛЮБОЙ ветке (отказ и успех) — один ответ

POST   /api/v1/auth/refresh
       Body: { refresh_token }
       Response: 200 { access_token, refresh_token, expires_in }

POST   /api/v1/auth/logout
       Response: 204

GET    /api/v1/auth/me
       Response: 200 { user profile }

PATCH  /api/v1/auth/password
       Body: { current_password, new_password }
       Response: 200
       -- new_password — политика §7.1 (422 при нарушении)

GET    /api/v1/auth/setup-status
       Response: 200 { is_configured: bool, password_policy: { min_length, max_length,
                       require_letter, require_digit, description } }
       -- password_policy — единый источник правила нового пароля для фронта (S8R-AUDIT-043)
       -- Возвращает true если в таблице users есть хотя бы один пользователь.
       -- Используется frontend для автоматического редиректа на /setup при первом запуске.
       -- Доступен без JWT.
       -- S8R-FIX-101: сбой запроса на /login (`LoginPage.setupStatusErrorText`): 429 (лимитер general) —
       -- «Слишком много запросов, повторите через N с» по Retry-After (без заголовка — «…повторите
       -- позже»); 5xx / нет ответа — «Сервер недоступен. Проверьте, что терминал запущен, и обновите
       -- страницу.» Порт в тексте не называется (за nginx его нет). 429 самого login — «Слишком много
       -- попыток…», тот же хелпер `tooManyText`.
```

### 4.2 Strategy API

Статусы (S8R-AUDIT-081, `app/strategy/status.py` — единый источник, фронтовые копии сверяются тестом): `status` — enum из 6 значений в OpenAPI; `PUT /strategy/{id}` со статусом — 422 вне графа ручных переходов (`paper`/`live` — только системные), 409 `strategy_has_live_sessions` при живых сессиях, 409 `strategy_status_changed` при гонке (условный UPDATE); `POST /trading/sessions` — 409 `strategy_status_not_startable` из draft/paused/archived. Автоперевод: любая запись статуса сессии идёт через `app/trading/session_status.py` (S8R-FIX-037: `set_session_status`, условные `set_sessions_status_if`/`pause_session_if_active`, `delete_session_row`; AST-страж запрещает прямые записи), setter сам вызывает `sync_strategy_status_with_sessions` в той же транзакции при пересечении границы живых сессий (меняет только tested/paper/live); `stopped` — терминальный, завершение бэктеста — `promote_draft_after_backtest` (`draft → tested`). Миграция данных `2c7bd0443aa6`.

**Смена статуса из меню — один запрос (S8R-FIX-103).** `StrategyStatusMenu` шлёт ровно один `PUT /strategy/{id}` `{status}` (`strategyApi.updateStatus`); стор правится только локально — `strategyStore.patchStrategyLocal` (оптимистично, затем статусом из ответа сервера, при ошибке — откат), без сети. Прежний сетевой `updateStrategy` рядом с `updateStatus` давал два PUT: проигравший получал 409 `strategy_status_changed`, UI откатывался при успехе на сервере, а откат шёл третьим PUT. Тексты подписей статусов во фронте (`STATUS_META`, `STATUS_MAP`, дашборд) — «Бумажная торговля» / «Боевая торговля», у инструментов «Бумажная / Песочница / Боевая» (S8R-FIX-104, ФТ §3.1).

```
GET    /api/v1/strategies
       Query: ?status=&page=&per_page=
       Response: 200 { items: [...], total, page, per_page }

POST   /api/v1/strategies
       Body: { name }
       Response: 201 { strategy }

GET    /api/v1/strategies/{id}
       Response: 200 { strategy with current_version }

PATCH  /api/v1/strategies/{id}
       Body: { name?, status? }
       Response: 200 { strategy }

DELETE /api/v1/strategies/{id}
       Response: 204

GET    /api/v1/strategies/{id}/versions
       Response: 200 { items: [...] }

POST   /api/v1/strategies/{id}/versions
       Body: { text_description?, blocks_json, generated_code, parameters_json? }
       Response: 201 { version }

GET    /api/v1/strategies/{id}/versions/{version_id}
       Response: 200 { version }

POST   /api/v1/strategies/{id}/generate-code
       Body: { blocks_json }
       Response: 200 { generated_code, validation_errors? }

POST   /api/v1/strategies/{id}/parse-template
       Body: { template_text }
       Response: 200 { blocks_json, warnings? }
       Error 422: { detail: "Шаблон не распознан", errors: [...] }
```

**Отчёт разбора и запрет сохранения (A-05, ТЗ v4.2; ФТ §3.2).** Фактические пути — реестр §4.0, `app/strategy/router.py`, схемы — `app/strategy/schemas.py` ↔ `frontend/src/api/strategyApi.ts`:

```
POST   /api/v1/strategy/parse-template
       Body: { template_text }
       Response: 200 { blocks_json, warnings: [str], report: ParseReport }
       -- warnings — тексты записей report (прежнее поле, для совместимости)

POST   /api/v1/strategy/{id}/versions
       Body: { text_description?, blocks_json, generated_code, parameters_json?, comment? }
       Response: 201 VersionResponse { …, parse_report: ParseReport | null }
       Error 422: { detail: "Описание стратегии содержит проблемы — версия не сохранена",
                    error_code: "description_has_problems", report: ParseReport }
       -- text_description в формате ИИ-чата (`is_reference_format`) разбирается на сервере ДО проверки
       -- идемпотентности; «проблема» → 422, версия не создаётся. Иначе отчёт пишется в версию;
       -- текст не в формате ИИ-чата → parse_report = null («отчёт не составлялся»).

ParseReport = { status: "ok" | "notes" | "problems",
                issues: [{ level: "problem" | "note", line: int | null, fragment, message, hint }] }
```

- `VersionResponse.parse_report` — из JSON-строки модели; нечитаемый JSON → `null`, а не 500. `VersionListItem` (список версий) не меняется.
- Копии версии отчёт не пересчитывают, а переносят: `_version_from` (`restore`, `snapshot`, «Дублировать») и `POST /versions/from-params` (`create_version(…, inherit_parse_report_from=source)`).
- Идемпотентность автоснимка (5 мин, `StrategyService._is_same_payload`, ФТ §19.1) сравнивает `blocks_json + generated_code + text_description` — исправленный после 422 текст при тех же блоках создаёт новую версию.
- Первая версия из `POST /strategy` (свободное описание карточки) не разбирается; новую стратегию фронт сохраняет `createWithFirstVersion` → `POST /strategy/{id}/versions`, отказ 422 откатывает созданную стратегию (ФТ §3.5).

### 4.3 Backtest API

```
POST   /api/v1/backtests
       Body: {
           strategy_version_id,
           ticker,
           timeframe,
           date_from,             -- "2021-01-01"
           date_to,               -- "2026-01-01"
           initial_capital,
           commission_pct,
           slippage_value,
           slippage_mode,          -- "pct" | "points"
           params_overrides?: {}   -- для grid search
       }
       Response: 202 { backtest_id, status: "running" }

GET    /api/v1/backtests/{id}
       Response: 200 { backtest with metrics }

GET    /api/v1/backtests/{id}/trades
       Query: ?page=&per_page=&sort_by=&order=
       Response: 200 { items: [...], total }

GET    /api/v1/backtests/{id}/equity-curve
       Response: 200 { points: [{timestamp, equity, drawdown}] }

DELETE /api/v1/backtests/{id}
       Response: 204

POST   /api/v1/backtests/optimize
       Body: {
           strategy_version_id, ticker, timeframe, date_from, date_to,
           initial_capital, commission_pct, slippage_pct,
           param_ranges: { "ema_period": { min: 10, max: 50, step: 5 } }
       }
       Response: 202 { optimization_id }

GET    /api/v1/backtests/optimize/{id}
       Response: 200 { status, results: [{params, metrics}], overfitting_warning? }
```

**Лот бэктеста (A-05, ТЗ v4.2; ФТ §4.1, §4.5).** Фактические пути — реестр §4.0 (`/api/v1/backtest…`), схемы — `app/backtest/schemas.py`, `router.py` ↔ `frontend/src/api/backtestApi.ts`:
- **Запуск** (`POST /backtest`, `/backtest/run-async`, `/backtest/{id}/rerun`, `/backtest/grid`): лот — `resolve_backtest_lot(db, ticker, user_id)` один раз на запуск (rerun — заново, Grid — один на все комбинации), пишется в `backtests.lot_size` и передаётся в движок (§5.3.1б).
- **`GET /backtest/{id}`**: `lot_size: int | null` (лот запуска; `null` — бэктест до A-05) и `legacy_share_lots: bool` (`is_legacy_share_lots`: `lot_size is null` и `instruments.lot_size > 1`). К `warning` дописывается `lot_unverified_warning` — «Лот N не сверен с T-Invest: …» или текст с обоими лотами, если лот справочника разошёлся с лотом бэктеста. Справочник читается из БД (`directory_instrument`), без сети.
- **`GET /backtest/`** (и `?strategy_id=`): те же `lot_size`, `legacy_share_lots`; справочник — одним запросом по набору тикеров (`directory_lots`).
- **`GET /backtest/{id}/export?format=csv|pdf`**: у `legacy_share_lots` колонка `volume_lots` называется «Акций» вместо «Лотов» (`generate_csv`/`generate_pdf(…, legacy_share_lots=)`).
- **`GET /backtest/strategy-params/{id}`** (Grid-автокомплит) не отдаёт служебные параметры `SERVICE_STRATEGY_PARAMS` (`lot_size`, `commission_pct`).
- **Паритетная сверка** и предупреждение «0 сделок» (`_position_size_warning`) берут лот и комиссию из строки `Backtest`; лот `NULL` — 1, как считали до A-05.

**«История догружается» (A-02, ТЗ v4.5; ФТ §4.2).** `GET /backtest/{id}` при `status = completed` читает свечи `get_candles(…, mode="viewer")` прямо в HTTP-запросе (может синхронно догрузить до года и поставить бумагу в фон) и по `MarketDataService.history_pending` дописывает к `warning` (`backtest.router.HISTORY_PENDING_WARNINGS`): `loading` — «История за период догружается — результат посчитан на неполных данных, повторите бэктест позже»; `no_market_data_account` — «История за период неполна — результат посчитан на неполных данных; часть истории T-Invest не загрузится: не выбран счёт для рыночных данных». Признак считается на момент просмотра, а не запуска. Пропуски дневок раньше 09.05.2018 в той же строке `warning` уточняются днями торгов ISS (§5.3.3).

**Время в ответе и выгрузке бэктеста (v4.8, починка времени бэктеста; ФТ §4.5, §19.3).** Правило «naive из БД = UTC» на backend — `app.common.datetime_utils.unix_utc`.
- **`GET /api/v1/backtest/{id}`**: `equity_curve[].time`, `candles[].time`, `trade_markers[].time` — unix-секунды UTC при любом поясе процесса; раньше зависели от `TZ` сервера. Формат и поля не менялись. `trades[].entry_date` / `exit_date` — по-прежнему naive ISO, UTC.
- **`GET /api/v1/backtest/{id}/export`** (csv, pdf): `entry_date`, `exit_date`, «Запущен», «Завершён» — МСК. Было UTC, формат `ГГГГ-ММ-ДД ЧЧ:ММ:СС` и заголовки прежние; «Период с/по» — календарные даты, без изменений.
- Схема БД и миграции не менялись.

**Договор исполнения (A-06, ТЗ v4.10; ФТ §22).** Запуск бэктеста и Grid — `dividend_tax_pct` (0…100, по умолчанию 13) и умолчание проскальзывания 0,1 % вместо 0; ответ `GET /backtest/{id}` и списка — поля договора (`sharpe_basis`, `legacy_execution_rules`, `money_market`, `price_pnl`, `dividend_pnl`, `benchmark.index_code`, `report_notes`, `row_kind`); новые маршруты `spread-check`, `session-defaults`, `legacy` — §4.18.

### 4.4 Trading API

```
POST   /api/v1/trading/sessions
       Body: {
           strategy_version_id,
           broker_account_id,
           ticker,
           mode,                  -- "paper" | "real"
           position_sizing_mode,  -- "fixed_sum" | "fixed_lots" | "percent"
           position_sizing_value,
           max_concurrent_positions?,
           cooldown_seconds?,
           initial_capital?       -- Для paper
       }
       Response: 201 { session }

GET    /api/v1/trading/sessions
       Query: ?status=&mode=
       Response: 200 { items: [...] }

GET    /api/v1/trading/sessions/{id}
       Response: 200 { session with stats }

PATCH  /api/v1/trading/sessions/{id}
       Body: { status: "paused" | "active" | "stopped" }
       Response: 200 { session }

GET    /api/v1/trading/sessions/{id}/trades
       Query: ?status=&page=&per_page=
       Response: 200 { items: [...], total }

POST   /api/v1/trading/sessions/{id}/close-position/{trade_id}
       Response: 200 { trade }

POST   /api/v1/trading/sessions/{id}/close-all
       Response: 200 { closed_count, trades: [...] }

GET    /api/v1/trading/sessions/{id}/stats
       Response: 200 { daily_pnl, total_pnl, open_positions, ... }
```

**«Стоп всему», лимиты при запуске, причина паузы (v4.7, A-04; ФТ §12.4; ADR 0040).**

```
POST   /api/v1/trading/sessions
       Body: { …, cb_limits?: CircuitBreakerConfigRequest | null }   -- те же поля и проверки, что у PUT /circuit-breaker/config/session/{id}
       -- строка circuit_breaker_configs(session_id) — тем же commit, что сессия; null или все поля пусты — строки нет
       409 trading_stopped — включён «Стоп всему» (тело ниже)

PATCH  /api/v1/trading/sessions/{id}/resume
       409 trading_stopped — включён «Стоп всему»

GET    /api/v1/trading/stop-all          → 200 StopAllState
POST   /api/v1/trading/stop-all          → 200 StopAllEnableResponse = StopAllState + { already_active: bool }
       -- cookie-сессия — с X-CSRF-Token; ответ — после закрытия всех позиций (включение доводится до конца и при обрыве запроса)
DELETE /api/v1/trading/stop-all          → 200 StopAllState (active: false)
       403 «Снять стоп можно только в веб-интерфейсе» — есть заголовок Authorization или нет cookie access_token; 403 — CSRF
       409 trading_not_stopped «Стоп не включён»

StopAllState { active: bool, enabled_at: str | null (ISO UTC, «Z»), enabled_source: "web" | "telegram" | null,
               closed_count: int | null, total_count: int | null,   -- N и M отчёта включения; null — закрытие идёт
               waiting_count: int,                                 -- живое: сделки сессий пользователя с exit_retry_kind IS NOT NULL
               report: StopAllReport | null }
StopAllReport { sessions_paused, closed_count, total_count, waiting_count,
                items: [{ trade_id, session_id, ticker, mode, result: "not_closed", reason, queued }] }   -- только незакрытые

409 trading_stopped: { "detail": "Торговля остановлена: включён «Стоп всему» (09.11.2026 16:29 МСК, из интерфейса). Снимите стоп в плашке вверху экрана — сессии останутся на паузе, каждую возобновите вручную.",
                       "error_code": "trading_stopped", "enabled_at": "2026-11-09T13:29:00Z", "enabled_source": "web" }
```

`SessionResponse` (список, детали, ответы запуска, паузы и возобновления): `pause_reason: str | null` — код причины паузы (§3.8), `has_session_limits: bool` — у сессии есть строка лимитов сессии (одним запросом на страницу, `_session_limit_ids`).

### 4.5 Market Data API

```
GET    /api/v1/market/instruments/search
       Query: ?q=SBER
       Response: 200 { items: [{ticker, name, type, lot_size, ...}] }

GET    /api/v1/market/instruments/{ticker}
       Response: 200 { instrument details }

GET    /api/v1/market/candles/{ticker}
       Query: ?timeframe=D&from=2024-01-01&to=2024-12-31
       Response: 200 { candles: [{t, o, h, l, c, v}] }

GET    /api/v1/market/calendar
       Query: ?year=2026
       Response: 200 { holidays: [...], sessions: {...} }

GET    /api/v1/market/status/{ticker}
       Response: 200 { is_trading, session_open, session_close, is_holiday }
```

**Признак «догружается» (A-02, ТЗ v4.5; ADR 0034).** Фактический маршрут — `GET /api/v1/market-data/candles` (реестр §4.0). `CandlesResponse` дополнен необязательными полями `history_pending: bool = false` и `history_pending_reason: "loading" | "no_market_data_account" | null` — из `MarketDataService.history_pending` после `get_candles` (`_candles_response`). `loading` — недостающее (T-Invest вне окна ISS, ISS в окне) поставлено в фон; `no_market_data_account` — недостающий кусок T-Invest фоном не загрузится: счёт для рыночных данных не выбран. Фронтенд графика поля пока не читает (`marketDataApi.ts` — типы есть).

**Часы торгов для линейки (Б-04, ТЗ v4.6; ADR 0052).** `app/market_data/router.py`, рядом с `/market-status`; `Depends(get_current_user)`, лимитер — категория `general`.

```
GET /api/v1/market-data/trading-hours?start=<ISO>&end=<ISO>
    Response: 200 TradingHoursResponse { start: datetime (UTC), end: datetime (UTC), trading_seconds: int }
    422: нет параметра или не время; end < start; end − start > TRADING_HOURS_MAX_DAYS (14 610 суток ≈ 40 лет);
         время у края календаря Python (OverflowError)
    401: без входа
```

`trading_seconds` — сумма пересечений с `[start, end)` периодов `EXCHANGE_KINDS` (аукцион открытия, торги, аукцион закрытия, вечер; `dealer` не входит) из `get_trading_schedule()` (§5.9.1). Дни — по МСК от даты `start` − 1 до даты `end` + 1: период дня МСК лежит и в соседних сутках UTC. День без строки — запасные часы (`fallback_day`), будничный праздник — ноль. Выходные сессии считаются для любой бумаги: признак торгов выходного дня у бумаги не смотрится, бумаги в запросе нет. Время с поясом приводится к UTC, без пояса считается UTC. Маршрут только читает кэш процесса: в T-Invest и ISS не ходит, в БД не пишет. Тесты — `tests/unit/test_market_data/test_trading_hours_route.py` (фикстуры расписания A-03: пт 23.10.2026 15:00 → пн 26.10.2026 15:00 МСК — 127 202 с).

### 4.6 Broker Account API

```
GET    /api/v1/broker-accounts
       Response: 200 { items: [...] }

POST   /api/v1/broker-accounts
       Body: { broker_type, name, api_key, api_secret?, account_id, account_type }
       Response: 201 { account (key masked) }
       Error 422: sandbox-токен, свободных счетов нет (S8R-FIX-115): «Sandbox-счета этого токена уже
                  подключены другим пользователем терминала…» | «…закрыты брокером — подключать нечего…»

GET    /api/v1/broker-accounts/{id}
       Response: 200 { account with balance }

PATCH  /api/v1/broker-accounts/{id}
       Body: { name?, api_key?, is_active? }
       Response: 200

PUT    /api/v1/broker-accounts/{id}/market-data-source        -- A-02 (ТЗ v4.5): только администратор (require_admin; иначе 403)
       Body: —
       Response: 200 BrokerAccountResponse { …, is_market_data_source: true }
       Error 404: не свой счёт администратора или нет такого
       Error 422: «Счёт для рыночных данных — только T-Invest: у этого счёта другой брокер»
                | «Выбрать можно только production-счёт: sandbox не подходит для рыночных данных»
                | «Счёт неактивен: включите его в настройках или выберите другой»
                | «У счёта нет ключа доступа: добавьте токен T-Invest или выберите другой счёт»
       -- признак у прежнего счёта снимается в той же транзакции; аудит broker.market_data_account_selected

DELETE /api/v1/broker-accounts/{id}
       Response: 204 | 409 { detail, error_code: "broker_account_in_use", sessions: [...] }  (S8R-FIX-052)
       -- S8R-FIX-116: detail ≤ 400 символов (`ACCOUNT_IN_USE_DETAIL_MAX_LEN` = MAX_LENGTH фронта):
       -- при длинном перечне сокращается перечень («и ещё K»), подсказки не режутся; подсказка живых
       -- сессий — «Работающие торговые сессии остановите на странице «Торговля».»

GET    /api/v1/broker-accounts/{id}/positions
       Response: 200 { positions from broker }

GET    /api/v1/broker-accounts/{id}/operations
       Query: ?from=&to=
       Response: 200 { items: [...] }

POST   /api/v1/broker-accounts/discover
       Body: { broker_type, api_key, api_secret? }
       Response: 200 { accounts: [{account_id, type, name}] }

-- Любой endpoint, где ключ счёта не расшифровался (S8R-FIX-119; в т.ч. sandbox-balance, раньше 500):
-- 409 { detail: "Ключ брокерского счёта не расшифровывается — введите ключ заново в «Настройки →
-- Брокер».", error_code: "broker_key_unreadable" } (§7.2)
```

С A-02 `BrokerAccountResponse` (все ответы со счетами, в том числе `POST`) несёт `is_market_data_source: bool`.

### 4.7 AI API

```
POST   /api/v1/ai/chat
       Body: { strategy_id, message, history?: [...] }
       Response: 200 { response, suggested_blocks?, updated_code? }
       (Streaming: SSE endpoint альтернативно)

GET    /api/v1/ai/status
       Response: 200 { available: true, provider: "claude", usage_today: 15, limit_today: 100 }

POST   /api/v1/ai/explain
       Body: { code_snippet }
       Response: 200 { explanation }
```

### 4.8 Notification API

```
GET    /api/v1/notifications/
       Query: ?unread_only=true&page=&per_page=
       Response: 200 { items: [...], unread_count }
       -- S8R-FIX-100: со слэшем — маршруты GET/DELETE объявлены как `/` (реестр §4.0); без слэша
       -- FastAPI отвечает 307 (лишний круг). Фронт (`notificationApi`) ходит по `/notifications/`.

PATCH  /api/v1/notifications/{id}/read
       Response: 200

POST   /api/v1/notifications/read-all
       Response: 200

GET    /api/v1/notifications/settings
       Response: 200 { settings per event type }

PATCH  /api/v1/notifications/settings
       Body: [{ event_type, telegram_enabled, email_enabled, inapp_enabled }]
       Response: 200
```

**A-05 (ТЗ v4.2; ФТ §10.1, §10.2):** `NotificationResponse.undelivered_channels: list[str]` — каналы (`telegram`, `email`), не доставленные после всех повторов; производное от CSV `notifications.channels_failed` (по умолчанию `[]`). Поле есть в ответе `GET /notifications/` и в WS-кадре `notification.new` (при создании оно пустое — отметка видна при следующей загрузке списка, нового WS-кадра нет); на фронте — `Notification.undelivered_channels?` (`src/api/types.ts`).

### 4.9 Settings / Profile API

```
GET    /api/v1/settings/profile
       Response: 200 { user profile + risk settings }

PATCH  /api/v1/settings/profile
       Body: { email?, risk_daily_loss_pct?, ... }
       Response: 200

POST   /api/v1/settings/telegram/link
       Response: 200 { token, expires_at }     -- токен secrets.token_urlsafe(9) (12 символов), TTL 5 мин (S8R-AUDIT-021)

DELETE /api/v1/settings/telegram/unlink
       Response: 204

GET    /api/v1/settings/telegram/status
       Response: 200 { linked: true, chat_id_masked: "***1234" }

-- AI Provider Settings --

GET    /api/v1/settings/ai/providers
       Response: 200 { items: [{id, provider_type, display_name, model_id, api_base_url,
                        daily_limit, monthly_limit, usage_today, usage_month,
                        is_active, is_verified, api_key_masked, last_used_at}] }

POST   /api/v1/settings/ai/providers
       Body: { provider_type, display_name, api_key, model_id, api_base_url?,
               daily_limit?, monthly_limit? }
       Response: 201 { provider (key masked) }

PATCH  /api/v1/settings/ai/providers/{id}
       Body: { display_name?, api_key?, model_id?, api_base_url?,
               daily_limit?, monthly_limit?, is_active? }
       Response: 200

DELETE /api/v1/settings/ai/providers/{id}
       Response: 204

POST   /api/v1/settings/ai/providers/{id}/verify
       -- Тестовый запрос к провайдеру для проверки ключа
       Response: 200 { verified: true, model: "claude-sonnet-4-6-...", latency_ms: 340 }
       Ключ отвергнут / адрес не прошёл SSRF-проверку (S8R-FIX-130):
                 200 { verified: false, error: "<причина>" }, is_verified → false

POST   /api/v1/settings/ai/providers/{id}/activate
       -- Делает указанного провайдера активным (деактивирует остальных)
       Response: 200

GET    /api/v1/settings/ai/usage
       Response: 200 { provider, usage_today, limit_today, usage_month, limit_month }
```

### 4.10 Health API

```
GET    /api/v1/health
       Response: 200 {
           status: "healthy" | "degraded" | "critical",
           components: {
               database: { status, latency_ms },
               broker_tinvest: { status, connected_accounts },
               ai_provider: { status, provider },
               telegram_bot: { status }
           },
           uptime_seconds,
           version
       }
```

### 4.11 Export API

Безопасность файлов (S8R-AUDIT-065, `app/common/export_safety.py`): `Content-Disposition` — `attachment; filename="<ASCII-санитизированное>"; filename*=UTF-8''<percent-encoded>` (RFC 5987) одной функцией `attachment_disposition` для всех скачиваний; ячейки CSV, начинающиеся с `= + - @ \t \r`, получают префикс `'` (`csv_safe_cell`), числа не трогаются.

```
GET    /api/v1/export/backtest/{id}/csv
       Response: 200 (file download, Content-Type: text/csv)

GET    /api/v1/export/backtest/{id}/pdf
       Response: 200 (file download, Content-Type: application/pdf)

GET    /api/v1/export/trades
       Query: ?session_id=&from=&to=&format=csv|xlsx
       Response: 200 (file download)

GET    /api/v1/export/tax-report
       Query: ?year=2025&format=xlsx|csv
       Response: 200 (file download)
```

### 4.12 WebSocket Protocol

**Endpoint:** `ws://localhost:8000/api/v1/ws`

> **Актуально с S8R (S8R-AUDIT-013/058):** токен передаётся cookie при WS-upgrade (не первым сообщением; описание ниже приведено к коду в S8R-FIX-017); отказ аутентификации — close 4401. **Лимиты:** ≤ `WS_MAX_CONNECTIONS_PER_USER` (30, диапазон 10–50) WS на пользователя по всем эндпоинтам, сверх — accept + close 4429 (клиент выдерживает общую паузу 60 с); ≤ 50 подписок на соединение (`limit_exceeded`); канал `market:<TICKER>:<TF>` проверяется как в REST `/candles/subscribe` (ТФ из `Timeframe`); JSON не-объект → `error` без разрыва. Отказ на upgrade (S8R-AUDIT-060) — accept → close с кодом (не до accept): истёкший JWT — 4401 без reason, отзыв по БД — 4401 `revoked`, сбой БД — 1013. Закрытие 4401 при logout — reason `logout`, при смене пароля/краже/отзыве — `revoked` (клиент не делает refresh). Закрытие 4401: logout — только соединения своей пары токенов; смена пароля — все, кроме текущей; повторный refresh (кража) — все; отзыв во время upgrade (память отзывов 120 с) — тоже 4401. `/ws/backtest` после терминального события закрывает клиент. nginx: `location ^~ /ws` (мультиплексор), `limit_conn` 60/IP.

**Авторизация** (приведено к коду в `S8R-FIX-017`; прежнее описание с кадром
`{action:"auth", token:"<jwt>"}` устарело с S8R-AUDIT-013/058 и вводило в
заблуждение — клиент такого сообщения не шлёт):

Токен уходит **cookie при upgrade** — `access_token`, тот же, что у REST. В URL
JWT не попадает никогда (серверные логи, history — gotcha-16), а отдельный
кадр авторизации не нужен: соединение уже установлено от имени пользователя.
Разбор токена — общий с HTTP (`decode_access_payload`, `S8R-FIX-017`: трёх
копий проверки подписи и типа больше нет).

Сервер подтверждает успех сообщением `{"type": "auth_ok"}` — его ждёт клиент,
прежде чем слать `subscribe` (до него подписки отклоняются). Отказ — `accept` и
затем `close` с кодом (не разрыв до accept, S8R-AUDIT-060): истёкший или чужой
JWT — 4401 без reason (клиенту имеет смысл обновить пару), отзыв по БД — 4401
`revoked` (refresh не поможет), сбой БД — 1013.

**Клиент -> Сервер (подписка):**
```json
{
    "action": "subscribe",
    "channel": "market:SBER:D"
}
```

**Клиент -> Сервер (отписка):**
```json
{
    "action": "unsubscribe",
    "channel": "market:SBER:D"
}
```

**Дилерский канал (A-03, ТЗ v4.1):** `{"action": "subscribe", "channel": "dealer:SBER:5m"}` → `subscribed`; ТФ только 1m–4h, иначе отказ `invalid_channel`. Канал публичный, как `market:`; подписка WS берёт держатель потока (`stream_manager.acquire(…, source="dealer")`), отписка и закрытие WS его снимают. События — как у основного канала: `{channel, event: "candle.update", data: {timestamp, open, high, low, close, volume, stream_gen, bar_complete, bar_pending}}` (§2.3, §5.4.5).

**Сервер -> Клиент (свечи):**
```json
{
    "channel": "market:SBER:D",
    "type": "candle",
    "data": { "t": 1711929600, "o": 285.40, "h": 286.10, "l": 284.90, "c": 285.80, "v": 12340000 }
}
```

**Сервер -> Клиент (сделка стратегии):**
```json
{
    "channel": "trades:42",
    "type": "trade_update",
    "data": { "trade_id": 123, "status": "filled", "price": 285.40, ... }
}
```

**Сервер -> Клиент (уведомление):**
```json
{
    "channel": "notifications",
    "type": "notification",
    "data": { "id": 456, "title": "...", "severity": "critical", ... }
}
```

**Сервер -> Клиент (прогресс бэктеста):**
```json
{
    "channel": "backtest:789",
    "type": "progress",
    "data": { "percent": 45, "current_date": "2023-06-15" }
}
```

**Сервер -> Клиент (health):**
```json
{
    "channel": "health",
    "type": "status_change",
    "data": { "component": "broker_tinvest", "status": "disconnected" }
}
```

**Сервер → Клиент (канал пользователя `system:{user_id}`, A-04, ТЗ v4.7).** Канал — только свой (`N == user_id`, `backtest/ws.py::_authorize_channel`); кадр — сообщение шины как есть, `{"channel", "event", "data"}`:

```json
{ "channel": "system:7", "event": "stop_all.changed",
  "data": { "active": true, "enabled_at": "2026-11-09T13:29:00Z", "enabled_source": "web",
            "closed_count": 0, "total_count": 0, "waiting_count": 0, "report": { … } } }
```

`stop_all.changed` (включение — после отчёта, снятие, повтор списка в runtime, если у владельца список сократился) → плашка; `cb.triggered` (копия события CB или «Стоп всему»: `event_type`, `reason`, `scope`, `session_id`, `paused_session_ids` у «Стоп всему») → перечитать историю, статус и состояние стопа; `cb.close_report` (`{event_id, closed_count, total_count}`) → перечитать историю. Кадр — не гарантия: фронт перечитывает состояние при монтировании, возврате фокуса и переподключении WS (ADR 0043).

**Heartbeat:** Сервер отправляет `{"type": "ping"}` каждые 30 секунд. Клиент отвечает `{"type": "pong"}`. При отсутствии pong в течение 90 секунд — соединение закрывается.

### 4.13 Chart Layout и Chart Drawings API (v4.3, Б-02)

**Раскладка графиков** — модуль `app/chart_layouts/` (по образцу `user_favorites`), префикс `/api/v1/chart-layout`, `Depends(get_current_user)`; ключ — пользователь из токена, чужую раскладку не прочитать и не перезаписать (ADR 0024). Имена полей на проводе — snake_case, как во фронтенде (`api/chartLayoutApi.ts`), без перекладки.

```
GET  /api/v1/chart-layout
     Response: 200 { layout: ChartLayoutDoc | null, updated_at: datetime | null }   -- нет раскладки → обе null
PUT  /api/v1/chart-layout
     Body: ChartLayoutDoc
     Response: 200 { layout, updated_at }   -- 422 неверный документ / > 64 КиБ; 401 без входа

ChartLayoutDoc {
  version: 1                                   -- Literal[1]; другая версия — 422
  kind: "1" | "2h" | "2v" | "4" = "1"          -- 1 / 2 рядом / 2 друг под другом / 2×2
  active_panel_id: str                         -- обязан быть id одной из panels
  links: { symbol: bool = false, crosshair: bool = true }
  panels: PanelSettingsDoc[1..4]               -- id уникальны; видимые — первые N по kind, скрытые хранятся
}
PanelSettingsDoc {
  id: str(1..8)                                -- "p1".."p4"
  ticker: Ticker | null = null                 -- app.common.ticker.Ticker; null — пустая панель
  timeframe: Timeframe = "D"
  view: { style: "candles"|"hollow"|"bars"|"line"|"area" = "candles", scale: "normal"|"log"|"percent" = "normal" }
  dealer: bool = false                         -- «Внебиржевые часы»
  adjust: "splits" | "none" = "splits"         -- «Как было» = none
  news: bool = true                            -- v4.9 (Б-07): метки новостей на панели; документ без поля — true (ADR 0066)
  indicators: list[dict] (≤ 50) = []           -- место B6 (A-09), не заполняется
  compare: list[dict] (≤ 10) = []              -- место G5 (Б-06), не заполняется
  replay: dict | null = null                   -- место E11 (Б-08), не заполняется
}
```

Предел документа — `MAX_LAYOUT_JSON_BYTES = 64 * 1024` (JSON в UTF-8, `ChartLayoutDoc._size`): места B6/G5/E11 принимают любые объекты. Документ хранится как `model_dump_json()` после валидации (§3.25). Новому содержимому мест B6/G5/E11 с иной структурой понадобится версия 2 документа — сервер этой версии принимает только 1. Лимитер — категория `general`.

**Рисунки по бумаге** — `app/chart_drawings/router.py` (ADR 0026; прежние маршруты с ТФ в пути сняты без переходного периода: старый фронтенд с новым backend и наоборот не работает — выкатывать вместе):

```
GET    /api/v1/charts/{ticker}/drawings                  -> ChartDrawingResponse[]   -- все ТФ бумаги; ticker_eq; ORDER BY display_order ASC NULLS FIRST, id ASC
POST   /api/v1/charts/{ticker}/drawings                  -> 201 ChartDrawingResponse  -- body ChartDrawingCreate { timeframe: Timeframe, type, data, style? }; ticker = known_canonical
DELETE /api/v1/charts/{ticker}/drawings[?timeframe=<tf>]  -> 204                       -- с v4.6 (Б-04): без timeframe — все ТФ бумаги; с ним — один ТФ
PATCH  /api/v1/charts/drawings/{drawing_id}              -> без изменений
DELETE /api/v1/charts/drawings/{drawing_id}              -> без изменений
```

`timeframe` в теле и query — значения `Timeframe` (иначе 422); статические пути объявлены выше динамических (gotcha-20).

**Б-04 (ТЗ v4.6; ADR 0051, 0053).** `DELETE /api/v1/charts/{ticker}/drawings` без `timeframe` удаляет рисунки пользователя этой бумаги на всех ТФ (тикер — `ticker_eq`, как у списка), с `timeframe` — один ТФ, как раньше; рисунки других бумаг и пользователей не трогаются. Прежний клиент с параметром работает; новый фронтенд со старым backend «Очистить» не выполнит (422) — выкатывать вместе. Новые `DrawingType`: `ray`, `extended`, `hray`, `channel`, `fib`; в `DrawingPayload` — `p3: DrawingPoint | None` и `levels: list[StrictFloat] | None` (`extra="ignore"` остаётся). Проверка `validate_payload_for_type` (422) — и при `POST`, и при `PATCH` со сменой типа:

```
ray, extended   -- обязательны p1 и p2
hray            -- обязателен p1
channel         -- обязательны p1, p2 и p3
fib             -- обязательны p1, p2 и levels: 1…FIB_MAX_LEVELS (20) конечных чисел, |L| ≤ FIB_LEVEL_ABS_MAX (10)
любой тип       -- levels, если переданы, — конечные числа (NaN и ±inf — 422)
```

Пределы уровней одинаковы на сервере (`schemas.py`) и во фронтенде (`primitives/fib.ts`). Тесты — `tests/unit/test_chart_drawings/test_chart_drawings_b04_types.py`, `test_chart_drawings_crud.py` (тест Б-02 «очистка без ТФ — 422» заменён на «очистка без ТФ — все ТФ бумаги»).

**Известные ограничения (ADR 0024, 0028).** Две вкладки или два устройства перезаписывают раскладку друг друга целиком. Защита «не перезаписывать, пока загрузка не удалась» (D01) — только во фронтенде: у документа нет версии, сервер не отличит `PUT` такого клиента от обычного. Тело `PUT` сверх предела всё равно читается целиком (до `client_max_body_size` nginx), отвергает его валидация.

### 4.14 News API (v4.4, Б-01)

**Лента новостей** — `app/news/router.py` (`queries.py`, `schemas.py`), префикс `/api/v1/news`, `Depends(get_current_user)` на всём роутере: лента общая, пользователь нужен только как «вошёл ли»; без входа — 401. Лимитер — категория `general` (путь не в `CATEGORY_PATHS`). Требования — ФТ §20.5.

```
GET /api/v1/news?ticker=&source=&type=&cursor=&limit=50
    ticker: Ticker | null          -- Annotated[Ticker | None, Query()] (gotcha-81); неверный формат — 422; регистр не важен
    source: str(≤ 32) | null       -- news_sources.code; неизвестный — пустая лента
    type:   str(≤ 20) | null       -- код из NEWS_TYPES (report … other); неизвестный — 422
    cursor: str(≤ 200) | null      -- next_cursor прошлой страницы; битый — 422
    limit:  int 1..200 = 50
    Response: 200 NewsPage { items: NewsItemOut[], next_cursor: str | null }
GET /api/v1/news/sources   -> [{ code, name }]    -- все источники, ORDER BY id (порядок строк миграции)
GET /api/v1/news/tickers   -> string[]            -- DISTINCT news_tickers.ticker по алфавиту

NewsItemOut {
  id: int
  source: { code, name }
  title: str
  url: str
  published_at: datetime         -- ISO с «Z» (iso_utc)
  received_at: datetime          -- ISO с «Z»
  tickers: str[]                 -- объединение бумаг всех разметок, по алфавиту
  labels: NewsLabelOut[]         -- { labeler, news_type, scope, sentiment, importance, model, prompt_version, tickers }
  reactions: NewsReactionOut[]   -- { ticker, status, due_at (ISO «Z»), before_pct, after_pct, before_no_trades, after_no_trades }
}
```

- **Порядок и страницы.** `ORDER BY published_at DESC, id DESC`; страница — по курсору, а не по смещению (пока пользователь листает, сверху приходят новые новости — смещение дало бы дубли). Курсор — base64url строки `"<published_at ISO>|<id>"` последней новости страницы; следующая страница — `published_at < t OR (published_at = t AND id < id)`. Запрос берёт `limit + 1` строку: лишняя — признак `next_cursor`.
- **Фильтры** сочетаются: `ticker` — `EXISTS` по `news_tickers` (тикер запроса приводится к верхнему регистру), `type` — `EXISTS` по `news_labels.news_type` (подходит любая разметка новости), `source` — по `news_sources.code`.
- Разметки, бумаги разметок и реакции страницы — тремя запросами на страницу (`_load_details`).
- Проценты реакции — `Decimal` (`Numeric(10,4)`), в JSON — строкой (`"-0.2118"`, gotcha-01); фронтенд держит их строкой до показа. Пустой час — `*_no_trades: true` и процент `null`.
- **С v4.9 (Б-07, ADR 0060)** `tickers` элемента, фильтры `ticker` и `type` и список `/news/tickers` — по **действующей** разметке (`ai`, если есть, иначе `dictionary`; SQL — `queries.active_label_subquery()`), а не по объединению разметок и не по «любой разметке»; реакции в ответе — только бумаг действующей разметки. Новые параметры и поля, маршруты Б-07 — §4.17.

### 4.15 Dividends API и соответствия тикеров (v4.5, A-02)

**Дивиденды** — `app/dividends/router.py` (`schemas.py`), префикс `/api/v1/dividends` задан в роутере (`app.include_router(dividends_router)`). Чтение — любой вошедший (`get_current_user`, без входа — 401); поправки — `require_admin` (не администратор — 403), каждое действие — в аудит той же транзакцией. Лимитер — категория `general`. Тикер пути — `Annotated[Ticker, Path()]` (S8R-AUDIT-065). Требования — ФТ §21.

```
GET /api/v1/dividends
    Response: 200 DividendTickersResponse {
      market_data_account_selected: bool,
      tickers: [{ ticker, discrepancies: int, smartlab_status: "ok"|"unavailable"|null,
                  smartlab_checked_at: datetime|null, tinvest_loaded_at: datetime|null }]   -- по алфавиту
    }
GET /api/v1/dividends/{ticker}
    Response: 200 DividendTableResponse {
      ticker, market_data_account_selected, smartlab_status, smartlab_checked_at, smartlab_error, tinvest_loaded_at,
      has_unresolved_discrepancies: bool,          -- read_dividends(db, ticker, date.min, date.max)
      rows: [{ record_date,
               tinvest:    { record_date, last_buy_date, payment_date, declared_date, amount, currency, dividend_type, regularity } | null,
               smartlab:   { record_date, last_buy_date, period, amount, currency, canceled } | null,
               correction: DividendCorrectionResponse | null,
               effective:  { record_date, last_buy_date, payment_date, amount, currency, source: "tinvest"|"correction" } | null,
               discrepancy: "missing_in_tinvest"|"missing_on_smartlab"|"differs"|"unpaired_correction"|null }]
    }
POST /api/v1/dividends/{ticker}/corrections                     -- администратор
    Body: DividendCorrectionCreate (extra="forbid") {
      action: "add"|"exclude"|"fix", record_date: date,
      amount?: Decimal > 0 (18,8), new_record_date?: date, last_buy_date?: date,
      comment: str 1..1000 (пробелы по краям срезаются, пустой — 422 «Комментарий обязателен») }
      -- add: amount обязателен, new_record_date запрещён; exclude: только record_date и comment;
      -- fix: amount и/или new_record_date
    Response: 201 DividendCorrectionResponse { id, ticker, action, record_date, amount, new_record_date, last_buy_date,
                                               comment, created_at, updated_at, created_by, author }
DELETE /api/v1/dividends/{ticker}/corrections/{correction_id}  -- администратор
    Response: 204 | 404 «Поправка не найдена»
```

- Деньги — `Decimal`, в JSON строкой; даты — `YYYY-MM-DD`; моменты — naive UTC.
- `market_data_account_selected = false` — расхождения не считаются (`discrepancy` везде `null`), страница прячет колонку T-Invest; API при этом отдаёт ранее загруженные строки T-Invest.
- Повтор `POST` с той же `(ticker, record_date)` заменяет поправку (ключ `uq_dividend_corrections`).

**Соответствия тикеров** — `app/admin/router.py` (`admin/ticker_aliases.py`, `admin/schemas.py`), `require_admin` на всём роутере `/api/v1/admin` (без входа — 401, не администратор — 403). Требования — ФТ §11.4.

```
GET    /api/v1/admin/ticker-aliases              -> 200 [TickerAliasResponse { id, ticker, iss_ticker, valid_from, valid_to, created_at, updated_at }]
POST   /api/v1/admin/ticker-aliases              -> 201 TickerAliasResponse
PUT    /api/v1/admin/ticker-aliases/{alias_id}   -> 200 TickerAliasResponse
DELETE /api/v1/admin/ticker-aliases/{alias_id}   -> 204
    Body (POST, PUT): TickerAliasWrite (extra="forbid") { ticker: Ticker, iss_ticker: Ticker, valid_from?: date, valid_to?: date }
      -- тикеры приводятся к верхнему регистру
    422: «Дата «с» не может быть позже даты «по»» | «Период пересекается с существующим соответствием T ← I (с — по)»
    404: «Соответствие тикера N не найдено»
```

### 4.16 Circuit Breaker API: лимиты сессии и история срабатываний (v4.7, A-04)

Все маршруты — `app/circuit_breaker/router.py`, пользователь; чужая или несуществующая сессия — 404 (`CircuitBreakerService._assert_session_owner`, факт существования чужой не раскрывается). Прежние `GET|PUT /config`, `PUT /config/session/{id}`, `GET /events`, `/events/session/{id}`, `/status` не менялись (у `PUT /config/session/{id}` — публикация ниже).

```
GET    /api/v1/circuit-breaker/config/inherited
       Response: 200 InheritedCBValues — по 8 полям CircuitBreakerConfigRequest:
                 { value: str | int | null, source: "global" | "profile" | "default" | "none", text: str }
                 -- text готов к показу: «1 %», «5 % (профиль)», «300 000 ₽», «25 % капитала (профиль)», «50», «60 с», «07:00 (умолчание)», «без ограничения»

GET    /api/v1/circuit-breaker/config/session/{session_id}
       Response: 200 { override: CircuitBreakerConfigResponse | null, inherited: InheritedCBValues }

PUT    /api/v1/circuit-breaker/config/session/{session_id}       -- было; после commit — session.limits_changed {has_session_limits: true}
DELETE /api/v1/circuit-breaker/config/session/{session_id}
       Response: 204 (строки нет — тоже 204); удаляет все строки сессии; после commit — session.limits_changed {has_session_limits: false}

GET    /api/v1/circuit-breaker/history?session_id=&offset=0&limit=20      -- limit 1…100
       Response: 200 CBHistoryPage { items: CBHistoryRow[], total, offset, limit }   -- total — число строк, не событий
CBHistoryRow { id (самое раннее событие строки), event_type, session_limit, first_at, last_at, event_count,
               scope: "all" | "money_class" | "own", money_class: "paper" | "sandbox" | "real" | null,
               action: "pause" | "pause_close" | "stop_all", trigger_value, limit_value, details,
               sessions: [{ session_id, ticker, mode, role: "source" | "paused" }],
               events: [{ id, created_at, session_id, ticker, trigger_value, limit_value, details, close_report }],
               close_report: CBCloseReport | null }      -- сумма отчётов событий строки
```

Правила истории (`app/circuit_breaker/history.py`, ADR 0042, 0045):
- серия — ключ `(event_type, session_limit)`, событие входит, если не дальше `SERIES_WINDOW` = 5 мин от **первого** события серии; `stop_all` не объединяется; строки — по убыванию `last_at`, страница — срез строк;
- фильтр `session_id` — «источник = сессия ИЛИ сессия в `circuit_breaker_event_sessions`»; серии собираются из отобранных событий;
- охват — из `action_taken` первого события; `money_class` — режим сессии-источника; `action` — `stop_all` / `pause` (`CB_PAUSE_WITHOUT_CLOSE`) / `pause_close`;
- тикер и режим сессии (и `money_class`) раскрываются, только если сессия принадлежит пользователю и её `started_at` не позже `created_at` события, иначе `null`;
- `limit_value = 0` у срабатывания (события до S8R-FIX-039) → `trigger_value`/`limit_value` = `null`; JSON `close_report`, который не читается, — `null` и WARNING `cb_close_report_unreadable`;
- три запроса на страницу (события, связи, свои сессии), без списков id в параметрах.

### 4.17 News API Б-07 и фоновые задачи ИИ (v4.9, Б-07)

Маршруты новостей — тот же роутер `app/news/router.py` (`/api/v1/news`, `Depends(get_current_user)` на всём роутере), фоновых задач ИИ — `app/ai/router.py` (`/api/v1/settings/ai`); новых роутеров нет. Лимитер — `general`. Требования — ФТ §20.8–§20.14; ADR 0060–0068. Даты в JSON — с «Z» (`iso_utc`), деньги — `Decimal` строкой.

```
GET /api/v1/news?…&sentiment=&from=&to=           -- + к §4.14
    sentiment: str(≤ 20) | null    -- код SENTIMENTS (positive | negative | neutral | mixed); неизвестный — 422
    from, to: datetime | null      -- интервал published_at включительно; from > to — 422
    NewsItemOut += {
      active: NewsLabelOut | null  -- действующая разметка: ai, иначе dictionary; разметок нет — null
      ai_note: str | null          -- disabled | no_provider | not_verified | limit | unavailable | invalid; только когда действует словарь
    }                               -- tickers и reactions — по действующей разметке; labels[] — все разметки, как было
GET /api/v1/news/by-ticker?ticker=&from=&to=     -> NewsItemOut[]   -- все три обязательны; по возрастанию published_at, не больше 2000;
                                                                    -- from > to и to − from > 400 суток — 422
GET /api/v1/news/securities      -> [{ ticker, shortname }]         -- акции TQBR словаря (кэш процесса 24 ч); нет — 503 «Список акций TQBR недоступен»
GET /api/v1/news/settings        -> NewsSettingsOut { collect_enabled, ai_labeling_enabled, alerts_enabled, chart_marks_enabled,
                                                      alerts_enabled_since, updated_at, ai_verified }
PUT /api/v1/news/settings        (require_admin; иначе 403)
    Body: NewsSettingsPatch { collect_enabled?, ai_labeling_enabled?, alerts_enabled?, chart_marks_enabled? }  -- StrictBool; лишнее поле — 422
    -> NewsSettingsOut; включение «ИИ-разметки» без пройденной проверки — 409 {"detail": {"code": "benchmark_required", "message"}}
POST /api/v1/news/ai-benchmark   (require_admin) -> 202 { id, status }
    409 {"detail": {"code": "no_provider" | "running", "message"}}
GET  /api/v1/news/ai-benchmark   (require_admin) -> { current: { config_id, model_id, prompt_version, verified } | null,
                                                      latest: AIBenchmarkOut | null }
    AIBenchmarkOut { id, config_id, provider_type, model_id, prompt_version, status (running | done | error), passed,
                     precision_pct, recall_hits, recall_total, out_of_list, controls_ok, result, calls,
                     prompt_tokens, completion_tokens, cost_usd, error, started_at, finished_at }
                     -- running старше 15 мин показывается как error «Прогон прерван (перезапуск сервера)»
GET  /api/v1/news/alerts         -> NewsAlertOut[]   -- неудалённые алерты news пользователя, новые первыми
POST /api/v1/news/alerts         -> 201 NewsAlertOut
    Body: NewsAlertFilter { tickers: Ticker[] | null, words: str[] = [], types: str[] = [], sentiments: str[] = [] }  -- extra=forbid
    422: «Для всего рынка задайте слова, тип или тональность», «Выберите хотя бы одну бумагу или «Весь рынок»»,
         «Бумаги нет в списке акций TQBR: …», «Пустое слово в фильтре», «Слово длиннее 100 символов: …»,
         «Не больше 50 слов в фильтре», «Неизвестный тип новости …», «Неизвестная тональность …»
    503: список TQBR недоступен при выбранных бумагах
PATCH  /api/v1/news/alerts/{alert_id}   Body: { paused: StrictBool } -> NewsAlertOut; снятие паузы ставит active_since = now
DELETE /api/v1/news/alerts/{alert_id}   -> 204 (is_active = false, история остаётся)          -- чужой, удалённый — 404
GET    /api/v1/news/alerts/{alert_id}/triggers -> NewsAlertTriggerOut[]   -- новые первыми, до 100
    NewsAlertOut { id, scope: "tickers" | "market", tickers, words, types, sentiments, paused, created_at, last_triggered_at, trigger_count }
    NewsAlertTriggerOut { id, triggered_at, status: sent | unchecked, note: by_dictionary | ai_unavailable | ai_disabled | null,
                          labeler: ai | dictionary, news: { id, title, url, source, published_at } | null }

GET /api/v1/settings/ai/background            (require_admin) -> AIBackgroundResponse
PUT /api/v1/settings/ai/background/provider   (require_admin) Body: { config_id: int | null }   -- null — снять отметку; чужая строка — 404
PUT /api/v1/settings/ai/background/limit      (require_admin) Body: { daily_token_limit: int 0..1 000 000 000 }   -- иначе 422
    AIBackgroundResponse {
      provider: { config_id, display_name, provider_type, model_id, source: "flag" | "active",
                  price_per_1m_prompt, price_per_1m_completion } | null      -- без ключа
      candidates: [{ id, display_name, provider_type, model_id, is_active, is_background }]   -- строки первого администратора
      daily_token_limit: int
      usage_today: { prompt_tokens, completion_tokens, total_tokens, cost_usd, calls }       -- сутки МСК, все задачи
    }
```

- Ответы `PUT` настроек и фоновых задач — полное состояние после изменения. Изменение пишется в аудит (`news.settings_changed` с `before` / `after`, `ai.background_provider_selected`, `ai.background_limit_changed`); правка без изменений аудит не пишет.
- Поле `news` панели раскладки — §4.13.
- **Известные ограничения:** `by-ticker` берёт первые 2000 новостей интервала по возрастанию — при большем числе теряются самые свежие; `usage_today.cost_usd` дня без расхода приходит строкой `"0E-8"`; `DELETE /api/v1/market-data/price-alerts/{id}` (A-03) вид алерта не проверяет и удалит свой алерт `news` по id.

### 4.18 Backtest и Trading API договора исполнения (v4.10, A-06)

Маршруты бэктеста — `app/backtest/router.py` (`/api/v1/backtest`), журнала — `app/trading/router.py` (`/api/v1/trading`); все — `Depends(get_current_user)`, новых роутеров в `main.py` нет. Статические пути `/spread-check`, `/session-defaults`, `/legacy` объявлены выше `/{backtest_id}` (gotcha-20). Decimal в ответах — строкой, время — ISO с «Z». Требования — ФТ §22; решения — ADR 0070–0079.

```
POST /api/v1/backtest, /backtest/run-async, /backtest/{id}/rerun, /backtest/grid      -- + к §4.3
    BacktestCreate / GridSearchRequest += {
      slippage_value: Decimal ≥ 0 = 0.1        -- было 0; одна константа execution.contract.DEFAULT_SLIPPAGE
      slippage_mode: "pct" | "points" = "pct"
      dividend_tax_pct: Decimal 0..100 = 13    -- DEFAULT_DIVIDEND_TAX_PCT; rerun переносит значение бэктеста (у бэктеста до A-06 — умолчание)
    }                                          -- GridSearchRequest: комиссия, проскальзывание и налог — в каждую комбинацию
GET  /api/v1/backtest/{id}                     -- + к §4.3 (ключи дописаны, прежние не переставлены)
    metrics += { bars_per_year: Decimal | null, sharpe_unreliable: bool }
    sharpe_basis: "bars" | "legacy_constants"  -- по версии правил
    legacy_execution_rules: bool               -- execution_rules_version IS NULL
    money_market: { pct: Decimal | null, source: "LQDT" | "RUONIA" | "RUONIA+LQDT" | null, verdict: "better" | "worse" | null }
    price_pnl, dividend_pnl: Decimal | null    -- price_pnl + dividend_pnl = metrics.net_profit
    dividend_tax_pct: Decimal | null
    benchmark.index_code: "MCFTRR" | "IMOEX"   -- MCFTRR при execution_rules_version ≥ 1; доходность — прежнее поле benchmark_imoex_pct
    report_notes: list[str]                    -- шаг цены неизвестен, прогрев неполный, расхождения / сбой выплат
    trades[] += { row_kind: "trade" | "dividend", price_note }
GET  /api/v1/backtest/                         -- элемент списка += legacy_execution_rules
GET  /api/v1/backtest/{id}/export?format=csv|pdf
    -- сделки: колонка «Вид» («Сделка» / «Начисление дивиденда»); подпись доходности индекса — «MCFTRR, %» / «IMOEX, %»

GET  /api/v1/backtest/spread-check?ticker=&slippage_value=&slippage_mode=pct|points
    -> { status: "ok" | "warning" | "unchecked", half_spread_pct, recommended_min_pct, reason }
    -- warning: половина спреда > проскальзывания; recommended_min_pct — половина спреда, вверх до 0,01 %
    -- unchecked: нет счёта для рыночных данных, биржа не торгует (ночь, дилер, аукционы), бумага не в normal_trading,
    --            пустая сторона стакана, сбой, нет ответа за 5 с; reason — текст для окна
GET  /api/v1/backtest/session-defaults?strategy_version_id=&ticker=&timeframe=
    -> { slippage_value, slippage_mode, source: "backtest" | "default", backtest_id: int | null, dividend_tax_pct: "13" }
    -- последний completed-бэктест своей версии на том же тикере и ТФ (completed_at, id по убыванию), иначе 0,1 %;
    -- чужая версия — 404
GET    /api/v1/backtest/legacy  -> { count }      -- свои бэктесты с execution_rules_version IS NULL
DELETE /api/v1/backtest/legacy  -> { deleted }    -- путь одиночного удаления: идущий (queued/running) — отмена у менеджера
                                                   -- фоновых бэктестов, затем сделки и записи; одна транзакция;
                                                   -- новые и чужие не трогаются; лог legacy_backtests_deleted

POST /api/v1/trading/sessions                  -- + к §4.4
    SessionStartRequest += {
      slippage_value: Decimal | null (≥ 0; для pct ≤ 100)
      slippage_mode: "pct" | "points" | null
      dividend_tax_pct: Decimal | null (0..100)
    }
    -- paper: null → session_slippage_default(...) (то же правило, что session-defaults) и 13 %;
    -- sandbox/real: проскальзывание 0 / pct, налог 13 % — поля запроса не учитываются
    SessionResponse += { slippage_value, slippage_mode, dividend_tax_pct }
GET  /api/v1/trading/sessions/{session_id}/trades/{trade_id}/executions
    -> ExecutionResponse[] по executed_at
       ExecutionResponse { id, side: "entry" | "exit", lots, executed_at, rule, rule_price, slippage_value, slippage_mode,
                           contract_price, actual_price, price_diff, partial }
    -- чужая сессия или сделка не этой сессии — 404
```

- `TradeResponse.slippage` у сделок paper — проскальзывание сессии в её единицах (§3.9); у sandbox/real — `null`.
- Фронтенд: `backtestApi.checkSpread`, `getSessionDefaults`, `getLegacyCount`, `deleteLegacy`, константы `DEFAULT_SLIPPAGE_PCT = 0.1`, `DEFAULT_DIVIDEND_TAX_PCT = 13` (`src/api/backtestApi.ts`; тест бэкенда сверяет их с константами договора); журнал — `tradingApi` (`…/executions`); типы — `api/types.ts`.
- **Известные ограничения:** `DELETE /legacy` удаляет по списку id без повторного условия «версия правил IS NULL и владелец» в самом DELETE (гонка только с переиспользованным id); таймаут `spread-check` 5 с охватывает и `ensure_figi_lookup`, пишущий в БД; `session-defaults` принимает любой ТФ.

### 4.19 Screener API и поправки секторов (v4.11, Б-03)

**Скринер** — `app/screener/router.py` (`schemas.py`), префикс `/api/v1/screener` задан в роутере (`app.include_router(screener_router)` в `main.py`); все маршруты — любой вошедший (`get_current_user`, без входа — 401). **Поправки секторов** — `app/screener/admin_router.py`, подключён в `app/admin/router.py` и наследует его префикс `/api/v1/admin` и `require_admin` (без входа — 401, не администратор — 403). Деньги, цены и веса — `Decimal` строкой, время — ISO UTC с «Z», пусто — `null`. Требования — ФТ §23; решения — ADR 0092 (снимок), 0093 (поправки), 0095 (порог).

```
GET /api/v1/screener/snapshot
    Response: 200 ScreenerSnapshotResponse { rows: ScreenerRow[], meta: ScreenerMeta }
              503 {"detail": "Биржа (ISS) не отвечает, повторим через 15 с"}   -- доски ISS у процесса не было ни разу
    ScreenerRow {
      ticker, short_name,
      price, prev_price, change_pct: str | null    -- change_pct = (price / PREVPRICE − 1) × 100, 2 знака, половина к чётному (как LASTTOPREVPRICE)
      price_source: "live" | "iss"                 -- iss: нет живых цен, broker_status ≠ trading, нет цены у брокера
                                                   --      или цена брокера не текущего торгового дня (broker_quote_is_current)
      price_time: str | null                       -- live — время цены T-Invest; iss — UPDATETIME на дату SYSTIME
      turnover_today, median_turnover, capitalization: str | null   -- VALTODAY, медиана VALUE за 60 дат, ISSUECAPITALIZATION (ISS при любом источнике)
      sector_code, sector_name: str | null, sector_source: "override" | "broker" | null
      list_level: int | null, imoex_weight: str | null
      broker_status: "trading" | "break" | "not_on_tqbr" | null, qual_only, short_enabled: bool | null
                                                   -- без счёта для рыночных данных — null у всех строк
      liquid: bool | null                          -- медиана ≥ порога пользователя; null — история не готова
    }
    ScreenerMeta {
      source: "live" | "delayed"                   -- live, если хоть одна строка с живой ценой
      live_status: "ok" | "no_market_data_account" | "no_broker" | "broker_error"
      live_time, iss_time: str | null, iss_stale: bool
      market_data_account: bool
      hint: str | null                             -- одна подсказка: iss_stale → «ISS не отвечает — данные на HH:MM»,
                                                   -- иначе по live_status; ok — null
      liquidity_ready: bool, liquidity_threshold: str, history_sessions: int
      scale_pct: str, scale_basis: "history" | "default"   -- default — 3 % до конца первого заполнения
      universe_source: "directory" | "iss_snapshot"        -- iss_snapshot — до первой синхронизации, колонки брокера пусты
      universe_synced_at, imoex_date: str | null
    }

GET /api/v1/screener/settings    -> 200 ScreenerSettingsResponse { liquidity_threshold: str, default_threshold: str }
PUT /api/v1/screener/settings
    Body: ScreenerSettingsWrite (extra="forbid") { liquidity_threshold: str | number }   -- 0 … 10¹² ₽, не больше 2 знаков
    -> 200 ScreenerSettingsResponse; любой отказ — 422 {"detail": "Порог — число от 0 до 1 000 000 000 000 ₽"}
GET /api/v1/screener/sectors     -> 200 [SectorOption { code, name }]       -- 10 секторов в порядке SECTOR_NAMES

GET    /api/v1/admin/sector-overrides              -> 200 [SectorOverrideResponse]   -- по тикеру
POST   /api/v1/admin/sector-overrides              -> 201 SectorOverrideResponse
PUT    /api/v1/admin/sector-overrides/{id}         -> 200 SectorOverrideResponse
DELETE /api/v1/admin/sector-overrides/{id}         -> 204
    Body (POST, PUT): SectorOverrideWrite (extra="forbid") { ticker: Ticker, sector_code: str }   -- тикер — к верхнему регистру
    SectorOverrideResponse { id, ticker, sector_code, sector_name, updated_at, updated_by }   -- updated_by — имя администратора, удалённый — null
    422: «Неизвестный сектор» | «Тикера нет во вселенной скринера (акции TQBR)»
         -- вселенная — in_screener_universe; до первой синхронизации — бумаги доски ISS EQIN
    409: «Поправка для тикера уже есть» (и гонка двух администраторов на уникальном индексе)
    404: «Поправка сектора N не найдена»
    503: вселенную нечем проверить (справочник пуст и доски ISS не было)
```

- Снимок всегда отдаёт всю вселенную (≈ 261 строка): фильтры — на фронтенде; от пользователя зависят только `liquid` и живые цены, поэтому кэшируются источники, а не ответ (§5.18.4).
- Изменение поправки — аудит той же транзакцией: `screener.sector_override_saved` (`created: true` / `false` и прежние значения `previous`), `screener.sector_override_deleted`, сущность `screener_sector_override`; изменение без отличий не пишется.
- Фронтенд: `src/api/screenerApi.ts` — `screenerApi.getSnapshot`, `getSettings`, `saveSettings`, `getSectors`; `sectorOverridesApi.list`, `create`, `update`, `remove`; типы `ScreenerRow`, `ScreenerMeta`, `ScreenerSnapshot`, `SectorOverride`. Схема `schemas.py` и `screenerApi.ts` меняются вместе; фронтенд-фикстура `components/screener/__fixtures__/snapshot.json` строится настоящим `build_snapshot` и сверяется с ним тестом.

---

## 5. BACKEND-МОДУЛИ: ДЕТАЛЬНАЯ СПЕЦИФИКАЦИЯ

### 5.1 Auth Module

**Ответственный: SEC (реализация) + BACK1 (интеграция)**

**Хеширование паролей:** Argon2id — параметры и детали см. раздел 7.1.

**JWT-токены:** HS256, access/refresh с ротацией — детали см. раздел 7.3. Revoked tokens хранятся в таблице `revoked_tokens` (см. раздел 3.21). Проверка `jti` при каждом запросе. Cleanup истёкших токенов — ежедневно по cron (см. раздел 5.9, job `cleanup_revoked_tokens`).

**Защита от брутфорса:**
- Поля `failed_login_count` и `locked_until` в таблице `users` (персистентное хранение — не теряется при рестарте)
- При неудачном логине — инкремент `failed_login_count` в БД
- После 5 неудачных попыток — установка `locked_until = now + 15 min`, блокировка аккаунта
- При блокировке — запись в audit_log
- Успешный вход — сброс `failed_login_count = 0` и `locked_until = NULL`

**Мастер первого запуска:**
- При старте: проверка `SELECT COUNT(*) FROM users`. Если 0 — endpoint `/api/v1/auth/setup` доступен без JWT. После создания первого пользователя — endpoint становится недоступен (возвращает 403).
- Frontend UX flow: при загрузке `LoginPage` вызывается `GET /api/v1/auth/setup-status`. Если `is_configured: false` — автоматический редирект на `/setup` (страница регистрации первого пользователя). `SetupPage` сама проверяет `setup-status` и при `is_configured: true` уводит на `/login` (S8R-AUDIT-033: ссылки «Создать аккаунт» на странице логина больше нет). На странице Setup — ссылка «Уже есть аккаунт? Войти» → `/login`.
- Дополнительные учётные записи (S8R-AUDIT-033, решение заказчика 2026-09-24): только администратор, `POST /api/v1/admin/users` (`require_admin`, тело `{ username, password }` с той же валидацией, что setup; 201 `UserResponse`, 409 — имя занято; создаваемый пользователь всегда `is_admin=false`). UI для этого нет.
- При недоступности backend — LoginPage показывает Alert «Backend недоступен» и ссылку на Setup.

**Подготовка к 2FA (фаза 2):**
- Таблица `users` проектируется с учётом будущих полей (добавляются миграцией в фазе 2): `totp_secret: String(32), NULLABLE`, `totp_enabled: Boolean, DEFAULT False`.
- Логин-flow содержит extension point: после проверки пароля — вызов `verify_second_factor(user)`, который в фазе 1 всегда возвращает `True`. В фазе 2 — проверка TOTP-кода.

### 5.2 Strategy Engine (text -> blocks -> code pipeline)

**Ответственный: BACK2 (AI-часть + парсер + генератор), BACK1 (ревью)**

**5.2.1 Блочная схема — внутренний формат:**

Блоки хранятся в JSON. Каждый блок имеет тип, параметры и связи:

```json
{
    "blocks": [
        {
            "id": "b1",
            "type": "indicator",
            "name": "EMA",
            "params": { "period": 20, "source": "close" },
            "output": "ema_20"
        },
        {
            "id": "b2",
            "type": "condition",
            "name": "crossover",
            "inputs": ["ema_20", "ema_50"],
            "params": { "direction": "up" },
            "output": "cross_signal"
        },
        {
            "id": "b3",
            "type": "entry_signal",
            "name": "long_entry",
            "conditions": ["cross_signal", "rsi_above_30"],
            "logic": "AND"
        },
        {
            "id": "b4",
            "type": "exit_signal",
            "name": "long_exit",
            "conditions": ["cross_down"],
            "stop_loss_pct": 2.0,
            "take_profit_pct": 5.0
        },
        {
            "id": "b5",
            "type": "money_management",
            "sizing": "pct_deposit",
            "value": 10.0,
            "max_positions": 1
        },
        {
            "id": "b6",
            "type": "time_filter",
            "start": "10:00",
            "end": "18:00",
            "timezone": "Europe/Moscow"
        }
    ]
}
```

**Версионирование формата:** JSON блочной схемы содержит корневое поле `"schema_version": 1`. При изменении формата блоков в будущих версиях приложения — добавляется миграция в Alembic, которая обновляет `blocks_json` всех записей `strategy_versions` до нового формата. Это гарантирует обратную совместимость при обновлении приложения.

**5.2.2 Режим B — парсер шаблона:**

`block_parser.py` реализует regex-based парсер для шаблона из Приложения A. Логика:
1. Разбить текст по секциям (ИНДИКАТОРЫ, УСЛОВИЯ ВХОДА, ...; поддерживается и старый нумерованный формат).
2. Для каждой секции — набор правил парсинга.
3. Распознать индикаторы по паттернам: `EMA(period=20)`, `RSI(14)`, `Bollinger Bands(20, 2)`.
4. Распознать условия: "пересекает снизу вверх" -> crossover up.
5. При ошибке парсинга — возврат `warnings` с указанием непонятных секций.

**Источник истины формата** — `app/strategy/template_format_rules.py`: `FORMAT_DESCRIPTION`,
`SECTION_HEADERS` (семь секций) и `EMPTY_TEMPLATE_7`. Шаблон, который UI отдаёт
пользователю (`EMPTY_TEMPLATE` и `TEMPLATE_EXAMPLE` в `TemplateModeModal.tsx`),
сверяется с ним тестом `tests/unit/test_strategy/test_template_format_sync.py`:
тест разбирает `.tsx` и сравнивает состав секций, а пример дополнительно
прогоняет через `TemplateParser().parse` — он обязан дать блоки без
предупреждений. Проверка добавлена потому, что шаблон в модалке разошёлся с
парсером (десять нумерованных секций против семи) и разъезд ничем не
обнаруживался: `S5R-BLOCKLY-MODE-B-MODAL`. Приём тот же, что у пары
`EVENT_MAP` ↔ `EVENT_TYPE_LABELS`.

**Точка входа (Q2=a, 2026-08-10):** кнопка «Режим B» в шапке `StrategyEditPage`
рядом с «Генерировать»; `onApply` — тот же `handleApplyTemplate`, что у
AI-панели, поэтому путь «текст → блоки → пометка кода устаревшим»
(`setCodeOutdated(true)`) един для обоих источников блоков. До этого компонент,
endpoint и парсер существовали, но не были связаны ни с одним элементом UI.

**5.2.2а Отчёт разбора описания (блок A-05, B12; ТЗ v4.2).** Требования — ФТ §3.2, §8.2, Приложение A. Пути — от `backend/app/`.
- **Типы** (`strategy/block_parser.py`): `ParseIssue(level: "problem" | "note", line: int | None, fragment: str, message: str, hint: str)` — `line` с 1 по тексту ровно в том виде, в каком он пришёл в запрос, `None` — запись о тексте в целом; `fragment` обрезается до `FRAGMENT_MAX_LEN = 200`. `ParseReport(issues)` — `status` (`ok` / `notes` / `problems`), `has_problems`, `to_dict()`, `from_dict()`. `ParseResult(blocks_json, report)`, `warnings` — вычисляемое свойство (тексты записей). `TemplateParser().parse(text) -> ParseResult` — сигнатура прежняя.
- **Признак формата ИИ-чата** — `is_reference_format(text)`: строка-заголовок из `SECTION_HEADERS` (`SECTION_HEADER_LINE_PATTERN`, двоеточие допустимо) **и** ссылка `\[I\d+\]` (`INDICATOR_REF_TOKEN_PATTERN`). Одна функция для разбора и для сохранения версии; прежний признак «подстрока `[I`» принимал за формат свободный текст вида «[Intraday]».
- **Формат ИИ-чата** — `_parse_reference`: секции с номерами строк, каждая строка сверяется с правилом **целиком** (`fullmatch`; правила — раздел «3а» `template_format_rules.py`: `INDICATOR_LINE_PATTERN`, атомы `ATOM_CROSSOVER/COMPARE/IN_ZONE_PATTERN`, `RISK_*`, `SIZING_*`, `FILTER_TIME_LINE_PATTERN`, `GRID_MARKER/DIFF_LINE_PATTERN`, `EMPTY_VALUE_MARKERS`). Условие — токены и рекурсивный спуск: ИЛИ < И < НЕ < (скобки | атом). В `blocks_json` логика — `{"type": "logic", "operator": "AND" | "OR" | "NOT", "conditions": [id, …]}` (у `NOT` один id), дети раньше родителя, у сигнала `"condition"` = id корня; `(A) И (B) ИЛИ (C)` → `OR[AND[A, B], C]`. Прежней формы `{"type": "condition", "operator": "not", "operand": null}` нет. Параметры индикатора хранятся как написаны (`INDICATOR_PARAMS`: MACD `fast/slow/signal` или `*_period`, Stochastic `%K/%D` или `k_period/d_period`, BB `period/std_dev`; позиционные — по порядку); `UNSUPPORTED_PARAM_FALLBACK[("MACD", "line")]` — заметка «используется линия MACD». Список для текста — `AVAILABLE_INDICATORS_TEXT`. Строка с «проблемой» в блоки не попадает целиком; по одному `entry_signal`, `exit_signal`, `time_filter` — вторая строка входа, выхода или фильтра даёт «проблему» (IR оставляет только последний сигнал). Записи сортируются по номеру строки.
- **Старые форматы** (нумерованные секции, заголовки без ссылок) — прежний разбор `_parse_legacy`; отчёт — заметка `_LEGACY_FORMAT_NOTE` и прежние предупреждения заметками без номера строки. Пустой текст — заметка «Пустой текст шаблона».
- **Правила уровней** («проблема» / «заметка», умолчания — нормы формата без заметки) — ФТ §3.2. Подсказка «как обойти» — `HINT_REMOVE_OR_EDITOR` («уберите строку или соберите эту часть блоками на вкладке «Редактор»»): режима «код» в редакторе нет, бриф его называл.
- **Сохранение** — `strategy/service.py`: `description_parse_report(text) -> str | None` (не формат ИИ-чата → `None`; `has_problems` → `DescriptionHasProblemsError(report)`, иначе `json.dumps(report.to_dict())`); `create_version(…, *, inherit_parse_report_from=None)` зовёт её **до** проверки идемпотентности; `_version_from` копирует `parse_report`. `common/exceptions.py`: `DescriptionHasProblemsError(ValidationError)`, `error_code = "description_has_problems"`, обработчик — 422 `{detail, error_code, report}` (§4.2).
- **Синхронизация шаблонов:** `tests/unit/test_strategy/test_template_format_sync.py` сверяет с `EMPTY_TEMPLATE_7` и «Пустой шаблон» панели (`SharedDescriptionPanel` берёт `EMPTY_TEMPLATE` из `TemplateModeModal.tsx`); пример ИИ-чата (`STRATEGY_TEMPLATE_FORMAT`, `app/ai/prompts.py`) обязан разбираться с пустым отчётом (`test_parse_report.py`). Эталоны «Заполнить по схеме» — `frontend/src/utils/__tests__/fixtures/generatedTemplatesBlocks.json` ↔ `tests/unit/test_strategy/test_generated_template_parse.py`. Фронтовые потери починены: ключи MACD и Stochastic в `flatBlocksToWorkspace.ts`, Volume в `blocksToTemplate.ts` — `[I1] Volume()`.
- **Тесты** — `tests/unit/test_strategy/test_parse_report.py` (набор потерь, четыре «непереводимых» описания), `tests/test_routers/test_strategy_router.py` (422, повтор сохранения с исправленным текстом); E2E — `frontend/e2e/a05-parse-report.spec.ts` (настоящий backend, отчёт после перезагрузки).
- **Вне блока:** передача отчёта в ИИ-чат (`app/ai/*`) — свой блок B12.
- **Ограничения:** длина описания не ограничена, строка с длинной серией пробелов разбирается квадратично (~5–6 с на 50 КБ, стопорит процесс — `template_format_rules.py`, `block_parser.py`); Blockly-стратегия с двумя блоками входа не сохраняется, пока лишнюю строку входа не убрать из описания, а запись отчёта называет первую строку, хотя исполняется последняя; после «Применить» в «Режиме B» отчёт в панели — по тексту окна, а текст панели прежний (`TemplateModeModal.tsx`, `StrategyEditPage.handleApplyTemplate`).

**5.2.3 Генератор кода:**

Генерация кода — `ir_codegen.ir_to_backtrader_code` (§5.2.4) из IR валидированных блоков; прежний `code_generator.py` удалён (S8R-AUDIT-019). Исторически генератор принимал JSON блочной схемы и генерировал Python-класс для Backtrader:

```python
# Шаблон генерации
class GeneratedStrategy(bt.Strategy):
    params = (
        ('ema_period', 20),
        ('rsi_period', 14),
        ...
    )

    def __init__(self):
        self.ema = bt.indicators.EMA(period=self.p.ema_period)
        ...

    def next(self):
        # Условия входа/выхода
        ...
```

Маппинг блоков на код:
- `indicator` -> `bt.indicators.XXX`
- `condition` -> Python-выражение в `next()`
- `entry_signal` -> `self.buy()` с условиями
- `exit_signal` -> `self.sell()` / `self.close()`
- `money_management` -> `bt.sizers.PercentSizer` или кастомный sizer
- `time_filter` -> проверка `self.data.datetime.time()` в `next()`

**5.2.4 Единый кодоген через StrategyIR (S8R, BUG-31):**

> С приёмки Sprint 8 backtrader-код генерируется из **того же промежуточного
> представления** (`StrategyIR`), что используют интерпретатор стратегий
> (live-торговля) и parity-сверка. Это единственный источник правды:
>
> ```
> blocks_json (канонический Blockly) → parse_blocks → StrategyIR ─┬─→ ir_to_backtrader_code → Python (backtrader)
>                                                                  └─→ evaluate/interpreter (live + parity-shadow)
> ```
>
> **Зачем:** раньше код для backtrader строила отдельная цепочка (фронтовые
> Blockly-генераторы → плоский формат → `code_generator.py`), которая
> рассинхронизировалась с блоками (теряла индикаторы из-за разного регистра имён,
> не вставляла `buy`) — бэктест прогонял НЕ ту стратегию. Единый IR убирает этот
> класс багов: backtrader и live согласованы по построению.
>
> **Реализация:** `app/strategy/ir_codegen.py` (`ir_to_backtrader_code`). `StrategyIR`
> (`app/strategy/ir.py`) включает индикаторы, условия входа/выхода, а также SL/TP,
> размер позиции и тайм-фильтр. Stochastic генерируется как `bt.indicators.StochasticFast`
> (числовой паритет %K с интерпретатором). Старый `code_generator.py` удалён вместе
> с его тестами (S8R-AUDIT-019), parity-тесты идут по `ir_codegen`. Подробности и ловушка —
> `Develop/stack_gotchas/gotcha-35-dual-block-translators-drift.md`.
>
> **Валидация значений полей блоков (S8R-AUDIT-002).** Значения `fields`
> Blockly-блока проверяются по схеме `BLOCK_FIELD_SCHEMA`
> (`app/strategy/block_allowlist.py`) на двух рубежах: на входе API
> (`validate_blocks_json` → 422 с текстом «блок, поле, допустимые значения») и в
> `parse_blocks` перед кодогеном. `SOURCE` ∈ {open, high, low, close, volume};
> числовые поля — только числа (bool и значения вне диапазона float — нет;
> числовая строка допустима, gotcha-01); enum-поля (`OPERATOR`, `DIRECTION`,
> `RIGHT_TYPE`, `TYPE`, `MODE`) — из перечислений фронта. Кодоген подставляет
> выражение источника из словаря соответствий (`_BT_SOURCE_EXPR`): пользовательская
> строка в исходник стратегии не попадает никогда. Запуск бэктеста и Grid Search
> для версии с невалидными блоками → 422 «Стратегия не может быть запущена: …».

### 5.3 Backtest Engine

**Ответственный: BACK1**

**5.3.1 Архитектура:**

`BacktestEngine` — обёртка над `backtrader.Cerebro`. Процесс:

1. Получить OHLCV-данные из `MarketDataService.get_candles()` (кеш или загрузка); с v4.10 (A-06) — с прогревом до даты начала (§5.17.7).
2. Валидировать данные через `OHLCVValidator`.
3. Создать ~~`bt.Cerebro()`~~ — с v4.10 (A-06) `cerebro_setup.build_cerebro`, одна настройка Cerebro и брокера для бэктеста и Grid (§5.17.2).
4. Загрузить стратегию в sandbox (`SandboxExecutor.load_strategy(code)`).
5. Добавить data feed через `bt.feeds.PandasData`.
6. Установить broker params: commission, slippage, initial_capital.
7. Добавить analyzers: ~~`bt.analyzers.SharpeRatio`~~ (удалён в v4.10 — годовой Sharpe `metrics.annual_sharpe` по значениям капитала баров теста, §5.17.8), `bt.analyzers.DrawDown`, `bt.analyzers.TradeAnalyzer`, custom `EquityCurveAnalyzer`.
8. Запустить `cerebro.run()` в отдельном процессе (multiprocessing) с таймаутом 30 секунд для CPU-limit.
9. Собрать метрики из analyzers.
10. Рассчитать benchmark (Buy & Hold, ~~IMOEX~~ — с v4.10 MCFTRR, у бэктестов до договора исполнения IMOEX; Buy & Hold — с дивидендами нетто, §5.17.6); с v4.10 — денежный рынок (§5.17.8).
11. Сохранить результат в БД.
12. Отправить событие `backtest.completed` в event bus.

**5.3.1а Правила аукционов (A-03, ТЗ v4.1; ФТ §4.2; ADR 0017, 0018).** Правила живут в брокере Backtrader, а не в коде стратегии (`app/strategy/*` не менялся):
- `backtest/auction_broker.py`: `AuctionAwareBroker(bt.brokers.BackBroker)` ставится в `BacktestEngine._setup_cerebro`. На баре аукциона открытия ордера не исполняются — остаются в очереди и исполняются на следующем баре по его open (сигнал на 23:00 → open 07:00; стоп, пробитый ночным гэпом, → open 07:00; в выходной — 10:00). На баре аукциона закрытия ордер, открывающий или увеличивающий позицию, снимается (сигнал теряется, на 19:00 не переносится); закрывающий исполняется.
- Метки баров (`AuctionMarks`, `auction_marks`) строит основной процесс (`BacktestEngine._auction_marks`) по таблице периодов (`TradingSchedule.is_opening_auction_bar` / `is_closing_auction_bar`, §5.9.1) — она загружена там, а не в spawn-процессе исполнения стратегии. Бар аукциона открытия — бар внутридневного ТФ целиком до начала торгов своего дня, пересекающий аукцион (1h 06:00, 4h 03:00, в выходной 1h 09:00 МСК); бар аукциона закрытия — начало бара в аукционе закрытия (5m 18:55). D/W/M баров аукционов нет.
- D/W/M: цена исполнения бара — open первой **сохранённой** минуты дня не раньше начала торгов (`_StoredMinuteOpens`: только `MarketDataService._read_stored(exclude_iss=True)` и покрытие `ohlcv_coverage` дня в окне торгов); за минутами в T-Invest и ISS не ходит (D01: `get_candles(1m, mode="backtest")` докачивал бы годы минут через общие лимиты). Минут или покрытия нет, сбой чтения — open бара и `backtest_trades.price_note = "цена аукциона"` (`AUCTION_PRICE_NOTE`). Следствие: цена сделки зависит от того, что уже лежит в БД (до и после фоновой догрузки — разные цены).
- Теневой прогон интерпретатора `parity.interpreter_trades(…, timeframe=)` живёт по тем же правилам; расхождение двух мест ломает проверку паритета.
- Фильтр O=H=L=C в `get_candles` удалён (§5.6.6): бары аукциона открытия есть в данных бэктеста.
- ~~`backtest/metrics.py`: `TRADING_MINUTES_PER_DAY = 805` — число (раньше выводилось из `DEFAULT_TRADING_START/END` и `SESSION_BREAK_*`, которых больше нет); годовой Sharpe по фактическому числу баров — A-06.~~ Удалено в v4.10 (A-06): годовой Sharpe — по барам теста (§5.17.8).
- ~~**Ограничения:** `GridSearchEngine` (`backtest/grid.py`, свой `bt.Cerebro()` без `AuctionAwareBroker`) правил аукционов не применяет; окно времени стратегии в бэктесте сравнивается с UTC, а задаётся в МСК (сдвиг 3 ч) — передано в A-06.~~ Сняты в v4.10 (A-06): Grid — `build_cerebro` с `AuctionAwareBroker` и метками аукционов (§5.17.2), окно — в МСК (§5.17.9). С v4.10 намерение входа на баре аукциона открытия ждёт следующего бара, на баре аукциона закрытия — снимается (`execute_entry_intent`).

**5.3.1б Размер позиции в лотах (блок A-05, C4; ТЗ v4.2).** Требования — ФТ §4.1, §4.3, §4.5. Пути — от `backend/app/`.
- **Одна формула** — `strategy/sizing.py`: `PERCENT_LOT_EXPR = "int(cash * pct / 100 / (price * lot * (1 + commission_pct / 100)))"` — число **лотов** «%»-позиции; `percent_lot_size(cash, pct, price, lot_size=1, commission_pct=0.0) -> int` считает то же (тест на эквивалентность; вызов без лота и комиссии — прежний счёт, на него опирается межмодульный тест live, совпадение с `trading.engine.calculate_position_lots` проверено тестом, `trading/*` не менялся); `shares_for_lots(lots, lot_size)`; `position_cost(lots, price, lot_size, commission_pct)` — деньги на вход с комиссией (проверка «хватит ли на N лотов»); `SERVICE_STRATEGY_PARAMS = {"lot_size", "commission_pct"}`. **С v4.10 (A-06)** `PERCENT_LOT_EXPR` удалён: формула лотов — только функции `sizing` (`entry_lots(*, fixed_lots, pct, cash, price, lot_size, commission_pct)`, `entry_affordable(lots, price, cash, lot_size, commission_pct)`), их зовут обёртка стратегии, тень и роутер (§5.17.2).
- **Кодоген** (`strategy/ir_codegen.py`): в `params` сгенерированной стратегии всегда `('lot_size', 1)`, `('commission_pct', 0.0)`; размер ордера — **акции**: без блока размера `size=self.params.lot_size`, «N лотов» — `position_size_lots * lot_size`, «%» — `PERCENT_LOT_EXPR` (подстановка имён по целым словам: `cash` → `self.broker.get_cash()`, `pct` → `self.params.position_size_pct`, `price` → `self.data.close[0]`, `lot` → `self.params.lot_size`, `commission_pct` → `self.params.commission_pct`) × `lot_size`. Капитал «%» — свободные деньги на баре. **Заменено в v4.10 (A-06):** кодоген ордеров не подаёт и размер не считает — `next()` ставит намерение `contract_request_entry(is_long)`, лоты по цене исполнения считает `WrappedStrategy.next_open` (§5.17.2); параметры размера, уровней, лота и комиссии остаются в `params`.
- **Движок** (`backtest/engine.py`): `BacktestEngine.run(…, lot_size=1)`; `service_strategy_kwargs(strategy_class, params, *, lot_size, commission_pct)` кладёт служебные параметры поверх `override_params`/комбинации Grid (перебором не подменяются) — только стратегии, которая их объявила (backtrader бросает `TypeError` на неизвестный kwarg). `TradeRecorder` получает тот же лот: `volume_lots = |size| // lot_size`, штуки — `volume_shares` (P&L %, цена выхода), `volume_rub` — как раньше. Grid (`backtest/grid.py`, `_prepare_grid_workload(…, lot_size)`) передаёт лот в payload воркера.
- **Тень и роутер:** `parity.interpreter_trades(…, lot_size=1)` и `_affordable` (в том числе «N лотов»: N × лот × цена с комиссией ≤ cash; short стоимостью не отклоняется — брокер backtrader с `shortcash=True`); после мержа с A-03 тень совмещает правила аукционов с проверкой денег (подача — по close сигнального бара до аукционов, исполнение — по open бара исполнения). `router._position_size_warning(…, lot_size=1, commission_pct=0.0)` — «0 сделок» с лотом и ценой лота (`lot_text`), для «N лотов» и входа без блока размера — только long. **С v4.10 (A-06)** деньги на вход проверяются и у шорта (`sizing.entry_affordable`, как paper; ADR 0070), тень — по договору: цена, лоты и деньги — по цене исполнения (§5.17.2).
- **Лот запуска** — `backtest/lot.py`: `resolve_backtest_lot(db, ticker, user_id)` — `MarketDataService.ensure_lot_size_strict(ticker, user_id=…)` (свежий справочник или боевой T-Invest пользователя — ответ пишется в `instruments`), при `LotSizeUnavailableError` — `ensure_lot_size(…)` (ISS / песочница / устаревший кэш / 1, в справочник не пишется); `max(lot, 1)`. `market_data/*` не менялся. `lot_unverified_warning(backtest_lot, instrument)` — при показе: лот справочника ≠ лоту бэктеста → текст с обоими лотами; иначе `lot_size_synced_at` пуст или старше `LOT_SIZE_TTL` (7 дней) → «Лот N не сверен с T-Invest…». `is_legacy_share_lots(backtest_lot, directory_lot)` — одно правило пометки; флаг не хранится, а вычисляется при показе (флаг миграции по справочнику пропустил бы MTSS: до сверки лот 1).
- **Тесты** — `tests/unit/test_strategy/test_sizing.py`; `tests/unit/test_backtest/test_engine_lots.py`, `test_grid_lots.py`, `test_parity_lots.py`, `test_router_lots.py`, `test_backtest_lot.py` (MTSS 1 → 10 на заглушке T-Invest), `test_export_lots.py`.
- **Ограничения:** предупреждение «лот не сверен» через 7 дней появляется и у бэктеста на сверенном лоте; при недоступном T-Invest запуск ждёт лот до ~20 с (`backtest/lot.py`); тест лота MTSS подменяет приватный `MarketDataService._fetch_lot_size_from_tinvest` — перепроверить после изменений `market_data/*`.

**5.3.2 Прогресс бэктеста:**

Custom analyzer `ProgressAnalyzer` вызывает callback каждые N баров, передавая текущую дату и процент завершения. Backend пушит это через WebSocket.

**5.3.3 Валидация OHLCV:**

```python
class OHLCVValidator:
    def validate(self, df: pd.DataFrame, ticker: str, timeframe: str) -> ValidationResult:
        errors = []
        warnings = []
        # 1. High >= Low
        # 2. OHLC в пределах ±50% от предыдущего Close
        # 3. Нет дубликатов по timestamp
        # 4. Нет пропусков (с учётом торгового календаря)
        return ValidationResult(errors=errors, warnings=warnings, is_valid=len(errors)==0)
```

**Пропуски дневок раньше календаря ISS (v4.5, A-02; находка тестового стенда 2026-11-06, `acbea74`; ADR 0033, «Последствия»).** Календарь (`MOEXCalendarService`, исключения `dailytable`) знает праздники только с `known_from` = 09.05.2018; раньше `validate_ohlcv` считает праздник будним днём, и после отказа от фантомных дневок (§5.6.8) праздник становился «пропуском» (SBER D 2017–2019: 08–09.05.2017, 01–02.01.2018). `backtest/service.py::ohlcv_quality_report` для D: если есть пропуски, отрезки между соседними дневками (`daily_gap_spans`) раньше `known_from` уточняются днями торгов ISS `history` бумаги — `iss_history.iss_trading_days(db, client, ticker, a, b)` (тикер ISS — по соответствиям; один запрос на отрезок), повторный `validate_ohlcv(…, known_days)`. Кэш `_ISS_GAP_DAYS` на процесс (ключ `(ticker, a, b)`, предел 4096 записей), общий предел ожидания `ISS_KNOWN_DAYS_TIMEOUT_SEC` = 15 с; пустой ответ ISS (облигация, валюта) или сбой/таймаут (`ohlcv_daily_gaps_iss_unavailable`, warning) — по календарю, как до A-02. Тесты — `tests/test_market_data/test_daily_gaps_before_calendar.py` (настоящие дневки SBER и страницы ISS `history` 2014–2018), `tests/unit/test_backtest/test_api.py` (через `GET /backtest/{id}`). Проверено живым бэктестом на тестовом стенде 09.11 (`088b9e6`, копия рабочей БД): SBER D 2017–2019 — `completed` без предупреждения, 758 дневок, 105 сделок, +36 987,74 ₽ — те же цифры, что у прогона 06.11 с ложным «пропуски — 2 (первый — 08.05.2017)»; шов на месте (06.03.2018 — T-Invest, 07.03 — ISS, 13.06 — T-Invest).

**5.3.4 Оптимизация (Grid Search):**

Для каждой комбинации параметров — отдельный запуск `cerebro.run()`. Используется `multiprocessing.Pool` для параллелизации. Максимум workers = `os.cpu_count() - 1`. Результаты агрегируются в таблицу. Overfitting-предупреждение: если топ-5 результатов имеют разброс метрик > 50% от лучшего. **С v4.10 (A-06)** комбинация считается `build_cerebro` с тем же договором, метками аукционов, дивидендами и прогревом, что одиночный бэктест; комиссия, проскальзывание и налог — из `GridSearchRequest`; Sharpe — `annual_sharpe` по значениям капитала анализатора `grid.BarValues` (§5.17).

### 5.4 Trading Engine

**Ответственный: BACK1**

**5.4.1 Компоненты:**

- `TradingSessionManager` — управление жизненным циклом сессий (start, pause, stop, resume). Делегирует `SessionRuntime.start/stop` для live-части.
  `S8R-AUDIT-029`: остановка — одна секция под `session_lifecycle`, защищённая от отмены запроса (`run_shielded_from_cancel`, перехват каждой повторной отмены anyio — gotcha-77, grace 275 с): снять listener без события → закрыть позиции (`_close_trades`: sandbox/real до 5 параллельно в собственных `AsyncSession`, paper по одной, дедлайн 120 с) → `stopped` → `positions.close_failed` (кроме позиций с принятым брокером встречным ордером) → одно `session.stopped`. Повторный «Стоп» закрывает остаток (422). DELETE сессии — под тем же локом. Пауза не публикует `session.stopped`.
- `SessionRuntime` *(введён в S5R.2, `app/trading/runtime.py`)* — **единственная production-точка вызова** `SignalProcessor.process_candle()`. Держит per-session listener'ы на каналах EventBus `market:{ticker}:{timeframe}`, загружает скользящее окно истории свечей при старте, реагирует на closed-свечи, публикует события в канал `trades:{session_id}`. Создаётся один раз в `main.py lifespan`, вызывает `restore_all(active_sessions)` при старте backend, `shutdown()` при остановке.
- `SignalProcessor` — получает от runtime скользящее окно свечей (`candles: list[dict]`, ~~по умолчанию N=200~~ с v4.10 (A-06) N = `warmup_bars(ir)` сессии, §5.17.7), прогоняет через стратегию в sandbox, генерирует сигналы. Сохранены алиасы `candle_open/candle_high/candle_low/candle_close/candle_volume` для обратной совместимости со старыми сгенерированными стратегиями.
- `OrderManager` — конвертирует сигналы в ордера, проверяет circuit breaker, отправляет в брокер, закрывает позиции (`close_position`, `close_all_positions`).
  Ревью MEDIUM (`S8R-REV-MQ`): довыполнение частичного входа — `executed_commission` кумулятивен по ордеру и пишется итогом (`_apply_entry_order_commission`): факт > 0 (после округления до копеек) заменяет оценку и не уменьшает записанный факт (источник `broker`); иначе записанный факт сохраняется, оценка пересчитывается на весь оборот входа, сделка без записи учитывается правилом первого fill (`_record_broker_commission`). Правило «значение брокера — факт» — одна функция `_broker_commission_is_fact` (семантика нуля — S8R-FIX-049). Средняя цена None/0 не перезаписывает цену и объём входа. Правило первого fill не изменилось.
  `S8R-FIX-049`: комиссия хранится по сторонам (`entry_/exit_commission`, `_source`, `_at`; итог `commission` = сумма, `_sync_commission_total`, вкл. paper). Правило факта одно для входа и выхода (`_broker_commission_is_fact`): > 0 после `quantize_money` у терминального ордера — `broker`; частичный кумулятив — `broker_partial`; 0 на real после окна `COMMISSION_SETTLE_WINDOW_SEC` = 600 с от исполнения — провизорный `broker_zero` (позднее значение > 0 заменяет); 0 в sandbox — не факт; вход без цены — `unknown`; итог `estimated`, если хоть одна сторона `estimated`/`unknown`. Досверщик — в `_periodic_recovery_loop` (каждые 300 с): SQL-отбор с LIMIT 20, сделки не старше 7 дней, стороны читаются у брокера независимо, отметка `commission_checked_at` до сети, лок `close_trade` ждёт ≤ 5 с (иначе — следующий проход); у закрытой сделки разница корректирует `DailyStat` дня закрытия (`closed_at` UTC) одной транзакцией. Сделки, закрытые до миграции `c04dc4f891a2`, — `commission_legacy=1`: стороны не раскладываются, досверщик не трогает. CB дневного лимита считает по валовому P&L.
  `S8R-AUDIT-031`: цена исполнения входа — `average_position_price` из `GetOrderState`, иначе `executed_order_price / (лоты × лот)`, иначе цена ответа `PostOrder`; ноль — не цена (маппер T-Invest отдаёт `Decimal 0` при пустом `MoneyValue`). Нет цены — сделка остаётся `pending`; дольше `UNKNOWN_OUTCOME_CONFIRM_AGE_SEC` — `filled` без цены: `position_mismatch_at` + нота, пауза сессии, критичное `position.mismatch`, SL/TP не ставятся, закрытие из терминала доступно, повторный ордер не отправляется. Реконсиляция: сделка без подтверждённого входа → `cancelled` без P&L и вне `trades_closed`; записанная комиссия брокера попадает в `realized_pnl`. Деньги (P&L, комиссия, paper-портфель, статистика, CB, корпоративные действия) округляются одной функцией `app/common/money.quantize_money` (копейки, `ROUND_HALF_UP`).
  `S8R-FIX-048`: закрытие и сверка `filled` без цены в sandbox/real (признак `is_priceless_broker_entry`): в единой точке учёта закрытия (`_apply_close_and_settle`, через неё идут и закрытие, и `finalize_exit_order`) — после исполнения выхода один `GetOrderState` ордера входа адаптером вызывающего (бюджет `ENTRY_PRICE_LOOKUP_TIMEOUT_SEC` = 5 с, отменяемый), правило S8R-AUDIT-031. Цена найдена — обычный P&L, сторона комиссии входа — факт или оценка от оборота (время стороны — исполнение входа). Не найдена — `closed`, `pnl`/`pnl_pct` NULL («P&L не определён»: closed + pnl NULL + entry NULL, API `pnl_undetermined`); `DailyStat`: `trades_closed +1`, `realized −commission`; CB учитывает её консервативной оценкой (S8R-FIX-063 ч.2, `app/circuit_breaker/unpriced.py`); без оценки — `cb.unpriced_trade` (critical, outbox транзакции закрытия, `stage_unpriced_trade_alert`; пути `_settle_close_locked`, сверка после переоткрытия, закрытие вне терминала S8R-FIX-052); статистика сессии — вне win rate и средней сделки, комиссия — в net и в кривой капитала; `trade.closed` — `pnl`/`net_pnl` null, Telegram «P&L не определён (брокер не сообщил цену входа)». Отмена во время дозапроса: запись закрытия и уведомление о частичном выходе — под `run_shielded_from_cancel` (grace `ORDER_CANCEL_GRACE_SEC`), исходная `CancelledError` пробрасывается всегда. Paper и `entry = 0` — `cancelled` (FIX-5).
  `S8R-FIX-063/065`: Цена исполнения ордера — `order_fill_price`: `average_position_price` → `executed_order_price/(лоты×лот)` → средневзвешенная `stages[].price`; ноль — не цена. Последний источник — `response.price`: цена или сумма/(лоты×лот), берётся кандидат в пределах ±20 % от оценки (рынок, иначе цена входа/сигнала, gotcha-33). Запись закрытия sandbox/real — один commit под `run_shielded_from_cancel` (`ORDER_CANCEL_GRACE_SEC`); свой адаптер — после записи. Выход без цены брокера — «P&L не определён» (`exit_price` NULL; `exit_signal_price` — только при рыночной оценке). Нереализованный P&L и объём позиции — по исполненным лотам (`position_unrealized_pnl`/`position_quantity`, S8R-FIX-062).
- `RiskMonitor` — мониторинг SL/TP по открытым позициям и их закрытие при срабатывании. В `paper` закрывает сам; в `sandbox`/`real` только принимает решение, а исполнение делегирует `OrderManager.close_position` — единственному пути отправки встречного ордера брокеру (`_close_via_broker`, 2026-08-05). Здесь же `derive_lot_size` — авторитетный множитель штук/лот, восстановленный из `volume_rub`. С A-03 (ТЗ v4.1): на закрытом баре — `check_sl_tp` (страховка) и `paper_bar_levels` (paper: на баре аукциона открытия не проверяется, на первом обычном баре дня при open за уровнем — выход по open), внутри бара — `check_sl_tp_intrabar` (только sandbox/real) и флаг процесса `_INTRABAR_EXIT_REQUESTED` (§5.4.6).
- `PaperTradingEngine` — эмуляция исполнения ордеров для paper trading; вся денежная мутация paper-портфеля идёт через `PaperPortfolioAccountant` (`apply_open` / `apply_close`).
  `S8R-AUDIT-076`: денежная мутация paper-портфеля (вход, закрытие, SL/TP, дивиденды/купоны/cash-in-lieu) — в секции `paper_portfolio_section` = `keyed_lock("paper_portfolio", session_id)` (после `close_trade`), взятой **до первой записи транзакции** (иначе asyncio-лок и SQLite-писатель дают дедлок, gotcha-78); баланс внутри перечитывается (`populate_existing`). Атомарный SQL не применяется: Numeric в SQLite хранится как REAL.

> **Нереализованный P&L нигде не хранится** — он считается на лету
> (`app/trading/unrealized.py`, `unrealized_pnl_by_session`) и одинаково у трёх
> потребителей: Circuit Breaker (`_check_daily_loss_limit`), карточка сессии
> (`TradingService.get_positions`) и дневной отчёт
> (`SchedulerService.send_daily_stats`). Величина устаревает на следующем баре,
> поэтому запись в горячий путь не добавлялась; колонка
> `daily_stats.unrealized_pnl`, которая никогда не заполнялась, удалена
> миграцией `e2f3a4b5c6d7` (S8R, 2026-08-03).

> **`PositionTracker` удалён** 2026-07-31 (`S8R-POSITIONTRACKER-DEAD-CODE`):
> класс не вызывался из `app/` ни разу с момента появления в S5, а его
> `calculate_unrealized_pnl` был четвёртой копией расчёта нереализованного P&L.

**5.4.2 Поток обработки сигнала (Real, после S5R):**

```
StreamManager (T-Invest gRPC) / MOEX publisher
    ↓ publish в EventBus канал market:{ticker}:{timeframe}
SessionRuntime._SessionListener  (подписан один на сессию)
    ↓ candle close detected, обновляет скользящее окно
    ↓
SignalProcessor.process_candle(session, candles=history + [candle])
    ↓ (стратегия в sandbox, расчёт индикаторов, генерация сигнала)
signal: BUY / SELL / HOLD
    ↓ (если BUY или SELL)
CircuitBreakerEngine.check_before_order(session, signal)
    ↓ (под per-user asyncio.Lock — см. Gotcha 5 в Develop/CLAUDE.md)
OrderManager.place_order(session, signal)
    ↓
BrokerAdapter.place_order(order)
    ↓
EventBus.publish(f"trades:{session.id}", {"type": "order.placed", "latency_ms": ...})
```

Канал `trades:{session_id}` публикует 6 типов событий: `session.started`, `session.stopped`, `order.placed` (с `latency_ms`), `trade.filled`, `trade.closed`, `cb.triggered`. Все Decimal-поля в payload'ах сериализуются строкой (Gotcha 1).

**Требование по latency:** Весь путь от получения свечи до `BrokerAdapter.place_order()` < 500ms. Для этого: стратегия предзагружена в RAM, индикаторы обновляются инкрементально, история свечей хранится в памяти runtime, listener не тянет её каждый раз заново.

**5.4.3 Восстановление при перезапуске:**

При старте `main.py lifespan`:
1. Создаётся единственный экземпляр `SessionRuntime(AsyncSessionLocal)`.
2. `SELECT * FROM trading_sessions WHERE status IN ('active', 'paused')`.
3. `await runtime.restore_all(sessions)`:
   - Для каждой сессии — если `timeframe` задан, создаётся listener на `market:{ticker}:{timeframe}`, предзагружается история. Legacy-сессии без `timeframe` пропускаются с warning.
   - **Paper-часть** реализована в S5R.2: подписка на EventBus, возобновление мониторинга.
   - **Real-часть** (сверка позиций с брокером для `mode=real` + подписка `StreamManager.subscribe`) — задача Sprint 6 **6.4 Recovery**.
4. При расхождении позиций с брокером — пауза + уведомление (6.4).
5. При остановке backend — `await runtime.shutdown()` отписывает все listener'ы и отменяет background-tasks (задел для 6.5 Graceful Shutdown).

**5.4.5 StreamManager (Market Data gRPC):**

- `app/market_data/stream_manager.py` поддерживает **одно** persistent gRPC `MarketDataStream` к T-Invest для обновлений свечей (с A-01 рядом живёт второе соединение — закрытые минуты, ниже и §5.6.6).
- Мультиплексирует подписки по ключу `(ticker, timeframe)`. **С A-01 (ТЗ v4.0) родная подписка T-Invest — только у `1m`; ключи остальных ТФ (`5m`, `15m`, `30m`, `1h`, `4h`, `D`, `W`, `M`) подписаны на `1m` и строят живую свечу общим `AggregatingCandle` (тот же агрегатор, что у истории, §5.6.6); карты ТФ → интервал T-Invest больше нет.** Ключи D/W/M открываются так же, как внутридневные (`needs_subscription` их не исключает): голова дня — минуты дня по REST, для W/M к ней добавляются закрытые D периода. В живую свечу идут только биржевые свечи; режим подписки — настройка `MARKET_DATA_STREAM_MODE` (`two_subscriptions` ~~по умолчанию~~ — `CANDLE_SOURCE_EXCHANGE`; `single_include_weekend` — одна подписка с `CANDLE_SOURCE_INCLUDE_WEEKEND` и раскладкой по `Candle.candle_source_type`, ~~дилерские свечи считаются и не маршрутизируются~~). **С A-03 (ТЗ v4.1, ADR 0021) по умолчанию — `single_include_weekend`** (настройка читается при создании мультиплексора): `exchange` — живая свеча `market:`, агрегаты, `bar.closed`, очередь алертов; `dealer` — свой `_KeyState` ключа `dealer:` (свой `AggregatingCandle`, поколения, дозагрузка головы дилерскими минутами), обновления — `candle.update` в `dealer:{ticker}:{tf}` (ТФ `DEALER_TIMEFRAMES` 1m–4h), закрытые минуты — в `ohlcv_dealer` (`ClosedBarTracker` → `BarLoader.store_stream_dealer_minutes`, без покрытия); `unknown` — warning и счётчик `unknown_source`, никуда. Ключи потока — `(ticker, timeframe, source)`; `subscribe`/`open_stream`/`ensure_stream`/`acquire`/`release` принимают `source="exchange"|"dealer"`; дилерский ключ едет на той же записи стрима и той же подписке 1m мультиплексора, что основной ключ того же ТФ. `two_subscriptions` — только откат: подписка `CANDLE_SOURCE_EXCHANGE`, дилерский канал пуст (warning). Закрытые минуты идут вторым соединением (`waiting_close=True`, свой экземпляр мультиплексора — §5.6.6); на токен приходится два соединения.
- **Один токен на процесс** (`S8R-FIX-011`, явное правило). Токен в ключ не
  входит: подписчики одного `(ticker, timeframe)` едут на токене того, кто
  открыл стрим первым. Для этого продукта это не ограничение, а факт —
  регистрация закрывается после первого пользователя (`S8R-AUDIT-033`), то
  есть процесс обслуживает одного владельца. Правило записано здесь, чтобы
  следующий, кто добавит второго пользователя или второй боевой токен, увидел:
  ключ стрима придётся расширять счётом, иначе чужие свечи поедут на чужом
  токене. Поведение после обрыва WS описано в §4.12: мультиплексор
  восстанавливает свои каналы сам, а REST-подписку на стрим повторяет клиент
  (`useWsAuthGeneration`).
- Auto-reconnect с exponential backoff при разрыве.
- Rate limiting через `TokenBucketRateLimiter` (см. Gotcha 4 в `Develop/CLAUDE.md`).

**5.4.4 Paper Trading Engine:**

- Отдельная реализация `PaperBrokerAdapter`, совместимая с интерфейсом `BaseBrokerAdapter`.
- ~~Исполнение ордеров по текущей рыночной цене + slippage (настроенный).~~ С v4.10 (A-06) — по договору исполнения: цену правила (open минуты, уровень, гэп, текущая цена) передаёт рантайм (`ExecutionPrice`), сдвиг — проскальзывание сессии (`session_fill_price`), §5.17.3.
- Расчёты T+1 по деньгам не моделируются (ТЗ v4.2, ФТ §7.4, дефект 14): деньги считает `PaperPortfolioAccountant` (Model A, BE-TRAD-06) — `balance` = свободные деньги, `blocked_amount` = капитал открытых позиций по цене входа, выручка продажи сразу возвращается в `balance`. Ветка блокировки T+1 в `PaperBrokerAdapter.place_order` (`trading/paper_engine.py`, пометка `DEAD PATH`) paper-режимом не вызывается; ~~удаляется в блоке A-06 вместе с `DEFAULT_SLIPPAGE_PCT` (B4)~~ удалена в v4.10 (A-06) вместе с `DEFAULT_SLIPPAGE_PCT`; метод `place_order` остался.
- Баланс и позиции хранятся в `paper_portfolios` и `live_trades`.

**5.4.6 Часы торгов, выходы по виду, стоп внутри бара, закрытие бара сессией, стопы у дилера (блок A-03, ТЗ v4.1).**

Как устроено — `docs/architecture.md` (разделы «Правила часов», «Список „закрыть при первой возможности“», «Сессия: закрытие бара, отложенный вход, стоп внутри бара», «Стопы у дилера»), почему — `docs/adr/0013…0016`, `0019`, `0020`, `0022`. Пути — от `backend/app/`. Требования — ФТ §1.6, §6.1.1, §6.3.

```
 market:{ticker}:1m ─▶ _IntrabarRiskWatcher ─▶ RiskMonitor.check_sl_tp_intrabar ─▶ OrderManager.close_position(exit_kind=STOP|TAKE)
 dealer:{ticker}:1m ─┘ (только при DEALER_STOPS_ENABLED; только стоп)            └▶ exit_verdict → DEFER / DEALER_CHECK → pending_exits.enqueue
 bar.closed / таймер bar_close_at+5 с / следующий бар ─▶ _SessionListener ─▶ стратегия ─▶ CB ─▶ process_signal
                                                                               └ входы закрыты → _DeferredEntry → первое обновление ≥ execute_at
                                                         (с v4.10: _PendingExecution; paper — всё по сигналу, минута ≥ not_before, §5.17.3)
 _periodic_recovery_loop (60 с) ─▶ pending_exits.retry_pending_exits ─▶ close_position(exit_kind=…)
```

**Правила часов — `common/trading_hours.py`** (ADR 0013). Фаза — только из таблицы периодов (`get_trading_schedule()`, §5.9.1); констант часов биржи в модуле нет. «Сейчас» правил — `trading_hours._now_msk` (единственная точка подмены в тестах).
- **Вход:** `entry_verdict(now=None) -> str | EntryGate | None` — `None` (открыто: торги и вечер будней), текст причины или `WEEKEND_FLAG_REQUIRED` (торги выходного дня — решает `weekend_flag` бумаги, неизвестен — закрыто). Тексты: `CLOSING_AUCTION_REASON` «аукцион закрытия — только выходы», `OPENING_AUCTION_REASON` «аукцион открытия — входы с начала торгов», `DEALER_HOURS_REASON` «часы дилера — входы не выставляются», `OUTSIDE_HOURS_REASON` «вне торговых часов MOEX», `NON_TRADING_DAY_REASON`, `NOT_WEEKEND_INSTRUMENT_REASON`. `entry_deferrable(now)` — аукцион открытия, часы дилера, «закрыто» (не аукцион закрытия). Обёртки прежних имён (их подменяют тесты): `market_closed_reason(now, *, weekend_flag)`, `is_market_open`, `is_session_open` (идёт ли биржевой период — фильтр connection-событий мультиплексора), `market_phase(now)`.
- **Выход:** `exit_verdict(kind: ExitKind, now=None, *, dealer_stops_enabled=None, weekend_flag=None) -> ExitVerdict(decision, reason)`; `ExitKind` — `MANUAL` | `STOP` | `TAKE` | `SIGNAL` | `FORCED`; `ExitDecision` — `ALLOW` | `DEFER` (ждать фазы, где вид разрешён) | `DEALER_CHECK` | `DENY` (отказ до отправки):

| Фаза | MANUAL | FORCED | STOP | TAKE | SIGNAL |
|---|---|---|---|---|---|
| торги, аукцион закрытия, вечер (в выходной — только бумага с `weekend_flag`) | ALLOW | ALLOW | ALLOW | ALLOW | ALLOW |
| аукцион открытия | ALLOW | DEFER | DEFER | DEFER | DEFER |
| часы дилера; торги выходного дня у бумаги без `weekend_flag` или с неизвестным (D03, ADR 0020) | ALLOW | ALLOW | `DEALER_CHECK` при `DEALER_STOPS_ENABLED`, иначе DEFER | DEFER | DEFER |
| закрыто (ночь, праздник, промежуток) | DENY («вне торговых часов MOEX: не торгуют ни биржа, ни дилер»; будничный праздник — «неторговый день MOEX (праздник)») | DEFER | DEFER | DEFER | DEFER |

  `exit_needs_weekend_flag(kind, now)` — нужен ли признак бумаги (только STOP/TAKE/SIGNAL в торги выходного дня): вызывающий спрашивает его (`pending_exits.exit_weekend_flag`; кэш процесса 12 ч, сбой — 60 с, при сбое — как без признака) только тогда. Ручное закрытие: `close_blocked_reason(now)` / `is_within_trading_hours(now)` — обёртки над `exit_verdict(MANUAL)`. `DEFAULT_TRADING_START/END` = 07:00 / 23:50 — только умолчание окна CB (§5.8); `SESSION_BREAK_START/END` удалены.
- **Вид выхода задаёт вызывающий:** ручное закрытие позиции и «Стоп» сессии (`TradingSessionManager`) — `MANUAL`; `RiskMonitor` — `STOP`/`TAKE`; выход стратегии (`process_signal`) — `SIGNAL`; аварийное закрытие CB и «Закрыть все» на экране (`TradingService.close_all_positions`) и в Telegram (`TelegramWebhookHandler._execute_closeall` → `OrderManager.close_all_positions`, теперь и сделки `pending`) — `FORCED` (D02, ADR 0019; `close_all_positions(…, exit_kind=ExitKind.FORCED)` по умолчанию). Невыполненные сделки «Закрыть все» — `CloseOutcome` «Выход отложен: …», в Telegram — «Закроется при первой возможности».
- **`OrderManager.close_position(trade_id, reason, *, confirm_closed_outside=False, exit_kind=ExitKind.MANUAL)`:** автоматический выход (не MANUAL) спрашивает `exit_verdict` в начале `_close_position_locked` (`_auto_exit_verdict`) — **до любого обращения к брокеру**; `DEFER` — `ExitDeferredError`, `DEALER_CHECK` — `dealer_exit_check`, не прошла — `DealerExitBlockedError` (оба — наследники `ValidationError`); у paper дилера нет — `DEALER_CHECK` читается как `DEFER`. Ручное закрытие проверяется в хвосте закрытия прежними именами — чинить оба места.

**Список «закрыть при первой возможности» — `trading/pending_exits.py`** (ADR 0014). Пометка — колонки `live_trades.exit_retry_kind/since/notified_at` (§3.9).
- `enqueue(db, trade_id, kind, reason)` — выход, который не прошёл (фаза, проверка дилера, отказ брокера); уведомление — существующее событие `order.error` в `trades:{session_id}` (нового `event_type` нет), при постановке и не чаще `NOTIFY_INTERVAL` = 15 мин на сделку. Ручное закрытие в список не идёт. `clear` / `clear_fields` — при успехе или закрытии сделки любым путём.
- `retry_pending_exits(session_factory, *, now=None)` — раз в минуту из `SessionRuntime._periodic_recovery_loop` (`PENDING_EXIT_RETRY_INTERVAL_SEC` = 60): сделки с пометкой; уже не открыта — снимается; `exit_verdict(kind)` ALLOW или DEALER_CHECK → тот же `close_position(…, exit_kind=kind)` с причиной `RETRY_CLOSE_REASONS[kind]` (`stop_loss`, `take_profit`, `exit_signal`, `cb_forced`); DEFER — ждёт; `OrderInFlightError` — ждёт (ордер разрешит recovery закрывающих ордеров). Каждая сделка — своей `AsyncSession`. Второго закрывающего ордера нет: повтор под локом сделки и с guard'ом «ордер в полёте» (`exit_broker_order_id`, `exit_order_placed_at`); повторов на самой отправке ордера нет (gotcha-56).
- CB `_force_close_open_positions` → `close_position(exit_kind=FORCED)`, отказ (кроме `OrderInFlightError`) → `enqueue(…, ExitKind.FORCED, …)`. С A-04 — каждая сделка в своей `AsyncSession` (§5.8); «Стоп всему» — `close_all_positions(exit_kind=FORCED)` (§5.4.7).
- Список переживает перезапуск; `restore_all` ничего особого не делает. На закрытом баре сделка в списке считается как `close_failed` — стратегия на этом баре не вызывается.

**Стоп и тейк внутри бара — `trading/runtime.py::_IntrabarRiskWatcher`** (ADR 0015; ~~только sandbox/real~~ с v4.10 — и paper, `mode="paper"`, ADR 0071, 0073, §5.17.3). Один на сессию, поднимается в `SessionRuntime.start`; 1m-ключ потока держит держатель `session:{id}` (тот же поток T-Invest), сторож потока (`_recover_stale_streams`, `_recover_watcher_streams`) пере-подписывает его, остановка сессии снимает.
- Читает `candle.update` из `market:{ticker}:1m` и события `trades:{session_id}`. Лонг — `low ≤ стоп` или `high ≥ тейк`, шорт — зеркально. Минута входа сделки и первая минута, в которой наблюдатель её увидел (в том числе первая после перезапуска), — по текущей цене (`close` обновления), дальше — по `high`/`low` минуты: high/low формирующегося бара ТФ сессии не используются (вход в 10:05 на баре 1h дал бы ложное срабатывание по low 10:00–10:05). Сделка, которой ещё нет в снимке (поздний fill, `opened_at` — время заявки), при касании другой сделки проверяется как увиденная в этой минуте — по текущей цене (`bbb632d`).
- Горячий путь: уровни открытых сделок — снимок в памяти (перечитка не чаще `INTRABAR_LEVELS_MAX_AGE_SEC` = 5 с и по событию сделки); БД и брокер — только при касании; повторные касания той же сделки гасит флаг процесса `_INTRABAR_EXIT_REQUESTED`. Попытка выхода — отдельной задачей: `SessionRuntime._check_sl_tp_intrabar` → `RiskMonitor.check_sl_tp_intrabar` → `close_position(exit_kind=STOP|TAKE)`; отказ и DEFER → список.
- Закрытый бар — страховка (`_run_risk_monitor`, `check_sl_tp`); сигналы стратегии — только на закрытом баре. ~~Paper внутри бара не проверяется (договор исполнения paper — A-06).~~ С v4.10 paper проверяется каждой минутой по договору (§5.17.3).

**Стопы у дилера — `trading/dealer_exit.py`** (ADR 0022). Признак `settings.DEALER_STOPS_ENABLED` (по умолчанию `false`; включает заказчик — `docs/env_vars.md`). Включён — наблюдатель дополнительно слушает `dealer:{ticker}:1m` (тот же 1m-ключ потока); дилерская минута трогает только стоп и только там, где бумагу торгует дилер; тейк по дилерской цене не срабатывает. Перед заявкой `dealer_exit_check(adapter, figi, instrument_uid)` токеном счёта сессии: `TInvestAdapter.get_trading_status` — статус `SECURITY_TRADING_STATUS_DEALER_NORMAL_TRADING`, флаги `api_trade_available_flag`, `market_order_available_flag`; затем `get_dealer_order_book` — разовая подписка потока `SubscribeOrderBook` с `ORDERBOOK_TYPE_DEALER` (унарный `GetOrderBook` в SDK `0.2.0-beta117` типа стакана не принимает), ожидание до `DEALER_BOOK_TIMEOUT_SEC` = 5 с; спред > `DEALER_SPREAD_LIMIT_BPS` = 20 б.п., нет стакана, статус или флаги не те — заявки нет, `DealerExitBlockedError`, сделка в списке, уведомление с ценой и спредом. Кэша и повторов самой проверки нет; статус и стакан — только перед заявкой, не чаще раза в минуту на сделку.

**Закрытие бара сессией и отложенный вход — `_SessionListener`** (ADR 0016; заменяет D3 A-01 и правило «закрыт приходом следующего бара» ADR 0005/0006, в том числе для D/W/M).
- Бар закрывается по первому из: `bar.closed` (`listen_bar_events`, канал `bars:{ticker}:{tf}`) с тем же `timestamp` и `revised == False` — OHLCV из события, `complete=False` — путь неполного бара; страховочный таймер `bar_close_at + BAR_CLOSE_SAFETY_DELAY` (5 с; синтетическое сообщение `session.bar_close_timer` в ту же очередь) — шина теряет события при переполнении очереди (256), подписка закрытых минут может упасть; приход бара с большим `timestamp`. Один бар не закрывается дважды (`_closed_bar_ts`), `revised` игнорируется. Последний бар дня и D — 23:50:02.
- `_DeferredEntry(signal, bar_ts, execute_at, reason)` — один на сессию, в памяти. `_entry_deferral`: `entry_deferrable(now)` → `execute_at = next_trading_start(now, weekend_flag=)` (дни торгов выходного дня — только при `weekend_flag is True`; горизонт 10 дней); paper — сигнал бара аукциона открытия → `trading_start` его дня. При полном лимите позиций (`count_open_trades ≥ max_concurrent_positions`) сигнал отбрасывается сразу, как днём (`max_positions_reached`, `bb8799e`), — иначе стоп на открытии освободил бы место устаревшему сигналу. Исполняется на первом обновлении потока с `ts >= execute_at`: снова CB → `process_signal`, paper — по цене этого обновления. Новый закрытый бар, «Стоп», пауза и возобновление отменяют; потеря при перезапуске безопасна. Сигнал в аукцион закрытия не откладывается (CB блокирует, сигнал теряется). **С v4.10 (A-06)** `_DeferredEntry` заменён `_PendingExecution` (ADR 0071): у sandbox/real — тот же отложенный вход с правилом `signal`; у paper все исполнения по сигналу — ожидающие и исполняются наблюдателем минут по open минуты (§5.17.3), «paper — по цене этого обновления» больше не действует.
- Порядок на закрытом баре прежний: страховочная SL/TP; не прошла или сделка в списке — стратегия не вызывается; исполненный внутри бара выход сигнал не подавляет.

**Тесты.** `tests/unit/test_common/test_trading_hours_phases.py`; `tests/test_trading/` — `test_exit_kinds.py`, `test_intrabar_exits.py`, `test_pending_exits.py`, `test_session_bar_close.py`, `test_dealer_stops.py`, `test_close_all_forced.py` (общие фазы — `exit_phases.py`); «сейчас» — autouse `_trading_hours_now` в `tests/test_trading/conftest.py` (среда 21.10.2026 12:00 МСК), свою фазу тест ставит подменой `trading_hours._now_msk`; подставные клиенты SDK дилера — `tests/unit/test_broker/dealer_fixtures.py`. Ордера блок не выставляет — исполнение проверено подставным брокером; на тестовом стенде 02–04.11 — стоп и тейк внутри бара на sandbox #3.

**Известные ограничения и долги v4.1.**
- Не проверено вживую: выход из списка «закрыть при первой возможности» (проверен на стенде A-04 10.11: paper, закрытие в 07:00:16 с уведомлением), ручное закрытие в часы дилера при выключенном признаке, путь `DEALER_CHECK` (признак выключен; поведение дилера на рыночную заявку терминала неизвестно).
- Выход, ждущий разрешённой фазы, уведомляет один раз, а не раз в 15 мин (`pending_exits.py`); уведомление проверки дилера при любом сбое пишет «нет ответа за 5 с» (`dealer_exit.py`); дилерский стакан может ждать квоту дольше 5 с под локом сделки (`adapter.py`); нет тестов на повтор из списка при ордере «в полёте» и на гонку выхода внутри бара со страховкой закрытого бара.
- ~~Сбой БД при аварийном закрытии CB останавливает остальные закрытия срабатывания (`circuit_breaker/engine.py`, нужен rollback)~~ — снято в A-04 (§5.8).
- Выход, который брокер отклоняет постоянно, уходит раз в минуту, пока позицию не закроют вручную. Касание без выхода повторяется не раньше перечитки снимка (5 с) или события сделки.
- ~~Paper и sandbox расходятся: paper сверяет уровни на закрытом баре (A-06).~~ Снято в v4.10 (A-06): paper сверяет уровни каждой минутой.
- История сессии при старте может содержать формирующийся бар, признак полноты в рантайме сессий не проверяется (замечено, не исправлено).
- Правила зависят от «сейчас»: тест закрытия без подмены `_now_msk` зависит от часа прогона (так `test_closeall_confirm_callback` падал ночью и в праздник 04.11 — закреплён `0952d9c`).

**5.4.7 «Стоп всему» (блок A-04, ТЗ v4.7; ФТ §12.4; ADR 0040, 0043, 0044).** Модуль `app/trading/stop_all.py`; как устроено — `docs/architecture.md`, ««Стоп всему» и интерфейс Circuit Breaker».

```
POST /trading/stop-all ─┐                           ┌ 1. keyed_lock("trading_stop", user_id): INSERT trading_stops + audit, commit
/stopall → confirm ─────┴▶ StopAllService.enable ──┤ 2. там же: active/suspended → paused, pause_reason='stop_all'; событие CB stop_all + связи, commit
                                                    │    → session.paused (trades:{sid}), cb.triggered (system:{uid})
                                                    ├ 3. без лока: OrderManager.close_all_positions(sid, exit_kind=FORCED), сессия за сессией, своя AsyncSession
                                                    └ 4. N/M → trading_stops и circuit_breaker_events.close_report, commit → cb.close_report, stop_all.changed
```

- **Включение** (`enable(user_id, source="web"|"telegram", request=)`) — отдельная задача под `asyncio.shield` (`_RUNNING`): доводится до конца и при обрыве HTTP-запроса или таймауте webhook. Включён — `(строка, None, already_active=True)` без действий. Шаг 2: сессии пользователя (владелец — через версию стратегии) `active`/`suspended` → `paused` условным `set_sessions_status_if(..., pause_reason="stop_all")`, уже стоящим на паузе — `set_pause_reason`; `stopped` не трогается; `expedite_partial_entries(sid)` — остаток частичного входа не докупается. Сбой шага 2 — ERROR `stop_all_pause_failed`, флаг остаётся, закрытие идёт.
- **Закрытие** — открытые сделки (`OPEN_POSITION_TRADE_STATUSES`: `filled`, `pending`) сессий пользователя в любом статусе, только `live_trades` (ручные бумаги не видны). Исключение сессии целиком — строки `not_closed` с `close_failure_reason(exc)` и `pending_exits.enqueue(…, FORCED, …)` её сделок. Перед закрытием — комментарий-точка для A-14 (снять условные заявки брокера прямо перед продажей; у ожидающих открытия — не снимать), кода нет. Отчёт — `StopAllReport` (N — `closed` + `closed_without_order`, M — все, `items` — `not_closed` с `queued`).
- **Снятие** (`disable`) — под `trading_stop`: `disabled_at`, audit `trading.stop_all.disabled`, commit, `stop_all.changed`; сессии и список не трогаются, строки в истории CB нет; не включён — `ConflictError` 409 `trading_not_stopped`. Маршрут — зависимость `require_browser_session` (`trading/router.py`). **Состояние** (`state`) — включённый стоп и живой `waiting_exit_count`.
- **Отсечка входа** — флаг перечитывается из БД: (1) `CircuitBreakerEngine.check_before_order` первой проверкой под per-user локом CB — `CheckResult(blocked, temporary, event_type="trading_stopped")`, без события, паузы и уведомления; (2) `OrderManager._open_entry` (`_entry_blocked_by_stop`, из `process_signal`) — после последнего сетевого вызова входа (лот, рубли счёта, переоткрытие песочницы), прямо перед INSERT `pending`; отложенный вход A-03 идёт тем же путём. Выходы флаг не проверяют.
- **Старт и «Возобновить»** — `_hold_trading_stop_lock`: `keyed_lock("trading_stop", owner_id)` и `assert_trading_not_stopped` → `trading_stopped_error` (409, `extra` — `enabled_at`, `enabled_source`). Старт: под ним `strategy_delete` → `session_start_account` → запись сессии и строки `cb_limits` одним commit; `runtime.start` и подписка уведомлений — после освобождения. «Возобновить»: под ним `session_lifecycle`; `trading_stop` отпускается сразу после commit `active` и `session.resumed`, до `runtime.start`.
- **Порядок локов** (`app/common/locks.py`): `trading_stop` → `session_lifecycle` → per-user CB → `close_trade` → `paper_portfolio` → `sandbox_reconcile_account`; ветвь старта — `trading_stop` → `strategy_delete` → `session_start_account`. `trading_stop` берут только `enable`, `disable`, `start_session`, `resume_session`; сети под ним нет; брать его из-под другого лока нельзя. Пауза, «Стоп» сессии, торговый цикл, CB, закрытия и restore его не берут.
- **Перезапуск** — `SessionRuntime.restore_all` → `_pause_sessions_of_stopped_owners`: у владельцев с включённым стопом `active`/`suspended` → `paused` с `stop_all` до подъёма слушателей (WARNING `session_restore_paused_trading_stopped`); сбой проверки — ERROR `session_restore_trading_stop_check_failed`, restore идёт дальше.
- **Повтор списка** (раз в минуту, `SessionRuntime`): до `retry_pending_exits` — `pending_exits.pending_exit_owners(db)` и идемпотентный `NotificationService.listen_session_events(sid, uid)` для всех пар (сессия в любом статусе, ADR 0044); после — `stop_all.changed` владельцам, у которых список сократился. `app/main.py` блоком не менялся; подписки не снимаются до перезапуска.
- **Telegram** (`notification/telegram_webhook.py`): `_handle_stopall` — стоп включён → текст состояния, иначе клавиатура `confirm_stopall` / `cancel_stopall`; `_execute_stopall` → `StopAllService(self._db_factory).enable(user_id, source="telegram")`, ответ — `format_stop_all_report`; сбой — по состоянию БД «включён, но отчёт не собран» или «не удалось включить». Команды снятия нет; `/help` и подтверждение `/closeall` дополнены.
- **Уведомления** — нового `event_type` нет, `EVENT_MAP` и `EVENT_TYPE_LABELS` не менялись: закрытия — прежние `positions.closed_all`, `trade.closed`, `order.error`.
- **Журнал:** `stop_all_enabled`, `stop_all_sessions_paused`, `stop_all_done closed= total= waiting= sessions_paused=`, `stop_all_disabled`, `cb_check_blocked_trading_stopped`, `entry_blocked_trading_stopped`; проблемы — `stop_all_pause_failed`, `stop_all_session_close_failed`, `stop_all_report_save_failed`, `stop_all_enqueue_failed`, `stop_all_enable_failed`, `stop_all_state_publish_failed`, `telegram_stopall_failed`, `pending_exit_notification_listen_failed`.
- **Тесты:** `tests/test_trading/` (`test_stop_all*.py`, `test_session_cb_limits_start.py`, `test_session_status_pause_reason.py`, `test_ws_sessions_pause_reason.py`, `test_pending_exits_in_flight.py`, `test_cb_force_close_isolation.py`), `tests/unit/test_notification/test_telegram_stopall.py`, `tests/test_circuit_breaker/test_a04_*.py`; параллельные записи — файловая SQLite + `NullPool` (gotcha-95). E2E — `e2e/a04-stop-all.spec.ts`, `e2e/a04-cb-history-limits.spec.ts` (на моках).
- **Известные ограничения:** сбой самого первого чтения позиций — флаг включён, отчёта нет, позиции не в списке; текст уведомления о постановке в список начинается с «Закрыть все» (`_close_trades`); `sessions_paused` = 0, когда все уже стояли на паузе; «Возобновить» держит `trading_stop`, пока ждёт `session_lifecycle` той же сессии; тест гонки «Возобновить против „Стоп всему“» не написан; сбой проверки флага в `restore_all` поднимает `suspended` в `active` (вход отсекают CB и `_open_entry`).

### 5.5 Broker Adapter

**Ответственный: BACK1**

**5.5.1 Абстрактный интерфейс:**

```python
class BaseBrokerAdapter(ABC):
    @abstractmethod
    async def connect(self) -> None: ...

    @abstractmethod
    async def disconnect(self) -> None: ...

    @abstractmethod
    async def get_accounts(self) -> list[BrokerAccount]: ...

    @abstractmethod
    async def get_balance(self, account_id: str) -> AccountBalance: ...

    @abstractmethod
    async def get_positions(self, account_id: str) -> list[Position]: ...

    @abstractmethod
    async def place_order(self, order: OrderRequest) -> OrderResponse: ...

    @abstractmethod
    async def cancel_order(self, order_id: str, account_id: str) -> None: ...

    @abstractmethod
    async def get_order_status(self, order_id: str, account_id: str) -> OrderStatus: ...

    @abstractmethod
    async def get_historical_candles(
        self, ticker: str, timeframe: str,
        from_dt: datetime, to_dt: datetime
    ) -> list[Candle]: ...

    @abstractmethod
    async def subscribe_candles(
        self, ticker: str, timeframe: str,
        callback: Callable[[Candle], Awaitable[None]]
    ) -> SubscriptionHandle: ...

    @abstractmethod
    async def subscribe_orderbook(
        self, ticker: str, depth: int,
        callback: Callable
    ) -> SubscriptionHandle: ...

    @abstractmethod
    async def subscribe_trades(
        self, ticker: str, callback: Callable
    ) -> SubscriptionHandle: ...

    @abstractmethod
    async def search_instruments(self, query: str) -> list[InstrumentInfo]: ...

    @abstractmethod
    async def get_instrument_info(self, ticker: str) -> InstrumentInfo: ...

    @abstractmethod
    async def get_operations(
        self, account_id: str, from_dt: datetime, to_dt: datetime
    ) -> list[Operation]: ...
```

**5.5.2 T-Invest реализация:**

- gRPC-клиент через `tinkoff.invest.AsyncClient`.
- **Окружение gRPC процесса (S8R-FIX-029/083/093).** `GRPC_DNS_RESOLVER` grpcio (1.84) читает **при импорте модуля** `grpc`, а не при первом канале, как предполагал S8R-FIX-083: тот выставлял переменную в lifespan строкой ниже `ensure_grpc_trusted_roots()`, а она сама импортирует grpc (`pkgutil.get_data("grpc._cython", …)`) — в логе `resolver=native`, в работе c-ares (10–11 с на разрешение имени при дедлайне unary 10 с). С S8R-FIX-093 значение по умолчанию `native` ставит `app/__init__.py` при импорте пакета (`app/grpc_env.apply_grpc_dns_resolver_default`, модуль перенесён из `app/broker/tinvest/grpc_env.py` в корень пакета, чтобы `app/__init__` не поднимал брокерский пакет); заданное оператором значение не трогается (`ares` — вернуть c-ares). lifespan только пишет лог: `grpc_dns_resolver_configured` / `_preset` / `_ineffective` (warning: grpc импортирован раньше пакета `app`, значение не подействовало). `GRPC_DEFAULT_SSL_ROOTS_FILE_PATH` (корни НУЦ, S8R-FIX-029) grpc читает лениво — она по-прежнему первым шагом lifespan. Обе переменные — окружение процесса, не `Settings` (`docs/env_vars.md`, гайд по деплою §6в).
- `StreamManager` — управление gRPC-стримами (candles, orderbook, trades). Одно gRPC-соединение, мультиплексирование подписок.
- Rate limiter (`S8R-AUDIT-048`, `rate_limiter.py`): token bucket на токен × сервис по справочнику лимитов (`tinvest_api_overview.md`): MarketData 600, Instruments 200, Operations 200, Orders 100, PostOrder 300, CancelOrder 100, Users 100 в мин; запросы подписки стрима — 100/мин; sandbox-токен — методы SandboxService в общие 200/мин, MarketData/Instruments — по своим квотам. Ключ реестра — отпечаток токена. Ожидание квоты: отправка ордеров — не дольше 10 с (`QuotaNotAcquiredError` = «не отправлено», выход повторяется следующим тиком), чтения — без потолка (ограничены внешними дедлайнами).
- Справочник инструмента по FIGI (`GetInstrumentBy`: тикер, название, тип, шаг цены) — один процессный кэш, TTL 24 ч, ≤ 4096 записей, один канал на пачку, дедуп одновременных промахов; ответ без тикера и ошибки не кэшируются.
- Маппер: пустой Timestamp брокера SDK отдаёт как 1970-01-01 — «нет времени» = None или эпоха: свеча пропускается (агрегированный WARNING), дата операции — None; неизвестное состояние операции — `UNKNOWN` (не `EXECUTED`).
- Автоматическое переподключение: exponential backoff 1s -> 2s -> 4s -> ... -> 60s, до 10 попыток.
- Состояние token bucket **не сохраняется** между рестартами (персистентность удалена — S8R-AUDIT-019).
- Песочница (`S8R-AUDIT-048`, `sandbox_recovery.py`): новый счёт открывается, только если у токена нет счёта со статусом не-`closed` (под `keyed_lock("sandbox_account_open")`); «живой» счёт — любой, кроме `closed`; под локом переоткрытия (`sandbox_recovery`, с S8R-FIX-115 — один на процесс; до него — `sandbox_recovery_user` по пользователю) перечитывается текущий `account_id` записи (соседнее переоткрытие → `UNCHANGED` с новым счётом); после любой смены счёта — сверка позиций по фактическому портфелю текущего счёта (идемпотентна; при ошибке чтения портфеля — пропуск до фоновой сверки); запись нового `account_id` — своей сессией, сессия вызывающего обязана быть зафиксирована.
- **Счёт песочницы — одной записи (S8R-FIX-115).** `account_id` песочницы глобален для токена, а токен подключают и разные пользователи; токен из шифртекста не сравнить (свой IV у записи). Поэтому: (а) переоткрытие (`_account_ids_taken_by_siblings`) считает занятыми `account_id` **всех** sandbox-записей, кроме своей (было — только записей того же пользователя); нет свободного живого — `UNAVAILABLE` с `SANDBOX_ALL_ACCOUNTS_TAKEN_MESSAGE`; (б) регистрация sandbox-токена (`BrokerService.create_account` → `_free_sandbox_accounts`) не заводит запись на счёт, закреплённый за записью другого пользователя (422 «…уже подключены другим пользователем терминала…»), и на закрытый у брокера счёт (422 «…закрыты брокером…»); свой `account_id` занятым не считается, даже если на него есть и унаследованная запись другого (иначе владелец не сменит ключ, S8R-AUDIT-062), своя запись на закрытый счёт ключ меняет по-прежнему. Отбор и вставка — одной секцией под `sandbox_recovery`, взятым **внутри** `create_account` (порядок `create_account` → `sandbox_recovery`, `app/common/locks.py`; обратного пути нет, под `sandbox_recovery` — только БД своей сессией запроса, без сети и других локов); тот же лок держит переоткрытие — регистрация и переоткрытие не закрепляют один живой счёт за двумя записями. Уникального индекса на `account_id` sandbox-записей в БД нет (пары разных пользователей, заведённые до запрета, законны) — защита только в процессе (один uvicorn-worker). Боевые счета у двух пользователей — как прежде (`UNIQUE (user_id, broker_type, account_id)`).
- Сверка после смены sandbox-счёта (`S8R-REV-LQ`, `app/trading/sandbox_reopen.py`): одна функция «переоткрыть + сверить» и одно правило запуска (REOPENED или счёт уже переоткрыт соседом) для движка, кнопки «Пересоздать sandbox-счёт» и восстановления при запросе баланса. Фантом — только сделка `filled`, вход которой исполнен до момента переоткрытия (`reopened_at`, граница — целая секунда), без выхода в полёте, FIGI нет в портфеле нового счёта; портфель — один раз за проход под локами `close_trade` (по возрастанию) → `sandbox_reconcile_account`, условия перепроверяются под локом сделки; при неточном моменте (сосед без записи) портфель «только деньги» считается непрочитанным. Сверка — фоновая задача (одна на счёт, своя сессия), пропущенные сделки — отложенный повтор (~210 с), портфель не прочитан во всех попытках — пометка расхождения, пауза, critical; занятые локи — повтор. `pending` старого счёта разрешает recovery ордера (NOT_FOUND → `failed` после окна подтверждения).
- Маппер `TInvestMapper` — конвертирует Protobuf-объекты в доменные модели.
- Мультиплексор стрима (`S8R-AUDIT-064`): ошибка маппинга одной свечи — свеча пропускается (ERROR на первую ошибку нового типа, далее агрегированный WARNING), без переподключения; ценовые алерты проверяются отдельной задачей из ограниченной очереди (figi, время свечи) → [min, max], при остановке очередь дочитывается ≤ 1 с. Монитор алертов (`S8R-FIX-051`) регистрируется в реестре мультиплексоров из lifespan (`set_default_price_alert_monitor`, читается в момент проверки); сам держит подписку 1m на FIGI с активными алертами через счёт владельца (`select_market_data_account`, только активные пользователи и счета); пересчёт — старт, создание/удаление/срабатывание алерта, создание/замена ключа/деактивация/удаление счёта (до `release_token`); FIGI — при создании алерта, NULL дозаполняется. Срабатывание — условный UPDATE + in-app уведомление + outbox `notification.dispatch` в одной транзакции; доставка отмечается по каналам, повтор при старте — фоном, только недоставленные. In-memory набор FIGI с алертами — остальные свечи не проверяются; лок пересчёта — в `locks.py`. **С A-03 (ТЗ v4.1):** в очередь алертов идут только биржевые свечи — `AlertMinute` (OHLC минуты, проверка по high/low, §5.6.7); раскладка свечи по `candle_source_type` в режиме `single_include_weekend` (§5.4.5): `exchange` — прежним путём и в очередь алертов, `dealer` — в дилерский ключ, `unknown` — warning и счётчик, никуда.
- **Чтения T-Invest A-03 (ТЗ v4.1)** — `TInvestAdapter`, только чтения (повторы — `_read_with_retry`, на отправке ордера повторов нет): `get_trading_schedules(exchange, from_, to)` — `TradingSchedules` (имя площадки сравнивается `casefold`, `bb8799e`), §5.9.1; `get_last_price(figi)` — `GetLastPrices` (цена создания алерта); шаг цены — из `GetInstrumentBy` тем же проходом, что лот (`_InstrumentRef.min_price_increment` → `instruments.min_price_increment`), и `get_price_step(figi)` — для окна алерта, если в справочнике шага нет; `get_trading_status` — `GetTradingStatus` (статус и флаги API для проверки дилера); `get_dealer_order_book(…, timeout=DEALER_BOOK_TIMEOUT_SEC)` — разовая подписка потока `SubscribeOrderBook` с `ORDERBOOK_TYPE_DEALER` (унарный `GetOrderBook` в SDK `0.2.0-beta117` тип стакана не принимает). Статус и стакан — только перед дилерской заявкой (§5.4.6). Известный долг: стакан может ждать квоту дольше 5 с под локом сделки.
- **Sandbox vs Production (per-account):** Поле `BrokerAccount.is_sandbox` определяет, к какому эндпоинту подключается адаптер (`sandbox-invest-public-api.tbank.ru` или `invest-public-api.tbank.ru`). Тип токена определяется автоматически при подключении через пробный вызов `get_accounts()` / `get_sandbox_accounts()`. Sandbox-аккаунты доступны только при `DEBUG=true`.

**5.5.3 MOEX ISS fallback:**

- HTTP REST-клиент через `httpx.AsyncClient`.
- Используется только для исторических данных (OHLCV), торгового календаря и информации об инструментах.
- Не поддерживает торговые операции.
- Endpoint: `https://iss.moex.com/iss/`
- Ошибки и повторы (`S8R-AUDIT-064`, `MOEXISSClient._request`): 5xx, сбой связи, 408/429 и ответ 200 не-JSON (заглушка техработ) — до 3 попыток внутри клиента (Retry-After: 0 — сразу, > 10 с — без ожидания, бюджет пауз 20 с на выборку), затем `TransientBrokerError` («связи нет» ≠ «данных нет»); 4xx — `BrokerError`; вызывающие сами не повторяют. Пагинация `get_paginated` — по `<блок>.cursor` (есть у `/history`), иначе до короткой страницы; предел 1000 страниц — `BrokerError` «выборка усечена». Купоны облигаций — один метод `get_coupons(ticker)`. Объём календарного дня в основном режиме — `get_daily_volumes(ticker, from_dt, to_dt) -> {дата МСК: штуки}` (A-01, дефекты 5–7): итог `history/engines/stock/markets/shares/boards/TQBR/securities/{ticker}` (колонки `TRADEDATE,VOLUME`) и сессия выходного дня `history/…/shares/sessions/5/boards/TQBR/securities/{ticker}` (`TRADEDATE,VOLUME,TRADE_SESSION_DATE`, с запасом назад `WEEKEND_LOOKBACK` — 16 сут), страницы по 100. Итог торгового дня ISS включает торги выходного и праздничного дня, отнесённые к нему (`TRADE_SESSION_DATE`): `calendar_day_volumes` вычитает их из итога и отдаёт объёмом своих календарных дней; к своему дню сессия прибавляется (у дня может быть и свой итог), строка с той же датой торгового дня (SBER 16.06.2025, объём 0) итог не меняет; отрицательный остаток — дня нет. Эталон дня для `ArchiveLots.admit` и сверки объёма (§3.5.3, §3.5.5). Дня нет или `VOLUME` 0 — бумага в TQBR в этот день не торговалась (праздник без сессии, приостановка).
- Последствия сбоя ISS: график без T-Invest — данные из кэша при наличии (ошибка фонового обновления — пометкой), 503 «MOEX ISS недоступен» только при пустом кэше; облигация — устаревший кэш, сбой купонов кэш не затирает; детект корпоративных действий прерывается после первого сбоя.

**5.5.4 Фабрика:**

```python
class BrokerFactory:
    _adapters = {"tinvest": TInvestAdapter}

    @classmethod
    def create(cls, broker_type: str, credentials: dict) -> BaseBrokerAdapter:
        adapter_class = cls._adapters.get(broker_type)
        if not adapter_class:
            raise UnsupportedBrokerError(broker_type)
        return adapter_class(**credentials)
```

**5.5.5 Классы ошибок и повторы (S8R-AUDIT-044):**

`app/broker/tinvest/errors.py::classify(exc)` разбирает исключение один раз (цепочка `__cause__`, код gRPC из SDK-`AioRequestError` или сырого `AioRpcError`, 5-значный код T-Invest, `ratelimit_reset`) в категорию:

| Категория | Коды / исключения | Чтения | Отправка/отмена ордера | Класс ошибки |
|---|---|---|---|---|
| TRANSIENT | UNAVAILABLE, DEADLINE_EXCEEDED, `BrokerTimeoutError`, INTERNAL 70001, RESOURCE_EXHAUSTED с `ratelimit_reset` | повтор (`_read_with_retry`) | без повтора, «ответ не получен — проверьте портфель» | `TransientBrokerError` |
| AUTH | UNAUTHENTICATED, PERMISSION_DENIED, локальный INVALID_METADATA ключа | без повтора | без повтора | `AuthBrokerError` |
| NOT_FOUND | NOT_FOUND | без повтора | — | `NotFoundBrokerError` |
| LINK | UNKNOWN, CANCELLED, ABORTED, INTERNAL ≠ 70001, сетевые `OSError` | без повтора | «ответ не получен — проверьте портфель» | `TransientBrokerError` |
| LOCAL | прочие исключения не из SDK (ошибка нашего кода) | без повтора | «внутренняя ошибка при отправке ордера» | `BrokerError` |

Повторы: UNAVAILABLE/DEADLINE — экспонента с джиттером, 70001 — фиксированно 2 × 4 с (песочница оживает ≥ 5 с, gotcha-56), RESOURCE_EXHAUSTED — пауза `ratelimit_reset + 1` с, если укладывается в потолок, иначе без повтора. Паузы — вне `asyncio.timeout` вызова; токен лимитера берётся на каждую попытку. Худший случай одного чтения — `read_worst_case_sec()` (3 × дедлайн + паузы = 38 с), им пользуется оценка попытки сверки рантайма. Классификация исхода ордера движком (`is_definitive_rejection`) не меняется.

**Ожидание лимита в бэктесте (2026-10-14).** `app/broker/tinvest/rate_limit_wait.py::RateLimitWait` — только для чтения свечей бэктеста и грида (параметр `rate_limit_wait` у `TInvestAdapter.get_historical_candles`; графики, торговля и стримы — без него): при `RESOURCE_EXHAUSTED` тот же чанк повторяется после паузы `ratelimit_reset + 1` с (без заголовка — 30 с, минутное окно квоты), не более 3 ожиданий и 180 с суммарно; `asyncio.timeout` выборки (`_load_data`) сдвигается `reschedule` ровно на фактическую паузу; загруженные чанки сохраняются в кэш. Ожидание — событие `waiting` (канал job и `backtest:{id}`, `stage: "waiting"`; ретранслятор `ws_backtest.py` его пропускает). Потолок исчерпан — `RateLimitExhaustedError` с текстом «Лимит запросов T-Invest исчерпан — повторите через N с». Не покрыто: резолв FIGI справочником при исчерпанном лимите не ждёт.

**Снятие баннера расхождения.** `SessionRuntime._clear_position_mismatch` той же транзакцией помечает прочитанными непрочитанные `recovery_mismatch` сессии (`mark_session_alerts_read`) и после commit шлёт кадр `notification.read` (канал уведомлений пользователя; event_type уведомления не вводится). Watchdog стримов: WARNING `stream_watchdog_figi_unresolved` — один раз на сессию и на каждый сетевой ответ брокера, повтор из кэша промахов (`FigiLookup.from_miss_cache`) — DEBUG.

**Единая таблица кодов gRPC (S8R-FIX-030).** `app/broker/grpc_codes.py` (модуль без зависимостей) — категория для каждого из 17 `StatusCode` (TRANSIENT, RATE_LIMITED, AUTH, NOT_FOUND, ORDER_REJECTED, REQUEST_REJECTED, LOST, UNCLASSIFIED) и наборы категорий потребителей: вид ошибки адаптера (`classify`), «заведомо отвергнут» и бизнес-отказ по заявке движка (`sandbox_recovery.is_definitive_rejection`, `is_broker_order_rejection`), терминальный стрим мультиплексора. Своих наборов кодов у потребителей нет; решения по всем кодам закреплены параметризованным тестом.

### 5.6 Market Data Service

**Ответственный: BACK1**

**Канонический тикер (`S8R-FIX-056`, `app/market_data/ticker_resolver.py`).** `canonical_ticker(db, value, user_id)` — кэш процесса (24 ч) → справочник `instruments` без учёта регистра (индекс `ix_instruments_ticker_upper`, доверие — при известном виде инструмента) → T-Invest → ISS SECID → фолбэк upper (не кэшируется, негативный кэш 60 с по ключу (тикер, есть ли T-Invest), свежие отметки обоих видов — ответ без чтения счёта, S8R-REV-FBE); single-flight по (тикер, есть ли T-Invest) в отдельной задаче под shield. Вызывается в точках входа API (свечи, sparkline, subscribe, инструмент, лого, облигации, бэктест и его rerun — период проверяется до канонизации, сессии до локов, алерты, избранное; detect — без сети). Тип `Ticker` — только формат. Сравнения тикеров в БД и коде — без учёта регистра (`ticker_eq`/`ticker_in`/`same_ticker`), `ohlcv_cache` — `IN (канон, upper)` с дедупликацией по бару (приоритет канона); канал стрима сессии — по канону. ISS: свечи фьючерсов — `engines/futures/markets/forts`; лот фьючерса из ISS не берётся (только T-Invest). Создание рисунка графика — `known_canonical` (кэш/справочник, без сети; не известен — тикер пути как есть, S8R-REV-FBE).

**Чистка ISS-строк кэша свечей (S8R-FIX-064).** При доступном T-Invest ISS-строки пары перед загрузкой проверяются `SELECT EXISTS` (без write-лока); если они есть — удаляются в отдельной короткой сессии на движке `self.db.bind` с commit (`_purge_iss_cache`). Сессия вызывающего (бэктест, торговля) пишущую транзакцию SQLite не получает и write-лок на время сетевой догрузки свечей не держит — раньше DELETE даже на 0 строк держал лок до commit результата, и остальные писатели падали «database is locked» после busy_timeout 30 с (S8R-FIX-045 — следствие). С A-01 (ТЗ v4.0) чистка удаляет и покрытие `moex_iss` тех же рядов в той же транзакции (`delete_bars_with_coverage`), проверка `EXISTS` смотрит и свечи, и покрытие; чистятся только хранимые ТФ (`1m`, `D`).

**Время и источники (S8R-AUDIT-063).** Время MOEX ISS — московское, приводится к UTC в парсере (`parse_iss_datetime`, `app/common/timezones.MSK` = фиксированный UTC+3 — одна зона на проект). При подключённом T-Invest строки ISS в `ohlcv_cache` не читаются (чтение хранимого ряда `_read_stored` и `last_cached_bar` — CB, unrealized, позиции, цена выхода; до A-01 — `_get_cached`, §5.6.6). Лот для sandbox/real — только T-Invest (строгий и ленивый путь одинаково, кэш лота песочницы по `(счёт, тикер)`), P&L — множитель сделки `LiveTrade.lot_size`; ISS-лот — только paper/viewer, в `instruments` не пишется. Миграция `6f3a063d1c2e` вычищает сдвинутые ISS-свечи.

**FIGI инструмента (S8R-AUDIT-045).** Единый источник FIGI — `MarketDataService.ensure_figi` (токен счёта сессии, промах кэшируется 15 мин по `(счёт, тикер)`). Ордер, история и стрим передают адаптеру `figi=` явно; адаптер ищет FIGI сам (перебор TQBR → TQCB → TQTF) только если FIGI не передан. Движок определяет FIGI **до** отправки ордера (сделка → запасной резолв с кэшем 15 мин по `(счёт, тикер)`) и пишет его в `live_trades.figi` — ключ сверки; не определён — ордер не уходит. `paper_<TICKER>` брокеру не передаётся (`broker_figi`). Выбор инструмента из ответа `find_instrument` (`select_tradable_instrument`) — один для FIGI, лотности и ордеров: точное совпадение тикера; среди торгуемых через API (`api_trade_available_flag`, иначе среди всех) — режим по приоритету TQBR → TQTF → TQCB → TQOB → TQIR → TQPI → TQIF; прочие класс-коды — только при отсутствии основных; при равенстве — наименьший `(class_code, figi)`. Порядок ответа брокера на выбор не влияет.

**Функции:**
1. `get_candles(ticker, timeframe, from_dt, to_dt, user_id=None, mode="viewer"|"backtest"|"trading", rate_limit_wait=None, adjust="splits"|"none")` — выравнивает диапазон по барам ТФ, по `ohlcv_coverage` находит непокрытое («запрошенное минус покрытие»), дозагружает его через `BarLoader` (T-Invest) или ISS, читает хранимый ряд (по умолчанию с учётом дроблений, §3.5) и собирает ТФ агрегатором; возвращает `(list[CandleData], source)`, `source` ∈ `tinvest` | `moex_iss` | `cache` (до A-01 — проверка кеша и `DataFrame`; подробности — §5.6.6).
2. `subscribe_realtime(ticker, timeframe, callback)` — создаёт подписку через `BrokerAdapter.subscribe_candles()`.
3. `validate_data(df, ticker, timeframe)` — вызов `OHLCVValidator`.
4. Какие участки нужно загрузить, определяет `coverage.missing_ranges(db, ticker, timeframe, source, start, end)` (с A-01; прежние `check_cache_coverage`, `_find_gaps`, `_tail_tolerance`, `_get_cached` удалены).

**Обрыв стрима и дозагрузка (S8R-AUDIT-012).** `stream_manager` открывает «раунд обрыва» `[последняя живая свеча до обрыва, первая после)` при (пере)создании стрима или смене `adapter.connection_epoch` (публичное свойство поверх `multiplexer.connection_epoch`). Живые свечи публикуются сразу с `bar_pending=True`; после `BACKFILL_DEBOUNCE_SEC` (1,5 с) фоновая задача одним REST-запросом минуток на тикер (общим для ключей тикера всех ТФ — с A-01 включая `30m` и голову дня D/W/M, §5.6.6; через адаптер стрима, `_unary`) дозагружает хвост бара, пережившего обрыв, и голову текущего периода; сливаются только минуты старше первой живой; агрегатор хранит 2 закрытых периода. Затронутые бары переопубликуются с тем же timestamp, исправленными OHLC, `bar_pending=False` и `bar_complete` (False — если в обрыве были сделки после конца периода или REST не удался). Поля `stream_gen`, `bar_complete`, `bar_pending` в `candle.update`. Listener (`runtime.py`) торгует бар только после подтверждения полноты; неполный — RiskMonitor по его high/low и по свежему бару, сигнал не исполняется; SL/TP на свежем баре подавляет сигнал на его закрытии. Задачи дозагрузки отменяются при снятии стрима и в `unsubscribe_all`. **Периоды, целиком накрытые обрывом (S8R-FIX-022, 2026-10-13):** переопубликовывались только бары из `pending_bars` и `gap_period`, а период, не попавший ни туда, ни туда, на график не приходил вовсе до перезагрузки страницы (агрегатор держит два последних). Теперь такие периоды собираются из дозагруженных минут и публикуются с `bar_complete=True`; предел `GAP_REPUBLISH_MAX_BARS = 60` на раунд, дальше — одна строка в лог, а график перечитает период обычным REST-запросом (длинный обрыв не заливает сокет). Торговлю это не включает: закрывающая свеча приходит из нового поколения стрима, и `_closed_bar_verdict` такие бары пропускает (`gen_changed` → SKIP).

**Выбор ключа для рыночных/справочных данных (S8R-AUDIT-003).** `select_market_data_account(db, user_id, preferred_account_id, sandbox, account_id)` — единственная точка: только счета пользователя, `encrypted_api_key` и `encryption_iv` NOT NULL, порядок prod → счёт сессии → id; без `user_id`/`account_id` — `None` (дальше ISS/None, «любой токен» не используется). `decrypt_market_data_token` — расшифровка. Лот: prod → ISS → устаревший authoritative-кэш → sandbox; sandbox-лот — authoritative только для `mode=sandbox`, в общий кэш не пишется; strict real/paper песочницу не спрашивает. FIGI для сверки — `select_market_data_account(account_id=счёт сессии)`. Владелец сессии — `strategy_version_owner_id`. Ограничение многопользовательского режима: общий стрим `(ticker, tf)` открывается токеном первого открывшего (сессии или графика).

**Курс USD/RUB ЦБ РФ (`app/market_data/cbr_rate.py`, `GET /market-data/usd-rate`; S8R-FIX-118).** Запрос `XML_daily.asp?date_req=<сегодня МСК, ДД/ММ/ГГГГ>`: без даты ЦБ днём отдаёт уже установленный курс следующего дня. Одна дата МСК (`today = datetime.now(MSK).date()`, снятая до запроса) идёт и в `date_req`, и в кэш (`_fetched_msk_date`), и в запасной `as_of` ответа без атрибута `Date` (раньше — дата UTC, с 00:00 до 03:00 МСК вчерашняя). Курс с `as_of > today` отвергается (`cbr_rate_future_date` → `CbrRateUnavailable`: есть кэш — `stale=true`, нет — 503). `_is_fresh` ложно при смене даты МСК: кэш не переживает полночь МСК даже в пределах TTL 6 ч.

**Алгоритм загрузки с учётом rate limits:**

Для T-Invest: данные загружаются порциями не более 1 года за запрос (для дневного ТФ). Для минутного ТФ — порции по 1 день. Между запросами — пауза, рассчитанная по token bucket.

**Алгоритм выбора источника рыночных данных:**

```
Приоритет:
1. Production-аккаунт T-Invest (is_sandbox=false, is_active=true)
   → MarketDataService, 600 req/min, точные данные
2. MOEX ISS (бесплатный публичный API, без токена)
   → Fallback, менее точные данные

Sandbox-аккаунт НИКОГДА не используется для получения рыночных данных:
- MarketDataService недоступен через sandbox-эндпоинт T-Invest
- Sandbox-эндпоинт содержит только SandboxService (эмуляция торговли)

При конфликте в ohlcv_cache:
- Данные T-Invest перезаписывают данные MOEX ISS (более точные)
- Данные MOEX ISS не перезаписывают данные T-Invest (ON CONFLICT DO NOTHING)
```

**Sandbox T-Invest — режим разработки:**

Sandbox-аккаунты доступны только при `DEBUG=true`. Используются для тестирования интеграции с API T-Invest (отправка ордеров в sandbox-среду). Обычному пользователю в production sandbox-токен добавить нельзя — система автоматически определяет тип токена и отклоняет sandbox в non-DEBUG режиме.

Подробная документация по T-Invest API: `tinvest_api_overview.md`, `tinvest_api_services.md`, `tinvest_api_sandbox.md`.

**Реал-тайм данные:**

`RealtimeDataManager` ведёт словарь активных подписок `{(ticker, timeframe): [callbacks]}`. При первой подписке на пару — создаёт подписку у брокера. При последней отписке — отменяет. Свечи агрегируются из tick-данных, если брокер не предоставляет свечной стрим нужного ТФ.

**5.6.5 Bond Service (расчёт НКД для облигаций):**

```python
class BondService:
    async def get_bond_info(self, ticker: str) -> BondInfo:
        """Получает параметры выпуска: номинал, ставка купона, даты купонов, периодичность.
        Источник: MOEX ISS API /iss/securities/{ticker}.json
        Кеширование: таблица bond_info_cache, обновление раз в сутки."""
        ...

    def calculate_nkd(self, bond: BondInfo, date: date) -> Decimal:
        """НКД = (Номинал × Ставка купона / 100) × (Дней с последнего купона / Дней в купонном периоде)
        Используется при расчёте P&L в бэктесте и боевой торговле облигациями."""
        ...

    async def get_nkd_at_date(self, ticker: str, date: date) -> Decimal:
        """Обёртка: получить BondInfo + рассчитать НКД на указанную дату."""
        ...
```

**5.6.6 Хранилище баров и поток закрытых баров (блок A-01, ТЗ v4.0).**

Как устроено — `docs/architecture.md`, почему — `docs/adr/0001…0009`, термины — `CONTEXT.md` (репозиторий кода, ветка `feat/008-a01-bar-storage`). Пути — от `backend/app/`. Требования — ФТ §11.4.

```
 T-Invest GetCandles ──┐                          ┌─ ohlcv_cache     1m, D   source = tinvest | moex_iss
 T-Invest history-data ─┼─▶ BarLoader ────────────┤─ ohlcv_dealer    1m дилерские (source = tinvest_dealer)
   (только фон, 1m)     │   bar_loader.py         └─ ohlcv_coverage  что уже загружено, по трём источникам
 MOEX ISS (viewer) ─────┴─▶ service._save_to_cache ─▶ те же таблицы (source = moex_iss)
   missing_ranges(покрытие): «запрошенное минус покрытие» — единственное, что идёт к источнику
 хранимый ряд («как было») ─▶ splits.adjust_for_splits ─▶ aggregation.aggregate ─▶ AggregateCache (LRU) ─▶ GET /candles · бэктест · сессии · график
 (1m → 5m 15m 30m 1h 4h;  D → W M;  adjust="none" — без пересчёта)
 T-Invest стрим ─▶ TInvestStreamMultiplexer ─▶ stream_manager ─▶ candle.update   канал market:{ticker}:{tf}
 T-Invest закрытые минуты ─▶ ClosedBarTracker (closed_bars.py) ─▶ bar.closed     канал bars:{ticker}:{tf}
 вход пользователя ─▶ auth/router.py ─▶ backfill.run_backfill ─▶ BarLoader(archive_lots=ArchiveLots, use_archive=True) ─▶ reconcile_volumes  ~~(до v4.5)~~ — с A-02: `SchedulerService.history_backfill` ─▶ `backfill.run_history_backfill` (§5.6.8)
 corporate_actions (split) ─▶ splits.load_splits: to_as_was перед записью · adjust_for_splits при чтении
```

| Модуль | Владеет | Чего не делает |
|---|---|---|
| `market_data/aggregation.py` | набор ТФ (`TIMEFRAMES`, в том числе `30m`), `stored_timeframe`, `period_start`/`next_period_start`, `aggregate`, `AggregatingCandle` | БД, сеть, часы (`now` передаёт вызывающий) |
| `market_data/bar_close.py` | правило «бар закрыт»: `bar_close_at`, `is_bar_closed`, `forming_bar_start`, `trading_end_utc`; `BAR_CLOSE_GRACE` = 2 с | БД, сеть; расписание — ~~константы (`DEFAULT_TRADING_END`, `MOEXCalendarService.WEEKEND_CLOSE`)~~ с A-03 — таблица периодов `get_trading_schedule()` (§5.9.1) |
| `market_data/coverage.py` | диапазоны `[from, to)` и `ohlcv_coverage`: `missing_ranges`, `add_coverage`, `delete_bars_with_coverage` | не знает источников свечей |
| `market_data/bar_loader.py` | `BarLoader.load` — единственный писатель свечей T-Invest и их покрытия; `TINVEST_MINUTE_SEAM` = 2018-06-13 (с A-02 — импорт из `seam.py`); с A-02 — без записи в окно ISS, фильтр фантомных дневок, отметка `tinvest_invalid`; с A-03 — `store_stream_dealer_minutes` (закрытые дилерские минуты потока в `ohlcv_dealer`, без покрытия) | не выбирает счёт и не резолвит FIGI: получает `BrokerContext` |
| `broker/tinvest/history_archive.py` | `fetch_history_year` — годовой архив минут (`history-data`) | не пишет в БД |
| `market_data/service.py` | `MarketDataService`: `get_candles`, `get_dealer_candles`, `broker_context`, `AggregateCache`, чтение цены из кэша; с A-03 — шаг цены (`ensure_lot_size` пишет `instruments.min_price_increment`, читатель — `instrument_price_step`) | свечи T-Invest не грузит в обход `BarLoader` |
| `market_data/backfill.py`, `volume_check.py` | ~~фоновая догрузка при входе~~ с A-02 — фоновая догрузка по расписанию (§5.6.8); сверка объёма дня с ISS | |
| `market_data/splits.py` | дробления из `corporate_actions` (`load_splits`, граница `split_opened_before`); `to_as_was` (перед записью), `adjust_for_splits` / `adjust_price` (при чтении), `active_splits` (ключ LRU), `forget_bars_before_new_splits` | свечи не пишет; удаляет только строки до границы позднего дробления (`delete_bars_with_coverage`) |
| `market_data/lot_history.py` | `ArchiveLots.admit` — какие дни архива идут в основной ряд (точное совпадение с итогом дня ISS `history` TQBR), окно биржевых минут и лот на дату; `EXCHANGE_WINDOWS`, `ALLOWED_LOTS`; `ohlcv_lot_history` | свечи не грузит и не пишет |
| `market_data/stream_manager.py` | ключи потока `(ticker, tf)` (с A-03 — `(ticker, tf, source)`), живая свеча, `candle.update` в `market:` и (A-03) `dealer:`; подключает трекер | в БД не пишет |
| `market_data/closed_bars.py` | `ClosedBarTracker`: таймеры закрытия, сборка бара, `bar.closed`; с A-03 сдаёт дилерские закрытые минуты на запись | |
| `broker/tinvest/multiplexer.py`, `mapper.py`, `adapter.py` | подписка, перевод свечи в `CandleData` (с `source`, `is_complete`), `get_historical_candles` | |

**Хранилище.** `ohlcv_cache` — только `1m` и `D`, строки «как было» (цена и штуки своего дня, §3.5); `ohlcv_coverage`, `ohlcv_dealer`, `ohlcv_volume_check`, `ohlcv_backfill_target`, `ohlcv_lot_history` — §3.5.1–3.5.5. Тикер в кэше и покрытии — канонический (`known_canonical`), при чтении учитываются варианты (`ticker_variants`). Время в БД — наивное UTC; метка D: T-Invest 00:00 UTC, ISS 21:00 UTC предыдущих суток; метки W/M — понедельник / 1-е число, 00:00 по дате МСК. Бар, которого нет внутри покрытого диапазона, — «сделок не было».

**Агрегатор.** Выравнивание корзин — кратные интервала от эпохи UTC (`30m` — :00/:30, `1h` — час UTC, `4h` — 00/04/08/12/16/20 UTC, как режет T-Invest); D — полночь даты метки; W, M — по дате МСК (D-бар ISS стоит на 21:00 UTC предыдущих суток, T-Invest — на 00:00 UTC). O — первый бар, H — max, L — min, C — последний, V — сумма; группа без баров даёт отсутствие бара; последняя корзина, в которую ещё приходят минуты, помечается формирующейся (`is_complete=False`). `TIMEFRAME_DURATIONS["M"]` — 30 суток (оценка длительности); границы месяца — только `next_period_start`. Пакетная `aggregate` (история) и `AggregatingCandle` (поток) на одном отрезке минут дают одинаковые бары (тест).

**Чтение свечей — `MarketDataService.get_candles`.** `(ticker, timeframe, from_dt, to_dt, user_id=None, mode, rate_limit_wait=None, adjust="splits") -> (list[CandleData], source)`; диапазон ≤ `MAX_REQUEST_SPAN` (10 лет) и выравнивается по барам ТФ; `mode != "viewer"` без T-Invest — `TInvestRequiredError`.
- Масштаб (дефект 4 тестового стенда): строки в БД — «как было»; `adjust="splits"` (по умолчанию; так читают графики, бэктест, сессии, поток) — бары до наступивших дроблений в текущем масштабе, `adjust="none"` — «как было» (задел Б-02, маршрут `/candles?adjust=`). Пересчёт хранимых рядов — только в `_read_stored` (`adjust_for_splits`; дробления — `load_splits` или уже прочитанные `_series`), цены из кэша — только в `_latest_bar` (`adjust_price`). Ряд ISS перед записью — `to_as_was(…, scale_volume=False)` (`_iss_stored_series`); производный ТФ ISS (`_iss_derived_series`, без кэша) — сначала «как было», затем `adjust`.
- С T-Invest (`_tinvest_series`): ISS-строки хранимого ТФ чистятся (`_purge_iss_cache` — вместе с покрытием `moex_iss`, одной транзакцией) → `missing_ranges(…, "tinvest", …)` в `[start, min(end, now))` (для 1m — не раньше шва) → `broker_context` + `BarLoader.load` → `_series` читает хранимый ряд и собирает агрегат. Сбой контекста (счёт, FIGI, лот) трактуется как недоступный источник: отдаётся кэш, ничего не пишется. С A-02 — только с шва; ISS в окне ТФ читается (§5.6.8).
- Без T-Invest (только viewer): хранимые 1m/D — `_iss_stored_series` (покрытие `moex_iss`); производные ТФ — `_iss_derived_series` прямо из ISS без кэша; `30m` — из 10m ISS. Покрытие ISS кончается на последней полученной свече (`ISS_DATA_DELAY_MARGIN` = 1 ч): иначе задержанные минуты запомнились бы как «сделок не было» (ADR 0009). Для границы по ряду ISS — `forming_bar_start(…, msk_midnight_labels=True)`.
- Агрегат закрытого диапазона производного ТФ — в `AggregateCache`: 32 записи, не более ~~500 000~~ 80 000 баров (A-02, G01), ключ `(ticker, timeframe, from, to, exclude_iss, наступившие дробления)` (`active_splits`; `None` — чтение «как было»): новое дробление или наступившая граница дают новый ключ, БД не переписывается; запись свечей и чистка вытесняют пересекающиеся записи (`invalidate`), `generation` не даёт положить агрегат, прочитанный до записи.
- Формирующийся бар (`is_complete=False`) идёт в ответ в viewer и trading, в БД и в LRU не попадает; бэктест его не получает. К результату — ~~фильтр аукционных свечей 1h/4h и~~ `_validate_candles` (фильтр O=H=L=C для 1h/4h удалён в A-03 во всех режимах — он убирал бары аукциона открытия 1h 06:00, 4h 03:00, в выходной 1h 09:00; решение заказчика 2026-10-26, ADR 0017).
- Цена из кэша для CB, P&L, карточек, Telegram, стратегий — `last_cached_bar` / `closed_bar_close_before` (подписи и смысл прежние, вызывающий код не менялся): close последней строки хранимого ряда (1m для внутридневных ТФ, D для D/W/M) раньше начала формирующегося бара ТФ сессии (`forming_bar_start`; у D из ISS — граница по полуночи МСК); у `closed_bar_close_before` ещё нижняя граница `max_age`. Масштаб (`_latest_bar` → `adjust_price`, параметр `scale_at`): `last_cached_bar` — close строки раньше границы наступившего дробления в текущем масштабе, как у графика; `closed_bar_close_before` (оценка CB закрытых неоценённых сделок) — в масштабе момента `moment`: сделка, закрытая до дробления, считалась в старых штуках, а `process_split` пересчитывает только открытые позиции. Время в ответе `last_cached_bar` — начало закрытого бара ТФ (по нему CB судит о свежести). Читатель смотрит и хранимый ряд, и строки самого ТФ сессии (`_price_timeframes`) — ради тестов, засевающих 1h.
- `get_dealer_candles` — только с T-Invest и только внутридневные ТФ (`ValueError` → 422 для D/W/M); покрытие `tinvest_dealer`, тот же `BarLoader`; маршрут `GET /api/v1/market-data/candles/dealer`. С v4.3 (Б-02) — `get_dealer_candles(self, ticker, timeframe, from_dt, to_dt, user_id, adjust: SplitAdjust = "splits")` → `_read_stored(ticker, "1m", start, end, exclude_iss=False, model=OHLCVDealer, adjust=adjust)`: в режиме «как было» дилерский ряд в том же масштабе, что основной (кэша агрегатов у дилерского ряда по-прежнему нет). Известный дефект (ADR 0011, заказчик 2026-10-26 — не чинить): T за 15–16.04.2026 при `adjust=none` — ×10.

**Загрузка — `BarLoader(session_factory, *, archive_lots=None).load(ticker, timeframe, ranges, ctx, *, now, rate_limit_wait=None, use_archive=False) -> LoadResult`.**
- `timeframe` — только `1m` или `D`. 1m — окна по суткам UTC, не раньше шва 2018-06-13; D — окна `TIMEFRAME_CHUNK_DAYS["D"]`.
- Окно `GetCandles` (адаптер шлёт `CANDLE_SOURCE_INCLUDE_WEEKEND`): закрытая `exchange` → `ohlcv_cache`; закрытая `dealer` (только 1m) → `ohlcv_dealer`; `unknown` не пишется; незакрытая не пишется и возвращается в `LoadResult.forming`.
- Невалидная свеча (`_is_valid_candle`) не пишется, окно с ней не покрывается, считается в `LoadResult.invalid` (лог `candle_invalid_skipped`, ADR 0008). Покрытие окна `[a, b)` — до `min(b, forming_bar_start, начало первой незакрытой свечи)`; при свече `unknown` покрытие окна не пишется ни для какого ряда. *С A-02 (ADR 0030) из покрытия вырезается только период свечи, §3.5.2.*
- Свечи окна и покрытие — одна транзакция в короткой сессии (`persist_with_retry_fresh`); после записи — `aggregate_cache.invalidate`. Сбой окна — в `LoadResult.failed`, дальше следующее; `RateLimitExhaustedError` — записанное остаётся, `rate_limited=True`, остальное не запрашивается.
- Масштаб: дробления бумаги перечитываются на каждое окно `GetCandles` (`_splits` → `load_splits`); свечи окна перед записью — `to_as_was` по дроблениям, наступившим к моменту ответа окна (`fetched_at` — фактическое время, не время старта загрузки; цена × rt/rf, объём × rf/rt), §3.5. Дробление, записанное или наступившее во время долгой фоновой догрузки, учитывается со следующего окна.
- Объём `GetCandles`: `volume = лоты × lot_size` (текущий лот — источник считает в нём); `TInvestMapper.candle_to_candle_data(candle, *, lot_size, source_override=None)` — `lot_size` обязателен. Для истории лот строгий (`ensure_lot_size_strict`; `LotSizeUnavailableError` — загрузки нет), для потока — нестрогий `ensure_lot_size` (живая свеча — показ).
- `use_archive=True` (только `backfill`; для 1m нужен `archive_lots`, иначе `ValueError`): прошедшие будние торговые дни 1m — из `history-data` по году; выходные, праздники, сегодня и годы с отвергнутым архивом — `GetCandles`. По каждому году — `ArchiveLots.admit` по всем минутам суток каждого дня (§3.5.5); дни с решением `None` уходят в `GetCandles`, у принятых пишутся только минуты окна биржевых часов (`ArchiveDay.takes`), объём = лоты × лот на дату; день «сделок не было» (`window=None`) покрывается без строк. Запросы графика, бэктеста и сессий архив не трогают. `fetch_history_year(token, figi, year)`: 404 — `None`; 429 — ожидание сброса и до 3 повторов; общий предел 30 запросов в минуту на процесс; битый архив — `HistoryArchiveError`; минута архива с ценой ≤ 0 не пишется, день не покрывается.

**Фон при входе.** *До v4.5; с A-02 заменено — §5.6.8 «Фоновая догрузка».* `auth/router.py` после логина запускает `backfill.run_backfill(user_id, AsyncSessionLocal)` (`create_task`, ошибка логин не блокирует; один прогон на процесс, повторный вход во время прогона ничего не запускает). Цели: `ohlcv_backfill_target` ∪ бумаги активных сессий пользователя (глубина — `INITIAL_PERIOD_DAYS` ТФ сессии, `30m` — 14 суток; хранимый ТФ — `stored_timeframe`); по ключу берётся самая ранняя глубина; сначала D, затем 1m, до начала сегодняшних суток UTC. Один `MOEXISSClient` на прогон — для `ArchiveLots` и для сверки; закрывается в `finally`. Затем `volume_check.reconcile_volumes`: закрытые полностью покрытые несверенные дни, объём 1m за день против итога дня ISS `history` TQBR, для дней без него — дневки `candles.json` (обе стороны «как было», §3.5.3); `classify_ratio` — допуск 1 %, кратность лота 10/100/0,1/0,01 → `lot_change` (теперь — только настоящая аномалия); расхождение пишется в `ohlcv_volume_check` и в лог `volume_check_mismatch`, данные не правятся. Токен — вошедшего пользователя.

**Поток и закрытые бары.**
- `stream_manager`: см. §5.4.5. `subscribe`/`open_stream`/`ensure_stream` принимают `user_id` (владелец стрима — для истории и трекера). Дозагрузка головы бара после обрыва (`_fetch_gap_minutes`, S8R-AUDIT-012) — один REST-запрос минут мимо `BarLoader`, только закрытые биржевые, в БД не пишутся; он общий для всех ТФ ключей тикера, включая `30m` и D/W/M.
- Закрытые минуты — отдельное соединение (`waiting_close=True`, свой экземпляр мультиплексора: `get_or_create_multiplexer(token, app_name, *, waiting_close=False)`); одна подписка на бумагу, пока у неё есть открытые ключи (`acquire`/`release` зовёт `stream_manager` вне `_key_locks`). Упавшая или отклонённая подписка переоткрывается с паузой 5 с → 5 мин, пауза сбрасывается первой минутой новой подписки. Квота подписок T-Invest (100 в минуту) расходуется второй подпиской 1m на бумагу.
- Бар ТФ закрывается таймером на `bar_close_at` (конец периода или торгов последнего торгового дня периода + `BAR_CLOSE_GRACE`), а не приходом следующей минуты; ночью и в неторговые дни таймеров нет. Конец торгов — 23:50 МСК (`DEFAULT_TRADING_END`) и 19:00 МСК в день торгов выходного дня; торговый ли день — календарь MOEX (до A-03 расписание по периодам в коде не заведено, два места знают часы константами: `bar_close.py` и `common/trading_hours.py`). **С A-03 (ТЗ v4.1):** открытие и конец торгов дня — из таблицы периодов (`TradingSchedule.day(d)`: `exchange_open` — начало первого биржевого периода, `trading_end` — конец последнего биржевого, не дилерского; сейчас 23:50 в будни и 19:00:01 в день торгов выходного дня — конец T-Invest включителен до секунды); день торгов выходного дня (`DaySchedule.weekend`) — только при `weekend_trading`; день без биржевых периодов — торгов нет; даты без строки — `fallback_day`. Констант часов в `bar_close.py` больше нет.
- `bar.closed` → `event_bus.publish("bars:{ticker}:{tf}", "bar.closed", payload)`; payload: `ticker`, `timeframe`, `timestamp` (ISO начала бара, UTC), `open`, `high`, `low`, `close` (float), `volume` (int, штуки), `complete`, `revised`. ~~Подписчиков нет (задел для алертов A-11, не `⚠️ NOT CONNECTED`)~~ С A-03 подписчики — торговые сессии (`_SessionListener.listen_bar_events`, §5.4.6), алерты A-11 присоединятся позже; это событие шины, не уведомление: в `EVENT_MAP` и `EVENT_TYPE_LABELS` его нет; WS-подписка клиента на `bars:*` отклоняется. Дилерские минуты в бары и `bar.closed` не попадают — трекер сдаёт их `BarLoader.store_stream_dealer_minutes` (A-03). Сверка D с родной дневкой с A-03 получает настоящий порог — `instruments.min_price_increment` (раньше колонку никто не заполнял, порог был 0).
- Неполный период (подписка открыта внутри периода, реконнект, обрыв) — бар из истории (`get_candles(…, "1m", период, mode="backtest")`) ∪ минуты потока; история неполна (`IncompleteHistoryError`) или недоступна — `complete=False`. Опоздавшая минута — пересчёт и повтор с `revised=True` (держатся два последних бара ТФ). D: сверка с родной дневкой в 00:30 UTC следующих суток, расхождение больше `min_price_increment` — повтор D с родными значениями; W/M не переопубликуются. W/M собираются из родных закрытых D периода плюс дни, которых в истории ещё нет.
- Признак торгов выходного дня трекер берёт только из кэша процесса (`weekend_trading_flag` без счёта портит этот кэш и отключил бы бумаге вход в CB — ADR 0007); нет кэша — W/M закрываются как у бумаги без торгов выходного дня, минуты выходных приходят пересчётом (`revised`).
- Сессии (`SessionRuntime`) не менялись: решение по бару — по приходу следующего (ADR 0005, 0006) — **с A-03 заменено:** бар закрывается по `bar.closed` / страховочному таймеру / следующему бару, вход вне торгов откладывается (ADR 0016, §5.4.6); D/W/M получают поток, `needs_subscription` их больше не исключает, watchdog их не пропускает. `SupportedTimeframe` (`trading/schemas.py`) включает `30m`, годовой коэффициент Sharpe для `30m` — `_INTRADAY_TF_MINUTES` (30 минут, прежняя формула; по фактическому числу баров — A-06).

**Инварианты.** В БД пишутся только закрытые биржевые 1m и D (`_save_to_cache` другого ТФ не примет). Строки — «как было»; пересчёт по дроблениям — только при чтении (`_read_stored`, `_latest_bar`), запись в масштабе «с учётом дроблений» — ошибка (при чтении пересчиталась бы второй раз). Свечи T-Invest пишет только `BarLoader`; удаляет свечи только `delete_bars_with_coverage` (иначе дыра запомнится как «сделок не было»), data-миграция делает то же SQL-ом по `ohlcv_coverage` (gotcha-90). Правило «бар закрыт» одно — `bar_close.py`. Токен T-Invest не попадает в `repr` и логи (`BrokerContext.adapter/token`, `StreamSource.api_key/api_secret` — `repr=False`; в архиве — только заголовок `Authorization`; логи фона без `exc_info`). Состояние процесса (LRU, трекер, флаг прогона, лимитер архива) — с reset-функцией в `tests/conftest.py` (`_reset_bar_storage_state`); процесс один (один uvicorn-worker). Сеть трекера (подписка, история) — вне `_key_locks`. Повторы брокера — только на чтениях; ордеров блок не касается.

**Тесты.** `tests/test_market_data/` — по модулю (`test_aggregation.py`, `test_bar_close.py`, `test_coverage.py`, `test_bar_loader.py`, `test_closed_bars.py`, `test_backfill.py`, `test_volume_check.py`, `test_candle_service.py`, `test_stream_aggregation.py`, `test_bar_storage_migrations.py`, `test_split_storage.py` — масштаб «как было», лот на дату, пересчёт при чтении на примерах T и SBER со стенда; `test_reference_days.py` — эталонные дни стенда: T у дробления, смена лота SBER, утренний дилер 2024, неполные дни LKOH и GAZP, 10.04.2025, праздничные сессии, суббота; дефект 7 — выходные и праздник в итоге следующего рабочего дня ISS (GAZP 10–12.10.2026, SBER 05–07.09 и 21–24.02.2026), архив SBER 2023 без минуты 09:59, лента T-Invest против ISS 21.10.2026; `tests/unit/test_moex_iss/test_client_daily_volumes.py` — `calendar_day_volumes`); фикстуры эталонных дней — `fixtures/reference_days/` (GAZP, LKOH, MTSS, SBER, T; запись — `backend/scripts/record_bar_reference_days.py`: только с явным `DATABASE_URL` копии БД, иначе код 3; `--ticker` дописывает `meta.json`; тесты — `tests/unit/test_scripts/test_record_bar_reference_days.py`); клиент `get_daily_volumes` — `tests/unit/test_moex_iss/test_client_daily_volumes.py`; фикстура `fixtures/tinvest_native_bars.csv` — 1m и родные 5m–4h из копии рабочей БД. Сверка агрегата с T-Invest вживую (включая родные `30m`) — `backend/scripts/check_aggregation_vs_tinvest.py` на тестовом стенде. Сеть в тестах закрыта (`_block_outbound_http`, `_block_live_grpc`): HTTP и gRPC — фейки.

**Известные ограничения и долги v4.0.**
- Вживую не проверено (T-Invest не был доступен при приёмке): загрузка биржевых и дилерских свечей, правило «неизвестный источник не пишется» (заполняет ли T-Invest источник у будних биржевых свечей — если нет, окна будут запрашиваться повторно), объём GAZP против ISS (дневка ISS запрашивается без `boards/TQBR` в расчёте, что для акций она отдаёт основной режим; если на стенде это не так — запрос надо привязать к борде), архивы, фоновая догрузка, закрытые минуты, `bar.closed`, живые D/W/M, сессии D/W/M (стопы, тейки, watchdog и восстановление после обрыва на этих ТФ в бою раньше не работали), сверка агрегата с родными свечами.
- День архива идёт в ряд только при точном совпадении с итогом дня ISS `history` TQBR; дни без совпадения, без подтверждённого лота, бумаги не из TQBR (фонды — архив для них не используется) и год при недоступном ISS грузятся `GetCandles` — больше запросов фоновой догрузки, данные верные. Пересчёт знает только дробления из `corporate_actions` (детект — каждые 6 часов, и без сессий): пока дробление не записано, строки `GetCandles` за дни до его границы, загруженные уже после неё, ложатся в БД в новом масштабе; детект, узнав дробление после границы, до записи удаляет строки бумаги до границы (`forget_bars_before_new_splits`, §5.12), они дозагружаются.
- Предполагается, что источники (`GetCandles`, ISS `candles.json`) пересчитывают историю ровно с 00:00 МСК `ex_date` (`split_opened_before`).
- Между уборкой рядов и записью дробления (§5.12) идущая в этот момент загрузка может записать окно в масштабе источника (дробление ещё не в `corporate_actions`).
- Бары W/M, взятые прямо из ISS (viewer без T-Invest, `_iss_derived_series`), внутри которых лежит граница дробления: объём части после дробления завышен в rt/rf раз: бар с началом до границы пересчитывается целиком, а разрезать готовый агрегат ISS по границе нельзя.
- Сверка объёма дней из архива тавтологична: в ряд они попадают только при точном совпадении с тем же `history`. Она ловит расхождения дней `GetCandles`.
- Для Б-02: T-Invest пересчитывает дилерские свечи по своей границе дробления — у T 15.04.2026, на два дня раньше `ex_date` 17.04. Чтение по умолчанию верно; «как было» (`adjust=none`) дилерского ряда T за 15–16.04 — ×10.
- Мимо архива (`GetCandles`, данные верные, лишние запросы): дни 2023 года — в архиве 2023 нет минуты открывающего аукциона 09:59 МСК (SBER 17.10.2023 — 106 700 шт), ≈ 250 запросов один раз на бумагу; единичные будни, где лента T-Invest (и архив, и `GetCandles`) расходится с ISS в 1–28 минутах (14.11.2025, 17.06 и 21.10.2026 — у всех бумаг), их сверка — `ok`. Выходные и праздничные сессии из архива не берутся (`GetCandles`), хотя в часы сессии архив равен ISS (дилер — вне их).
- Окно от границы дробления до `process_pending` (раз в 6 ч): цена из кэша уже в новом масштабе, а позиция — ещё нет. Это ограничение было и раньше — для живой цены.
- Минуток T-Invest раньше шва нет; внутридневной истории до 2018-06-13 нет до блока A-02. *Снято в v4.5: 1m до шва — ISS (§5.6.8).*
- Окно со стабильно невалидной свечой не покрывается и перезапрашивается каждым обращением (для D окно — год). *Снято в v4.5 (ADR 0030).*
- Без T-Invest бар истории считается закрытым по расписанию, даже если минуты или дневные до конца бара не загрузились.
- Гонка закрытия и переоткрытия потока может стереть состояние живого ключа; префикс W/M живого потока запрашивается раз в день (сбой на первой свече дня оставляет бар W/M неполным до конца дня); соединение закрытых минут тоже публикует «связь потеряна/восстановлена» и может съесть интервал подавления настоящей.
- `AggregateCache` — до 500 000 баров на процесс (≈ 300 МБ), порог к снижению — `AGGREGATE_CACHE_MAX_BARS`. *С v4.5 — 80 000 (`AGGREGATE_CACHE_MAX_BARS`).*

**Дополнения A-03 (ТЗ v4.1) к хранилищу и потоку.** Дилерские свечи потока (ADR 0021): закрытые минуты — в `ohlcv_dealer` по всем бумагам с открытым потоком (держатель — график, сессия, монитор алертов), покрытие поток не пишет (ряд, пополняемый потоком, полным не считается — окно истории будет запрошено); в режиме по умолчанию свеча без `candle_source_type` не маршрутизируется вовсе — если T-Invest где-то не заполняет источник у биржевых свечей, они пропадут из потока с warning (в фикстурах блока — только дилерские свечи выходных 24–25.10; на тестовом стенде 02–04.11 биржевые свечи нового потока приходили). Известные долги: REST-подписка на `dealer:` без WebSocket заводит состояние ключа без держателя (`stream_manager.py`); канал `dealer:` вживую не проверен (подписчика нет до Б-02). Тесты — `tests/test_market_data/test_dealer_stream.py`, `test_dealer_minutes_store.py`, `test_auction_bars_kept.py`.

**5.6.7 Уровневые алерты (блок A-03, ТЗ v4.1; ADR 0023).** Требования — ФТ §5.8. Пути — от `backend/app/`.
- **Модель** — `market_data/models.py`: `Alert` (`alerts`, вид `level`; `armed` — взведён, `is_active` — не удалён; поля D2 — задел A-11) и `AlertTrigger` (`alert_triggers`), §3.13. `PriceAlert` удалён.
- **Маршруты** — `market_data/price_alert_router.py`, префикс `/api/v1/market-data`: `POST|GET /price-alerts`, `DELETE /price-alerts/{id}` (мягкое удаление), `GET /price-alerts/quote` (§4.0). Создание (`AlertLevelCreate`: `ticker`, `target_price`; `condition` старого клиента отбрасывается): `canonical_ticker` → `resolve_alert_figi` → `current_quote` — `TInvestAdapter.get_last_price` (`GetLastPrices`) токеном счёта пользователя, иначе ISS с `warning` «задержка 15 мин» (`ISS_DELAY_WARNING`); цены нет — 409 `NO_PRICE_DETAIL`; `round_to_price_step(target_price, price_step)`; уровень = цене — 422 `LEVEL_EQUALS_PRICE_DETAIL`; `condition = "above" if level > price else "below"`, `price_at_creation`, `price_source`; затем `PriceAlertMonitor.refresh`.
- **Шаг цены** (справочник `instruments`, иначе `TInvestAdapter.get_price_step`) — `common/price_step.py`: `round_to_price_step` (к ближайшему шагу; одна функция с B3 и C1), `price_step_decimals`, `quote_price_decimals(step, price)` — знаки шага, без шага — знаки цены, от 2 до 8 (`D04`). Шаг — `instruments.min_price_increment` из `GetInstrumentBy` (`_InstrumentRef.min_price_increment`) тем же проходом, что лот, в `MarketDataService.ensure_lot_size`: при первом обращении к бумаге и раз в сутки (`PRICE_STEP_TTL`, по `lot_size_synced_at`), попытка по тикеру не чаще раза в час (`PRICE_STEP_RETRY`); читатель — `instrument_price_step(db, ticker)`; неположительный шаг — как отсутствующий.
- **Проверка** — `market_data/price_alert_monitor.py::PriceAlertMonitor` (S8R-FIX-051: сам держит подписку 1m на FIGI с активными алертами токеном счёта владельца). Мультиплексор кладёт в очередь `AlertMinute` (OHLC минуты) **только биржевых** свечей потока обновлений (дилерские — нет; соединение закрытых минут алерты не проверяет). `check_alerts_for_figi(figi, minute)`: `level_touched` — рост `high ≥ уровень`, падение `low ≤ уровень`; минута создания — только по текущей цене (`close`); open уже за уровнем — срабатывание по open с пометкой гэпа. Срабатывание — условный `UPDATE alerts … WHERE armed AND is_active` (повтор исключён), строка `AlertTrigger`, уведомление `stage_notification` — одна транзакция; сбой — откат, алерт взведён. Повторное взведение (`_rearm`) — на смене минуты, когда close закрытой минуты вернулся на сторону создания (`armed = True`). Подписка снимается, когда активных (не удалённых) алертов по FIGI у токена не осталось — сработавший, но не удалённый держит подписку.
- **Уведомление** — `system.alert` → `event_type` `alert`, заголовок «Алерт», тело `"{ticker}: цена поднялась|опустилась до {цена} (уровень {уровень})"` или `"{ticker}: уровень {уровень} пройден гэпом, цена {цена}"` (`format_price`), `related_entity_type = "alert"` (кнопка Telegram и переход — по `Alert.ticker`), §5.7.
- **Фронтенд** — §6.4.
- **Тесты** — `tests/test_market_data/test_price_alerts.py`; фронт — `PriceAlertModal.test.tsx`, `priceAlertStore.test.ts`; E2E — окно алерта.
- **Ограничения:** уровень меньше половины шага округляется до 0 (`price_alert_router.py`); рядом с уровнем — серия срабатываний (повторное взведение); без брокера направление — по цене ISS с задержкой; откат миграции снова делает сработавшие алерты активными и теряет историю; переход к графику из уведомления «Алерт» — вне блока.

**5.6.8 Старая история через шов, фон по расписанию, счёт для рыночных данных (блок A-02, ТЗ v4.5; ADR 0030–0035).** Требования — ФТ §11.4 («Старая история через шов»), §7.7, §4.2. Как устроено — `docs/architecture.md` (главы «Окна источников — `seam.py`», «Чтение свечей», «Старая история ISS — `IssHistoryLoader`», «Фоновая догрузка — `backfill.py`»), термины — `CONTEXT.md`, решения — `docs/adr/0030-g02-invalid-candle-uncovers-only-its-period.md`, `0031-a02-iss-window-per-timeframe-and-seam.md`, `0032-a02-iss-daily-label-midnight-utc-msk-date.md`, `0033-a02-phantom-daily-bars-iss-days-and-copy-rule.md`, `0034-a02-sync-one-year-rest-in-background-pending-flag.md`, `0035-a02-market-data-account-flag-on-broker-account.md`. Пути — от `backend/app/`.

```
 seam.py: TINVEST_MINUTE_SEAM 2018-06-13 · ISS_MINUTE_FROM 2011-12-01 · ISS_DAILY_FROM 2018-03-07 · PHANTOM_DAILY_BEFORE 2019-01-01
 GET /candles, бэктест ─▶ get_candles(viewer|backtest) ─▶ T-Invest вне окна ISS, ≤ SYNC_TINVEST_SPAN (365 сут), самое свежее ─▶ BarLoader
                              └─ недостающее (T-Invest старше года, ISS в окне) ─▶ backfill.request_history(ticker) + history_pending
 SchedulerService.history_backfill (04:00 MSK + старт) ─▶ run_history_backfill ─▶ collect_universe ─▶ очередь ─▶ один исполнитель:
     T-Invest D с 2000 ─▶ T-Invest 1m с шва (архивы)  — BarLoader, broker_context(ticker, account_id=market_data_account(db).id)
     ISS D окна ─▶ ISS 1m от шва назад по месяцам   — IssHistoryLoader (тикер ISS — ticker_aliases.iss_ticker_periods)
     ─▶ reconcile_volumes ─▶ history_backfill_done
 BarLoader: D раньше 2019 (акции, фонды) ─▶ IssHistoryLoader.trading_days ─▶ фантом не пишется (daily_phantom_skipped)
```

| Модуль | Владеет | Чего не делает |
|---|---|---|
| `market_data/seam.py` | константы шва и окон; `iss_window`, `iss_load_window`, `outside_iss_window`, `inside_iss_window`, `iss_daily_label` | БД, сеть; миграции держат копию литералов |
| `market_data/iss_history.py` | `IssHistoryLoader` — единственный писатель старой истории ISS: `load_minutes`, `load_window_daily`, `trading_days`; `iss_trading_days` (для проверки ряда бэктеста); `COVERAGE_SOURCE_ISS_INVALID` | свечи T-Invest не пишет; 1m синхронно не грузит |
| `market_data/backfill.py` | `collect_universe`, очередь и исполнитель (`history-backfill`), `run_history_backfill`, `request_history`, `stop_history_backfill`; сверка объёма после очереди | вход пользователя его не запускает; чужой токен не берёт |
| `market_data/market_data_account.py` | `market_data_account(db)`, `select_market_data_source(…)` с аудитом | запросы пользователя (`service.select_market_data_account`) не трогает |
| `market_data/ticker_aliases.py` | `iss_ticker_periods`, `forget_alias_history` | не коммитит |
| `admin/ticker_aliases.py` | CRUD соответствий, проверка пересечения периодов, аудит, вытеснение `AggregateCache` | |
| `market_data/service.py` | `_purge_iss_cache` с шва, чтение ISS в окне (`_tinvest_series_rows`), `SYNC_TINVEST_SPAN` / `_freshest`, `history_pending` (`HistoryPending`), `_series_chunks`, `_load_iss_window_daily` (viewer без T-Invest) | свечи T-Invest не пишет |
| `market_data/bar_loader.py` | без записи в окно ISS; фильтр фантомов (`trading_days`, `on_iss_shares_market`), отметка `tinvest_invalid` | ISS не пишет |
| `broker/moex_iss/client.py` | `get_minute_candles` (без режима торгов, окно — месяц), `get_history_days` (`history` всех режимов, строка с наибольшим объёмом на дату, страницы по 100), `get_board_tickers` (`SECID` режима) | |
| `scheduler/moex_calendar.py` | `known_from` — первая дата `dailytable` (у ISS — 09.05.2018) | |

**Окна и чтение (ADR 0031).** Ряд пользователя с T-Invest стыкуется по «окну ISS» хранимого ТФ (`seam.iss_window`): 1m — всё раньше шва (грузится с 2011-12-01), D — `[2018-03-07, шов)` по дате МСК метки. ISS-строки читаются только в окне своего ТФ, T-Invest пишет только вне окна (`seam.outside_iss_window`). `_purge_iss_cache` удаляет ISS-строки и вырезает покрытие `moex_iss` только с шва (дневки — по дате МСК: ISS-дневка 13.06.2018 со старой меткой `2018-06-12 21:00` численно раньше шва, но вне окна). ISS-дневки раньше 07.03.2018 (их пишет просмотр без T-Invest) не удаляются и в ряд с T-Invest не читаются. `exclude_iss=True` у Circuit Breaker (`circuit_breaker/engine.py`, `unpriced.py`) — цена последнего бара не меняется.

**Старая история ISS (`IssHistoryLoader`; ADR 0032).** `load_minutes` — `[2011-12-01, шов)` минус покрытие `moex_iss`, окнами по календарному месяцу МСК от поздних к ранним (≈ 25 страниц, выборка не упирается в `ISS_MAX_PAGES`); ошибка окна — окно не покрыто, остальные окна бумаги в этом вызове не запрашиваются (`iss_history_window_failed`); пустой ответ — окно покрыто. `load_window_daily` — дневки по датам МСК окна: OHLC и объём из `candles` (без режима торгов), свеча возвращается «как было» (`splits.to_as_was(…, scale_volume=False)`), затем `close` → `LEGALCLOSEPRICE` из `history` (строка с наибольшим объёмом на дату; база `history` — «как было», второго пересчёта нет, gotcha-92); метка — 00:00 UTC даты МСК (`iss_daily_label`); покрытие — сутки МСК дат окна. Нет `LEGALCLOSEPRICE` — `iss_window_daily_without_legal_close`. `trading_days` — дни с `VOLUME > 0` по `history` всех режимов; сбой — `BrokerError`. Время — `parse_iss_datetime` (МСК → UTC, фиксированное +3), объём — штуки. Запись — `ohlcv_cache` (`source='moex_iss'`, `ON CONFLICT DO NOTHING`, пачки по 2000) и покрытие одной транзакцией в своей сессии (`persist_with_retry_fresh`), затем `aggregate_cache.invalidate`.

**Фантомные дневки (ADR 0033).** `BarLoader(…, trading_days=IssHistoryLoader.trading_days)`: дневка T-Invest с датой МСК раньше 2019-01-01 у бумаги рынка акций ISS (`on_iss_shares_market`: акции, фонды; вид неизвестен — как у акций) пишется, только если день есть в `trading_days`; иначе — `daily_phantom_skipped`, день покрыт. Календарь ISS за окно пуст, а у T-Invest дневки есть — ISS бумагу не знает: без фильтра. `trading_days` упал — дневки окна раньше 2019 года не пишутся, окно в `failed`. Окна D у таких бумаг не пересекают 2019-01-01 и вырезают окно ISS.

**Синхронно и «догружается» (ADR 0034).** `get_candles(…, mode="viewer"|"backtest")` с T-Invest: недостающее вне окна ISS — только самые свежие `SYNC_TINVEST_SPAN` (365 суток, `_freshest`); `trading` — целиком, как раньше. После загрузки `_history_pending` по покрытию: недостающее T-Invest вне окна (до начала сегодняшних суток UTC) и ISS в окне (1m — не раньше 2011-12-01); период с отметкой `*_invalid` недостающим не считается. Есть недостающее — `backfill.request_history(ticker)` и `self.history_pending = HistoryPending(reason)`: `no_market_data_account`, если недостающий T-Invest есть, а `market_data_account(db)` — `None`, иначе `loading`. Признак — атрибут экземпляра (как `fetch_quality`), сбрасывается в начале каждого `get_candles`; читать сразу после вызова тем же экземпляром. Без T-Invest (viewer): минутки до шва синхронно не грузятся (фон), дневки окна — `_load_iss_window_daily`, ISS — по тикеру ISS (`_fetch_via_iss`). Производный внутридневной ТФ читается кусками ≤ 31 суток UTC (`_series_chunks`).

**Фоновая догрузка (`backfill.py`; ADR 0034).** Запуск — задание `history_backfill` (§5.9) → `run_history_backfill(db_factory)` (сам не бросает; ждёт, пока исполнитель разберёт очередь); бумага синхронного запроса — `request_history` (фабрика — идущего прогона или `database.AsyncSessionLocal`). Вселенная `collect_universe(db)` — `DISTINCT` тикеры `trading_sessions` (любой статус), `backtests`, `alerts` с `is_active`, `user_favorites` (`kind='instrument'`), `ohlcv_backfill_target`; тикеры — канонические (`known_canonical`), по алфавиту. Очередь `deque` + множество (без повторов), один исполнитель на процесс (`asyncio.Task` `history-backfill`), один `MOEXISSClient` на исполнителя (страницы ISS строго по одной; для старой истории, `ArchiveLots` и сверки). По бумаге (`_load_ticker`): счёт `market_data_account(db)` — есть и лимит не исчерпан: `_load_tinvest` — `broker_context(ticker, account_id=…)`, `BarLoader(archive_lots=ArchiveLots, trading_days=…)`, `missing_ranges(…, "tinvest", depth, today)` для D с `DAILY_FROM` (2000-01-01) и 1m с шва, `use_archive=True`; `RateLimitExhaustedError` — T-Invest-часть остальных бумаг прогона пропускается (`backfill_rate_limited`); нет счёта — один раз за прогон `market_data_account_missing`. Фьючерс — ISS-часть не нужна (`history_backfill_iss_skipped`). Затем `load_window_daily` и `load_minutes`. После опустошения очереди — `reconcile_volumes` по всем бумагам. `stop_history_backfill` (из `SchedulerService.shutdown`) гасит исполнителя, очередь забывается — следующий прогон продолжит «запрошенное минус покрытие». Логи — без `exc_info` (в кадрах — расшифрованный токен, S8R-AUDIT-036). `auth/router.py` больше не создаёт задачу `backfill-user-{id}`.

**Счёт для рыночных данных (ADR 0035)** — §3.2. Фон T-Invest (свечи, дивиденды §5.15) идёт токеном только этого счёта (`decrypt_market_data_token`, в памяти). Счёт может выпасть из выбора без события (владелец деактивирован, счёт неактивен, ключ удалён) — `market_data_account` вернёт `None`, фон T-Invest молча стоит.

**Соответствия тикеров** — §3.5.6. Действуют на все загрузки ISS (`_fetch_via_iss`, `IssHistoryLoader`, `trading_days`); не действуют на лот архива и сверку объёма (`lot_history.py`, `volume_check.py` спрашивают ISS по каноническому тикеру — известный долг).

**Константы** (новых настроек в `config.py` нет):

| Константа | Значение | Где |
|---|---|---|
| `TINVEST_MINUTE_SEAM` | 2018-06-13 | `market_data/seam.py` (в `bar_loader.py` — импорт; литерал в миграциях A-01 и A-02) |
| `ISS_MINUTE_FROM` | 2011-12-01 (`candleborders` SBER — 15.12.2011) | `seam.py` |
| `ISS_DAILY_FROM` | 2018-03-07 | `seam.py` |
| `PHANTOM_DAILY_BEFORE` | 2019-01-01 | `seam.py` |
| `SYNC_TINVEST_SPAN` | 365 суток | `market_data/service.py` |
| `MAX_REQUEST_SPAN` | 3650 суток (без изменений: запрос с 2011 года до сегодня режется до первых 10 лет) | `service.py` |
| `AGGREGATE_CACHE_MAX_BARS` | 80 000 (было 500 000; G01) | `service.py` |
| `_SERIES_CHUNK` | 31 сутки | `service.py` |
| `DAILY_FROM` | 2000-01-01 — начало дневок T-Invest в фоне | `market_data/backfill.py` |
| `_WRITE_BATCH` | 2000 строк | `iss_history.py`, `bar_loader.py` |
| `ISS_KNOWN_DAYS_TIMEOUT_SEC` | 15 с | `backtest/service.py` |

**Логи:** `history_backfill_started` (`tickers`), `history_backfill_done`, `history_backfill_failed`, `history_backfill_ticker_failed`, `history_backfill_stop_failed`, `history_backfill_no_db`, `history_backfill_iss_skipped`, `history_backfill_job_failed` (планировщик), `market_data_account_missing`, `backfill_target_done` / `backfill_target_failed` / `backfill_target_skipped`, `backfill_rate_limited`, `backfill_volume_checked` / `backfill_volume_check_failed`, `bar_loader_done`, `daily_phantom_skipped`, `archive_days_via_getcandles` (A-01), `iss_history_loaded`, `iss_history_window_failed`, `iss_candle_invalid_skipped`, `iss_window_daily_without_legal_close`, `ohlcv_daily_gaps_iss_unavailable`.

**Тесты:** `tests/test_market_data/` — `test_a02_seam.py`, `test_a02_migration.py`, `test_a02_market_data_account.py`, `test_a02_ticker_aliases.py`, `test_a02_audit_actions.py`, `test_iss_history.py`, `test_series_through_seam.py`, `test_series_chunks.py`, `test_history_pending.py`, `test_history_pending_api.py`, `test_history_backfill_schedule.py`, `test_market_data_account_context.py`, `test_daily_gaps_before_calendar.py`, `test_backfill.py`, `test_bar_loader.py`; `tests/test_admin/test_admin_ticker_aliases.py`; `tests/unit/test_broker/test_market_data_source.py`; `tests/unit/test_moex_iss/test_client_old_history.py`. Фикстуры — настоящие ответы (`tests/test_market_data/fixtures/a02/`: 1m ISS SBER 07–14.06.2018, GAZP 03.03.2014 в режиме `TQBS`, `history` SBER 2014–2018, 1m T-Invest SBER 13.06.2018, дневки SBER T-Invest 2014–2018 с фантомами), скрипт записи — `backend/scripts/record_a02_fixtures.py`. E2E — `frontend/e2e/a02-backtest-history-pending.spec.ts`.

**Известные ограничения.** Признак «догружается» — на экземпляре сервиса: общий экземпляр на конкурентные вызовы даст гонку. `GET /backtest/{id}` грузит свечи (до года) и будит фон прямо в HTTP-запросе; признак в бэктесте — на момент просмотра. До выбора счёта D глубже года по бумаге без кэша не загрузится (лимит года действует и на D). Ряд на стыке проверен на SBER (close ±0,05 % на 13.06.2018); у других бумаг расхождение возможно. Время ISS 12.2011–25.10.2014 — МСК − 3 ч, на час позже настоящего UTC (весь проект — фиксированный МСК, `common/timezones.py`). «Одна страница ISS за раз» — на исполнителя, не на процесс: синхронные запросы ISS идут параллельно с фоном. Сверка объёма — после каждого опустошения очереди (`backfill.py`). Лот архива и сверка объёма — по каноническому тикеру. Облигация и бумага неизвестного вида: окно D — только фоном (синхронный путь его вырезает). Отметка `*_invalid` остаётся после исправления источником — скрывает один бар от признака до ночного фона. Новое дробление стирает ISS бумаги раньше границы вместе с покрытием (ADR 0010) — ISS до шва перегружается фоном. Бэктест без T-Invest — 409 и на периоде целиком до шва (`TInvestRequiredError`, как до блока). Узкая гонка остановки: команда остановки, пришедшая в доли секунды, пока прогон собирает список бумаг, фоновую загрузку не останавливает (замечание тестового стенда к выкатке; записано в `PROGRESS.md` блока). Мелкие долги: окно D по дате МСК живёт вне `seam.py`; `_msk_day_start` скопирован в трёх модулях; `_ISS_SOURCES` дублирует `_ISS_CACHE_SOURCES`.

### 5.7 Notification Service — ✅ реализовано S6

**Доставка (S8R-AUDIT-083).** Каналы изолированы (`asyncio.gather(return_exceptions=True)`): сбой Telegram не мешает Email и наоборот, in-app пишется независимо. Повторы только транзиентных ошибок: Telegram — 3 попытки, backoff 1/2 с, `RetryAfter` ≤ 30 с, `BadRequest`/`Forbidden`/`InvalidToken` — без повтора (gotcha-82), таймауты `HTTPXRequest` 5/10/10/5 с; SMTP (aiosmtplib) — 3 попытки на сеть/4xx, 5xx без повтора, `SMTP_TIMEOUT_SEC=15`. EventBus при переполнении очереди подписчика — `logger.error` + счётчик и метрика `event_bus.dropped`. Гарантированная доставка через `pending_events` (§2.4, §3.20) **реализована**: факты — S8R-AUDIT-086 и S8R-FIX-065, внешние каналы — строка `notification.dispatch` (S8R-FIX-051 для ценового алерта; с A-05 — для каждого уведомления) и фоновый повтор по расписанию (ниже, «Повтор внешней доставки»). Прежняя пометка «не реализована, таблица есть, потребителя нет» (S8R-FIX-040) устарела с S8R-AUDIT-086 и снята в ТЗ v4.2.

**Ответственный: BACK2 (core) + OPS (deploy)**

**Архитектура (реализация S6):**
- `NotificationService` (`app/notification/service.py`) — ядро маршрутизации: `create_notification()`, `listen_session_events()`, `stop_listening()`, `dispatch_external()`.
- Инстанциируется в `main.py` lifespan как `app.state.notification_service`.
- `listen_session_events()` вызывается при старте каждой торговой сессии через `engine.py`.
- EVENT_MAP: 6 типов событий (session_started/stopped, order_placed, trade_filled/closed, cb_triggered).
- **A-03 (ТЗ v4.1):** `EVENT_MAP["system.alert"] = ("alert", …, "Алерт")` — уровневый алерт (`PriceAlertMonitor` → `stage_notification`); `SETTINGS_INHERIT_FROM["alert"] = "price_alert"` — новый тип берёт каналы, настроенные для прежнего «Ценового оповещения»; `system.price_alert` остаётся для подписей истории, новых событий не получает. `EVENT_TYPE_LABELS.alert = 'Алерт'` (`NotificationSettingsPage.tsx`), синхронность — `test_event_sync_publishers.py`. Связанная сущность `alert` → тикер для кнопки Telegram — `Alert.ticker` по id (прежние `price_alert` — по `instrument`). Выход в списке «закрыть при первой возможности» уведомляет существующим `order.error` (§5.4.6).
- **Б-07 (ТЗ v4.9):** `EVENT_MAP["news.alert"] = ("news_alert", "info", "Новость по бумаге", …)` — алерт по новостям; строку outbox ставит `app/news/alerts.py` (`stage_for_delivery`, канал `news_alerts:{user_id}`), уведомление строит повтор outbox (`news.alert` — в `OUTBOX_REPLAY_EVENTS`, доставка своих строк после commit и при старте); `SETTINGS_INHERIT_FROM["news_alert"] = "alert"`; почты нет (`EMAIL_ALLOWED_EVENTS` не менялся); `EVENT_TYPE_LABELS.news_alert = 'Новость по бумаге'`; ключей `EVENT_MAP` — 22 (`test_event_sync_publishers.py`). Связанной сущности нет — кнопки Telegram нет (§5.16.5).
- Lazy-init нотификаторов: `TelegramNotifier` и `EmailNotifier` создаются только при наличии соответствующих env-переменных.
- Маршрутизация на основе `user_notification_settings`.

**Повтор внешней доставки (блок A-05, D5; ТЗ v4.2).** Требования — ФТ §10.1, §10.2, §17.7, §17.8. Пути — от `backend/app/`.
- **Строка на каждое уведомление.** `_create_notification_impl` (прямой `create_notification`, факты outbox, фан-аут `connection.*`) и `stage_notification` (ценовой алерт) пишут строку `notification.dispatch` (`_stage_dispatch_row`, payload — §3.20) в той же транзакции, что INSERT `Notification`, если `_plan_external_channels` вернул хотя бы один канал; после commit — фоновая доставка `_schedule_external_dispatch(…, outbox_id=…)` → `_deliver_outbox`. Правка только в `notification/*`; `trading/*` и издатели событий не менялись.
- **План каналов** — `_plan_external_channels(db, user_id, event_type, settings)` (только чтение БД): Telegram — включён в настройках, `TelegramNotifier` настроен на сервере и есть активная `TelegramLink`; почта — включена, `EmailNotifier` настроен, событие в `EMAIL_ALLOWED_EVENTS` и у пользователя есть `email`. Канал без получателя не планируется — не повторяется и не помечается (нотификатор отдаёт только `bool`, постоянную причину от сбоя по нему не отличить).
- **Доставка** — `_deliver_outbox`: каналы из `data.delivered` пропускаются; каждый оставшийся канал — через единую точку `dispatch_external` (по одному каналу за вызов, `_only_channel`), успех — `_record_delivered` (строка и `channels_sent` одной транзакцией). Все доставлены → строка забрана. Неудача — `_record_failed_attempt`: `failed_attempts + 1`, `next_attempt_at` = конец попытки + `DELIVERY_RETRY_DELAYS_SEC[попытка − 1]` (`(10, 60, 300, 1800)`); после `DELIVERY_MAX_ATTEMPTS = 5` — каналы в `Notification.channels_failed` (CSV, `_mark_channels_failed`) и строка забрана, одной транзакцией. DEV_MODE (`external_delivery_suppressed`) — строка забирается без отправки и без отметки.
- **Сторож «строка в работе»** — `self._dispatch_in_flight: set[int]`: его проверяют живая доставка, повтор при старте и фоновый повтор, строку в работе второй проход не берёт. Процесс один (один uvicorn-воркер), в порядок локов `common/locks.py` множество не входит.
- **Фоновый повтор** — `retry_due_deliveries(*, now=None, limit=OUTBOX_REPLAY_LIMIT) -> int` (сколько строк взято): `load_unprocessed(db, event_type="notification.dispatch")`; строка старше `OUTBOX_REPLAY_MAX_AGE_SEC` (1 ч) → `_expire_dispatch` (claim, затем строка перечитывается в той же транзакции; недоставленные каналы плана → `channels_failed`, отправки нет; уже забранная другим путём — без отметки); срок подошёл (`next_attempt_at`, без поля — `created_at`) → `_replay_dispatch(…, now, due_by)` по текущим настройкам пользователя; срок в будущем или строка в работе — пропуск. Повтор при старте (`replay_pending_events` → `_replay_dispatch` без срока) шлёт сразу и считается попыткой. Задача планировщика — §5.9.
- **API и фронт:** `NotificationResponse.undelivered_channels` (§4.8); `components/notifications/UndeliveredChannels.tsx` (§6.8). Новых `event_type` нет, `EVENT_MAP` не менялся.
- **Тесты** — `tests/test_notification/test_a05_delivery_retry.py` (Telegram недоступен 10 минут → доставка повтором один раз; почта доставлена, Telegram нет → повтор только Telegram; пять неудач → отметка; строка старше окна), заглушки `_send_telegram`/`_send_email`, подставное `now`; `tests/test_notification/conftest.py` — файловая БД + `NullPool` (gotcha-95), `test_delivery_isolation.py` — окно между UPDATE и COMMIT отметки; E2E — `frontend/e2e/a05-notification-undelivered.spec.ts` (моки).
- **Вне блока:** вебхук Telegram (входящие команды) — отложен.
- **Ограничения:** сбой записи «доставлено» после успешной отправки → повтор канала через 10 с (редкий дубль: `gather` без `return_exceptions`); `pending_events` без индекса и чистки — фоновый повтор раз в 10 с читает необработанные строки типа целиком (§3.20); отметка видна при следующей загрузке списка (WS-кадра нет); вживую повтор не проверен — на тестовом стенде Telegram и почта выключены, проверка после выкатки (`deployment_guide.md` §7).

**TelegramNotifier** (`app/notification/telegram.py`):
- Библиотека: `python-telegram-bot` v20+ в webhook-режиме.
- Webhook endpoint: `POST /api/v1/notification/telegram/webhook`.
- Верификация webhook через `X-Telegram-Bot-Api-Secret-Token`.
- HTML parse_mode, severity→emoji маппинг, inline-кнопки для session/trade.
- **Привязка:** `POST /api/v1/notification/telegram/link-token` → токен `secrets.token_urlsafe(9)` (12 символов, TTL 5 мин, in-memory store); `link_store.consume(token, requester)` — лимит неудач 5/15 мин на пользователя Telegram (иначе чат), 100/15 мин глобально, успех не сбрасывает счётчик, память ограничена (S8R-AUDIT-021). Пользователь отправляет `/start <токен>` боту.
- **Webhook handler** (`app/notification/telegram_webhook.py`): команды `/start` (привязка), `/positions`, `/status`, `/help` + callback queries.
  - **Подпись режима (S8R-FIX-117).** Одна на все ответы — `session_mode_label(mode)` по `SESSION_MODE_LABELS` (`paper` → «бумажная», `sandbox` → «песочница», `real` → «реальный»; неизвестный режим — как есть): `/status`, `/positions`, кнопки `/close`, заголовок блока счёта `/balance` («(песочница)» / «(реальный)»). Прежние тернарники знали два режима из трёх: `/status` — песочница как «Paper», `/positions` и `/close` — как «real». `/status` — сессии `active`/`paused` (стратегия, тикер, ТФ, режим), без позиций; позиции — `/positions`. `/balance` — все активные счета с `account_id` параллельно, блок на счёт (Доступно / Заблокировано под заявки / В позициях / Итого, S8R-FIX-084); сбой счёта — `❌ <текст>`, у исключения без текста — «ошибка без текста (<класс>)» (S8R-FIX-119).
- ⚠️ Stack Gotcha 17: `patch.object(bot_instance, ...)` невозможен из-за frozen attrs v20+. Мокировать через `patch.object(type(bot), ...)`.

**EmailNotifier** (`app/notification/email.py`):
- SMTP через `aiosmtplib`, HTML-шаблон, severity-маппинг цветов.
- Используется только для критических событий (circuit breaker, потеря связи, дневная статистика, восстановление).
- Настраивается в `.env`: `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM`.

**InAppNotifier:**
- `create_notification()` создаёт запись в `notifications`.
- EventBus publish → WS канал `notifications:{user_id}`.

**Форматы величин в текстах (2026-10-14):**
- `app/common/money.format_price(Decimal)` — `format(value.normalize(), "f")`: цена без хвоста нулей `Numeric(18, 8)` и без экспоненты («300», «100.5»), значащие знаки не округляются (шаг цены облигаций — до 4–5 знаков). Применяется: ценовой алерт `price_alert` (уровень и текущая цена, S8R-FIX-120), `trade.opened` (`price`) и `trade.filled` (S8R-FIX-125).
- `trade.filled` («Сделка исполнена», `runtime.py`): `price` = `trade.entry_price` (цена исполнения; для sandbox/real — брокера), при `entry_price is None` (`pending`) — `signal.price`. Раньше — всегда `signal.price`, и сообщение расходилось с «Позиция открыта» по той же сделке.
- `trade.closed` — без изменений: тело с `net_pnl` (= `pnl − commission`) и комиссией (`trade_closed_payload`); `pnl` в payload и в журнале — валовой.

**REST API** (`app/notification/router.py`): 6 + 2 endpoints:
- `GET /api/v1/notifications/` — лента с пагинацией (со слэшем; без него 307, S8R-FIX-100; `DELETE /api/v1/notifications/` — удалить все)
- `GET /api/v1/notifications/unread-count` — счётчик
- `PATCH /api/v1/notifications/{id}/read` — пометить прочитанным
- `PUT /api/v1/notifications/read-all` — прочитать все
- `GET /api/v1/notifications/settings` — настройки каналов
- `PUT /api/v1/notifications/settings/{event_type}` — обновить настройку
- `POST /api/v1/notification/telegram/link-token` — генерация токена привязки
- `POST /api/v1/notification/telegram/webhook` — Telegram webhook

**dispatch_external** (`app/notification/dispatchers.py`):
- Реальная реализация: поиск TelegramLink/User.email в БД, вызов `TelegramNotifier.send()` и `EmailNotifier.send()`.
- Возвращает `list[str]` каналов, куда отправлено.

### 5.8 Circuit Breaker

**Конфигурация (S8R-AUDIT-067).** `PUT /circuit-breaker/config` (и `/config/session/{id}`) **заменяет конфиг целиком** — клиент отправляет весь объект. Диапазоны входа (`app/common/risk_limits.py`): проценты 0 < x ≤ 100 (4 знака), суммы 0 < x ≤ 999 999 999 999,99 (2 знака, дробный хвост округляется `quantize_money`), сделок в день 1…10 000, cooldown 0…86 400 с (тот же предел у запуска сессии), целые — strict; окно часов непустое с учётом умолчаний ~~10:00–23:50~~ 07:00–23:50 (с A-03); иначе 422. GET отдаёт `invalid_fields` — сохранённые ранее значения вне диапазонов. Движок читает конфиг и пользователя **одним снимком** (`CBConfigSnapshot`) на `check_before_order`; опасные сохранённые значения (≤ 0 там, где 0 блокирует всё; перевёрнутое окно часов) не применяются: в глобальном — «не задано», в override сессии — из глобального по полю (часы и пара fixed/pct — парой); прочие значения сверх пределов применяются как есть. Часы торговли учитывают override сессии. Несколько глобальных строк (session_id IS NULL) — берётся минимальный id + WARNING `cb_config_duplicate_rows`. `GET /circuit-breaker/status`: `daily_loss_limit` — в рублях (`daily_loss_threshold`), процент — `daily_loss_limit_pct`; при нулевом капитале и процентном лимите — `null`.

**Ответственный: SEC (логика) + BACK1 (интеграция)**

```python
class CircuitBreakerService:
    async def check_before_order(self, session: TradingSession, signal: Signal) -> CheckResult:
        """Вызывается перед каждым ордером. Возвращает ALLOW или BLOCK с причиной."""
        checks = [
            self._check_daily_loss_limit(session),
            self._check_max_drawdown(session),
            self._check_position_size_limit(signal),
            self._check_daily_trade_limit(session),
            self._check_duplicate_instrument(session, signal),
            self._check_cooldown(session),
            self._check_trading_hours(signal),
            self._check_insufficient_funds_streak(session),
        ]
        for check in checks:
            result = await check
            if result.blocked:
                await self._trigger(session, result.reason)
                return result
        return CheckResult(blocked=False)

    async def _trigger(self, session, reason):
        """Ставит стратегию/все стратегии на паузу, отправляет уведомления."""
        ...
```

**Часы торгов и аварийное закрытие (A-03, ТЗ v4.1; ФТ §12.4; ADR 0013).**
- `_check_trading_hours`: для sandbox/real — сначала `trading_hours.market_closed_reason(now, weekend_flag=…)` по фазе таблицы периодов (торги и вечер будней — открыто; аукционы, часы дилера, ночь, праздник — закрыто; торги выходного дня — только с `weekend_flag`); для paper — только аукционы открытия и закрытия (`market_phase(now) in _AUCTION_PHASES` → «Рынок закрыт: …»), прочие часы paper не ограничивают. Затем окно часов пользователя (конфиг CB, override сессии); умолчание — `DEFAULT_TRADING_START/END` = **07:00–23:50** (было 10:00–23:50; зеркало — `src/utils/riskLimits.ts`). Окно шире окна биржи сохраняется как есть (с прежним предупреждением), вход вне торгов для sandbox/real всё равно блокирует правило входа. Блокировки временные (`temporary=True`): сигнал пропускается, сессия остаётся `active`; сессия при этом откладывает вход до начала торгов (§5.4.6).
- `_get_trading_day_start` — `TradingSchedule.trading_day_start(now)`: начало первого биржевого периода сегодняшнего дня МСК (аукцион открытия 06:50), если он уже начался, иначе — того же периода ближайшего предыдущего дня с торгами (≤ 10 дней), иначе полночь МСК. Раньше — жёстко 10:00 MSK (пункт про 10:00 ниже устарел).
- Аварийное закрытие (`_force_close_open_positions`) — `close_position(exit_kind=ExitKind.FORCED)`: в часы дилера проходит как ручное, в аукцион открытия и ночью — `ExitDeferredError`; любой отказ, кроме `OrderInFlightError` (ордер доведёт recovery), → `pending_exits.enqueue(…, ExitKind.FORCED, …)` (§5.4.6). Плоские id сделок снимаются до закрытий (откат сессии делает ORM-объекты expired, gotcha-37). ~~Известный дефект: сбой БД при закрытии одной сделки останавливает остальные закрытия срабатывания (нужен rollback).~~ Снято в A-04 — ниже.
- G01 — свежесть цены D для нереализованного P&L: дневка свежая, если с её дня (дата МСК метки) до сегодня прошло не больше `PRICE_FRESHNESS_BARS` = 3 торговых будней по таблице (`trading_days_between`), а не календарных суток: раньше в понедельник и после праздника P&L сессии D в CB не учитывался.

**«Стоп всему», лимиты сессии, причина паузы, отчёт закрытия (A-04, ТЗ v4.7; ФТ §12.4; ADR 0040–0042).**
- `check_before_order` — первой проверкой под per-user локом: `is_trading_stopped` → `TRADING_STOPPED_EVENT` (`trading_stopped`), временный отказ (§5.4.7).
- **Снимок конфигов** `CBConfigSnapshot(global_config, session_config, user)`: `global_config` — опасные значения сняты (S8R-AUDIT-067); `session_config` — `own_config`: только заданные и не опасные поля строки сессии. `effective` — `merge_configs` (поле сессии, иначе общее; пары «сумма / % убытка» и часы — парой; ~~строка сессии целиком~~ — так было до A-04); `own(name)` — своё поле или `None`; `general(name)` → `effective_config.general_limit`: глобальный → профиль (5 % / 15 % / 25 % капитала) → умолчание часов 07:00–23:50 (ADR 0041).
- `_check_daily_loss_limit`, `_check_max_drawdown`, `_check_daily_trade_limit`: сначала общий лимит (`general`) с охватом по `CB_PAUSE_SCOPE`; затем `own(...)`, если задано, — по этой сессии (её убыток и капитал, её сделки), `scope = own`, `session_limit=True`. `_check_position_size_limit`, `_check_cooldown`, `_check_trading_hours` — по `effective`, `session_limit` = поле своё.
- `_trigger`: событие с `session_limit`; причина `cb_pause_reason(event_type, session_limit=)` (`'session:<тип>'` у своего лимита) тем же условным UPDATE, что пауза; связи `circuit_breaker_event_sessions` по `affected_session_ids` тем же commit; после commit — `cb.triggered` в `trades:{sid}` (уведомление, outbox), `session.paused` с `pause_reason` в `trades:{paused}`, копия `cb.triggered` в `system:{user_id}`. После закрытия — `build_close_report(db, outcomes)` → `close_report`, commit, `publish_close_report` (`cb.close_report` в `system:{user_id}`); сбой записи — WARNING `cb_close_report_save_failed`, пауза и закрытия не откатываются.
- `_force_close_open_positions(session_ids, reason)` (R20): плоские id `filled`-сделок; каждая — своя `AsyncSession` фабрики `_own_sessions()`, `OrderManager.close_position(exit_kind=FORCED)`; сбой — `rollback` своей сессии, `pending_exits.enqueue(FORCED)` (кроме `OrderInFlightError`), `CloseOutcome("not_closed", close_failure_reason(exc))`, WARNING `cb_force_close_failed`; остальные закрываются; последовательно. `close_failure_reason`: `StatementError` → «ошибка базы данных: <текст драйвера>» (раньше «[]»); правило одно для CB, «Стоп всему» и «Закрыть все».
- Подсказки «как в общих» — `get_inherited_values(db, user_id)` / `inherited_values(configs)` из того же `general_limit`, что проверки. Сервис — `get_session_config`, `delete_session_config`, `get_inherited`, `get_history` (§4.16), `publish_limits_changed`.
- **Известные ограничения:** ошибки SQLAlchemy вне `StatementError` (`PendingRollbackError`) — причина английским текстом; у CB сбой чтения пометки «в списке» теряет весь отчёт, сделка без сессии выпадает из него (`build_close_report`); срабатывание, никого не поставившее на паузу, ничего не закрывает — в истории без N из M; дневной убыток разных классов за 5 минут — одной строкой с охватом первого события; у событий до W8j (до 15.05.2026) `action` — `pause_close`, хотя закрытий не было; фильтр по сессии с переиспользованным id показывает и события прежней сессии (без тикера, ADR 0045).

**Расчёт дневных убытков:**
- S8R-FIX-063 ч.2: Расчёт дневных убытков»: «+ Σ min(оценка, 0) сделок „P&L не определён“ (`unpriced_cb_losses`). Цены: вход — факт → `entry_signal_price` → close последнего закрытого бара ≤ `opened_at` − ТФ; выход — факт → `exit_signal_price` → бар ≤ `closed_at` − ТФ. Давность бара: ≤ 24 ч для внутридневных ТФ, ≤ 7 суток для D и выше; ISS не берётся. Сдвиг `CB_UNPRICED_ADVERSE_PCT` = 1 % против позиции, количество — `position_quantity`. Комиссия не вычитается — как у валового `pnl`. Собственный лимит сессии — доля из классового расчёта (`pnl_by_session`). Пометка причины — „включая оценку N сделок без подтверждённой цены“». «Max drawdown» (sandbox/real): «equity включает оценённый убыток таких сделок всех дней
- Агрегация `SUM(pnl)` по `live_trades` за текущий торговый день. Начало торгового дня определяется по рынку инструмента: фондовый/долговой — 10:00 MSK, валютный — 10:00 MSK (несмотря на продлённую сессию до 23:50). Сброс дневных счётчиков — в 10:00 MSK следующего торгового дня по календарю MOEX. *С A-03 — начало первого биржевого периода дня по таблице периодов (`trading_day_start`, см. выше), одно для всех бумаг.*
- Плюс unrealized P&L открытых позиций (текущая цена - цена входа).
- Сравнение с `user.risk_daily_loss_pct * account_balance` или `user.risk_daily_loss_fixed`.

**Max drawdown:**
- `peak_equity` обновляется при каждом закрытии позиции **и на каждой закрытой свече** — `CircuitBreakerEngine.update_peak_on_bar(session, close_price)` из `SessionRuntime._handle_candle` (S8R-FIX-013). Раньше пик поднимался только когда дело доходило до проверок входа: между сигналами цена могла сходить вверх и вернуться, просадка считалась от старого пика, и лимит срабатывал позже, чем настроен (при редких сигналах мог не сработать вовсе). Вызов без сети (цена из свечи) и **без per-user CB-лока**: лок держится на горячем пути «сигнал → ордер», а пик только растёт — потерянное обновление исправляет следующий бар.
- `current_drawdown = (peak_equity - current_equity) / peak_equity * 100`.
- **Состав `current_equity` одинаков во всех режимах (S8R-FIX-015).** Песочница и реальный счёт — equity по сделкам сессии плюс открытые позиции по свежей цене (`broker_equity_by_session`, S8R-AUDIT-068). Бумажный режим с 2026-10-13 — `paper_equity_by_session`: `balance + blocked_amount + unrealized`, где нереализованная часть приходит из того же `unrealized_pnl_by_session` и с той же проверкой свежести цены (живая свеча сигнала, иначе свежий бар кэша, S8R-AUDIT-070); свежей цены нет — открытая часть не учитывается вовсе. До этого paper считался только по реализованному P&L: позиция в минусе лимит не трогала до закрытия, то есть защита включалась ПОСЛЕ убытка, а для шортов расхождение с sandbox/real было заметнее.
- **Пик и статус считаются ТЕМ ЖЕ составом** (находка `/code-review` к S8R-FIX-015). `paper_equity_by_session` — единственный источник бумажного equity: его используют проверка просадки, `update_peak_on_bar` и блок «текущее состояние» над формой риск-менеджмента (`CircuitBreakerService.get_status`). Иначе состав расходился: денежный учёт `PaperTradingEngine` поднимал `paper_portfolios.peak_equity` только по `balance + blocked_amount`, и прибыль открытой позиции опускала просадку, но пика не поднимала — когда та же прибыль уходила, просадка считалась от величины, которой капитал уже не был (лимит снова срабатывал позже); статус же показывал 0 % там, где движок видел просадку. `paper_peak_equity(portfolio, equity)` — симметричный аналог `broker_peak_equity`.
- При `current_drawdown >= user.risk_max_drawdown_pct` -> trigger.

**Нехватка средств (insufficient funds streak):**
- Поле `trading_sessions.consecutive_fund_skips` (Integer, DEFAULT 0).
- При пропуске сигнала из-за нехватки средств — инкремент `consecutive_fund_skips`.
- При успешном размещении ордера — сброс в 0.
- При `consecutive_fund_skips >= 3` — автоматическая пауза стратегии + уведомление во все каналы.

### 5.9 Scheduler

**Ответственный: BACK1**

Реализация на `APScheduler` (AsyncIOScheduler). Jobs:

| Job | Расписание | Описание |
|---|---|---|
| `sync_moex_calendar` | Ежедневно 00:05 MSK | Загрузка торгового календаря из MOEX ISS (S8R-AUDIT-093: `dailytable`) |
| `update_moex_sessions` | Ежедневно 06:00 MSK | Обновление расписания торговых сессий |
| `daily_backup` | Ежедневно 01:00 MSK | Бэкап SQLite (через `sqlite3 .backup`) |
| `cleanup_revoked_tokens` | Каждые 24 ч от старта + сразу при старте (S8R-AUDIT-043) | Удаление строк `revoked_tokens` с `expires_at < now`, идемпотентно |
| `send_daily_stats` | Ежедневно 19:00 MSK (после закрытия фондового) | Дневная статистика в Telegram и email |
| `refresh_instrument_info` | Еженедельно вс 12:00 | Обновление данных об инструментах (лотность, шаг цены) |
| `detect_corporate_actions` | Каждые 6 часов | Проверка дивидендов/купонов/сплитов через MOEX ISS (S8R-FIX-008: было описано «ежедневно 08:00 MSK», имя задачи и период приведены к коду — `IntervalTrigger(hours=6)`). Тикеры — `CorporateActionService.detection_tickers()`: сессии `active`/`paused` и сессии любого статуса со сделками в начисляемых статусах (S8R-FIX-129). Тикеров нет — `detect_corporate_actions([])` (A-01, дефект 4): синхронизируются только сплиты (`splits.json`, все бумаги) — ряды свечей читаются с их учётом; раньше без сессий детект не шёл |
| `reset_ai_daily_usage` | Ежедневно 00:00 MSK | Сброс `usage_today` в `ai_provider_configs` |
| `reset_ai_monthly_usage` | 1-е число месяца 00:00 MSK | Сброс `usage_month` в `ai_provider_configs` |
| `sync_trading_schedule` | Ежедневно 00:10 MSK + сразу при старте (`next_run_time`) | A-03: таблица периодов из T-Invest `TradingSchedules` на сегодня + 7 дней (`trading_schedule.sync_from_tinvest`, §5.9.1). Рестарт не оставляет процесс на вчерашней версии |
| `refresh_weekend_trading_schedule` | Ежечасно 01:00–23:00 MSK (`CronTrigger(minute=0, hour="1-23")`) | A-03: в субботу, воскресенье и праздник (не торговый день календаря) — `sync_trading_schedule`; в торговый будень сразу выход. Расписание выходного дня меняется в тот же день (25.10.2026 биржу отменили за день) |
| `retry_notification_delivery` | Каждые 10 с (`IntervalTrigger(seconds=10)`, `misfire_grace_time=600`, `max_instances=1`; без `next_run_time` — повтор при старте делает lifespan) | A-05 (ТЗ v4.2): `SchedulerService.retry_notification_delivery` → `NotificationService.retry_due_deliveries()` — повтор недоставленных Telegram/почты по сроку в строке `notification.dispatch` и отметка «не доставлено» (§5.7); без сервиса уведомлений — no-op, сбой прохода — warning `notification_retry_pass_failed`, взятые строки — лог `notification_retry_pass rows=N`. Вставка — между `refresh_backtest_jobs_rate` и `cleanup_revoked_tokens` |
| `backfill_trading_schedule_iss` | Сразу при старте + раз в сутки (`IntervalTrigger(days=1)`) | A-03: история таблицы периодов из минут ISS SBER с 13.06.2018 (`backfill_from_iss`); заполненные и отмеченные пустыми даты пропускает. Нет ISS-клиента у календаря — warning `trading_schedule_iss_backfill_no_client` |
| `history_backfill` | Ежедневно 04:00 MSK (`CronTrigger(hour=4, minute=0, timezone="Europe/Moscow")`) + сразу при старте (`next_run_time`) | A-02 (ТЗ v4.5): `backfill.run_history_backfill` — вселенная бумаг в очередь единственного исполнителя, T-Invest — счётом для рыночных данных, ISS до шва (§5.6.8). Прогон долгий (первый на рабочей БД — ≈ 2 ч); повторный запуск только ставит бумаги в очередь идущего исполнителя. Ошибка — `history_backfill_job_failed`. |
| `load_tinvest_dividends` | Ежедневно 04:30 MSK | A-02: `dividends.service.load_tinvest_dividends` — `GetDividends` по акциям TQBR (`SHARE_SECTYPES`) счётом для рыночных данных, пауза 0,8 с между бумагами, отказ по лимиту — ожидание и повтор (бюджет на прогон); без счёта — `market_data_account_missing`, загрузка не идёт (§5.15). Ошибка — `load_tinvest_dividends_failed`. |
| `check_smartlab_dividends` | Ежедневно 05:00 MSK | A-02: `dividends.service.check_smartlab` — акции TQBR (`SHARE_SECTYPES`), не проверявшиеся 7 суток, не больше `ceil(N/7)` за прогон (38 из 261), пауза 5 с между страницами (§5.15). Ошибка — `check_smartlab_dividends_failed`. |
| `news_collect` | Каждые 60 с (`IntervalTrigger(seconds=60)`, `misfire_grace_time=600`, `max_instances=1`, `coalesce`; без `next_run_time` — первый проход через минуту после старта) | Б-01 (ТЗ v4.4): `NewsService.run_collect_cycle` — опрос включённых `news_sources` и запись новых новостей (§5.14.1), затем разметка словарём новостей без разметки — и после неудачного сбора (§5.14.2). Клиенты HTTP закрываются в `SchedulerService.shutdown()`. С Б-07 (ТЗ v4.9; задание и его параметры не менялись): в начале прохода — чтение `news_settings`, «Загрузка» выключена — выход до сети; после словаря — шаг ИИ-разметки задачей (`ai_labeling.run_ai_step`) и шаг алертов по новостям (`alerts.run_alert_step`) — по сроку ~10 с и ещё раз после ответа модели (§5.16) |
| `news_reactions` | Каждые 60 с (параметры — как у `news_collect`) | Б-01: `NewsService.run_reaction_cycle` — заводит реакции для новых пар «новость — бумага» и считает до 20 с наступившим сроком по 1m ISS (§5.14.3). Отдельное задание — запросы свечей не задерживают сбор новостей. С Б-07: в начале прохода — главный выключатель «Загрузка и обработка новостей», выключен — выход до запросов свечей (§5.16.6) |
| `screener_nightly` | Ежедневно 01:30 MSK (`CronTrigger(hour=1, minute=30, timezone="Europe/Moscow")`) + через 60 с после старта (`next_run_time`) | Б-03 (ТЗ v4.11): `SchedulerService.screener_nightly` → `screener.jobs.run_screener_pass` (первый вызов процесса — `after_start=True`): статистика из БД, ожидание прохода `history_backfill`, дозагрузка итогов дня ISS, синхронизация справочника вселенной (после старта — только если последней не было или она старше суток), первое заполнение истории, статистика (§5.18.1). 01:30 — итоги дня готовы после вечерней сессии, вне окон 04:00 / 04:30 / 05:00. Ошибка — `screener_nightly_failed`. |

С A-02 `SchedulerService.shutdown()` после остановки планировщика гасит исполнителя фоновой догрузки (`backfill.stop_history_backfill()`), чтобы остановка процесса не ждала часового прогона.

Б-07 (ТЗ v4.9) новых заданий не добавил, `app/scheduler/*` не менялся: ИИ-разметка, проверка алертов по новостям и выключатели живут в проходах `news_collect` / `news_reactions`; проверка модели на эталоне — фоновая задача процесса (`asyncio`) по `POST /api/v1/news/ai-benchmark`, не задание планировщика (§5.16.3).

Б-03 (ТЗ v4.11) добавил одно задание — `screener_nightly` (строка выше): `app/scheduler/service.py` — регистрация и метод-обёртка; ISS-часть прохода сама ждёт конца прохода `history_backfill` (`iss_gate.wait_for_history_backfill`), поэтому после старта синхронизация справочника идёт только после фона A-02 (на копии рабочей БД — ≈ 2 ч).

Суммы дневной статистики (`real PnL`, `unreal` по сессиям и итог) форматирует `_signed_rub` (`app/scheduler/service.py`, S8R-FIX-121): `quantize_money` (копейки, половина — вверх), знак, пробел между тысячами; ноль после округления — «0.00 ₽» без знака. Прежний `{:+,.0f}` округлял до рубля и давал «-0 ₽» для −0,04.

**MOEX Calendar Service:**

```python
class MOEXCalendarService:
    async def is_trading_day(self, date: date, market: str = 'stock') -> bool: ...
    async def get_session_times(self, market: str) -> SessionTimes: ...
    async def is_trading_hours(self, dt: datetime, market: str) -> bool: ...
    async def next_trading_day(self, date: date) -> date: ...
```

Fallback-значения (если MOEX ISS недоступен) захардкожены в config.

**Фактическая реализация (S8R-AUDIT-093, 2026-09-24):** `app/scheduler/moex_calendar.py`.
- Источник — `GET /iss/engines/stock.json?iss.only=dailytable&iss.meta=off` (`MOEXISSClient.get_trading_calendar`), колонки `date`, `is_work_day` — **точечные исключения** биржи: праздник в будни → неторговый, рабочий день по переносу (в т. ч. суббота) → торговый. Прежний ресурс `…/markets/shares/dates.json` календаря не содержит (ISS отвечает 200 описанием рынка, gotcha-70).
- Решение «торговый день»: исключение из `dailytable` → иначе недельное правило Пн–Пт. **Суббота и воскресенье — неторговые** (прежняя политика терминала; `timetable` ISS, где с 2025 все 7 дней рабочие из-за торгов выходного дня, не используется — открытый вопрос заказчику). Статичный `RUSSIAN_HOLIDAYS` — только fallback, пока данные ISS не загружены.
- Методы синхронные: `is_trading_day(d)`, `get_market_status()`, `time_until_close()`; `async load_calendar() -> bool` (таймаут 15 с; сбой, пустой ответ, ответ без блока — warning, ранее загруженные данные сохраняются, старт не блокируется; успех — лог `moex_calendar_loaded count=N`).
- **Один экземпляр на процесс:** lifespan создаёт `MOEXCalendarService(iss_client=MOEXISSClient(settings.MOEX_ISS_BASE_URL))`, регистрирует `set_calendar_service()`, кладёт в `app.state.calendar_service`, грузит на старте; scheduler (DI), `market_data/router`, мультиплексор T-Invest читают `get_calendar_service()`. Часы сессий и проверки CB/pre-check (карточка S8R-AUDIT-072) этим не менялись.
- **С A-03 (ТЗ v4.1)** календарь отвечает только «торговый ли день» (`is_trading_day`, `is_weekend_session_day`); часов торгов он не хранит. `get_market_status(now=None)` и `time_until_close()` — по фазе таблицы периодов (§5.9.1): `auction` (аукцион открытия), `online` (торги будня), `weekend_session` (торги в не торговый день календаря), `closing_auction`, `evening`, `dealer`, вне периодов — `weekend` в не торговый день, иначе `offline`; `time_until_close` — до конца текущего биржевого периода.
- **С PR #140 (2026-11-17, мерж-коммит `ede13c3`, ADR 0110; миграций нет).** `is_trading_day` и `is_weekend_session_day` сначала спрашивают таблицу периодов: `TradingSchedule.stored_trading_day(d)` — день со строками → `not weekend`, отметка «ISS: минут нет» (`trading_schedule_empty_days`) → `False`, иначе `None`; `stored_weekend_session_day(d)` — выходной день с периодом торгов (в том числе праздник в будни) → `True`. При `None` — `dailytable` и Пн–Пт, без ISS — `RUSSIAN_HOLIDAYS`, как раньше. `DaySchedule.weekend` — по периодам самого дня (`trading_schedule.trading_day_by_periods(day, kinds)`: есть `evening` → торговый; биржевые периоды раньше `WEEKEND_SESSIONS_SINCE` = 01.03.2025 → торговый; аукцион закрытия в Пн–Пт → торговый; иначе — нет); `load`, `sync_from_tinvest` (`day_from_tinvest`) и `day_from_iss_minutes` календарь больше не спрашивают. Отметки «ISS: минут нет» — в кэше процесса: `TradingSchedule.load` читает `trading_schedule_empty_days`, `backfill_from_iss` после каждой пачки передаёт новые (`mark_empty`). Круга «календарь ↔ расписание» нет: `stored_*` запасного дня не строят, `fallback_day` спрашивает календарь только для даты без строки. Потребители (`last_buy_day`, `dividend_last_buy_date`, `accrual_last_buy_date`, правила входа и выхода, статус рынка, обновление расписания в выходные, выбор дней архива, проверка пропусков) не менялись. Скрипт эталона B14 (`record_b14_etalon.py::table_trading_day`) остаётся со своим правилом: без сети у него нет ISS.

**5.9.1 Таблица периодов расписания — `app/scheduler/trading_schedule.py`, `app/scheduler/models.py` (блок A-03, ТЗ v4.1; ADR 0012).** Требования — ФТ §1.6. Единственный источник часов: правила входа и выхода (`common/trading_hours.py`, §5.4.6), «бар закрыт» (`market_data/bar_close.py`, §5.6.6), Circuit Breaker (§5.8), бэктест (`AuctionAwareBroker`, §5.3) и paper, статус рынка. Модуль — лист: из `app.*` импортирует только календарь и зону МСК, БД и сеть — только в `async`-функциях загрузки (зависимости импортируются лениво).

```
 T-Invest TradingSchedules ─┐
 MOEX ISS минуты SBER (фон) ┴▶ trading_schedule_periods ─▶ TradingSchedule (кэш процесса) ─ get_trading_schedule() ─┐
     ┌───────────────────────────────────────────────────────────────────────────────────────────────────────────┘
     ▼
 trading_hours (вход · выход по виду) · bar_close · Circuit Breaker · бэктест и paper · статус рынка · линейка графика (`/market-data/trading-hours`, Б-04)
```

- **Модель:** `PeriodKind` — `opening_auction`, `trading`, `closing_auction`, `evening`, `dealer`; `EXCHANGE_KINDS` — все, кроме `dealer`. `MarketPhase` — те же плюс `closed`. `SchedulePeriod(kind, start, end)` — наивное UTC, конец исключён. `DaySchedule(day, periods, source: "tinvest"|"iss"|"fallback", weekend)`: `phase_at(t)`, `exchange_open` (начало первого биржевого периода), `trading_start` (начало первого `trading`), `trading_end` (конец последнего биржевого), `periods_of(kind)`; `weekend = not calendar.is_trading_day(d)` — торги такого дня (Сб/Вс, праздник с торгами) — торги выходного дня, только для бумаг с `weekend_flag`. Таблицы — §3.24.
- **Доступ:** `get_trading_schedule()` — синглтон процесса (`set_trading_schedule`, `reset_trading_schedule` для тестов); `TradingSchedule.day(d)`, `phase_at(t)`, `is_opening_auction_bar(tf, bar_start)` (бар внутридневного ТФ целиком до `trading_start` дня и пересекает аукцион открытия: 1h 06:00, 4h 03:00, в выходной 1h 09:00), `is_closing_auction_bar(tf, bar_start)` (начало бара в аукционе закрытия: 5m 18:55), `next_trading_start(moment, *, weekend_flag)` (начало ближайшего `trading` строго после момента; дни `weekend` — только при `weekend_flag is True`; горизонт 10 дней), `trading_day_start(moment)` (для дневных лимитов CB), `trading_days_between(a, b)` (торговые будни в `(a, b]` — свежесть цены D в CB). Чтение синхронное, без БД и сети: его зовут горячие пути.
- **Загрузка:** lifespan создаёт `TradingSchedule(calendar=calendar_service)`, регистрирует (`set_trading_schedule`, `app.state.trading_schedule`) и `load()`-ит из БД **до** планировщика и `restore_all`; сбой — warning `trading_schedule_startup_load_failed`, работа на запасных константах. После каждой синхронизации — `replace_days`.
- **Запасной день** `fallback_day(d, *, calendar)` — дата без строки (время МСК, конец исключён): торговый будень (в том числе рабочая суббота) — 06:50–07:00 / 07:00–18:55 / 18:55–18:59:31 / 19:00:01–23:50; Сб/Вс с торгами выходного дня (`is_weekend_session_day`) — дилер 02:00–09:40:01, аукцион 09:50–10:00, торги 10:00–19:00:01, дилер 19:00:01–23:50; Сб/Вс без торгов — дилер 02:00–23:50:01; будничный праздник — периодов нет. Warning `trading_schedule_fallback` раз в сутки; кэш запасных дней — до 4096 дат. Календарь сломан — `_SafeCalendar` в `trading_hours` (Пн–Пт без торгов выходного дня).
- **T-Invest** — `sync_from_tinvest(session_factory)`: токен — счёт рыночных данных первого активного пользователя (по `users.id`) со счётом T-Invest (`select_market_data_account`, ключ расшифровывается в памяти); `TInvestAdapter.get_trading_schedules(SCHEDULE_EXCHANGE = "moex_mrng_evng_e_wknd_dlr", today, today + 7)`; имя площадки в ответе сравнивается без регистра (`bb8799e`: живой T-Invest отвечает `MOEX_MRNG_EVNG_E_WKND_DLR`, и до исправления стенд работал на запасных часах); `day_from_tinvest` раскладывает интервалы в периоды (конец включителен до секунды — +1 с; обобщённые дубли типов интервалов второго периода не дают, смежные периоды одного вида сливаются — `_merge`); запись `save_days(…, keep_tinvest=False)` через `persist_with_retry_fresh` — строки T-Invest замещают любые строки своих дат; лог `trading_schedule_synced` (`days`, `first`, `last`). Нет счёта (`trading_schedule_sync_no_account`), сбой T-Invest (`trading_schedule_sync_failed`), нет площадки или дней — warning, таблица не меняется.
- **ISS** — `backfill_from_iss(session_factory, iss_client, *, since=SCHEDULE_HISTORY_SINCE (2018-06-13), until=вчера)`: минуты SBER TQBR (`SCHEDULE_ISS_TICKER`) по дням от поздних к ранним, пауза `ISS_BACKFILL_PAUSE_SEC` = 0,2 с, запись пачками по 50 дней (`keep_tinvest=True` — строки T-Invest не трогаются) и кэш после каждой пачки. Пропускаются даты с любыми строками, будничные праздники, Сб/Вс без торгов выходного дня по календарю и даты из `trading_schedule_empty_days` (ISS ответил без минут — отметка пишется вместе с пачкой и переживает перезапуск). `day_from_iss_minutes`: аукцион открытия — первая минута вида xx:59, за которой без разрыва идут торги (период xx:50–xx+1:00), начало торгов — следующая минута (иначе — первая минута); аукцион закрытия — от минуты после первого разрыва ≥ 2 мин между 18:30 и 19:10 до минуты после второго такого разрыва, или до первой минуты ≥ 19:00, или до конца дня; вечер — от первой минуты ≥ 19:00 после аукциона закрытия до последней минуты + 1 мин. Часов дилера ISS не знает. Сбой дня — warning `trading_schedule_iss_day_failed`, день не пишется и не отмечается (возьмётся следующим проходом); итог — `trading_schedule_iss_backfill_done`. Замер на копии рабочей БД (2026-10-26, остановлен по сроку 40 мин): 1 997 дней запрошено, 0,83 дня/с, записано 1 500 дней (2021-05-12…2026-10-23, 5 615 строк), сбоев 0; полный первый проход 13.06.2018 → вчера — ≈ 2 900 запросов, ≈ 58–60 мин фоном (`deployment_guide.md` §7).
- **Тесты и фикстуры:** `tests/unit/test_scheduler/` (таблица, загрузка, статус рынка; загрузчики фикстур — `schedule_fixtures.py`), фикстуры — `tests/fixtures/trading_schedule/` (настоящие ответы T-Invest 20–25.10.2026, в том числе расписание 25.10 до и после перемены, и минуты ISS SBER за семь дат; `meta.json`), запись — `backend/scripts/record_schedule_fixtures.py` (не тесты); состояние процесса сбрасывает `tests/conftest.py`.
- **Ограничения:** расписание одно на все бумаги (выведено по минутам SBER; правила разрыва минут проверены на фикстурах за семь дат — на прочих днях возможна ошибка); ошибка таблицы — вход и выход не в тот час и неверное время закрытия бара; если расписания T-Invest на дату нет, праздничная сессия (23.02, 12.06, 04.11) считается неторговой — `fallback_day` будничного праздника периодов не даёт, ISS-история праздники пропускает; после перезапуска без T-Invest день, на который T-Invest ответил «торгов нет», до следующей синхронизации получает запасные часы; состояние — в памяти одного процесса (один uvicorn-worker).

### 5.10 AI Service

**Ответственный: BACK2 (AI-интеграция)**

**5.10.1 Pluggable Provider Architecture:**

```python
class BaseLLMProvider(ABC):
    @abstractmethod
    async def chat(self, messages: list[Message], system_prompt: str) -> LLMResponse: ...

    @abstractmethod
    async def stream_chat(self, messages: list[Message], system_prompt: str) -> AsyncGenerator[str, None]: ...

    @abstractmethod
    def count_tokens(self, text: str) -> int: ...

class ClaudeProvider(BaseLLMProvider):
    def __init__(self, api_key: str, model: str = "claude-opus-5-5"):
        ...

class OpenAIProvider(BaseLLMProvider):
    def __init__(self, api_key: str, model: str = "gpt-4o"):
        ...
```

**Конфигурация AI — двойной источник (БД + .env):**

Приоритет: **БД > .env**. Если в БД есть активный `ai_provider_configs` с `is_active=True` — используется он. Если нет — fallback на переменные `.env`. Это позволяет:
- Настроить AI-провайдера через UI (основной сценарий для пользователя)
- Использовать `.env` как fallback для автоматизированных/серверных сценариев

**Хранение API-ключей в БД:**
- Ключ шифруется тем же `CryptoService` (AES-256-GCM), что используется для брокерских ключей.
- В UI ключ отображается маскированным (первые 4 + последние 4 символа).
- При добавлении нового провайдера — рекомендация выполнить verify (тестовый запрос).
- Сброс `usage_today` — ежедневно через scheduler (новый job `reset_ai_daily_usage`). Сброс `usage_month` — первого числа каждого месяца.

**Логика выбора провайдера при запросе к AI:**
```python
async def get_active_provider(self, user_id: int) -> BaseLLMProvider:
    # 1. Проверить БД: ai_provider_configs WHERE user_id=X AND is_active=True
    db_config = await self.repo.get_active(user_id)
    if db_config:
        api_key = self.crypto.decrypt(db_config.encrypted_api_key, db_config.encryption_iv)
        return self.create_provider(db_config.provider_type, api_key, db_config.model_id, db_config.api_base_url)
    # 2. Fallback на .env
    if settings.AI_API_KEY:
        return self.create_provider(settings.AI_PROVIDER, settings.AI_API_KEY, settings.AI_MODEL)
    # 3. AI недоступен
    raise AIProviderNotConfiguredError("AI-провайдер не настроен. Настройте в разделе Настройки → AI.")
```

**Fallback-конфигурация в `.env`:**
```
AI_PROVIDER=claude            # 'claude' | 'openai' — используется только если в БД нет активного провайдера
AI_API_KEY=                   # Используется только если в БД нет активного провайдера
AI_MODEL=claude-opus-5-5     # дефолт кода (S8R-FIX-033, решение заказчика); AI_DAILY/MONTHLY_LIMIT удалены
```

**Поддерживаемые провайдеры:**

| provider_type | API | Примечание |
|---|---|---|
| `claude` | Anthropic Messages API | Модели Claude Sonnet / Opus |
| `openai` | OpenAI Chat Completions API | Модели GPT-4o / GPT-4.1 |
| `deepseek` | OpenAI-совместимый | deepseek-chat, deepseek-reasoner. Base URL: `https://api.deepseek.com/v1` |
| `gemini` | OpenAI-совместимый | Gemini 2.5-pro, 2.5-flash. Base URL: `https://generativelanguage.googleapis.com/v1beta/openai` |
| `mistral` | OpenAI-совместимый | Mistral large, medium, small. Base URL: `https://api.mistral.ai/v1` |
| `groq` | OpenAI-совместимый | Llama 3.3-70b, Mixtral 8x7b. Base URL: `https://api.groq.com/openai/v1` |
| `qwen` | OpenAI-совместимый | Qwen plus, turbo, max. Base URL: `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| `openrouter` | OpenAI-совместимый | Любая модель через OpenRouter. Base URL: `https://openrouter.ai/api/v1` |
| `custom` | OpenAI-совместимый API | Для локальных LLM (Ollama, LM Studio и др.). Указывается `api_base_url` |

**5.10.2 Системный промпт:**

```
Ты — AI-ассистент в торговом терминале для рынка MOEX. Помогаешь трейдерам создавать и оптимизировать торговые стратегии.

КОНТЕКСТ РЫНКА:
- Московская биржа (MOEX): акции, облигации, ETF/БПИФ, валютные пары
- Основная сессия фондового рынка: 10:00–18:40 MSK
- Лотность: акции торгуются лотами (например, SBER = 10 акций в лоте)
- Валюта: рубль (₽)
- Позиции long и short (направление входа задаёт стратегия), без плеча.
- Фьючерсы не поддерживаются.

ДОСТУПНЫЕ ИНДИКАТОРЫ:
SMA, EMA, Bollinger Bands, RSI, MACD, Stochastic, ATR, Volume MA и другие индикаторы Backtrader.

ТВОЯ ЗАДАЧА:
1. Помочь пользователю сформулировать торговую стратегию.
2. Задать уточняющие вопросы для полноты описания.
3. Предложить блочную схему стратегии в формате JSON.
4. Объяснить преимущества и риски предложенных решений.

ФОРМАТ БЛОЧНОЙ СХЕМЫ:
{JSON schema описание}

Отвечай на русском языке. Будь конкретен и прагматичен.
```

**5.10.3 Контекстное сжатие:**

При превышении 80% лимита контекстного окна модели:
1. Суммаризировать предыдущие сообщения в один блок.
2. Сохранить последние 3 сообщения и финальное решение.
3. Уведомить пользователя.

**5.10.4 Rate Limiting / Budget:**

`S8R-AUDIT-014`: таблица `ai_usage_daily(date PK, prompt_tokens, completion_tokens)` — расход серверного `AI_API_KEY` на инстанс (миграция `1d92db59f28d`); `AI_GLOBAL_DAILY_TOKEN_LIMIT`/`AI_GLOBAL_MONTHLY_TOKEN_LIMIT` (0 = выкл.); `check_budget` — per-user + глобальный потолок при серверном ключе (429), `increment_usage` — атомарный upsert и при fallback, per-user — атомарный UPDATE; `/ai/explain` учитывается; `/ai/status` не раскрывает provider/model серверного ключа. Потолок мягкий (проверка до запроса).

`AIBudgetTracker` хранит количество использованных запросов в `daily_stats` (или in-memory counter, который flush-ится в БД). При достижении лимита -> возврат ошибки, UI переключается на режим B.

**5.10.5 Фоновые задачи (v4.9, Б-07).** Вне чата модель вызывают ИИ-разметка новостей и проверка на эталоне — через `app/ai/background.py` (§5.16.1): провайдер — строка первого администратора с `is_background` (иначе его активная), ключ `.env` и `get_active_provider(user_id)` не используются; у провайдеров — метод `complete_json` (ответ по схеме JSON, таймаут, `max_retries=0`); учёт — `ai_background_usage` по суткам МСК, лимит — `ai_background_settings.daily_token_limit`. Чат, `ai_usage_daily`, `AI_GLOBAL_*` и счётчики строк провайдера (`*_tokens_used`) фоновые задачи не трогают.

### 5.11 Code Sandbox

**Лимиты исполнения (S8R-AUDIT-006):** компиляция стратегии и `cerebro.run` бэктеста — в spawn-процессе (`app/backtest/exec_limits.py`) с дедлайном `BACKTEST_EXEC_TIMEOUT_SEC` (SIGKILL, статус `failed` с причиной), отмена бэктеста убивает процесс (gotcha-83); комбинация Grid Search — с дедлайном от старта в воркере, зависший воркер — ошибка комбинации, Pool пересоздаётся; лимит памяти RLIMIT_AS (`BACKTEST_EXEC_MEMORY_LIMIT_MB` сверх базы процесса) — только Linux, на macOS не ставится; мониторинг памяти через psutil (эскиз §5.11.2) не реализован. `app/sandbox/executor.py` удалён (S8R-AUDIT-019): код стратегии компилируется только в `BacktestEngine._compile_strategy` из IR.

**Ответственный: SEC**

> **Фактическая модель после S8R-AUDIT-001 (2026-09-24, решение заказчика Q4-001 = «а») — приоритет над эскизами 5.11.1–5.11.2 ниже.**
>
> 1. **Исполняется только код, порождённый кодогеном из IR валидированных блоков.** `runtime_backtrader_code(blocks_json)` (`strategy/ir_codegen.py`) больше не принимает сохранённый `generated_code` как fallback: пустые, вырожденные (нет ни входа, ни выхода), битые блоки или блоки неизвестного типа → `StrategyNotExecutableError` (наследник `ValidationError`, HTTP 422, «Версия стратегии не содержит блоков: исполнение произвольного кода отключено, пересоберите стратегию в конструкторе»). Единая проверка — `ir.parse_executable_blocks` (allow-list типов и полей блоков `validate_blocks_json` → IR); её вызывают старт бэктеста, rerun, фоновый запуск, Grid Search, старт и возобновление торговой сессии, live-маршрутизация свечей. Поле `generated_code` хранится (превью в редакторе) и нигде не исполняется.
> 2. **Live-торговля — только IR-интерпретатор** (`evaluator.evaluate`). Legacy-путь (`_get_strategy_code` → `CodeSandbox.execute` → разбор `"buy" in output`, третий генератор `_blocks_to_sandbox`) удалён. Сессия на версии без исполнимых блоков: старт и «Возобновить» → 422; при восстановлении после рестарта — переводится в `paused` с причиной, listener не поднимается; на свече — HOLD + warning.
> 3. **Эндпоинты `/api/v1/sandbox/execute` и `/api/v1/sandbox/analyze` сняты с регистрации** (UI их не использовал): исполнение произвольного кода по API отключено.
> 4. **Граница исполнения — прокси вместо модулей** (`app/sandbox/module_proxies.py`): в namespace `CodeSandbox` и `BacktestEngine._compile_strategy` кладутся `types.SimpleNamespace` с явным allow-list атрибутов (`MODULE_ALLOWED_ATTRS`: `datetime` — классы дат/времени; `math` — публичные функции; `decimal` — `Decimal`, исключения, режимы округления; `backtrader`/`bt` — `Strategy`, `indicators` (только классы индикаторов), `Order`, `TimeFrame`). Ни один атрибут прокси не является модулем (инвариант проверяется при построении); прокси создаются заново на каждый запуск. Раньше разрешённый модуль давал `datetime.sys.modules['os']` — выполнение произвольного кода.
> 5. **AST-анализатор — первый рубеж, не граница:** allow-list атрибутов на именах разрешённых модулей (`datetime.sys` → отказ), `from <модуль> import <имя>` — только из allow-list, перекрытое импортом имя проверяется по объединению allow-list модуля и публичных атрибутов импортированного объекта. Цепочки (`bt.indicators.x`) и псевдонимы через присваивание анализатор не отслеживает — их держат прокси (тесты `test_audit_s8r_escape.py` с отключённым анализатором).
> 6. **Builtins бэктеста** — allow-list `_STRATEGY_BUILTINS` (без `print`, `getattr`, `type` и т. п.), `__import__` → `_safe_import`, отдающий прокси.
> 7. Лимиты CPU/RAM из эскиза ниже в коде по-прежнему не применяются — карточка S8R-AUDIT-006.

**5.11.1 AST-анализ (pre-execution):**

```python
class ASTAnalyzer:
    FORBIDDEN_NAMES = {
        'eval', 'exec', 'compile', '__import__', 'getattr', 'setattr',
        'delattr', 'globals', 'locals', 'vars', 'dir', 'type',
        'open', 'input', 'breakpoint', '__builtins__', '__subclasses__',
    }

    ALLOWED_IMPORTS = {
        'backtrader', 'bt', 'pandas', 'pd', 'numpy', 'np',
        'talib', 'ta', 'datetime', 'math',
    }

    def analyze(self, code: str) -> AnalysisResult:
        tree = ast.parse(code)
        violations = []
        for node in ast.walk(tree):
            # Check imports (итерация по всем именам — import os, sys)
            if isinstance(node, ast.Import):
                for alias in node.names:
                    root_module = alias.name.split('.')[0]
                    if root_module not in self.ALLOWED_IMPORTS:
                        violations.append(f"Запрещённый импорт: {alias.name}")
            elif isinstance(node, ast.ImportFrom):
                root_module = node.module.split('.')[0] if node.module else ''
                if root_module not in self.ALLOWED_IMPORTS:
                    violations.append(f"Запрещённый импорт: {node.module}")

            # Check forbidden calls
            if isinstance(node, ast.Call):
                name = self._get_call_name(node)
                if name in self.FORBIDDEN_NAMES:
                    violations.append(f"Запрещённый вызов: {name}")

            # Check attribute access to __dunder__
            if isinstance(node, ast.Attribute):
                if node.attr.startswith('__') and node.attr.endswith('__'):
                    if node.attr not in ('__init__', '__next__'):
                        violations.append(f"Запрещённый доступ: {node.attr}")

        return AnalysisResult(is_safe=len(violations)==0, violations=violations)
```

**5.11.2 Исполнение:**

```python
class SandboxExecutor:
    CPU_LIMIT_SECONDS = 30
    MEMORY_LIMIT_MB = 512

    def execute_backtest(self, code: str, data: pd.DataFrame, params: dict) -> BacktestResult:
        # 1. AST анализ
        analysis = self.ast_analyzer.analyze(code)
        if not analysis.is_safe:
            raise UnsafeCodeError(analysis.violations)

        # 2. (эскиз до S8R-AUDIT-001; RestrictedPython удалён — S8R-AUDIT-050) Compile через RestrictedPython
        byte_code = compile_restricted(code, '<strategy>', 'exec')

        # 3. Запуск в subprocess с ограничениями
        # Используем multiprocessing.Process с resource.setrlimit
        result = self._run_in_subprocess(byte_code, data, params)
        return result

    def _run_in_subprocess(self, byte_code, data, params):
        """Запуск в отдельном процессе с лимитами CPU и RAM."""
        # resource.setrlimit(resource.RLIMIT_CPU, (30, 30))
        # resource.setrlimit(resource.RLIMIT_AS, (512*1024*1024, 512*1024*1024))
        ...
```

**Ограничение ресурсов по платформам:**
- **Linux:** `resource.setrlimit(RLIMIT_CPU)` + `resource.setrlimit(RLIMIT_AS)` — полноценная поддержка.
- **macOS:** `resource.setrlimit(RLIMIT_CPU)` работает; `RLIMIT_AS` игнорируется ядром — использовать мониторинг через `psutil.Process().memory_info().rss` каждые 0.5 сек в отдельном потоке с принудительным `Process.kill()` при превышении лимита 512 МБ.
- **Windows:** `psutil` для мониторинга CPU-времени и RAM + `Process.kill()` при превышении лимитов.

### 5.12 Corporate Action Handler

S8R-FIX-067: перед отбором облигаций для `bondization` тикеры открытых позиций с типом `unknown`/пустым (`UNTYPED_INSTRUMENT_TYPES`) дозапрашиваются по FIGI (`broker_figi` справочника, иначе сделки) через `MarketDataService.backfill_instrument_types`. Токен — счета владельца сессии: production вперёд, среди счетов контура — счёт сессии, неактивный заменяется другим активным счётом владельца; пары разных владельцев не смешиваются (S8R-AUDIT-003). Бюджет на прогон — `BOND_SCAN_TYPE_BACKFILL_MAX_REQUESTS=20` реально начатых запросов (FIGI из кэша промахов не считаются) и общий дедлайн 15 с; при остатке меньше `BOND_SCAN_TYPE_BACKFILL_MIN_SLICE_SEC=1` новые запросы не начинаются, FIGI не помечается. Найденные типы пишутся в `instruments` одной транзакцией и не позже дедлайна. Сеть — в `_fetch_iss`, вне единицы записи. Сбой — отбор по эвристике тикера.

**Ответственный: BACK1**

Модуль обработки корпоративных действий (сплиты, дивиденды, купоны).

**Источники данных:** дивиденды — ~~T-Invest `InstrumentsService.GetDividends` (S8R-FIX-122, с 2026-10-14)~~ с v4.10 (A-06) — история выплат `read_dividends` (§5.15) через `execution.dividends.load_dividend_schedule` (§5.17.6); сплиты — MOEX ISS `/iss/statistics/engines/stock/splits.json`, купоны — `/iss/securities/{secid}/bondization.json` (S8R-AUDIT-091). Проверка каждые 6 часов через scheduler job `detect_corporate_actions` (S8R-FIX-008: прежний текст «ежедневно 08:00 MSK» расходился с кодом). Уведомление (`SchedulerService.detect_corporate_actions`, снимок `_DetectedAction` c `record_date`): дивиденд и купон — `закрытие реестра DD.MM.YYYY, последний день покупки DD.MM.YYYY` (реестр — `record_date`, у купона `ex_date` — дата выплаты; последний день покупки — `accrual_last_buy_date`); сплит — `первый день торгов в новом масштабе DD.MM.YYYY`. Календарь недоступен — только дата реестра. До S8R-FIX-122 вторая дата называлась `ex-date` и была на торговый день раньше настоящей.

**Тикеры детекта и адресаты (S8R-FIX-129).** Джоба берёт тикеры `detection_tickers()` (верхний регистр, один на бумагу — иначе `SBER`/`sber` давали две строки и двойное начисление): сессии `active`/`paused` и сессии любого статуса, у которых есть сделки в `_ACCRUAL_TRADE_STATUSES`, открытые или закрытые не раньше `_DIVIDEND_HORIZON_DAYS` назад — начисление (`accrual_position_clauses`) статус сессии не смотрит, и без этого выплата позиции остановленной сессии не детектировалась. Остановленная сессия без таких сделок тикер не добавляет. Уведомление о дивиденде/купоне: владельцы сессий `active`/`paused` по тикеру плюс `accrual_recipient_user_ids(action)` — владельцы позиций, которым причитается выплата (те же условия, что у начисления); один пользователь — одно уведомление; сбой этого поиска (WARNING `corporate_action_recipients_failed`) не лишает уведомления владельцев живых сессий. О сплите — владельцы живых сессий плюс `split_position_user_ids(action)`: владельцы позиций, которые `process_split` пересчитывает (`filled`/`open`, открыта до 00:00 МСК дня сплита), статус сессии не важен (S8R-FIX-132).

**Знак выплаты (S8R-FIX-131).** `process_dividend` / `process_coupon`: сумма позиции умножается на +1 для `is_long_direction(trade.direction)` и на −1 для шорта (вход шорта — `direction="sell"`): держатель шорта на отсечке компенсирует выплату. Купон: `знак × (купон − nkd_entry) × штуки`. Paper — изменение `balance`, sandbox/real — `DailyStat` за день. Уведомление держателю шорта — с суффиксом `; по короткой позиции сумма спишется` (S8R-FIX-133, `accrual_short_user_ids`).

**Отсечка T+1 (S8R-FIX-122, решение заказчика 2026-10-14).** `accrual_last_buy_date(record_date)`: ex-date = последний торговый день ≤ даты реестра (`_trading_day_on_or_before`, календарь MOEX S8R-AUDIT-093; торги выходного дня торговыми днями календаря не считаются), последний день покупки — последний торговый день < ex-date. `accrual_opened_before` = 00:00 МСК дня после последнего дня покупки (naive UTC); `accrual_position_clauses`: `opened_at < порог` и (`closed_at IS NULL` или `closed_at ≥ порог`). Прежний `accrual_ex_date` (S8R-FIX-008) возвращал последний день покупки и ставил порог на его 00:00 — на торговый день раньше. Правило сверено с `last_buy_date` T-Invest по 13 выплатам (тест `test_fix_122_t1_cutoff.py`). Купон без `recorddate` ISS: `record_date` = торговый день перед `coupondate` (было: сама `coupondate`).

**Дивиденды T-Invest (S8R-FIX-122).** ISS `securities/{secid}/dividends.json` отвечает 200 описанием бумаги (gotcha-70), дивиденды остались только в CCI по подписке — ISS-путь дивидендов удалён (`_iss_dividend_rows`, `iss_dividends_block_missing` больше нет). Адаптер: `TInvestAdapter.get_dividends(figi, from_, to)` — квота `INSTRUMENTS`, дедлайн `_unary`, повтор транзиентных сбоев `_read_with_retry` (только чтение), naive-границы — UTC; разбор `TInvestMapper.dividend_to_dividend_info` → `DividendInfo` (`app/broker/base.py`): `amount_per_share` — `Decimal` из `dividend_net` (`MoneyValue`), валюта в верхнем регистре, даты событий — календарный день МСК (брокер кодирует их полуночью UTC; пустой `Timestamp` → `None`). Детект: `_fetch_tinvest_dividends(tickers, today)` вне единицы записи — для каждого тикера (облигации пропускаются) владельцы сессий по тикеру (`_dividend_targets`: активные/на паузе вперёд, сессии со счётом вперёд) → `select_market_data_account(user, preferred_account_id=счёт сессии)` (production вперёд, gotcha-57; S8R-AUDIT-003 — токены владельцев не смешиваются) → `ensure_figi_lookup` тем же счётом → один адаптер на счёт за прогон (закрывается в `finally`); `_read_dividends` — один запрос. Владельцы перебираются, пока один не ответит (ревью п.3): нет счёта, FIGI или отказ по инструменту — следующий. Окно `_dividend_window`: от 00:00 МСК дня самой ранней сделки по тикеру (статусы `filled/open/closed`) до `today + 367` дней. Нет счёта ни у одного владельца — WARNING `dividends_no_broker_account`; FIGI не найден — `dividends_figi_unresolved`; отказ по инструменту — `detect_dividends_failed`. `TransientBrokerError` (WARNING `detect_tinvest_unavailable` с `broker_account_id`), `AuthBrokerError` и ошибка расшифровки ключа выводят из прогона **счёт** (`unusable`), а не весь прогон (ревью п.2): его тикеры достаются другим владельцам, оставшиеся без ответа — один WARNING `dividends_skipped_account_unavailable` со списком. Ошибки БД (выбор владельцев/счёта) пробрасываются. `_store_dividends(ticker, rows, today)`: `record_date` брокера → `ex_date` и `record_date` (ключ дедупа `uq_corp_action` как у ISS-эпохи), последний день покупки и дата выплаты брокера — в `description`; пропуск без реестра/суммы, `Cancelled` (INFO), не `RUB` и пустая валюта (WARNING `tinvest_dividend_currency_unsupported`; маппер пустую валюту рублями не считает). `_sync_pending_dividends` (ревью п.1): непроведённые дивиденды тикера с `ex_date` в окне ответа (`_DividendAnswer.window_from/to`) сверяются с ним — сумма изменилась → обновление (WARNING `tinvest_dividend_amount_changed`), выплаты нет в ответе (отмена, нулевая сумма, перенос реестра) → `DELETE` без начисления (WARNING `tinvest_dividend_withdrawn`); проведённые не трогаются. Дивиденд с реестром < `today` без получателей (`_has_accrual_recipients`) — `processed=True` (история без уведомления); причитающийся — непроведённым, его начислит `process_pending` (первый прогон после обновления начисляет пропущенные — решение заказчика 2026-10-14). Миграции нет.

**Дробления и ряды свечей (A-01, ТЗ v4.0, дефект 4 тестового стенда; решение заказчика 2026-10-24).** Строки `action_type='split'` — источник пересчёта рядов свечей (`market_data/splits.py`, §3.5): `_store_splits` сохраняет сплиты всех бумаг из `splits.json`, не только с сессиями. Планировщик вызывает детект и без живых сессий — `detect_corporate_actions([])`: синхронизируются только сплиты, потому что ряды свечей читаются с их учётом (раньше без сессий детект не шёл, §5.9). Уборка рядов идёт **до** записи дроблений, в той же сессии: `detect_corporate_actions` после сетевой части вызывает `_forget_bars_before_new_splits(snapshot.split_rows)` — новые (ещё не записанные) сплиты ответа передаются `forget_bars_before_new_splits`: агрегаты бумаги в LRU вытесняются; если граница (`split_opened_before(ex_date)`) уже наступила, строки бумаги до границы в `ohlcv_cache` (1m, D; T-Invest и ISS) и `ohlcv_dealer` удаляются вместе с покрытием (`delete_bars_with_coverage`) и сверками `ohlcv_volume_check` дней до `ex_date`, одним commit (WARNING `split_bars_purged`): источник мог пересчитать их до этого детекта, а записаны они как «как было». Удалённое дозагрузится по запросу и фоновой догрузкой. Затем `_store_detected` пишет дробления. Сбой уборки — `rollback`, ERROR `split_bars_purge_failed`; новые дробления в этот прогон **не записываются** (`skip` для `_store_splits`) и останутся новыми до следующего прогона, детект не падает. Порядок «уборка, затем запись» не оставляет записанного дробления с неубранными рядами. Дробление с ещё не наступившей границей строк не трогает.

**Предупреждение бэктеста о сплите (S8R-AUDIT-091, ФТ §4.2).** `backtest/service.corporate_action_warning` → поле `warning` ответа бэктеста (`backtest/router._build_backtest_response`, один Alert с диагностикой сайзинга). С A-01 бэктест получает ряд с учётом дроблений (`get_candles`, `adjust="splits"`), разрыва в `ratio` раз нет, и текст сообщает об этом: один сплит — `В периоде был сплит DD.MM.YYYY (rf:rt): цены и объёмы до него пересчитаны в текущий масштаб.`; несколько — `В периоде были сплиты DD.MM.YYYY (rf:rt), …: цены и объёмы до них пересчитаны в текущий масштаб.` Прежний текст «цены до сплита не скорректированы» и оговорка «back-adjust — кандидат Sprint 9» сняты решением заказчика: пересчёт сделан.

**Логика обработки:**

```python
class CorporateActionHandler:
    async def process_split(self, action: CorporateAction) -> None:
        """Сплит: для всех open live_trades по тикеру — пересчёт
        entry_price (÷ ratio) и volume_lots (× ratio).
        Запись в audit_log. Уведомление пользователю."""
        ...

    async def process_dividend(self, action: CorporateAction) -> None:
        """Дивиденд: создание записи в daily_stats с учётом суммы
        (amount_per_share × volume). Уведомление пользователю."""
        ...

    async def process_coupon(self, action: CorporateAction) -> None:
        """Купон по облигации: аналогично дивиденду, с учётом НКД
        через BondService.calculate_nkd()."""
        ...
```

A-02 (v4.5) обработчик не менял: начисления по сделкам читают выплаты `GetDividends` ключом владельца сессии, как прежде (**с v4.10, A-06, — `read_dividends`; `_fetch_tinvest_dividends` удалён; последний день покупки дивидендов — `execution.dividends.last_buy_day` с T+2 до 31.07.2023, начисление — днём гэпа с налогом сессии и отметкой `dividend_accruals`; `accrual_last_buy_date` остался для купонов — §5.17.6**); история выплат по бумаге со сверкой — §5.15, её потребители — A-06 (B2), Б-05, Б-06.

### 5.13 Tax Report Generator

XLSX (openpyxl): строковые поля пишутся как текст (`xlsx_text`, `data_type='s'`) — не формула; числа — числами (S8R-AUDIT-065).

База (S8R-AUDIT-095): `max(0, Σ share + bond + etf)` за год (единая база обращающихся ЦБ, ст. 214.1); тип — `instruments.type` (`_resolve_instrument_types`, один запрос на отчёт; `ensure_figi_lookup` пишет `InstrumentShort.instrument_type` вместе с FIGI), `_tax_category` → share/bond/etf, эвристика по тикеру — fallback с пометкой в файле отчёта.

Лоты и файлы (S8R-AUDIT-096): `TaxLot.report_id` (FK, CASCADE); регенерация за год заменяет расчётные лоты (удаляются лоты прежних отчётов года в той же единице работы), прежние отчёты остаются доступны для скачивания — каждый отчёт хранит собственный файл `tax_report_{report.id}.{ext}`. FIFO не используется: одна сделка — один лот (`_build_lots`).

S8R-FIX-043: `_prepare_inputs` до `persist_with_retry`, один раз на генерацию: сделки года, лотность (`_resolve_lot_size_fallbacks`) и дозапрос типов (`_prefetch_instrument_types` → `MarketDataService.ensure_instrument_types` — обёртка `backfill_instrument_types`, строки справочника одной транзакцией). Единица работы сети не делает; первая попытка берёт готовые сделки, повтор перечитывает их (gotcha-37). Дозапрос: тикеры лотов отчёта и сделок «P&L не определён» с типом `unknown`/пустым, вес — Σ|realized_pnl| по `ticker.upper()` с лотностью (у сделок без цены — |комиссия|, S8R-REV-FBE); один FIGI — один `GetInstrumentBy`; бюджет `TAX_TYPE_BACKFILL_MAX_REQUESTS=20`, общий дедлайн `TAX_TYPE_BACKFILL_TIMEOUT_SEC=15` (только сеть); промахи — `TtlCache` по FIGI (`grpc_codes`: TRANSIENT/LOST и запрос, оборванный дедлайном, — 15 мин; NOT_FOUND, пустой тип, чужой тикер — 24 ч; AUTH/RATE_LIMITED — не кэшируются); ответ пишется в `instruments` (type+figi) отдельной сессией; метрика `market_data.ensure_instrument_types`. Горячий путь `ensure_figi_lookup` тип не дозапрашивает. ПФИ = `PFI_TYPES` {futures, option, currency} — построчный раздел вне базы и итогов, результат в единицах цены, без суммы; прочие типы — `by_type["other"]` в базе. Строки файла — общий `_report_rows` для xlsx/csv.

Комиссия лота (S8R-FIX-049): покупка и продажа берут комиссию своей стороны сделки (`entry_commission`/`exit_commission`); сделки, закрытые до миграции `c04dc4f891a2` (`commission_legacy=1`), — итог делится 50/50, как раньше.

**Ответственный: BACK1**

Модуль: `export/tax_report.py`

**Логика формирования налогового отчёта для 3-НДФЛ:**
- Метод **FIFO** для определения финансового результата по закрытым сделкам.
- Раздельный учёт: акции, облигации (+ купонный доход отдельной строкой), ETF/БПИФ.
- Группировка по налоговым периодам (календарный год).
- Учёт комиссий брокера (уменьшают налогооблагаемую базу).
- Для облигаций: разделение ценовой составляющей и НКД.
- Колонки экспорта: дата сделки, тикер, ISIN, направление, цена покупки, цена продажи, объём (лоты / штуки), комиссия, финансовый результат.
- Форматы: xlsx (`openpyxl`), csv.

### 5.14 Новости — `app/news/` (v4.4, Б-01)

Требования — ФТ §20; схема — §3.26, API — §4.14, задания — §5.9, страница — §6.10. Решения и их причины — ADR 0101–0104 в репозитории кода (`docs/adr/0101-archive-title-and-link-only.md`, `0102-labels-one-record-per-labeler.md`, `0103-reaction-1m-from-exact-publication-time.md`, `0104-ticker-dictionary-rules.md`); карта модуля — `docs/architecture.md`, глава «Новости (блок Б-01)».

```
 news_sources (url, kind) ─► FeedFetcher ─► parse_rss | parse_iss_sitenews ─► NewsCollector ─► news_items
 ISS: акции TQBR (кэш 24 ч) + news_dictionary_terms ─► TickerDictionary.match(title) ─► label_unlabeled ─► news_labels + news_tickers
 пара «новость — тикер» без реакции ─► news_reactions: pending (due_at = P + 75 мин) ─► ISS get_candles(тикер, 1m) ─► compute_reaction ─► done | failed
 queries.list_news (курсор) ─► GET /api/v1/news* ─► newsApi.ts ─► newsStore ─► NewsPage (опрос 60 с)
```

**Файлы:** `constants.py` (коды, интервалы, пороги, `User-Agent`, лимиты — без адресов лент), `models.py`, `feeds.py`, `collector.py`, `dictionary.py`, `labeling.py`, `reactions.py`, `service.py` (единственное место сборки), `queries.py`, `schemas.py`, `router.py`. Общие файлы тронуты точечно: `main.py` (импорт моделей, роутер `/api/v1/news`), `scheduler/service.py` (два задания и закрытие клиентов в `shutdown()`). Модули `market_data` и `broker` только вызываются (из ISS-клиента — `get_json`, `get_candles`, `iss_path`); `app.broker.tinvest`, `app.trading`, `app.sandbox` не импортируются (`test_news_does_not_touch_broker_trading_sandbox`); адреса лент в коде не встречаются (`test_feed_addresses_live_only_in_sources_table`). **Настроек в `config.py` блок не добавил** — всё в константах:

| Константа | Значение | Что |
|---|---|---|
| `COLLECT_INTERVAL_SECONDS` / `REACTION_INTERVAL_SECONDS` | 60 / 60 с | период заданий `news_collect` / `news_reactions` |
| `FEED_TIMEOUT` (`feeds.py`) | connect 5, read 15, write 5, pool 5 с | таймауты httpx на запрос ленты |
| `FETCH_DEADLINE_SECONDS` | 30 с | общий срок на скачивание одного источника (`asyncio.timeout`; сервер, отдающий тело по байту, не держит проход) |
| `MAX_FEED_BYTES` | 2 000 000 Б | предел тела — по `Content-Length` и по распакованному потоку |
| `HTTP_USER_AGENT` | `MoexTerminal-NewsCollector/1.0` | подпись запросов к лентам |
| `ISS_SITENEWS_LINK_TEMPLATE` | `https://www.moex.com/n{id}` | ссылка сообщения биржи (в ответе ISS её нет) |
| `MAX_ERROR_TEXT` | 1000 | обрезка `last_error` |
| `REACTION_DELAY` / `REACTION_WINDOW` | 75 / 60 мин | срок расчёта P + 75 мин / окна «до» и «после» |
| `REACTION_BATCH` / `REACTION_MAX_ATTEMPTS` | 20 / 10 | реакций за проход / неудач до `failed` |
| `TQBR_CACHE_TTL` / `TQBR_LOAD_DEADLINE_SECONDS` (`dictionary.py`) | 24 ч / 20 с | кэш списка акций TQBR / срок его загрузки |

**5.14.1 Сборщик** (`feeds.py`, `collector.py`; ADR 0101).
- `NewsCollector.collect_once()`: включённые источники (`enabled`) опрашиваются параллельно (`asyncio.gather`), каждый ровно один раз; запись — последовательно, своя транзакция на источник (SQLite — один писатель). `received_at` — время после ответа всех источников, общее для прохода: медленный источник сдвигает его всем (таймаут ЦБ через прокси — ≈ 15 с).
- `FeedFetcher` — один `httpx.AsyncClient` на процесс (создаётся лениво, закрывается в `SchedulerService.shutdown()`), `follow_redirects=True`; транспорт не подменяется — действуют прокси окружения `HTTP_PROXY` / `HTTPS_PROXY` / `NO_PROXY` (gotcha-84). Ответ не 200, таймаут, ошибка сети, превышение срока или размера — `FeedError`, то есть сбой этого источника.
- **Защита разбора без новых зависимостей** (решение заказчика 2026-10-25): стандартный `ElementTree` с `# nosec B405/B314` и обоснованием рядом, `backend/.bandit` не правился; до разбора — лимит размера, отказ при нулевом байте (UTF-16/32: поиск разметки по байтам там слеп) и при `<!DOCTYPE` / `<!ENTITY` в любом регистре; многобайтовая кодировка, которой нет в expat, — `FeedError`.
- `parse_rss`: корень `rss`, `channel/item`; берутся `title` (HTML-сущности раскрыты `html.unescape`), `link`, `guid`, `pubDate` (RFC 822 → naive UTC; без смещения — UTC); `description` не читается. Элемент без заголовка, без ссылки, со ссылкой не `http(s)` или с битой датой пропускается (`news_feed_item_skipped`). `external_id` = guid, без него — ссылка.
- `parse_iss_sitenews`: блок `sitenews` по имени и колонкам `id`, `title`, `published_at` (gotcha-70: на чужой ресурс ISS отвечает 200 другим блоком — это сбой, а не пустая лента); `published_at` — МСК → UTC; `external_id` = id сообщения, ссылка — по шаблону.
- **Запись:** дубли внутри ответа схлопываются по `external_id`, между проходами — `INSERT … ON CONFLICT (source_id, external_id) DO NOTHING` пачками по 500, число новых — `rowcount`; в той же транзакции источник получает `last_polled_at = last_success_at = now`, `consecutive_failures = 0`. Сбой — `last_polled_at`, `last_error` (≤ 1000 символов), `last_error_at`, `consecutive_failures + 1`. После выздоровления `last_error` остаётся записью о последнем сбое — здоровье источника смотреть по `consecutive_failures` и `last_success_at`.
- **Логи:** `news_source_collected`, `news_source_failed` (warning, каждый проход упавшего источника), `news_source_unexpected_error`, `news_source_store_failed`, `news_collect_cycle_done`, `news_collect_cycle_failed`; длительность прохода — `timed_event("news.collect_cycle")`.

**5.14.2 Словарь и разметка** (`dictionary.py`, `labeling.py`; ADR 0102, 0104).
- **Список акций TQBR** — `GET /iss/engines/stock/markets/shares/boards/TQBR/securities` (`iss.only=securities`, колонки `SECID,SHORTNAME,SECTYPE`); берутся SECTYPE `1` и `D` (обыкновенные) и `2` (привилегированные), паи фондов той же доски отброшены (на 2026-10-25 — 261 из 510 строк). `DictionaryProvider` держит список в памяти процесса 24 ч, загрузка — со сроком 20 с; сбой — работа на прежнем списке, прежнего нет — `get()` → `None`, разметка откладывается (`news_labeling_postponed`), новости уже сохранены. На диске список не кэшируется: после перезапуска грузится заново.
- `TickerDictionary.build(securities, terms)`: имя бумаги — SHORTNAME без пометок вида и формы (`ао`, `ап`, `-п`, `3ао`, `ПАО`, `МКПАО`, префикс `i`, кавычки), нижний регистр, `ё → е`; одно имя у обыкновенной и привилегированной акции — обыкновенной; псевдоним бумаги, которой нет в списке TQBR, не действует. Термины читаются из `news_dictionary_terms` при каждой сборке словаря — правка таблицы действует со следующего прохода без выпуска.
- `match(title)`: совпадение целыми словами (внутри фразы допустимы пробел, дефис, «+», кавычки, точка без пробела — «ДОМ.РФ», «М.Видео»; двоеточие, запятая, тире — граница фразы); у последнего слова варианта от 4 букв — падежные окончания; исключение гасит совпадение внутри своей фразы (исключение без тикера — любое), кроме имени в кавычках вплотную (с v4.9 — только если исключение состоит ровно из этого имени, ниже); из вложенных совпадений разных бумаг остаётся более длинное («Газпром нефть» → SIBN, не GAZP). Результат — отсортированные тикеры без повторов.
- **Стартовый набор** — `_ALIASES` / `_EXCLUSIONS` ревизии `b01e7a5c3d92` (187 / 30): бренды и полные названия («газпром нефть», «норникель», «сбер», «т-банк», «пятерочка»), латинские тикеры и названия ликвидных бумаг («gazp», «ozon», «vk»), дочерние «Россетей», «glorax» → `GLRX` (SHORTNAME ISS — «ГЛОРАКС», в заголовках — латиницей; пропуск найден на стенде 05.11, `e769c1e`); исключения — «самолет», «лента», «полюс», «система», «газ», «пик спроса», «индекс мосбиржи» и т. п. Набор составлен по общему знанию, а не по эталону D8.
- **Кавычки — с v4.9 (Б-07, ADR 0105 заменяет часть ADR 0104):** `dictionary.py::_suppressed` — имя в кавычках спасается только от исключения той же длины, что и совпадение (`exclusion.size == hit.size`: «самолет» ↔ «Самолёт»); исключение-фраза длиннее имени («на мосбирже», «фк акрон», «газпром медиа») гасит его и в кавычках. Ревизия `b07d34cd8897` добавляет 11 исключений (§3.29); переразметки архива нет.
- `label_unlabeled(session, dictionary, limit=500)` — в задании `news_collect` после сбора (и после неудачного сбора): новостям без строки `labeler = 'dictionary'`, старые первыми, — одна `NewsLabel(labeler='dictionary', news_type='other')` и строки `news_tickers`; ничего не найдено — пустая разметка. Архив при правке терминов не переразмечается.
- **Замер на эталоне D8** — консольный `backend/scripts/news_benchmark.py` (`python -m scripts.news_benchmark <эталон.json> <фикстура TQBR>`, вне `app/`: терминал его не вызывает): 108 заголовков — найдено 36 пар, верных 36; из 37 верных пар эталона — 36.

**5.14.3 Реакция «до / после»** (`reactions.py`; ADR 0103).
- `ReactionWorker.run_once()`: (1) заводит `pending` для пар `news_tickers` без строки реакции — `due_at = published_at + REACTION_DELAY`, `ON CONFLICT (news_id, ticker) DO NOTHING` (одна реакция на пару, сколько бы разметок ни назвали бумагу); (2) берёт до 20 строк `pending` с `due_at ≤ now` (`ORDER BY due_at, id`); (3) свечи — `MOEXISSClient.get_candles(ticker, "1m", дата МСК (P − 60 мин), дата МСК (P + 60 мин))`, один запрос на (тикер, даты) за проход, ошибка тоже запоминается на проход.
- `compute_reaction(candles, published_at)`: цена(t) — `close` последней свечи с `end ≤ t` (свеча 16:59 при публикации 16:59:54 не берётся); окно «до» — свечи с `begin` в [P − 60 мин, P), «после» — в [P, P + 60 мин); в окне нет свечей — `*_no_trades = true`, процент NULL; цены на начало окна нет, а свечи в окне есть (торги открылись внутри окна) — база = `open` первой свечи окна; свечи есть, но к концу окна не закрылась ни одна — процент NULL при `no_trades = false`. Процент = (последняя / база − 1) × 100, `quantize(0.0001, ROUND_HALF_UP)`.
- **Эталон:** OZON, P = 2026-10-19 16:59:54 МСК, настоящий ответ ISS (`tests/test_news/fixtures/iss_candles_ozon_1m/`) — `before_pct = −0.2118`, `after_pct = −0.8207` («до −0,21 %, после −0,82 %»). Числа плана (−0,28 / −1,15) — 10-минутная сетка исследования; расхождение закреплено тестом (`test_ozon_reaction_matches_customer_reference`).
- Сбой ISS — `attempts + 1`, `last_error`, повтор в следующем проходе; 10-я неудача — `failed` («реакция недоступна»). Успех — `done`, `computed_at`. Строка обновляется только из `pending`.
- **Логи:** `news_reaction_failed`, `news_reaction_cycle_done`, `news_reaction_cycle_failed`; `timed_event("news.reaction_cycle")`.

**5.14.4 `NewsService`** (`service.py`) — сборка для планировщика: `FeedFetcher`, `NewsCollector`, свой `MOEXISSClient(settings.MOEX_ISS_BASE_URL)` (общий для словаря и реакций, закрывается в `close()`), `DictionaryProvider`, `ReactionWorker`. `run_collect_cycle()` и `run_reaction_cycle()` глотают любое исключение с логом (соглашение `SchedulerService`).

**Тесты:** `backend/tests/test_news/` — разбор на настоящих сохранённых ответах пяти RSS-лент, ЦБ (`rss_cbr.xml`, UTF-8 с BOM) и ISS `sitenews`, список TQBR, 1m OZON за 19.10.2026; сеть в тестах закрыта, ответы подаются через `httpx.MockTransport`; БД — `alembic upgrade head` один раз на сессию и копия через `sqlite3.Connection.backup` на тест; путь «опрос → архив → API ленты» за один проход — `test_news_published_30s_ago_reaches_the_feed_in_one_collect_pass`. Frontend — vitest (`newsApi`, `newsStore`, `NewsPage`) и E2E `frontend/e2e/b01-news.spec.ts` на моках API.

**Известные ограничения.** ЦБ через прокси окружения не отвечает: `ReadTimeout` каждый проход, warning раз в минуту (≈ 1440 строк в сутки) — лечится `NO_PROXY` (`deployment_guide.md` §7). Повтор пропуска одного и того же элемента ленты тоже логируется каждый проход. `<!DOCTYPE` внутри CDATA заголовка делает сбоем всю ленту (разметка DTD ищется по всему телу, а не только в прологе); неизвестное имя кодировки даёт `LookupError` — сбой источника через ветку неожиданного исключения, а не `FeedError`. Общий срок 30 с на источник тестом не покрыт. Зависший ISS растягивает проход реакций (до ~30 мин); новые пары ищутся по всей таблице связей раз в минуту. Новость в первую минуту торгов может получить «до» без числа и без «нет торгов». Список типов продублирован (`constants.NEWS_TYPES` и `NEWS_TYPES` в `newsApi.ts`), теста синхронности нет. Комментарии `tests/test_news/helpers.py` и докстринг ревизии устарели — считают, что ответа ЦБ нет.

### 5.15 Дивиденды — `app/dividends/` (v4.5, A-02)

Требования — ФТ §21; схема — §3.27, API — §4.15, задания — §5.9, страница — §6.11. Решения — ADR 0035 (счёт для рыночных данных), 0036 (`docs/adr/0036-a02-dividends-discrepancies-on-read-corrections-table.md`: расхождения при чтении, поправки отдельной таблицей).

```
 ISS boards/TQBR/securities (SECID, SECTYPE ∈ {1, 2, D}) ─▶ load_tinvest_dividends (04:30 MSK; market_data_account; от давно не загружавшихся, пауза 0,8 с; лимит — RateLimitWait) ─▶ TInvestAdapter.get_dividends(figi, 2000-01-01 … сегодня + 366 сут) ─▶ _payable ─▶ dividend_payments (замена строк бумаги) + dividend_sources.tinvest_loaded_at
 check_smartlab (05:00 MSK; ≤ ceil(N/7) бумаг, пауза 5 с) ─▶ smartlab.fetch_smartlab_page ─▶ parse_smartlab_dividends ─▶ _merge_by_record_date ─▶ dividend_smartlab_rows | dividend_sources (unavailable)
 POST / DELETE /dividends/{ticker}/corrections (администратор) ─▶ save_correction / delete_correction ─▶ dividend_corrections + audit_log
 _reconcile при чтении: «T-Invest + поправки» ↔ Смартлаб ─▶ dividend_overview · dividend_table · read_dividends(db, ticker, from, to) ─▶ GET /dividends* ─▶ DividendsPage
```

**Файлы:** `models.py`, `service.py` (загрузка, сверка, поправки, расхождения, чтение), `smartlab.py` (HTTP и разбор), `schemas.py`, `router.py`. Общие файлы — точечно: `main.py` и `cli/users.py` (импорт моделей, роутер), `scheduler/service.py` (две задачи), `common/audit.py` (действия), `broker/moex_iss/client.py` (`get_board_tickers`). Новых зависимостей нет (разбор — стандартный `html.parser`). Настроек в `config.py` нет:

| Константа | Значение | Что |
|---|---|---|
| `BOARD` | `TQBR` | «весь рынок» дивидендов и сверки — `SECID` режима по ISS |
| `SHARE_SECTYPES` | `{"1", "2", "D"}` | только акции и депозитарные расписки (`SECTYPE` ISS); паи фондов (J, 9, A, B) не берутся; тот же набор — у словаря новостей (`news/dictionary.py`) |
| `TINVEST_PAUSE_SECONDS` | 0,8 с | пауза между бумагами — не больше 150 запросов в минуту при квоте InstrumentsService 200 на токен |
| `TINVEST_RATE_LIMIT_MAX_WAITS` / `TINVEST_RATE_LIMIT_MAX_TOTAL_WAIT_SEC` | 10 / 600 с | бюджет ожидания сброса лимита на весь прогон; исчерпан — прогон встаёт |
| `TINVEST_DIVIDENDS_FROM` / `TINVEST_DIVIDENDS_AHEAD` | 2000-01-01 / 366 суток | окно дат реестра `GetDividends` |
| `SMARTLAB_RECHECK` / `SMARTLAB_PAUSE_SECONDS` | 7 суток / 5 с | бумага — не чаще раза в неделю; пауза между страницами |
| `PAIR_MAX_DAYS` / `PAIR_MAX_RELATIVE` | 3 дня / 3 % (от суммы Смартлаба) | пара выплат |
| `DATE_DIFFERS_MAX_DAYS` | 31 день | «дата отличается» — не дальше месяца, иначе разные выплаты |
| `SMARTLAB_URL` (`smartlab.py`) | `https://smart-lab.ru/q/{ticker}/dividend/` | страница бумаги |
| `MAX_PAGE_BYTES` / `_TIMEOUT` | 2 000 000 Б / connect 5, read 15, write 5, pool 5 с | предел тела и таймауты |
| `COMMENT_MAX_LEN` (`schemas.py`) | 1000 | комментарий поправки |

**5.15.1 Загрузка T-Invest.** `load_tinvest_dividends(db_factory)`: `market_data_account(db)` — нет: warning `market_data_account_missing` (`job=load_tinvest_dividends`), выход; `TInvestAdapter(sandbox=False).connect(*decrypt_market_data_token(account))` — ключ не годен: `dividends_tinvest_key_unusable`. Список бумаг — `MOEXISSClient.get_board_tickers("TQBR", sectypes=SHARE_SECTYPES)` (сбой — `dividends_board_tickers_failed`, пустой список — `dividends_board_tickers_empty`, выход); порядок — `_oldest_loaded_first` (без `tinvest_loaded_at`, затем давнее всех; при равенстве — по тикеру); между бумагами `asyncio.sleep(0,8)`. По бумаге: `ensure_figi_lookup(ticker, broker_account_id=account.id)` (нет FIGI — `dividends_figi_unresolved`), `_get_dividends_waiting` (`adapter.get_dividends(figi, from, to)`; `TransientBrokerError` с `FailureKind.RATE_LIMITED` — `RateLimitWait.wait` и повтор того же запроса; один бюджет на прогон), `_payable` (без `Cancelled` и пустых; одна дата в одной валюте — сумма; разные валюты — `tinvest_dividend_currency_mixed`), `_replace_payments` в `persist_with_retry_fresh` (3 попытки). `AuthBrokerError` — `dividends_tinvest_key_rejected`, прогон останавливается; `RateLimitExhaustedError` (бюджет ожидания исчерпан) — `dividends_tinvest_rate_limited` (`left`, `loaded`, `failed`) и итог с `stopped=rate_limited`, прогон встаёт; отказ по лимиту при поиске FIGI не пережидается (бумага — первой следующей ночью); прочий сбой бумаги — `dividends_tinvest_failed`, её строки прежние. Итог — `dividends_tinvest_loaded` (`tickers`, `failed`). Логи — без `exc_info` и текста ключа.

**5.15.2 Сверка со Смартлабом.** `check_smartlab(db_factory)`: список акций TQBR (`SHARE_SECTYPES`); срок — `smartlab_checked_at ≤ now − 7 суток` (не проверявшиеся — `datetime.min`); сортировка `(последняя попытка, тикер)`, срез `ceil(len/7)`; между страницами `asyncio.sleep(5)`. `fetch_smartlab_page` — свой `httpx.AsyncClient` на запрос, `trust_env` (прокси окружения), переадресации не выполняются, не 200 / больше 2 МБ / сеть — `SmartlabUnavailable`. `parse_smartlab_dividends` — `<table>` с атрибутом `financials-param-csv-download-data`, начинающимся с «Выплаченные дивиденды»; колонки по заголовку: «дата отсечки», «дивиденд» обязательны, «дата T-1», «Период» — если есть; `<tr class="dividend_canceled">` → `canceled`; валюта по знаку (₽, руб, $, €). Удача — `_merge_by_record_date` (строки одной даты — сумма, периоды через «; », отменена — только если отменены все) и замена строк бумаги, `smartlab_status='ok'`; любой сбой — `unavailable`, `smartlab_error` (≤ 200), строки не трогаются (`smartlab_check_unavailable`). Попытка считается проверкой. Итог — `smartlab_check_done` (`checked`, `unavailable`, `due`); сбой записи — `smartlab_check_store_failed`.

**5.15.3 Поправки.** `save_correction(db, ticker, admin=…, action, record_date, amount, new_record_date, last_buy_date, comment, request)` — upsert по `(ticker, record_date)` и `audit(…, "dividend.correction_saved")` в одной транзакции (коммит — роутер); `delete_correction` — удаление и `dividend.correction_deleted`, нет строки — `False` → 404. Ответ `correction_view` — с `author` (имя пользователя `created_by`).

**5.15.4 Расхождения и чтение.** `_reconcile` при каждом чтении: «T-Invest + поправки» (`add` добавляет или подменяет сумму на дату, `exclude` убирает, `fix` меняет сумму и/или дату) против строк Смартлаба; сначала пары «дата ±3 дня и сумма ±3 %», затем `differs` (дата ±3 дня, сумма вне 3 %; или сумма в 3 %, дата дальше 3, но ≤ 31 дня), остаток — `missing_in_tinvest` / `missing_on_smartlab`; поправки без опоры — `unpaired_correction`. Сверяются только годы (по дате реестра) со строками Смартлаба и даты не позже сегодня МСК (`today_msk`); без `tinvest_loaded_at` или без счёта для рыночных данных расхождения не считаются; исключённая выплата гасит строку Смартлаба на дату ±3 дня; `canceled` расчёт не читает. `dividend_overview` (список бумаг со счётчиками — расчёт по всем бумагам заново), `dividend_table` (строки бумаги), `read_dividends(db, ticker, date_from, date_to) -> DividendHistory(payments, has_unresolved_discrepancies)` — единственная функция чтения для потребителей (A-06, Б-05, Б-06; в A-02 её зовёт только маршрут `GET /dividends/{ticker}` — разрешённый задел блока).

**Тесты:** `backend/tests/unit/test_dividends/` — `test_load_tinvest.py`, `test_check_smartlab.py`, `test_smartlab_parse.py`, `test_smartlab_fetch.py`, `test_discrepancies.py`, `test_router.py`, `test_scheduler_jobs.py`, `test_canceled_migration.py`, `test_record_script.py`; фикстуры — настоящие ответы `GetDividends` и страницы Смартлаба PHOR и GAZP, записанные 2026-11-03 (`fixtures/`; скрипт `backend/scripts/record_a02_dividend_fixtures.py`); сеть в тестах закрыта. Приёмка PHOR (201 ₽, 05.07.2025: «нет в T-Invest» → после `add` выплата в `read_dividends`, расхождения нет) и GAZP (52,53 ₽, 20.07.2022: «нет в T-Invest» → после `exclude` нет). Frontend — vitest (`DividendsPage.test.tsx`, `TickerAliasesCard.test.tsx`, `BrokerAccountListMarketData.test.tsx`, `BrokerSettingsMarketDataWarning.test.tsx`) и E2E `frontend/e2e/a02-dividends-market-data-aliases.spec.ts` на моках API.

**Известные ограничения.** Вживую на тестовом стенде 07–09.11 — загрузка T-Invest (на `088b9e6` — 261 акция за 5 мин 7 с, отказов по лимиту нет), разбор настоящих страниц Смартлаба (GAZP — 10 пар и расхождение 52,53 ₽ с пометкой отмены), поправки «Исключить» (GAZP, AVAN), «Добавить» (AQUA) и «Удалить поправку», сохранность поправок при перезагрузке T-Invest; PHOR — по фикстурам (на стенде дойдёт до сверки 14.11; решение заказчика 09.11). Поправку с экрана можно внести только у строки с расхождением; без счёта, вне лет Смартлаба и при «сверка недоступна» — только через API. Без счёта «Итог» показывает суммы ранее загруженного T-Invest без пометки. Частичная отмена на Смартлабе в одну дату реестра — без пометки. Разбор держится на разметке сайта (атрибут таблицы). Ручного запуска загрузки и сверки по бумаге нет. Список — акции TQBR (`SHARE_SECTYPES`); у бумаг без выплат возможны пустые ответы, без FIGI — `dividends_figi_unresolved`. Дробление акций сверка не учитывает (тестовый стенд: BELU 1:8, 2024 — 11 ложных расхождений). Лимит «страница на бумагу в неделю» — на каждый экземпляр терминала: стенд и рабочий терминал сверяют каждый сам.

### 5.16 Новости: ИИ-разметка, алерт, метки, выключатели — `app/news/`, `app/ai/background.py` (v4.9, Б-07)

Требования — ФТ §20.8–§20.15; схема — §3.13, §3.21, §3.29; API — §4.17; фронтенд — §6.4, §6.7, §6.10. Решения — ADR 0060–0069 и 0105 в репозитории кода (`docs/adr/0060-b07-active-label-ai-else-dictionary.md` … `0069-b07-prompt-v3-dictionary-candidates-and-reported-cost.md`, `0105-dictionary-quotes-save-name-only-from-its-own-exclusion.md`); карта — `docs/architecture.md`, раздел «ИИ-разметка, алерт и метки (блок Б-07; ADR 0060–0069)». Модули торговли и брокера не импортируются (охранный тест Б-01 остался).

```
 проход news_collect (раз в 60 с)
   │  news_settings из БД: collect_enabled = false → выход до любого сетевого вызова (ADR 0065)
   ▼
 сбор → news_items → словарь: news_labels(dictionary) + news_tickers            ← хранится всегда, запасной путь
   ├─ шаг ИИ задачей (ai_labeling.run_ai_step): ворота disabled → no_provider → not_verified → limit
   │     └─ пачка ≤ 10 новых (подсказка b07-v3 с кандидатами словаря) → background.complete_json (≤ 20 с)
   │           ├─ ответ верен → news_labels(ai) + news_tickers + учёт ai_background_usage
   │           ├─ неверен / сбой → повтор до ~10 с от начала прохода, дальше по попытке за проход; 3 неудачи или 2 мин → ai_note
   │           └─ архивная пачка 10 (≤ 40 с): ИИ ответил, шаг ИИ ≤ 15 с, израсходовано < 80 % лимита
   └─ шаг алертов (alerts.run_alert_step): стадия 1 — по сроку ~10 с (asyncio.wait), стадия 2 и оставшееся — после ответа модели
         └─ alert_triggers + outbox news.alert → NotificationService → уведомление news_alert (приложение, Telegram)
 GET /news, /news/by-ticker → по действующей разметке → NewsPage, NewsMarkersLayer
```

**5.16.1 Фоновые задачи ИИ** (`app/ai/background.py`, методы `complete_json` в `app/ai/providers/*`; ADR 0061, 0069).
- **Провайдер:** владелец — первый администратор (`min(users.id) WHERE is_admin`); `resolve_background_provider` — его строка с `is_background`, иначе его активная (`source: flag | active`); нет — `None` (пометка `no_provider`). Ключ `.env` (`AI_API_KEY`) не берётся никогда. `open_background_provider` расшифровывает ключ строки только в памяти и создаёт провайдера через `ProviderFactory.create_async` (та же SSRF-проверка и закреплённый резолв, что у чата). `set_background_provider` — только на строке владельца (чужая — 404), аудит `ai.background_provider_selected`. Сменился первый администратор — фоновые задачи переходят на строки следующего.
- **Вызов** `complete_json(provider, system, user, schema, schema_name, timeout_s, max_tokens)`: OpenAI-совместимые — `response_format` `json_schema` со `strict: true`; у OpenRouter ещё `extra_body` `provider.require_parameters = true` и `reasoning.enabled = false` (D01: рассуждение не укладывается в срок); ответ 400 — один повтор без схемы (схема — в системной подсказке); Claude — схема только в подсказке. Клиент — `with_options(timeout=…, max_retries=0)` и `asyncio.timeout` на весь вызов с повтором. Итог — `JsonCompletion` (разобранный JSON, токены из `usage` — без `usage` оценка `count_tokens`, задержка, `cached_tokens`, `cost_usd`) или `BackgroundAIError` (`timeout` / `unavailable` / `invalid_json`). В журнал — только вид сбоя и класс исключения, без текста запроса и ответа.
- **Цена вызова** (доработка, ADR 0069): `providers/base.py::reported_cost_and_cache` читает у OpenRouter `usage.cost` (кредит = $) и `usage.prompt_tokens_details.cached_tokens`; `record_usage` берёт сообщённую цену, а если её нет (OpenAI, Ollama, ответ без `cost`) — `tokens × цена строки за 1 млн / 1 000 000`, `Decimal`, `quantize(1E-8)`. Прочитанное из кэша входит в токены лимита; отдельной колонки у него нет — оно в журнале пачки и записи прогона.
- **Учёт** — атомарный upsert строки `ai_background_usage` («сутки МСК + строка + модель + задача»: `news_labeling` / `news_benchmark`) на каждый вызов, и на неудачный тоже, если ответ был. **Лимит** — `budget_left = daily_token_limit − израсходовано за сутки МСК`, проверяет вызывающий перед вызовом; `set_daily_token_limit` — 0…`MAX_DAILY_TOKEN_LIMIT` (1 000 000 000), аудит `ai.background_limit_changed`. Кэша нет: провайдер и лимит читаются из БД каждый проход.

**5.16.2 ИИ-разметка** (`app/news/ai_labeling.py`; ADR 0062, 0069).
- `NewsService._label_new_items`: словарь → `asyncio.create_task(run_ai_step(...))` → `asyncio.wait(timeout=DICTIONARY_DECISION_DELAY)` → шаг алертов → дождаться шага ИИ → шаг алертов ещё раз, если ИИ ответил позже. Срок повторов в проходе — `started + AI_DECISION_SECONDS`. Список TQBR — `dictionary.tqbr_securities()` (общий кэш процесса с словарём; новый публичный доступ).
- **Ворота** `_gate`: `ai_labeling_enabled = false` → `disabled`; нет провайдера → `no_provider`; `ai_benchmark.is_verified` ложно → `not_verified`; `budget_left ≤ 0` → `limit`. Пометка пишется только новым новостям без пометки, полученным за `AI_GIVE_UP_AFTER` (2 мин); вызова модели нет. Лимит проверяется и перед каждым вызовом — кончился посреди прохода, оставшимся новым — `limit`.
- **Новые новости** — без разметки ИИ, `ai_attempts < AI_MAX_ATTEMPTS` (3), `received_at` за 2 мин, от новых к старым, пачками по `AI_BATCH_SIZE` = 10 (решение заказчика 11.11, было 20); `AI_CALL_TIMEOUT_SECONDS` = 20, `AI_MAX_TOKENS` = 4000, пауза перед повтором — 0,5 с. Неудача — `ai_attempts + 1`; 3 неудачи — `unavailable` (сеть, таймаут, HTTP) или `invalid` (ответ не прошёл проверку); прошло 2 мин после неудачи — `unavailable` (`_expire_overdue` в начале прохода, лог `news_ai_given_up`). Успех снимает пометку.
- **Запрос** — `build_messages`: системная часть — подсказка и список TQBR «тикер — короткое название» (от пачки не зависит, её кэширует провайдер); пользовательская — пронумерованные с нуля заголовки и под каждым «словарь: …» — тикеры разметки словарём этой новости («словарь: —» — нет). **`PROMPT_VERSION = "b07-v3"`** (`b07-v1` — до настоящего прогона, `b07-v2` — с 10.11, `b07-v3` — с доработки 12.11); смена текста или схемы — новая версия и новая проверка на эталоне. Ссылок и данных пользователя, счёта, позиций в запросе нет.
- **Ответ** — `{"items": [{index, scope, news_type, sentiment, importance, tickers}]}` (`SCHEMA_NAME = "news_labels"`, `strict`, тикеры — перечислением списка). `parse_response`: ровно по элементу на заголовок, `index` 0…n−1, без лишних и недостающих полей, `scope` — `SCOPES` или `null`, `news_type` — `NEWS_TYPES`, `sentiment` — `SENTIMENTS` (пишется только при `scope = "company"`), `importance` 1–3, тикеры — только из списка; охвата нет — бумаг нет; «компания» без бумаг — ответ неверный (D03 отменён). Любое нарушение — `InvalidAIResponse` на всю пачку; тикеры вне списка считаются (`out_of_list` в `news_ai_batch_failed`).
- **Запись** — одна транзакция на пачку после ответа (сеть вне транзакций): `NewsLabel(labeler="ai", model, prompt_version, scope, news_type, sentiment, importance)` + `news_tickers`, прежняя разметка ИИ перезаписывается, учёт вызова — в той же транзакции. Провайдер открывается один раз на проход.
- **Архив** — после новых, одна пачка `ARCHIVE_BATCH_SIZE` = 10 (старше 2 мин, без разметки ИИ, `ai_attempts < ARCHIVE_MAX_ATTEMPTS` = 5, от новых к старым), одна попытка со сроком `ARCHIVE_CALL_TIMEOUT_SECONDS` = 40, только если в проходе ИИ ответил, от начала шага ИИ прошло ≤ `ARCHIVE_START_WITHIN_SECONDS` = 15 (не от начала прохода: сбор при недоступном ЦБ сам идёт ~15 с; F2) и израсходовано < 80 % лимита (`ARCHIVE_RESERVE_SHARE` = 0,2). Стадию алертов архивная разметка не трогает.

**5.16.3 Проверка на эталоне** (`app/news/ai_benchmark.py`; ADR 0063).
- Эталон — `backend/app/news/data/d8_news_labels.json` (копия `d8_news_labels.json` документации байт в байт, тест сверяет sha256; каталог `data/` исключён из `.gitignore` явно).
- `run_benchmark` — та же подсказка, схема и `parse_response`, что у ленты; `BENCHMARK_BATCH_SIZE = AI_BATCH_SIZE` (10; 108 заголовков — 11 вызовов), `BENCHMARK_ATTEMPTS = AI_MAX_ATTEMPTS` (3 попытки на пачку; три неудачи — заголовки пачки «без ответа»), срок вызова `BENCHMARK_CALL_TIMEOUT_SECONDS` = 60, прогон — не дольше `BENCHMARK_DEADLINE_SECONDS` = 14 мин (иначе `error` «не уложился в срок»). Кандидаты словаря для `b07-v3` — `TickerDictionary` из того же списка TQBR и терминов БД.
- **«Пройдено»:** `precision_pct ≥ 95` (`PRECISION_MIN_PCT`), `recall_hits ≥ 35` (`RECALL_MIN_HITS`), `out_of_list = 0`, `CONTROL_NO_TICKERS = (19, 40)` без бумаг, `CONTROL_ABRD = 71` — ровно `{"ABRD"}`; контрольный засчитывается только с ответом модели. Тип и тональность — таблицы совпадений без порога.
- **Ворота:** `is_verified(info)` — последний прогон со `status = 'done'` связки (`config_id`, `model_id`, текущая `PROMPT_VERSION`) пройден; прогоны `error` вердикта не меняют. Его же зовёт `settings.ai_labeling_verified` (409 `benchmark_required`) и `_gate` разметки.
- **Запуск** — фоновая задача процесса (`asyncio`, множество `_TASKS`; один uvicorn-worker): `POST /news/ai-benchmark` → строка `running` → 202; `running` старше `BENCHMARK_STALE_AFTER` (15 мин) — `error` «Прогон прерван (перезапуск сервера)». Лимит дня проверку не блокирует, расход — задачей `news_benchmark`.
- **Настоящий прогон сборки** — `backend/scripts/record_b07_benchmark.py` (вне `app/`): копия рабочей БД через `sqlite3.Connection.backup`, ключ владельца расшифровывается в памяти; на рабочую БД и всё внутри `~/MoexTerminal` скрипт отказывает (код 2; путь из `make_url`, без учёта регистра, по inode). Ответы моделей — `backend/tests/test_news/fixtures/ai/<модель>.json` (параметры `batch_size` и `max_attempts`, тексты ответов, хэш запроса, токены, задержка); автотест воспроизводит записи `b07-v3` без сети; записи `b07-v2` — в `fixtures/ai/b07-v2/`, не воспроизводятся.

**5.16.4 Действующая разметка** (`queries.active_label_subquery()`; ADR 0060): id строки `news_labels` с `labeler = 'ai'`, иначе с `'dictionary'`. По ней — `tickers` и `reactions` ответа, фильтры `ticker` / `type` / `sentiment`, `GET /news/tickers`, `by-ticker` и алерты; `NewsItemOut.active` и `ai_note`. `ReactionWorker` не менялся — заводит реакции для пар всех разметок.

**5.16.5 Алерт `news`** (`app/news/alerts.py`; ADR 0064, 0067, 0068).
- **Создание** `create_news_alert`: `NewsAlertFilter` (`extra="forbid"`); `_clean_words` — пробелы схлопнуты, пустое — 422, длиннее `MAX_WORD_LENGTH` (100) — 422, дубли по `normalize_word` убраны, больше `MAX_WORDS` (50) — 422; типы и тональности — из `NEWS_TYPES` / `SENTIMENTS`; `tickers: null` без слов, типа и тональности — 422 `MARKET_NEEDS_FILTER`; бумаги — пересечение со списком TQBR (`securities is None` — `SecuritiesUnavailable` → 503). Строка `Alert(kind="news", ticker=None, news_filter=…, active_since=now)`.
- **Совпадение** `matches`: бумаги — пересечение с бумагами действующей разметки (`tickers: null` — любая новость, в том числе без бумаг); слова — `normalize_word` (`casefold`, `ё → е`, пробелы схлопнуты) и `word_in_title` = `re.search(r"(?<!\w)" + re.escape(word), title)` — **с начала слова** (доработка 12.11, ADR 0068; заменяет «подстроку» R35 и ADR 0064); типы и тональности — «любой из».
- **Стадии** `news_items.alert_stage`: `new` → (стадия 1) `partial` → (стадия 2) `done`; `skipped` — «Алерты» выключены. Стадия 1 — алерты без типа и тональности (`needs_ai` ложно): решение, когда есть разметка ИИ, или `ai_note` задан, или `now ≥ received_at + DICTIONARY_DECISION_DELAY` (10 с) — по действующей разметке, по словарю — `note = by_dictionary`. Стадия 2 — алерты с типом или тональностью: по разметке ИИ; `ai_note = disabled` — `unchecked` / `ai_disabled`; другой `ai_note` или `now ≥ received_at + TYPED_WAIT` (2 мин) — `unchecked` / `ai_unavailable`; «не проверено» — только если бумаги и слова совпали по словарю, без уведомления и без `trigger_count + 1`.
- **Кому положена новость:** алерт `is_active`, без `paused_at`, `received_at ≥ active_since` (или `created_at`) и `≥ news_settings.alerts_enabled_since`; `received_at − published_at > MAX_NEWS_AGE` (1 сутки) — новость проходит стадии без решений: ни уведомления, ни строки истории (решение заказчика 11.11, ADR 0067).
- **Запись и доставка** — одной транзакцией: условный `UPDATE news_items SET alert_stage` из ожидаемой стадии, `INSERT … ON CONFLICT (alert_id, news_id) DO NOTHING` в `alert_triggers` (`price` NULL), `last_triggered_at`, `trigger_count + 1`, `outbox.stage_for_delivery(channel="news_alerts:{user_id}", event="news.alert", data)`; после commit — `outbox.deliver_staged`. `EVENT_MAP["news.alert"]` → тип `news_alert`, severity `info`, заголовок «Новость по бумаге», текст `{subject}: {title}\n{source}, {published_msk} МСК{labeler_text}\n{url}` (`subject` — бумаги алерта ∩ бумаги разметки или «Рынок»; `labeler_text` — « · по словарю»); `SETTINGS_INHERIT_FROM["news_alert"] = "alert"`; почты нет (`EMAIL_ALLOWED_EVENTS` не менялся); `news.alert` — в `OUTBOX_REPLAY_EVENTS` (строку, не доставленную до падения процесса, доставит повтор при старте). Ключей `EVENT_MAP` стало 22. У уведомления нет связанной сущности — кнопки «Открыть график» в Telegram нет.
- **Пауза** `set_paused` — `paused_at`; снятие ставит `active_since = now`. **Удаление** — `is_active = false`. **История** — `list_triggers`, новые первыми, до `TRIGGERS_LIMIT` (100). Лимита на число алертов нет (развилка 10, A-11).

**5.16.6 Выключатели** (`app/news/settings.py`; ADR 0065).
- `read_news_settings(session)` → `NewsSettingsView`; строки нет — умолчания (загрузка, алерты, метки включены, ИИ выключен) и warning `news_settings_missing`. `NewsService._enabled_settings(job)` читает строку **в начале каждого прохода** `news_collect` и `news_reactions`, кэша нет; `collect_enabled = false` — `None`, проход выходит до сети; не прочиталась — `news_settings_read_failed`, проход пропускается. В журнал — только переход: `news_collect_skipped` (`reason=collect_disabled`) / `news_collect_resumed`.
- `ai_labeling_enabled` читает `_gate` шага ИИ; `alerts_enabled = false` — шаг алертов переводит новости `new` / `partial` в `skipped` (`news_alerts_skipped`); `chart_marks_enabled` читает только фронтенд.
- `PUT /news/settings` (`require_admin`): `NewsSettingsPatch` (`StrictBool`, `extra="forbid"`); включение `ai_labeling_enabled` без `ai_labeling_verified` — `NewsSettingsConflict` → 409 `benchmark_required`; включение `alerts_enabled` ставит `alerts_enabled_since = now`; изменение — аудит `news.settings_changed` (`before` / `after`), без изменений — без аудита. Уже включённую «ИИ-разметку» смена модели не сбрасывает.

**5.16.7 Журнал.** Разметка: `news_ai_batch_done` (`count`, `duration_ms`, токены, `cached_tokens`, `cost_usd`, `archive`), `news_ai_batch_failed` (`reason`, `out_of_list`), `news_ai_skipped` (`note`, `items`), `news_ai_given_up`, `news_ai_step_done`, `news_ai_step_failed`, `ai_background_open_failed`, `ai_background_call_failed`, `ai_json_schema_rejected`; проверка: `news_ai_benchmark_started` / `_done` / `_failed` / `_task_failed`; алерты: `news_alert_created`, `news_alert_decided`, `news_alert_step_done`, `news_alert_step_failed`, `news_alert_filter_invalid`, `news_alerts_skipped`; настройки: `news_settings_changed`, `news_settings_missing`, `news_settings_read_failed`, `news_collect_skipped`, `news_collect_resumed`. Заголовки и ответы модели в журнал не пишутся.

**5.16.8 Тесты.** `backend/tests/test_news/` — `test_ai_labeling`, `test_ai_benchmark`, `test_news_alerts`, `test_news_alert_cycle`, `test_news_alert_notification`, `test_settings`, `test_securities`, `test_api`, `test_dictionary`; подставной провайдер `ai_stub.py` с записанными ответами (`ai_helpers.py`), сеть закрыта; `tests/unit/test_ai/test_background.py`, `test_background_router.py`, `test_complete_json.py`; `tests/unit/test_record_b07_benchmark.py`; миграции — `tests/unit/test_migration.py`. Frontend — vitest (`NewsPage`, `NewsSettingsTab`, `AIBackgroundCard`, `NewsAlertModal`, `NewsAlertsPanel`, `NewsMarkersLayer`, `newsMarkers`, `NewsMarkerPrimitive`, `newsAlertStore`, `newsSettingsStore`) и E2E на моках `frontend/e2e/b07-news-feed.spec.ts`, `b07-news-alerts.spec.ts`, `b07-settings.spec.ts`, `b07-chart-news.spec.ts` (моки — `e2e/fixtures/news_mocks.ts`); `b01-news.spec.ts` изменён по ADR 0060.

**5.16.9 Проверка на тестовом стенде** (11–12.11, кандидат `5a49ea6`, копия рабочей БД, ИИ выключен, процесс не перезапускался) — сутки без ИИ **PASS WITH NOTES**: 1197 новостей, у всех `ai_note = disabled`, разметок `ai` — 0; стадия алертов — 1187 `done`, 10 `skipped` (окно «Алерты выкл»); 117 строк `sent` (`by_dictionary`) и 117 уведомлений `news_alert`, задержка получение → срабатывание — медиана 0,0 с, максимум 0,1 с; 73 строки `unchecked` / `ai_disabled` у алерта с типом «дивиденды»; выключатели («Загрузка» — 10 мин без новостей и реакций, одна строка `news_collect_skipped` на задание; «Алерты» — `skipped` и сдвиг `alerts_enabled_since`), метки SBER / GAZP 1m / 1h / D ✔; журнал — 0 Traceback, 0 error; ЦБ — `ReadTimeout` (прокси окружения процесса стенда). Правило `MAX_NEWS_AGE` вживую не сработало (максимум «публикация → получение» — 3 ч). Замечания 1–4 закрыты доработкой (`227df39`, `0644eb6`, `ae9d885`); доработку на стенде отдельно не проверяли. Проверки на эталоне — ФТ §20.10.

**5.16.10 Известные ограничения.**
- Срок вызова ленты — 20 с, эталона — 60 с: прошедшая эталон модель при всплеске новостей может получать `timeout` в ленте (на пачке 20 у `deepseek/deepseek-v4-pro` вызов шёл 12–29 с, на пачке 10 — 6–15 с).
- Неудача архивной пачки засчитывается всем 10 её новостям; проход с архивной пачкой длится до «сбор + ~15 + 40 с» — следующий запуск задания ждёт (`max_instances=1`); после первого прохода с 300+ новыми — около 33 вызовов подряд.
- `unavailable` после 2 мин не различает таймаут и неверный ответ (вид последней неудачи не хранится).
- Модели с обязательным рассуждением (Qwen3.8 Max — 400 на `reasoning.enabled = false`) проверку не проходят; `reasoning` шлётся только OpenRouter.
- Прогон проверки — состояние процесса: перезапуск обрывает его, строка `running` до 15 мин; худший прогон (11 × 3 × 60 с) не укладывается в 14 мин — `error`.
- `by-ticker` — первые 2000 по возрастанию (свежие теряются); после сбоя загрузки настроек метки не возвращаются сами (фронтенд).
- `DELETE /market-data/price-alerts/{id}` (A-03) удалит свой алерт `news` по id; у уведомления нет кнопки «Открыть график».
- Правка словаря, новые исключения и правило кавычек не переразмечают уже размеченные новости; спортивные обороты вне набора («матч с Акроном») дают AKRN.
- `GET /settings/ai/background`: `cost_usd` нулевого дня — строкой `"0E-8"`; у строк не-OpenRouter стоимость — по ценам строки, без скидки кэша.
- Комментарий модуля `app/ai/background.py` и докстринг `AIBackgroundUsage` после доработки по-прежнему говорят «стоимость — по ценам строки»; фактически — сообщённая цена, иначе по ценам (`record_usage`).

### 5.17 Договор исполнения — `app/execution/`, бэктест и Grid, paper, журнал, дивиденды, Sharpe (v4.10, A-06)

Требования — ФТ §22; схема — §3.6–§3.9, §3.30; API — §4.18; фронтенд — §6.5, §6.6. Как устроено — `docs/architecture.md` репозитория кода, раздел «Договор исполнения (блок A-06)»; термины — `CONTEXT.md`; решения — ADR 0070–0079 (`docs/adr/0070-backtest-execution-contract-next-open-oco-slip-out.md` … `0079-strategy-time-window-msk-and-legacy-backtests-marked.md`). Пути — от `backend/app/`. Брифинг: развилки 1–5 и вопрос 2 («текущая цена») — решения заказчика 2026-11-10.

```
 БЭКТЕСТ / GRID
 BacktestCreate / GridSearchRequest (DEFAULT_SLIPPAGE 0,1 %, DEFAULT_DIVIDEND_TAX_PCT 13 %)
   └▶ router: strategy_warmup_bars ─▶ BacktestEngine.run / _prepare_grid_workload
        ├─ _load_data (warmup_request_start) → warmup_slice: N баров до даты начала, trade_from = дата начала
        ├─ series_auction_marks (таблица периодов, exec_open D/W/M) · resolve_price_step
        ├─ series_dividends: read_dividends → last_buy_day → gap_bar_index → ((бар гэпа, сумма), …)
        └─ ContractParams(slippage, price_step, dividends, dividend_tax_pct, trade_from) ── payload ──▶ процесс исполнения
              build_cerebro: Cerebro(cheat_on_open) + AuctionAwareBroker(метки, coo, slip_out=True)
              сгенерированный next(): contract_request_entry(is_long) / self.close()
              WrappedStrategy.next_open: credit_bar_dividends → (бар прогрева — снять) → execute_entry_intent:
                 Market входа + Stop + Limit(oco=стоп); защитный ордер без позиции снимает брокер
   ◀─ annual_sharpe · price_pnl + dividend_pnl · B&H с дивидендами нетто · MCFTRR
   └▶ _attach_money_market (LQDT / RUONIA) ─▶ _save_result (execution_rules_version = 1) ─▶ _run_parity_check (тень)

 PAPER
 закрытый бар ТФ ─▶ страховка check_sl_tp (paper_bar_exit) ─▶ сигнал ─▶ CB (вход) ─▶ _PendingExecution(not_before, bar_open)
 market:{T}:1m ─▶ _IntrabarRiskWatcher(mode="paper"): remember_minute → accrue_session_dividends (новая дата МСК)
                  → _pending_minute_verdict → _execute_pending(ExecutionPrice(bar_open, open, начало минуты))
                  → paper_minute_exit → RiskMonitor._close_paper_exits (журнал, деньги, DailyStat, commit)
 «Закрыть» / «Стоп» / «Закрыть все» / «Стоп всему» / CB / список ─▶ _paper_current_price − проскальзывание (manual / forced)
```

**5.17.1 Пакет `app/execution/`** — чистые правила договора (ADR 0070–0075); БД и сети не знает, кроме `dividends.load_dividend_schedule`. Его зовут обёртка стратегии и тень бэктеста, Grid, paper, корпоративные действия, эталон и сценарии — своих формул у них нет.
- `contract.py`: `Slippage(value, mode: "pct" | "points")`, `Slippage.zero()`, `DEFAULT_SLIPPAGE` = 0,1 %, `DEFAULT_DIVIDEND_TAX_PCT` = 13, `EXECUTION_RULES_VERSION` = 1, `ExecutionRule` (`bar_open`, `signal`, `stop_level`, `take_level`, `gap`, `manual`, `forced`), `PRICE_QUANT` = 1e-8; `apply_slippage(price, side, slippage)` (покупка вверх, продажа вниз), `round_level_to_step(level, entry, step)` (к входу; `None` — как есть), `protective_levels(entry_fill, is_long, *, stop_pct, take_pct, stop_points, take_points, step)`, `touched_level(is_long, stop, take, low, high)` (оба → `stop`), `level_exit(kind, level, bar_open, is_long, slippage)` (open за уровнем → `gap`), `exit_side`.
- `dividends.py`: `T1_SETTLEMENT_SINCE` = 31.07.2023, `last_buy_day(record_date, is_trading_day)` (T+1, до 31.07.2023 — T+2; торговый день — `MOEXCalendarService.is_trading_day`, сессии выходного дня торговыми днями не считаются), `gap_bar_index(bar_msk_dates, last_buy)`, `next_session_date(last_buy, *, weekend_flag, schedule)` (для live; сессии за 30 дней нет — следующий календарный день), `dividend_cash(amount, units, is_long, tax_pct)`, `load_dividend_schedule(db, ticker, date_from, date_to) -> DividendSchedule(events, has_unresolved_discrepancies)` — `read_dividends`, только RUB; `last_buy_date` T-Invest после 31.07.2023 — только сверка (WARNING `dividend_last_buy_mismatch`, расчёт не меняет).
- `warmup.py`: `MIN_WARMUP_BARS` = 200, `WARMUP_PERIOD_FACTOR` = 5, `max_indicator_period(ir)` (умолчания периодов — как `evaluator`), `warmup_bars(ir)`, `warmup_request_start(timeframe, date_from, bars)` (запас на ночи, выходные, праздники; торговый день оценён снизу в 8 ч).

**5.17.2 Бэктест и Grid по договору** (ADR 0070; backtrader 1.9.78.123).
- `backtest/cerebro_setup.py::build_cerebro(*, data_df, strategy_cls, strategy_kwargs, initial_capital, commission_pct, contract, auction_marks)` — единственное `bt.Cerebro(` в `app/backtest`: `Cerebro(cheat_on_open=True)`, `AuctionAwareBroker` с `set_coo(True)` и `set_slippage_perc|fixed(…, slip_open=True, slip_limit=True, slip_match=True, slip_out=True)`. Его зовут `engine._setup_cerebro` и `grid._run_single_backtest`; Grid получает метки аукционов и `ContractParams` в payload тем же кодом, что одиночный бэктест (`router._prepare_grid_workload`). `ContractParams(slippage, price_step, trade_from, dividends, dividend_tax_pct)` — pickle в процесс исполнения.
- **Решение в `next()`, исполнение в обёртке.** Кодоген (`ir_codegen._gen_next`) ордеров не подаёт и размер не считает: вход — намерение `self.contract_request_entry(is_long=…)`, выход по сигналу — `self.close()`; параметры размера, уровней, лота и комиссии остаются в `params`. `WrappedStrategy.next_open` следующего бара (`engine.execute_entry_intent`, доверенный код): бар аукциона открытия — намерение ждёт; бар аукциона закрытия — снимается; цена исполнения — `exec_open` (D/W/M) или `open[0]` со сдвигом (`close/high/low` бара не читаются — тест по коду обёртки); лоты — `sizing.entry_lots` от `broker.get_cash()` и цены исполнения; `sizing.entry_affordable` (лонг и шорт, как paper) не прошёл — ни одного ордера; иначе Market входа, Stop, Limit (`oco=` стоп) — в этом порядке, уровни — `protective_levels` от цены исполнения с шагом. Роль ордера — `order.info["a06_role"]` (`entry` / `stop` / `take` / `forced`). Ордера договора подаются с `_checksubmit=False` (деньги уже проверены; псевдо-исполнение «вход → стоп → тейк» у шорта на все деньги снимало бы стоп через `oco`), проверка денег брокера при исполнении остаётся.
- `AuctionAwareBroker._try_exec`: защитный ордер без позиции (или в ту же сторону) — `cancel` + уведомление + `_ococheck`, не исполнение (дефект 10). `close()` обёртки снимает защитные ордера; принудительное закрытие в конце ряда не дублирует закрытие по сигналу того же бара.
- **Шаг цены** — `service.resolve_price_step`: `instrument_price_step` → `MarketDataService.ensure_lot_size` → без округления и строка `PRICE_STEP_MISSING_NOTE` в `report_notes`.
- `TradeRecord.exit_reason` (`signal` / `stop` / `take` / `forced`) — по роли закрывшего ордера; в БД не пишется (нужен эталону). Тейк — Limit backtrader ровно по уровню: известное отличие от договора (у paper — уровень со сдвигом).
- **Тень** `parity.interpreter_trades(ir, candles, …, *, timeframe, price_step, exec_open, trade_from, dividends, dividend_tax_pct)` повторяет каждое правило; сверка — по барам, как раньше (`compare_signals`, `fill_offset=1`), гейт live не менялся. «Ориентир P&L» сверки — без лотов, комиссий и проскальзывания и в гейт не входит.
- `sizing.PERCENT_LOT_EXPR` удалён: формула лотов — только функции `sizing` (`entry_lots`, `entry_affordable`), их зовут обёртка, тень и роутер (`_position_size_warning` проверяет шорт как лонг).

**5.17.3 Paper: ожидающее исполнение, наблюдатель минут, стоп и тейк, текущая цена** (ADR 0071, 0073).
- `trading/runtime.py`: `_PendingExecution(signal, kind: entry | exit, bar_ts, not_before, rule, reason, cb_checked)` заменил `_DeferredEntry` A-03 (у sandbox/real — тот же отложенный вход, правило `signal`). У paper все исполнения по сигналу — ожидающие, `not_before` = начало следующего бара ТФ (`next_period_start`); вход, отложенный A-03 до торгов, — `max(начало торгов, начало следующего бара)` с проверкой CB на исполнении (обычный вход проверяется CB на закрытии бара; выход CB не проходит). Одно на сессию, хранится у наблюдателя: новый сигнал заменяет; «Стоп», пауза, автопауза с возобновлением снимают; закрытый бар ТФ с началом ≥ `not_before` без исполнения — снимает (кроме бара аукциона открытия — перенос на торги); при полном лимите позиций вход отбрасывается сразу; перезапуск теряет.
- `SessionRuntime.start` поднимает `_IntrabarRiskWatcher(mode="paper")` и подписку `market:{ticker}:1m` под держателем `session:{id}` (если ТФ ≠ 1m) — как у sandbox/real; дилерские минуты paper не слушает. Наблюдатель: `remember_minute` (`trading/minute_prices.py`); на первой минуте новой даты МСК и после старта — `CorporateActionService.accrue_session_dividends(session_id, on_date)`; ожидающее — `_pending_minute_verdict` (метки 1m таблицы периодов, как у бэктеста: аукцион открытия — ждать, закрытия — выход исполнить, вход снять; сбой расписания — ждать) → `_execute_pending` → `OrderManager.process_signal` / `close_position(…, execution=ExecutionPrice(bar_open, open минуты, её начало))`; бар, закрытый событием уже после первой минуты нового бара, исполняется по её open сразу при регистрации.
- **Цена исполнения** — `execution_journal.session_fill_price` = `apply_slippage(rule_price, сторона, проскальзывание сессии)`; лоты, `entry_price`, `volume_rub`, комиссия, деньги и проверка денег — от неё; нет денег — `rejected`, уровней и строки журнала нет. `entry_signal_price` — close сигнального бара, `exit_signal_price` — цена правила; `LiveTrade.slippage` — значение сессии. SL/TP — `calc_sl_tp_prices(…, price_step=)` через `protective_levels` (paper — с шагом справочника, sandbox/real — без шага, как раньше).
- **Стоп и тейк каждой минутой** — `risk_monitor.paper_minute_exit` по снимку уровней в памяти (перечитка раз в 5 с и по событиям сделок): вся минута, в том числе минута входа (по `opened_at`); `touched_level` / `level_exit`; минута аукциона открытия — ничего, первая минута торгов — проверка open. Фаза — `_auto_exit_verdict(STOP|TAKE)`; запрещено (кроме аукциона открытия) — `pending_exits.enqueue` с видом стоп/тейк. Закрытие — `RiskMonitor._close_paper_exits`: `close_trade` по возрастанию id → `paper_portfolio_section` → перечитка сделки → `_apply_close` по цене исполнения → строка журнала → `_settle_paper_close` → `DailyStat` → `trade.closed` в outbox → commit; пока закрытие идёт — `_INTRABAR_EXIT_REQUESTED`. Страховка закрытого бара — прежнее решение `paper_bar_levels` с ценами договора (`paper_bar_exit`) и тем же путём; сделку, закрытую наблюдателем, не трогает. Причина `trade.closed` — из решения; `_sl_tp_reason` удалён.
- **Текущая цена ручных и аварийных закрытий paper** — `OrderManager._paper_current_price(ticker, session, *, entry_price)` → `(цена, источник)`: ~~`latest_minute_price(ticker, now=, max_age_sec=120)` (по началу минуты) → `minute`; `last_cached_bar(…, "1m")` → `cache_1m`; `_market_price_or_none` (close бара ТФ) → `cache_timeframe`; цена входа → `entry` (лог `close_position_no_market_price`).~~ С PR #139 (17.11, мерж-коммит `7ed702c`; дополнения ADR 0072/0073) — общая цепочка `OrderManager._market_price_chain(ticker, timeframe, mode)` → `(цена, источник) | None`: `latest_minute_price(ticker, now=)` не старше 2 мин → `minute`; close последней 1m кэша (`_cached_close(…, "1m", …)`, бывший `_market_price_or_none`) → `cache_1m`, если её время не раньше конца последнего закрытого бара ТФ сессии (`next_period_start(bar_ts, timeframe)`); иначе close бара ТФ → `cache_timeframe`. Условие звена `cache_1m` — для D/W/M (бар читается из дневок, старая 1m, в том числе ISS-минута до шва 2018 года, перебивала вчерашнее закрытие); у 1m–4h бар собран из тех же 1m. `_paper_current_price` при `None` — цена входа → `entry` (лог `close_position_no_market_price`). Sandbox/real — `_broker_market_estimate(ticker, timeframe, mode)` → `Decimal | None`: та же цепочка без цены входа; от неё — цена правила `manual` / `forced` журнала (`broker_exit_rule_price`), `exit_signal_price` выхода без цены брокера (S8R-FIX-063) и сверка запасной цены ответа брокера (`response_fill_price`); деньги и P&L — по цене брокера. Гейт гонок закрытия — `test_close_position_concurrency`. Цена исполнения — со сдвигом против позиции; правило — `execution_journal.rule_for_close(reason, exit_kind)`: `MANUAL` → `manual`, `FORCED` → `forced`, список с видом STOP/TAKE/SIGNAL — свой, без вида — по причине.
- Расчёты T+1 не моделируются; из `PaperBrokerAdapter.place_order` убраны ветка T+1 и `DEFAULT_SLIPPAGE_PCT`, метод остался (дефект 14).

**5.17.4 Проскальзывание сессии, умолчание окна и спред** (ADR 0073).
- `TradingSession.slippage_value/slippage_mode/dividend_tax_pct` (§3.8). `TradingService.start_session`: paper — поля запроса или `backtest/session_defaults.py::session_slippage_default(db, user_id, strategy_version_id, ticker, timeframe) -> SlippageDefault(slippage, source, backtest_id)` (последний `completed` бэктест своей версии на каноническом тикере и ТФ, `completed_at`, `id` по убыванию; иначе `DEFAULT_SLIPPAGE`) и налог 13; sandbox/real — 0 / `pct` и 13. Маршрут `GET /backtest/session-defaults` — то же правило.
- `backtest/spread_check.py::check_spread(db, user_id, ticker, slippage, *, now=None) -> SpreadCheck(status, half_spread_pct, recommended_min_pct, reason)`: счёт `select_market_data_account` → фаза `trading_hours.market_phase` (стакан — только основные торги и вечер) → FIGI токеном того же счёта (`ensure_figi_lookup`) → `get_trading_status` (`normal_trading`) → `TInvestAdapter.get_order_book_top(instrument_id, *, depth=1)` (новый унарный `GetOrderBook`, чтение через `_read_with_retry`, пустая сторона — `None`). Половина спреда — `(ask − bid) / 2` в % от середины; пункты переводятся в % от той же середины; больше проскальзывания — `warning` с рекомендацией вверх до 0,01 %. Вся цепочка — `SPREAD_CHECK_TIMEOUT_SEC` = 5 с; адаптер — как `broker_context` (токен в памяти, `disconnect` всегда, в журнал не пишется). Логи `spread_check_timeout`, `spread_check_failed`. Квота `MARKET_DATA` счёта данных — два унарных вызова на проверку.

**5.17.5 Журнал исполнений** (ADR 0072; таблица — §3.30).
- `trading/execution_journal.py`: `ExecutionPrice(rule, rule_price, at)`, `session_slippage`, `session_fill_price`, `record_execution(db, *, trade, session, side, lots, rule, rule_price, contract_price, actual_price, executed_at, partial=False)` — только `db.add`, без flush и commit: строка ложится той же фиксацией и под тем же локом, что сделка (сбой commit — ни сделки, ни строки); `record_broker_entry`, `broker_executed_at`, `increment_fill_price`, `broker_exit_rule_price`, `fill_entry_actual_price`, `rule_for_close`.
- **Paper:** вход — в `_fill_paper_trade_locked` внутри `paper_portfolio_section` перед единственной фиксацией (S8R-AUDIT-076); выход по сигналу — в `_settle_close_locked` после `_apply_close`; стоп/тейк — в `_close_paper_exits`. `contract_price = actual_price`, разница 0.
- **Sandbox/real** (поведение ордеров не меняется, gotcha-56): вход — правило `signal`, цена правила и договорная — `entry_signal_price`, фактическая — `order_fill_price`, момент — `OrderStatus.executed_at`; точки — обе ветки `filled` в `OrderManager._apply_entry_response` перед их commit. Вход без цены брокера — `actual_price` и разница `NULL`; цена нашлась дозапросом при закрытии — та же строка дописывается в транзакции закрытия. Довыполнение остатка (`_extend_partial_entry`) — строка `partial=True` с ценой приращения. Выход — `_settle_close_locked` (единая точка `close_position` и доучёта `finalize_exit_order`): правило — `rule_for_close`, у доучёта без вида — вид из списка (`exit_retry_kind`), иначе по причине; цена правила — уровень (стоп/тейк), цена сигнала, оценка рынка (`exit_estimate`) у ручного и аварийного. Закрытия без ордера (вне терминала, отмена `pending`) строк не дают.
- Чтение — `TradingService.get_trade_executions(session_id, trade_id, user_id)` → маршрут §4.18. Новый путь исполнения обязан писать строку в своей транзакции.

**5.17.6 Дивиденды** (ADR 0074).
- **Бэктест и Grid:** основной процесс (`engine.series_dividends(ticker, bar_starts, db)`) берёт `load_dividend_schedule`, находит бар гэпа (`gap_bar_index`) и передаёт `((бар, сумма брутто), …)` в `ContractParams.dividends` вместе с `dividend_tax_pct`; сбой чтения — бэктест без дивидендов и строка `report_notes` (WARNING `backtest_dividends_unavailable`), неразобранное расхождение (`has_unresolved_discrepancies`) — строка `report_notes`. Обёртка в `next_open` бара гэпа до исполнения намерения (`engine.credit_bar_dividends`) берёт позицию до исполнений бара и зовёт `broker.add_cash(dividend_cash(…))` — лонг × (1 − налог), шорт − полная сумма; запись `DividendRecord(bar_time, direction, lots, amount_per_share, cash)`. Тень зачисляет те же деньги (меняют размер «%» и проверку денег следующих входов).
- **Отчёт:** `price_pnl + dividend_pnl = net_profit`; начисления — `backtest_trades` с `row_kind = 'dividend'` (§3.7), в тень и маркеры не идут. **Эталоны** (`backtest/benchmark.py`): «купить и держать» — (close_last + Σ нетто) / close_first − 1, выплата бара 0 держателю не причитается; эталон-индекс — `benchmark_index_code(execution_rules_version)`: MCFTRR (ISS, рынок `index`, `client.INDEX_TICKERS`) при версии ≥ 1, IMOEX у старых; дата дневки индекса — дата МСК метки.
- **Paper и учёт sandbox/real** (`corporate_actions/service.py`): детект берёт выплаты `read_dividends` через `load_dividend_schedule` (окно дат реестра — от первой сделки по тикеру до года вперёд), `CorporateAction` — как раньше (ключ — дата реестра, сумма брутто); `_fetch_tinvest_dividends` (`GetDividends` токеном владельца) удалён. Начисление — не раньше дня гэпа (`accrual_gap_date`: первая сессия после последнего дня покупки, `next_session_date`), позиции — `accrual_position_clauses` с `last_buy_day`; лонг — `dividend_cash(…, tax_pct=session.dividend_tax_pct)`, шорт — полный; paper — баланс портфеля; все режимы — `DailyStat` дня гэпа через `upsert_daily_stat`. Зачисляют рантайм paper (`accrue_session_dividends`, §5.17.3) и задача корпоративных действий (раз в 6 ч, `process_pending` → `process_dividend`) — днём гэпа то, что сессии не зачислили; повтор исключает `dividend_accruals` (UNIQUE) и локи сессий; `processed` — когда незачисленных позиций не осталось. Действие с зачислениями сверка с источником не снимает. Сверка не удаляет непроведённые выплаты бумаги, которую A-02 ещё ни разу не загружал (ремонт F1). Купоны — прежнее правило T+1 (`accrual_last_buy_date`), сплиты — как были.
- Без счёта для рыночных данных выплаты T-Invest пусты: в бэктесте, paper и учёте — только поправки. Тикер, который загрузка A-02 ни разу не загружала, в детект попадает (поправки пишутся), но его непроведённые строки не сверяются — WARNING `dividends_tinvest_not_loaded` раз на прогон.

**5.17.7 Прогрев** (ADR 0075; развилка 1 — прогрев в A-06, A-07 файлов `backtest/*` и `trading/*` не трогает).
- Роутер считает `engine.strategy_warmup_bars(ir, params)` (параметры кодогена `<var>_period`, `_fast`, `_slow`, `_signal`, `_k`, `_d`; Grid — по наибольшему значению диапазонов) и передаёт в `BacktestEngine.run(…, warmup_bars=)` / `_prepare_grid_workload(…, warmup_bars=)`. `_load_data` запрашивает окно от `warmup_request_start`, `warmup_slice` оставляет N баров до даты начала; меньше — `warmup_note` в `report_notes` («прогрев неполный: N из M баров»); за период теста баров нет — «Нет рыночных данных». Синхронная загрузка T-Invest — по-прежнему не больше года (A-02).
- `ContractParams.trade_from` (наивное UTC): на баре раньше него `next_open` намерение снимает; решение последнего бара прогрева исполняется на первом баре теста. Кривая капитала, B&H, индекс, Sharpe и «баров в году» — по барам [начало, конец); свечи ответа API — без прогрева. Тень идёт по тому же ряду (`BacktestResult.candle_series`, `trade_from` — `exclude=True`, наружу не отдаётся).
- **Live:** история сессии — `warmup_bars(ir)` баров вместо постоянных 200 (`DEFAULT_HISTORY_SIZE` = 200 остался запасным значением, лог `session_runtime_history_size_fallback`): `SessionRuntime._preload_history(ticker, tf, user_id, *, bars)` расширяет окно запроса (`PRELOAD_HISTORY_ATTEMPTS` = 4, окно ×2), `_append_history(…, size=)` держит ряд этой длины; не набрал — WARNING `session_runtime_history_warmup_incomplete`.

**5.17.8 Годовой Sharpe и денежный рынок** (ADR 0078; дефект 12).
- `backtest/metrics.py`: `bars_per_year(bar_times, date_from, date_to)` = баров в [начало, конец) ÷ ((конец − начало) в днях / 365,25), два знака; `annual_sharpe(values, bar_times, date_from, date_to, *, initial_value=None) -> SharpeResult(ratio, bars_per_year, unreliable)` — доходности капитала по барам [начало, конец), среднее / σ генеральной совокупности, ставка 0, × √(баров в году) (эквивалентно `SharpeRatio(annualize=True, riskfreerate=0)` с этим фактором — тест до 1e-6); `sharpe_unreliable` — короче 3 календарных месяцев (не хранится). `calculate_metrics` принимает готовый `SharpeResult`. Значения капитала — `EquityCurveObserver` (бэктест) и `grid.BarValues` (Grid; границы теста — в payload). Удалены `bt.analyzers.SharpeRatio`, `sharpe_params`, `sharpe_analyzer_kwargs`, `TRADING_DAYS_PER_YEAR`, `TRADING_MINUTES_PER_DAY` (805). `sharpe_basis` ответа — `bars` / `legacy_constants`.
- `backtest/money_market.py::money_market_return(db, d1, d2) -> MoneyMarketReturn(pct, source)`: LQDT — `MOEXISSClient.get_board_history(ticker, board, from_, till)` (новый публичный метод; `LEGALCLOSEPRICE`, без него `CLOSE`), доска TQTF по 19.06.2026 и TQBR с 22.06.2026 (`LQDT_TQBR_SINCE`); до 22.07.2022 (`LQDT_FIRST_DAY`) — RUONIA: `backtest/cbr_ruonia.py::fetch_ruonia` — SOAP-метод `Ruonia` сервиса `https://www.cbr.ru/DailyInfoWebServ/DailyInfo.asmx` (GET сервис не принимает), `httpx` с прокси окружения, таймаут чтения 20 с; накопление `Π(1 + r/100/365)` по календарным дням (в выходные — ставка пятницы); через 22.07.2022 — `RUONIA+LQDT`. Кэш — `money_market_quotes` (§3.30). Общий срок — `MONEY_MARKET_TIMEOUT_SEC` = 60 с; нет данных или сбой — `pct=None` (WARNING `money_market_unavailable`, `cbr_ruonia_request_failed` / `cbr_ruonia_bad_status`), бэктест завершается как обычно. Считается по завершении бэктеста в `router._run_backtest_task` (`_attach_money_market`, своя сессия БД, кэш коммитится отдельно от результата); вердикт `money_market_verdict`: `net_profit_pct` больше — `better`, иначе `worse`. Фикстуры ISS и ЦБ пишет `backend/scripts/record_a06_fixtures.py`, тесты в сеть не ходят.
- Sharpe в статистике paper-сессии (`SessionStatsResponse`) не добавлен — развилка 5, с C3 (A-12).

**5.17.9 Окно стратегии в МСК и бэктесты до договора** (ADR 0079).
- Одно правило: время начала бара в МСК (фиксированное UTC+3, `common/timezones.MSK`) сравнивается с окном включительно. `evaluator._in_time_window`: aware — `astimezone(MSK)`, наивное (UTC) — `+3 ч`; `parity._in_window` зовёт его же; кодоген — `(self.data.datetime.datetime(0) + timedelta(hours=3)).time()`. Тест — `tests/unit/test_strategy/test_a06_time_window_msk.py`.
- `backtests.execution_rules_version`: NULL — «до договора исполнения» (миграция ничего не пересчитывает и не удаляет), новые — ORM-дефолт `EXECUTION_RULES_VERSION`, `BacktestService._save_result` пишет её ещё раз (запись из очереди до выкатки, посчитанная новым движком, — по договору). Признак `legacy_execution_rules` — в ответе и списке; удаление — `DELETE /backtest/legacy` (§4.18). Старые бэктесты остаются гейтом live (parity), пока не удалены. Новое правило договора — новая `EXECUTION_RULES_VERSION`.

**5.17.10 Сценарии договора и эталон B14** (ADR 0076, 0077).
- Сценарии — `backend/tests/fixtures/execution_contract/*.json` (формат `a06-execution-contract/1`: бары с метками аукционов, параметры, сигналы по номерам баров, выплаты, ожидаемые сделки и отказы, у тейка — `backtrader_expected`); 11 файлов `01_gap_through_stop` … `11_dividend_gap_bar`. Прогоны — `backend/tests/execution_contract/scenarios.py`: `run_backtest` (процесс исполнения бэктеста со стратегией-заглушкой формы кодогена), `run_paper` (сессия paper и настоящий `SessionRuntime`, бары — синтетический поток минут, дивиденд — `accrue_session_dividends`), `mismatches`; тесты `test_contract_backtest.py`, `test_contract_paper.py`; мутации договора их роняют.
- Эталон — `backend/scripts/record_b14_etalon.py` (`record` — по копии БД или стенду, без сети, бары только из `ohlcv_cache`, на рабочей БД — отказ, код 2; `replay` — по файлам, без БД; `check A B` — побайтно и инварианты; коды 0 / 1 / 2). Каталог формата `b14-etalon/1`: `manifest.json`, `versions/v<id>.json`, `bars/<тикер>_<ТФ>_<с>_<по>.csv` (с прогревом, метками аукционов и `exec_open`), `dividends/<тикер>.json` (с посчитанным `last_buy_day`), `price_steps.json`; JSON — ключи по алфавиту, отступ 2. Запись = выгрузка входных файлов + `replay` того же каталога. С `14d76e3` (находка стенда) `record` берёт торговые дни для последнего дня покупки из таблицы периодов БД (вечер или аукцион закрытия будня — торговый день; торги выходного и праздничного дня, 24–30.03.2022 и даты вне таблицы — календарь процесса): на копии стенда со всеми выплатами T+1 (540) расхождений с T-Invest 0, у календаря ISS — 3. В `backend/tests/fixtures/b14_etalon/` пока — маленький образец тестовой БД (`tests/unit/test_backtest/test_b14_etalon.py`); эталон стенда (16.11: 99 версий из 106, 36 215 сделок, ≈ 27 МБ, в git ≈ 2,9 МиБ) придёт отдельным PR (ветка `test/008-b14-etalon`; образец переезжает в `b14_etalon_sample/`).

**5.17.11 Гейты, тестовый стенд и известные ограничения.**
- **Сборка** (10.11, `b698a60`; baseline `92eb2ee`): pytest `DEBUG=true` — 8353 passed / 1 skipped (было 8048 / 1); режим CI — 8353 / 1, покрытие 92,87 %; ruff, mypy (269 файлов), bandit (Medium/High 0) — чисто; `alembic upgrade head` + `check` на чистой БД — чисто; фронт — typecheck, build, lint 0 / 0 / 0, vitest 2419 (273 файла; было 2364); E2E полным набором — 279 passed / 1 skipped, A-06 — 9 из 9. Слепая приёмка из чистого клона — 36 из 41 пункта «реализовано», 4 — «только на стенде», 1 — «частично» (тень сверяется на случаях B3 через кодоген, но не прогоняется на самих файлах сценариев). Код-ревью — 10 «чинить сейчас» исправлены ремонтом F1, 19 мелких — в «Известных багах» `.autopilot/PROGRESS.md`.
- **Перед мержем** (13.11, локальные гейты — CI GitHub недоступен): на `06a231f` — pytest в режиме CI 8623 passed / 1 skipped, покрытие 93,05 %; ruff, mypy, `alembic check`, bandit — чисто; фронт — lint, typecheck, build, smoke `dist`, vitest 2544; E2E — 298 passed / 1 skipped (на `b168a2d`, код приложения после не менялся). На голове PR `14d76e3` (правка скрипта эталона и три его теста, код приложения не менялся) гейты backend повторены — pytest в режиме CI 8626 passed / 1 skipped, покрытие 93,06 % (комментарий к PR #131).
- **Тестовый стенд 15–17.11** (кандидат `06a231f`, копия рабочей БД, `NO_PROXY` с `www.cbr.ru`) — **PASS WITH NOTES** (17.11 00:05 МСК): 1211 из 1213 сделок четырёх paper-сессий SBER 1m/5m за 16.11 совпали с бэктестом той же версии побайтно по минутам и ценам до 5 знаков (журнал — по две строки на сделку, цена по договору = фактической); sandbox #3 — 8 ордеров, журнал `signal` + цена брокера, `risk_monitor_intrabar_touch` 3; Sharpe 1h ↔ D — 3,1 % / 1,6 % с выплатой; «купить и держать» SBER — +37,94 %, три строки `row_kind = dividend`, `dividend_last_buy_mismatch` нет; денежный рынок — `RUONIA`, `RUONIA+LQDT`, `LQDT`; спред — `ok` / `warning` / `unchecked`; эталон — `record` / `replay` / `check` совпали.
- **Известные ограничения** (замечания стенда; в A-06 не правятся):
  1. Предупреждения «стоп ≤ проскальзывания» нет: стоп 0,1 % при 0,1 % после округления к шагу встаёт на open — выход каждым баром (ADR 0070–0071, следствие правил).
  2. ~~Цена правила `manual` / `forced` у sandbox/real — `_market_price_or_none` = последний бар 1m кэша, а биржевые минуты потока в кэш не пишутся (только дилерские) — оценка устаревает (на стенде на 25 мин). Деньги и P&L не задеты; задеты строка журнала и `exit_signal_price`, когда брокер не вернул цену. Отдельная правка — PR #139 (ветка `fix/008-broker-exit-estimate`, на 17.11 не влит): общая цепочка `OrderManager._market_price_chain` (минута ≤ 2 мин → кэш 1m, если минута не раньше конца закрытого бара ТФ, → кэш ТФ), у sandbox/real — без цены входа (`_broker_market_estimate`); своего ADR нет — дополнения ADR 0072 и 0073 от 17.11.~~ **Исправлено 17.11** — PR #139 (`7ed702c`), §5.17 «Текущая цена ручных и аварийных закрытий»: `_market_price_chain` / `_broker_market_estimate`. Гейты на голове `1b0f403` (после слияния с `develop` `ede13c3`): pytest CI-режим 8656 passed / 1 skipped, 93,08 %; ruff, mypy, bandit, alembic check (голова `8bddf72586cf`); фронт 2544. Живой сделки на sandbox #3 не было.
  3. После обрыва потока `_PendingExecution` исполняется по первой пришедшей минуте, хотя `stream_backfilled` через секунду приносит минуту `not_before` (5 сделок из 1206 у бэктеста v102 за день, ≈ 0,06 % в цене входа) — на будущее A-07 / поток.
  4. ~~`MOEXCalendarService` (календарь ISS) не знает праздничной сессии 12.06.2026: `last_buy_day` у выплат с реестром 15.06.2026 (GEMA, LVHK, RGSS) расходится с T-Invest — отдельная правка календаря по таблице периодов (ADR 0110).~~ **Исправлено 17.11** — PR #140 (`ede13c3`, ADR 0110), §5.9: календарь сначала спрашивает таблицу периодов; сверка кодом приложения на копии БД стенда — 541 выплата T+1, 0 расхождений с T-Invest (было 3), `load_dividend_schedule` по 540 событиям — 0 `dividend_last_buy_mismatch`.
  - Из «Известных багов» сборки: ожидающее, принятое позже первой минуты нового бара, идёт по open последней минуты, минута входа для уровней — по `opened_at`, снятие за целый бар без исполнения — лог info (нужен warning); стоп paper в запрещённой фазе закрывается из списка по текущей цене, а не по open первой разрешённой минуты; журнал sandbox/real пишет выход на все лоты входа при частичном встречном ордере, `executed_at` выхода — момент записи, доученный без вида выход — `manual` без цены правила; задача начислений берёт признак торгов выходного дня из кэша процесса (`market_data._cached_weekend_flag`, без кэша — понедельник вместо субботы гэпа); `round_level_to_step` может перенести уровень за цену входа, если вход не на сетке, а расстояние меньше шага; Grid не показывает `report_notes`; денежный рынок не запоминает сбой ЦБ (до ~25 с на бэктест раньше 22.07.2022); откат миграции оставляет строки начислений в `backtest_trades`; `take_no_slippage` эталона стоит и у тейка гэпом, `record` без единой снятой версии возвращает 0 (ADR 0077 — 1); фронт — журнал до 4 знаков, «Запустить заново» старой paper-сессии копирует 0/0, ответ умолчаний после смены тикера/ТФ перезаписывает налог, введённый руками, «Запустить» быстрее ~1 с после смены ТФ уходит с прежним проскальзыванием, после одиночного удаления число «старых» не перечитывается, запуск из ячейки тепловой карты берёт значения сетки из localStorage; `docs/architecture.md` кода — строка про удалённую константу 805.

### 5.18 Скринер — `app/screener/` (v4.11, Б-03)

Требования — ФТ §23; схема — §3.31, API — §4.19, задание — §5.9, страницы — §6.12. Как устроено — `docs/architecture.md` репозитория кода, раздел «Скринер (блок Б-03)»; термины — `CONTEXT.md` («Вселенная скринера», «Ликвидная бумага», «Снимок рынка», «Поправка сектора», «Шкала карты»); решения — ADR 0090–0095 (`docs/adr/0090-screener-universe-and-broker-status.md` … `0095-screener-liquidity-threshold-per-user.md`). Брифинг: восемь развилок — решения заказчика 2026-11-16, все по рекомендации; макеты — «Скринер Б», «Карта А».

```
 ночью 01:30 МСК и через минуту после старта (screener_nightly → jobs.run_screener_pass)
   1 статистика из БД ─▶ 2 ждёт фон A-02 ─▶ 3 дозагрузка дат ─▶ 4 синхронизация ─▶ 5 первое заполнение ─▶ 6 статистика
   ISS доска TQBR (EQIN) + analytics/IMOEX ──▶ universe.sync_universe ──▶ instruments (in_screener_universe, short_name, list_level)
   T-Invest Shares (1 вызов, счёт для рыночных данных) ──┘              + broker_* и sector
   ISS history (по дате / по бумаге) ──▶ history.* ──▶ screener_iss_day_totals ──▶ MarketStats в памяти (медиана 60, шкала P90 за 250)

 каждый опрос страницы (5 с — live, 15 с — задержка; только пока вкладка видна)
   GET /screener/snapshot ──▶ snapshot.build_snapshot
      доска ISS (кэш процесса 15 с, iss_lock) ─┐   цена, Δ дня, оборот, капитализация
      GetLastPrices (кэш 5 с на пользователя,  ├─▶ живая цена — только «торгуется», ключ счёта пользователя,
        адаптер T-Invest на счёт в памяти)     │   и только текущего торгового дня ISS (иначе — ISS с пометкой)
      instruments + screener_sector_overrides ─┤   сектор: поправка > брокер > «без сектора»
      MarketStats + screener_user_settings ────┘   liquid = медиана ≥ порог (null, пока история не набрана)
```

**Файлы:** `universe.py` (вселенная, `classify_broker`, синхронизация, кэш IMOEX), `history.py` (итоги дня, первое заполнение, дозагрузка, `compute_stats`), `snapshot.py` (снимок, кэши, адаптеры), `sectors.py` (`SECTOR_NAMES`, `resolve_sector`), `overrides.py` (поправки и аудит), `user_settings.py` (порог), `iss_gate.py` (замок ISS модуля, ожидание фона A-02), `jobs.py` (проход), `models.py`, `schemas.py`, `router.py`, `admin_router.py`. Общие файлы — точечно: `main.py` (импорт моделей, роутер), `admin/router.py` (подключение роутера поправок), `scheduler/service.py` (задание), `common/audit.py` (два действия), `market_data/models.py` (колонки `Instrument`); новые методы источников, прежние не менялись: `MOEXISSClient.get_board_snapshot`, `get_board_day_totals`, `get_ticker_day_totals`, `get_index_weights`; `TInvestAdapter.get_shares` (квота `INSTRUMENTS`, `Shares(INSTRUMENT_STATUS_ALL)`) и пакетный `get_last_prices` (квота `MARKET_DATA`, `LAST_PRICE_EXCHANGE`, один запрос на список), оба — `_read_with_retry`; `get_last_price` алертов не тронут. Настроек в `config.py` нет:

| Константа | Значение | Что |
|---|---|---|
| `universe.BOARD` / `UNIVERSE_INSTR_ID` | `TQBR` / `EQIN` | вселенная — акции доски (паи фондов `IFA1`, `IFTF`, `IFAY` — нет) |
| `universe.BREAK_TRADING_STATUS` | `break_in_trading` | единственный статус торгов записи TQBR, который значит «перерыв» (D01) |
| `universe.IMOEX_TTL_SEC` / `IMOEX_RETRY_SEC` | 24 ч / 15 мин | кэш состава IMOEX; после сбоя — не чаще |
| `history.TURNOVER_SESSIONS` | 60 | медиана `VALUE` — по последним 60 датам таблицы |
| `history.SCALE_SESSIONS` / `SCALE_PERCENTILE` | 250 / 90 | шкала карты — 90-й перцентиль \|Δ\| за 250 дат |
| `history.DEFAULT_SCALE_PCT` | 3 % | шкала до конца первого заполнения (`scale_basis = default`) |
| `history.HISTORY_LOOKBACK_DAYS` | 400 | первое заполнение: «сегодня − 400 дней … вчера» (≥ 250 сессий) |
| `history.APPEND_MAX_DATES` | 10 | дозагрузка: не больше дат за проход |
| `history.FILL_PAUSE_SEC` / `FILL_MAX_CONSECUTIVE_FAILURES` | 0,1 с / 3 | пауза между бумагами; три сбоя подряд — «ISS не отвечает», проход встаёт |
| `snapshot.ISS_SNAPSHOT_TTL_SEC` | 15 с | доска ISS — общий кэш процесса; после сбоя — следующая попытка не раньше |
| `snapshot.LIVE_PRICES_TTL_SEC` / `LIVE_PRICES_TIMEOUT_SEC` | 5 с / 5 с | кэш цен на пользователя (и неудачи тоже); срок `GetLastPrices` |
| `user_settings.DEFAULT_LIQUIDITY_THRESHOLD` | 10 000 000 ₽ | порог, если строки пользователя нет |
| `jobs.SYNC_MAX_AGE` | 24 ч | синхронизация после старта — только если последняя старше |

**5.18.1 Проход** (`jobs.run_screener_pass(db_factory, *, after_start)`, ADR 0091). Шаги: `history.refresh_stats` (из БД, без сети) → `iss_gate.wait_for_history_backfill` (ждёт задачу исполнителя `backfill._running_worker()` с перепроверкой раз в 30 с — кончившегося исполнителя мог сменить новый; `backfill.py` не правился) → `history.append_missing_dates` → `universe.sync_universe` (после старта — только если `last_universe_sync` старше `SYNC_MAX_AGE` или `None`) → `history.fill_history`, если у бумаги вселенной нет `iss_history_from` → `refresh_stats`. Сбой шага — warning `screener_pass_step_failed` (`step`, `error_type`), следующий шаг идёт; итог — `screener_pass_done` (`after_start`, `synced`, `duration_sec`). `SchedulerService.screener_nightly` передаёт `after_start=True` только первому вызову процесса; сбой — `screener_nightly_failed`. Все ISS-запросы модуля — под `iss_gate.iss_lock()` (одна страница модуля за раз, без `gather`).

**5.18.2 Синхронизация справочника** (`universe.sync_universe`, ADR 0090). Доска ISS (`get_board_snapshot`) → акции `EQIN`; пустая вселенная или сбой — `screener_iss_failed`, синхронизация не пишет. `analytics/IMOEX` → кэш процесса (`imoex_composition`, `cached_imoex_composition`). `Shares` — счётом `market_data_account(db)` (ключ — `decrypt_market_data_token`, только в памяти): нет счёта — warning `market_data_account_missing`, колонки брокера не трогаются; сбой — `screener_shares_failed`. `classify_broker`: запись `class_code = TQBR` → `trading`, при `trading_status = break_in_trading` — `break`; записи нет — `not_on_tqbr`; флаги квал / шорт, `uid`, FIGI — из записи TQBR; сектор — из записи TQBR, если непустой, иначе из первой по `class_code` записи другого класса. Код вне `SECTOR_NAMES` (кроме `other`) — `screener_sector_unknown`. Бумага ушла с доски — `in_screener_universe = 0`, строка остаётся. Итог — `screener_sync_done` (`universe`, `imoex`, `broker` — `synced` / `no_account` / `failed`, `trading`, `break`, `not_on_tqbr`, `sector_broker`, `sector_override`, `without_sector`, `added`, `removed`).

**5.18.3 История и статистика** (`history.py`, ADR 0091). `fill_history` — бумаги вселенной без `iss_history_from` по одной, `get_ticker_day_totals` за `HISTORY_LOOKBACK_DAYS`, после бумаги — отметка той же транзакцией; журнал `screener_history_fill_progress` (каждые 20 бумаг) и `screener_history_fill_done` (`pages`, `rows`, `tickers`, `failed`, `left`, `history_from`). `append_missing_dates` — будние даты после последней даты таблицы до вчера (МСК), `get_board_day_totals`, только бумаги вселенной; сбой даты — проход встаёт на ней (иначе пропуск не догрузился бы); будний день без строк (праздник) пропускается; журнал `screener_history_appended`. `compute_stats` → `MarketStats` в памяти процесса: медиана `VALUE` бумаги по её строкам среди последних 60 различных дат (`VALUE` null → 0; строк нет — медианы нет, `liquid = false`); шкала — 90-й перцентиль \|CLOSE_t / CLOSE_prev − 1\| соседних строк бумаг вселенной среди последних 250 дат (`statistics.quantiles(…, method="inclusive")`), %, 2 знака. История готова ⇔ у каждой бумаги вселенной есть `iss_history_from`; не готова — шкала 3 %, `liquid = null`. Пересчёт — `refresh_stats` (журнал `screener_stats_refreshed`: `ready`, `liquid_default` при 10 млн ₽, `sessions`, `scale_basis`, `scale_pct`), после рестарта — первым шагом прохода и при первом снимке (`ensure_stats`), без сети.

**5.18.4 Снимок** (`snapshot.build_snapshot(db, user)`, ADR 0092). Доска — `get_board`: общий кэш 15 с, один запрос в полёте под `iss_lock`; сбой ISS — прежняя доска с `iss_stale = true` (журнал `screener_iss_failed`), доски не было — `ScreenerUnavailable` → 503. Живые цены — только если `market_data_account(db)` выбран **и** `select_market_data_account(db, user.id)` даёт свой счёт T-Invest с ключом (production вперёд, затем sandbox); иначе `live_status = no_market_data_account | no_broker`. Один `get_last_prices` по `broker_uid` бумаг `trading`; кэш 5 с на пользователя, замок на пользователя (ленивый на loop, gotcha-71), срок — `asyncio.timeout(5)` (gotcha-87); сбой, таймаут или нерасшифровываемый ключ — все цены ISS, `broker_error` (неудача кэшируется на 5 с; журнал `screener_live_prices_failed` — `user_id` и тип ошибки, без текста: в нём мог бы оказаться ключ). **`broker_quote_is_current(quote_time, iss_time, today_msk)`** (ремонт R24 по стенду, `c4cf969`): цена брокера живая, только если её дата МСК не раньше даты МСК данных строки доски (`iss_row_time`: `UPDATETIME` на дату `SYSTIME`, позже `SYSTIME` — прошлые сутки; нет — сегодня); цена без времени — не текущая; иначе строка идёт по ISS (`price_source = iss`), `broker_status` не меняется. Адаптер T-Invest — в памяти процесса по ключу `(account.id, account.updated_at)`: ключ расшифровывается и `connect` зовётся только при создании (одна строка `tinvest_connected` на счёт, R29); сменился ключ или счёт — новый адаптер, старому `disconnect()` (сбой — `screener_adapter_disconnect_failed`); сети адаптер не держит. Цена ISS — `LAST`, нет — `PREVPRICE`. Без счёта для рыночных данных `broker_status`, флаги и сектор брокера — `null` у всех строк. До первой синхронизации вселенная — бумаги доски `EQIN` (`universe_source = iss_snapshot`). Состояние процесса сбрасывает `reset_snapshot_state()` (из `reset_screener_state()` — autouse-фикстуры тестов).

**5.18.5 Секторы и поправки** (`sectors.py`, `overrides.py`, ADR 0093). `resolve_sector(broker_code, override_code)` — поправка (код из `SECTOR_NAMES`), иначе код брокера из словаря, иначе «без сектора»; поправка с кодом вне словаря сектор брокера не прячет. Снимок читает поправки на каждый запрос. `create_override` / `update_override` / `delete_override` — проверки (§4.19), запись и `common.audit.audit` в одной транзакции; гонка за тикер на уникальном индексе — 409, а не 500.

**5.18.6 Скрипты** (`backend/scripts/`, на выкатке не запускаются). `record_b03_fixtures.py` — запись фикстур (ISS — доска, `history` по дате и по бумаге, IMOEX, итоги 250 дат сжатым CSV; `--user-id` — живой `Shares` / `GetLastPrices`; `--frontend-snapshot` — JSON снимка для фронтенда настоящим `build_snapshot`). `check_b03_sector_indices.py` — составы отраслевых индексов ISS против действующих секторов справочника БД из `DATABASE_URL` (рабочая БД — отказ): «совпало N из M», расхождения с пометкой «соглашение» или «НЕ из списка»; коды 0 / 1 / 2.

**5.18.7 Гейты, тестовый стенд и известные ограничения.**
- **Сборка** (16.11, `ee31ed9`; Autopilot semi/strict, база — голова PR #131 `14d76e3`): pytest в режиме CI (отдельный venv по lock) — 8800 passed / 1 skipped, покрытие 93 %; `DEBUG=true` — 8800 / 1; ruff, mypy (287 файлов), bandit — чисто; `alembic upgrade head && alembic check` — без дрейфа, одна голова `b03e5c71a9d4`; фронт — typecheck, lint, build, vitest 2665; E2E полным набором на своих портах — 322 passed / 1 skipped (было 298 / 1). Слепая приёмка из чистого клона нашла плитки карты 5,6–7,5 px без брокера — ремонт F2 («Прочие секторы»); ревью — 9 находок, в том числе кэш адаптера T-Invest без сверки владельца счёта, — ремонт F1.
- **Перед мержем** (18.11, локальные гейты — CI GitHub недоступен): на `2ce61d7` (после мержа `develop` `da73abf`) — pytest в режиме CI 8835 passed / 1 skipped, покрытие 93 %; ruff, mypy, bandit, `alembic check` — чисто; фронт — 2665 тестов. Голова PR `d1457ec` отличается записью в `.autopilot/PROGRESS.md`. Влит 2026-11-18 (PR #142, мерж-коммит `98d8f23`).
- **Тестовый стенд 17–18.11** (кандидаты `5510746` и `2ce61d7`, копия рабочей БД, счёт #2 для рыночных данных — только на копии) — **PASS WITH NOTES**: `screener_sync_done universe=261 imoex=44 broker=synced trading=260 break=1 not_on_tqbr=0 sector_broker=254 sector_override=4 without_sector=['GAZC','GAZS','GAZT']`; коды секторов `Shares` — только 10 кодов словаря, `other` у HIMCP и KRKOP, пусто у ZVEZ, GAZC, GAZS, GAZT, `screener_sector_unknown` — 0; ночной проход 01:30 18.11 — `break=1` (AMEZ), 7,2 с; первое заполнение — `pages=780 rows=72020 failed=[]` за ≈ 7 мин; `liquid_default=107`, `scale_pct=3.80`; `check_b03_sector_indices.py` — 89 из 96, код 0; `GetLastPrices` при открытой карте — ≈ 6,3 вызова в минуту (опрос 5 с = TTL кэша), ошибок квоты 0; после R24 снимок 18.11 11:57 МСК — 261 строка, `live` 243, `iss` 18.
- **Известные ограничения** (в Б-03 не правятся):
  1. Синхронизация после старта ждёт полный проход `history_backfill` (на копии рабочей БД — ≈ 2 ч); счёт для рыночных данных, отмеченный после синхронизации, даёт колонки брокера только ночью (`SYNC_MAX_AGE`), без подсказки.
  2. `broker_status` — снимок синхронизации: «перерыв» держится до следующей.
  3. «В избранное» из скринера (`userFavoritesStore.add`) ставит бумагу в фон A-02 — догрузка 1m (на стенде AFLT — +85 тыс. строк, +12 МБ за минуту).
  4. `screener_iss_day_totals` не урезается.
  5. Замок `iss_lock` — только для модуля: запрос пользователя к ISS посреди прохода скринера не исключён — общий ограничитель ISS на процесс остаётся известным багом A-02; запрос снимка ждёт замок, пока страница фона скринера в полёте.
  6. Пользователь, удаливший счёт и сразу заведший новый под тем же `id`, получает прежний адаптер до правки счёта (ключ свой, чужого нет).
  7. Предупреждения SDK `InstrumentType '11'` (`USE_DEFAULT_ENUM_IF_ERROR`) в журнале — шум SDK, были и до Б-03.

---

## 6. FRONTEND-МОДУЛИ: ДЕТАЛЬНАЯ СПЕЦИФИКАЦИЯ

**Ответственный: FRONT1 + FRONT2 (реализация), UX (дизайн, ревью)**

### 6.1 Layout / Shell

- Тёмная тема: CSS-переменные `--bg-primary: #1a1a2e`, `--bg-secondary: #16213e`, `--text-primary: #e0e0e0`, `--accent: #0f3460`, `--green: #00b894`, `--red: #e74c3c`.
- Left Sidebar (ширина 60px свёрнутая, 240px развёрнутая): иконки + текст для разделов (Дашборд, Стратегии, Бэктесты, Торговля, Счёт, Настройки). С v4.4 (Б-01) — пункт «Новости» (`sidebarItems.ts`: `IconNews`, путь `/news`, `data-testid="sidebar-news-link"`) сразу после «Графиков» (§6.10). С v4.5 (A-02) — пункт «Дивиденды» (`sidebarItems.ts`: `IconCoin`, путь `/dividends`, `data-testid="sidebar-dividends-link"`) сразу после «Счёта» (§6.11). С v4.11 (Б-03) — пункты «Скринер» (`IconListSearch`, `/screener`, `sidebar-screener-link`) и «Карта рынка» (`IconLayoutGrid`, `/market-map`, `sidebar-market-map-link`) сразу после «Дивидендов» (§6.12).
- Header (48px): логотип, виджет баланса (компактный), индикатор здоровья (цветной кружок), колокольчик уведомлений (с badge), username. **A-04 (ТЗ v4.7):** `StopAllButton` — красная, `data-testid="stop-all-btn"`, подпись с `sm`, ниже — значок `IconHandStop`; при включённом стопе недоступна с подсказкой «Стоп уже включён» → `StopAllConfirmModal` (`POST /trading/stop-all`) → `StopAllReportModal`. В `AppShell` над содержимым — `StopAllBanner` (`stop-all-banner`, `stop-all-banner-closed`, `stop-all-banner-waiting`, `lift-stop-btn`) → `LiftStopModal` (`lift-stop-confirm`, `lift-stop-warning`; `DELETE /trading/stop-all`). Плашка грузит состояние при монтировании, возврате фокуса и переподключении WS и слушает `system:{userId}` (`stop_all.changed` → `applyEvent`, `cb.triggered` → перечитать); ответ без `active` — состояние `null`, не исключение. Тексты и склонения — `components/layout/stopAllFormat.ts`.
- **Подвал `StatusFooter.tsx` — статус рынка (A-03, ТЗ v4.1).** `MarketStatus.status` (`src/api/marketDataApi.ts`) → `STATUS_CONFIG`: `online` «MOEX Online», `evening` «MOEX Вечерняя», `auction` «MOEX Аукцион открытия», `closing_auction` «MOEX Аукцион закрытия — только выходы» (новый), `dealer` «Часы дилера» (новый; вместо отсчёта — «Биржа закрыта, торгует дилер»), `weekend_session` «Торги выходного дня», `weekend` «Выходной», `offline` «MOEX Offline». Отсчёт `COUNTDOWN_LABEL`: «До закрытия вечерней», «До начала торгов» (аукцион открытия), «До конца аукциона закрытия», иначе «До закрытия». Источник — `GET /market-data/market-status` по таблице периодов (§5.9). Раньше 18:40–18:50 подвал показывал «Online» при закрытых входах.
- Content area: занимает оставшееся пространство.
- Минимальная поддерживаемая ширина: 1280px.
- **Узкий экран < `sm` (Mantine по умолчанию 48em = 768 px; S8R-FIX-108, ФТ §13.1).** `AppShell navbar.collapsed.mobile = !uiStore.mobileNavOpened` — ниже `sm` Mantine растягивает навбар на 100 % ширины, и без `collapsed.mobile` он закрывал страницу. Два `Burger` в `Header`: `hiddenFrom="sm"` — `toggleMobileNav` (`aria-label` «Открыть меню» / «Закрыть меню», `aria-expanded`, `data-testid="mobile-nav-burger"`), `visibleFrom="sm"` — прежний `toggleSidebar` (240↔60 px). Смена `location.key` закрывает меню (`AppLayout`, `closeMobileNav`). Строка шапки `wrap="nowrap"`, название скрыто ниже `xs`, поиск `flex: 0 1 400px`. `Sidebar` ниже `sm` всегда с подписями (свёрнутость десктопа там не действует); у каждого `NavLink` — `aria-label={item.label}` (S8R-FIX-099: в свёрнутом виде пункт без подписи не имел доступного имени). Ограничение: пункты — `<a>` без `href`, с клавиатуры не фокусируются (так было и до фикса).
- Русский язык всюду. Формат чисел: `Intl.NumberFormat('ru-RU')`. Формат дат: `DD.MM.YYYY`. **Пояс показа — МСК (S8R-FIX-109):** `utils/formatters.ts` — `formatDate`/`formatDateTime`/`formatTime` с `timeZone: MSK_TIME_ZONE` (`'Europe/Moscow'`), строки backend (naive UTC) — через `parseBackendDate`; на них переведены журнал сделок, карточка и страница сессии, тултип P&L, время проверки счёта, даты налоговых отчётов. Места с собственным `toLocale*` без зоны (список ~~и сделки~~ бэктеста, уведомления, история версий) пока показывают пояс браузера. С v4.8 (починка времени бэктеста) сделки и графики результата бэктеста — по МСК: строки backend (naive UTC) — через `utcIsoToUnix` / `parseUtcTimestamp` (`utils/mskTime.ts`), §6.5.

### 6.2 Dashboard (Главный дашборд)

**Маршрут:** `/`

**Компоненты:**
- `StrategyTable` — таблица стратегий с колонками из ФТ 3.1. TanStack Table с сортировкой по любой колонке. Цветная плашка статуса. Для каждой строки — dropdown с действиями (Открыть, Бэктест, Запустить торговлю, Архивировать).
- `BalanceWidget` — карточка справа: баланс Real + баланс Paper (раздельно). Суммарный unrealized P&L всех активных стратегий. Обновление через WebSocket.
- `QuickActions` — кнопки "Новая стратегия", "Быстрый бэктест".
- `ActiveStrategiesCards` — горизонтальная лента карточек активных стратегий (Real и Paper): тикер, P&L, мини-sparkline.

### 6.3 Strategy Editor

**Маршрут:** `/strategies/:id`

**Две вкладки:** «Содержание» и «Редактор».

**Вкладка «Содержание»:** Описание стратегии (textarea) + параметры (стоп-лосс, тейк-профит, размер позиции).

**Вкладка «Редактор» — два режима:**

**Режим Blockly (по умолчанию):**
- **Слева:** Blockly workspace с кастомным toolbox (Mantine Accordion, сворачиваемый 240→0px).
- **Справа:** SharedDescriptionPanel — описание стратегии + кнопки «AI-помощник», «Пустой шаблон», «Применить на схеме», «Заполнить по схеме».
- **Внизу:** toolbar (Zoom+/-, Undo, Redo, Очистить, Генерировать, Запустить бэктест).

**Режим AI-помощник (свайп-анимация 300ms):**
- **Слева (40%):** описание стратегии (редактируемый textarea).
- **Справа (60%):** AI-чат (SSE streaming). Header: выбор провайдера (dropdown), счётчик запросов. Кнопки: «Очистить историю», «Вернуться к редактору».
- Resizable разделитель (PanelGroup).

**Blockly-конфигурация — кастомные категории:**
- **Индикаторы:** SMA, EMA, RSI, MACD, Bollinger, ATR, Stochastic, CCI, Volume. Каждый — блок с параметрами (период, source, описание).
- **Условия:** Crossover, CrossUnder, GreaterThan, LessThan, InZone, Timeframe.
- **Логика:** AND, OR, NOT.
- **Сигналы:** EntryLong, ExitLong. Динамическое количество условий И/ИЛИ (N входов).
- **Фильтры:** TimeFilter, DayOfWeekFilter.
- **Управление капиталом:** PositionSizer (%), StopLoss (%), TakeProfit (%).

Кнопка «Генерировать» запускает генерацию Python-кода по блочной схеме. Код отображается в панели. Кнопка disabled без блоков на workspace.

Кнопка «Запустить бэктест» → BacktestLaunchModal. Disabled если код устарел (блоки ≠ код) или содержит ошибки.
- Modal с textarea для вставки + кнопка "Скопировать шаблон" (копирует Приложение A в clipboard).
- При нажатии "Распознать" -> `POST /api/v1/strategies/{id}/parse-template` -> блоки загружаются в Blockly workspace.

**Отчёт разбора (A-05, ТЗ v4.2; ФТ §3.2, §8.2).**
- `components/strategy/ParseReportView.tsx` — `ParseReportView({ report })`, один компонент на три места: панель `SharedDescriptionPanel` (контейнер `description-parse-report`, показ при непустом `parseReport`), окно «Режим B» `TemplateModeModal` (после «Проверить»; вместо прежнего `Alert` с `warnings`) и модалка «Просмотреть» `VersionsHistoryDrawer` (контейнер `versions-preview-report` над вкладками кода). `null` — «Отчёт не составлялся»; пустой — «Проблем и заметок нет»; иначе «Проблемы — эти строки не попали в блоки, версия с ними не сохранится» и «Заметки — перенесено с изменением» со счётчиками, в записи — «Строка N» / «Весь текст», фрагмент, «Что делать: …». Testid: `parse-report` (`data-state`), `parse-report-summary`, `parse-report-problems`, `parse-report-notes`, `parse-report-issue` (`data-level`), `parse-report-none`.
- `strategyStore.parseReport` / `setParseReport` — последний отчёт в общем сторе (панель смонтирована дважды — Blockly и AI-режим, и из AI-режима отчёт раньше терялся). Пишут: «Применить на схеме», «Применить» в «Режиме B», `saveVersion` (отчёт созданной версии), `fetchStrategy` (отчёт последней версии), отказ 422 `description_has_problems`. Сбрасывают: «Пустой шаблон», «Заполнить по схеме».
- `hooks/useStrategySave.ts`: 422 `description_has_problems` — отчёт в стор, тост «Описание содержит проблемы» / «Версия не сохранена: N проблем(а) в описании» (склонение; считаются только «проблемы»); страница не трогается, признак несохранённых изменений не снимается; у новой стратегии — откат, как при любом 4xx (`createWithFirstVersion`).
- «Пустой шаблон» панели = `EMPTY_TEMPLATE` модалки (= `EMPTY_TEMPLATE_7`). Логика Blockly — `logic_and` / `logic_or` / `logic_not` (`flatBlocksToWorkspace.ts`), параметры MACD `fast/slow/signal` и Stochastic `%K/%D` доходят до полей блоков; `blocksToTemplate.ts` пишет Volume как `[I1] Volume()`. Мёртвые `StrategyDescription`, `AIModePanel`, `TemplatePanel` не менялись.

### 6.4 Charts (Графический модуль)

**Маршрут:** `/charts/:ticker` (standalone) или встроен в другие экраны. Фактически — `/chart/:ticker?` (`pages/ChartPage.tsx`); с v4.3 (Б-02) страница собрана из панелей — «Панели графика» ниже.

**Компоненты:**
- `CandlestickChart` — TradingView Lightweight Charts. `createChart()` с dark theme config.
- `VolumePane` — отдельная панель внизу (histogram).
- `IndicatorPanel` — dropdown сверху для добавления индикаторов. Каждый индикатор — отдельная LineSeries/HistogramSeries. Настройки: период, цвет (ColorPicker), толщина.
- `TimeframeSelector` — кнопки 1м | 5м | 15м | 30м | 1ч | 4ч | Д | Н | М (`30м` — с A-01, ТЗ v4.0; тип `Timeframe` в `src/api/types.ts`, `sequentialIndex.ts`, `marketDataStore.ts` — `INITIAL_PERIOD_DAYS['30m']` = 14, зеркало `market_data/prefetch.py`). Живая свеча любого ТФ принимается через `candle.update`; на Д/Н/М последняя свеча обновляется в течение дня потоком с бэкенда (§5.4.5; фронт-код `useChartLiveData` не менялся, поведение закреплено тестом `useChartLiveData.dailyCandle.test.ts`). С v4.3 (Б-02): `TimeframeSelector({value, onChange, compact?})` стоит в заголовке каждой панели и стор графика не читает (избранные ТФ — кнопками при любой раскладке, в нескольких панелях — `compact-xs`); `useChartLiveData` удалён — живые кадры принимает `useCandleStream`, тесты дневной свечи перенесены в `useCandleStream.test.ts`.
- `TradeMarkers` — маркеры на графике (треугольник вверх = вход, треугольник вниз = выход) через `chart.addMarkers()`.
- `DrawingTools` (Should) — ~~toolbar справа~~ toolbar слева (с v4.3 — `DrawingToolbar` активной панели, на узком экране — горизонтально над графиком): Trendline, Horizontal, Rectangle. Координаты сохраняются через API.
  - **Привязка точек на intraday (sequential-index, S8R-FIX-102).** Бар sequential-серии несёт время свечи в `customValues.barTime` (и `setData`, и live-`update`). `DrawingPoint.t` — время свечи (источник правды), `logical` — номер бара на момент клика; при каждой отрисовке номер бара ищется по `t` в текущем массиве (`coords.ts`, `sequentialIndex.ts`), правее последнего бара — экстраполяция шагом ТФ, левее загруженного окна — точка не рисуется до дозагрузки. ~~Старые точки (`t` — номер бара, разобранный как unix-время, год < 2000) рисуются по `logical`, как раньше; перетаскивание привязывает их к свече.~~ С v4.6 (Б-04) таких точек нет — их удаляют миграция `b6b5064c0be6` и `purgePre2000LocalDrawings` (ADR 0054); `logical` для позиции не читается, номер бара на любом ТФ — только `drawingBarIndex` (ниже, «Рисунки на всех ТФ …»). Drag, vline, копия, концы long/short — в барах серии. Раньше в `t` писался индекс (`1969-12-31…`), и рисунок рисовался по `logical` — номер бара указывал на другую свечу при каждом сдвиге скользящего окна (`now − INITIAL_PERIOD_DAYS`) и после подгрузки истории слева. Дневные ТФ не менялись; API не менялся.
  - **Смена набора (тикер:ТФ)** — первая загрузка: `autoScale:true` → начальный вид → фиксация `autoScale:false` в rAF. Раньше `autoScale:false` переживал `setData` нового набора, если его свечи были в кэше (график не перемонтируется), и шкала оставалась от прежнего ТФ (находка qa2 Н-6).
- `OrderBookPanel` (Could) — выдвижная панель справа с уровнями bid/ask.
- `TradesStreamPanel` (Could) — лента сделок внизу.
- `PriceAlertModal` — окно уровневого алерта (A-03, ТЗ v4.1; ФТ §5.8): поля «Условие» («Выше/Ниже») нет — направление ставит сервер; при открытии — `priceAlertApi.getQuote(ticker)` (`GET /price-alerts/quote`): «Текущая цена: …» и подсказка `↑ сработает, когда цена поднимется до 300 (+2,2 %)` / `↓ … опустится до …`, меняется при вводе; `decimalScale` и `step` поля — из `price_decimals`/`price_step` (FEES 0,0605 вводится без округления; без шага — по знакам текущей цены, 2..8, `D04`); `warning` ISS показывается строкой «Цена MOEX ISS: задержка 15 мин»; создание — `priceAlertStore.createAlert` (возвращает `{alert, error}`), отказ сервера (409/422, «уровень равен текущей цене — сдвиньте уровень») — в окне, алерт не создаётся. `CandlestickChart` не менялся: стрелка линии алерта уже берётся из `condition` ответа, маршруты `/price-alerts` — прежний формат.
- ~~Дилерский канал `dealer:{ticker}:{tf}` (§2.3) фронтенд пока не открывает — показ дилерских свечей и переключатель «внебиржевые часы» — блок Б-02.~~ С v4.3 канал открывает переключатель «Внебиржевые часы» панели — «Панели графика» ниже. Бары аукциона открытия (O=H=L=C у 1h/4h) с A-03 приходят в данных графика (фильтр удалён, §5.6.6).

**Пометка задержки ISS (A-05, E3; ТЗ v4.2; ФТ §5.5).** `marketDataStore.lastSource: DataSource` — источник текущего набора свечей: пишет только успешный `fetchCandles` активного набора, сбрасывается в `setTicker`, `setTimeframe` и при сбое без графика; `fetchOlderCandles` его не меняет; прежнее `dataSource` с 3-секундным сбросом (подпись источника) не тронуто. `CandlestickChart` показывает постоянный элемент `data-testid="chart-source-delay"` «MOEX ISS · задержка 15 мин» (под подписью источника, одной колонкой), если `lastSource === 'moex_iss'`, или `lastSource === 'cache'` и список счетов загружен без активного T-Invest (`broker_type === 'tinvest' && is_active`, боевой или песочница — тот же признак, что backend `_has_active_tinvest_account`); список запрашивается `useSettingsStore.fetchAccounts()` только при `cache`, до ответа (или при ошибке) пометки нет. Источник `cache` backend отдаёт и для строк ISS, и для строк T-Invest — различить можно только по счетам. Опроса ISS для живой свечи нет. Тесты — vitest на `CandlestickChart`/`marketDataStore`, E2E — `frontend/e2e/a05-chart-iss-delay.spec.ts` (моки). **Ограничение:** у пользователя с T-Invest-счётом без рабочего ключа при источнике `cache` пометки нет (ключ фронт не видит). **С v4.3 (Б-02)** общего `marketDataStore.lastSource` нет — поле ряда `SeriesState.lastSource` (пишет только успешная загрузка ряда: первая, «Повторить», обновление ряда без подписки; подгрузка слева и живые кадры его не меняют, сбой без свечей сбрасывает); решение — хук `hooks/useIssDelay.ts` в `ChartPanel` (`useIssDelay(main.lastSource)`; список счетов при `cache` — один общий запрос `fetchAccountsShared` на все панели), график получает проп `issDelay`. Легенда OHLC, 3-секундная подпись источника и пометка — одна абсолютная колонка в левом верхнем углу (`top: 8; left: 12; flex-direction: column`): в узкой панели легенда переносится на вторую строку, и пометка опускается вместе с ней (стенд 2026-11-05, `dad3c68`; раньше блок стоял на фиксированном `top: 28` и закрывал вторую строку легенды). Тесты — `marketDataStore.lastSource.test.ts`, `useIssDelay.test.ts`, `CandlestickChart.sourceDelay.test.tsx`, E2E `a05-chart-iss-delay` «2×2 (Б-02): пометка … под легендой».

**Панели графика (Б-02, ТЗ v4.3; ФТ §5.1, §5.4, §5.5, §19.4; ADR 0024–0029).** Вид — макет А (выбран заказчиком 2026-10-26).
- **Страница** `pages/ChartPage.tsx`: общая полоса `ChartLayoutBar` (`chart-layout-bar`: «назад», раскладка `layout-kind-1|2h|2v|4`, `Switch` `link-symbol` и `link-crosshair`, справа — кнопка алерта и бейджи для тикера активной панели, `session-banner`, избранное); слева `DrawingToolbar` активной панели; сетка `chart-grid` (`data-kind`, CSS grid по `kind`) из `ChartPanel` (`chart-panel-<id>`: `data-active`, `data-ticker`, `data-timeframe`, `data-style`, `data-scale`, `data-dealer`, `data-adjust`, `data-sync-bar`). Заголовок панели — `ChartPanelHeader` (`chart-panel-header-<id>`, `panel-ticker-<id>`, `TimeframeSelector`, `panel-style-menu` / `panel-style-<style>`, `panel-scale-<scale>`, `panel-dealer-toggle`, `panel-dealer-hint`, `panel-adjust-toggle`; `wrap` — перенос строки). Активация — `onPointerDownCapture` / `onFocusCapture` панели → `setActivePanel`. Узкий экран — `useMediaQuery(max-width < theme.breakpoints.sm)` (48em), значение сразу, без кадра раскладки рабочего стола: только активная панель, `Tabs` `chart-panel-tabs` / `chart-panel-tab-<id>`, тулбар — горизонтально над графиком, `link-symbol` скрыт. Пока раскладка не пришла — `chart-layout-loading`; ни у одной видимой панели нет тикера — `chart-empty-state`; пустая панель — `chart-panel-empty-<id>`. В панели `minHeight: 500` обёртки графика снят CSS-правилом `.b02-chart-panel-body` — высоту задаёт сетка.
- **Адрес:** `/chart/:ticker` → `setPanelTicker(активная)` (повторный переход на тот же адрес при включённой связи символа и разных тикерах панелей — тоже выбор); адрес следует за активной панелью — `navigate('/chart/<тикер>' + search, {replace: true, state: {fromPanel: true}})`; `/chart` без тикера с сохранённой раскладкой уходит на тикер активной. `?session=N` — тикер и ТФ сессии на активную панель один раз на пару «сессия + ТФ»; сессия на странице одна. Прочие параметры адреса, как и раньше, не читаются.
- **Раскладка** — `api/chartLayoutApi.ts` (типы §4.13 один в один) и `stores/chartLayoutStore.ts`: `PANEL_COUNT` (1/2/2/4), `defaultLayout(ticker)` (kind `'1'`, `p1` — ticker, `D`, candles/normal, `dealer: false`, `adjust: 'splits'`; связи symbol `false`, crosshair `true`), `normalizeLayout(raw, fallbackTicker)` (битое или неизвестное → умолчание, тикер не по формату → тикер из адреса, `null` — пустая панель, повторный или битый id → новый `p<N>`, панели сверх 4 отбрасываются, недостающие до `kind` — по правилу роста, активная не из видимых → первая видимая), `visiblePanels`. Действия: `load(fallbackTicker)` — один раз за вход, 401/403 — умолчания без уведомления; `setKind` — рост: тикер активной, ТФ — первый из `D, 1h, 5m, 1m, 15m, 4h, W, M, 30m`, не занятый видимыми панелями этого тикера; уменьшение — скрытые остаются в `panels`, активная меняется местами с последней видимой; `setActivePanel` (только видимая), `updatePanel`, `setPanelTicker` (при `links.symbol` — на все видимые), `setLinks`, `clearSaveError`, `reset` (из `authStore.logout`). **Сохранение:** изменение (без разницы с прежним документом — ничего) → таймер `SAVE_DEBOUNCE_MS = 500` → `PUT` всего документа; в полёте не больше одного `PUT` — правки за время запроса уходят следующим с последним состоянием (`saveQueued`); ответ запроса прежнего входа (эпоха `reset`) очередь не трогает; ошибка → `saveError = LAYOUT_SAVE_ERROR` («Раскладка не сохранилась», уведомление `layout-save-error`). **D01 (ADR 0028):** после сбоя `GET` (кроме 401/403) таймер повторяет `GET` вместо `PUT`: раскладки нет → `PUT` текущей; есть → показывается серверная; сбой → `LAYOUT_LOAD_ERROR` («Раскладка не загрузилась с сервера — изменения не сохранятся, пока связь не вернётся»). В localStorage раскладка не пишется (BUG-20).
- **Ряды свечей** (`stores/marketDataStore.ts`, ADR 0025): `SeriesRef {ticker, timeframe, adjust, source: 'exchange'|'dealer'}`, `seriesKey` = `` `${source}:${ticker}:${timeframe}:${adjust}` ``, `streamKey` = `` `${source}:${ticker}:${timeframe}` ``; `series: Record<seriesKey, SeriesState>` (`candles`, `loading`, `loadingMore`, `noMoreData`, `error`, `errorCode` — `tinvest_required` (409) / `unsupported_timeframe` (422 дилерского ряда), `refreshError`, `dataSource` — сброс через 3 с, `lastSource`, `loaded`), `streamChannels`, `instruments`. `ensureSeries(ref)` — без сети, если ряд загружен и под подпиской или запрос в полёте; ряд с ошибкой — заново; **D02 (ADR 0029)** — ряд, оставшийся без держателя подписки, при следующем показе обновляется одним запросом. `reloadSeries` («Повторить»), `fetchOlderSeries(ref, before?)` (дилерский ряд — за тот же период, что основной, от его первой свечи), `applyLiveFrame(ticker, tf, source, frame)` — во все ряды ключа обоих `adjust`, `ensureInstrumentInfo`. `ChartPanel` зовёт `ensureSeries` только при монтировании и смене ряда (не из эффекта от `series[key]` — иначе цикл запросов на ошибке). Прежние общие `currentTicker`, `currentTimeframe`, `candles`, `fetchCandles`, `upsertLiveCandle`, `crosshairData`, `loading` удалены. Кэш `candlesCache` в localStorage — только у рядов `exchange` + `splits` (ключ `${ticker}:${tf}`, как раньше); ряд «как было» и дилерский при первом показе — со спиннером. Ряды живут в памяти вкладки до её перезагрузки (`MAX_CANDLES = 10 000` на ряд). `api/marketDataApi.ts`: `getCandles(…, adjust = 'splits')`, `getDealerCandles(…, adjust)`, `subscribeCandleStream(ticker, tf, source = 'exchange')`; `DataSource` дополнен `'tinvest_dealer'`.
- **Поток** — `hooks/useCandleStream.ts`: страница монтирует по одному `CandleStream({ticker, timeframe, source})` на уникальный `streamKey` видимых панелей раскладки (на узком экране — тоже всех видимых, хотя на экране одна). `POST /candles/subscribe` — один при открытии; повтор — по `useWsAuthGeneration` только после переподключения WS (запрос, ушедший при открытом сокете до `auth_ok`, на этом `auth_ok` не повторяется — `isWsAuthed()`). Канал принимается, только если префикс совпадает с источником (`market:` / `dealer:`). Кадр `candle.update` → `applyLiveFrame` с флагами `bar_pending` / `bar_complete` / `stream_gen`; держатель отмечает ключ «под подпиской» (`retainStream`) для D02. REST-отписки нет; WS-отписка — при размонтировании. **`mergeLiveCandle(candles, frame)`:** тот же `timestamp`, что у последней свечи, и `bar_pending` → `{open: прежний, high: max, low: min, close: кадра, volume: max}`; без `bar_pending` — замена целиком; новее — добавление; старее (S8R-FIX-022) — замена или вставка в окне, кадр `bar_pending` старее последней — игнор; время — `toUtcUnix`. `CandlestickChart` доносит поправленные не последние свечи до серии (до `MAX_PATCHED_BARS = 50` — точечно, больше — `setData`).
- **График панели** — `CandlestickChart` работает только от пропсов (`candles`, `timeframe`, `datasetKey`, `view`, `onScaleChange`, `dealerCandles`, `overlayInfo`, `sessionId`, `sequentialMode`, `alertLines`, `wsChannel`, `loading`, `loadingMore`, `noMoreData`, `dataSource`, `issDelay`, `onLoadOlder`, `onCrosshairTime`, `syncTimeSec`, `onChartReady`); стор графика не читает. `components/charts/priceSeries.ts`: `addPriceSeries(chart, style)` — `CandlestickSeries` (полые — прозрачный `upColor`), `BarSeries`, `LineSeries` и `AreaSeries` по закрытию; `toPriceSeriesData`; `priceScaleModeFor` → `priceScale('right').applyOptions({mode})` (Normal / Logarithmic / Percentage — от первого видимого бара); `LOG_HINT_RATIO = 3`; `visibleRangeRatio(candles, from, to)` = max(high)/min(low) видимых баров. Тип `PriceSeries = ISeriesApi<'Candlestick'|'Bar'|'Line'|'Area'>` — во всех слоях и примитивах; смена вида пересоздаёт серию и зовёт `onChartReady` с новой. Подсказка `log-scale-hint` (`-enable`, `-dismiss`) — только D/W/M, не в лог-шкале, при `visibleRangeRatio > 3`; «×» действует до смены `datasetKey` (тикер, ТФ, `adjust`). Подгрузка слева на D/W/M сравнивает время первой свечи с MSK-сдвигом (раньше не срабатывала). Легенда при полной замене ряда переходит на новую последнюю свечу, если смотрела на прежнюю последнюю или курсора нет.
- **Перекрестие** — `stores/chartCrosshairStore.ts` (`source: {panelId, timeSec} | null`, `publish(panelId, timeSec | null)`; не сохраняется). Панель публикует настоящее UTC-время начала бара под курсором (`onCrosshairTime`); соседние при `links.crosshair` получают `syncTimeSec` и ставят `setCrosshairPosition` на бар `barIndexForTime(starts, timeSec)` (последний бар с началом ≤ времени; в последовательном режиме — по `customValues.barTime`). Защита от эха: чужое перекрестие дальше не публикуется, а `crosshairMove` без `sourceEvent` (библиотека повторяет сохранённое перекрестие после обновления данных или шкалы) наружу не уходит — иначе перекрестие «залипало» на живых свечах. Подсвеченный бар — `data-sync-bar` (ISO).
- **Дилерская серия** (ADR 0027): только в последовательном режиме (1m–4h). `sequentialIndex.dealerSequentialData(mainUnix, dealer)` — k-я из m дилерских свечей между барами i и i+1 основного ряда получает `time = i + k/(m+1)` (`timeToBarTime` — для подписей оси `formatSequentialAxisTime` и легенды); вторая `CandlestickSeries` приглушённого цвета (`rgba(…, 0.35)`, без ценовой метки), объём не рисуется. `primitives/coords.ts`: в последовательном режиме X точки — `timeToCoordinate(<номер бара основного ряда>)`, не `logicalToCoordinate` (для дробной позиции lightweight-charts 5.2.1 даёт 0); `clickToDrawingPoint` пишет `logical` — номер бара основного ряда (на дилерской свече — бар слева), `t` — время свечи под курсором. Доступность — `dealerUnavailableReason(tf, settingsStore.hasBrokerConnection)`: не D/W/M («Внебиржевые часы — на ТФ до 4ч»), есть брокер («Подключите T-Invest»); ответ 409 `tinvest_required` → `updatePanel({dealer: false})` и подсказка `panel-dealer-hint` до смены тикера/ТФ или следующего нажатия. В режиме `MARKET_DATA_STREAM_MODE=two_subscriptions` сервер отдаёт `channel: null` — дилерский ряд без живого обновления, только история.
- **Рисунки** (ADR 0026): `api/chartDrawingsApi.ts` — `list(ticker)`, `create(ticker, timeframe, body)` → `POST {timeframe, ...body}`, `update(id, body)`, `remove(id)`, `clearAll(ticker, timeframe)` → `DELETE ?timeframe=`. `utils/drawingsPersistence.ts`: `DRAWINGS_STORAGE_VERSION = 2`, ключ `drawings:{userId|anon}:{ticker}` → `{version: 2, items}` (у каждого рисунка — `timeframe`), `MAX_DRAWINGS_PER_KEY = 100` на пару «бумага + ТФ», `migrateLocalDrawingKeys(userId)` — ключи v1 `drawings:{u}:{ticker}:{tf}` своего пользователя и `anon` переносятся в v2 (ТФ — из ключа v1; id, уже лежащий в v2, не задваивается; v1 удаляется только после успешной записи v2; битый v1 и нечитаемый v2 не трогаются; повторный вызов ничего не меняет). `stores/chartDrawingsStore.ts`: `byTicker` (ключ `upper(ticker)`), `setUser` (смена — сброс и один `migrateLocalDrawingKeys`), `ensureTicker` (одна загрузка на бумагу: ТФ, о которых знает сервер, — с сервера, остальные — из localStorage), `add` / `clearAll` с контекстом `{ticker, timeframe}`, `selectDrawings(state, ticker, tf)`. `DrawingsLayer {…, ticker, timeframe, isActive}` рисует рисунки своего тикера и ТФ на любой панели, создаёт, двигает и удаляет — только при `isActive`; `DrawingToolbar {ticker, timeframe, horizontal?}` и его клавиши — для активной панели; смена активной панели, её тикера или ТФ сбрасывает инструмент на «курсор» и снимает выделение. С v4.6 (Б-04) — пункт «Рисунки на всех ТФ, новые инструменты и линейка» ниже: `clearAll(ticker)` без ТФ, `MAX_DRAWINGS_PER_TICKER` вместо `MAX_DRAWINGS_PER_KEY`, `selectDrawings(state, ticker)`, `DrawingsLayer` рисует рисунки своего тикера на панели любого ТФ.
- **Рисунки на всех ТФ, новые инструменты и линейка (Б-04, ТЗ v4.6; ФТ §19.4; ADR 0050–0054).**
  - **Показ:** `chartDrawingsStore.selectDrawings(state, ticker)` — все рисунки бумаги на панели любого ТФ; `timeframe` рисунка — «где нарисован» (список тулбара: «<тип> · <тикер> · нарисован на <ТФ>»). Правка на одной панели сразу видна на всех; создаёт, двигает и удаляет только активная (`DrawingsLayer isActive`). «На передний план» — максимум `display_order` по бумаге + 1.
  - **Правило точки (ADR 0050):** точка встаёт на бар основного ряда, в который попадает её время. Бар `i` — последний с началом не позже времени, длится `[начало, min(начало следующего, начало + шаг))`, шаг — минимальная разница соседних начал; у M (шаг ≥ 27 суток) — до начала следующего календарного месяца МСК. Время за концом бара (ночной разрыв, выходной, пропущенный бар, время дилерской свечи) — первый бар после. Последовательный режим — `sequentialIndex.seqIndexForTime`, обычный (D/W/M) — `coords.regularBarIndexForTime`; единственный путь «время → номер бара» — `coords.drawingBarIndex(point, chart, series)`, X — `pointToX`. Правее последнего бара — экстраполяция шагом ТФ (у M — по месяцам), ветки дилерского хвоста `seqBarForTime` не менялись; левее загруженной истории — `null`, точка не рисуется. `logical` позиции не задаёт (поле необязательное, `clickToDrawingPoint` пишет его справочно); удалены запасные пути `pointToX` по `logical` и поиск по ISO, ветка «сначала `logical`» в `shiftPoint`, ветка «год < 2000», `utils/drawingsMigration.ts`. Сдвиг: точка, оставшаяся на своём баре, `t` не меняет; перешедшая на другой бар получает время его начала. Перекрестие (`barIndexForTime`, «последний бар с началом ≤ времени») — своё правило, не менялось.
  - **Предел и «Очистить» (ADR 0051):** `utils/drawingsPersistence.ts` — `MAX_DRAWINGS_PER_TICKER = 100` на бумагу; проверяет только `add` (`warning` «Не больше 100 рисунков на бумагу — удалите лишние.»), сохранение в localStorage ничего не срезает. `clearAll(ticker)` — память, ключ `drawings:{u}:{ticker}`, `chartDrawingsApi.clearAll(ticker)` → `DELETE /charts/{ticker}/drawings` без `timeframe`; окно `chart-clear-confirm`: «Удалить все рисунки <ТИКЕР> на всех ТФ (<N>)? Это действие нельзя отменить.».
  - **Старые рисунки (ADR 0054):** `purgePre2000LocalDrawings(userId)` — сразу за `migrateLocalDrawingKeys` в `setUser` (его зовёт `ChartPage`, то есть при первом открытии графика после входа): удаляет рисунки с точкой года < 2000 по правилу миграции из ключей пользователя и `anon` (иначе `adoptFromVariant` вернул бы их в набор), отметка `drawings:pre2000-purged:{userId|anon}` — после успешного прохода; отметку не читают `migrateLocalDrawingKeys` и `loadLocalDrawings`. Предупреждения «в старом формате» нет.
  - **Тулбар:** `DrawingToolbar` — таблицы `TOOL_BUTTON_META` (кнопки: подпись, клавиша, test-id, иконка) и `LINE_TOOL_META` (меню «Линии»). Клавиши: `V`, `T`, `A` (`ray`), `X` (`extended`), `H`, `G` (`hray`), `R`, `L`, `C` (`channel`), `F` (`fib`), `M` (линейка), `Esc`, `Delete`; в поле ввода и с Ctrl/Cmd/Alt не срабатывают. Test-id: `chart-tool-ray`, `-extended`, `-hray` (пункты меню), `chart-tool-channel`, `-fib`, `-ruler`.
  - **Примитивы:** `TrendlinePrimitive` — `trendline`, `ray` (за второй точкой до края pane), `extended` (в обе стороны); обе точки на одном баре — отрезок. `HlinePrimitive` — `hline` и `hray` (от бара `data.p1` вправо, бейдж цены на шкале — 4 знака). `ChannelPrimitive` — `p1`, `p2` и параллельная граница через бар и цену `p3`, заливка. `FibPrimitive` — уровень L на `p2 + L × (p1 − p2)` (`primitives/fib.ts::fibLevelPrice`), линии между точками по времени, подпись «0,618 · 312,45», пунктир `p1`–`p2`. Новый тип обязан попасть в `DrawingsLayer::PRIMITIVE_FACTORY` (от неё зависят `createPrimitive` и `data-ids`); якоря и попадание — `primitives/anchors.ts`, `hitTest.ts`.
  - **Окно уровней:** `FibPropertiesModal` (`fib-properties-modal`) — «Редактировать» контекстного меню слоя для `fib`; `parseFibLevelInputs`: пусто или не число, больше 20, |L| > 10 — «Сохранить» недоступна, ошибки в `fib-properties-error`; `normalizeFibLevels` — уникальные по возрастанию; сохранение — `update(id, {type, data: {...data, levels}, style})` → `PATCH`. По умолчанию `DEFAULT_FIB_LEVELS` = 0; 0,236; 0,382; 0,5; 0,618; 0,786; 1. Копия рисунка переносит `levels`.
  - **Линейка (ADR 0052)** — не рисунок: состояние `DrawingsLayer` и `RulerPrimitive` (прямоугольник, рост — зелёный, падение — красный), стор и API рисунков не трогает. Shift + протянуть по месту без рисунка (прокрутка графика на это время отключена) или инструмент `ruler` (клик — начало, движение — конец, клик — фиксация); следующий клик или Esc убирает. Подпись — DOM `chart-ruler-label` вверху по центру панели, `rulerLabel.ts::formatRulerLabel`: Δ цены (2 знака), % от цены начала (1 знак), бары — разность `drawingBarIndex` концов, часы торгов — один `marketDataApi.getTradingHours(start, end)` после фиксации (до ответа «…», ошибка «—»), «дн.» — разница дат МСК.
  - **Для E2E:** скрытый `drawings-layer-state` (`data-ticker`, `data-timeframe`, `data-active`, `data-ids`). `e2e/b04-drawings.spec.ts` — 12 тестов на моках: «сервер» рисунков и часов торгов в `e2e/fixtures/chart_panels.ts` (`drawingWrites`, `hoursRequests`, опция `tradingHours`); правились `b02-chart-panels.spec.ts` и `s7-drawing-tools.spec.ts`. vitest — `primitives/__tests__/{fib,hitTest,b04Primitives,coords.p3,coords.b04}.test.ts` (оснастка `b04Env.ts`), `__tests__/{rulerLabel,FibPropertiesModal,DrawingToolbar.b04,DrawingsLayer.b04,DrawingsLayer.ruler}.test.*`.
  - **Известные ограничения:** точка правее последнего бара без дилерских свечей при их включении встаёт на последний бар основного ряда (дефект Б-02 не закрыт); точка, перенесённая на другой бар, теряет время внутри бара; часы торгов за годы до торгов выходного дня завышены (запасные часы `fallback_day` дают выходную сессию субботе и воскресенью в любой год — правка A-03 по решению заказчика); линейка за десятки лет на M — ≈ 0,15 с в event loop (`_exchange_seconds`) и сбрасывает кэш запасных дней расписания; зафиксированная линейка при смене вида панели теряет прямоугольник (эффект `setRange` не зависит от серии); подпись линейки не следует за прокруткой; `p3` сохраняется и у типов, которым не нужен, цена точки принимает `NaN` у любого типа (было и до Б-04); «Очистить» удаляет основной ключ localStorage — варианты старых ключей (`anon`, верхний регистр) для занятых ТФ могут вернуться; бейдж цены `hline` не округляется (`String(price)`, как до Б-04).
- **Метки новостей (Б-07, ТЗ v4.9; ФТ §20.13; ADR 0066).**
  - **Слой** `components/charts/NewsMarkersLayer.tsx` — рядом с `DrawingsLayer` в `ChartPanel`, график и серию берёт через `onChartReady`; обёрнут в `ErrorBoundary` с пустой заменой — сбой меток не роняет панель (доработка `4085804`: из-за `candles[0].timestamp` без значения падала страница графика в E2E «view_chart»; без строки времени у первой свечи слой метки не грузит — `intervalStart`). Грузит `newsApi.getNewsForTicker(ticker, from, to)` (`GET /news/by-ticker`) при смене ряда и раз в 60 с (`REFRESH_INTERVAL_MS`), от первого бара ряда, но не глубже 399 суток (`MAX_SPAN_MS`; предел маршрута — 400); подписок на поток нет.
  - **Номер бара** — `newsMarkers.ts::groupNewsByBar`: на 1m–4h — только `drawingBarIndex` (правило ADR 0050: бар, в который попадает время; вне баров — первый после; правее последнего — экстраполяция шагом ТФ; левее загруженной истории — не рисуется); **на D/W/M — `calendarBarIndex`: бар календарной даты МСК** новости (сутки, неделя с понедельника, месяц; дата МСК метки одна у всех разметок дневок — T-Invest и окно ISS 00:00 UTC, ISS до шва 21:00 UTC предыдущих суток; дня без бара — первый после; доработка `227df39`). Одна метка на бар (`NewsBarMarker {barKey, idx, time, beyond, count, items, fromIso, toIso}`).
  - **Рисует** `primitives/NewsMarkerPrimitive.ts` (новый файл; примитивы Б-04 не менялись): кружок с числом у нижнего края области цены, X — `timeToCoordinate` времени бара на каждом кадре (правее последнего — логическая позиция + `beyond`), `hitTest` по кружкам последнего кадра → `externalId = barKey`.
  - **Подсказка** `news-marker-tooltip` по `subscribeCrosshairMove` + `hitTest`: `tooltipRows` — первые `TOOLTIP_MAX_ROWS` = 5 новостей бара (время МСК, источник, заголовок до двух строк, тип, «оценка ИИ: …», реакция «до …, после …» только у `done`), строка «ещё N» (`news-marker-more`); выводится поверх страницы (`tooltipPlacement`: над меткой, не помещается — сбоку, целиком в окне), а не внутри панели. **Клик** (`subscribeClick`) — `navigate('/news?ticker=&from=&to=')` (`newsFeedUrl`: интервал бара — на D/W/M сутки/неделя/месяц МСК — расширенный до времени его новостей); при выбранном инструменте рисования клик метке не принадлежит.
  - **Флаг панели** `panel.news` (по умолчанию `true`, `normalizePanel` — `boolOr(raw.news, true)`, сервер — `PanelSettingsDoc.news`, §4.13), значок `panel-news-toggle-<id>` в `ChartPanelHeader` → `updatePanel`; `data-news` на корне панели. Метки есть, только если `panel.news` и в `news_settings` включены `collect_enabled` и `chart_marks_enabled` (`newsSettingsStore`; не загрузились — как выключено).
- **Прочее:** `OpenPositionsLayer {…, livePrice}` — цена последней свечи ряда панели; маркеры сделок (`useTradeMarkers`) — `sessionId` панели только при совпадении её тикера с тикером сессии; линии алертов — по тикеру панели (`sameTicker`); `StatusFooter` не менялся (статус торгов MOEX). Фильтр `filteredCandles` 07–20 UTC удалён вместе с упоминаниями — утро на внутридневных ТФ видно.
- **Тесты:** vitest — `chartLayoutStore`, `chartCrosshairStore`, `marketDataStore.series`, `useCandleStream`, `useIssDelay`, `ChartPage.panels`, `ChartPanel`, `ChartPanelHeader`, `ChartLayoutBar`, `CandlestickChart.panel`, `priceSeries`, `sequentialIndex`, `coords.dealer`, `DrawingsLayer`, `drawingsPersistence`, `chartDrawingsStore`; фикстуры кадров `src/test/fixtures/live-frames-{W,M}.json`; E2E `frontend/e2e/b02-chart-panels.spec.ts` (моки `e2e/fixtures/chart_panels.ts`, `page.route` + `routeWebSocket`). Backend — `test_chart_layout_router.py`, `test_chart_drawings_crud.py`, `test_market_data_router.py` (`adjust` у дилерского ряда), `test_split_storage.py`, `test_stream_one_subscription_per_ticker.py` (SBER 1m/5m/1h/D — одна подписка обновлений 1m и одна закрытых минут; четыре бумаги — четыре и четыре), `test_migration.py`.
- **Известные ограничения:** правка раскладки меньше чем за 500 мс до перезагрузки или закрытия вкладки не сохраняется (дебаунс без сброса на `pagehide`); правка, попавшая в `PUT`, ушедший из очереди, ставит своим таймером ещё один `PUT` с тем же документом (лишний одинаковый запрос; в полёте по-прежнему один); перекрестие с D на 1h подсвечивает последний бар предыдущего дня; дилерский ряд не догружается за последний период, если подгрузка основного ряда влево застала дилерский запрос в полёте (`ChartPanel.onLoadOlder`); точка рисунка правее последнего бара, поставленная без дилерских свечей, при их включении встаёт на последний бар основного ряда, ~~а точки «год < 2000» (до S8R-FIX-102) с дилерскими свечами могут сместиться — до Б-04~~ с v4.6 таких точек нет; варианты старых ключей рисунков (верхний регистр, `anon`) для уже занятых ТФ остаются невидимыми; ~~предупреждение «N рисунок(ов) в старом формате» выходит на SBER любого ТФ (рисунки бумаги грузятся всеми ТФ) — уходит в Б-04~~ с v4.6 предупреждения нет; тест «одна подписка на тикер» читает внутренности мультиплексора; «Внебиржевые часы» с живым каналом `dealer:` вживую не проверены — разовая проверка на стенде 07–08.11.

**Real-time:**
При открытии графика: `ws.send({action: "subscribe", channel: "market:SBER:D"})`. При получении candle — `series.update(candle)`. При закрытии — `ws.send({action: "unsubscribe", ...})`.

### 6.5 Backtest Results

**Маршрут:** `/backtests/:id`

**Четыре вкладки:**

**Tab "Обзор":**
- Сетка метрик (SimpleGrid 4×2): Чистая прибыль (₽ + %), Кол-во сделок, Доля прибыльных (Win Rate), Профит-фактор, Макс. просадка, Коэффициент Шарпа (~~годовой, по ТФ бэктеста — `metrics.sharpe_params`, S8R-AUDIT-077; было: по дневным доходностям~~ с v4.10 — годовой по барам теста, §5.17.8), Ср. P&L на сделку (среднее по ВСЕМ), Ср. длительность (часы/дни по ТФ).
- Цветовая кодировка: хорошо (#1a2e1a/#69db7c), нейтрально (#25262B/#C1C2C5), плохо (#2e1a1a/#ff8787). Макс. просадка — всегда нейтральный.
- Benchmark-сравнение: горизонтальные бары (Стратегия / Buy & Hold / ~~IMOEX~~ эталон-индекс — с v4.10 подпись по `benchmark.index_code`: MCFTRR, у старых IMOEX).
- Кнопки CSV и PDF в header.

**Tab "График":**
- Свечной график с маркерами: вход ▲ зелёный (low свечи, 2x), выход ▼ красный (high свечи, 2x). Текст — лоты (фиксированный шрифт, canvas overlay).
- Зоны сделок: цветные полосы между входом и выходом (зелёная = прибыль, красная = убыток).

**Tab "Показатели":**
- Equity curve (LineSeries зелёная #69db7c) + drawdown (AreaSeries красная #ff8787) + Buy & Hold (серая #868E96) + ~~IMOEX~~ эталон-индекс (синяя #74C0FC; с v4.10 — MCFTRR / IMOEX по `benchmark.index_code`, `BenchmarkChart` подписывает «Buy & Hold — с дивидендами за вычетом налога; MCFTRR — индекс полной доходности»).
- Toggle ₽ / % для переключения шкалы. Метрики в левой панели. Легенда.

**Tab "Сделки":**
- `TanStack Table`: #, Тип (Long/Short), Вход (дата+цена, двустрочная ячейка), Выход, Объём, P&L (зелёный/красный), Длительность, Комиссия.
- Фильтр SegmentedControl: Все (N) / Прибыльные (N) / Убыточные (N).
- Сортировка, пагинация (10 записей/страница). Кнопка "Экспорт CSV".

**Лоты и пометка «лоты штуками» (A-05, C4; ТЗ v4.2; ФТ §4.5).** `backtestApi.ts`: `Backtest.lot_size?: number | null`, `legacy_share_lots?: boolean` (§4.3). `BacktestResultsPage` при `legacy_share_lots` — `Alert` «Лоты штуками» (`data-testid="backtest-legacy-lots"`) и `BacktestTrades legacyShareLots`: объём «N шт.» вместо «N лот», колонка CSV «Акций» вместо «Лоты». `BacktestListPage` и `StrategyBacktestsTable` — значок `components/backtest/LegacyShareLotsBadge.tsx` («лоты штуками», подсказка «Посчитан до перехода на лоты: «N лотов» = N акций. Перезапустите, чтобы посчитать в лотах.», `data-testid="backtest-legacy-lots-badge"`). Тесты — `*.legacyLots.test.tsx`.

**«История догружается» (A-02, ТЗ v4.5; ФТ §4.2).** Фронтенд не менялся: строки `HISTORY_PENDING_WARNINGS` приходят в `warning` ответа `GET /backtest/{id}` (§4.3) и показываются существующим жёлтым «Предупреждение» результата (E2E `frontend/e2e/a02-backtest-history-pending.spec.ts`).

**Время по МСК (v4.8, починка времени бэктеста; ФТ §4.5, §19.9).** Правило «naive из БД = UTC» — `utcIsoToUnix` и `parseUtcTimestamp` (`utils/mskTime.ts`); графики «График» и «Показатели» сдвигают время на `MSK_OFFSET_SEC`, как основной график: время свечей, маркеров, зон сделок и кривой капитала совпадает с основным графиком той же бумаги и ТФ, дневка — на своей дате. Таблица сделок («Вход» / «Выход» — «ДД.ММ.ГГГГ ЧЧ:ММ» по МСК; раньше — только дата по UTC; то же в CSV из таблицы) и панель сделки (`TradeDetailsPanel`) — по МСК. Одна сделка — одно время на маркере, в таблице, панели, CSV и PDF; лоты на маркерах, цвет зон и клик по зоне от пояса машины не зависят. **Известные ограничения:** у D/W/M-бэктестов время сделки — «03:00» (метка дневки T-Invest — 00:00 UTC; показывать для D/W/M только дату — отдельная правка формата); у старых бэктестов маркеры и точки кривой на барах выходных и ночи, которых больше нет в основном ряду, на графике не рисуются (бара нет — поведение данных, не этой правки).

**Договор исполнения (A-06, ТЗ v4.10; ФТ §22).** `api/backtestApi.ts` — поля §4.18, `SlippageMode`, константы `DEFAULT_SLIPPAGE_PCT` = 0.1 и `DEFAULT_DIVIDEND_TAX_PCT` = 13 (одни для трёх окон).
- `components/backtest/ExecutionReport.tsx` (`backtest-execution-report`) под метриками «Обзора»: «Sharpe X (баров в году: N, ставка 0)» или «(по константам часов)» при `sharpe_basis = legacy_constants`, бейдж «ненадёжно» (`backtest-sharpe-unreliable`); «Денежный рынок за тот же период: ±X % (источник)» / «нет данных» и бейдж «лучше / хуже денежного рынка»; «Прибыль: от цены …, дивиденды … (налог …)»; `report_notes` строками.
- `LegacyExecutionBadge` («до договора исполнения», `backtest-legacy-execution-badge`, с подсказкой) — в `BacktestListPage`, `StrategyBacktestsTable` и на `BacktestResultsPage` (там же `Alert` «До договора исполнения»); `BacktestListPage` — кнопка «Удалить все старые (N)» (`getLegacyCount`) и окно подтверждения с числом, необратимостью и последствиями → `deleteLegacy`, уведомление «Удалено старых бэктестов: N».
- `BacktestTrades`: строка `row_kind = dividend` — «Начисление дивиденда», цена — «… на акцию (брутто)»; в CSV из таблицы — колонка «Вид»; маркеров на графике у начислений нет.
- Окна запуска: `BacktestLaunchModal` — «Проскальзывание» и «Налог на дивиденды, %»; `GridSearchForm` — «Комиссия, %», «Проскальзывание» (значение и «Единица»), «Налог на дивиденды, %»; запуск из ячейки `GridSearchHeatmap` передаёт значения сетки (`gridSearchUtils.ts`). Под полем проскальзывания — `components/backtest/SpreadWarning.tsx` (`backtestApi.checkSpread`, дебаунс `SPREAD_CHECK_DEBOUNCE_MS` = 500 мс, ответ привязан к значениям запроса): «Результат будет завышен, рекомендуем не меньше X %» / «Спред не проверен: <причина>»; значение поля не меняет.
- E2E и скриншоты — `frontend/e2e/` (спеки A-06), `frontend/e2e/screenshots/a06/`.

**Прогресс бэктеста:**
Во время выполнения — progress bar + текущая дата через WebSocket `backtest:{id}`.

### 6.6 Trading Panel

**Маршрут:** `/trading`

**Компоненты:**
- `ActiveSessionsList` — карточки всех активных сессий. Каждая карточка: тикер, стратегия, **badge таймфрейма** (`1m`/`5m`/.../`D`/`W`/`M`, прочерк для legacy), режим (PAPER жёлтый / REAL красный), unrealized P&L, sparkline, кнопки (Пауза, Стоп, График).
- `SessionDashboard` (`/trading/sessions/:id`) — детальный дашборд сессии:
  - Открытые позиции (таблица): тикер, направление, цена входа, текущая цена, unrealized P&L, время удержания, кнопки (Закрыть, График).
  - Закрытые сделки (таблица).
  - Статистика: дневной P&L, суммарный P&L, Win Rate, количество сделок.
  - Equity curve сессии.
- `LaunchSessionModal` — модалка: выбор стратегии (dropdown), брокерский аккаунт (dropdown), счёт (dropdown), инструмент (search), **селектор таймфрейма** (`<Select data-testid="session-timeframe-select">`, 9 значений с `30m` (A-01, ТЗ v4.0; `SUPPORTED_TIMEFRAMES` в `sessionRequest.ts`), `D` по умолчанию, required), режим (Paper/Real toggle), sizing settings. Для Real — confirm-диалог с предупреждением.
- **A-04 (ТЗ v4.7):** `SessionCard` и `SessionDashboard` — ⚠ причины паузы (`pause-reason`, `pause-reason-details` → `/settings?tab=risk&cb_session=<id>`; текст — `utils/pauseReason.ts::pauseReasonLabel`, только при `status === 'paused'`), «Возобновить» недоступна при включённом стопе (`useStopAllStore`, подсказка `RESUME_BLOCKED_BY_STOP_ALL`); на карточке — пометка `session-limits-badge` («лимиты сессии», по `has_session_limits`) и кнопка `session-limits-btn` «Лимиты» (не у `stopped`) → `SessionLimitsModal` (`GET|PUT|DELETE /circuit-breaker/config/session/{id}`; поля — `components/risk/CbLimitsFields`, плейсхолдер «как в общих: …» из `inherited.text`, проверки — `validateCbForm`). `LaunchSessionModal` — свёрнутый раздел «Лимиты Circuit Breaker (необязательно)» (значения общих — `GET /config/inherited`, поля не монтируются, пока раздел свёрнут), `cb_limits` в `POST /trading/sessions` (`sessionRequest.ts`); 409 `trading_stopped` — текст в окне; при включённом стопе — предупреждение заранее. `useTradingSessionsWS` вливает `pause_reason` и `has_session_limits` из `session_state`.
- **A-06 (ТЗ v4.10; ФТ §22.2–§22.4):** `LaunchSessionModal` — для paper рядом с комиссией «Проскальзывание» (значение и «Единицы» — % / пункты; умолчание и подпись «из последнего бэктеста» / «по умолчанию» — `backtestApi.getSessionDefaults`) с `SpreadWarning` и «Налог на дивиденды, %» (13); у sandbox/real полей проскальзывания нет; `sessionRequest.ts` (`buildSessionStartRequest`, `rerunDraftFromSession`) — три поля запроса. `TradesTable` — колонка «Проскальзывание» в единицах сессии (у sandbox/real — прочерк), раскрытие строки сделки → `TradeExecutionJournal` (`tradingApi`, маршрут `…/executions`; таблица «Вид, Лоты, Время, Правило, Цена правила, Проскальзывание, Цена по договору, Фактическая, Разница», подписи правил — `executionJournalFormat.ts`; пусто — «Журнал исполнений пуст»).

### 6.7 Account / Settings

**Маршрут:** `/account` и `/settings`

**Account (раздел «Счёт»):**
- `BalanceCards` — баланс по каждому брокерскому счёту + валюте (таблица). Мульти-валютность: `RUB`/`USD`/`EUR`/`HKD`/`CNY` без автоконвертации в рубли.
- `PositionsTable` — позиции у брокера (включая внесистемные), колонки: тикер, название, instrument_type, quantity, avg_price, current_price, unrealized_pnl, currency, **badge `source`** (`Стратегия` / `Внешняя`), blocked.
  - Все Decimal-поля приходят с backend как строки (см. Gotcha 1 в `Develop/CLAUDE.md`) и парсятся на фронте через `toNum()`.
- `OperationsTable` — история операций с фильтром по дате (`from`/`to`) и пагинацией (`offset`/`limit` ≤ 500). Типы: `BUY`/`SELL`/`DIVIDEND`/`COUPON`/`COMMISSION`/`TAX`/`OTHER`, состояния: `EXECUTED`/`CANCELED`/`PROGRESS`. Иконки для типов.
- Кнопка «Налоговый отчёт» (экспорт).

**Settings:**
- **Профиль:** смена пароля, email.
- **Брокеры:** список подключённых, добавление нового (форма: тип, API-ключ, discover → выбор счетов), удаление. Ключ отображается маскированным. Имя счёта для показа — `utils/brokerAccountName(name, id)` (S8R-FIX-107): пустое или из пробелов имя (sandbox-счёт T-Invest приходит с `name=""`) → «№<id>»; используется в списке и модалках счетов, форме подключения, окне запуска, отказах конфликта и «Запустить заново», выборе счёта на «Счёте» (`??` пустую строку не ловил). **С A-02 (ТЗ v4.5; ФТ §7.7)** — `BrokerAccountList.tsx`: администратору (`authStore.user.is_admin`) — колонка «Рыночные данные» (подсказка «Этим счётом идут фоновая догрузка истории T-Invest и загрузка дивидендов»), у production-счёта T-Invest — `Switch` (`data-testid="market-data-source-{id}"`, `aria-label` «Счёт для рыночных данных: <имя>»), у остальных — «—»; включённый и неактивный счёт — переключатель недоступен с подсказкой («Чтобы сменить счёт, выберите другой» / «Счёт неактивен: включите его, чтобы выбрать»); удача — уведомление «Счёт для рыночных данных выбран», отказ сервера — текстом над таблицей (`market-data-source-error`). `BrokerSettingsPage.tsx`: администратору при загруженном списке без отмеченного счёта — жёлтый `Alert` «Выберите счёт для рыночных данных: без него фоновая догрузка истории T-Invest и загрузка дивидендов не идут» (`data-testid="market-data-source-missing"`). Не администратору колонки и пометки нет. Пометка строится по своим счетам администратора, а не по ответу сервера (известное ограничение).
- **Telegram:** привязка (показ токена, QR-код), статус привязки, отвязка.
- **Уведомления:** матрица event_type × channel (чекбоксы). С A-03 — строка «Алерт» (`EVENT_TYPE_LABELS.alert`, каналы по умолчанию — от `price_alert`, §5.7). С Б-07 (ТЗ v4.9) — строка «Новость по бумаге» (`EVENT_TYPE_LABELS.news_alert`, каналы по умолчанию — от `alert`, §5.16.5).
- **Риск-менеджмент:** поля из профиля пользователя (daily loss, max drawdown, position size, trade limit). С A-03 умолчания окна часов сессии — `DEFAULT_TRADING_START = '07:00'`, `DEFAULT_TRADING_END = '23:50'` (`src/utils/riskLimits.ts`, зеркало `common/trading_hours.py`; было 10:00–23:50), `RiskSettingsPage.tsx` подписывает их «пусто — 07:00 (начало торгов MOEX)» / «пусто — 23:50 (конец вечерней сессии MOEX)»; E2E — умолчания окна часов. Форма окна часов сессии в интерфейсе CB — блок A-04. **A-04 (ТЗ v4.7):** у каждого поля — охват срабатывания (`components/risk/cbLimitsText.ts`: `CB_SCOPE_HINTS`, у лимитов сессии — `CB_SESSION_SCOPE_HINTS`), общая подсказка `CB_GENERAL_HINT`, текст про пустые поля `CB_EMPTY_FIELDS_HINT` вместо «Пустое поле — без ограничения»; под формой — `CbHistoryTable` (`cb-history`, `cb-history-row`, `cb-history-expand`, `cb-history-details-row`; 20 строк, `Pagination` при `total > 20`, фильтр «Сессия» из `GET /trading/sessions` без параметров — первые 50 (умолчание `limit`); `?cb_session=<id>` в адресе — фильтр, мусор — без фильтра; подписи — `cbHistoryFormat.ts`, `pauseReason.ts::cbCheckLabel`). История и статус перечитываются по `cb.triggered` и `cb.close_report` из `system:{userId}`.
- **AI-провайдеры:** список подключённых провайдеров (таблица: имя, тип, модель, статус, использование). Добавление нового (форма: тип провайдера [Claude/OpenAI/Custom], API-ключ, модель, base URL для custom). Кнопка «Проверить ключ» (тестовый запрос). Переключение активного провайдера. Ключ отображается маскированным. Счётчик использования (сегодня/месяц) и лимиты.
  - **Фоновые задачи (Б-07, ТЗ v4.9; ФТ §20.9, §20.10)** — `components/settings/AIBackgroundCard.tsx` на странице `AISettingsPage`, только администратору (`user.is_admin`; остальным `GET /settings/ai/background` — 403): «Провайдер для фоновых задач» (строки владельца; пусто — «Активная строка»), «Дневной лимит токенов (вход + выход)» с кнопкой «Сохранить», «Расход за сегодня: вход …, выход … токенов, $… (вызовов N)», кнопка «Проверить на эталоне» (`POST /news/ai-benchmark`, опрос `GET` до итога) и итог: пройдена / не пройдена, точность (порог 95 %), полнота (порог 35 из 36), тикеры вне списка, контрольные № 19 / 40 / 71, вызовы, токены, цена, задержка, «Ошибки по заголовкам», таблицы «Тип: эталон → модель» и «Тональность: эталон → модель»; «Модель или подсказка сменились — нужна проверка». API — `api/aiSettingsApi.ts`.
- **Новости (Б-07, ТЗ v4.9; ФТ §20.14)** — вкладка `/settings?tab=news`, `components/settings/NewsSettingsTab.tsx`: четыре `Switch` (`news-setting-collect`, `-ai`, `-alerts`, `-marks`) с описаниями; меняет администратор (`PUT /news/settings`), остальным — «Меняет администратор.» и выключатели только для чтения; при выключенной загрузке нижние недоступны с подсказкой «Сначала включите загрузку новостей», «ИИ-разметка» без проверки — «Сначала пройдите проверку на эталоне в «AI-провайдерах»»; ответ 409 и прочие сбои — `news-settings-error`. Состояние — `newsSettingsStore` (§6.9).
- **Система:** директория для бэкапов, ротация, ручной бэкап, логи.

### 6.8 Notification Center

**Компоненты:**
- `NotificationBell` (в Header) — иконка колокольчика с badge (unread_count). При клике — выдвижная панель справа.
- `NotificationPanel` — список уведомлений (бесконечный скролл). Каждое: иконка severity, заголовок, время (relative: "5 мин назад"), клик → детали / навигация.
- `CriticalBanner` — красный баннер на полную ширину для severity = critical. Отображается поверх контента. Кнопка "Закрыть".
- WebSocket `notifications` канал для push в реальном времени.
- **Отметка «не доставлено» (A-05, D5; ТЗ v4.2; ФТ §10.2):** `components/notifications/UndeliveredChannels.tsx` — `UndeliveredChannels({ channels })` по `Notification.undelivered_channels`: строка на канал (`telegram` → «не доставлено в Telegram», `email` → «не доставлено на почту», `data-channel`), контейнер `data-testid="notification-undelivered"`; один компонент в обеих копиях строки уведомления — `NotificationList` (страница «Уведомления») и `NotificationDrawer` (колокольчик). Видна при следующей загрузке списка.

### 6.9 Zustand Stores

```
authStore:      { user, token, isAuthenticated, login(), logout(), refresh() }
strategyStore:  { strategies, current, versions, fetch(), create(), update() }
backtestStore:  { backtests, current, progress, run(), fetchResults() }
tradingStore:   { sessions, activeSession, positions, launch(), pause(), stop(), closePosition() }
marketStore:    { subscriptions, candles, orderbook, subscribe(), unsubscribe() }
notificationStore: { notifications, unreadCount, fetch(), markRead(), markAllRead() }
healthStore:    { status, components, subscribe() }
settingsStore:  { profile, brokerAccounts, notifSettings, fetch(), update() }

-- v4.3 (Б-02), §6.4 «Панели графика»:
marketDataStore:     { series, streamChannels, instruments, marketStatus, favoriteTimeframes,
                       ensureSeries(), reloadSeries(), fetchOlderSeries(), applyLiveFrame(), ensureInstrumentInfo() }
                       -- фактическое имя «marketStore»; ряды по seriesKey вместо одного набора candles
chartLayoutStore:    { layout, loaded, saveError, load(), setKind(), setActivePanel(), updatePanel(),
                       setPanelTicker(), setLinks(), clearSaveError(), reset() }      -- раскладка на сервере
chartCrosshairStore: { source, publish() }                                            -- перекрестие между панелями
chartDrawingsStore:  { userId, byTicker, currentTool, selectedId, warning, setUser(), ensureTicker(), clearWarning(),
                       add(), update(), remove(), clearAll(ticker), bringToFront(), duplicate() }
                       -- рисунки по бумаге; с v4.6 (Б-04) — на всех ТФ, selectDrawings(state, ticker), предел на бумагу

-- v4.4 (Б-01), §6.10 «Лента новостей»:
newsStore:           { filters {ticker, source, type}, items, nextCursor, loading, loadingMore, loaded, error,
                       sources, tickers, load(), loadMore(), refresh(), setFilter(), loadFilterOptions() }
                       -- курсор «Показать ещё», ответы прежнего «поколения» отбрасываются

-- v4.5 (A-02), §6.7 «Брокеры», §6.11:
settingsStore:       + { accountsLoaded, setMarketDataSource(id) }
                       -- accountsLoaded: список счетов хотя бы раз загружен (до этого «счёт не выбран» не показывается);
                       -- setMarketDataSource: PUT /broker-accounts/{id}/market-data-source, признак ставится выбранному
                       -- и снимается у остальных счетов списка, без глобального loading/error

-- v4.7 (A-04), §6.1, §6.6:
stopAllStore:        { state: StopAllState | null, fetch(), enable(), disable(), applyEvent() }
                       -- источник — GET /trading/stop-all и кадры stop_all.changed канала system:{userId};
                       -- ответ без active — state = null (кнопка доступна, плашки нет)

-- v4.9 (Б-07), §6.4 «Метки новостей», §6.7, §6.10:
newsStore:           + filters { sentiment, from, to }                                 -- тональность действующей разметки; интервал из адреса /news?ticker=&from=&to=
newsSettingsStore:   { settings: NewsSettings | null, loading, loaded, load(), save(patch) → текст ошибки | null, reset() }
                       -- выключатели «Новости» (GET|PUT /news/settings); читают лента (плашка), график (метки), настройки
newsAlertStore:      { alerts, loading, loaded, error, triggers: {id → NewsAlertTrigger[]}, fetch(), create(body),
                       setPaused(id, paused), remove(id), fetchTriggers(id), reset() }       -- «Мои алерты по новостям»
                       -- оба новых стора сбрасываются в authStore.logout

-- v4.11 (Б-03), §6.12 «Скринер» и «Карта рынка»:
screenerStore:       { snapshot, userId, loading, error, threshold, thresholdError, thresholdSaving,
                       fetchSnapshot(userId), loadSettings(), saveThreshold(value) → bool, reset() }
                       -- один снимок на обе страницы; ошибка опроса не стирает прежний снимок; чужой userId — сброс;
                       -- сбрасывается в authStore.logout
```

### 6.10 News — лента новостей (v4.4, Б-01)

**Маршрут:** `/news` (`pages/NewsPage.tsx`, маршрут в `App.tsx`), пункт меню «Новости» (`components/layout/sidebarItems.ts`) — сразу после «Графиков». Требования — ФТ §20.5, API — §4.14.

- **API** `src/api/newsApi.ts`: типы `NewsItem`, `NewsLabel`, `NewsReaction`, `NewsPage`; `NEWS_TYPES` и подписи `NEWS_TYPE_LABELS` (report «отчёт», dividends «дивиденды», deal «сделка», sanctions «санкции», regulator «регулятор», management «менеджмент», exchange «биржа», other «прочее») — подписи только во фронтенде, backend отдаёт коды; `getNews(query)` (пустые фильтры и курсор в запрос не попадают), `getSources()`, `getTickers()`; ответ нормализуется — `{}` и неполные элементы страницу не роняют.
- **Store** `src/stores/newsStore.ts` (§6.9): `NEWS_PAGE_SIZE = 50`; `load()` — первая страница с текущими фильтрами, ответ прежнего «поколения» (фильтр успели сменить) отбрасывается; `loadMore()` — следующая страница по курсору без дублей; `refresh()` — свежая первая страница поверх загруженного (`mergeFresh`: если полная свежая страница не пересекается с загруженным, лента заменяется ею — дыры между ними не остаётся), заодно обновляет список бумаг фильтра, сбой молчит; `setFilter()` — сброс ленты и загрузка; `loadFilterOptions()` — источники и бумаги для фильтров.
- **Страница:** при монтировании — `load()` и `loadFilterOptions()`, затем `refresh()` раз в 60 с (`NEWS_REFRESH_INTERVAL_MS`), пока страница открыта; три `Select` — «Бумага» (`searchable`, `clearable`, «Нет бумаг с новостями»; выбранная бумага остаётся в списке, даже если список ещё не загружен), «Источник», «Тип»; таблица в `Table.ScrollContainer minWidth={960}`; кнопка «Показать ещё» — пока есть `nextCursor`.
- **Строка ленты:** время — `formatDateTime` (МСК, §6.1); заголовок — `Anchor target="_blank" rel="noopener noreferrer"` только для `http(s)`, иначе текстом; бумаги — `Badge`; тип — `Badge` с `flexShrink: 0` (в узкой ячейке «прочее» иначе усыхало до «п…») и «кто разметил» (`словарь` / `ИИ: модель, версия`), без разметки — «не размечена»; реакция — строка на бумагу: `done` — «TICKER: до ±x,xx %, после ±y,yy %» (запятая, настоящий минус, плюс у роста), `*_no_trades` — «— нет торгов», `pending` до срока — «посчитается в ЧЧ:ММ» (`due_at` по МСК), после срока — «считается…», `failed` — «реакция недоступна».
- **Состояния:** загрузка (`news-loading`), пусто (`news-empty`: пустой архив или «ничего не найдено по выбранным фильтрам»), ошибка (`news-error`, кнопка «Повторить»).
- `data-testid`: `news-page`, `news-filter-ticker`, `news-filter-source`, `news-filter-type`, `news-item`, `news-type`, `news-reaction`, `news-load-more`, `news-empty`, `news-error`, `sidebar-news-link`.
- **Известные ограничения:** при ширине окна 1024 px таблица (960 px) не помещается — по горизонтали прокручивается вся страница (с 1280 px помещается; не чинится — решение заказчика); на 1280 px знак «%» реакции может перенестись на отдельную строку (между числом и «%» обычный пробел; с v4.9 — неразрывный, ниже).

**Б-07 (ТЗ v4.9; ФТ §20.11, §20.12, §20.14).**
- **API** `newsApi.ts`: `NewsItem.active`, `NewsItem.ai_note`; `SENTIMENTS` и `SENTIMENT_LABELS` (positive «+», negative «−», neutral «0», mixed «неоднозначно»), `AI_NOTE_LABELS` (disabled «ИИ выключен», no_provider «нет провайдера ИИ», not_verified «модель не проверена», limit «лимит ИИ», unavailable и invalid — «ИИ недоступен»), `NEWS_ALERT_UNCHECKED_LABELS` (ai_unavailable «ИИ недоступен», ai_disabled «ИИ выключен»); `getNews` — `sentiment`, `from`, `to`; `getNewsForTicker`, `getSecurities`, `getNewsSettings` / `updateNewsSettings`, `getNewsAlerts` / `createNewsAlert` / `setNewsAlertPaused` / `deleteNewsAlert` / `getNewsAlertTriggers`, `getAIBenchmark` / `startAIBenchmark` — по §4.17.
- **Страница:** фильтр «Тональность» (`Select`, «Любая»); фильтры из адреса `/news?ticker=&from=&to=` читаются один раз при монтировании и заменяют прежние фильтры ленты целиком (невалидная дата отбрасывается); интервал — значком со снятием (`Снять фильтр по интервалу`); плашка `news-collect-disabled` «Загрузка новостей выключена в настройках. Лента показывает архив.» при `collect_enabled = false`.
- **Строка ленты** — по `item.active`: тип, «словарь» / «ИИ: модель, версия» (`news-labeler`) или «словарь: <пометка>» (`news-ai-note`), у разметки ИИ — «важность N» (`news-importance`); под реакцией — «оценка ИИ: +» (`news-sentiment`, только у `active.labeler = 'ai'`); `tickers` — бумаги действующей разметки.
- **Алерты** — кнопка «Алерт по новостям» (`news-alert-create`) и `components/news/NewsAlertModal.tsx`: «Бумаги» (`MultiSelect` из `GET /news/securities`, бумага фильтра ленты подставлена) / «Весь рынок»; «Слова» — через запятую, подсказка «… с начала слова («ставк» найдёт «ставки», но не «отставки») …» (доработка `0644eb6`); «Тип», «Тональность (оценка ИИ)» с пояснением про ожидание ИИ до 2 минут; отказ сервера (422) — текстом в окне. `NewsAlertsPanel.tsx` — «Мои алерты по новостям»: условие одной строкой (`newsAlertText.alertConditionText`), «на паузе», «срабатываний: N», «Пауза» / «Возобновить», «История» (новость, источник, время, «ИИ» / «словарь», «не проверено: …»; удалённая — «новость удалена из архива»), «Удалить» с подтверждением; при выключенных алертах — «Алерты по новостям выключены в настройках — новости не проверяются.».
- **Тесты:** vitest — `NewsPage`, `NewsAlertModal`, `NewsAlertsPanel`, `NewsSettingsTab`, `AIBackgroundCard`, `newsApi`, `newsStore`, `newsAlertStore`, `newsSettingsStore`; E2E на моках — `b07-news-feed`, `b07-news-alerts`, `b07-settings`, `b07-chart-news`, `b01-news` (ответ с `active`).

### 6.11 Дивиденды и соответствия тикеров (v4.5, A-02)

**Маршрут:** `/dividends/:ticker?` (`pages/DividendsPage.tsx`, маршрут `dividends/:ticker?` в `App.tsx`), пункт меню «Дивиденды» (`components/layout/sidebarItems.ts`, `IconCoin`) — сразу после «Счёта». Требования — ФТ §21.5, API — §4.15.

- **API** `src/api/dividendsApi.ts`: типы `DiscrepancyKind`, `CorrectionAction`, `SmartlabStatus`, `DividendTicker`, `DividendTickersResponse`, `TinvestPayment`, `SmartlabPayment`, `DividendCorrection`, `EffectivePayment`, `DividendRow`, `DividendTable`, `CorrectionCreate`; `dividendsApi.getTickers()`, `getTable(ticker)`, `saveCorrection(ticker, data)`, `deleteCorrection(ticker, id)`. Деньги держатся строкой до показа.
- **Список бумаг:** `Бумага · Расхождения` (`Badge`, жёлтый при > 0) `· Сверка со Смартлабом`; `Switch` «Только с расхождениями», поиск «Тикер»; клик — `/dividends/<тикер>`. Пусто — «Данных о дивидендах пока нет: история T-Invest загружается раз в сутки ночью, сверка со Смартлабом — раз в неделю на бумагу.»
- **Таблица бумаги:** «Дата реестра · T-Invest · Смартлаб · Поправка · Итог» (+ «Действия» у администратора); строка с расхождением — жёлтый фон и подпись вида («Нет в T-Invest», «Нет на Смартлабе», «Сумма или дата отличаются», «Поправка без пары»); у поправки — «Добавлено / Исключено / Исправлено», комментарий, дата, автор; у строки Смартлаба с `canceled` — «отменена на Смартлабе»; сверху — статус сверки и дата загрузки T-Invest («История T-Invest загружена …» / «ещё не загружена» / «не загружается: нет счёта для рыночных данных»).
- **Кнопки** (`rowActions`): `missing_in_tinvest` — «Добавить», «Исключить» (при `canceled` — «Исключить» первой, в окне «Добавить» — предупреждение «На Смартлабе выплата отмечена отменённой»); `missing_on_smartlab` — «Исключить», «Исправить»; `differs` — «Исправить», «Исключить»; `unpaired_correction` и любая строка с поправкой — «Удалить поправку». Окно поправки (`dividend-correction-modal`): «Сумма на акцию, ₽», «Новая дата реестра» («Пусто — дата реестра не меняется»), «Комментарий» («Обязателен: что и почему поправлено»); значения подставлены со Смартлаба; ошибка — «Не удалось сохранить поправку» или текст сервера; после сохранения таблица и список перечитываются.
- **Без счёта:** жёлтый `Alert` «Выберите счёт для рыночных данных: без него выплаты T-Invest не загружаются.» со ссылкой «Настройки → Брокер» (`/settings?tab=broker`) администратору или «Счёт выбирает администратор.»; колонка T-Invest пустая с пометкой (страница прячет ранее загруженные строки, которые отдаёт API).
- `data-testid`: `dividends-page`, `dividends-no-account`, `dividends-only-discrepancies`, `dividends-search`, `dividends-tickers`, `dividend-ticker-{T}`, `dividend-ticker-{T}-discrepancies`, `dividends-empty`, `dividends-error`, `dividends-table-card`, `dividends-ticker-title`, `dividends-smartlab-status`, `dividends-table`, `dividends-tinvest-note`, `dividend-row-{date}`, `dividend-discrepancy-{date}`, `dividend-tinvest-{date}`, `dividend-smartlab-{date}`, `dividend-smartlab-canceled-{date}`, `dividend-correction-{date}`, `dividend-effective-{date}`, `dividend-action-{add|exclude|fix}-{date}`, `dividend-action-delete-{date}`, `dividend-correction-modal`, `dividend-correction-amount`, `dividend-correction-new-date`, `dividend-correction-comment`, `dividend-correction-canceled-warning`, `dividend-correction-error`, `dividend-correction-save`, `sidebar-dividends-link`.

**Соответствия тикеров** — `pages/admin/TickerAliasesCard.tsx` на `AdminLandingPage` (`/admin`): заголовок «Соответствия тикеров» и счётчик, таблица «канонический · тикер ISS · с · по», «Добавить», «Изменить», «Удалить» (подтверждение); поля «Канонический тикер», «Тикер ISS», «С», «По (включительно)»; «с» позже «по» — ошибка до отправки (`ticker-alias-period-error`), отказ сервера — текстом (`ticker-alias-error`). API — `adminApi.ts`. `data-testid`: `ticker-aliases-card`, `ticker-aliases-count`, `ticker-alias-add`, `ticker-alias-modal`, `ticker-alias-ticker`, `ticker-alias-iss-ticker`, `ticker-alias-valid-from`, `ticker-alias-valid-to`, `ticker-alias-save`, `ticker-alias-delete-modal`, `ticker-alias-delete-confirm`.

**График:** фронтенд графика не менялся (зона Б-02) — `marketDataApi.ts` знает поля `history_pending`, `history_pending_reason`, но не показывает их.

**Известные ограничения:** без счёта «Итог» показывает суммы T-Invest без пометки; кнопки поправок только у строк с расхождением.

### 6.12 Скринер, карта рынка и поправки секторов (v4.11, Б-03)

**Маршруты:** `/screener` (`pages/ScreenerPage.tsx`) и `/market-map` (`pages/MarketMapPage.tsx`, `React.lazy` — `d3-hierarchy` не попадает в основной бандл), маршруты `screener` и `market-map` в `App.tsx`; пункты меню «Скринер» (`IconListSearch`, `data-testid="sidebar-screener-link"`) и «Карта рынка» (`IconLayoutGrid`, `sidebar-market-map-link`) в `components/layout/sidebarItems.ts` — сразу после «Дивидендов». Требования — ФТ §23.5–§23.7, API — §4.19; решения — ADR 0094 (карта), 0095 (порог).

- **API и стор:** `src/api/screenerApi.ts` (§4.19). `stores/screenerStore.ts` (§6.9) — один снимок на обе страницы: `fetchSnapshot(userId)` хранит последний удачный ответ, ошибка опроса его не стирает (страница показывает текст над старым снимком; без снимка — текст 503); чужой `userId` — сброс (у снимка свой порог и свои живые цены); `loadSettings()`, `saveThreshold(value)` → `true` / `false` (текст ошибки — `thresholdError`, его показывает поле); `reset()` — в `authStore.logout`.
- **Опрос:** `components/screener/useVisiblePolling.ts` — интервал `pollIntervalMs(meta)` (`screenerFormat.ts`): 5 с при `meta.source = live`, 15 с при `delayed`; на скрытой вкладке (`visibilitychange`) тики пропускаются, вернулась — опрос идёт дальше.
- **Источник цен** — `PriceSourceBadge` (`sourceInfo(meta)`): «live · T-Invest · ЧЧ:ММ» (бирюзовый) или «задержка 15 мин · данные на ЧЧ:ММ» (оранжевый), `data-source="live"|"delayed"`; подсказка — откуда цены и как часто обновляются, у «live» — что бумаги «нет на TQBR», «перерыв» и без сделок за текущий торговый день идут по MOEX ISS со значком часов; плюс `meta.hint`.
- **«Скринер»** (макет «Скринер Б»): заголовок, поиск «Тикер или название», бейдж источника, строка статуса и счётчик «N из 261»; слева `ScreenerSidebar` — `Switch` «Только ликвидные», поле «Порог: медианный оборот, ₽/день» с кнопкой сохранения (ошибка сервера — у поля), фильтры «Сектор», «Уровень листинга», «У брокера» (с числом бумаг у каждого значения), «Индекс IMOEX» («все» / «в индексе» / «вне»), сброс; фильтрация, счётчики значений и сортировка — чистые функции `screenerFilters.ts` (`filterRows`, `facetCounts`, `sectorOptions`). `ScreenerTable` — Mantine `Table` (`stickyHeader`, `ScrollArea`, `miw` 900): «в избранное» (`userFavoritesStore.add('instrument', тикер)`; уже в избранном — «в избранном»), «Бумага», «Цена» (у `price_source = iss` — значок часов с подсказкой «ISS, задержка 15 мин · ЧЧ:ММ»), «Δ день», «Оборот», «Медиана», «Капит.», «Сектор» (значок «Поправка администратора» у `sector_source = override`; цвет значка — `SECTOR_COLORS`), «Ур.», «IMOEX», «У брокера» («торгуется» / «перерыв» / «нет на TQBR», значки «Только для квалифицированных инвесторов» и «Шорт доступен»; пусто — подсказка `meta.hint` или «Данных брокера нет»); сортировка по любой колонке; клик по строке — `/chart/<тикер>`. `liquidity_ready = false` — пометка «Ликвидность считается — показаны все бумаги (первое заполнение истории ISS)», переключатель и поле порога недоступны.
- **«Карта рынка»** (макет «Карта А»): переключатель «Рынок» / «IMOEX», «Только ликвидные», бейдж источника, название открытого сектора и «← Весь рынок». Холст `MarketMapCanvas` (`<canvas>` во всю ширину страницы, `devicePixelRatio`, размер — `ResizeObserver` родителя): раскладка — `treemap.layoutMarket` (`d3-hierarchy` `treemapSquarify`, координаты дробные; «рынок» — корень → группы секторов → плитки по капитализации, «IMOEX» — плитки по весу без групп и зазоров); слияние: плитка с любой стороной меньше `MIN_TILE_PX` = 10 → «ещё N» (`kind: 'more'`, тикеры слитых — в `tickers`), узкая группа (ниже 18 + 10 px или уже 10 px) → «Прочие секторы (N)» (`kind: 'others'`, ключ `others`); бумага без капитализации — счётом в «ещё N». Цвет — `mapColors.tileColor(Δ, meta.scale_pct)` (палитра `MAP_PALETTE_DARK`, насыщенность `min(|Δ| / шкала, 1)`, нет изменения — серый); подпись — `mapView.tileLabel` (где помещается), подсказка `MarketMapTooltip` у курсора (`mapView.tooltipPosition`); какие строки на карте — `mapView.selectMapRows` (по умолчанию `liquid !== false`; открытый сектор или «Прочие» — все бумаги, и неликвидные). `MapLegend` — шкала из семи ступеней: «шкала ±X % — 90-й перцентиль движений за день (250 сессий)» или «шкала ±X % до конца заполнения истории ISS». Клик по заголовку сектора или «ещё N» — сектор; по «Прочим» — карта их секторов; по плитке — `/chart/<тикер>`. Переключение режима и «← Весь рынок» сбрасывают открытую группу.
- **Атрибуты холста для E2E** (пиксели не читаются): `data-tiles`, `data-min-tile` (наименьшая сторона среди плиток и групп), `data-mode`, `data-sector`, `data-sectors`, `data-tickers`, `data-groups` (JSON: ключ, вид, прямоугольник, высота заголовка, секторы группы).
- **Поправки секторов** — `pages/admin/SectorOverridesCard.tsx` на `AdminLandingPage` (`/admin`), по образцу `TickerAliasesCard`: заголовок «Поправки секторов» и счётчик, таблица «Тикер · Сектор · Кто правил», «Добавить» / «Изменить» — окно (поля «Тикер» с подсказкой «Акция TQBR из вселенной скринера» и «Сектор» — `Select` из `GET /screener/sectors`), «Удалить» — подтверждение; отказ сервера — текстом в окне («Не удалось сохранить поправку» или текст 422 / 409).
- `data-testid`: `screener-page`, `screener-search`, `screener-price-source`, `screener-status-line`, `screener-count`, `screener-liquidity-pending`, `screener-loading`, `screener-empty`, `screener-error`, `screener-liquid-toggle`, `screener-threshold`, `screener-threshold-save`, `screener-threshold-error`, `screener-filter-sector`, `screener-filter-level`, `screener-filter-broker`, `screener-filter-imoex`, `screener-filter-imoex-{any|in|out}`, `screener-filters-reset`, `screener-table`, `screener-sort-{ключ}`, `screener-row-{T}`, `screener-fav-{T}`, `screener-delayed-{T}`, `screener-broker-{T}`, `screener-qual-{T}`, `screener-short-{T}`; `market-map-page`, `market-map-mode`, `market-map-liquid-toggle`, `market-map-price-source`, `market-map-sector-name`, `market-map-back`, `market-map-canvas`, `market-map-tooltip` (`data-ticker`), `market-map-legend`, `market-map-scale-note`, `market-map-liquidity-pending`, `market-map-hint`, `market-map-loading`, `market-map-error`; `sector-overrides-card`, `sector-overrides-count`, `sector-overrides-table`, `sector-overrides-empty`, `sector-overrides-error`, `sector-override-add`, `sector-override-modal`, `sector-override-ticker`, `sector-override-sector`, `sector-override-save`, `sector-override-error`, `sector-override-delete-modal`, `sector-override-delete-confirm`; `sidebar-screener-link`, `sidebar-market-map-link`.
- **Тесты:** vitest — `components/screener/__tests__/treemap.test.ts` (раскладка и слияние на данных «без брокера» и «история не готова» при 1200×700, 1168×665, 928×415), `mapColors.test.ts`, `mapView.test.ts`, `pages/__tests__/ScreenerPage.test.tsx`, `MarketMapPage.test.tsx`, `pages/admin/__tests__/SectorOverridesCard.test.tsx` (холст в jsdom не рисует — `getContext` подменяется, размер задаёт подставной `ResizeObserver`); E2E на моках API — `e2e/b03-screener.spec.ts`, `b03-market-map.spec.ts`, `b03-sector-overrides.spec.ts` (моки — `e2e/fixtures/screener_mocks.ts`, скриншоты — `e2e/screenshots/b03/`, в `.gitignore`); бэкенд — `backend/tests/test_screener/`, `backend/tests/unit/test_screener/` на записанных ответах ISS и подставном T-Invest (`backend/tests/fixtures/b03/`).
- **Известные ограничения:** вес IMOEX без «%»; «Ещё 3 бумаг»; переход к графику — только мышью (строка и плитка без клавиатуры); цена без хвостовых нулей; тик опроса не ждёт прежний запрос; без секторов брокера — одна группа «Прочие секторы (2)», подсказка у «ещё N» в «Прочих» неточна.

---

## 7. РЕАЛИЗАЦИЯ БЕЗОПАСНОСТИ

**Ответственный: SEC**

### 7.1 Пароли

- Алгоритм: Argon2id (библиотека `argon2-cffi`)
- Параметры: `memory_cost=65536, time_cost=3, parallelism=4, hash_len=32, salt_len=16`
- Валидация нового пароля (регистрация, создание администратором, смена): минимум **12** символов (до 128), минимум 1 буква + 1 цифра (S8R-AUDIT-043, тип `NewPassword`, правило отдаётся в `setup-status`). Вход со старыми паролями не проверяется политикой; при успешном входе хеш с устаревшими параметрами пересчитывается (`check_needs_rehash`). Хеширование и verify — в `asyncio.to_thread`.

### 7.2 Шифрование API-ключей брокеров

`S8R-AUDIT-042`: шифртекст привязан к записи — к AAD новых записей добавляется контекст (`|broker_account:<id>` для ключей брокера, `|ai_provider:<id>` для ключей AI); формат (MAGIC + key_id, IV) не менялся. Чтение: с контекстом → header-only (записи 041) → легаси; шифртекст, переставленный в чужую строку, не читается, если он записан после 042. `account_id` при записи — обязательный параметр helper'ов. Старые записи привязываются CLI `rebind-encrypted-secrets` (и `rotate-encryption-key`); снимки до перепривязки содержат непривязанные шифртексты — полностью закрывает ротация ключа. Откат кода ниже 042 после перепривязки требует повторного ввода ключей.

`S8R-AUDIT-041`: шифртекст = `MTK\x01` + key_id (8 байт HKDF, info `moex-terminal-key-id`) + AES-GCM, заголовок — AAD; формат без заголовка читается (обратная совместимость). Код до 041 новый формат не читает — откат кода после деплоя требует повторного ввода ключей. Относительный SQLite-путь в `DATABASE_URL` разрешается от корня backend/.

`S8R-FIX-119`: `decrypt_broker_key` / `decrypt_broker_credentials` (`app/broker/crypto_helpers.py`) превращают `InvalidTag` в `BrokerKeyUnreadableError` — подкласс `InvalidTag` (прежние `except InvalidTag` в CLI ротации и `_read_token` работают как раньше) с текстом `BROKER_KEY_UNREADABLE_TEXT` («Ключ брокерского счёта не расшифровывается — введите ключ заново в «Настройки → Брокер».»). У `InvalidTag` текста нет (`str(exc) == ""`), отсюда были пустой блок «❌ » в Telegram `/balance`, 500 на `sandbox-balance` и «брокер не вернул текст» в «Закрыть все». HTTP — 409 `{detail, error_code: "broker_key_unreadable"}` (`register_exception_handlers`, любой endpoint); `close_failure_reason` берёт `detail`, запасной текст — «закрытие не удалось (<класс>), текста ошибки нет» (без ссылки на брокера); лог закрытия — та же причина и `error_type`.

- Алгоритм: AES-256-GCM
- Ключ шифрования: из `ENCRYPTION_KEY` в `.env` (32 байта base64-encoded)
- IV: 12 байт, генерируется `os.urandom(12)` для каждого ключа, хранится в `encryption_iv`
- Tag: 16 байт, аппендится к ciphertext

```python
# common/crypto.py
from cryptography.hazmat.primitives.ciphers.aead import AESGCM

class CryptoService:
    def __init__(self, key: bytes):  # 32 bytes from env
        self._aesgcm = AESGCM(key)

    def encrypt(self, plaintext: str) -> tuple[bytes, bytes]:
        iv = os.urandom(12)
        ct = self._aesgcm.encrypt(iv, plaintext.encode(), None)
        return ct, iv

    def decrypt(self, ciphertext: bytes, iv: bytes) -> str:
        return self._aesgcm.decrypt(iv, ciphertext, None).decode()
```

### 7.3 JWT

- Алгоритм: HS256
- Access Token TTL: 30 минут
- Refresh Token TTL: 7 дней
- Refresh Token: одноразовый; при каждом refresh — старый инвалидируется, выдаётся новый
- При logout: access + refresh → в `revoked_tokens`. `S8R-AUDIT-039`: logout всегда 204 и стирает три cookie (refresh — по path `/api/v1/auth` и прежнему `/api/v1/auth/refresh`), валидный access не требуется; отзыв одной транзакцией — access+rjti при валидном access (Bearer, иначе cookie), refresh-jti при валидном refresh (cookie, иначе тело ≤ 4 КБ); сбой БД не мешает стиранию. Refresh-cookie: HttpOnly, SameSite=Strict, path `/api/v1/auth`. Фронт: `refreshSession` → `refreshed | rejected | unavailable`; повтор только на 429 (≤ 3, `Retry-After` ≤ 10 с), 5xx/сеть — `unavailable` без повтора и без выхода, пауза 10 с; выход только на 401/403.
- Субъект (`sub`), отзыв `jti`, `is_active` и версия токенов проверяются единым предикатом `authenticate_access_payload` в HTTP, на WS-upgrade (отказ — accept + close 4401, S8R-AUDIT-060) и в dash-mount `/api/v1/admin/metrics`. Уже открытые WS при logout/деактивации не разрываются до переподключения (S8R-FIX-017). Деактивированный пользователь: `POST /auth/login` — 401 «Аккаунт деактивирован»; Telegram/email-уведомления не уходят; бот на любую команду и привязку отвечает «Учётная запись отключена администратором» (S8R-AUDIT-013).
- Версия токенов (S8R-AUDIT-035): claim `ver` = `users.token_version` (миграция `c5e8b2a7f913`, по умолчанию 0; токен без `ver` = версия 0). Проверяется в HTTP (`get_current_user`, `AdminAuthASGIMiddleware`), на WS-upgrade (`ws_authenticate`) и в `refresh`. Смена пароля и повторное предъявление использованного refresh (признак кражи) выполняют атомарный `token_version + 1` → все ранее выданные пары недействительны; `PATCH /auth/password` сразу отдаёт новую пару в cookie. Принятый риск: потерянный ответ `/refresh` с повтором старого cookie разлогинивает все устройства (отказ в безопасную сторону).
- Секрет: `SECRET_KEY` в `.env` (в коде — `settings.SECRET_KEY`; имя `JWT_SECRET_KEY` в ранних редакциях ТЗ устарело). При `DEBUG=False`: ≥ 32 байт UTF-8, ≥ 8 различных символов, без префикса `dev-` и маркеров плейсхолдера `change-me`/`change_me`/`changeme` (S8R-AUDIT-034; порог 32 байт — по рецепту карточки и паритету с `ENCRYPTION_KEY`, прежние «64 символа» в коде не проверялись никогда)

> **Примечание для фазы 2:** при переходе к мультипользовательскому режиму рекомендуется миграция на RS256 (асимметричные ключи) для минимизации последствий компрометации ключа. При HS256 компрометация секрета позволяет подделать токены всех пользователей.

### 7.4 CSRF

- Backend генерирует CSRF-токен при логине, возвращает в cookie `csrf_token` (SameSite=Strict, HttpOnly=False чтобы JS мог читать).
- Frontend отправляет этот токен в заголовке `X-CSRF-Token` для всех POST/PATCH/DELETE запросов.
- Backend проверяет совпадение cookie и заголовка.
- `S8R-AUDIT-038`: при cookie-сессии (`access_token`/`refresh_token`) мутирующий запрос требует источник (`Origin`, иначе `Referer`) из `CORS_ORIGINS` и double-submit — без fail-open при отсутствии cookie. login/setup проверяют источник всегда (login-CSRF); logout/refresh — при cookie-сессии; все четыре освобождены от double-submit; без источника при cookie-сессии → 403. Клиенты без cookie (Bearer/CLI/Telegram webhook) не проверяются. Safe-запрос с cookie-сессией без `csrf_token` переиздаёт её (`set_csrf_cookie`). Allowlist (`app/common/origin.py`) нормализуется (http/https, регистр, `/`, порт по умолчанию; `*` и мусор отбрасываются) и общий для CORSMiddleware, WS и CSRF. SPA: `<meta name="referrer" content="strict-origin-when-cross-origin">`.
- logout проверяет источник и без cookie-сессии (чужой Origin → 403) (S8R-AUDIT-039).
- `S8R-AUDIT-040`: Secure у access/refresh/csrf и при их стирании — `app/common/cookies.cookie_secure`: `COOKIE_SECURE` не задан → в production всегда Secure, при `DEBUG=true` — по схеме запроса; `auto` — `url.is_secure` (X-Forwarded-Proto только от доверенного прокси: nginx пропускает `https` лишь от адресов cloudflared); `true`/`false` — принудительно.
- Для WebSocket: двухэтапная авторизация (см. раздел 4.12).

### 7.5 Rate Limiting

Реализация: in-memory sliding window (через `asyncio` + `dict`).

```
Общий лимит API:        100 req/min per user
Торговые операции:       10 req/min per user
Auth (login/setup):      5 req/min по (IP, username) + потолок 20 req/min на IP
Auth (refresh):          5 req/min по (sub, IP)
AI chat:                 10 req/min per user
```

`S8R-AUDIT-037`: IP — IPv4-адрес или IPv6-префикс /64. Реальный IP выставляет uvicorn (`--proxy-headers --forwarded-allow-ips`, доверие только nginx `172.28.0.10`) из `X-Forwarded-For`, который nginx перезаписывает `$remote_addr`; `CF-Connecting-IP` переводит в `$remote_addr` сам nginx (`real_ip_header`, `set_real_ip_from`). IP-потолок проверяется до чтения тела; username берётся из JSON-тела ≤ 4 КБ, иначе ключ только по IP. Состояние in-memory на процесс, не более 10 000 ключей (вытеснение холодных). Блокировка после 5 неудачных входов запрещает только вход (не refresh); по истечении блокировки счётчик обнуляется.

При превышении: HTTP 429 с заголовком `Retry-After`. AI SSE: ≤ 3 параллельных стрима на пользователя, слот занимается первым действием эндпоинта (403), освобождается в `__call__` ответа (S8R-AUDIT-058). Заголовок открыт для JS через CORS `expose_headers` (S8R-AUDIT-039).

### 7.6 HTTP Security Headers

Два слоя (`S8R-AUDIT-057`):

- **Backend** — `SecurityHeadersMiddleware` (`app/middleware/security_headers.py`) на ответы роутов: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy`, HSTS, CSP (ослабленный — только для Dash-префикса `/api/v1/admin/metrics`), `Cache-Control: no-store` по умолчанию (`no-cache` при `ETag`; свой `Cache-Control` эндпоинта сохраняется). Файлы отчётов 3-НДФЛ и экспорта бэктеста — `no-store` явно. WS-хендшейки, preflight и 500 из `ServerErrorMiddleware` middleware не покрывает.
- **nginx** (`nginx.conf`) — тот же набор на **все** ответы (SPA, статика, preflight, 500 backend, 502/504 самого nginx), если backend не прислал свой (`map $upstream_http_* $sec_*`); `add_header … always` только на уровне `server` (свой `add_header` в location отменил бы набор). CSP SPA: `default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'` (Mantine/Blockly), `img-src 'self' data: blob: https://invest-brands.cdn-tinkoff.ru`, `media-src 'self'`, `connect-src 'self' ws://$http_host wss://$http_host`, `object-src 'none'`, `frame-ancestors 'none'`. Кэш: `/assets/` (хешированные) — `public, max-age=2592000, immutable`, остальное (index.html, SPA-fallback) — `no-cache`. Blockly media — локально `/blockly-media/`. Проверка конфигурации — job `nginx-config` в CI (`nginx -t` + curl-проба заголовков).

**Периметр API** (`S8R-AUDIT-059`): Swagger/ReDoc/`/openapi.json` — только при `DEBUG=true`. CORS — явные `allow_methods` (GET/POST/PUT/PATCH/DELETE) и `allow_headers` (Authorization, Content-Type, X-CSRF-Token), `expose_headers` — Retry-After, Content-Disposition; `CORS_ORIGINS` со `*` при `DEBUG=false` — отказ старта (валидатор `Settings` и `check_production_env.sh`, общая нормализация `app/common/origin_syntax.py`), http-origin не-localhost — предупреждение.

**Клиентская сессия и refresh** (`S8R-AUDIT-060`): эпоха сессии — только в `frontend/src/api/session.ts` (`resetSessionState('login'|'logout')`, `onSessionReset`); исход refresh, начатого в прежней эпохе, — `stale` для всех потребителей (HTTP-интерцептор, aiStreamClient, WS), повтор refresh проверяет эпоху перед каждой попыткой; `logout()` ждёт refresh в полёте (≤ 10 с), затем сбрасывает сессию и закрывает WS, затем `POST /auth/logout` — иначе параллельный refresh с отозванной cookie срабатывал как «кража» (reuse-detection) и выбрасывал пользователя на всех устройствах. Поздний 4401 в течение 10 с после свежего refresh — переподключение без нового POST.

### 7.7 Маскирование секретов в логах

`S8R-AUDIT-042`: bot-token Telegram (`<5–12 цифр>:<35 символов>`, путь `/bot<токен>` на любом хосте) маскируется в итоговом тексте: процессор значений structlog `scrub_event_values` + обёртка финального рендерера; для stdlib-логгеров (PTB, uvicorn, apscheduler) — фабрика `LogRecord`: `msg`/`args` поэлементно (структура `args` сохраняется — форматтер access-лога uvicorn не ломается), traceback — в `exc_text` только при найденном секрете. Логгеры `httpx`/`httpcore` — WARNING, если уровень не задан явно. Токен T-Invest в логах — только `token_fingerprint` (sha256[:16]); маска ключа AI — `…abcd`. Реестры мультиплексоров и лимитеров T-Invest ключуются отпечатком токена.

Процессор structlog `mask_secrets` (`app/common/logging_config.py`, S8R-AUDIT-036): значение поля, имя которого целиком или последним `_`-сегментом равно `api_key` / `token` / `secret` / `password` / `encryption_key` (регистронезависимо; `access_token`, `api_secret`, `bot_token` — маскируются, `token_version`, `token_prefix` — нет), заменяется на `***` целиком (хвост токена — тоже утечка, S8R-AUDIT-042; прежнее «`****` + последние 4 символа» не было реализовано). Маскируются только поля верхнего уровня вызова логгера; вложенные словари и текст сообщения — нет.

Traceback исключений — `ConsoleRenderer(exception_formatter=structlog.dev.plain_traceback)`: локальные переменные кадров не печатаются независимо от наличия `rich` (при `rich` дефолт structlog печатал их, включая расшифрованный токен брокера). Там, где в кадрах есть секреты (prefetch рыночных данных, планирование prefetch при логине), вместо `exc_info=True` пишется `error_type` + `error=str(e)`.

### 7.8 Проверка .env при старте

```python
# config.py
class Settings(BaseSettings):
    JWT_SECRET_KEY: str     # min 64 chars
    ENCRYPTION_KEY: str     # base64, 32 bytes decoded
    AI_API_KEY: str = ""
    TELEGRAM_BOT_TOKEN: str = ""
    ...

    @field_validator("JWT_SECRET_KEY")
    @classmethod
    def jwt_key_length(cls, v: str) -> str:
        if len(v) < 64:
            raise ValueError("JWT_SECRET_KEY must be at least 64 characters")
        return v
```

При старте: проверка `os.stat('.env').st_mode` — если файл readable others, выводится WARNING.

**Защита от слабых секретов в production (S8R-AUDIT-034, 2026-09-24):**
При `DEBUG=False` `model_validator` (`config.py::_weak_secret_reason`) для `SECRET_KEY` и `ENCRYPTION_KEY` проверяет по порядку: префикс `dev-` → маркер плейсхолдера `change-me`/`change_me`/`changeme` без учёта регистра (все значения, когда-либо лежавшие в `.env.example`) → длина ≥ 32 байт UTF-8 → ≥ 8 различных символов. Нарушение → `RuntimeError` при импорте настроек: процесс не стартует (прежний `warnings.warn` легко пропускался). Пороги — из `CryptoService` (одна точка истины). `ENCRYPTION_KEY` дополнительно проверяется в `lifespan` первым действием (`get_crypto_service()` в strict-режиме), а не лениво при первой операции шифрования. Preflight контейнера `scripts/check_production_env.sh` применяет те же четыре правила (sync-тесты в `tests/unit/test_config.py`). При `DEBUG=True` dev-дефолты допустимы.

---

### 7.9 Изоляция данных пользователей (S8R-AUDIT-016)

Записи пользователя (стратегии, версии, бэктесты, фоновые jobs, ценовые оповещения, разметка графиков) выбираются по id с фильтром владельца в том же SQL-запросе (`app/common/ownership.py`: `owned_select`, `get_owned_or_404`). Чужой и несуществующий id → одинаковый 404, оракула существования нет. 403 — только admin-доступ, CSRF/Origin и лимиты.

## 8. РАЗВЁРТЫВАНИЕ И ЭКСПЛУАТАЦИЯ

**Ответственный: OPS**

### 8.1 Установка и запуск (нативный, без Docker)

**Системные предусловия:**
- Python 3.11+ с pip
- Node.js 24 LTS с pnpm (jsdom 30 и vitest 5 на Node 20 не работают)
- Системные библиотеки: pango и cairo (WeasyPrint); TA-Lib C library больше не нужна (зависимость удалена 2026-10-19)

**Конфигурация сборки (`backend/pyproject.toml`):**
- В `[tool.setuptools.packages.find]` указать `include = ["app*"]`, чтобы setuptools не включал директории `data/`, `alembic/` как пакеты (flat-layout error).

**Скрипт установки (`scripts/install.sh`):**

```bash
#!/bin/bash
set -e

echo "=== Установка MOEX Terminal ==="

# 1. Создание Python venv
cd backend
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -e ".[dev]"

# 2. Инициализация БД
alembic upgrade head

# 3. Установка frontend-зависимостей
cd ../frontend
pnpm install

# 4. Сборка frontend (production)
pnpm build

echo "=== Установка завершена ==="
echo "Скопируйте .env.example в .env и заполните настройки."
echo "Запуск: ./scripts/start.sh"
```

**Скрипт запуска (`scripts/start.sh`):**

```bash
#!/bin/bash
set -e

# Проверка .env
if [ ! -f .env ]; then
    echo "ОШИБКА: Файл .env не найден. Скопируйте .env.example в .env и заполните настройки."
    exit 1
fi

set -a
source .env
set +a

# Запуск backend
cd backend
source .venv/bin/activate
alembic upgrade head  # Применить новые миграции при обновлении
uvicorn app.main:app --host 127.0.0.1 --port 8000 &   # --reload — только dev (DEBUG=true), scripts/start.sh решает сам (S8R-FIX-033)
BACKEND_PID=$!

# Запуск frontend (dev mode) или раздача статики
cd ../frontend
pnpm dev --port 3000 &
FRONTEND_PID=$!

echo "Backend: http://127.0.0.1:8000"
echo "Frontend: http://127.0.0.1:3000"

# Graceful shutdown
trap "kill $BACKEND_PID $FRONTEND_PID 2>/dev/null; exit" SIGINT SIGTERM
wait
```

**Procfile (для Honcho — опциональный удобный запуск):**

```
backend: cd backend && source .venv/bin/activate && uvicorn app.main:app --host 127.0.0.1 --port 8000
frontend: cd frontend && pnpm dev --port 3000
```

**Production-режим (для финальной установки):**
- Frontend собирается через `pnpm build`, статика раздаётся через FastAPI `StaticFiles` middleware или отдельный Uvicorn static mount.
- Один процесс Uvicorn обслуживает и API, и статику frontend.
- Для автозапуска на macOS — LaunchAgent plist, на Linux — systemd unit.

### 8.2 .env.example

Полный перечень настроек (имя, дефолт, назначение; оператору / служебные / compose) — `docs/env_vars.md` в репозитории кода; тест `test_config.py` сверяет его с `Settings` в обе стороны (S8R-AUDIT-051). Ниже — выдержка с фактическими именами.

```
# === SECURITY ===
SECRET_KEY=<≥ 32 байт, S8R-AUDIT-034>
ENCRYPTION_KEY=<≥ 32 байт>

# === AI ===
AI_PROVIDER=claude
AI_API_KEY=
AI_MODEL=claude-opus-5-5   # дефолт кода (S8R-FIX-033)
AI_GLOBAL_DAILY_TOKEN_LIMIT=1000000
AI_GLOBAL_MONTHLY_TOKEN_LIMIT=20000000

# === TELEGRAM ===
TELEGRAM_BOT_TOKEN=

# === EMAIL (optional) ===
SMTP_HOST=
SMTP_PORT=465
SMTP_USER=
SMTP_PASS=
SMTP_FROM=

# === DATABASE ===
DATABASE_URL=sqlite+aiosqlite:///./data/terminal.db

# === GENERAL ===
LOG_LEVEL=INFO
BACKUP_DIR=backups
BACKUP_KEEP_LAST=7
BACKUP_CRON=03:00
```

### 8.3 Backup

- `sqlite3 .backup` в отдельный файл с timestamp.
- Ротация: хранятся последние `BACKUP_KEEP_LAST` снимков.
- CLI-утилита `python -m app.cli restore --backup /path/to/backup.db`.

### 8.4 Логирование

Приведено к факту в `S8R-FIX-003`: прежний текст описывал подсистему, которой
нет (три файла и `RotatingFileHandler` внутри приложения), и из-за этого
эксплуатация ждала раздельных логов брокера и аудита, а на стенде рос один
файл без предела — 25 МБ к 08.07.

- **Один поток, stdout.** `structlog` → `ConsoleRenderer` (human-readable).
  Отдельного JSON-рендерера и файловых handler'ов в `app/` нет; уровень —
  `LOG_LEVEL` (по умолчанию INFO). Маскирование секретов — процессорами
  `mask_secrets` / `scrub_event_values` и `_ScrubbingRenderer` поверх итоговой
  строки, включая traceback (`S8R-AUDIT-042`).
- **Файл — снаружи процесса.** Дев-стенд: `scripts/start.sh` дублирует stdout
  в `backend/logs/dev.log` через `tee -a`. В Docker файла нет вовсе — логи
  забирает `docker logs` (драйвер демона и решает про ротацию).
- **Ротация — в `start.sh`**, по размеру: `rotate_log` при старте и
  `watch_log_rotation` раз в минуту, `DEV_LOG_MAX_BYTES` × `DEV_LOG_KEEP`
  копий. Внутри приложения ротации нет и не планируется: писатель — `tee`,
  и два механизма на один файл расходились бы.
- **Права.** `umask 077` перед созданием и `chmod 600` на файл и копии
  (`S8R-FIX-003`): в лог попадают идентификаторы счетов, заголовки запросов и
  тексты ошибок брокера, а права по умолчанию отдавали их любому локальному
  пользователю машины.
- **Аудит — не в логах.** Действия пользователя пишутся в таблицу `audit_log`
  (§3.11) с append-only триггерами; файла `audit.log` нет.
- **Вызовы брокера** логируются в общий поток событиями (`tinvest_*`), с
  таймингом и маскированием; отдельного `broker.log` нет.

### 8.5 Health Check

Endpoint `/api/v1/health` — **readiness** (S8R-AUDIT-054):
1. Database: `SELECT 1 FROM users LIMIT 1` (S8R-AUDIT-059: голый `SELECT 1` в SQLite не видит пропавшую схему) → `database: connected | disconnected`.
2. Планировщик: `scheduler_running`, `scheduler_jobs` (число зарегистрированных задач).
3. `cb_state` — агрегат Circuit Breaker по активным сессиям (`ok | warn | triggered | unknown`).
4. `tinvest_connected` — жива ли фоновая задача стрима хотя бы одного мультиплексора T-Invest (gotcha-34) **и** нет ли у него опубликованной потери связи без восстановления (`connection_lost_reported`, S8R-FIX-137: обрыв дольше 30 с в торговые часы — `false` до `connection.restored`); на статус не влияет (paper и ISS-viewer работают без брокера).
5. `version` — версия пакета.

`status: ok` + HTTP 200 — только при `database=connected` и `scheduler_running=true`; иначе `status: degraded` + **HTTP 503** с тем же телом. Healthcheck контейнера (compose, Dockerfile, `start_period` 60 с) смотрит на код ответа. Liveness — `GET /api/v1/health/live` → 200 всегда.
Предел (S8R-FIX-031): вся работа `/health` с БД (проба, проверка токена, `cb_state`, `ai_configured`) — отдельная задача со своей сессией; ответ ждёт её ≤ 3 с (`_READINESS_TIMEOUT_SEC`, меньше `timeout` healthcheck Docker 5 с — Docker получает тело 503), иначе `database: "timeout"` + 503, `cb_state: "unknown"`, `ai_configured` — по серверному ключу (иначе null). Отмена не прерывает запрос в потоке aiosqlite, поэтому зависшая проба доживает в фоне; пока она моложе 60 с, новая не запускается (ответ сразу `timeout`), старше — запускается одна новая. Незабранная ошибка пробы — одно WARNING. Авторизованный ответ дополнительно несёт `ai_configured`/`telegram_configured` («настроено/нет», без сетевых вызовов, на статус не влияют). Проверки AI-провайдера и Telegram `getMe()` по сети в health не делаются.

**Кому что отдаётся** (`S8R-AUDIT-059`): без токена — только `{status}` с тем же HTTP-кодом (healthcheck контейнера не меняется); токен есть и БД доступна — проверка как у `get_current_user` (`user_from_access_token`, первый кандидат Bearer → cookie): отвергнут → 401 (фронт обновляет пару), принят → полное тело; БД недоступна — в БД не ходим, полное тело по одной подписи токена (срок не проверяется; компромисс: отозванный, но подписанный токен при лежащей БД видит состояние систем), `cb_state=unknown`. Одна сессия БД на запрос.

### 8.6 Graceful Shutdown — ✅ реализовано S6

При получении SIGTERM/SIGINT (`app/trading/runtime.py:shutdown()`):
1. Установить флаг `shutting_down = True`.
2. Прекратить приём новых HTTP-запросов.
3. Снять listener'ы с новых свечей (флаг `accepting`) и дождаться текущих циклов обработки свечи. Отправка ордера и запись его id в БД защищены от отмены (`asyncio.shield`, S8R-AUDIT-074, gotcha-75): отменённый вызывающий дожидается их до `ORDER_CANCEL_GRACE_SEC = 45` с.
4. Для каждой активной trading session: сохранить состояние в БД (status → 'suspended'); остановить listener'ы **параллельно**.
5. Прогнать recovery без порога возраста (разрешаются только ордера, найденные у брокера по id/клиентскому ключу; «ордера нет» — по правилу 120 с, S8R-AUDIT-024), затем дождаться pending orders (timeout 30 с); фоновая recovery-задача отменяется только после этого. Весь шаг 3–5 — под общим дедлайном `SHUTDOWN_TOTAL_TIMEOUT_SEC = 45` с; `stop_grace_period` контейнера backend — 60 с (Docker по умолчанию убивает через 10 с).
5. `stream_manager.unsubscribe_all()` — отписка от всех gRPC streaming подписок.
6. Закрыть gRPC-каналы, WebSocket-соединения.
7. Отправить `create_notification(event_type="system_shutdown", title="Система остановлена", severity="warning")` — доставляется в **in-app и Telegram**. ⚠️ Уточнено 2026-07-31: Email уходит только по белому списку `EMAIL_ALLOWED_EVENTS` (`app/notification/email.py`), и `system_shutdown` в него не входит; см. ФТ §19.12.
8. Закрыть БД-соединения.

**Recovery при следующем старте** (реализован в S5R + S6):
- `restore_all()` в `runtime.py` находит сессии со статусом `active`/`suspended`.
- `S8R-FIX-019`: весь `restore_all` обёрнут в `asyncio.wait_for` с общим бюджетом `RESTORE_TOTAL_BUDGET_SEC = 45` (`app/main.py`). У каждого унарного вызова брокера свой дедлайн ~10 с (S8R-AUDIT-030), и при молчащем брокере сумма частных дедлайнов превышала `start_period: 60s` healthcheck — Docker перезапускал backend, restore начинался заново. Запас 15 с — на `alembic upgrade head` и остальной старт, которые входят в тот же `start_period`. Превышение логируется (`session_runtime_restore_budget_exceeded`), не глотается; не уложились — HTTP поднимается без части сессий, их статус в БД остаётся `suspended`/`paused`, подберёт периодический проход или следующий старт.
- `S8R-FIX-018`: `_run_shutdown_recovery` передаёт `_recover_orphan_exit_orders(pause_from=RESTORE_PAUSE_FROM)` — на остановке сессии уже `suspended`, и пауза «только из `active`» частичный выход не доставала. Фоновый цикл остаётся на умолчании `('active',)`. Постоянный отказ старта без владельца — `_pause_unrestorable(..., terminal=True)` → `stopped`: `paused` входит в `BLOCKING_SESSION_STATUSES` и осиротевшая сессия держала бы пару «счёт+тикер» и запрет на удаление счёта навсегда.
- `S8R-FIX-020`: периодическая сверка берёт тот же приём, что restore — отбор сессий вынесен в `_sessions_to_reconcile()` (только БД, до сетевой части), портфели счетов греются параллельно через `_prefetch_restore_portfolios(..., log_prefix=...)` под одним бюджетом, дальше сверка идёт по тёплому кэшу. Последовательный проход тратил до ~55 с на счёт (`POSITION_CHECK_RETRIES`) и всё это время держал watchdog стримов (BUG-27) и доучёт осиротевших ордеров — они в том же цикле.
- Real-сессии: сверка позиций через `get_real_positions()`, расхождения → pause + critical notification.
- Paper-сессии: восстановление виртуального баланса из БД, notification `session_recovered`.
- Возобновление подписок и мониторинга через `SessionRuntime.start()`.
- `S8R-AUDIT-008`: `active` в БД ⇔ живой listener. `suspended → active` — после успешного `start()` (`_set_status_if`); не записалось — listener снимается. Внутри `restore_all` пауза по расхождению и частичному выходу — для `active` и `suspended` (`RESTORE_PAUSE_FROM`), фоновые проверки — только `active`; после recovery статусы перечитываются. Временный сбой `start()` → `suspended`, пара «счёт+тикер» сохраняется, `session_recovered` (warning), если статус записан; `NotFoundError` → `paused`, пара освобождается, без уведомления. `start()` не выходит молча: без владельца — `NotFoundError`, без таймфрейма — `ValidationError`. Не больше одного уведомления о паузе на сессию за restore.
- `S8R-AUDIT-009`: `_prefetch_restore_portfolios` — портфели счетов restore-сессий параллельно (`gather`), дедлайн `POSITION_RECONCILE_TIMEOUT_SEC` на проход, повтор при `BrokerTimeoutError` (`RESTORE_PREFETCH_RETRY_DELAY_SEC`), результат — в `portfolio_cache`; чтение БД — до бюджета. Сверка restore — `cache_only`: нет кэша и есть `_confirmable_trade` (без выхода в полёте, с FIGI) → `PositionsUnconfirmedError` → `paused` + `position_mismatch_note` (если пусто) одной транзакцией, `recovery_mismatch` critical; запись паузы — 1 повтор, затем critical-лог. Путь без дедлайна (фон): `BrokerTimeoutError` → кэш `None` (перевод в одном месте `_load_portfolio`).
- `S8R-AUDIT-010`: пара «счёт+тикер» раздаётся по тирам: `active` → `suspended` (с таймфреймом, исполнимые) → `paused`, внутри — по `id`; цикл — по упорядоченному списку. Судьба проигравшего дубликата — после цикла (`_settle_duplicate_loser`): победитель поднят → `paused` через `_pause_unrestorable` (`cause=duplicate_instrument`, одно уведомление `session_recovered` warning, `session.paused`); не поднят → `suspended` (повтор на следующем старте). Прежний тезис версии 2.7 (3) «статус не переписывается» отменён.
- `S8R-AUDIT-011`: сигнатура сбоя песочницы (gotcha-56) — в портфеле sandbox только деньги при обязательных FIGI (`_required_figis`: подтверждаемые сделки сессий `active`/`suspended`) → перезапрос `POSITION_CHECK_RETRIES` (3–5) с задержкой `POSITION_CHECK_RETRY_DELAY_SEC` (5–10 с; вне диапазона — граница + warning); попытка не начинается, если не влезает в бюджет. Фон: счётчик подряд идущих неполных проходов по (счёт, FIGI) в памяти; < 3 — «сверка не выполнена» без паузы, 3 — расхождение подтверждено (пауза + critical), сброс счётчика; полный ответ сбрасывает. Ответ с другими бумагами без нашей — настоящий. Real-счёт — без повторов. Restore — причина в `portfolio_cache` (`_PortfolioUnconfirmed`: не ответил / без бумаг), честный текст пометки, одно critical.
- `S8R-FIX-124` (найдено на живом стенде): с `S8R-FIX-020` периодическая сверка ходит через `_prefetch_restore_portfolios` (с дедлайном), а счётчик неполных проходов жил только в `_load_portfolio` без дедлайна — неполный ответ песочницы не подтверждался никогда, исчезнувшая у брокера позиция сессию не паузила. Теперь `_reconcile_active_sessions` зовёт предзагрузку с `count_incomplete_passes=True`: `_PortfolioIncomplete` идёт в `_count_incomplete_pass` (подтверждён на `POSITION_INCOMPLETE_CONFIRM_PASSES = 3` — портфель в кэш как «у брокера 0»), ответ по делу сбрасывает счётчики счёта; старт (без флага) — как прежде. Ручная сверка (`reconcile_session_with_broker`, `manual=True`) счётчик не двигает и не подтверждает: неполный ответ — `(True, None)`, «сверка не выполнена» (лог `manual_reconcile_incomplete_portfolio`); полный ответ сбрасывает счётчики только бумаг, присутствующих в нём (`_present_figis`); бумаги самой (остановленной) сессии добавлены в обязательные (`required`), иначе «только деньги» шло бы за настоящий ответ. Пауза сверкой (`_flag_position_mismatch`, `paused_now`) после commit публикует `session.paused` `{session_id, status: "paused", reason}` — раньше события не было вовсе (§11.6, S8R-FIX-097).

### 8.7 Целевые метрики производительности

| Метрика | Целевое значение | Методика замера |
|---|---|---|
| Загрузка главного дашборда | < 2 сек | От HTTP-запроса до полного ответа API (DOMContentLoaded на frontend) |
| Запуск бэктеста (5 лет, Д-таймфрейм) | < 30 сек | От `POST /backtests` до `status=completed` |
| Получение свечи → расчёт индикаторов | < 2 сек | Лог `SignalProcessor.process_candle()` |
| Генерация сигнала → отправка ордера | < 500 мс | От signal до `BrokerAdapter.place_order()` |
| Отображение сделки на графике | < 1 сек | От генерации сигнала до WS push клиенту |
| Telegram-уведомление от сделки | < 3 сек | От события до callback Telegram API |
| Доступность (локальный режим) | Best effort | Graceful shutdown при остановке |

> **Примечание:** Метрика «генерация сигнала → отправка ордера» измеряется от момента, когда стратегия сгенерировала сигнал (после расчёта индикаторов), до момента отправки ордера в API брокера. Время получения и обработки свечи учитывается отдельно (метрика «получение свечи → расчёт индикаторов»).

### 8.8 Ротация секретов

**CLI-утилиты** (`S8R-AUDIT-041`):

- `python -m app.cli rotate-encryption-key [--old-key=K --new-key=K]` (или `OLD_ENCRYPTION_KEY`/`NEW_ENCRYPTION_KEY`; значение, начинающееся с `-`, — только через `=`). Требует остановленного backend: проверяются `127.0.0.1:8000` и `backend:8000`/`BACKEND_COMPOSE_ADDR`, иначе код 3. Новый ключ — правила production-секрета, ≠ `SECRET_KEY`, без `$ # кавычек` и перевода строки. Порядок: read-only проверка старого ключа → снимок БД (`BackupService`) → перешифровка `broker_accounts`, `ai_provider_configs` одной транзакцией; строки без IV пропускаются; повтор идемпотентен по key_id; значения аргументов в выводе не появляются. CLI стартует и при слабых текущих ключах с `DEBUG=false`. После выполнения обновить `ENCRYPTION_KEY` в `.env`.
- На старте сервера key_id ключа сверяется с данными: расхождение → лог `encryption_key_mismatch` (critical), старт не блокируется.
- `python -m app.cli rotate-jwt-secret` — `users.token_version + 1` всем (S8R-AUDIT-035): все сессии завершаются. `SECRET_KEY` в `.env` меняет оператор, затем рестарт.

### 8.9 Стратегия миграции SQLite → PostgreSQL (фаза 2)

**Фаза 1 (SQLite, S8R-AUDIT-097):** движок на `NullPool` — соединение на сессию, PRAGMA WAL / `busy_timeout=30000` / `foreign_keys=ON` на каждом соединении (листенер `connect`); in-memory БД — прежний пул. Причина — вложенные сессии (`create_notification`, `_persist_instrument_cache`): малый пул голодал бы. Цена — ≈ +0,65 мс на открытие сессии. Запись вне бэктеста (налоговый отчёт, корпоративные действия, избранное) при «database is locked» повторяется целиком (`persist_with_retry`, 3 попытки: откат → пауза вне локов → единица работы заново; сеть — вне единицы работы; проведённое корпоративное действие при повторе пропускается). При переходе на PostgreSQL — ограниченный пул.

SQLite-специфичные решения, требующие замены при миграции:
- Триггеры `prevent_audit_delete` / `prevent_audit_update` → PostgreSQL triggers (синтаксис `CREATE FUNCTION` + `CREATE TRIGGER`).
- Partial unique `uq_active_session (WHERE status = 'active')` → PostgreSQL partial unique index (нативная поддержка).
- `sqlite3 .backup` → `pg_dump`.
- WAL mode → стандартная конфигурация PostgreSQL.
- OHLCV-кеш: добавить партиционирование по `ticker` при > 1M записей.

**Инструмент миграции:** Alembic (уже в стеке) с отдельным `env.py` для PostgreSQL.
**Скрипт:** `python -m app.cli migrate-to-postgres --source sqlite:///data/terminal.db --target postgresql://...`

### 8.10 Deployment Architecture (Mac mini production-ready, S8 W3) — ✅ реализовано

Утверждено заказчиком 2026-05-12 (arch_design_s8 §7.2, batch 3 пункт 10). Полный гайд установки: [deployment_guide.md](deployment_guide.md).

**Topology:**

```
End users → Cloudflare edge (TLS termination, DDoS protection)
              ↓ outbound tunnel (cloudflared)
          Mac mini (macOS 14+, Apple Silicon)
              ↓ launchd auto-start at boot
          Docker Compose v2
              ├── moex-frontend (nginx:alpine)  :80
              │     ├── SPA static (React 19 + Vite build)
              │     └── reverse proxy /api/, /ws/ → backend:8000
              └── moex-backend (python:3.11-slim)  :8000
                    ├── FastAPI + Uvicorn
                    ├── SQLAlchemy + alembic (миграции на старте)
                    ├── APScheduler (backup_job, MOEX-calendar, корп. действия, дневная статистика)
                    ├── tinkoff-investments gRPC (patched SDK)
                    └── volumes: sqlite-data, sqlite-backups
```

**Компоненты:**

| Компонент | Образ / технология | Назначение |
|-----------|--------------------|------------|
| `moex-backend` | python:3.11-slim (по digest), multi-stage (builder → runtime) | FastAPI + uvicorn |
| `moex-frontend` | node:24-alpine (builder) → nginx:alpine (runtime) | SPA static + reverse proxy |
| `sqlite-data` (named volume) | bind to /app/data/app.sqlite | Production БД (WAL-mode) |
| `sqlite-backups` (named volume) | bind to /app/backups/ | APScheduler backup_job snapshots |
| launchd plist `com.moex.terminal` | macOS auto-start | `docker compose up -d` при загрузке |
| Cloudflare Tunnel `cloudflared` | TLS termination + публичный домен | Без открытых портов на роутере |

**Безопасность:**
- `.env.production` не коммитится (`.gitignore` правило `.env.*`). Содержит `SECRET_KEY`, `MASTER_KEY`, `TINVEST_TOKEN`.
- nginx внутри контейнера слушает только :80; TLS — на Cloudflare edge.
- `AdminAuthASGIMiddleware` защищает `/api/v1/admin/metrics` (Plotly Dash) — JWT + is_admin check (см. Gotcha 31).
- SecurityHeadersMiddleware (CSP/HSTS/XFO/XCTO/Referrer/Permissions) применяется на всех HTTP-ответах.

**Обновление:** `git pull && docker compose build && docker compose down && docker compose up -d`. Миграции применяются в entrypoint backend контейнера (`alembic upgrade head` перед uvicorn).

**Backup:** `backup_job` в APScheduler (daily 03:00 МСК, см. `app/scheduler/service.py`) → `/app/backups/*.sqlite`. Опционально cron на хосте копирует на внешний диск (см. deployment_guide.md §6.2).

**Monitoring:** `GET /api/v1/health` extended (статус CB, T-Invest, scheduler, jobs). Plotly Dash `/api/v1/admin/metrics` показывает performance графики (только admin).

**Файлы:**
- `Develop/docker-compose.yml`, `Develop/Dockerfile.backend`, `Develop/frontend/Dockerfile`, `Develop/nginx.conf`, `Develop/.dockerignore`.
- `Документация по проекту/launchd/com.moex.terminal.plist`.
- `Документация по проекту/deployment_guide.md` — 9 разделов (платформа, предусловия, установка, launchd, Cloudflare Tunnel, backup/restore, обновление, мониторинг, troubleshooting).

---

## 9. СТРАТЕГИЯ ТЕСТИРОВАНИЯ

**Ответственный: QA (координация, E2E, ручное тестирование), все специалисты (unit-тесты своих модулей)**

### 9.1 Unit-тесты (pytest)

**Coverage target: >= 80%**

| Модуль | Ключевые тест-кейсы |
|---|---|
| **auth** | Хеширование/верификация пароля; JWT генерация/валидация/отзыв; блокировка после 5 неудачных попыток; разблокировка через 15 мин |
| **strategy/ir_codegen** | Генерация кода для каждого типа блока из IR; паритет с интерпретатором; edge-case: пустая стратегия; невалидные параметры |
| **strategy/block_parser** | Парсинг валидного шаблона; отсутствующие секции; некорректный формат индикаторов |
| **backtest/engine** | Расчёт P&L (long); учёт комиссии и проскальзывания; расчёт метрик (Sharpe, drawdown, profit factor); корректность equity curve; учёт лотности; учёт НКД для облигаций |
| **trading/order_manager** | Создание market/limit/stop ордеров; валидация шага цены; округление до лотности |
| **trading/position_tracker** | Открытие/закрытие позиции; расчёт unrealized P&L; частичное исполнение; расчёт slippage |
| **circuit_breaker** | Каждый вид лимита: daily loss, max drawdown, position size, trade count; cooldown; блокировка short; блокировка дубликатов |
| **market_data/validator** | High < Low detection; gap detection с учётом календаря; аномальные значения; дубликаты |
| **sandbox/ast_analyzer** | Все запрещённые конструкции; валидный код проходит; edge-case escape-техники |
| **common/crypto** | Encrypt/decrypt round-trip; различные IV; невалидный ключ |

**Окружение тестового движка (S8R-FIX-023/024, 2026-10-13).** `tests/conftest.py`:

- `PRAGMA foreign_keys=ON` ставится на **каждое** соединение тестового движка — прод (`app/common/database.py`) проверку включает, а тесты шли без неё и не видели того класса ошибок, который на бою даёт `IntegrityError` в рантайме (фиктивный `user_id`, сделка на удалённую сессию, версия без стратегии). Снимается **только** на teardown: `Base.metadata.drop_all` роняет таблицы в порядке, который не удовлетворяет циклу `strategies ↔ strategy_versions`, и падал бы на `DROP TABLE strategy_versions`; проверять FK при сносе схемы незачем.
- `GRPC_ENABLE_FORK_SUPPORT=0` выставляется **до** импортов, тянущих grpc: обработчики `pthread_atfork` ставятся при инициализации библиотеки, и та же переменная в окружении подпроцесса бесполезна. Это снимает SIGABRT «`fork_posix.cc`: Other threads are currently calling into gRPC» в тестах, запускающих подпроцессы (`test_config.py::test_preflight_*`), из-за которого в разных прогонах падали разные параметризации.
- `faulthandler_timeout=400` живёт в `addopts` (`pyproject.toml`), а не в памяти оператора: зависший прогон даёт стеки, а не вечное ожидание.
- Реестр `keyed_lock` — процессный (S8R-FIX-123): autouse-фикстура `_reset_keyed_locks` зовёт `app.common.locks._reset_for_tests()` до и после каждого теста. `asyncio.Lock`, переживший свой тест (ссылка из traceback, gotcha-71), привязан к его event loop, и следующий тест с тем же ключом падал «… is bound to a different event loop» (флак `delete_account_race`). В приложении цикл один, реестр не сбрасывается.
- `audit_log.user_id` объявлен `ON DELETE SET NULL`, а таблица защищена append-only триггерами — конфигурации противоречат друг другу, и удаление пользователя с аудитом обрывается триггером. Поведение закреплено тестом (`tests/unit/test_audit_log_user_delete.py`, триггеры создаются из того же DDL, что в `alembic/env.py`); менять FK — перестройка append-only таблицы с историей, решение заказчика.

**CLI в тестах (S8R-FIX-002).** Подпроцесс `python -m app.cli.*` обязан получать окружение, в котором валидатор настроек проходит (`DEBUG=true` + `DATABASE_URL` временной БД), и проверяться **по эффекту** (запись в БД, код возврата), а не по «код возврата не ноль»: иначе тест зелёный на процессе, упавшем до работы. Окружение подпроцесса не наследует переменные, совпадающие с полями `Settings` (S8R-FIX-096, `test_start_scripts._env_without_settings`): у pydantic-settings окружение сильнее файла, и `DEBUG=false` job'а CI перекрывал `DEBUG=true` временного `.env` — тест был зелёным только локально. Тест данных restore-CLI (`test_backup_cli`) запускается с `--server-stopped` и не зависит от занятости :8000; охрану порта проверяет отдельный тест на своём эфемерном порту.

### 9.2 Интеграционные тесты

| Тест | Описание |
|---|---|
| **Broker adapter (mock)** | Mock gRPC-сервер, эмулирующий T-Invest API: place order, get positions, candle stream. Проверка маппинга, rate limiting, reconnect |
| **Database operations** | Полный CRUD для каждой таблицы; каскадное удаление; unique constraints; audit_log append-only |
| **AI service (mock)** | Mock LLM-провайдер: проверка prompt construction, context compression, budget tracking |
| **Sandbox execution** | Запуск безопасного кода → success; запуск вредоносного кода → block; CPU timeout; memory limit |
| **Template parser → code generator** | End-to-end: шаблон текст → блоки → код → валидация кода (компилируется без ошибок) |
| **WebSocket** | Подписка/отписка; получение обновлений; heartbeat; авторизация |

### 9.3 E2E-тесты (Playwright)

| Сценарий | Шаги |
|---|---|
| **Полный цикл бэктеста** | Login → Создать стратегию (блоки) → Сгенерировать код → Запустить бэктест → Дождаться результатов → Проверить метрики и график |
| **Paper trading** | Login → Выбрать стратегию → Запустить Paper → Проверить появление карточки → Подождать сигнала (mock market data) → Проверить сделку в таблице |
| **Circuit breaker** | Login → Запустить стратегию → Имитировать серию убыточных сделок → Проверить срабатывание CB → Проверить уведомление → Стратегия на паузе |
| **Восстановление после перезапуска** | Запустить стратегию → Имитировать остановку backend → Перезапустить → Проверить: сессия восстановлена, уведомление пришло |
| **First run wizard** | Открыть пустое приложение → Пройти мастер настройки → Проверить: пользователь создан, можно логиниться |

### 9.4 Инструменты

- **Backend:** pytest + pytest-asyncio + pytest-cov + httpx (TestClient)
- **Frontend:** Vitest + React Testing Library + MSW (Mock Service Worker)
- **E2E:** Playwright
- **CI:** GitHub Actions → lint (ruff + eslint) → type check (mypy + tsc) → unit tests → integration tests → E2E (при наличии)

**Окружение backend-job в CI (S8R closeout, 2026-07-28).** `app/config.py` содержит валидатор `check_production_secrets`: при `DEBUG=False` значения `SECRET_KEY`/`ENCRYPTION_KEY`, начинающиеся с `dev-`, **жёстко** роняют приложение на импорте. В CI нет `.env`, поэтому job `backend` обязан задавать секреты сам — иначе `pytest` падает не на тестах, а на `import app.main` из `tests/conftest.py`.

Контракт:

| Переменная | Значение в CI | Почему так |
|---|---|---|
| `DEBUG` | `false` | Тесты идут в том же режиме, что и production-сборка: `CryptoService` в strict-режиме, auth-cookie с `Secure`. `DEBUG=true` заглушил бы гейт секретов целиком |
| `SECRET_KEY` | не-дефолтное фиктивное значение | Гейт остаётся живым: возврат на `dev-*` снова уронит job |
| `ENCRYPTION_KEY` | не-дефолтное фиктивное значение, **≥ 32 байт** | Требование `CryptoService` (AES-256-GCM через HKDF), проверяется в strict-режиме |

Значения — публичные тестовые константы прямо в `ci.yml`. Реальные секреты в CI не заводятся, `.env` не коммитится.

**E2E и `networkidle` (S8R closeout).** В спеках **запрещён** `page.waitForLoadState('networkidle')`: терминал держит постоянный WebSocket, сеть не «затихает», и длинный прогон встаёт дольше собственного `timeout` теста без единого `failed`. Ждать нужно конкретное условие — `expect(locator).toBeVisible()` или `waitForResponse`. Подробности — `Develop/stack_gotchas/gotcha-46-e2e-networkidle-and-vite-cache.md`.

**Схема БД против моделей (S8R closeout).** Alembic-миграции обязаны полностью покрывать `Base.metadata`: проверка `tests/unit/test_migration.py::test_fresh_db_schema_matches_models` применяет миграции к пустой БД и сверяет **и таблицы, и колонки**. Сверка только по списку таблиц дефект не ловит — расходятся и колонки внутри существующих. Любая новая колонка/таблица в моделях без миграции ломает **чистую установку** (существующие стенды при этом работают, и дефект остаётся невидимым до развёртывания). Миграции, догоняющие drift, писать **идемпотентными** (добавлять объект только если его нет), иначе `upgrade head` упадёт на рабочих БД.

**`auth-hardening.spec.ts`** работает против ЖИВОГО backend (cookie-flow не мокается) и требует отдельного стенда: адреса задаются через `PW_API_URL` / `PW_WS_URL` / `PW_ORIGIN`, значение `PW_ORIGIN` обязано совпадать с `CORS_ORIGINS` backend'а, а `LOGIN_RATE_LIMIT_PER_MINUTE` на стенде должен быть не ниже 10 (спека логинится в 6 тестах из 7, дефолт — 5).

### 9.5 Роль QA-инженера в процессе тестирования

QA-инженер встроен в процесс разработки с первого спринта, а не только в финальную фазу:

| Спринт | Задачи QA |
|---|---|
| S1 | Подготовка тест-кейсов по wireframes. Ручное тестирование авторизации |
| S2 | Тест-кейсы для графиков и брокерского подключения. Ручная проверка |
| S3 | Первые E2E-тесты (Playwright): создание стратегии, Blockly, mode B |
| S4 | E2E: полный цикл бэктеста, AI chat (mock). Верификация метрик |
| S5 | E2E: Paper Trading, Circuit Breaker. **Верификация финансовых расчётов** (P&L, лотность, НКД) |
| S6 | E2E: уведомления, восстановление. **Security-тестирование** (совм. с SEC): CSRF, sandbox, brute-force |
| S7 | E2E: Should-фичи (версионирование, экспорт, алерты) |
| S8 | Регрессионный прогон всех E2E. Дополнение пропущенных сценариев. Performance-тестирование (совм.) |

**Принцип:** каждый спринт завершается QA-приёмкой. Баги фиксируются в текущем спринте, а не копятся до S8.

---

## 10. ФАЗЫ РАЗРАБОТКИ

> **Примечание:** Детальное распределение задач по исполнителям, ревьюерам, зависимостям и параллельности — см. **План работ команды (development_plan.md v2.0)**. Ниже приведено краткое описание спринтов, их целей и результатов.

### Состав команды (9 человек)

| # | Роль | Сокращение |
|---|---|---|
| 1 | Технический архитектор | ARCH |
| 2 | Backend-разработчик #1 (senior) | BACK1 |
| 3 | Backend-разработчик #2 | BACK2 (объединяет бывш. DB + AI) |
| 4 | Frontend-разработчик #1 (senior) | FRONT1 (Blockly, Charts, Trading) |
| 5 | Frontend-разработчик #2 | FRONT2 (формы, настройки, дашборд) |
| 6 | UX/UI-дизайнер | UX |
| 7 | QA-инженер | QA |
| 8 | DevOps-специалист | OPS |
| 9 | Специалист по безопасности | SEC |

### Спринт 1 (2 недели): Фундамент

**Все специалисты параллельно. UX начинает wireframes с первого дня.**

| Задача | Ответственный |
|---|---|
| Wireframes ключевых экранов + UI-kit Mantine (dark theme) | UX |
| Инициализация проекта: backend (FastAPI + SQLAlchemy), frontend (React + Vite + Mantine) | OPS + BACK1 + FRONT1 |
| Скрипты установки/запуска (install.sh, start.sh, .env) | OPS |
| БД: все модели SQLAlchemy, Alembic init + первая миграция | BACK2 |
| Auth module: регистрация, логин, JWT, refresh, brute-force protection | SEC |
| Frontend shell: layout, sidebar, header, routing, dark theme (по wireframes UX) | FRONT1 + FRONT2 |
| Health endpoint | BACK1 |
| CI/CD pipeline (GitHub Actions) | OPS |
| Тест-кейсы авторизации + ручное тестирование | QA |

**Результат:** Можно залогиниться, увидеть дашборд по wireframes, health endpoint работает. Wireframes утверждены. QA подтвердил auth.

### Спринт 2 (2 недели): Данные и графики

| Задача | Ответственный |
|---|---|
| UX: дизайн экранов графиков и настроек брокера | UX |
| Broker Adapter: T-Invest — подключение, get_accounts, get_balance, get_historical_candles | BACK1 |
| MOEX ISS клиент (fallback) + MOEX Calendar Service | BACK2 |
| Market Data Service: кеш OHLCV, загрузка, валидация | BACK1 + BACK2 |
| Crypto Service (AES-256-GCM) для API-ключей | SEC |
| Frontend: Графики (TradingView LW Charts), таймфрейм, volume (по дизайну UX) | FRONT1 |
| Frontend: Настройки брокера, добавление API-ключа | FRONT2 |
| Тест-кейсы + ручное тестирование: графики, брокер, шифрование | QA |

**Результат:** Подключён брокер, можно просматривать графики. API-ключи зашифрованы. QA подтвердил.

### Спринт 3 (2 недели): Стратегии + Блочный редактор + Защита API

| Задача | Ответственный |
|---|---|
| UX: дизайн Strategy Editor (Blockly workspace, AI chat sidebar) | UX |
| Strategy CRUD API + модели | BACK1 |
| Code Generator: блоки → Python/Backtrader | BACK2 |
| Template Parser (режим B) | BACK2 |
| Code Sandbox: код только из IR блоков + AST-анализ + дочерний процесс с лимитами (RestrictedPython удалён, S8R-AUDIT-050) | SEC |
| **CSRF protection (middleware, токены)** | SEC |
| **Rate limiting внутреннего API** | SEC |
| Blockly: кастомные блоки (индикаторы, условия, сигналы) | FRONT1 |
| Frontend: Strategy Editor page (по дизайну UX) | FRONT1 |
| Frontend: режим B — модальное окно | FRONT2 |
| E2E-тесты: создание стратегии, Blockly, mode B | QA |

**Результат:** Стратегия создаётся в блочном редакторе или через шаблон. Sandbox блокирует вредоносный код. **API защищён CSRF и rate limiting.** QA E2E пройден.

### Спринт 4 (2 недели): AI-интеграция + бэктестинг

| Задача | Ответственный |
|---|---|
| UX: дизайн экранов бэктеста + AI sidebar chat | UX |
| AI Service: pluggable providers, хранение ключей в БД | BACK2 |
| AI chat API + SSE streaming | BACK2 + BACK1 |
| Backtest Engine: Backtrader wrapper, метрики, equity curve, benchmark | BACK1 |
| Backtest API + прогресс через WebSocket | BACK1 |
| Frontend: AI sidebar chat | FRONT2 |
| Frontend: Backtest Results (3 вкладки, по дизайну UX) | FRONT1 |
| Frontend: форма запуска бэктеста + Настройки AI-провайдеров | FRONT2 |
| E2E-тесты: полный цикл бэктеста, AI chat (mock) | QA |

**Результат:** Полный цикл: стратегия → бэктест → результаты. AI-ключи через UI. QA E2E пройден.

### Спринт 5 (2 недели): Торговля + Bond/Corporate Actions — ✅ завершено + S5R + S5R-2

| Задача | Ответственный | Статус |
|---|---|---|
| UX: дизайн Trading Panel (Paper vs Real визуализация) | UX | ✅ S5 |
| Trading Engine: session manager, signal processor, order manager, position tracker | BACK1 | ✅ S5 |
| Paper Trading Engine (виртуальный портфель; T+1 по деньгам не моделируется — ФТ §7.4, уточнено в ТЗ v4.2) | BACK1 | ✅ S5 |
| Broker Adapter: place_order, cancel_order, subscribe_candles (streaming) | BACK1 | ✅ S5 |
| Rate limiter для брокерского API | BACK1 | ✅ S5 |
| Circuit Breaker (все виды лимитов) | SEC | ✅ S5, уточнено в S6 (temporary блокировки) |
| **Bond Service (расчёт НКД) + Corporate Action Handler** | BACK2 | ✅ S5 |
| **Налоговый экспорт (Tax Report Generator, FIFO, 3-НДФЛ)** | BACK2 | ✅ S5 |
| Frontend: Trading Panel (по дизайну UX) | FRONT1 | ✅ S5 |
| Frontend: Real-time графики через WS + индикаторы | FRONT1 | ✅ S5 |
| Frontend: экран «Счёт» (баланс, позиции, налоговый отчёт) | FRONT2 | ✅ S5 |
| E2E-тесты: Paper Trading, Circuit Breaker, Bond P&L | QA | ✅ S5 (23 E2E) |

**Результат:** ✅ Paper Trading + Circuit Breaker + НКД + корп. действия. 548 backend tests + 23 E2E. **S5R (closeout):** Live Runtime Loop замкнут, реальные позиции T-Invest через FIGI, schema drift устранён, 109 Playwright на моках, 15 Stack Gotchas. **S5R-2:** Chart hardening (prefetch, sequential-index, 401 fix, TF-aware upsertLiveCandle).

### Спринт 6 (2 недели): Уведомления + восстановление — ✅ завершено

| Задача | Ответственный | Статус |
|---|---|---|
| UX: дизайн Notification Center + Telegram-сообщений | UX | ✅ S6 |
| Telegram-бот: привязка 6-digit TTL 5min, уведомления, webhook | BACK2 | ✅ S6 DEV-2 |
| Email-уведомления (SMTP, aiosmtplib, авто-TLS 465/587) | BACK2 | ✅ S6 DEV-2 |
| In-app уведомления (WS `/ws` multiplexer) + Event Bus | BACK1 | ✅ S6 DEV-1 |
| Восстановление после перезапуска (Paper + Real с сверкой позиций) | BACK1 | ✅ S6 DEV-1 |
| Graceful Shutdown (pending orders 30s timeout, multi-channel notification) | OPS + BACK1 | ✅ S6 DEV-1 |
| Frontend: Notification Center (Bell, Drawer, CriticalBanner, /notifications page) | FRONT2 | ✅ S6 DEV-4 |
| T-Invest SDK upgrade beta59 → beta117, Stream Multiplex (persistent gRPC) | BACK1 | ✅ S6 DEV-3 + DEV-5 |
| E2E infrastructure (playwright-nightly CI, ISS mocks, ~40 тестов на моках) | QA | ✅ S6 DEV-6 |
| E2E S6 + Security (CSRF, rate limit, sandbox escape, notification ACL) | QA + SEC | ✅ S6 DEV-7 (10 E2E + 23 security) |
| **Должны бы быть в S7, сделаны в S6:** Should-команды Telegram (/close, /closeall, /balance), Scheduler Service (APScheduler), Price alerts, расширенные карточки сессий, PauseConfirmModal, маркеры сделок на графике | BACK2 + BACK1 + FRONT1 | ✅ S6 (ARCH follow-up + closeout) |

**Результат:** ✅ 685 backend + 250 frontend + 10 E2E = **945 тестов, 0 failures**. 18 Stack Gotchas. PASS (чистый).

### Спринт 7 (2 недели): Полировка + Should-фичи

> **Примечание:** Must-задачи (Bond, Corporate Actions, налоговый экспорт) перенесены в S5. Часть Should-фич (Telegram команды, Price alerts) реализованы досрочно в S6. Здесь — оставшиеся Should + техдолг.

| Задача | Ответственный | Статус |
|---|---|---|
| UX: финальная полировка UI, консистентность, юзабилити-тест | UX | ⬜ S7 |
| Версионирование стратегий (Should) | BACK1 + FRONT2 | ⬜ S7 |
| Grid Search оптимизация (Should) | BACK1 + FRONT2 | ⬜ S7 |
| Экспорт CSV/PDF (Should) | BACK2 | ⬜ S7 |
| Telegram-команды /close, /closeall, /balance (Should) | BACK2 | ✅ уже в S6 |
| Ценовые алерты (Should) | BACK1 + FRONT2 | ✅ уже в S6 |
| Inline-кнопки Telegram (CallbackQuery handler) | BACK2 | ⬜ S7 (перенос из S6) |
| NotificationService singleton через DI | BACK1 | ⬜ S7 (перенос из S6 Review) |
| 5 event_type подключить к runtime (`trade_opened`, `partial_fill`, `order_error`, `all_positions_closed`, `connection_lost/restored`) | BACK1 | ⬜ S7 (перенос из S6) |
| Инструменты рисования на графике (Should) | FRONT1 | ⬜ S7 |
| Дашборд: виджет баланса, health indicator, sparklines | FRONT1 + FRONT2 | ⬜ S7 |
| Дисклеймер (first-run wizard) | FRONT2 + BACK2 | ⬜ S7 |
| Backup/restore | OPS + BACK1 | ⬜ S7 |
| WS-обновление карточек сессий (вместо polling 10s) | BACK1 + FRONT1 | ⬜ S7 (плановая задача) |
| Интерактивные зоны бэктеста + аналитика | FRONT1 | ⬜ S7 (из memory) |
| Фоновый запуск бэктеста (в header) | BACK1 + FRONT1 | ⬜ S7 (из memory) |
| E2E-тесты S7 | QA | ⬜ S7 |

**Результат:** Phase 1 feature-complete. UX утвердил финальный вид интерфейса.

### Спринт 8 (2 недели): Регрессия + стабилизация

> **Примечание:** Основной объём тестов написан в S2–S7 (QA встроен в каждый спринт). S8 — регрессия, performance, security audit, юзабилити-тест.

| Задача | Ответственный |
|---|---|
| Регрессионный прогон всех E2E-тестов | QA |
| Дополнение unit-тестов: coverage ≥ 80% | BACK1 + BACK2 + SEC |
| Frontend unit-тесты (ru-RU, Blockly, WS reconnect) | FRONT1 + FRONT2 |
| Performance testing: целевые метрики | BACK1 |
| Security audit: crypto, sandbox, CSRF, headers | SEC + QA |
| UX: финальный юзабилити-тест с пользователем | UX |
| Bug fixing по результатам тестов | Все |
| Исправление UX-багов по юзабилити-тесту | FRONT1 + FRONT2 |
| Документация: README, deployment guide | OPS |

**Результат:** Production-ready Phase 1. Sign-off: ARCH + QA + UX.

### Зависимости между спринтами

```
Спринт 1 (фундамент + wireframes)
    ↓
Спринт 2 (данные + графики)     ← зависит от auth и DB
    ↓
Спринт 3 (стратегии + CSRF)     ← зависит от market data
    ↓
Спринт 4 (AI + бэктест)         ← зависит от стратегий и sandbox
    ↓
Спринт 5 (торговля + Bond)      ← зависит от бэктест engine и broker
    │
    ├──────────────────┐
    ▼                  ▼
Спринт 6 (уведомления)   Спринт 7 (Should — параллельно)
    │                  │
    ├──────────────────┘
    ▼
Спринт 8 (регрессия)            ← после feature freeze
```

---

## СВОДНАЯ МАТРИЦА ОТВЕТСТВЕННОСТИ

> **Примечание v2:** Матрица обновлена под 9 ролей. Полная RACI-матрица — см. development_plan.md v2.0.

| Модуль / Задача | ARCH | BACK1 | BACK2 | FRONT1 | FRONT2 | UX | QA | OPS | SEC |
|---|---|---|---|---|---|---|---|---|---|
| Архитектурные решения | **P** | R | R | R | | R | | R | R |
| Code review (межмодульные) | **P** | | | | | | | | |
| Code review (backend) | | **P** | | | | | | | |
| Code review (frontend) | | | | **P** | | R | | | |
| Схема БД, миграции | | R | **P** | | | | | | R |
| FastAPI app, middleware | | **P** | R | | | | | | R |
| Auth module (JWT, Argon2) | | R | | | | | | | **P** |
| Strategy Engine (parser, codegen) | | R | **P** | | | | | | |
| Blockly editor | | | | **P** | | R | | | |
| AI Service (providers, prompts) | | R | **P** | | | | | | R |
| AI chat frontend | | | | | **P** | R | | | |
| Backtest Engine | | **P** | | | | | | | |
| Trading Engine | | **P** | | | | | | | R |
| Broker Adapter (T-Invest) | | **P** | | | | | | | |
| Market Data Service | | **P** | R | | | | | | |
| Charts (TradingView) | | | | **P** | | R | | | |
| Notification Service | | | **P** | | R | | | | |
| Telegram Bot | | | **P** | | | | | R | R |
| Circuit Breaker | | R | | | | | | | **P** |
| Code Sandbox | | R | | | | | | | **P** |
| Crypto, CSRF, Rate Limiting | | | | | | | | | **P** |
| Bond Service, Corp. Actions | | | **P** | | | | | | |
| Tax Report Generator | | | **P** | | R | | | | |
| Wireframes, UI-kit | | | | R | R | **P** | | | |
| Юзабилити-тестирование | | | | | | **P** | R | | |
| Dashboard, Settings UI | | | | | **P** | R | | | |
| Trading Panel | | | | **P** | | R | | | |
| Deploy scripts, CI/CD | | | | | | | | **P** | |
| Backup, Logs, Health | | R | | | | | | **P** | |
| E2E-тесты (Playwright) | | | | R | R | | **P** | | |
| Ручное тестирование | | | | | | | **P** | | |
| Unit/Integration-тесты | | **P** | **P** | R | R | | R | | **P** |
| Security audit | | | | | | | R | | **P** |
| Performance testing | | **P** | | R | | | R | | |
| Документация | | R | R | | | | | **P** | |

**P** = Primary (основной исполнитель), **R** = Reviewer (ревью / консультация)

---

## 11. SPRINT 7 — РЕАЛИЗАЦИЯ SHOULD-ФИЧ (Phase 1 feature-complete) — ✅ S7 (2026-04-26)

> Раздел добавлен в v1.4 по итогам Sprint_7_ARCH (7.R). Описывает технические артефакты Sprint 7 (17 задач). Файлы расположены в `Develop/backend/app/` и `Develop/frontend/src/`.

### 11.1 Модель `strategy_versions` — расширение

```sql
-- Alembic миграция c1d4e5f6a7b8_add_strategy_versions_meta
ALTER TABLE strategy_versions ADD COLUMN created_by INTEGER NULL REFERENCES users(id);
ALTER TABLE strategy_versions ADD COLUMN comment TEXT NULL;
CREATE INDEX idx_sv_history ON strategy_versions (strategy_id, version_number DESC);
```

Backfill: `created_by = strategies.user_id` (subquery в данных-миграции). `comment` остаётся NULL для старых записей.

### 11.2 Endpoints версионирования (контракт C4)

| Метод | URL | Описание |
|-------|-----|----------|
| GET | `/api/v1/strategy/{id}/versions/list` | Компактный список (без `blocks_json`) |
| GET | `/api/v1/strategy/{id}/versions/by-id/{version_id}` | Полная версия по id |
| POST | `/api/v1/strategy/{id}/versions/snapshot` | Создание именованной версии (`{comment}`) |
| POST | `/api/v1/strategy/{id}/versions/{version_id}/restore` | History-preserving restore |
| GET | `/api/v1/strategy/{id}/versions/{version_number}` | Legacy: версия по version_number |

⚠️ Порядок маршрутов критичен (gotcha-20): статические сегменты (`list`, `snapshot`, `by-id`, `restore`) объявлены **выше** `/{version_number}`, иначе FastAPI пытается распарсить `list` как int → 422.

### 11.3 Модель `backtest_jobs`

```sql
CREATE TABLE backtest_jobs (
  id TEXT PRIMARY KEY,                    -- uuid hex
  user_id INTEGER NOT NULL REFERENCES users(id),
  strategy_id INTEGER NOT NULL REFERENCES strategies(id),
  strategy_version_id INTEGER NULL REFERENCES strategy_versions(id),
  job_type TEXT NOT NULL,                 -- 'single' | 'grid'
  status TEXT NOT NULL,                   -- 'queued' | 'running' | 'done' | 'error' | 'cancelled'
  progress INTEGER NOT NULL DEFAULT 0,
  params_json TEXT NOT NULL,
  result_json TEXT NULL,
  error_message TEXT NULL,
  created_at DATETIME NOT NULL DEFAULT (datetime('now')),
  updated_at DATETIME NOT NULL DEFAULT (datetime('now')),
  finished_at DATETIME NULL
);
CREATE INDEX idx_bj_user_active ON backtest_jobs (user_id, status);
```

### 11.4 BacktestJobManager (singleton)

`app/backtest/jobs.py`. Хранит `dict[job_id, asyncio.Task]` в памяти + persist в `backtest_jobs`. API: `submit(user_id, runner, job_type, params)` / `get(job_id)` / `list_user(user_id, status?)` / `cancel(job_id)` / `shutdown()`. Per-user cap=3, защищён `asyncio.Lock`. Создаётся в `lifespan` (`app/main.py:91`), сохраняется в `app.state.backtest_job_manager`. Cleanup на shutdown через `asyncio.shield` для финального `_update_status`.

**Обычный бэктест — job `single` (S8R-FIX-127, решение заказчика 2026-10-14).** `POST /backtest` и `POST /backtest/{id}/rerun` ставят расчёт через менеджер (`_submit_single_backtest_job`, как `POST /backtest/run-async`), а не через `BackgroundTasks`: ответ 202 `{backtest_id, status:"running", job_id}` (поле `job_id` добавлено, остальное без изменений). Отмена — `POST /backtest/jobs/{job_id}/cancel` (маршрута `POST /backtest/{id}/cancel`, на который слал фронт, не было — «Не удалось отменить бэктест»): `CancelledError` прерывает и паузу `RateLimitWait`; бэктест → `cancelled`, событие `backtest.cancelled`. Фронт (`backtestStore.jobIds`, `BacktestProgress`) отменяет через `backtestApi.cancelJob` и показывает фактический итог (WS-событие / перечитывание бэктеста), а не локальный «Отменён»; `backtestApi.cancel` удалён. Cap=3 общий для `single`/`background`/`grid`; отказ — `BacktestJobLimitError` → 422 `{detail:"Превышен лимит параллельных бэктестов (макс. 3)", error_code:"backtest_job_limit"}`, предпроверка лимита — до создания строки `Backtest`. Ревью ветки (DEV-127R): после возврата раннера job попадает в `_finishing` — `cancel()` → False, эндпоинт `not_active`; запись итога (`done/error/cancelled`) и постановка `submit`/`_enqueue`, уборка строки бэктеста — через `run_through_cancels` (`anyio.CancelScope(shield=True)`); `_enforce_cap` — самовосстановление: строки `queued/running` без задачи процесса → job `error` «Задача бэктеста потеряна — запустите заново», бэктест → `failed`, в лимит не идут; `_mark_running` — best-effort, `settle_unfinished_backtest` переводит бэктест `single`, чей job кончился без раннера; `check_cap` до `db.add` в `/backtest`, `/rerun`, `/run-async` (страховка — в `submit`); `shutdown()` → `interrupted_reason`: идущий `single` → `failed` «Прервано: перезапуск сервера или сбой записи статуса под нагрузкой БД» (job — `cancelled`, как прежде); `DELETE /backtest/{id}` сначала `cancel_for_backtest`. Фронт: `cancelBacktest` — «Отмена…», перечитывание бэктеста до 5 с (шаг 0,5 с): `cancelled` → «Отменён», `completed` → результат и тост «Бэктест успел завершиться до отмены», `not_active` при `running` → тост «Расчёт этого бэктеста на сервере не найден…». S8R-FIX-128 (2026-10-15): «Запустить в фоне» своего счёта лимита по карточкам бейджа не ведёт (обычные бэктесты в бейдж не попадают, текст «…фоновых» расходился с сервером) — отказ только от сервера, тост `notifyJobLimit` с его текстом; `StrategyBacktestsTable` — `STATUS_BADGES` (значок + полное название в подсказке и `aria-label`): `cancelled` — «×» «Отменён», `queued`/`pending` — «...», неизвестный — «?» с кодом статуса. **Провал задачи доходит до job'а значением, а не через БД (2026-10-13).** `_run_backtest_task` возвращает `str | None` — текст ошибки; runner (`_make_single_backtest_runner`) по нему и принимает решение `job='error'`. Раньше решение принималось перечитыванием `backtests.status`, а пометка `failed` идёт через `persist_with_retry` с веткой «последний рубеж» на свежей сессии и веткой «подберёт реконсиляция» — в зазоре между пометкой и чтением провал превращался в `done` с ложным уведомлением «Бэктест завершён» (BUG-31 закрывал это по следствию). Перечитывание осталось источником метрик для `result_json` и запасной проверкой; фоновый путь `create_backtest` (`BackgroundTasks`) возвращаемое значение игнорирует. **`shutdown()` под потолком (S8R-FIX-009):** общий бюджет `SHUTDOWN_WAIT_SEC = 10.0` (меньше дедлайна торгового рантайма 45 с и `stop_grace_period` 60 с Docker). Раньше ожидание было без предела: отмена корутины не прерывает расчёт в потоке, а обработчик отмены дописывает результат в БД под write-локом SQLite — до `busy_timeout` (30 с), и контейнер успевал получить SIGKILL. Оставшиеся задачи — одним WARNING; исключения завершившихся забираются (иначе «Task exception was never retrieved» уже после остановки). **Одно уведомление на job (S8R-FIX-114).** `RUNNER_NOTIFIES_JOB_TYPES = {"single"}`: одиночный бэктест (`run-async`) уведомляет сам `_run_backtest_task` — тот же код, что у синхронного `POST /backtest` («SBER · 5m · P&L −4.26% · 18 сделок · Sharpe −0.25», `related_entity_type="backtest"`); менеджер для этих типов `_notify_completed` не зовёт. Раньше менеджер слал второе `backtest_completed` из прочерков («— (SBER, 5m): P&L —, Sharpe —») — в `params` job нет `strategy_name`, в `result` нет `metrics`. Grid Search уведомляет менеджер (`_completed_body`): заголовок «Grid Search завершён», тело «Grid Search · <тикер> · <ТФ> · комбинаций: N/M[ · с ошибкой: K][ · лучший Sharpe X, P&L ±Y%]» по первой строке `matrix` с числовым Sharpe (`matrix` отсортирована по убыванию Sharpe), `related_entity_type="backtest_job"`.

### 11.5 GridSearchEngine (multiprocessing.Pool)

`app/backtest/grid.py`. ТЗ §5.3.4 соблюдён: `multiprocessing.Pool` (spawn-context, workers = `os.cpu_count() - 1`). Worker `_run_single_backtest(payload: dict) -> dict` — module-level pickle-friendly функция; импортирует `backtrader`/`pandas` сам, восстанавливает DataFrame из `list[dict]` свечей. OHLCV грузится один раз в main-процессе (`router.py::_prepare_grid_workload`) и передаётся как pickle-friendly base_payload. Pool управляется внутри `loop.run_in_executor(None, _drive_pool_sync)` — event loop не блокируется.

Hard cap: ≤5 параметров, ≤1000 комбинаций (raise 422 ДО Pool.spawn). Overfitting flag: если `(sharpe[0] - sharpe[4]) / sharpe[0] > 0.5` → `overfitting_warning: true`. См. также gotcha-21 (UPDATE-секция).

**Ожидание worker-слотов (S8R-FIX-009).** Слоты `GridWorkerSlots` общие на процесс (S8R-AUDIT-078). Пока их держит сосед, job остаётся в статусе `queued` и получает событие `waiting` с текстом причины (`SLOT_POLL_SEC = 1.0` между проверками); раньше он переходил в `running` и стоял на 0 % — на экране неотличимо от зависания. Отмена в этом состоянии работает штатно.

### 11.6 WS endpoints

#### `/ws/trading-sessions/{user_id}` (контракт C1)

`app/trading/ws_sessions.py`. Авторизация — cookie при upgrade (S8R-FIX-017: прежнее «auth первым сообщением» устарело; JWT не в URL — gotcha-16). После `auth_ok` backend:
1. Шлёт `{event:"snapshot", sessions: [...]}`. Снапшот — одна сессия БД и фиксированное число запросов (два агрегата по `IN`, связи сессии не грузятся; S8R-FIX-044); при сбое сводки — сессии с `positions`/`last_trade_at` = null.
2. Подписывается на `trades:{session_id}` для каждой активной сессии + `system:{user_id}` (для новых сессий).
3. Форвардит deltas: `position_update`, `trade_filled`, `pnl_update`, `session_state`.
   - **`status` в `session_state` (S8R-FIX-097).** `session.paused` / `session.resumed` / `session.stopped` форвардятся как `session_state`, фронт вливает payload в карточку — поэтому каждый издатель кладёт `status`: `TradingSessionManager.pause_session` / `resume_session` / `stop_session` (`session.status`), `SessionRuntime._pause_unrestorable` (`target_status`: `paused` или `stopped`), пауза сверкой `_flag_position_mismatch` (S8R-FIX-124), CB — как и раньше. Без поля пауза «из другого места» не меняла карточку на `/trading` до перезагрузки. Публикация остановки рантайма при shutdown не тронута (статус там `suspended`).
   - **Причина паузы и лимиты сессии (A-04, ТЗ v4.7).** Строка снапшота (`_session_row`) несёт `pause_reason` и `has_session_limits` (одним запросом `circuit_breaker_configs.session_id IN (…)`; сбой сводки — `null`). `session.paused` / `session.resumed` — с `pause_reason` (CB и «Стоп всему» — код, ручные — `null`); новое `session.limits_changed` (`{session_id, has_session_limits}`) форвардится как `session_state`.

Cleanup: try/finally — отписка от всех каналов при WebSocketDisconnect.

#### `/ws/backtest/{job_id}` (контракт C2)

`app/backtest/ws_backtest.py`. Тот же auth-pattern. Сообщения: `queued` / `started` / `progress` / `done` / `error` / `cancelled`. Активен пока `job.status in ('queued','running')` + 30 сек grace после терминального события (гарантия доставки финального события).

### 11.7 BackupService

`app/backup/service.py`. WAL-aware SQLite copy: `BEGIN IMMEDIATE` → `PRAGMA wal_checkpoint(FULL)` → `copy2(.db)` + копирование `*.db-wal` / `*.db-shm` (gotcha-19). Postgres ветка: `pg_dump --no-owner --format=custom`. Ротация: keep_last (default 7). Restore: validate → atomic rename → `alembic upgrade head`. APScheduler-job `backup_db_daily` в `app/scheduler/service.py:362` (cron из `BACKUP_CRON`, default 03:00 UTC). CLI: `python -m app.cli.backup {create,list,restore,rotate}` (argparse, без typer).

### 11.8 ChatRequest.context_items (контракт C3)

`app/ai/chat_schemas.py`:

```python
class ChatContextType(StrEnum):
    chart = "chart"
    backtest = "backtest"
    strategy = "strategy"
    session = "session"
    portfolio = "portfolio"

class ChatContextItem(BaseModel):
    type: ChatContextType
    id: int | str | None = None  # str для tickers (chart), int для остальных, None для portfolio

class ChatRequest(BaseModel):
    strategy_id: int | None = None
    message: str = Field(min_length=1, max_length=4000)
    history: list[ChatMessage] = []
    context: dict | None = None  # legacy (блок-режим стратегии)
    context_items: list[ChatContextItem] = Field(default_factory=list, max_length=5)  # NEW
```

### 11.9 slash_context резолверы

`app/ai/slash_context.py`. 5 резолверов (chart/backtest/strategy/session/portfolio), каждый делает SELECT с ownership-check (`AND user_id = current_user.id`, иначе 404 без leak'ов). Sanitization: удаление control-chars (`[\x00-\x08\x0B\x0C\x0E-\x1F\x7F]`), экранирование подделок маркеров `[CONTEXT`/`[USER`, обрезка до 2 КБ. Финальный prompt оборачивается в `[CONTEXT type=... id=...]\n...\n[/CONTEXT]\n` + `[USER]\n...\n[/USER]` (защита от prompt injection).

### 11.10 Модель `chart_drawings` (mini-CRUD)

Модель `ChartDrawing` уже существовала в `app/common/models.py` (initial-миграция `3d3e4e3036a6_initial_schema.py:144`). В S7 добавлен модуль `app/chart_drawings/` с 5 endpoints под префиксом `/api/v1/charts/`:

| Метод | URL |
|-------|-----|
| GET | ~~`/api/v1/charts/{ticker}/{tf}/drawings`~~ → с v4.3 `/api/v1/charts/{ticker}/drawings` (все ТФ бумаги) |
| POST | ~~`/api/v1/charts/{ticker}/{tf}/drawings`~~ → с v4.3 `/api/v1/charts/{ticker}/drawings` (ТФ — поле тела `timeframe`) |
| PATCH | `/api/v1/charts/drawings/{id}` |
| DELETE | `/api/v1/charts/drawings/{id}` |
| DELETE | ~~`/api/v1/charts/{ticker}/{tf}/drawings` (clear all)~~ → с v4.3 `/api/v1/charts/{ticker}/drawings?timeframe=<tf>` (очистка одного ТФ); с v4.6 — `timeframe` необязателен, без него — все ТФ бумаги |

Frontend `chartDrawingsApi.ts` совпадает 1:1. Все endpoints проверяют ownership. С v4.3 (Б-02, ADR 0026) рисунки хранятся по «пользователь + бумага», индекс `idx_drawing_user_ticker` (§3.14); контракт — §4.13. С v4.6 (Б-04) — новые типы и проверки §4.13, ревизия данных `b6b5064c0be6` (§3.14).

### 11.11 users.wizard_completed_at + endpoint (контракт C6)

```sql
-- Alembic миграция d2e3f4a5b6c7_add_users_wizard_completed_at
ALTER TABLE users ADD COLUMN wizard_completed_at DATETIME NULL;
```

Endpoint `POST /api/v1/users/me/wizard/complete` (модуль `app/users/`, после fix-волны 2026-04-26 — переехал из `/auth/me/...`). `GET /api/v1/users/me` отдаёт `UserResponse` с полем `wizard_completed_at: datetime | null`.

`S8R-AUDIT-043`: `POST /api/v1/users/me/disclaimer/accept` → `{ ok, disclaimer_accepted_at }` — условный UPDATE (если пусто) + аудит `user.disclaimer_accepted` в одной транзакции; `disclaimer_accepted_at` отдаётся в `/auth/me` и `/users/me`. Гейт: `TradingService.start_session` и `resume_session` без принятия → 403 `{ detail, error_code: "disclaimer_required" }` (владелец и отметка — одним JOIN, владелец не найден → 404); автоматический restore — без гейта. Фронт: шаг «Риски» мастера вызывает accept; прошедшим мастер раньше — блокирующая `DisclaimerModal` (текст и подписи — ФТ §18.1, кнопка «Выйти»); 403 `disclaimer_required` → перепроверка профиля → модалка.

### 11.12 Account balance/history (контракт C9)

Модуль `app/account/` (`schemas.py`, `service.py`, `router.py`). Endpoint `GET /api/v1/account/balance/history?days=N` → `[{ts, total_value, trading_pnl, active_capital, currency:'RUB'}]` (`active_capital` — S8R-FIX-073, капитал активных на дату сессий; отдаётся явно, чтобы фронт не восстанавливал состояние «активных сессий нет» вычитанием `total_value − trading_pnl`). Источник — агрегация on-the-fly: `TradingSession.initial_capital` + cumulative `DailyStat.realized_pnl` + текущий `PaperPortfolio.balance` для today. Без snapshot-таблицы (overhead не оправдан для 30-точечного sparkline).

### 11.13 NotificationService DI singleton (контракт C7)

`app/notification/dependencies.py`: `get_notification_service` Depends-helper. Singleton сохраняется в `app.state.notification_service` (lifespan, `app/main.py:75`). Все потребители — через `Depends(get_notification_service)` или `request.app.state.notification_service`. **Грep-инвариант:** `grep -rn "NotificationService()" app/` = 0 вне `main.py` (на 2026-04-26 подтверждено). Listener `listen_broker_status` подключён в lifespan (`app/main.py:81`); `stop_broker_listener` в reverse shutdown.

### 11.14 EVENT_MAP — 5 новых записей (MR.5)

`app/notification/service.py:76-104`:

| EventBus key | event_type | Severity | Шаблон |
|--------------|-----------|----------|--------|
| `trade.opened` | trade_opened | info | "Позиция открыта" |
| `order.partial_fill` | partial_fill | info | "Частичное исполнение" |
| `order.error` | order_error | warning | "Ошибка выставления ордера" |
| `connection.lost` | connection_lost | warning | "Соединение с T-Invest потеряно" |
| `connection.restored` | connection_restored | info | "Соединение с T-Invest восстановлено" |

Тело `connection.lost` — `{broker_name}: {reason}`; `reason` собирает `stream_loss_reason(exc)` (`app/broker/tinvest/errors.py`, S8R-FIX-138): «поток данных прерван, переподключение — автоматически [КОД]», при терминальной ошибке ключа (S8R-AUDIT-061) — человеческий текст `_human_text` и «поток данных остановлен [КОД]». Repr исключения в уведомление не попадает — он в логах `multiplexer_disconnected` / `multiplexer_terminal_error`.

Production publish-сайты:
- `trade.opened`: `app/trading/engine.py:846` (paper) + `:1201` (real)
- `order.partial_fill`: `app/trading/engine.py:1171`
- `order.error`: `app/trading/engine.py:885` (helper `_publish_order_error` + try/except в `process_signal:812`)
- `connection.lost`: `app/broker/tinvest/multiplexer.py:244` (под флагом `_connection_event_published`)
- `connection.restored`: `app/broker/tinvest/multiplexer.py:221` (после первого успешного response)

### 11.15 Telegram CallbackQueryHandler (контракт C8)

`app/notification/telegram_webhook.py`. Регистрация через `_app.add_handler(CallbackQueryHandler(self._handle_callback))` в `__init__`. Поддержка callback'ов:
- `confirm_closeall` / `cancel_closeall` (S6) — оставлены.
- `open_session:{session_id}` — ownership-check через JOIN `Strategy.user_id`. Deep link `{settings.FRONTEND_URL}/trading?session={id}`.
- `open_chart:{TICKER}` — канонический тикер, регистр сохраняется; формат — `TICKER_RE` (латиница, цифры, «.», «-», «_», 1–20, без «..», S8R-REV-FBE). Deep link `{settings.FRONTEND_URL}/chart/{TICKER}`.

`TelegramNotifier.send` принимает `ticker` явно — `_build_keyboard(ticker)` строит `open_chart:{TICKER}`, а не `open_chart:{trade_id}`.

### 11.16 Cross-DEV контракты Sprint 7

| # | Поставщик | Потребитель | Контракт |
|---|-----------|-------------|----------|
| C1 | BACK1 (`ws_sessions.py`) | FRONT1 (`useTradingSessionsWS`) | WS `/ws/trading-sessions/{user_id}` (auth-first JWT, snapshot+delta) |
| C2 | BACK1 (`ws_backtest.py`) | FRONT1 (`useBacktestJobWS`) | WS `/ws/backtest/{job_id}` (auth-first, queued/started/progress/done/error/cancelled) |
| C3 | BACK2 (`chat_schemas.py`) | FRONT1 (`aiApi.ts`) | `ChatRequest.context_items: list[ChatContextItem]` (legacy `context: dict` сохранён) |
| C4 | BACK2 (`strategy/router.py`) | FRONT2 (`VersionsHistoryDrawer`) | 4 endpoint'а версионирования + alembic `c1d4e5f6a7b8` |
| C5 | BACK1 (`backtest/router.py:1140`) | FRONT2 (`GridSearchModal`) | `POST /api/v1/backtest/grid` → `{job_id, total, status}` (через WS C2) |
| C6 | BACK2 (`users/router.py`) | FRONT2 (`FirstRunWizardGate`) | `POST /api/v1/users/me/wizard/complete`; `GET /users/me` → `wizard_completed_at` |
| C7 | BACK2 (`notification/dependencies.py`) | BACK1, OPS | `request.app.state.notification_service` или `Depends(get_notification_service)` |
| C8 | BACK2 (`telegram_webhook.py`) | (deep link) | CallbackData scheme `open_session:{id}`, `open_chart:{TICKER}` |
| C9 | BACK1 (`account/router.py`) | FRONT2 (`BalanceWidget`) | `GET /api/v1/account/balance/history?days=N` |

---

## История изменений

| Версия | Дата | Автор | Описание |
|--------|------|-------|----------|
| 1.0 | 2026-03-23 | ARCH | Первая версия на основе ФТ v2.0 |
| 1.1 | 2026-04-07 | Sprint_4_Review | §5.10: AI providers — добавлены DeepSeek, Gemini, Mistral, Groq, Qwen, OpenRouter. §6.3: Strategy Editor — 2 вкладки, два режима (Blockly + AI), SharedDescriptionPanel, toolbar. §6.5: Backtest Results — 4 вкладки с полным описанием |
| 1.2 | 2026-04-17 | Sprint_6_ARCH | §5.7: NotificationService — полная реализация (service, telegram, email, dispatchers, REST API 8 endpoints, WS bridge). §8.6: Graceful Shutdown — реальная последовательность с pending orders 30s и multi-channel notification + recovery |
| 1.3 | 2026-04-24 | Sprint_6_Review | Доработки реализации S5/S5R/S6 синхронизированы с ФТ v2.3 (карточки сессий, Bond, Tax FIFO, маркеры сделок, sequential mode, Stream Multiplex, Recovery, Graceful Shutdown). |
| 1.4 | 2026-04-26 | Sprint_7_ARCH (7.R) | **Phase 1 feature-complete.** Добавлен §11 «Sprint 7 — реализация Should-фич» с разделами: 11.1 модель `strategy_versions` (расширенная: `created_by`, `comment`, индекс `idx_sv_history`); 11.2 endpoints версионирования + history-preserving restore; 11.3 модель `backtest_jobs` (фоновые бэктесты); 11.4 `BacktestJobManager` (singleton, asyncio.Lock, per-user cap=3); 11.5 `GridSearchEngine` (multiprocessing.Pool spawn-context, hard cap 1000, overfitting flag); 11.6 WS endpoints `/ws/trading-sessions/{user_id}` и `/ws/backtest/{job_id}` (auth-first JWT, snapshot+delta); 11.7 `BackupService` (WAL-aware SQLite + Postgres-ветка, APScheduler `backup_db_daily`, CLI argparse); 11.8 расширение `ChatRequest.context_items: list[ChatContextItem]` (legacy `context: dict` сохранён); 11.9 `app/ai/slash_context.py` — резолверы 5 типов + sanitization + ownership 404; 11.10 модель `chart_drawings` (initial-миграция S0, REST mini-CRUD под `/api/v1/charts/{ticker}/{tf}/drawings`); 11.11 `users.wizard_completed_at` + endpoint `POST /api/v1/users/me/wizard/complete`; 11.12 `app/account/` модуль (endpoint `GET /account/balance/history?days=N` — агрегация on-the-fly без snapshot-таблицы); 11.13 NotificationService DI (`get_notification_service` Depends + `app.state.notification_service` singleton, `grep "NotificationService()" app/` = 0 вне `main.py`); 11.14 EVENT_MAP — добавлены 5 типов (trade.opened, order.partial_fill, order.error, connection.lost, connection.restored); 11.15 Telegram CallbackQueryHandler (`open_session`, `open_chart` deep links + ownership-check); 11.16 Cross-DEV контракты C1–C9 формализованы. |
| 1.3 | 2026-04-24 | Sprint_6_Review | §10 (фазы разработки): S5 помечен ✅ (548 tests + 23 E2E) с S5R/S5R-2 closeout; S6 помечен ✅ (945 tests, 18 Stack Gotchas) с явной разбивкой DEV-1..7; S7 актуализирован — Should-команды Telegram и price alerts отмечены как сделанные в S6, добавлены переносы (inline-кнопки, NS singleton, 5 event_type, WS карточки сессий). || 1.5 | 2026-05-13 | Sprint_8 W3 | §8.4 «Deployment Architecture (Mac mini + Docker)»; §7 — admin role, security headers, ASGI mount auth; §9 — coverage gate 80%, bandit/safety в CI; §4 — performance baseline W2. |
| 1.6 | 2026-07-27 | Code Review P0–P1 + Sprint_8_Review | §7 — auth Model A (3 cookie, CSRF double-submit, WS cookie-auth на upgrade, cross-tab single-flight refresh) и правило «клиент мимо axios сам ставит `X-CSRF-Token`»; §3/§5 — `PaperPortfolioAccountant`, семантика `balance`/`blocked_amount`/`equity`, инвариант `Δequity == trade.pnl`, CB-drawdown на paper; §6 — контракт слоя разметки графика (`requestUpdate` в `attached()`, единая выборка `pickTopHit`); §8 — Alembic уважает `DATABASE_URL`, drain paper-сессий при выкатке; §9 — baseline 2026-07-27 (2186 pytest / 772 vitest / 7-7 auth E2E). Синхронизировано с ФТ v2.8. |
| 1.7 | 2026-07-27 | Sprint_8_Review — доведение (8 замечаний приёмки) | §4 — `GET /notifications/settings` отдаёт канонические 18 типов + производное `email_supported` (единый источник `EMAIL_ALLOWED_EVENTS`, фронт не дублирует); контракт отказов постановки бэктеста (503 + `Retry-After` / 422, тело `{"detail": <русский текст>}`) и запрет показывать сырой текст axios — единая точка разбора `utils/apiError.ts`. §5 — новый `BACKTEST_DATA_TIMEOUT_SEC` (180 с) + перевод job в `error`/`backtests` в `failed` с fallback на свежей сессии; инвариант «нет `queued`-строк без job'а»; локализация 13 предупреждений `block_parser.py`. §6 — тема вынесена в `theme.ts`, контраст dimmed в dark 4.04→7.83 : 1 (AAA), обязательный catch-all во вложенных `<Routes>`, заглушка `/admin/metrics`. §7 — обязательные `autoComplete="new-password"` + `data-lpignore` для полей секретов. §9 — 2213 pytest / 828 vitest, Stack Gotchas 36 → 45. Синхронизировано с ФТ v2.9. |
