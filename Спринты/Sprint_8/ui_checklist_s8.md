# UI-чеклист Sprint 8 (M4 Production-ready)

> Расширяет `ui_checklist_s7.md` — **все** проверки оттуда продолжают применяться без изменений.
> Применяется при приёмке Sprint 8 (8.R ARCH-ревью) и QA-прогонах перед M4 production rollout.
> База S7 — `Спринты/ui_checklist_s7.md` (193 строки, 7 секций). База S5R — `Спринты/ui_checklist_s5r.md`.
>
> Источник истины S8: `Спринты/Sprint_8/arch_design_s8.md` §6 (UI scope) + §8 (waves) + 5 отчётов W1/W2 в `Sprint_8/reports/` + код в `Develop/frontend/src/` (HEAD `s8/sprint-8` = `5aea186`).

---

## S8.1 Admin role (frontend, контракт C-S8-7)

- [x] Sidebar показывает пункт «Администрирование» (`IconShield`, `data-testid="sidebar-admin-link"`) **только** если `useAuthStore().user.is_admin === true`. Источник: `Develop/frontend/src/components/layout/Sidebar.tsx:15` (`navItems.filter(item => !item.adminOnly || isAdmin)`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Обычный пользователь (`is_admin=false`) при попытке открыть `/admin` через прямой URL получает редирект на `/` + toast «Доступ ограничен» — `ProtectedAdminRoute`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Sidebar для admin содержит ровно 8 пунктов: Стратегии, Графики, Бэктесты, Торговля, Счёт, Уведомления, Настройки, **Администрирование**. Для не-admin — 7 (без админки). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Backend `/api/v1/admin/*` endpoints отдают 403 без `is_admin=true` (DEV-1 W1, `require_admin` dependency). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Bootstrap: первый зарегистрированный пользователь автоматически получает `is_admin=true` (BACK1 W1). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [~] CLI `python -m app.cli.users grant_admin <username>` работает и логирует действие в structlog. (❌ 2026-10-14 `reports/s8r_checklist_run_qa1.md` → S8R-FIX-098, исправлено в ветке `s8r/fix-checklist-run`, перепроверено на коде ветки — PASS (`reports/s8r_checklist_recheck_qa6.md`); ✅ после мержа)
- [x] `/auth/me` response содержит поле `is_admin: bool`. UI-тип `AuthUser.is_admin?: boolean` в `api/types.ts`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] `/admin` landing page рендерится с `data-testid="admin-landing-page"`, заголовок «Администрирование», бейдж «admin» зелёный, ссылка на `/admin/metrics`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)

## S8.2 Plotly Dash `/admin/metrics` (контракт C-S8-8)

- [x] Страница `GET /api/v1/admin/metrics/` без JWT — 401. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Без `is_admin=true` — 403 (через `AdminAuthASGIMiddleware`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] С `is_admin=true` — 200 + Dash HTML (`#react-entry-point`, `_dash-config`; title «MOEX Terminal — Admin Metrics»), графики `.dash-graph` отрисованы. (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa1.md`; прежняя устарела)
- [x] Рендерится 7 графиков на живых данных кольцевого буфера `@timed_event` (signal→order полное окно и хвост, оценка стратегии, Dashboard LCP, Telegram-webhook, Telegram по командам, backtest jobs по таблице). Без замеров — пустой график с подписью «Нет замеров …»; рестарт backend обнуляет буфер. (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa1.md`; прежняя устарела)
- [x] Графики обновляются при reload (Dash callback re-render). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] CSS dark theme применяется (`prefers-color-scheme: dark` — Dash inline style). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)

## S8.3 ErrorBoundary (S7R-FRONTEND-ERROR-BOUNDARY-MISSING)

- [x] Top-level `<ErrorBoundary level="app">` обёртывает `<Routes>` в `App.tsx:32`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Per-widget ErrorBoundary вокруг **каждого** dashboard виджета: Balance / Health / Positions / Sparkline (`DashboardPage.tsx:182-197`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] При throw в одном виджете — рендерится fallback UI (Mantine `Alert` с `IconAlertTriangle`), соседние виджеты остаются рабочими (не белый экран). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] `data-testid="error-boundary-balance|health|positions|sparkline"` присутствуют. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Кнопка «Повторить» внутри fallback — сбрасывает error state и пытается re-render. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] `componentDidCatch` пишет в console: `[ErrorBoundary level=widget] <error>` + componentStack. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Chart-страница: `<ErrorBoundary>` вокруг `<CandlestickChart>` (`ChartPage.tsx:293-309`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)

## S8.4 Strategy status change UI (S7R-STRATEGY-STATUS-CHANGE-UI, контракт C-S8-2 интегрировано)

- [x] В строке стратегии Dashboard'a — Mantine `Menu` с target = Badge (`StrategyStatusMenu.tsx:60`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Badge меняет label + цвет по статусу: Черновик (gray), Протестирована (blue), Paper Trading (yellow), Real Trading (red), Пауза (orange), Архив (gray). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Клик по бейджу раскрывает меню «Изменить статус» с пятью пунктами — все статусы, кроме текущего; недопустимые переходы (`STRATEGY_STATUS_TRANSITIONS`) — `disabled` с пометкой «(недоступно)». (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa2.md`; прежняя устарела)
- [x] Optimistic update: при клике статус меняется в UI до ответа сервера. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [~] При успехе PATCH `/strategy/{id}` (S3 endpoint, body `{status}`) — green toast «Статус обновлён». (❌ 2026-10-14 `reports/s8r_checklist_run_qa2.md` → S8R-FIX-103, исправлено в ветке `s8r/fix-checklist-run`, перепроверено на коде ветки — PASS (`reports/s8r_checklist_recheck_qa6.md`); ✅ после мержа)
- [x] При ошибке — rollback оптимистичного статуса + red toast. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] **S8 W3** Фильтр `Pause` присутствует в SegmentedControl на DashboardPage (`DashboardPage.tsx:171` — S7R-STRATEGY-STATUS-PAUSED-FILTER). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)

## S8.5 Dashboard widgets новые (контракты C-S8-1, C-S8-2, C-S8-3)

- [x] **SparklineWidget** (`data-testid="dashboard-widget-sparkline-24h"`) — 24h SVG sparkline для выбранного тикера. Использует `getRecentInstruments()[0] ?? 'SBER'` как ticker. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Endpoint `GET /market-data/sparkline?ticker=X&hours=24` возвращает `{points: [{t, p}], current}` (C-S8-2). UI рендерит через `MiniSparkline` (inline SVG, не lightweight-charts — обход Gotcha-24). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] States Sparkline: loading (Skeleton), error (Alert + кнопка «Повторить»), empty (пустой график + «Нет данных»), no-ticker placeholder («Выберите тикер»). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] **HealthWidget** расширен полями: `cb_state`, `tinvest_connected`, `scheduler_running`, `scheduler_jobs` (C-S8-1). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Health badge цвет соответствует `cb_state`: ok=green, warn=yellow, triggered=red. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] **BalanceWidget** — sparkline история обрезается с момента первой активности (`accountApi.getBalanceHistory(30, since_first_activity=true)`, C-S8-3). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] `/health` обновляется по WS-каналу `health` + страховочный опрос раз в 60 с (S7R-HEALTH-WS-MIGRATION). (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa1.md`; прежняя устарела)
- [x] States all 3 widgets: loading skeleton, error retry, empty. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] data-testids: `dashboard-widget-sparkline-24h`, `dashboard-widget-health`, `dashboard-widget-balance`, `dashboard-widget-positions` — все 4 widget'а на одной строке `<SimpleGrid cols={{base:1,sm:2,lg:4}}>`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)

## S8.6 Wizard Telegram test button (контракт C-S8-4)

- [x] ~~Блок «Свой бот» (bot_token + chat_id)~~ — снят (S8R-AUDIT-084, 2026-09-30): на шаге 4 блока «Свой бот» нет, полей Bot token / Chat ID нет. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Кнопка «Отправить тестовое сообщение» (`data-testid="wizard-telegram-test-button"`) — в блоке привязанного чата; до привязки недоступна или объясняет «сначала привяжите Telegram». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] При клике — `POST /notifications/telegram/test` **без тела** (S8R-AUDIT-084): сообщение приходит серверным ботом в привязанный чат; при `DEV_MODE` — «DEV_MODE — не отправлено». (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3 (`DEV_MODE=false`): `POST /notifications/telegram/test` без тела → 200 «Сообщение отправлено в привязанный чат», «ℹ️ Тестовое уведомление» пришло — подтвердил заказчик)
- [x] Четвёртый тест за минуту — отказ по лимиту (3/мин). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Toast «✅ Сообщение отправлено» при 200 или «❌ Ошибка: <detail>» при 4xx/5xx. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] При завершении wizard (`handleFinish`) — telegram_enabled автовключается для критичных event_types через `notificationApi.bulkUpdate()` (правило `project_wizard_notifications_save`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)

## S8.7 Event type labels синхронизация UI ↔ EVENT_MAP (контракт C-S8-9)

- [x] `EVENT_TYPE_LABELS` (`NotificationSettingsPage.tsx`) содержит ровно набор `EVENT_MAP` backend (`app/notification/service.py`); таблица на экране = ответ `GET /notifications/settings`. (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa2.md`; прежняя устарела)
- [x] 13 UI типов (S7): `trade_opened`, `trade_closed`, `partial_fill`, `cb_triggered`, `order_error`, `all_positions_closed`, `session_recovered`, `connection_lost`, `connection_restored`, `backtest_completed`, `daily_stats`, `corporate_action`, `price_alert`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] 4 новых backend-типа S8 (W2 / C-S8-9): `session_started`, `session_stopped`, `order_placed`, `trade_filled` — лейблы добавлены, синхронны с `app/notification/event_map.py`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [~] Включить Telegram + Email для типа `trade_filled` → реальная сделка → **in-app + Telegram** по самой сделке. **Email подтверждается на событии из белого списка** `EMAIL_ALLOWED_EVENTS` — например `all_positions_closed`, которое порождается закрытием позиции той же сделкой (`notifications.channels_sent = in_app,telegram,email`). (Telegram-часть ✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: по сделкам `order_placed`/`trade_opened`/`trade_filled`/`trade_closed` — `in_app,telegram`; «Закрыть все» сессии #8 → `all_positions_closed` «Сессия #8: закрыто 1 позиций» — `in_app,telegram`. Email не проверялся: у пользователя email-канал выключен, SMTP на стенде не настраивался)
  ⚠️ Email по событиям сделок (`order_placed`, `trade_opened`, `trade_filled`, `trade_closed`) не рассылается **намеренно**: иначе активная стратегия слала бы письмо на каждую сделку. API отдаёт по ним `email_supported: false`, тумблер Email в UI погашен — это не дефект (`S8R-S87-EMAIL-NOT-FOR-TRADE-EVENTS`, решение заказчика 2026-07-31).
- [~] data-testid `event-type-row-<event_name>` или таблица с `<tr data-event-type=...>` для каждого ивента. (❌ 2026-10-14 `reports/s8r_checklist_run_qa2.md` → S8R-FIX-106, исправлено в ветке `s8r/fix-checklist-run`, перепроверено на коде ветки — PASS (`reports/s8r_checklist_recheck_qa6.md`); ✅ после мержа)
- [x] Дефолт: `in_app_enabled=true`, остальные `false`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)

## S8.8 Drawing tools editing (S7R-DRAWING-EDITING + S7R-DRAWING-INTRADAY-COORDS)

- [x] Drag фигуры (trendline / rect / label) мышью — плавное перемещение, persist в backend + fallback localStorage. Источник: `DrawingsLayer.tsx:485` (pointer handler Phase 4). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Изменение углов через handle (vertex/corner). Cursor меняется при наведении на handle (`cursor: pointer` / `nwse-resize`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [~] Sequential-index координаты на intraday TF (1m/5m/15m/1h/4h) — корректные. Детектор `isSeriesInSequentialMode()` через `series.data()[0].time < 1e6` (`coords.ts`, DEV-3 W1). (❌ 2026-10-14 `reports/s8r_checklist_run_qa2.md` → S8R-FIX-102, исправлено в ветке `s8r/fix-checklist-run`, перепроверено на коде ветки — PASS (`reports/s8r_checklist_recheck_qa6.md`); ✅ после мержа)
- [x] Backspace / Delete удаляет выделенную фигуру (через `DrawingToolbar` keyboard listener). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Esc снимает выделение + переключает tool на cursor. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Контекстное меню (правый клик) — пункт «Удалить» работает. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] `legacy drawings` без `logical` на intraday TF — не рендерятся (`pointToCoord → null`). Сознательный выбор «hide > misposition»; backfill миграция = S8R-DRAWING-LEGACY-BACKFILL в S9. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)

