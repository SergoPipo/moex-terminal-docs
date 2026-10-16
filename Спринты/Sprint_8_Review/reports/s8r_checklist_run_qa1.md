# Сплошная проверка UI-чеклиста S8 — пакет qa1 (2026-10-13)

**Стенд:** 1 (backend `127.0.0.1:8101`, фронт `localhost:5274`), код `wt-s8r-check` = `develop` `e527a05`, БД — копия.
**Разделы:** S8.1, S8.2, S8.3, S8.5, S8.10, S8.13, S8.14, S8.16, S8.17, S8.18, S8.34, S8.39, S8.44 (78 пунктов).
**Спеки:** `wt-s8r-check/frontend/e2e-check/qa1/*.spec.ts`. **Доказательства:** `s8r-evidence/checkrun/results/qa1/` (скриншоты `S8.x-*.png`, логи `pytest_*.log`, `vitest_*.log`, `s834_*.log`, `s817_*.log`, `s8_10_headers_backend.txt`, `s8_1_cli_grant_admin.log`, `s8_39_clean_run.log`).

## Итог

| Статус | Кол-во |
|---|---|
| ✅ PASS | 65 |
| ❌ FAIL | 3 |
| 🔁 OUTDATED | 8 |
| ⏸ TELEGRAM | 1 |
| ⚠️ BLOCKED | 1 |
| **Итого** | **78** |

S8.16-6 помечен в самом чеклисте «не визуальное (skip)» — проверен тестами и учтён в PASS.

## Таблица

