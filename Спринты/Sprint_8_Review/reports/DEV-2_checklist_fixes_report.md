# DEV-2 — фиксы сплошной проверки UI-чеклиста S8 (2026-10-14)

Worktree `wt-s8r-fix`, без коммитов; Node 20 (24 нет, baseline зелёный).

## 1. Реализовано (RED → GREEN → мутация)

- **103**: один PUT, стор правится и откатывается локально (`patchStrategyLocal`). RED: `called 1 times, but got 2 times`.
- **104**: «Бумажная/Боевая торговля»; у инструментов «Бумажная/Песочница/Боевая». RED: `Unable to find … Бумажная торговля`.
- **105**: открытый список сам не закрывается. RED: `expected false to be true`.
- **106**: testid `event-type-row-*`. RED: `Unable to find [data-testid="event-type-row-trade_opened"]`.
- **107**: `brokerAccountName` (8 мест), `||` в PauseConfirmModal. RED: `'…на счёте «» уже е…' to contain '«№7»'`.
- **108**: `collapsed.mobile` + мобильный бургер, переход закрывает меню; шапка 320 px без переноса. RED: e2e на старом коде `not.toBeInViewport … ratio 1`.
- **109**: общие форматтеры в МСК (+`formatTime`): журнал, карточка сессии, PnL-тултип, налоговые отчёты, проверка счёта. RED: `'12.10.2026 11:28' to be '…18:28'`.
- **110**: `formatPercent`. **111**: `minWidth 240` + перенос кнопок, вживую 66 → 398 px. **112**: 7 графиков. **113**: «Удалить» заблокирована во время DELETE (RED: `got 2 times`).
- **2.12 Н-5 — PASS**: на 429 — «Слишком много запросов», статус не сбрасывается. **Н-4 — PASS**: DateMaskInput хранит свой текст, `<label for>`.

## 2. Файлы

Новые: `utils/brokerAccountName.ts` + 10 тест-файлов; изменены 25 файлов `src/` и `e2e/s5-circuit-breaker.spec.ts`.

## 3. Тесты

vitest 188/188 файлов, 1441/1441; ~40 новых тестов. `pnpm typecheck` и `pnpm lint` — 0. E2E (моки, CI=1, свой Vite): 173 passed / 9 failed / 1 skipped, все падения вне моих правок (п. 6). Стенд 3 — скриншоты в `s8r-evidence/checkrun/results/dev2/`: PUT 200 ×1; dblclick → DELETE 204 ×1; 320 px; МСК при браузере в Берлине; «№4».

## 4. Integration points

Все новые функции вызываются в prod-коде (grep). NOT CONNECTED нет.

## 5. Контракты

Без изменений.

## 6. Проблемы, развилки

- **Оркестратору**: `s6-notifications` (2) и `s6-critical-banner` (1) красные из-за S8R-FIX-100. URL стал `/notifications/`, а моки ловят `**/notifications?*`. `auth-hardening` (6) требует backend с чистой БД.
- 104: у строки с сессией подпись берётся из `SESSION_MODE_LABEL` — без его правки фикс был бы невидим.
- DEV-3: backend `STRATEGY_STATUS_LABELS["live"]="Боевая"`, а в UI «Боевая торговля»; пустое имя sandbox-счёта.
- Вне торговых экранов время ещё браузерное (бэктесты, уведомления, версии).
- Перезагрузка `/admin/*` уводит на «/» до загрузки профиля (ProtectedAdminRoute, не правил).
- Убрать: `s8r-evidence/checkrun/dev2-vite-cache` (`rm -r` запрещён). Свой Vite остановлен, конфиг удалён. Стенд 3: временная стратегия #7 создана и удалена; testuser1 открывал настройки брокера.

## 7. Gotchas

40, 45, 46, 60.

## 8. Новые gotchas

react-imask на каждом `componentDidUpdate` переписывает маску из `value` → контролируемое поле теряет частичный ввод.

## 9. Плагины

typescript-lsp → `pnpm typecheck`; context7 не авторизован → d.ts Mantine; playwright; `mattpocock-skills:tdd`.