## S8.9 AIChat / StrategyDescription apply (mock blocks_json)

- [x] В редакторе стратегии кнопка «AI-помощник» (панель «Параметры стратегии») переключает AI-режим: слева панель описания (`description-textarea`, `SharedDescriptionPanel`), справа AI-чат. (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa2.md`; прежняя устарела)
- [x] Описание стратегии вводится в общей панели описания (`SharedDescriptionPanel`) — та же панель в обычном и AI-режиме. (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa2.md`; прежняя устарела)
- [x] Кнопка «Применить на схеме» (`apply-to-blocks-btn`) отправляет описание в `POST /strategy/parse-template` (парсер шаблона, без AI) и получает `blocks_json`. (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa2.md`; прежняя устарела)
- [x] При успехе — `onApplyBlocks(blocksJson)` пробрасывается в `BlocklyWorkspace`, блоки рендерятся. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] При warnings — Alert с предупреждениями (например, «не найден тикер X»). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Loading state — `<Loader>` блокирует кнопку. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)

## S8.10 SecurityHeadersMiddleware (CSP / HSTS — поведение UI)

- [x] Все HTTP-ответы backend содержат headers: `Strict-Transport-Security`, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Content-Security-Policy: default-src 'self'; frame-ancestors 'none'`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy` (запрет camera/microphone/geolocation/payment/usb). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Frontend не использует inline-scripts (CSP нарушит). Все `<script>` через bundler chunks. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Frontend не пытается embed-нуть приложение в iframe (X-Frame-Options DENY). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] WebSocket handshake (101) не получает security headers (приемлемо). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] `client.test.ts` стабилен (S8R-CLIENT-TEST-FLAKY закрыт в W4). (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa1.md`; прежняя устарела)

## S8.11 BG-backtest panel (S7 baseline + S8 polish)

- [x] **Bg-backtest badge** в шапке (`data-testid="bg-backtest-badge"`) показывает count активных + завершённых jobs. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Click → Popover со списком (см. S7.17 чеклист). Полностью унаследовано из S7. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] У завершённого Grid Search в списке кнопка «Показать результат» (`bg-backtest-open-{job_id}`) открывает окно `bg-backtest-grid-result-modal` с тепловой картой (`GridSearchHeatmap`). (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa2.md`; прежняя устарела)
- [~] **AUTO-COLLAPSE (S7R-BG-BACKTEST-AUTOCOLLAPSE)** — W3 Поток A: при `running.length === 0 && done.length > 0` panel сворачивается через 5 сек. (~1ч задача). (🔁 2026-10-14: по решению оркестратора (S8.36 п.2) открытый список фоновых бэктестов сам НЕ закрывается при завершении последнего — S8R-FIX-105 в ветке `s8r/fix-checklist-run`, ✅ после мержа; сворачивание карточек — S8.36)
- [x] Cap = 3 параллельных bg-backtests (`MAX_CONCURRENT_BACKGROUND_BACKTESTS`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)

## S8.12 Backtest analytics tabs (S7R-E2E-7.16 + S8R-ANALYTICS-*)

- [x] `/backtests/{id}` — 4 вкладки: Обзор / График / Показатели / Сделки. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] **Equity zones** на ценовом графике (`InstrumentChart.tsx:226`) — `data-testid="equity-curve-zone-{idx}"` DOM overlay поверх canvas, `pointerEvents:none` на overlay, hover на zone — подсветка trade-range. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] **Trade row click → TradeDetailsPanel** (`BacktestTrades.tsx:285`) — `onRowClick` prop пробрасывается из `BacktestResultsPage`; per-row `data-testid="backtest-trade-row-{trade_id}"`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] При клике на сделку — раскрывается `<TradeDetailsPanel>` с entry/exit (time, price, pnl, duration). (✅ 2026-10-14: время, цена, P&L, длительность — `reports/s8r_checklist_run_qa2.md`; формулировка обновлена — причина входа/выхода в этой версии не показывается, решение заказчика 2026-10-14, ФТ §19.9; строка «Причина: —» убрана в ветке `s8r/fix-checklist-run`)
- [x] `data-testid="pnl-histogram"` (гистограмма P&L) и `data-testid="win-loss-donut"` рендерятся на вкладке «Обзор» (наследовано из S7). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Кнопка «Запустить торговлю из бэктеста» — открывает `LaunchSessionModal` с предзаполненными параметрами (ticker/timeframe/initial_capital/strategy_id) — наследовано из S7. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)

## S8.13 API contract / Paginated types (контракт C-S8-5)

- [x] Generic `PaginatedResponse<T> { items: T[]; total: number; ... }` в `api/types.ts`. Type-guard `isPaginatedResponse<T>()`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Хелпер `unwrapPaginated<T>()` — defensively принимает legacy/null/unexpected формат, возвращает `T[]`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Все API-вызовы с `response_model=PaginatedResponse` корректно типизированы (нет runtime crash «cannot read items of undefined»). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Во время загрузки виджеты показывают Skeleton. (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa1.md`; прежняя устарела)
- [x] Empty state «Нет данных» рендерится при `items.length === 0`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Точки потребления `unwrapPaginated()`: `tradingStore.ts:60,162,196`, `ActivePositionsWidget.tsx:82`, `CandlestickChart.tsx:805`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)

## S8.14 Performance / Observability (целевые ТЗ + контракт BACK1 W2)

- [x] `@timed_event` декоратор в `app/common/observability.py` применён на 3 hot path: `SignalProcessor.process_candle` (`signal.process`), `TInvestAdapter.place_order` (`order.place`), `TelegramWebhookHandler.process_update` (`telegram.handle`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] structlog event: `event="timed_event", timed_event=<name>, duration_ms=<float>`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [~] **Целевые метрики** (отображаются в `/admin/metrics` после persistent storage): (⚠️ 2026-10-14: стендом не берётся — `reports/s8r_checklist_run_qa1.md`)
  - p95 signal-to-order < 500ms.
  - p95 Telegram webhook < 3000ms.
  - Dashboard LCP < 2s (chrome devtools manual measurement).
- [x] Финальные метрики W2 baseline: backend pytest 1490 passed @ 80% coverage; frontend vitest 544 passed; playwright nightly 158 passed. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)

## S8.15 6 сквозных юзабилити-сценариев (UX W3)

> Проверены **анализом кода + 11 playwright скриншотов** в `Sprint_8/screenshots/`.
> Backend HTTP не отвечал в момент UX-прогона — реальное e2e отложено в QA W3.

### Сценарий 1: Новый пользователь от регистрации до завершения wizard

- [x] На настроенной системе `/register` и `/setup` уводят на `/login`; формы регистрации нет (S8.39, ФТ §2.1). (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa3.md`; прежняя устарела)
- [x] Пользователь с `wizard_completed_at IS NULL` после входа видит мастер первого запуска (`FirstRunWizardGate`). Новые учётные записи создаёт администратор; мастер `/setup` — только на чистой БД. (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa3.md`; прежняя устарела)
- [x] **Шаг 1-2** Wizard — приветствие + дисклеймер (галочка required). Источник: `FirstRunWizard.tsx`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] **Шаг 3** Брокер — Select T-Invest (disabled), Radio paper/real. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] **Шаг 4** Уведомления — Telegram, Email (in-app всегда checked + disabled), **кнопка «Проверить подключение» новинка S8** (W2 8.D.5.4). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] **Шаг 5** Финиш — `POST /users/me/wizard/complete` → редирект на `/dashboard`, повторный логин не показывает wizard. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)

### Сценарий 2: Стратегия → бэктест → запуск Paper Trading

- [x] Dashboard «Новая стратегия» → `/strategies/new` (Blockly editor). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Save → strategy появляется в таблице на `/dashboard`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] **Strategy status menu (S8 W1)** — клик по Badge → Menu, выбор статуса → optimistic update → toast. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Бэктест → результаты (`/backtests/:id`) → 4 вкладки. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] **Equity zones (S8 W2)** — hover на зоне трейда, **Trade row click → TradeDetailsPanel (S8 W2)**. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Кнопка «Запустить торговлю из бэктеста» → `LaunchSessionModal` с предзаполнением. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)

### Сценарий 3: Live торговля → мониторинг → закрытие

- [x] Строка позиции: тикер, направление, цена входа, текущая, лоты, P&L ₽ и два процента — «% поз.» (к стоимости входа) и «% кап.» (к капиталу сессии). (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa3.md`; прежняя устарела)
- [x] **Sparkline 24h (S8 W2)** на DashboardPage — мини-график выбранного тикера. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] **Health badge extended (S8 W2)** — цвет соответствует `cb_state` (ok/warn/triggered). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [~] Telegram `/status` команда — reply с актуальными позициями (backend BACK2). (2026-10-14, живой сценарий на рабочем стенде, sandbox #3: `/status` (ответ прислал заказчик) — три активные стратегии; по ФТ §9.3 `/status` — статус стратегий, позиции — `/positions` (формулировка пункта устарела; `/positions` показал GAZP 10 шт −10,30 ₽ и SBER 1 шт +0,19 ₽ — цифры верны). Sandbox подписан «Paper» в `/status` и «(real)» в `/positions` — S8R-FIX-117 в ветке `s8r/fix-checklist-run`, ✅ после мержа)
- [x] Кнопка «Закрыть позицию» → ордер выставлен, P&L зафиксирован, маркер на графике. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: «Закрыть» SBER #57 в карточке сессии #9 → ордер sell брокеру, выход 286,13, P&L +0,67 зафиксирован, маркеры Buy 285,46 / Sell на `/chart/SBER?session=9`)

### Сценарий 4: Bg-backtest параллельные jobs

- [x] BacktestLaunchModal «В фоне» 3 раза → badge count=3. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [~] Завершение → декремент badge + toast «Бэктест X завершён». (❌ 2026-10-14 `reports/s8r_checklist_run_qa3.md` → S8R-FIX-114, исправлено в ветке `s8r/fix-checklist-run`, перепроверено на коде ветки — PASS (`reports/s8r_checklist_recheck_qa6.md`); ✅ после мержа)
- [x] Click badge → Popover с прогресс-барами + кнопками «Открыть результат». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Попытка 4-го параллельного → toast «Превышен лимит параллельных бэктестов». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)

### Сценарий 5: Admin role + Plotly Dash /admin/metrics

- [x] Bootstrap или CLI: `users.is_admin = True` для `sergopipo`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] После релогина — Sidebar пункт «Администрирование» виден. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Click → `/admin` (AdminLandingPage) → ссылка на `/admin/metrics`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] `/api/v1/admin/metrics/` — Plotly Dash с 7 графиками по живым замерам (полное окно signal→order с CB, хвост, оценка стратегии, LCP, Telegram webhook, Telegram по командам, backtest jobs); график без замеров — с подписью «Нет замеров». SPA `/admin/metrics` — заглушка со ссылкой. (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa3.md`; прежняя устарела)
- [x] Non-admin user пытается открыть `/admin/metrics` напрямую → 403 (через AdminAuthASGIMiddleware). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)

### Сценарий 6: Drawing tools editing + intraday

- [x] `/chart/SBER` (timeframe день) — нарисовать trendline, drag, изменить угол. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Backspace удаляет выделенное, Cmd+Z (если undo реализован). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [~] Switch на 5m/15m intraday TF — новая линия. Sequential-index координаты корректны. (❌ 2026-10-14 `reports/s8r_checklist_run_qa3.md` → S8R-FIX-102, исправлено в ветке `s8r/fix-checklist-run`, перепроверено на коде ветки — PASS (`reports/s8r_checklist_recheck_qa6.md`); ✅ после мержа)
- [x] Известное ограничение: legacy drawings без `logical` не рендерятся на intraday (backfill = S8R-DRAWING-LEGACY-BACKFILL в S9). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)

## S8.16 Cross-DEV contracts verification (UX как потребитель)