| Раздел | п. | Статус | Доказательство / диагноз |
|---|---|---|---|
| S8.1 | 1 | ✅ PASS | `s8_01_admin` — у admin пункт `sidebar-admin-link` с `svg.tabler-icon-shield`; фильтр `Sidebar.tsx:15`. `S8.1-1-admin-sidebar.png` |
| S8.1 | 2 | ✅ PASS | testuser1 → `/admin` и `/admin/metrics` → редирект на `/` + тост «Доступ ограничен». На dev-стенде тоста ДВА (StrictMode дважды вызывает эффект `ProtectedAdminRoute.tsx`), на prod-сборке — один (`s8_prod_preview`, `S8.1-2-prod-toast.png`) |
| S8.1 | 3 | ✅ PASS | admin: ровно 8 пунктов в указанном порядке; testuser1: 7, без админки (`S8.1-3-user-sidebar.png`) |
| S8.1 | 4 | ✅ PASS | `/admin/ping`: аноним 401, testuser1 403 «Admin privileges required», admin 200; `/trading/sessions?all_users=true` у не-админа 403 |
| S8.1 | 5 | ✅ PASS | Вживую на ЧИСТОЙ БД (временный backend на :8101, `s8_39_clean`): первый пользователь `qa1_first` → `/auth/me` `is_admin: true`. Код `auth/service.py:154-161`, `bootstrap_first_user`; `tests/test_admin/test_first_run_admin.py` (37 passed, `pytest_admin.log`) |
| S8.1 | 6 | ❌ FAIL | CLI работает (копия БД через `sqlite3.backup`: `grant_admin testuser1` → «is now admin», exit 0; повтор — идемпотентно; несуществующий — exit 1; `s8_1_cli_grant_admin.log`), но **в structlog ничего не пишет** — см. Д-3 |
| S8.1 | 7 | ✅ PASS | `/auth/me` → `is_admin: true/false` (bool); тип `User.is_admin?: boolean` в `api/types.ts:8` |
| S8.1 | 8 | ✅ PASS | `admin-landing-page`, H2 «Администрирование», бейдж «admin» зелёный (`rgb(140,233,154)`), ссылка `admin-metrics-link` → `/api/v1/admin/metrics/` (`S8.1-8-admin-landing.png`) |
| S8.2 | 1 | ✅ PASS | `curl` к `BACK/api/v1/admin/metrics/` без токена → 401 «Missing authentication token» |
| S8.2 | 2 | ✅ PASS | testuser1 → 403 и на страницу, и на `_dash-layout` (`AdminAuthASGIMiddleware`) |
| S8.2 | 3 | 🔁 OUTDATED | 200 + Dash рендерится (title «MOEX Terminal — Admin Metrics»), но маркера `_dash-app-content` в Dash 4.4.1 нет — корень `#react-entry-point`, `#_dash-config` (`S8.2-metrics-dash.png`) |
| S8.2 | 4 | 🔁 OUTDATED | Графиков 7, данные ЖИВЫЕ (кольцевой буфер `metrics_store`, S8R-METRICS-DASH-MOCK-ONLY): signal→order полное окно / хвост, оценка стратегии, LCP, Telegram-webhook, Telegram по командам, backtest jobs. Пустые — с подписью «Нет замеров …» |
| S8.2 | 5 | ✅ PASS | layout — функция (`metrics_dash.py serve_layout`); после reload 7 графиков; после отправки LCP график LCP сменился с «Нет замеров» на 24 точки (`S8.14-3-dash-lcp.png`) |
| S8.2 | 6 | ✅ PASS | Тёмная тема: фон контейнера `rgb(26,27,30)`, графики `plotly_dark` (`rgb(17,17,17)`) |
| S8.3 | 1 | ✅ PASS | `App.tsx` — `<ErrorBoundary level="app" testId="app-error-boundary">` вокруг `<Routes>`; сработал вживую (см. S8.3-7: «Что-то пошло не так» + «Перезагрузить страницу») |
| S8.3 | 2 | ✅ PASS | `DashboardPage.tsx:264-279` — 4 обёртки level=widget |
| S8.3 | 3 | ✅ PASS | `page.route` sparkline `{points:[null,null]}` → TypeError в рендере → Alert «Sparkline недоступен» с `IconAlertTriangle`; Balance/Health/Positions живы (`S8.3-3-widget-fallback.png`) |
| S8.3 | 4 | ✅ PASS | `error-boundary-balance|health|positions|sparkline` в коде; fallback sparkline найден по testid |
| S8.3 | 5 | ✅ PASS | «Повторить» (`error-boundary-sparkline-retry`) → виджет перерисован, цена 305 (`S8.3-5-widget-retry-ok.png`) |
| S8.3 | 6 | ✅ PASS | console.error `[ErrorBoundary] render error {message, stack, componentStack, level:'widget'}`; componentStack содержит `SparklineWidget`. Формат строки отличается от пункта (`level` — поле объекта) |
| S8.3 | 7 | ✅ PASS | `ChartPage.tsx:328-346` — boundary `error-boundary-chart` вокруг `CandlestickChart`. Наблюдение Д-5: битые свечи роняют весь app — падает сам `ChartPage.tsx:123`, вне этого boundary |
| S8.5 | 1 | ✅ PASS | Пустой localStorage → запрос `ticker=SBER&hours=24`; `recentInstruments=["GAZP"]` → GAZP |
| S8.5 | 2 | ✅ PASS | Живой ответ `{points[203]: {t,p}, current: 286.54}`; SVG `dashboard-sparkline-svg`, canvas 0 |
| S8.5 | 3 | ✅ PASS | loading (Skeleton), error 503 → «Не удалось получить sparkline» + деталь сервера + «Повторить», empty → «Нет данных за 24 ч» (без графика). «Выберите тикер» — только в vitest (`SparklineWidget.test.tsx`), на дашборде недостижим (fallback `'SBER'`) |
| S8.5 | 4 | ✅ PASS | Живой `/health`: `cb_state, tinvest_connected, scheduler_running, scheduler_jobs` (+ `database, ai_configured, telegram_configured`) |
| S8.5 | 5 | ✅ PASS | Подмена cb_state: ok → green `rgb(64,192,87)`, warn → yellow `rgb(252,196,25)`, triggered → red `rgb(250,82,82)` (`S8.5-5-health-cb-*.png`) |
| S8.5 | 6 | ✅ PASS | Запрос `/account/balance/history?days=30&since_first_activity=true` |
| S8.5 | 7 | 🔁 OUTDATED | Опрос `/health` — 60 с + WS-канал `health` (S7R-HEALTH-WS-MIGRATION): `page.clock` +31 с — 0 запросов, +61 с — 2 (виджет + футер) |
| S8.5 | 8 | ✅ PASS | Balance/Positions/Health: Skeleton → 500 → Alert + «Повторить» → пусто: «Нет данных по балансу», «Нет активных позиций», Health — жёлтые «нет данных» (`S8.5-8-*.png`) |
| S8.5 | 9 | ✅ PASS | 4 testid, при 1280px одна строка (y=123, 4×240px). В коде `cols={{base:1, sm:2, md:3, lg:4}}` (добавлен md:3) |
| S8.10 | 1 | ✅ PASS | `curl` к backend: 200/401/404/422, `/docs`, `/openapi.json` — все 6 заголовков. Preflight OPTIONS (CORS) — без них: по S8R-AUDIT-057 их ставит nginx (`nginx.conf` map `$sec_*`, тесты `test_nginx_conf.py` + security headers — 108 passed). Dash — ослабленный CSP по BUG-9 |
| S8.10 | 2 | ✅ PASS | prod-сборка за `vite preview` с CSP nginx (`script-src 'self'`): 9 экранов (вкл. Blockly, график) — inline-скриптов 0, нарушений script-src 0. Единственное нарушение — connect-src на 307-редирект `/notifications` (артефакт стенда, Д-6) |
| S8.10 | 3 | ✅ PASS | `<iframe>` в коде и на 9 экранах нет; `X-Frame-Options: DENY` + `frame-ancestors 'none'` |
| S8.10 | 4 | ✅ PASS | handshake `/ws` → `101 Switching Protocols` без security-заголовков |
| S8.10 | 5 | 🔁 OUTDATED | Карточка S8R-CLIENT-TEST-FLAKY закрыта (W4, `vi.useFakeTimers`); `client.test.ts` 3 прогона подряд — 6/6 зелёные |
| S8.13 | 1 | ✅ PASS | `api/types.ts:24-46` — `PaginatedResponse<T>`, `isPaginatedResponse` |
| S8.13 | 2 | ✅ PASS | `unwrapPaginated` (`types.ts:56`); `paginated.test.ts` зелёный; legacy-массив вместо `{items}` на `/trading` — 3 карточки, без ошибок |
| S8.13 | 3 | ✅ PASS | Оба backend-эндпоинта `response_model=PaginatedResponse` (`/trading/sessions`, `/trades`) типизированы; `/trading`, карточка сессии, `/admin`, график с маркерами — 0 runtime-ошибок |
| S8.13 | 4 | 🔁 OUTDATED | `data-testid="loading-paginated"` никогда не было в коде (`git log -G` пусто); загрузка — Skeleton в виджетах |
| S8.13 | 5 | ✅ PASS | `items=[]` → «Нет активных сессий» (`empty-state`), сделки → «Нет сделок» (`no-trades`); текст не буквально «Нет данных» |
| S8.13 | 6 | ✅ PASS | Потребители на месте, строки сдвинулись: `tradingStore.ts:124,244,353`, `ActivePositionsWidget.tsx:80`, бывш. `CandlestickChart:805` → `hooks/useTradeMarkers.ts:191`; плюс `SessionDashboard`, `LaunchSessionModal`, `AdminLandingPage` |
| S8.14 | 1 | ✅ PASS | `engine.py:2887` `SignalProcessor.process_candle` (`signal.process`), `adapter.py:1441` `TInvestAdapter.place_order` (`order.place`), `telegram_webhook.py:276` `TelegramWebhookHandler.process_update` (`telegram.handle`) |
| S8.14 | 2 | ✅ PASS | `observability.py:_emit` → `log.debug("timed_event", timed_event=…, duration_ms=…)`; уровень DEBUG (BUG-27), в логе стенда не виден; `test_observability.py` и др. — 41 passed |
| S8.14 | 3 | ⚠️ BLOCKED | Цифр под нагрузкой нет. Цепочка работает: LCP браузер → `POST /observability/lcp` 204 → график Dash. Локально без нагрузки: LCP prod 48–128 мс, dev 564 мс; signal→order — только редкие замеры paper-сессий стенда (~20 мс); Telegram — нет замеров (бот выключен) |
| S8.14 | 4 | ✅ PASS | Выше baseline: pytest 5863 passed / покрытие 92 %; vitest 1376; nightly CI на `e527a05` — 182 passed / 1 skipped. Красный CI backend — см. S8.17-5 |
| S8.16 | 1 | ✅ PASS | = S8.5-5 |
| S8.16 | 2 | ✅ PASS | = S8.5-2 (203 живые точки) |
| S8.16 | 3 | ✅ PASS | = S8.5-6 |
| S8.16 | 4 | ⏸ TELEGRAM | Мастер открыт подменой `/auth/me`, шаг 4: кнопка `wizard-telegram-test-button` (`FirstRunWizard.tsx:435`, есть aria-label) → `POST /notifications/telegram/test` → 503 «Telegram-бот не настроен на сервере» → тост. Доставки нет: бот на копии выключен |
| S8.16 | 5 | ✅ PASS | = S8.13-3/6; потребителей 8 (было 5) |
| S8.16 | 6 | ✅ PASS | «не визуальное (skip)» — проверено тестами `test_multiplexer_singleton.py` (в составе 17 passed, `pytest_s816.log`) |
| S8.16 | 7 | ✅ PASS | `Sidebar.tsx:12,15` + S8.1-1/3 |
| S8.16 | 8 | ✅ PASS | = S8.2-3..6 вживую |
| S8.16 | 9 | 🔁 OUTDATED | `EVENT_TYPE_LABELS` — 21 ключ (добавлены события S8R: `cb_unpriced_trade`, `positions_close_failed` и др.); синхронность с `EVENT_MAP` держит `test_event_sync_publishers.py` (зелёный) |
| S8.17 | 1 | ✅ PASS | `vitest run` (без каталогов e2e) — 176 files / 1376 passed / 0 failed; flaky `client.test.ts` нет |
| S8.17 | 2 | ✅ PASS | `tsc -b` в копии фронта — exit 0 (`s834_tsc.log`) |
| S8.17 | 3 | ✅ PASS | `eslint . --max-warnings 0` — 0/0 (цель W3 достигнута, порог теперь в `package.json`) |
| S8.17 | 4 | ✅ PASS | Nightly CI на `e527a05` (run 38415340556): 182 passed, 1 skipped. Локально не гонялся: `playwright.config.ts` смотрит на :5173/:8000 (рабочий стенд — запрещено) |
| S8.17 | 5 | ❌ FAIL | CI `develop` `e527a05` (run 38415107236): **3 failed** `test_start_scripts.py::TestStartSh` — Д-1. Локально 5863 passed / 1 failed (`test_backup_cli` — занят :8000 рабочим стендом, Д-2), покрытие 92 % |
| S8.17 | 6 | ✅ PASS | `bandit -r app/ -ll -c .bandit` — exit 0; CI security-scan (Bandit + Safety) зелёный на `e527a05` |
| S8.17 | 7 | ✅ PASS | Контраст на карточке `rgb(46,46,46)`: зелёный `+10.00%` 7.78, красный 5.86, подписи Health 6.85. Жёлтый в виджетах только у индикатора, не у текста |
| S8.17 | 8 | 🔁 OUTDATED | У кнопки теста Telegram `aria-label` уже есть (`FirstRunWizard.tsx:436`). NavLink с IconShield в развёрнутом сайдбаре имеет имя «Администрирование»; в свёрнутом — нет (Д-4) |
| S8.17 | 9 | ✅ PASS | «Меню» → сайдбар 240→60 px; hover → tooltip «Администрирование» справа (x=56 > 49) (`S8.17-9-sidebar-collapsed-tooltip.png`) |
| S8.18 | 1 | ✅ PASS | 2 активные paper-сессии → «Активных сессий: 2» на Дашборде, Графиках, Настройках, Торговле |
| S8.18 | 2 | ❌ FAIL | Через UI на `/trading` всё работает: стоп последней → «Нет активных сессий» за 230 мс; пауза через UI исключается из счётчика. Но **список на `/trading` по WS статусы не получает**: пауза/стоп «из другого места» → кадр `{"event":"session_state","session_id":7,"payload":{"session_id":7}}` без `status`; карточка остаётся «АКТИВНА» и через 60 с, футер догоняет только опросом за 60 с — Д-7 (`S8.18-2a-external-pause-card-stale.png`) |
| S8.18 | 3 | ✅ PASS | Одинаково на 4 страницах. Сценарий ревью: `/trading` → Дашборд → стоп #7 через API → подпись обновилась через 60 с (минутный опрос) |
| S8.18 | 4 | ✅ PASS | После загрузки `/trading` все `/api` обрываются, `page.clock` +2 мин: футер «Активных сессий: 1» сохранён, в `main` 0 Alert, карточки на месте (`S8.18-4-backend-down.png`) |
| S8.18 | 5 | 🔁 OUTDATED | Бейдж показывает «Online» (CSS: ONLINE), «ПОДКЛЮЧЕНО» в коде не было. Поведение верно: после reload в dev бейдж держится 6 с+, живы по одному `/ws` и `/ws/trading-sessions/1` на загрузку |
| S8.34 | 1 | ✅ PASS | `tsc -b` exit 0 (копия фронта в scratchpad с отдельным tsbuildinfo, чтобы не писать в `Develop/frontend/node_modules/.tmp`) |
| S8.34 | 2 | ✅ PASS | `vite build --outDir results/qa1/dist` — exit 0, `dist/index.html` + assets |
| S8.34 | 3 | ✅ PASS | `ci.yml:171,173` — «Type check (tsc -b)», «Production build (как в Dockerfile)»; оба ✓ и в PR #62 (run 38414127489), и на `develop` (run 38415107236) |
| S8.34 | 4 | ✅ PASS | В копию `HealthWidget.tsx` добавлено `const x: number = "…"` → `tsc -b` exit 2, TS2322; после отката — exit 0 (`s834_tsc_injected_error.log`) |
| S8.39 | 1 | ✅ PASS | Временный backend :8101 на пустой БД (alembic head): `/login` → `/setup` → «Создать аккаунт» → 201 → `/` с мастером (`S8.39-1-*.png`); стенд возвращён (200/200) |
| S8.39 | 2 | ✅ PASS | `/setup` → `/login`; на `/setup` виден только Loader, поля «Подтвердите пароль» не появлялось (MutationObserver) |
| S8.39 | 3 | ✅ PASS | На `/login` нет текста «Первый запуск / Создать аккаунт» и `a[href="/setup"]` |
| S8.39 | 4 | ✅ PASS | `POST /auth/setup` → 403 «Регистрация закрыта: пользователь уже создан…»; 12 открытий `/login` → 0 ответов 429, вход формой → 200 |
| S8.44 | 1 | ✅ PASS | Живьём «База данных — OK» (green); при 503 `degraded`, `database: disconnected` → «НЕДОСТУПНА» (red) (`S8.44-1-*.png`) |
| S8.44 | 2 | ✅ PASS | Остановить планировщик на стенде без правки кода нельзя. Цепочка проверена по частям: backend `test_health.py` (scheduler_stopped → 503 `degraded`, `scheduler_running:false`); фронт e2e (подмена 503 → Scheduler «STOPPED», red, без заглушки «недоступно»); vitest `HealthWidget.test.tsx` «503 degraded (планировщик остановлен)» |

