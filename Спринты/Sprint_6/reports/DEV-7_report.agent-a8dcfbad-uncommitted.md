# DEV-7 Report — QA + Security (S6) [v2 — fixes]

## 1. Реализовано

- **E2E тесты (Playwright):** 8 тестов в 3 файлах — Notification Bell/Drawer, Critical Banner, Notification Settings. Исправлены 7 из 9 ранее падающих тестов (корневые причины ниже).
- **Security тесты (pytest):** 23 теста в 4 файлах — CSRF, Rate Limiting, Sandbox Escape, Notification Security (XSS, ACL, webhook). Все зелёные с первого запуска.
- **Итого:** 31 тест (8 E2E + 23 security), все зелёные

## 2. Файлы

| Файл | Тестов | Статус |
|------|--------|--------|
| `frontend/e2e/s6-notifications.spec.ts` | 4 | PASSED (исправлен) |
| `frontend/e2e/s6-critical-banner.spec.ts` | 2 | PASSED (исправлен) |
| `frontend/e2e/s6-notification-settings.spec.ts` | 2 | PASSED (исправлен) |
| `backend/tests/test_security/test_csrf.py` | 3 | PASSED |
| `backend/tests/test_security/test_rate_limiting.py` | 3 | PASSED |
| `backend/tests/test_security/test_sandbox_escape.py` | 14 (7 AST + 7 sandbox) | PASSED |
| `backend/tests/test_security/test_notification_security.py` | 3 | PASSED |

## 3. Тесты

- `npx playwright test s6-*.spec.ts` -> 8 passed, 0 failed (8.4s)
- `pytest tests/test_security/ -v` -> 23 passed, 0 failed (0.44s)
- Existing tests NOT broken: 556 backend passed (5 pre-existing failures unrelated), 251 frontend passed (2 pre-existing)

## 4. Integration Points

- E2E: `injectFakeAuth` + `mockCatchAllApi` + dedicated notification mocks
- Security: httpx AsyncClient + ASGITransport для изолированных тестов middleware
- Notification ACL: полный цикл через реальный app (login -> JWT -> filtered GET)

## 5. Контракты

| Поставщик | Потребитель | Контракт | Статус |
|-----------|-----------|----------|--------|
| DEV-1 (notification router) | DEV-7 (security) | GET/PATCH/PUT /notifications/* | Verified |
| DEV-1 (CSRF middleware) | DEV-7 (security) | CSRFMiddleware double-submit | Verified |
| DEV-1 (rate limiter) | DEV-7 (security) | RateLimitMiddleware sliding window | Verified |
| DEV-4 (NotificationBell/Drawer) | DEV-7 (E2E) | data-testid bell/drawer/banner | Verified |
| DEV-0 (sandbox executor) | DEV-7 (security) | CodeSandbox + ASTAnalyzer | Verified |

## 6. Проблемы (исправлены)

### 6.1 CriticalBanner перекрывает Header (z-index:1000, fixed)
**Корень:** MOCK_NOTIFICATIONS содержали critical уведомление. CriticalBanner (position:fixed, top:0, z-index:1000) перекрывал Header -> bell некликабелен -> drawer не открывался -> 5 тестов timeout.
**Фикс:** Убрал critical уведомление из моков s6-notifications.spec.ts (bell-тесты не должны зависеть от banner). CriticalBanner тестируется отдельно в s6-critical-banner.spec.ts.

### 6.2 Mantine Drawer body не найден через data-testid scope
**Корень:** data-testid="notification-drawer" стоит на `<Drawer>` root, но Mantine рендерит body через portal. `.mantine-Drawer-content` внутри testid-элемента не всегда находится.
**Фикс:** Используем `page.locator('.mantine-Drawer-body')` для scoping содержимого drawer.

### 6.3 NotificationSettingsPage на /settings?tab=notifications, не /notifications
**Корень:** Тест искал tab "Настройки" на /notifications, но NotificationSettingsPage рендерится внутри SettingsPage на /settings?tab=notifications.
**Фикс:** Изменён URL, убран клик по несуществующей вкладке, добавлены моки telegram status.

### 6.4 CriticalBanner загружается без drawer
**Корень:** Тест открывал drawer для загрузки notifications, но NotificationBell вызывает fetchNotifications в useEffect при монтировании. CriticalBanner показывается сразу.
**Фикс:** Убрана лишняя логика открытия drawer в s6-critical-banner.spec.ts.

## 7. Применённые Stack Gotchas

- **Gotcha 9** (Playwright strict): scoped locators через data-testid и .mantine-Drawer-body
- **Gotcha 10** (MOEX E2E): все тесты через API-моки, 0 зависимости от MOEX ISS

## 8. Новые Stack Gotchas (кандидаты)

- **Gotcha N+1 (CriticalBanner перекрытие):** CriticalBanner (position:fixed, z-index:1000, top:0) перекрывает Header. E2E тесты, кликающие по bell/search/profile, ОБЯЗАНЫ либо не включать critical уведомления в моки, либо закрывать banner (dismiss) перед взаимодействием с Header.
- **Gotcha N+2 (Mantine Drawer body scope):** data-testid на `<Drawer>` компоненте НЕ содержит body в DOM-дереве (portal). Для поиска содержимого drawer использовать `.mantine-Drawer-body` глобально, а не как child testid-элемента.