- [x] **C-S8-1 (health extended):** HealthWidget показывает `cb_state` цвет — ✅ через анализ кода (`HealthWidget.tsx`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] **C-S8-2 (sparkline 24h):** SparklineWidget рендерит SVG с реальными точками от `marketDataApi.getSparkline()` — ✅. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] **C-S8-3 (balance history range):** BalanceWidget шлёт `since_first_activity=true`, sparkline обрезается с первой активности — ✅. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] **C-S8-4 (Telegram test):** кнопка работает в Wizard step 4 — ✅ (`wizard-telegram-test-button` в `FirstRunWizard.tsx:512`). (✅ 2026-10-14: кнопка мастера шлёт тот же `POST /notifications/telegram/test`; доставка серверным ботом подтверждена на рабочем стенде (S8.6 п.3), кнопка на шаге 4 — `reports/s8r_checklist_run_qa4.md`)
- [x] **C-S8-5 (paginated types):** `unwrapPaginated()` потребляется в 5 точках, нет runtime-crash — ✅. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] **C-S8-6 (multiplexer singleton):** не визуальное (skip). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] **C-S8-7 (is_admin field):** Sidebar пункт условный — ✅ (`Sidebar.tsx:12,15`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] **C-S8-8 (/admin/metrics):** страница рендерится — ✅ через анализ `metrics_dash.py` + `AdminAuthASGIMiddleware`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] `EVENT_TYPE_LABELS` совпадает ровно с набором `EVENT_MAP` backend; синхронность проверяет `tests/test_notification/test_event_sync_publishers.py`. (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa1.md`; прежняя устарела)

## S8.18 Статус-футер и WS торговых сессий (S8R, 2026-07-31)

Добавлено по итогам находки `S8R-FOOTER-NO-ACTIVE-SESSIONS`. Подпись в футере
была строкой-константой «Нет активных сессий» — она не зависела ни от каких
данных и врала при живой сессии.

- [x] При **активной** торговой сессии футер (`data-testid="trading-mode"`) показывает «Активных сессий: N», где N — число сессий в статусе `active`. Скриншот: `Sprint_8_Review/screenshots/b1_footer_active_sessions.png`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [~] `paused` и `stopped` в счётчик **не** входят: остановили последнюю сессию → подпись сразу вернулась к «Нет активных сессий» (без перезагрузки страницы — список сессий на `/trading` живёт по WS). (❌ 2026-10-14 `reports/s8r_checklist_run_qa1.md` → S8R-FIX-097, исправлено в ветке `s8r/fix-checklist-run`, перепроверено на коде ветки — PASS (`reports/s8r_checklist_recheck_qa6.md`); ✅ после мержа)
- [x] Число берётся из `GET /trading/dashboard` (опрос раз в минуту плюс внеочередной — при изменении состава сессий) и показывается одинаково на всех страницах: Дашборд, Графики, Настройки. ⚠️ Отдельно проверить сценарий из код-ревью: открыть `/trading` с активными сессиями → уйти на Дашборд → остановить сессии из другого места → подпись обязана обновиться (список сессий в store не очищается и вне `/trading` не живёт по WS, поэтому опираться на него нельзя). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Backend недоступен → футер не ломается и **не** подменяет содержимое страницы `/trading` алертом ошибки (опрос идёт мимо `tradingStore.error`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Бейдж WS на `/trading` при живом соединении — «Online» (зелёный); после перезагрузки в dev не уходит в «Переподключение»; на вкладку одно соединение `/ws/trading-sessions/{uid}`. (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa1.md`; прежняя устарела)

---

## S8.19 Дневной отчёт в Telegram — нереализованный P&L (S8R, 2026-08-03)

Добавлено по итогам `S8R-DAILY-STATS-UNREALIZED-NO-LOT`. Отчёт считал
нереализованный P&L как `(цена − вход) × лоты` — **без** множителя штук/лот,
и для инструментов с `lot_size = 10` (SBER, GAZP, VTBR — большинство акций
MOEX) занижал величину ровно в 10 раз.

- [x] Открыта позиция по инструменту с `lot_size = 10` (например SBER). В дневном отчёте (`event_type=daily_stats`, 19:00 МСК или ручной запуск `SchedulerService.send_daily_stats`) значение `unreal` совпадает с «Нереализованный P&L» на карточке сессии `/trading/sessions/{id}` — **до копейки**. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] То же число показывает Circuit Breaker при расчёте дневного лимита. Три источника (отчёт, карточка сессии, CB) обязаны сходиться на одних данных — расхождение и было дефектом. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Позиция, открытая **вчера** и не закрытая сегодня, входит в `unreal` дневного отчёта. Раньше не входила: открытые позиции брались из выборки «сделки за сегодня», и переходящая экспозиция в отчёте не отражалась вовсе. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Сессия без открытых позиций: в строке отчёта `unreal +0 ₽`, строка не пропадает и не ломает итоговую сумму. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Позиция по инструменту, у которого в `OHLCVCache` нет свечей нужного таймфрейма: позиция в сумму **не** входит (выдуманный убыток хуже отсутствующего), отчёт не падает. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] **Сессия без сегодняшних сделок, но с открытой позицией — в отчёте есть** (`S8R-DAILY-STATS-IDLE-SESSION-HIDDEN`, исправлено 2026-08-03): строка вида «0 откр / 0 закр, real PnL +0 ₽, unreal …». Раньше такая сессия не попадала в отчёт вовсе, а если других сессий не было — отчёт не отправлялся, и живая экспозиция выглядела как «ничего не происходит». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Сессия **без** позиций и без сделок строки по-прежнему **не** получает — «нет активности» должно что-то значить. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Сессия, у которой есть и сегодняшняя сделка, и открытая позиция, даёт **одну** строку; итоговая сумма не задваивается. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)

---

## S8.17 Общие проверки за S8

- [x] **vitest:** `pnpm test` → 544 passed / 0 failed (+2 flaky `client.test.ts` pre-existing, baseline `S7` 528 + 16 новых тестов S8 widget'ов = 544). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] **TypeScript:** `pnpm typecheck` (`tsc -b`; `tsc --noEmit` в проекте с references ничего не проверяет — gotcha-60) → 0 errors. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] **Lint:** 0 errors / 9 warnings (baseline после W2; цель W3 Поток A — `--max-warnings 0`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] **Playwright nightly baseline 158:** не падает. Скриншоты для новых компонентов S8 (Sparkline / Health-extended / Wizard-step4 / Admin-landing) — частично собраны UX W3 (см. `Sprint_8/screenshots/`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [~] **Backend pytest:** 1490 passed / 0 failed / coverage 80%+ (gate active в W3). (❌ 2026-10-14 `reports/s8r_checklist_run_qa1.md` → S8R-FIX-096, исправлено в ветке `s8r/fix-checklist-run`, перепроверено на коде ветки — PASS (`reports/s8r_checklist_recheck_qa6.md`); ✅ после мержа)
- [x] **Bandit/Safety:** 0 medium+ findings (см. `security_audit_s8.md` — все medium вынесены в S9-backlog). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] **Dark theme контраст:** новые виджеты (SparklineWidget, HealthWidget extended, BalanceWidget с since_first_activity) — green/red/yellow проходят ≥4.5:1. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [~] **a11y:** `IconShield` в Sidebar имеет implicit label через NavLink (`label={item.label}`); кнопка test Telegram в wizard — без `aria-label` (TODO в S9-backlog). (🔁 2026-10-14: формулировка обновлена — «a11y: пункты сайдбара имеют доступное имя и в развёрнутом, и в свёрнутом виде; у кнопки теста Telegram в мастере `aria-label`»; доступные имена свёрнутого меню и кнопки уведомлений — S8R-FIX-099/094 в ветке `s8r/fix-checklist-run`, ✅ после мержа)
- [x] **Sidebar collapsed 240→60:** Tooltip с label position="right" работает на admin item (`Sidebar.tsx:33`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)

---

## Известные DELTA vs макет / S9-backlog

> Список несовпадений UI с UX-макетами S7 + найденные UX-баги для S9 (severity medium-high заносятся в `Sprint_8_Review/backlog.md` оркестратором по итогам UX W3).

### DELTA с UX-макетами (некритичные)

- **Plotly Dash графики — MOCK данные.** Реальные метрики появятся в S9 после persistent metrics-storage (см. отчёт DEV-4 W2 §6).
- **`/admin/metrics`** — не имеет custom dark theme; Dash default light-on-dark. Допустимая адаптация для production-rollout.
- **AdminLandingPage** — минимальный placeholder (карточка + ссылки на endpoint'ы). UX-макет admin-панели с user management + grant_admin UI = S9 scope.

### UX-баги найдены (UX W3, severity medium / low — кандидаты в S9-backlog)

- **S8R-UX-WIZARD-TG-NO-ARIA** — кнопка `wizard-telegram-test-button` без `aria-label`. Screen reader просто читает «отправить тестовое сообщение» (текст кнопки) — приемлемо, но добавление aria сделает UX полнее. Severity: low. ~10мин.
- **S8R-UX-ADMIN-LANDING-EMPTY** — `/admin` landing — placeholder без полезной информации сверх ссылки на `/admin/metrics`. Не блокер, но S9 должен расширить (текущие метрики snapshot, активные сессии, последние ошибки). Severity: medium-low. ~4ч (S9).
- **S8R-UX-DASH-4COL-OVERFLOW** — Dashboard `<SimpleGrid cols={{base:1,sm:2,lg:4}}>` — на ширине 1024-1280px 4 виджета визуально тесные. Желательно перейти на `lg:3, xl:4`. Severity: low. ~10мин.
- **S8R-UX-DRAWING-LEGACY-BACKFILL** — legacy drawings без `logical` не рендерятся на intraday TF. Sознательный выбор. Backfill миграция = medium severity, ~3ч (DEV-3, S9).
- **S8R-UX-PLOTLY-DARK-THEME** — `/admin/metrics` рендерится в light Dash style (тёмный фон, светлые графики, контраст слабоват). Желательно `template='plotly_dark'`. Severity: low. ~30мин (BACK1, S9).
- **S8R-UX-WIZARD-TG-TEST-DISABLED-HINT** — кнопка test Telegram disabled пока оба поля не заполнены, но нет подсказки почему. Tooltip «Заполните токен и chat_id» — UX-улучшение. Severity: low. ~10мин.

### Архитектурные карточки (medium-high — кандидаты блокеров)

- **S8R-UX-HEALTH-WS-MIGRATION** (medium-high) — REST polling 30s для /health — приемлемо для S8, но для production-rollout WS-подписка через `/ws health` снизит трафик. ~4ч (FRONT2, W3 Поток A).
- **S8R-UX-SPARKLINE-NO-WS** (medium) — SparklineWidget делает один fetch при mount + при смене ticker. Live-обновление через WS отложено. Через WS C-S8-2 streaming OHLCV → SparklineWidget real-time. ~4ч (S9).
- **S8R-UX-MULTICURRENCY-TOGGLE** (medium) — переключение валюты USD/RUB в BalanceWidget — карточка W3 Поток A (опционально, иначе S9). ~6ч.

> **Замечание:** окончательные карточки `S8R-UX-*` оркестратор переносит в `Sprint_8_Review/backlog.md` (или `Sprint_9_Review/backlog.md`) после UX W3. UX-агент не модифицирует backlog напрямую.

---

## Summary checklist coverage

| Категория | Пунктов | Источник |
|-----------|---------|----------|
| S7 базовые (через `ui_checklist_s7.md`) | ~80 | наследовано |
| S5R базовые (через `ui_checklist_s5r.md`) | ~30 | наследовано |
| S8.1 Admin role | 8 | C-S8-7 |
| S8.2 Plotly Dash | 6 | C-S8-8 |
| S8.3 ErrorBoundary | 7 | S7R-ERROR-BOUNDARY |
| S8.4 Strategy status menu | 7 | S7R-STRATEGY-STATUS |
| S8.5 Dashboard widgets | 9 | C-S8-1/2/3 |
| S8.6 Wizard Telegram test | 6 | C-S8-4, S8R-AUDIT-084 |
| S8.7 Event sync 17 типов | 6 | C-S8-9 |
| S8.8 Drawing editing | 7 | S7R-DRAWING-* |
| S8.9 AIChat apply | 6 | S8 mock |
| S8.10 SecurityHeaders | 5 | S8R-SEC-HEADERS |
| S8.11 BG-backtest panel | 5 | S7 + S8.D.7 |
| S8.12 Backtest analytics | 6 | S7R-E2E-7.16 + S8R-ANALYTICS-* |
| S8.13 Paginated types | 6 | C-S8-5 |
| S8.14 Performance | 4 | BACK1 W2 |
| S8.15 6 сценариев | 30 | UX W3 |
| S8.16 9 Cross-DEV contracts | 9 | C-S8-1..9 |
| S8.17 Общие S8 | 9 | baseline |
| S8.18 Статус-футер и WS сессий | 5 | S8R-FOOTER-NO-ACTIVE-SESSIONS |
| S8.19 Дневной отчёт — unrealized + idle-сессии | 8 | S8R-DAILY-STATS-UNREALIZED-NO-LOT + S8R-DAILY-STATS-IDLE-SESSION-HIDDEN |

## S8.20 Комиссия в бумажной торговле (S8R, 2026-08-03)

Добавлено по итогам `S8R-PAPER-NO-COMMISSION`. Paper не удерживал комиссию
вообще, тогда как бэктест той же стратегии её вычитает — результат paper был
систематически оптимистичнее, а сверять его с бэктестом ради этого и нужно.

- [x] Модалка запуска сессии (`LaunchSessionModal`) содержит поле «Комиссия, %» (`data-testid="commission-pct-input"`) со значением по умолчанию **0,05** — тем же, что предлагается для бэктеста. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Поле принимает 0 (удержания нет) и дробные значения до 4 знаков; отрицательное не принимается. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] После запуска paper-сессии с ненулевой ставкой и первой покупки: свободный кэш уменьшился на стоимость позиции **плюс** комиссию, а «в позиции» (`blocked_amount`) — ровно на стоимость позиции. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] В журнале сделок закрытой позиции колонка «Комиссия» заполнена и равна сумме удержаний входа и выхода (раньше колонка всегда пустовала). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] P&L сделки показывается **до** комиссии, а изменение капитала счёта равно P&L минус комиссия обеих сторон — так же считает бэктест. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Сессия, запущенная **до** обновления (ставка 0): комиссия не удерживается, поведение прежнее — ретроактивного пересчёта нет. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)