## Дефекты

**Д-1 (❌ S8.17-5, high для CI): `test_start_scripts.py` красный в CI `develop`.**
Где: CI run 38415107236 (`e527a05`), job backend → Unit tests: `TestStartSh::test_check_mode_starts_nothing`, `test_sh_invocation_does_not_break`, `test_dash_invocation_does_not_break`. Получено: `start.sh --check` → exit 1, «FATAL: … CORS_ORIGINS пуст … Задайте реальные секреты». Ожидалось: exit 0.
Корень: `backend/tests/unit/test_start_scripts.py:199,219,239`. Подпроцессу передаётся `env={**os.environ, "MOEX_ENV_FILE": …}`, а job CI задаёт `DEBUG: "false"` (`ci.yml`, env job backend). Переменная окружения сильнее `.env` из `tmp_path`, поэтому `DEBUG=true` из файла не действует, и срабатывает production-гейт. Локально `DEBUG` в окружении нет — тест зелёный. Нужно чистить в env подпроцесса `DEBUG`/`CORS_ORIGINS`/секреты (или явно ставить их). PR #62 смержен при красном CI.

**Д-2 (low, гигиена тестов): `tests/integration/test_backup_cli.py::test_backup_create_then_restore_recovers_data` зависит от свободного :8000.**
Локально падает: «a server is listening on 127.0.0.1:8000 — stop the backend before restore». Порт занят рабочим стендом. Тест не герметичен: нужно передавать `--server-stopped` или свободный порт. В CI зелёный.