## S8.21 Метрика полного окна сигнал → ордер (S8R, 2026-08-03)

Добавлено по итогам `S8R-METRIC-SIGNAL-TO-ORDER-BLIND`.

- [x] На `/admin/metrics` **первым** идёт график «Signal → order, полное окно с Circuit Breaker», под ним — прежний «Signal → order latency». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Под нагрузкой значения на первом графике заметно выше, чем на втором: разница и есть ожидание за локом Circuit Breaker, которого прежняя метрика не видела. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] При отсутствии замеров график рисуется пустым с подписью «нет замеров», а не правдоподобными числами. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
## S8.22 Комиссия в sandbox/real и тариф счёта (S8R, 2026-08-04)

Добавлено по итогам `S8R-SANDBOX-REAL-COMMISSION-NOT-RECORDED` (Q2) и
`S8R-PAPER-COMMISSION-BROKER-RATE` (Q5). До этого комиссию писал только
paper-путь: в sandbox/real её удерживает брокер, а терминал об этом не знал —
«чистый P&L» вычитал ноль, и sandbox с бэктестом не сверялся.

- [x] «Настройки → Брокер», редактирование счёта: есть поле «Комиссия брокера, %» (`data-testid="broker-commission-input"`); принимает 0 и дробные значения, отрицательное и >100 не принимает. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Значение сохраняется и после перезагрузки страницы показывается прежним. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Запуск **paper**-сессии без явной ставки берёт тариф счёта; ставка, введённая в модалке запуска, имеет приоритет над тарифом. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Sandbox-сделка: в журнале сделок колонка «Комиссия» заполнена (раньше всегда пустовала). (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: у sandbox-сделок #54–#57 колонка «Комиссия» заполнена (1,06 / 1,06 / 1,06 / 0,28 ₽))
- [~] Если комиссия посчитана по тарифу (песочница брокером её не удерживает), значение показано со знаком **≈**; при наведении — подсказка «Оценка по тарифу счёта, а не удержание брокера». (⏸ 2026-10-14: в живом сценарии песочница вернула факт брокера (`commission_source=broker`, 0,05 % с каждой стороны) — оценки по тарифу не возникло; рендер «≈» с подсказкой проверен на paper-сделке (`reports/s8r_checklist_run_qa*.md`). Оговорка «песочница брокером её не удерживает» устарела: удержание в песочнице есть)
- [x] Комиссия, подтверждённая брокером, показана **без** знака ≈. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: журнал сессии #7 — «1,06 ₽» у #54 и #55 без «≈» (комиссия подтверждена брокером))
- [~] Sandbox-сессия и paper-сессия на одних входных данных дают одинаковый «чистый P&L». (⏸ 2026-10-14: нужно ORDER; без этого проверено — `reports/s8r_checklist_run_qa4.md`)

## S8.23 Комиссия за период в статистике сессии (S8R, 2026-08-04)

Добавлено по решению заказчика (Q6): удержание вычиталось из результата, но
самого числа пользователь не видел — разница между результатом стратегии и
изменением счёта выглядела необъяснённой.

- [x] Вкладка «Статистика» сессии содержит карточку «Комиссия за период» (`data-testid="metric-commission"`) с суммой удержаний. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Если хотя бы одна сделка периода посчитана по тарифу, заголовок карточки — «Комиссия за период (оценка)». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Сумма в карточке равна сумме колонки «Комиссия» журнала сделок за тот же период. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] «Суммарный P&L» = валовой P&L сделок − «Комиссия за период». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)

## S8.24 Режим сделки в отчёте 3-НДФЛ (S8R, 2026-08-04)

Добавлено по итогам `S8R-TAX-SANDBOX-IN-DECLARATION` (Q3=b) и
`S8R-TAX-PNL-LOSES-LOT-SIZE` (Q1).

- [x] Выгрузка 3-НДФЛ (xlsx и csv) содержит колонку «Режим» со значением «песочница» либо «реальный счёт». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Если в отчёте есть sandbox-строки, ниже таблицы напечатано примечание о том, что деньги в песочнице виртуальные и такие строки не подлежат декларированию. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Колонка количества называется «Кол-во, шт» и содержит количество в ШТУКАХ (лоты × размер лота), а не в лотах. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Для инструмента с размером лота 10 финансовый результат строки совпадает с P&L сделки из журнала за вычетом её комиссии. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: 3-НДФЛ 2026 (csv и xlsx) — GAZP (лот 10), «Кол-во, шт» 10; #54: −1,60 − 1,06 = −2,66, #55: 0 − 1,06 = −1,06 — совпадает с P&L журнала за вычетом комиссии)
- [x] Сумма колонок «Комиссия (вход)» и «Комиссия (выход)» строки в точности равна комиссии сделки в журнале (без расхождения в копейку). (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: «Комиссия (вход)» 0,53 + «Комиссия (выход)» 0,53 = 1,06 — ровно комиссия сделки в журнале)

## S8.25 Закрытие по SL/TP доходит до брокера (S8R, 2026-08-05)

Добавлено по итогам `S8R-SLTP-SANDBOX-NO-BROKER-ORDER` (Q1=a). До этого
срабатывание уровня закрывало позицию только в учёте терминала: у брокера она
оставалась открытой, и расходились P&L, число позиций и лимит одновременных
позиций.

- [x] Sandbox-сессия, стоп сработал: в журнале сделок сделка «Закрыта», а на странице «Счёт» позиции по этому тикеру нет (данные берутся у брокера). (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: v102 (SL 0,1 %) на GAZP — стоп сработал, сделка #54 «Закрыта»; позиции у брокера нет (портфель — через API: страница «Счёт» показывает только реальные счета, решение S8R-FIX-077))
- [x] Цена выхода в журнале — фактическая цена исполнения у брокера, а не ровно уровень стопа. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: #54 — уровень стопа 106,0938, выход 106,04 (фактическое исполнение брокера))
- [x] Комиссия закрытой по стопу sandbox-сделки заполнена; при фактическом удержании брокера показана без знака ≈, при расчёте по тарифу — со знаком ≈. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: комиссия #54 заполнена фактом брокера (0,53 + 0,53) и показана без «≈»)
- [x] Комиссия выхода начислена **один раз** (сумма колонки «Комиссия» = вход + выход, не больше). (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: #54 — вход 0,53 + выход 0,53 = 1,06, выход начислен один раз)
- [~] Вне торговых часов MOEX сделка со сработавшим уровнем остаётся открытой (статус не меняется), P&L не выставляется. (⏸ 2026-10-14: нужно ORDER; без этого проверено — `reports/s8r_checklist_run_qa4.md`)
- [~] При отказе брокера приходит уведомление «автозакрытие … не выполнено, позиция остаётся открытой», а сделка в журнале остаётся открытой. (⏸ 2026-10-14: нужно EVENT; без этого проверено — `reports/s8r_checklist_run_qa4.md`)
- [x] Paper-сессия: закрытие по SL/TP работает как раньше, средства возвращаются в бумажный портфель. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)

## S8.26 Повторное закрытие одной сделки (S8R, 2026-08-05)

Добавлено по итогам `S8R-CLOSE-POSITION-NO-LOCK` (Q2=a).

- [x] Двойной клик по «Закрыть» в карточке позиции: закрытие происходит один раз, второй запрос отвечает понятной ошибкой «Невозможно закрыть сделку со статусом closed». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] После двойного клика в журнале ровно одна закрытая сделка, а не две записи и не двойная комиссия. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Paper: баланс портфеля после двойного клика вырос на одну выручку, а не на две. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] «Закрыть все» при нескольких открытых позициях закрывает каждую ровно один раз (число закрытых = числу бывших открытых). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)

## S8.27 Уведомление о закрытии сделки (S8R, 2026-08-05)

Добавлено по итогам `S8R-TELEGRAM-TRADE-CLOSED-GROSS-PNL` (Q3=a).

- [x] Уведомление «Сделка закрыта» (Telegram и центр уведомлений) показывает **чистый** результат — P&L за вычетом комиссии. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] В том же уведомлении комиссия показана отдельной величиной. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Если комиссия рассчитана по тарифу, а не удержана брокером, в тексте есть пометка «(оценка)». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] В тексте уведомления нет незаполненных плейсхолдеров вида `{net_pnl}`. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)

## S8.28 Сверка позиций с брокером и доучёт закрывающего ордера (S8R, 2026-08-05)

Добавлено по итогам `S8R-EXIT-ORDER-NOT-TRACKED` (Q1=a) и
`S8R-SANDBOX-POSITIONS-NOT-RECONCILED` (Q2=a).

- [x] Карточка сессии с расхождением показывает красный бейдж «Расхождение с брокером» рядом со статусом. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] В подсказке бейджа видны **оба** числа — сколько бумаг у брокера и сколько в терминале, — и номер сделки. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [~] Сессия с расхождением своих позиций переведена в «Пауза», а сама сделка в журнале осталась открытой: `exit_price` и P&L пусты, статус не «Закрыта». (❌ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3 (A3): GAZP продан вне терминала — сессия #8 на паузу НЕ встала, три прохода подряд «сверка не выполнена» → S8R-FIX-124 в ветке `s8r/fix-checklist-run`, ✅ после мержа)
- [~] Уведомление о расхождении приходит один раз, а не на каждом перезапуске приложения при том же самом расхождении. (⏸ 2026-10-14: нужно ORDER; без этого проверено — `reports/s8r_checklist_run_qa4.md`)
- [x] Когда позиции сошлись, после следующей сверки бейдж пропадает сам. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: «у брокера 20 шт, у нас 10 шт» по #8 снялась сама на следующей сверке после возврата к 10 шт, без перезапуска)
- [~] Позиция, о которой знает брокер, но не знает терминал, показывается с оговоркой «возможно, куплено вручную» и сессию не останавливает. (⏸ 2026-10-14: нужно ORDER; без этого проверено — `reports/s8r_checklist_run_qa4.md`)
- [~] После перезапуска backend сделка, чей закрывающий ордер исполнился у брокера, доезжает до «Закрыта» с фактической ценой и комиссией (не с ценой уровня стопа). (⏸ 2026-10-14: нужно ORDER; без этого проверено — `reports/s8r_checklist_run_qa4.md`)
- [~] Такая сделка учтена в дневной статистике ровно один раз (число закрытых не задвоилось). (⏸ 2026-10-14: нужно ORDER; без этого проверено — `reports/s8r_checklist_run_qa4.md`)
- [x] Пока закрывающий ордер в полёте, кнопка «Закрыть» отвечает понятным «Ордер на закрытие уже отправлен брокеру», а второй ордер не уходит. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3 (A2): два одновременных «Закрыть» по #55 — к брокеру ушёл один ордер выхода; второй запрос — 422 «статус closed» (первый успел завершиться); текст «Ордер на закрытие уже отправлен брокеру» для запроса в полёте — тестом `test_close_position_concu…`)
- [~] При сбое песочницы пользователь видит «Песочница брокера временно недоступна, попробуйте ещё раз», а не код gRPC. (⏸ 2026-10-14: нужно EVENT; без этого проверено — `reports/s8r_checklist_run_qa4.md`)
- [~] При сбое на отправке ордера сообщение дополнительно предупреждает сверить портфель у брокера — ордер мог уйти. (⏸ 2026-10-14: нужно EVENT; без этого проверено — `reports/s8r_checklist_run_qa4.md`)

## S8.29 Настоящий FIGI и фоновая сверка позиций (S8R, 2026-08-06)

Добавлено по итогам `S8R-SANDBOX-TRADES-FAKE-FIGI` (Q1=a),
`S8R-RECONCILE-LOCK-DIVERGENCE` (Q2=b) и `S8R-RECONCILE-NOT-PERIODIC` (Q3=a).

- [x] Позиция, открытая стратегией в песочнице или на реальном счёте, при сверке **сопоставляется** с портфелем брокера: сессия остаётся активной, бейджа «Расхождение с брокером» нет. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: сделки #54–#57 с настоящим FIGI (`BBG004730RP0`, `BBG004730N88`), сверка сопоставила их с портфелем — бейджа нет, сессии активны)
- [x] Бумажный режим не изменился: paper-сделки открываются и закрываются как раньше, сверка их не касается. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [~] Расхождение обнаруживается **в течение дня**, а не только при запуске приложения: закрыв позицию вручную в приложении брокера, бейдж видно в течение ближайших 15 минут без перезапуска терминала. (❌ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3 (A3): позиция, исчезнувшая у брокера, паузы не дала — S8R-FIX-124 в ветке, ✅ после мержа)
- [x] Разобравшись с расхождением, бейдж исчезает сам после очередной сверки — перезапуск терминала не нужен. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: пометка «20 против 10» снялась на ближайшей сверке без перезапуска)
- [x] Вторая сессия на том же брокерском счёте (по **другому** тикеру) не останавливает первую: обе продолжают торговать. По одному тикеру две сессии на счёте с 2026-08-06 запрещены — см. S8.30. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: #8 (GAZP) и #9 (SBER) на одном счёте работали одновременно, сверка не остановила ни одну)
- [x] Ручная докупка той же бумаги на счёте стратегии не ставит сессию на паузу — расхождение показывается, торговля продолжается. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3 (A4): докупка 10 GAZP вне терминала → «у брокера 20 шт, у нас 10 шт», сессия #8 осталась активной)
- [~] Нехватка бумаг у брокера по-прежнему останавливает сессию: это единственный случай, когда стратегия торгует несуществующей позицией. (❌ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3 (A3): нехватка бумаг сессию не остановила — S8R-FIX-124 в ветке, ✅ после мержа)
- [~] Сессия, возобновлённая после расхождения и снова ушедшая на паузу, сопровождается уведомлением — «Возобновить» не выглядит нерабочей кнопкой. (⏸ 2026-10-14: нужно ORDER; без этого проверено — `reports/s8r_checklist_run_qa4.md`)
- [~] Пока идёт закрытие позиции, сессия не уходит на паузу с сообщением «у брокера позиции нет» — встречный ордер в полёте расхождением не считается. (⏸ 2026-10-14: нужно ORDER; без этого проверено — `reports/s8r_checklist_run_qa4.md`)
- [~] Недоступность брокера во время фоновой сверки не подвешивает приложение: графики продолжают обновляться, уведомления приходят. (⏸ 2026-10-14: нужно EVENT; без этого проверено — `reports/s8r_checklist_run_qa4.md`)

## S8.30 Один инструмент — одна сессия на счёте (S8R, 2026-08-06)

Добавлено по итогам `S8R-DUPLICATE-SESSION-PER-INSTRUMENT` (Q1=a, Q2=a, Q3=a,
Q4=a, Q6=a) и `S8R-RECONCILE-NO-ACCOUNT-DEDUP`.

- [x] Вторую сессию по тому же тикеру на том же брокерском счёте запустить нельзя: кнопка «Запустить» неактивна, а под формой видно объяснение. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Текст отказа называет инструмент, счёт и **номер** мешающей сессии, а не «ошибка 422» и не пустой экран. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [~] Тот же тикер на **другом** счёте запускается свободно — запрет не шире, чем нужно. (⚠️ 2026-10-14: стендом не берётся — `reports/s8r_checklist_run_qa4.md`)
- [x] Другой тикер на том же счёте запускается свободно. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Разные таймфреймы запрет не обходят: пул бумаг у брокера общий независимо от таймфрейма. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Бумажный режим не ограничен: две paper-сессии по одному тикеру запускаются свободно — и на разных таймфреймах, и на одинаковых настройках, в том числе от одной версии стратегии. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] У каждой такой paper-сессии свой портфель: сделки и P&L не смешиваются между ними. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [~] «Возобновить» у сессии на паузе отказывает, если по её инструменту на счёте уже работает другая, — и объясняет, какая именно. (⏸ 2026-10-14: нужно ORDER; без этого проверено — `reports/s8r_checklist_run_qa4.md`)
- [x] Остановленная (`stopped`) сессия запуску новой по тому же инструменту не мешает: история сделок не блокирует торговлю. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [~] После перезапуска терминала пара конфликтующих сессий в истории не ломает восстановление: поднимается торговавшая до остановки (активная, затем приостановленная — раньше стоявшей на паузе); вторая, если победитель поднялся, переводится на паузу с уведомлением «Пара счёт+тикер занята сессией #N», иначе остаётся «приостановлена» и повторяется при следующем запуске (S8R-AUDIT-010). (⏸ 2026-10-14: нужно ORDER; без этого проверено — `reports/s8r_checklist_run_qa4.md`)
- [x] Сверка с брокером не показывает ложного расхождения, когда бумаги счёта числятся за несколькими сессиями: сравнивается портфель счёта с суммой по счёту. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: бумаги счёта за двумя сессиями (#8 GAZP, #9 SBER) — ложного расхождения нет ни у одной)

## S8.31 «Запустить заново» для песочницы и реального счёта (S8R, 2026-08-10)

- [x] Остановленная **sandbox**-сессия перезапускается кнопкой «Запустить заново»: новая сессия создаётся с тем же счётом, тикером, таймфреймом и режимом `sandbox` (раньше — ошибка 422). (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: «Запустить заново» у остановленной sandbox-сессии #8 → новая #11 (GAZP, sandbox, счёт #3, 1m, версия 103, 1 лот, 0,05 %). Сессия, запущенная с подтверждением сверки движков (#7), перезапуск не проходит — S8R-FIX-126)
- [~] Остановленная **real**-сессия перезапускается так же: режим не подменяется на «бумажный». (⏸ 2026-10-14: нужно ORDER; без этого проверено — `reports/s8r_checklist_run_qa4.md`)
- [x] Paper-сессия перезапускается как прежде, ставка комиссии переносится. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] После перезапуска открывается карточка **новой** сессии, а не прежней. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [~] Если по этому инструменту на этом счёте уже работает другая сессия — кнопка «Запустить заново» неактивна, а под шапкой видно, какая именно сессия мешает (номер, тикер, счёт). (❌ 2026-10-14 `reports/s8r_checklist_run_qa4.md` → S8R-FIX-107, исправлено в ветке `s8r/fix-checklist-run`, перепроверено на коде ветки — PASS (`reports/s8r_checklist_recheck_qa6.md`); ✅ после мержа)
- [x] У сессии, созданной до появления обязательного таймфрейма, кнопка неактивна с объяснением «не сохранён таймфрейм» — а не запускается на чужом таймфрейме. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Форма запуска сессии продолжает работать как раньше: paper без счёта, sandbox/real со счётом, конфликт подсвечивается заранее. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)

## S8.32 Удаление брокерского счёта с живыми сессиями (S8R, 2026-08-10)

- [x] Счёт с сессией в статусе «Активна» удалить нельзя: показывается отказ, счёт остаётся в списке. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] То же для сессии на паузе и для приостановленной рестартом сервера. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [~] В тексте отказа перечислены мешающие сессии: номер, тикер и статус каждой, плюс подсказка «остановите их на странице Торговля». (❌ 2026-10-14 `reports/s8r_checklist_run_qa4.md` → S8R-FIX-116, исправлено в ветке `s8r/fix-checklist-run`, перепроверено на коде ветки — PASS (`reports/s8r_checklist_recheck_qa6.md`); ✅ после мержа)
- [x] Модалка удаления остаётся открытой — текст отказа можно дочитать. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Счёт только с остановленными (`stopped`) сессиями удаляется штатно. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] После отказа привязка сессий к счёту не изменилась: сессии продолжают отображаться на своём счёте, торговля не прервана. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Счёт без сессий удаляется как раньше. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)

## S8.33 Курс доллара в виджете баланса (S8R, 2026-08-10)

- [x] Переключатель RUB/USD пересчитывает баланс по курсу ЦБ РФ, а не по «примерно 90». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [~] Сумма в долларах совпадает с ручным расчётом «рубли ÷ курс ЦБ на сегодня» (курс — cbr.ru). (❌ 2026-10-14 `reports/s8r_checklist_run_qa3.md` → S8R-FIX-118, исправлено в ветке `s8r/fix-checklist-run`, перепроверено на коде ветки — PASS (`reports/s8r_checklist_recheck_qa6.md`); ✅ после мержа)
- [x] Выбор валюты запоминается между перезагрузками страницы. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Если ЦБ недоступен, но курс был получен раньше: под суммой видна дата, на которую курс установлен. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Если курса нет вовсе: переключатель валюты **не показывается**, баланс в рублях отображается штатно, виджет не показывает ошибку. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Повторное открытие дашборда в пределах нескольких часов не вызывает новых обращений к ЦБ (курс кэшируется). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)

## S8.34 Сборка production-образа фронтенда (S8R, 2026-08-10)

- [x] `cd Develop/frontend && pnpm typecheck` — 0 ошибок (это `tsc -b`, а не `tsc --noEmit`). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] `pnpm build` проходит и создаёт `dist/` — ровно та команда, которой собирается frontend-образ. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] В CI на PR присутствуют шаги «Type check (tsc -b)» и «Production build» и оба зелёные. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Намеренная ошибка типа в любом `.tsx` роняет `pnpm typecheck` (проверка, что гейт снова живой). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)

## S8.35 Повторный резолв FIGI по нерезолвящемуся тикеру (S8R, 2026-08-10)

- [x] При работающей sandbox-сессии по тикеру, которого у брокера нет, запись `figi_unresolved` в логе появляется **один раз**, а не на каждую свечу. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: sandbox-сессия #10 по ZZZZ (брокер не знает) — `figi_unresolved` один раз, между попытками брокер не опрашивается)
- [x] Спустя 15 минут попытка резолва повторяется: в логе снова `figi_unresolved` — либо `figi_synced`, если инструмент появился. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: повтор резолва через 15 мин 25 с (12:08:45 → 12:24:10 МСК))
- [~] Кратковременная недоступность брокера **не** отключает попытки на 15 минут: после восстановления связи FIGI подхватывается на ближайшем сигнале. (⏸ 2026-10-14: нужно EVENT; без этого проверено — `reports/s8r_checklist_run_qa4.md`)
- [x] Сделки по обычным тикерам по-прежнему получают настоящий FIGI — сверка с брокером расхождений не показывает. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: GAZP/SBER получили настоящий FIGI, сверка без расхождений)

## S8.36 Авто-сворачивание фоновых бэктестов (S7R, 2026-08-10)

- [x] Завершённый фоновый бэктест исчезает из бейджа примерно через 30 секунд после готовности. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Пока список бэктестов открыт, карточка не исчезает; отсчёт идёт после закрытия списка. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Карточка с ошибкой остаётся до тех пор, пока её не убрать вручную (текст причины больше нигде не виден). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Отменённый бэктест сворачивается наравне с успешным. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] После перезагрузки страницы давно завершённые карточки не возвращаются в бейдж. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)

## S8.37 Тариф брокерского счёта для paper-сессии (S8R, 2026-08-10)

- [x] В форме запуска paper-сессии есть переключатель «Ставка вручную» / «Тариф брокерского счёта». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] При выборе тарифа появляется список счетов; для выбранного видна его ставка («Тариф счёта: 0,3 % с каждой стороны сделки»). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Счёт с нулевым тарифом сопровождается предупреждением, что комиссия удерживаться не будет. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Запуск с выбранным источником «тариф счёта», но без счёта, не проходит — поле подсвечено ошибкой. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Запущенная сессия удерживает комиссию по тарифу счёта: сумма видна в колонке «Комиссия» сделки. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Изменение тарифа счёта после запуска **не** меняет ставку уже работающей сессии. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] В режимах sandbox и real переключателя и поля комиссии нет — комиссию считает брокер. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)

## S8.38 Режим B — импорт стратегии из текстового шаблона (S5R, 2026-08-10)

- [x] В шапке редактора стратегии есть кнопка «Режим B» рядом с «Генерировать». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Кнопка открывает окно «Импорт стратегии из шаблона» с двумя кнопками копирования: пустой шаблон и заполненный пример. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Пустой шаблон содержит семь секций (ИНДИКАТОРЫ … ФИЛЬТРЫ) и **не** содержит названия стратегии, инструментов и таймфрейма. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] «Проверить» на скопированном примере показывает список распознанных блоков без ошибок и предупреждений. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] «Применить» переносит блоки в редактор, после чего код помечается устаревшим («Блоки изменены — код устарел»). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] При пустом поле кнопка «Проверить» недоступна, «Применить» — недоступна до успешной проверки. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)

## S8.39 Регистрация закрыта после первого пользователя (S8R-AUDIT-033, 2026-09-24)