**Д-3 (❌ S8.1-6, low): `grant_admin`/`revoke_admin` не пишут в structlog.**
`backend/app/cli/users.py:_grant_admin`/`_revoke_admin` (≈стр. 95–175) только печатают в `print()`, structlog-события нет (так с `9f1c741`). Выдача прав администратора не оставляет следа аудита. Варианты: добавить `structlog` (`admin_granted`/`admin_revoked`, username, кто, когда) или переформулировать пункт.

**Д-4 (наблюдение, a11y low, вне формулировки S8.17-8): в свёрнутом сайдбаре у пунктов нет доступного имени.**
`components/layout/Sidebar.tsx:22` — `label={collapsed ? undefined : item.label}`, `aria-label` нет. Tooltip Mantine имени не даёт: ARIA-снимок в свёрнутом виде — только `img`. Нужен `aria-label={item.label}` на `NavLink`.

**Д-5 (наблюдение, low): ErrorBoundary графика не покрывает сам `ChartPage`.**
Ответ `/market-data/candles` с `candles:[null,…]` роняет приложение в app-fallback: `TypeError … 'close'` в `pages/ChartPage.tsx:123` (`currentPrice = filteredCandles[...].close`), вне `error-boundary-chart`. Сценарий искусственный (backend такого не отдаёт); при желании — защитить вычисление `currentPrice` или поднять boundary выше.