- [x] На чистой БД `/login` сам уводит на `/setup`; после создания пользователя открывается приложение. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] На настроенной системе прямой переход на `/setup` уводит на `/login` (кратко виден индикатор загрузки, формы регистрации нет). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] На странице входа нет ссылки «Первый запуск? Создать аккаунт». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] Повторный `POST /api/v1/auth/setup` на настроенной системе → 403 «Регистрация закрыта…»; частые открытия `/login` (опрос `setup-status`) не приводят к 429 при входе. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)

## S8.40 Удаление стратегии, расхождение позиций, пределы риск-параметров (S8R-AUDIT-069/100, 025, 080, 2026-09-25)

- [x] Удаление стратегии с работающей/приостановленной сессией: красное уведомление с текстом отказа и перечнем сессий (номер, тикер, статус), модалка удаления остаётся открытой, стратегия не удалена. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Удаление стратегии без живых сессий проходит как раньше; повторное удаление той же стратегии (двойной клик) не даёт ошибки сервера. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] «Настройки уведомлений»: есть строка «Расхождение позиций с брокером»; каналы включаются и сохраняются. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Конструктор: блок «Стоп-лосс» в режиме «%» не даёт ввести больше 50, в режиме «пункты» — до 100; сохранение стратегии со стоп-лоссом вне диапазона (через API) даёт понятный отказ «какой блок, какое поле» (S8R-AUDIT-080). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)

- [x] Удаление стратегии с остановленными сессиями (S8R-FIX-026): модалка предупреждает об архиве; уведомление «Стратегия перенесена в архив»; стратегия исчезает из «Все» (счётчик уменьшается), видна в «Архив», в окне запуска сессии не предлагается; повторное удаление — отказ «уже в архиве». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)

## S8.41 Short-стратегия в торговле (S8R-AUDIT-032, 2026-09-25)

- [x] Paper-сессия стратегии с направлением «Short»: первая сделка в таблице сделок — «Short», вход по сигналу входа, выход по сигналу выхода; P&L сделки = (вход − выход) × штуки — **до** комиссии, как у long (S8.20 п.5); изменение капитала = P&L − комиссия обеих сторон. (✅ 2026-10-14: формулировка обновлена по факту — `reports/s8r_checklist_run_qa3.md`; прежняя устарела)
- [x] После закрытия прибыльного short бумажный баланс (капитал) вырос на P&L сделки, после убыточного — уменьшился; «Заблокировано» вернулось к 0. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)
- [x] Long-стратегия: условие выхода срабатывает при отсутствии позиции → ордера и сделки нет, сессия не встаёт на паузу, уведомлений о Circuit Breaker нет. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa3.md`)

## S8.42 Статус стратегии и запуск сессии (S8R-AUDIT-081, 2026-09-30)

- [x] Новая стратегия («Черновик»): в окне «Запустить торговлю» до нажатия кнопки видна подсказка «Запустите бэктест — статус станет «Протестирована» — или отметьте стратегию протестированной вручную»; запуск не проходит. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [~] После успешного бэктеста стратегия-черновик сама становится «Протестирована»; запуск paper-сессии переводит её в «Бумажная торговля», sandbox/real — в «Боевая». (❌ 2026-10-14 `reports/s8r_checklist_run_qa2.md` → S8R-FIX-104, исправлено в ветке `s8r/fix-checklist-run`, перепроверено на коде ветки — PASS (`reports/s8r_checklist_recheck_qa6.md`); ✅ после мержа)
- [x] Остановка последней сессии возвращает стратегию в «Протестирована»; стратегия в «Паузе»/«Архиве» запуск не принимает (текст причины в окне запуска), остановка сессии их статус не меняет. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Меню статуса: «Бумажная»/«Боевая» вручную не выбираются; смена статуса при работающих сессиях отклоняется понятным сообщением. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Дашборд: чипы «Все / Черновик / Протестированы / Активные / Пауза / Архив»; «Активные» = бумажная + боевая торговля. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)

## S8.43 Баланс счёта: доступно / в позициях (S8R-AUDIT-046, 2026-09-30)

- [x] Страница «Счёт», брокерский счёт с бумагами: «Всего» = оценка портфеля, «Доступно» = свободные рубли (не равно «Всего», если есть бумаги), «В позициях» = стоимость бумаг. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Счёт только с деньгами (рубли + валюта): «В позициях» = 0. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa4.md`)
- [x] Telegram `/balance` показывает те же «доступно» и «в позициях», без отрицательных сумм. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: `/balance` (заказчик) — оба счёта блоками, «Доступно» и «В позициях» как в API, отрицательных сумм нет)

## S8.44 Виджет «Состояние систем» при сбое (S8R-AUDIT-054, 2026-09-30)

- [x] В виджете есть строка «База данных» (OK / НЕДОСТУПНА). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)
- [x] При остановленном планировщике виджет показывает красный Scheduler, а не «Состояние систем недоступно». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa1.md`)

## S8.45 Мастер и настройки уведомлений (S8R-AUDIT-085, 2026-09-30)

- [x] Мастер, шаг 4: без привязки Telegram (`/start <код>`) статус «не подключён»; после успешного теста без привязки — по-прежнему «не подключён», Telegram для критичных событий не включается. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] «Очистить все» в списке уведомлений открывает подтверждение; «Отмена» ничего не удаляет. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)
- [x] Ошибка сохранения настройки (например, сервер недоступен): переключатель возвращается назад, показывается текст ошибки. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa2.md`)

## S8.46 Торги выходного дня (S8R-FIX-025, 2026-10-01)

- [~] Суббота/воскресенье 10:00–19:00 MSK: в шапке статус «Торги выходного дня» и время до закрытия (до 19:00); вне окна — «Выходной». (⏸ 2026-10-14: нужно EVENT; без этого проверено — `reports/s8r_checklist_run_qa3.md`)
- [~] Sandbox-сессия по бумаге, торгуемой в выходные (например, SBER), получает сигнал входа в субботнее окно — ордер уходит; по бумаге без торгов выходного дня — отказ «инструмент не торгуется в выходные». (⏸ 2026-10-14: нужно EVENT; без этого проверено — `reports/s8r_checklist_run_qa3.md`)
- [~] В субботнее окно при обрыве связи с брокером приходит уведомление «связь потеряна». (⏸ 2026-10-14: нужно EVENT; без этого проверено — `reports/s8r_checklist_run_qa3.md`)

## S8.47 Пароль, вход и дисклеймер (S8R-AUDIT-043, 2026-10-01)

- [x] Мастер настройки и «Профиль → Смена пароля»: пароль короче 12 символов или без буквы/цифры не принимается, подсказка — текст политики (не сырой текст ошибки сервера). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)
- [x] Вход с неверным паролем, с несуществующим логином и с заблокированной учёткой — одинаковое сообщение «Неверный логин или пароль»; при частых попытках — «Слишком много попыток, повторите через N с». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)
- [x] Пользователь, не принимавший дисклеймер: после входа — окно дисклеймера (текст ФТ §18.1, чекбокс «Я прочитал и принимаю условия», кнопки «Принимаю» и «Выйти»); «Выйти» разлогинивает. — ✅ заказчик, тестовый стенд, 2026-10-10
- [x] После «Принимаю» окно больше не появляется (в т.ч. после перезагрузки); запуск и «Возобновить» сессии работают. — ✅ заказчик, тестовый стенд, 2026-10-10
- [x] До принятия: запуск новой сессии и «Возобновить» отклоняются с открытием окна дисклеймера. — ✅ заказчик, тестовый стенд, 2026-10-10

## S8.48 Связь в реальном времени (S8R-AUDIT-060, 2026-10-02)

- [x] Остановить backend при открытом дашборде: после ~10 попыток переподключения — баннер «Нет связи с сервером — обновите страницу» с кнопкой «Повторить»; после запуска backend и «Повторить» баннер исчезает, котировки и уведомления идут. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)
- [x] Открыть терминал в двух браузерах (два устройства), выйти в одном — во втором сессия остаётся рабочей (нет выхода «везде»). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)
- [x] Оставить вкладку открытой дольше времени жизни токена (30 мин) — графики и статусы сессий продолжают обновляться без повторного входа и без баннера «Сессия истекла». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)
- [x] После обновления версии терминала обычная перезагрузка страницы (F5) показывает новую версию (без жёсткой очистки кеша). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)

## S8.49 Экран «Счёт»: «Заблокировано под заявки» (S8R-FIX-054, 2026-10-02)

- [x] На «Счёт» — 4 карточки: Всего / Доступно / В позициях / Заблокировано под заявки; на узком экране — 2×2, суммы не переносятся. (✅ 2026-10-12, рабочий стенд с отвечающим брокером: Всего 1 378 549,75 ₽ / Доступно 7 818,30 ₽ / В позициях 1 370 731,45 ₽ / Заблокировано 0,00 ₽)
- [x] **API-часть пройдена, телеграм-часть ждёт мержа PR #52 (S8R-FIX-084).** Выставить лимитную заявку у брокера (sandbox, **SBER 100 лотов по 250 ₽** — цена заведомо вне рынка, заявка висит) → `GET /api/v1/broker/accounts/balances` для sandbox-счёта: `blocked` равен стоимости заявки, `available` уменьшился на неё же; Telegram `/balance` показывает ту же сумму строкой «Заблокировано под заявки»; после отмены заявки обе величины возвращаются. **Переформулировано 2026-10-10 (S8R-FIX-077, решение заказчика — вариант «а»):** экран «Счёт» показывает только реальные счета (решение 2026-04-28), sandbox там не виден, а единственный реальный счёт — «Только для чтения» без прав торговли, поэтому UI-часть пункта проверяется на реальном счёте с торговыми правами, когда он появится; сама величина `blocked` для sandbox проверяется через API и Telegram (backend её отдаёт и для sandbox — фильтрует только фронт). **✅ 2026-10-12 — API-часть пройдена** (причина «молчания» брокера найдена и устранена: резолвер c-ares разрешал имя за 10–11 с при дедлайне unary-вызова 10 с, см. S8R-FIX-083). Заявка SBER 1 лот по **258 ₽** на счёте #3 (`d05ec40c-…`): 250 ₽ брокер отклоняет — `[30099] price is outside the limits`, дневной коридор 257,86–306,15 ₽, лот = 1 акция. `blocked` 0 → **258,129 ₽** (258,00 заявка + 0,129 комиссия 0,05 %), `available` 99 993,957 → 99 735,828 ₽, `total` не изменился; после отмены обе величины вернулись к исходным. Те же числа отдаёт `GET /api/v1/broker/accounts/balances` приложения. **Telegram-часть недостижима по устройству бота:** `/balance` показывает ОДИН счёт по приоритету «real с правами → real → sandbox» (`telegram_webhook.py::_handle_balance`), а при наличии real-счёта #2 песочница не выбирается никогда — заведена S8R-FIX-084. (✅ 2026-10-14, живой сценарий на рабочем стенде, sandbox #3: лимитная SBER 1 лот по 270,00 → `/balance` «Заблокировано под заявки: 270.13 ₽», доступно 98 376,15; после отмены — «0.00 ₽», доступно 98 646,29 (API приложения — те же числа))
- [x] При недоступном брокере все четыре карточки показывают «—» с подсказкой, а не «0,00 ₽» (✅ 2026-10-10 на тестовом стенде после S8R-FIX-074; **подтверждено и на рабочем стенде** с живым брокером, не отвечающим по unary). Telegram `/balance` — **частично** (2026-10-10): канал и обработчик работают, при недоступном брокере бот отвечает «❌ Ошибка получения баланса: Брокер T-Invest не ответил за 10 с (получение портфеля)» — по-человечески, без нулей и стектрейса; строка «Заблокировано под заявки» ждёт отвечающего брокера (будни).

## S8.50 Риск-менеджмент — форма Circuit Breaker (S8R-FIX-055, 2026-10-02)

- [x] `/settings` → «Риск-менеджмент»: поля заполнены сохранёнными значениями, над формой — состояние по трём классам (бумажная / песочница / реальный счёт). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)
- [x] «Просадка %» = 150 → ошибка у поля, запрос не уходит; «,5» в «Дневной лимит %» принимается. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)
- [x] Сохранить → «Настройки сохранены»; после перезагрузки страницы значения те же; пустое поле — «без ограничения». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)
- [x] Ранее сохранённое значение вне диапазона подсвечено «вне допустимого диапазона — исправьте». (✅ 2026-10-10, тестовый стенд)
- [~] На телефоне (320 px) форма и таблица состояния без горизонтальной прокрутки страницы. (❌ 2026-10-14 `reports/s8r_checklist_run_qa5.md` → S8R-FIX-108, исправлено в ветке `s8r/fix-checklist-run`, перепроверено на коде ветки — PASS (`reports/s8r_checklist_recheck_qa6.md`); ✅ после мержа)

## S8.51 Журнал сделок: «P&L не определён» (S8R-FIX-048, 2026-10-03)