**Д-6 (наблюдение, low): `GET /api/v1/notifications` без слэша → 307.**
`frontend/src/api/notificationApi.ts:19,24` зовёт `/notifications`, FastAPI отвечает `307` на `/notifications/`. Каждая загрузка — лишний круг. `Location` абсолютный по `Host`: за nginx (`Host $host`, без порта) ведёт на тот же origin. Но при публикации не на 80/443 или за прокси с `changeOrigin` редирект уходит на другой origin и упирается в CSP `connect-src`. Так случилось в моей prod-проверке за `vite preview`. Лечится слэшем во фронте.

**Д-7 (❌ S8.18-2, medium): WS-кадр `session_state` для паузы/возобновления/остановки не несёт `status`.**
Шаги: открыть `/trading` с активной paper-сессией; поставить её на паузу «из другого места» (API / другая вкладка / Telegram `/stop`). Ожидалось: карточка «ПАУЗА», футер сразу −1. Получено: кадр `{"event":"session_state","session_id":7,"payload":{"session_id":7}}`, карточка «АКТИВНА» бессрочно (до перезагрузки), футер догоняет минутным опросом.
Корень: `backend/app/trading/engine.py:2590-2594` (`session.paused` → `{"session_id"}`), `:2673-2677` (`session.resumed`), `:2853-2861` (`session.stopped` — без `status`). `ws_sessions.py:395-408` форвардит их как `session_state`, фронт `tradingStore.updateSessionFromWS` (`stores/tradingStore.ts:380`) вливает payload, а статуса в нём нет. CB-путь сделан правильно: `circuit_breaker/engine.py:1604-1607` кладёт `"status": "paused"`. Фикс: добавить `status` в payload трёх публикаций (или проставлять в `ws_sessions` по имени события) + тест на кадр.