- [x] Сделка sandbox/real, закрытая без цены входа (брокер не сообщил), — в журнале «P&L не определён» с подсказкой вместо числа; комиссия показана. (✅ 2026-10-10, тестовый стенд)
- [x] Telegram-уведомление о закрытии такой сделки — «не определён (брокер не сообщил цену входа)», без суммы и без «P&L −комиссия». (✅ 2026-10-12, копия данных стенда): на сделке с `entry_price IS NULL` production-код сборки payload даёт `pnl_text = 'не определён (брокер не сообщил цену входа)'`, `pnl = None`, `commission = '7.25'`. Причина выбрана верно — не «закрыта вне терминала»: приоритет в `_undetermined_text` учитывает `close_reason` только когда он есть. Доставленное сообщение не снималось: стенд копии с `DEV_MODE=true`, чтобы не слать заказчику тестовые уведомления.
- [x] Карточка сессии: процент прибыльных и средняя сделка такую сделку не учитывают; «Суммарный P&L» и последняя точка кривой капитала совпадают. (✅ 2026-10-10: Win Rate 50 %, средняя −22,05 ₽, суммарный P&L −54,62 ₽)
- [x] Старая закрытая paper-сделка без P&L (legacy) пометки не получает. (✅ 2026-10-10, сессия «ТЕСТ: legacy paper без P&L»)

## S8.52 Редактор стратегии: сохранение и ошибки (S8R-FIX-038, 2026-10-05)

- [x] Новая стратегия с ошибкой в блоках → «Сохранить» → ошибка по-русски с названием поля; в списке стратегий пустой «Новой стратегии» не появилось. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)
- [x] Пока идёт «Генерировать», кнопка «Сохранить» недоступна (и наоборот). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)
- [x] Сбой генерации в «Сгенерировать и сохранить» → пометка «код устарел» остаётся, версия не создана. (✅ 2026-10-10, тестовый стенд: backend остановлен на время шага → ошибка показана; после подъёма «Сохранить» снова даёт модалку «Код устарел», в истории новой версии нет, в шапке прежняя v4)
- [x] Дашборд: ошибка «Дублировать» — красное уведомление с текстом сервера, баннер «Не удалось загрузить стратегии» — только при ошибке загрузки списка. (✅ 2026-10-10, прогон оркестратора на копии данных стенда): «blocks_json: неизвестные типы блоков: teststand_unknown_block…», копия не создана, баннера нет
- [x] «Дублировать» стратегию с блоками → в списке «<имя> (копия)» (черновик); открыть — блоки и код те же, что у последней версии исходной (S8R-FIX-069). (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)

## S8.53 Выход без цены брокера и уведомления о закрытиях (S8R-FIX-062/063/065, 2026-10-05)

- [x] Сделка, закрытая без цены выхода от брокера: в журнале «≈ X (не подтверждена)» (или «—», если рыночной оценки нет); подсказка называет, какой цены не хватает. (✅ 2026-10-10, прогон оркестратора на копии данных стенда): выход «≈ 7 080,00 ₽ (не подтверждена)», «—» у сделки без оценки, «P&L не определён» ×3, комиссии показаны. Серый маркер «≈» на графике не проверен — без брокера свечи не грузятся
- [x] Частично исполненный вход (3 из 10 лотов): карточка сессии и таблица позиций показывают 3 лота и P&L по 3 лотам. (✅ 2026-10-10, прогон оркестратора на копии данных стенда): позиция 3 лота, P&L −120,30 ₽. Telegram `/positions` — без бота не проверить
- [x] «Стоп» сессии с открытой позицией → Telegram «Сделка закрыта» приходит. (✅ 2026-10-12, рабочий стенд, `DEV_MODE=false`): у paper-сессии #1 SBER 1h была открыта Long 11 лот @ 284,58; «Стоп» → `risk_monitor_trade_closed exit_price=284.39 pnl=-2.09 reason=session_stop trade_id=53`, `trading_session_stopped`, затем `telegram_notification_sent` ×3 (warning/info/success) — доставлено. Карточка после перезагрузки: «−348,77 ₽, Сделок: 14 (3W / 11L), WR 21 %» — закрытая позиция учтена 11-м убытком. **Вариант «сессия на паузе» не покрыт** — сессии на паузе с открытой позицией на стенде нет. Попутно найден дефект экрана: сразу после «Стоп» карточка показывает «Сделок: 0» и P&L «—» до перезагрузки — S8R-FIX-085.
- [x] Налоговый отчёт: сделки без подтверждённой цены — в отдельном разделе, не в базе. (✅ 2026-10-10, прогон оркестратора на копии данных стенда): раздел «Сделки без подтверждённой цены — уточните в брокерском отчёте» с SBER (≈ 265) и тремя LKOH; SBER 250→280 в основной таблице +297,35 ₽; база 0 — сальдирование с убытками песочницы (решение Q3=b)

## S8.54 Удаление счёта и закрытие остатка, CB и сделки без цены (S8R-FIX-052/063, 2026-10-07)

- [x] Удалить брокерский счёт, на котором есть остановленная sandbox-сессия с открытой позицией → отказ с перечнем сессий (номер, тикер, статус, число позиций); счёт не удалён. Удалить саму такую сессию → отказ «у сессии остались открытые позиции». (✅ 2026-10-10, прогон оркестратора на копии данных стенда): перечень №7 GAZP / №8 LKOH / №9 SBER; отказ по сессии №7
- [x] Страница остановленной сессии с открытой позицией → кнопка «Закрыть позиции»; после нажатия — итог по каждой сделке (закрыта / закрыта без ордера / не закрыта с причиной). (✅ 2026-10-10, прогон оркестратора на копии данных стенда): «Закрыто: 0 из 1», причина «не удалось достоверно проверить позицию у брокера… Встречный ордер не отправлен»
- [x] Позиция закрыта вне терминала (в приложении брокера) → «Закрыть позиции» закрывает сделку без ордера («закрыта вне терминала»); при неопределённости — отказ и кнопка «Позиция закрыта у брокера» с диалогом подтверждения «Ордер брокеру не отправляется…». (✅ 2026-10-10, прогон оркестратора на копии данных стенда): диалог показан, после подтверждения «Позиция №53 отмечена закрытой… Ордер не отправлялся», P&L 0 (известная S8R-FIX-071), сессию удалось удалить
- [x] «Закрыть позицию и пауза»: если позиция не закрылась — **пауза не ставится ✅, причина не показана ❌**. (2026-10-12, копия данных стенда): активная sandbox-сессия с открытой позицией, ключ счёта нечитаем → «Пауза» → «Закрыть позицию и пауза» → `close_all_positions_done count=0 failed=1`, сессия осталась `active`, позиция открыта — пауза корректно не поставлена. Но `POST /close-all` вернул `{"result":"not_closed","reason":"","error_code":null}`: причина пустая, модалка закрылась, уведомления нет. Заведена **S8R-FIX-087**; пункт закроется после её исправления. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)
- [x] Настройки уведомлений: строка «Сделка без цены и без оценки (Circuit Breaker)»; у пользователя без своей настройки каналы совпадают с «Circuit Breaker». (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)

## S8.55 Экран «Счёт» при сбое брокера, виджет «Баланс», статус брокера в футере (S8R-FIX-074/073/075/076, 2026-10-10)

- [x] Брокер недоступен (фиктивный ключ на тестовом стенде): «Счёт» открывается — четыре карточки показывают «—», страница НЕ заменяется одним красным сообщением. (✅ 2026-10-10)
- [x] В блоке «Позиции» — сообщение с текстом сервера («Брокер T-Invest не ответил за 10 с (получение портфеля)»), а не «Request failed with status code 503». (✅ 2026-10-10)
- [x] В блоке «История операций» сообщение стоит НАД таблицей: выбор диапазона дат и пагинация доступны, запрос можно повторить. (✅ 2026-10-10; попутно замечено: под ошибкой таблица пишет «Нет операций за выбранный период» — S8R-FIX-076)
- [x] Сбой скачивания налогового отчёта: сообщение над таблицей отчётов, кнопки «Скачать» остаются на месте. (✅ 2026-10-10, прогон оркестратора на копии данных стенда): «Ошибка скачивания отчёта» над таблицей, строка 2026 и кнопка «Скачать» на месте
- [x] Главная, все сессии остановлены: виджет «Баланс» показывает «Нет активных сессий» и подпись «Накопленный P&L: …» вместо отрицательной суммы как баланса. (✅ 2026-10-10, прогон оркестратора на копии данных стенда). ВАЖНО: сессии должны быть остановлены ДО сегодняшнего дня — остановленная сегодня ещё считается активной на сегодняшнюю дату
- [x] Футер при неотвечающем брокере (S8R-FIX-075): плашка **не** зелёная — «Брокер: настроен» (серая), зелёное «Брокер: на связи» только при живом стриме T-Invest; без подключённого счёта — «Брокер: не подключён». Сверить с виджетом «Состояние систем»: они больше не противоречат. (✅ 2026-10-10, видно на скриншоте редактора стратегии со стенда 6b197c6: «Брокер: настроен»)
- [x] «Счёт» при сбое операций (S8R-FIX-076): под сообщением об ошибке в таблице — «Данные не получены — см. сообщение выше». (✅ 2026-10-10, прогон оркестратора на копии данных стенда)
## S8.56 Сверка с брокером и «P&L не определён» вне терминала (S8R-FIX-070, S8R-FIX-071, 2026-10-10)

- [x] У остановленной sandbox/real-сессии с плашкой «Расхождение с брокером» в карточке есть кнопка «Сверить с брокером» (оранжевая, рядом с «График»); у paper-сессии и у работающей сессии кнопки нет. (✅ 2026-10-12, прогон оркестратора на копии данных стенда — свой backend :8100 + vite :5273, пароль заменён только в копии): у sandbox-сессии #3 три кнопки — «Сверить с брокером», «График», «Удалить»; у остановленной paper-сессии #1 только «График» и «Удалить». **Оговорка:** работающей sandbox/real-сессии на стенде нет, проверено на paper (остановленной и активной).
- [x] Нажатие при сошедшихся позициях: плашка исчезает без перезагрузки страницы, фильтры списка сессий не сбрасываются. (✅ 2026-10-12, прогон оркестратора на копии данных стенда — свой backend :8100 + vite :5273, пароль заменён только в копии): при включённом фильтре «Остановленные» плашка пропала, список остался отфильтрованным (активная сессия не вернулась). Лог: `manual_reconcile_done result=match session_id=3`, `POST /trading/sessions/3/reconcile 200`; в БД `position_mismatch_at → NULL`; в журнале действий `session.reconciled {"mismatch_cleared": true}`. Вызов к живому sandbox-счёту, не мок.
- [x] Нажатие при реальном расхождении: плашка остаётся, её текст обновляется на actual. (✅ 2026-10-12, прогон оркестратора на копии данных стенда — свой backend :8100 + vite :5273, пароль заменён только в копии): создана открытая сделка 3 лота SBER, которой у брокера нет → `result=mismatch`, плашка на месте, подсказка в UI «SBER: у брокера позиции нет, у нас открыто 3 шт (сделки №54). Позиции не изменены — сверьте с приложением брокера.»; время пометки обновилось на момент сверки, то есть текст не остался прежним.
- [x] Брокер не отвечает: сообщение «Не удалось сверить позиции: брокер не ответил…», плашка на месте. (✅ 2026-10-12, прогон оркестратора на копии данных стенда — свой backend :8100 + vite :5273, пароль заменён только в копии) с `TINVEST_UNARY_TIMEOUT_SEC=0.01`: «Не удалось сверить позиции: брокер не ответил или портфель пришёл неполным. Пометка расхождения оставлена, повторите позже», плашка не снята.
- [x] Удаление брокерского счёта при висящем расхождении — 409; после успешной сверки счёт удаляется. (✅ 2026-10-12, прогон оркестратора на копии данных стенда — свой backend :8100 + vite :5273, пароль заменён только в копии): `DELETE /broker-accounts/3` → **409** `broker_account_in_use`, в теле `position_mismatch: true` и текст «У сессий висит расхождение позиций с брокером — снимите его кнопкой «Сверить с брокером» на странице сессии»; после `reconcile` → `{"status":"cleared"}` тот же запрос вернул **204**.
- [x] Журнал сделок: позиция, закрытая в приложении брокера, показывает «P&L не определён», подсказка — «Позиция закрыта вне терминала…». (✅ 2026-10-12, прогон оркестратора на копии данных стенда — свой backend :8100 + vite :5273, пароль заменён только в копии): строка 25.06.2026 — вход 295,02 ₽, выход «—», P&L «P&L не определён», комиссия 12,50 ₽; подсказка «Позиция закрыта вне терминала — итог сделки неизвестен, в статистике дня не учтён». Соседние сделки той же сессии показывают обычный P&L — пометка адресная.
- [x] Telegram-уведомление о такой сделке: «не определён (позиция закрыта вне терминала)», без суммы и без «минус комиссия». (✅ 2026-10-12, прогон оркестратора на копии данных стенда — свой backend :8100 + vite :5273, пароль заменён только в копии). **Способ проверки:** вызван production-код сборки payload (`trade_closed_payload`) на реальной сделке из копии — `pnl_text = 'не определён (позиция закрыта вне терминала)'`, `pnl_undetermined = True`, `pnl = None`, `commission = '12.50'`. Доставленное сообщение не снималось: стенд копии работает с `DEV_MODE=true`, чтобы не слать заказчику тестовые уведомления.
- [x] Налоговый отчёт (csv): сделка в разделе «Сделки без подтверждённой цены», её комиссия в базу не входит. (✅ 2026-10-12, прогон оркестратора на копии данных стенда — свой backend :8100 + vite :5273, пароль заменён только в копии): отчёт за 2026 — `Комиссии:;0`, база 0; в разделе «Сделки без подтверждённой цены — уточните в брокерском отчёте» строка `05.08.2026;SBER;buy;песочница;33;295.02;—;12.50`. Подтверждает на данных решение заказчика 2026-10-12 (вопрос 5, вариант «а»).
- [x] «Состояние систем» и экран «Счёт» после сверки не изменились. (✅ 2026-10-12, прогон оркестратора на копии данных стенда — свой backend :8100 + vite :5273, пароль заменён только в копии): снимки `GET /broker/accounts/balances` и `GET /health` до и после `reconcile` (`status: cleared`) совпали посимвольно.


## S8.57 Экраны после перевода на правила React-компилятора (S8R-FIX-080, 2026-10-12)

- [x] График: переключение таймфрейма (Д → 1ч) перерисовывает свечи нового ТФ, overlay OHLC пересчитывается, свечей старого ТФ в наборе не остаётся и график не «мигает». (✅ 2026-10-12, Playwright-прогон оркестратора 2026-10-12 на своём vite :5273 из ветки `s8r/fix-080-react-compiler-rules` с моками `e2e/fixtures/api_mocks.ts`; скриншоты — `s8r-evidence/s8r-fix-080/`): на Д — 60 свечей, «Источник: T-Invest», overlay ОТКР 304,50 / МАКС 307,50 / МИН 301,50 / ЗАКР 305,50; после клика «1ч» — свечи часового набора, sequential-index режим без разрывов, overlay пересчитан. Сброс WS-канала при смене ТФ закреплён юнит-тестом `ChartPage.channelReset.test.tsx` (мутация «ключ без таймфрейма» → падение).
- [x] Главная: при первом открытии все четыре виджета наполняются данными — «Баланс» (сумма, sparkline, дневная дельта), «Состояние систем», «Активные позиции», sparkline инструмента, — без промежуточного кадра с пустым состоянием. (✅ 2026-10-12, Playwright-прогон оркестратора 2026-10-12 на своём vite :5273 из ветки `s8r/fix-080-react-compiler-rules` с моками `e2e/fixtures/api_mocks.ts`; скриншоты — `s8r-evidence/s8r-fix-080/`): Баланс 1 372 500 ₽ / +1 200 ₽ (+0,09 %) за день со sparkline, «Состояние систем» с четырьмя строками, «Активные позиции 0», SBER 295,1 / +5.39 % со sparkline. Отсутствие кадра «Нет данных за 24 ч» у sparkline закреплено юнит-тестом (мутация «loading без начального значения» → падение).
- [x] Редактор стратегии: при открытии `/strategies/{id}` поля заполняются данными ЭТОЙ стратегии, а не предыдущей открытой (описание, версия, код, блоки). (частично, 2026-10-12): закреплено юнит-тестом `StrategyEditPage.test.tsx` — форма наполняется ответом `fetchStrategy`, мутация «описание из стора вместо ответа» → падение. **Визуально на стенде не снималось:** нужен путь «открыть стратегию A → вернуться к списку → открыть стратегию B» на реальных данных. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)


## S8.58 Таблицы и подписи после хвостов аудита (S8R-FIX-079/007/014/008/022, 2026-10-13)

- [x] Бэктест → «Сделки»: таблица листается по 10 строк, подпись «Показано 1-10 из 25» меняется на «11-20» и «21-25», на последней странице строк меньше десяти. (✅ 2026-10-13, Playwright `e2e/s8r-fix-079-tables-v9.spec.ts` на своём vite :5273 из ветки `s8r/fix-tails-ui`, моки `e2e/fixtures/api_mocks.ts`): 25 сделок, три страницы, наборы строк не совпадают.
- [x] Бэктест → «Сделки»: переключатель «Все / Прибыльные / Убыточные» оставляет только соответствующие сделки И возвращает на первую страницу (иначе после фильтра виден пустой экран). (✅ 2026-10-13, тот же прогон): со страницы 3 фильтр «Прибыльные (13)» → «Показано 1-10 из 13», все P&L положительные; «Убыточные (12)» → все отрицательные.
- [x] Бэктест → «Сделки»: клик по заголовку сортирует в обе стороны — «P&L» как число (не как строку), «Вход» как дату (01.01 раньше 25.01, а не по алфавиту). (✅ 2026-10-13, тот же прогон): первый клик по «P&L» — от +2 500 ₽, второй — от −2 400 ₽; «Вход» — 01.01.2026 ↔ 25.01.2026.
- [x] Экран «Счёт» → «Позиции»: сортировка по «Тикер» (алфавит) и «P&L» (число, первый клик — от большего плюса), все столбцы на месте, включая бейдж источника «Стратегия»/«Внешняя». (✅ 2026-10-13, тот же прогон): SBER/GAZP/LKOH, один бейдж «Стратегия» и два «Внешняя».
- [x] Торговля → журнал сделок: сделка в статусе `pending` подписана «ожидает брокера», а в позиции вместо прочерка стоит та же подпись. (частично, 2026-10-13): закреплено юнит-тестами (4). **Визуально не снималось** — нужен ордер, который брокер не подтвердил сразу. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)
- [x] Форма запуска сессии: под суммой видно «1 лот = N шт ≈ X ₽», и при сумме меньше стоимости лота появляется предупреждение. (частично, 2026-10-13): закреплено юнит-тестами (3 frontend, 2 backend). **Визуально не снималось** — нужен инструмент с лотом > 1 на живом стенде. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)
- [x] Уведомление о корпоративном событии содержит обе даты: «закрытие реестра DD.MM.YYYY, ex-date DD.MM.YYYY» (при недоступном календаре MOEX — только дату реестра). Требует события по бумаге в портфеле. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)
- [x] График: после обрыва стрима бар, переживший обрыв, исправляется на своём месте, а часы, целиком попавшие в обрыв, появляются без перезагрузки страницы. Требует обрыва стрима на живом брокере. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)
- [x] Настройки → «Риск-менеджмент»: в блоке состояния над формой просадка бумажной сессии с ОТКРЫТОЙ позицией совпадает с той, по которой движок останавливает торговлю (учитывает нереализованный P&L). Требует живой paper-сессии с открытой позицией. (✅ 2026-10-14, сплошная проверка — `reports/s8r_checklist_run_qa5.md`)


| S8.20 Комиссия в paper | 6 | S8R-PAPER-NO-COMMISSION |
| S8.21 Метрика полного окна | 3 | S8R-METRIC-SIGNAL-TO-ORDER-BLIND |
| S8.22 Комиссия sandbox/real и тариф | 7 | S8R-SANDBOX-REAL-COMMISSION-NOT-RECORDED, S8R-PAPER-COMMISSION-BROKER-RATE |
| S8.23 Комиссия за период | 4 | Q6, решение заказчика 2026-08-04 |
| S8.24 Режим сделки в 3-НДФЛ | 5 | S8R-TAX-SANDBOX-IN-DECLARATION, S8R-TAX-PNL-LOSES-LOT-SIZE |
| S8.25 SL/TP доходит до брокера | 7 | S8R-SLTP-SANDBOX-NO-BROKER-ORDER |
| S8.26 Повторное закрытие сделки | 4 | S8R-CLOSE-POSITION-NO-LOCK |
| S8.27 Уведомление о закрытии сделки | 4 | S8R-TELEGRAM-TRADE-CLOSED-GROSS-PNL |
| S8.28 Сверка с брокером и доучёт ордера | 11 | S8R-EXIT-ORDER-NOT-TRACKED, S8R-SANDBOX-POSITIONS-NOT-RECONCILED, S8R-TINVEST-SANDBOX-FLAKY-70001 |
| S8.29 Настоящий FIGI и фоновая сверка | 10 | S8R-SANDBOX-TRADES-FAKE-FIGI, S8R-RECONCILE-LOCK-DIVERGENCE, S8R-RECONCILE-NOT-PERIODIC |
| S8.30 Один инструмент — одна сессия на счёте | 11 | S8R-DUPLICATE-SESSION-PER-INSTRUMENT, S8R-RECONCILE-NO-ACCOUNT-DEDUP, S8R-PAPER-CONFLICT-BY-VERSION-NOT-TIMEFRAME |
| S8.31 «Запустить заново» для sandbox/real | 7 | S7R-SESSION-RERUN-PAYLOAD-BROKEN |
| S8.32 Удаление счёта с живыми сессиями | 7 | S8R-DELETE-ACCOUNT-ORPHANS-SESSIONS |
| S8.33 Курс доллара от ЦБ РФ | 6 | S9-MULTICURRENCY-CBR-RATE |
| S8.34 Сборка production-образа фронтенда | 4 | S8R-FRONTEND-BUILD-BROKEN |
| S8.35 Повторный резолв FIGI | 4 | S8R-FIGI-NO-NEGATIVE-CACHE |
| S8.36 Авто-сворачивание фоновых бэктестов | 5 | S7R-BG-BACKTEST-AUTOCOLLAPSE |
| S8.37 Тариф счёта для paper-сессии | 7 | S8R-PAPER-TARIFF-UNREACHABLE-FROM-UI |
| S8.38 Режим B — импорт из шаблона | 6 | S5R-BLOCKLY-MODE-B-MODAL, S5R-BLOCKLY-MODE-B-CHECK |
| S8.39 Регистрация закрыта после первого пользователя | 4 | S8R-AUDIT-033 |
| S8.40 Удаление стратегии, расхождение позиций, пределы риск-параметров | 5 | S8R-AUDIT-069, S8R-AUDIT-100, S8R-AUDIT-025, S8R-AUDIT-080 |
| S8.41 Short-стратегия в торговле | 3 | S8R-AUDIT-032 |
| S8.42 Статус стратегии и запуск сессии | 5 | S8R-AUDIT-081 |
| S8.43 Баланс счёта: доступно / в позициях | 3 | S8R-AUDIT-046 |
| S8.44 Виджет «Состояние систем» при сбое | 2 | S8R-AUDIT-054 |
| S8.45 Мастер и настройки уведомлений | 3 | S8R-AUDIT-085 |
| S8.46 Торги выходного дня | 3 | S8R-FIX-025 |
| S8.47 Пароль, вход и дисклеймер | 5 | S8R-AUDIT-043 |
| S8.48 Связь в реальном времени | 4 | S8R-AUDIT-060 |
| S8.49 «Заблокировано под заявки» | 3 | S8R-FIX-054 |
| S8.50 Риск-менеджмент (форма CB) | 5 | S8R-FIX-055 |
| S8.51 «P&L не определён» | 4 | S8R-FIX-048 |
| S8.52 Редактор: сохранение, ошибки, «Дублировать» | 5 | S8R-FIX-038, S8R-FIX-069 |
| S8.53 Выход без цены, уведомления закрытия | 4 | S8R-FIX-062/063/065 |
| S8.54 Удаление счёта, остаток, CB без цены | 5 | S8R-FIX-052/063 |
| S8.55 Экран «Счёт» при сбое брокера, виджет «Баланс», статус брокера в футере | 7 | S8R-FIX-074, S8R-FIX-073, S8R-FIX-075, S8R-FIX-076 |
| S8.56 Сверка с брокером, «P&L не определён» вне терминала | 9 | S8R-FIX-070, S8R-FIX-071 |
| S8.57 Экраны после правил React-компилятора | 3 | S8R-FIX-080 |
| S8.58 Таблицы и подписи после хвостов аудита | 9 | S8R-FIX-079, S8R-FIX-007, S8R-FIX-014, S8R-FIX-008, S8R-FIX-022, S8R-FIX-015 |
| **Итого S8 (новых пунктов)** | **358** | |

**Расширение S7 базы:** 136 новых пунктов > требуемых 50. Можно дополнить дополнительными edge-cases по факту QA-прогона в 8.R.