Прочее, замеченное по ходу (не дефекты пунктов):
- На `/login` при любой ошибке `setup-status`, включая 429, показывается «Backend недоступен. Убедитесь, что сервер запущен на порту 8000.» (`pages/LoginPage.tsx:69-71,117`). Порт захардкожен, в production неверен. 429 в моих прогонах — от плотных тестов (лимит `general` 200/мин).
- Ответ `POST /trading/sessions` содержит `"user_id": null` (сессии #7, #8). Не разбирал — вне моих разделов.

## Предложения по формулировкам OUTDATED

- **S8.2-3:** «С `is_admin=true` — 200 + Dash HTML (`#react-entry-point`, `_dash-config`; title “MOEX Terminal — Admin Metrics”), графики `.dash-graph` отрисованы».
- **S8.2-4:** «Рендерится 7 графиков на ЖИВЫХ данных кольцевого буфера `@timed_event` (signal→order полное окно и хвост, оценка стратегии, Dashboard LCP, Telegram-webhook, Telegram по командам, backtest jobs по таблице). Без замеров — пустой график с подписью “Нет замеров …”; рестарт backend обнуляет буфер».
- **S8.5-7:** «`/health`: WS-канал `health` + страховочный опрос раз в 60 с (S7R-HEALTH-WS-MIGRATION)».
- **S8.10-5:** удалить; либо «`client.test.ts` стабилен (S8R-CLIENT-TEST-FLAKY закрыт в W4)».
- **S8.13-4:** удалить; либо «во время загрузки виджеты показывают Skeleton».
- **S8.16-9:** «`EVENT_TYPE_LABELS` совпадает с `EVENT_MAP` backend (на `e527a05` — 21 ключ); проверяет `test_event_sync_publishers.py`».
- **S8.17-8:** «a11y: пункты сайдбара имеют доступное имя и в развёрнутом, и в свёрнутом виде; у кнопки теста Telegram в мастере `aria-label`». Это требует фикса Д-4.
- **S8.18-5:** «Бейдж WS на `/trading` при живом соединении — «Online» (зелёный); после перезагрузки в dev не уходит в «Переподключение»; на вкладку одно соединение `/ws/trading-sessions/{uid}`».
- Поправить ссылки на строки (не OUTDATED): S8.3-1 `App.tsx:32` → `App.tsx:~83`; S8.3-2 `DashboardPage.tsx:182-197` → `264-279`; S8.3-7 `ChartPage.tsx:293-309` → `328-346`; S8.13-6 — см. таблицу; S8.16-4 `FirstRunWizard.tsx:512` → `435`; S8.3-6 — формат `'[ErrorBoundary] render error', {level, componentStack}`; S8.5-9 — `cols` с `md:3`.

## Что нужно от оркестратора

- **⏸ TELEGRAM (S8.16-4):** доставка теста из мастера. Нужен стенд с `TELEGRAM_BOT_TOKEN` и привязанным чатом: мастер, шаг 4 → «Отправить тестовое сообщение» → сообщение пришло в чат, тост «Сообщение отправлено в Telegram». Без бота проверено: кнопка вызывает `POST /notifications/telegram/test`, ошибка 503 показана тостом.
- **⚠️ BLOCKED (S8.14-3):** p95 signal→order < 500 мс и p95 Telegram < 3000 мс — только под реальной нагрузкой (активные сессии на торгах, бот включён); снимать с `/admin/metrics` прод-стенда. LCP — на прод-стенде за nginx.
- Ордеров и брокерских сессий пакету не требовалось.

## Изменения стенда-копии 1 (для сведения)

- Созданы paper-сессии #7 (GAZP, сейчас остановлена) и #8 (LKOH, активна); исходная paper #2 (LKOH) остановлена через UI (проверка S8.18-2).
- Backend стенда 1 дважды временно заменялся на backend с чистой БД (`scratchpad/clean_qa1.db`) и поднимался заново той же командой, что `checkstand.sh up`. Сейчас `backend 200 frontend 200`, `WebSocket … 403` в логе нет.
- CLI `grant_admin` выполнялся только на отдельной копии БД в scratchpad: testuser1 на стенде по-прежнему не admin.
- Кэш cookie (`.state-*.json`) удалён; временный `vite preview` (:5299) остановлен. В `results/qa1/dist` лежит prod-сборка для S8.34.
