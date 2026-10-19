# Прогресс фиксов аудита — 2026-09-24

Промпт цикла: `prompt_s8r_fixes.md` (локальный). Порядок карточек — `prompt_s8r_audit_fixes.md` §1–§4, полный текст карточек — `audit_2026-09.md` §3.

## Ответы заказчика (Такт 1, 2026-09-24)
Q1=d (всё подряд до исчерпания ресурса, остановка на границе пакета) · Q2=a (PR #27 мержит исполнитель) · Q3=a (ветка на пакет `s8r/fix-blocker|high|medium|low`, коммит+пуш после карточки, PR после гейтов, **мерж — по команде заказчика**) · Q4: 001=a (исполняется только код из IR; fallback `generated_code` удаляется), 033=a (`/auth/setup` закрыт после первого пользователя, новые — только admin) · Q5: 032=**b** (чинить учёт денег под short, ФТ менять), 024/030=a (10 с), 027=a (свободный кэш / доступные средства), 070=a (paper и sandbox/real раздельно), 091=a (ISS-источники сплитов/купонов + предупреждение, без back-adjust) · Q6: 081=a (запуск только `active`) + **автоперевод статуса при запуске — да**; 087 — по рекомендации: (а) делистинг → понизить до Could, (б) валидация OHLCV → реализовать, (в) автопауза после N ошибок → реализовать, (г) частичное исполнение → реализовать (таймаут/отмена остатка/уведомление поверх 025), (д) §12.8 → актуализировать ФТ, (е) §4.2 → актуализировать ФТ; 095=a (единая база обращающихся ЦБ, ст. 214.1 НК РФ) · Q7=a (S-1/S-2/S-7 на счёте #3 после HIGH) · Q8=**b** (DEV-субагенты по `prompt_template.md`, ≤ 2 одновременно) · Q9=a (:8000/:5173) · Q10=a · Q11=**b** (F/H1/I2 ревьюерами sonnet после пакетов) · Q12: проверка `backend/logs/dev.log` поручена исполнителю «только числа» — **токена нет** (см. ниже) · Q13=b (команды — текстом в отчёте).

### Q12 — проверка `Develop/backend/logs/dev.log` (2026-09-24, только счётчики, значения не выводились)
Файл 25 МБ, последняя запись 08.07, права `rw-r--r--`. `api_key = '` → 0; рамки rich-traceback `│` → 0; токеноподобные `t.[A-Za-z0-9_-]{30,}` → 0; `api_key` → 20 — все в тексте исходника внутри traceback (`api_key, api_secret = …(` из `service.py:605`), значений нет. Вывод: перевыпуск токена не требуется; гигиена по желанию — `chmod 600`.

### Ответы заказчика на накопившиеся вопросы (2026-09-30, продолжение цикла)
- **Торги выходного дня (093/072)** — разрешить входы sandbox/real в Сб/Вс **по расписанию ISS** (для инструментов, реально торгующихся в выходные); ФТ §1.6/1.7 меняются. → карточка S8R-FIX-025.
- **Удаление стратегии с историей сессий (069/015)** — **архивировать**: при наличии истории стратегия переводится в `archived` и скрывается из списка, история цела; без истории — физическое удаление. → S8R-FIX-026.
- **`block_shorts` (032)** — **удалить флаг** (колонка, API-поле, проверка; миграция). → S8R-FIX-027.
- **Порог `SECRET_KEY` (034)** — **32 байта** (как сделано); команда проверки длины ключа — заказчику текстом.
- **Дневной лимит CB (070)** — **три класса раздельно**: paper / sandbox / real. → S8R-FIX-028.
- **PR #28/#29** — мержить оба сейчас: #28 → develop, база #29 → develop, зелёный CI, мерж #29, CI на develop.
- **S-1/S-2/S-7 на счёте #3** — сейчас, первым шагом, на коде `s8r/fix-high`.

### Уточнение Q6-081 (2026-09-30)
Статуса `active` в модели нет (6 значений). Решение заказчика: запуск сессии — из **tested / paper / live** (как фильтр «Активные»); draft, paused, archived — отказ с подсказкой. Автоперевод при запуске: paper-сессия → `paper`, sandbox/real → `live` (live старше paper); при остановке **последней** живой сессии стратегии → `tested`. Дополнение (2026-09-30): после **успешного бэктеста** стратегия `draft` → `tested` автоматически (прочие статусы бэктест не меняет).

### Ответы заказчика 2026-10-01 (перед продолжением)
- Сертификат НУЦ Минцифры получен: `Test/russian_trusted_root_ca_pem.crt` (в docs-репо не коммитить), SHA-256 `D2:6D:…:CF:31` — совпал с отданным сервером T-Invest и с gotcha-55; действует до 27.02.2032.
- **S-1/S-2/S-7** — сейчас, на коде MEDIUM (`s8r/fix-medium`), один прогон на оба пакета.
- **Находки S8R-FIX** — после пакета LOW, в этом цикле, отдельным пакетом (кроме требующих решения заказчика).
- **ПФИ в налогах (S8R-FIX-043)** — фьючерсы, валюта, опционы — отдельной строкой вне базы ЦБ («рассчитайте с бухгалтером»).
- **Экран «Счёт»** — четвёртая карточка «Заблокировано под заявки» (в пакет LOW).
- **Мерж PR** — исполнитель мержит сам при зелёном CI: #29 после живых сценариев → PR корня (FIX-029) → PR MEDIUM после трёх осей ревью → PR LOW; после каждого — CI на develop.
- **AI_MODEL по умолчанию (S8R-FIX-033)** — `claude-opus-5-5`.

### Ответы заказчика 2026-10-07 (после сдачи пакета S8R-FIX)
- **S8R-FIX-063 — да:** сделка без подтверждённой цены учитывается в дневном лимите убытка CB консервативной оценкой (реализовать в следующем цикле).
- **S8R-FIX-052 — запретить:** удаление брокерского счёта отклоняется (409), если на него ссылается остановленная сессия с открытыми позициями (дополнение к запрету A4 для active/paused/suspended).
- **S8R-FIX-058 — на текущий момент нет:** фьючерсы в поиске инструментов не показываем; карточка остаётся отложенной.

### Ответы заказчика 2026-10-10 (вечер, такт 1 нового цикла)
- **S8R-FIX-077 — вариант (а):** пункт S8.49 п.2 переформулировать (sandbox проверяется через `GET /broker/accounts/balances` и Telegram `/balance`, UI — на реальном счёте с правами, когда появится). Решение 2026-04-28 (sandbox только в «Торговле») не отменяется; показ sandbox на «Счёте» — предложение в S9.
- **S8R-FIX-070 — вариант (а):** делать действие «Сверить с брокером» для остановленной sandbox/real-сессии + учёт неснятой пометки в запрете удаления счёта.
- **S8R-FIX-071 — вариант (а):** чинить полностью — признак «P&L не определён» на «закрыта без ордера терминала», с учётом в журнале, Telegram, CB (063 ч.2) и налоговом разделе.
- **Dependabot — вариант (в), всё разом:** 11 открытых PR (#33–#43), включая мажоры `cryptography` 46→50, `protobuf` 4.25→7.36, `openai` 2→3, `@tanstack/react-table` 8→9, `vitest` 4→5, `@types/node` 24→26. Проверки (включая Playwright E2E) исполнитель запускает сам, заказчика не привлекать.
- **Лицензии GPL/LGPL — вариант (а):** утверждено для self-hosted single-user; решение обратимо — при любом распространении пересматривается (оговорка в `docs/licenses.md`).
- **Остаток чеклиста на живом брокере** (заявка SBER 100×250, `/balance`, S8.51 п.2, S8.53 п.3, S8.54 п.4) — к обсуждению в будни, параметры пока не подтверждены.
- **Рабочий стенд** остаётся работать как есть (`DEV_MODE=false`, Telegram живой, ngrok под Proxifier) — не гасить.
- **Docker-сборка на Mac mini** (A2/A8) — вопросы «Docker установлен? доступ по SSH или сам?» остались без ответа, поднять перед деплоем.

### Новый блокер — TLS T-Invest (2026-09-30, обнаружен при подготовке S-1/S-2/S-7)
Уточнение: проблема известна с 2026-08-05 — **gotcha-55** (обход `GRPC_DEFAULT_SSL_ROOTS_FILE_PATH`), но в гайд/compose/Dockerfile не попала; утверждение «стенд заказчика тоже не подключится» — не проверено (зависит от окружения заказчика). T-Bank перевёл gRPC API (`*.tinkoff.ru`, `*.tbank.ru`, sandbox и prod) на сертификаты НУЦ Минцифры (Russian Trusted Root CA). gRPC терминала (grpc 1.80, встроенные корни) не доверяет корню → **терминал не подключается к T-Invest вообще**. Диагностика без токена: стандартные корни → `Tls handshake failed`/DEADLINE; + корень Минцифры → `UNAUTHENTICATED 40003` (TLS проходит). Отпечаток корня, отданного сервером: SHA-256 `D2:6D:2D:02:31:B7:C3:9F:92:CC:73:85:12:BA:54:10:35:19:E4:40:5D:68:B5:BD:70:3E:97:88:CA:8E:CF:31`. Это же — причина `CERTIFICATE_VERIFY_FAILED` в тестах.
Решения заказчика: (1) **корень встраивается в код, только для gRPC** (стандартные корни + Минцифры, PEM в репо, отпечаток закреплён тестом) → карточка S8R-FIX-029 (BLOCKER, первой в `develop`); (2) PEM скачивает заказчик с gosuslugi.ru/crt в `s8r-evidence/fixes/russian_trusted_root_ca.pem`, исполнитель сверяет отпечаток с отданным сервером; (3) S-1/S-2/S-7 — сегодня на коде HIGH с временным корнем (переменная окружения только для процесса стенда), затем мерж #29.
PR #28 смёржен 2026-09-30 (`15bb244`), база #29 → `develop`.

## Среда
worktree A: `/Users/sergopipo/Documents/Claude_Code/wt-s8r-fixes` | ветка: `s8r/fix-blocker` (от `origin/develop` 7ddb970, запушена: `9f676aa`) | PR: #28 | develop на входе: `eb769cc` → после мержа PR #27 `7ddb970`
worktree B (параллельный DEV, detached HEAD, без веток): `/Users/sergopipo/Documents/Claude_Code/wt-s8r-fixes-b`
стенд: поднимался для E2E BLOCKER (PID 63943/63944), остановлен; БД стенда удалена; кэш Vite `wt-s8r-fixes/frontend/node_modules/.vite*` — уйдёт с worktree | симлинк .env: нет
гейты BLOCKER «после» (9f676aa): pytest **2711 / 19 xfailed / 0**, coverage **87 %**, ruff 0, mypy Success (179), bandit M0/H0, typecheck/lint/build 0, vitest 925 + 1 флейк под нагрузкой (S8R-FIX-005), E2E 171/1/0.
гейты «до» (worktree A, 7ddb970): pytest **2625 passed / 25 xfailed / 0 failed** (193 с), coverage **87 %** | ruff 0 | mypy Success (178) | bandit Medium 0 / High 0 (Low 46) | alembic heads `a7b8c9d0e1f2` | typecheck 0 | lint 0 | build ok | vitest 924 passed + 2 expected fail / 132 файла | E2E — после пакета | gotcha: 60, INDEX v24

## Карточки
| Карточка | Пакет | Статус | RED (фактическая строка) | GREEN | Мутация | Гейты | Коммит | Ревью треб./рецепт/качество | Примечание |
|---|---|---|---|---|---|---|---|---|---|
| S8R-AUDIT-033 | BLOCKER | ✅ | `assert 201 == 403`; гонка `[201, 201] == [201, 403]`; admin `404 == 422`; лимитер `'general' == 'auth'` | 3 новых файла тестов + доказательный | setup без гейта → 3 failed; setup-status→auth → 1 failed | 2633/24xf/0; ruff 0; mypy ok; bandit 0; tsc 0; lint 0; build ok; vitest 926 | `967fe10` | — / — / — (после пакета) | /code-review: setup-status попадал в лимит auth — исправлено |
| S8R-AUDIT-002 | BLOCKER | ✅ | `DID NOT RAISE (ValidationError, ValueError)` ×2 | 2 доказательных + 15 новых + grid 422 | `_INDICATOR_COMMON = {}` → 12 failed; grid без проверки / OverflowError → red | 2662/23xf/0 (wt B); ruff 0; mypy ok; bandit 0; фронт без изменений | `47bf073` (cherry-pick `856d05f`) | — / — / — | /code-review: grid обходил проверку, int-переполнение → 500 — исправлено |
| S8R-AUDIT-001 | BLOCKER | ✅ | `AnalysisResult(is_safe=True, issues=[])`; `ExecutionResult(success=True, output='S8R_AUDIT_001_CWD=…')`; `DID NOT RAISE` (engine, bt.sys); `assert 202 == 422` (D1-04) | escape 30 + no_legacy 3 + restore/resume + router 404 | «голый datetime в namespace» → граница красная; `kind` из сырого type → red | 2711/19xf/0; ruff 0; mypy ok (179); bandit 0; tsc 0; lint 0; build ok; vitest 926 | `c6bf111` | — | /code-review ×2 (9+3 находки) исправлены; 028 закрыта целиком, 019 частично; gotcha-61 |
| S8R-AUDIT-034 | HIGH | ✅ | `DID NOT RAISE <class 'RuntimeError'>` (18 failed); lifespan без проверки ключа | 39 passed | `reason = None` → 13 failed | 2692/21xf/0; ruff 0; mypy ok; bandit 0; tsc 0; lint 0; build ok; vitest 926 | `b02f784` | — | /code-review не требуется (config/main/scripts) |
| S8R-AUDIT-035 | HIGH | ✅ | `DID NOT RAISE <class 'ValueError'>` ×2; `DID NOT RAISE AuthenticationError`; ws `TypeError … await`; ревизии нет (6 failed) | 6 passed + round-trip | «не инкрементировать версию» → 3 failed | 2698/21xf/0; ruff 0; mypy ok; bandit 0; tsc 0; lint 0; build ok; vitest 926 | `c7c3775` | — | миграция `c5e8b2a7f913` (down `a7b8c9d0e1f2`); /code-review: 2 находки — риск рецепта, принят |
| S8R-AUDIT-036 | HIGH | ✅ | «локал api_key утёк в traceback»; `api_key=SENTINEL…` в выводе (12 failed) | 13 passed | «убрать plain_traceback» → red | 2711/21xf/0; ruff 0; mypy ok; bandit 0; tsc 0; lint 0; build ok; vitest 926 | `2414d59` | — | /code-review: чисто; **gotcha-67** записан (INDEX v25) |
| S8R-AUDIT-007 | HIGH | ✅ | `assert 'failed' == 'pending'` (×2) | 16 passed | «вернуть failed в except» → 3 failed | 2765/16xf/0; ruff 0; mypy ok; bandit 0; tsc/lint/build 0; vitest 925+1 флейк | `a5f5c5b` | — | /code-review чисто |
| S8R-AUDIT-024 | HIGH | ✅ | `18 failed, 1 passed` (`AttributeError: client_order_id`); круги ревью: `17`/`11`/`2 failed` | 46 тестов `test_order_path_unknown_outcome.py` | «без запроса статуса», «без порога», «recovery без лока», «повтор внутри except» → red | 2820/16xf/0; ruff 0; mypy ok; bandit 0; tsc/lint/build 0; фронт не менялся | `9eb05c2` | — | миграция `d4f1a9c2b7e0` (down `c5e8b2a7f913`); /code-review ×3; gotcha-71 |
| S8R-AUDIT-030 | HIGH | ✅ | `E TimeoutError` (9 failed / 1 passed); ревью: `DID NOT RAISE BrokerError` ×2, `__cause__ = None` | 14 + 119 смежных | «без scope», «проглотить таймаут», «return False» → red | 2871/16xf/0; ruff 0; mypy ok; bandit 0; tsc/lint/build 0; фронт не менялся | `c04379b` | — | Q5=10 с; /code-review: 3 находки исправлены; gotcha-74 |
| S8R-AUDIT-074 | HIGH | ✅ | `assert None == 'sb-stop-1'` (broker_order_id потерян); `recovery не прогнан при shutdown … assert []`; ревью: `10.01s`, `3.24s` для 6 сессий | 5 тестов | «без shield», «без commit после отправки» → red | 2885/16xf/0; ruff 0; mypy ok; bandit 0; tsc/lint/build 0; фронт не менялся | `c4965e7` | — | + `stop_grace_period: 60s`; /code-review: 3 находки исправлены; gotcha-75 | | | | | | | |
| S8R-AUDIT-075 | HIGH | ✅ | `ордер в полёте должен дойти до пользователя / assert 0 == 1`; гонка: лишнее «не выполнено»; ревью: `OperationalError` выходит из check_sl_tp; кэш не чистится | 7 тестов | «вернуть таймер», «raise вместо None», «без prune» → red | 2894/16xf/0; ruff 0; mypy ok; bandit 0; фронт не менялся | `698d0e7` | — | /code-review: 2 находки исправлены | | | | | | | |
| S8R-AUDIT-061 | HIGH | ✅ | `assert 'FIGI-B' in ['FIGI-A']` (команда съедена); `unexpected keyword argument 'on_error'`; `singleton не снят`; `release 0 == 1` + 7 RED по кругам ревью | тесты reconnect/ack/терминала/держателей | «общая очередь», «игнорировать ack», «terminal=None», «без перепроверки listener» → red | 2834/16xf/0 (wt B); ruff 0; mypy ok; bandit 0; фронт не менялся | `54e88dc` | — | /code-review ×2 (4+3); gotcha-76 |
| S8R-AUDIT-025 | HIGH | ✅ (+ контрольная правка `fdfbb5c`) | `filled_lots=6`; `P&L … 700.00`; `множитель … 7` | 14 тестов | «volume_lots в P&L», «выход из опроса на partially_filled» → red | 2927/16xf/0; ruff 0; mypy ok; bandit 0; tsc/lint/build 0; vitest 934 | `1bba46b` | — | миграция `e9e5c919fbbf`; /code-review 6 находок исправлены; DEV на Opus |
| S8R-AUDIT-026 | HIGH | ✅ | `assert 1 == 0` (бюджет < лота); CB `размер 6000 > 5000`; `после resume нет уведомления` | test_position_sizing_budget + CB + 2 доказательных | «max(1)» → 7 failed | 2951/13xf/0; ruff 0; mypy ok; bandit 0; tsc/lint/build 0; vitest 937 | `cbd6541` | — | /code-review 4 находки; фронт-предупреждение убрано (S8R-FIX-014) |
| S8R-AUDIT-080 | HIGH | ✅ | `assert 201 == 422`; `DID NOT RAISE ValueError`; live `Decimal('50') == Decimal('98')` (SL из generated_code) | 53 backend + 4 vitest | «снять gt=0», «без проверки диапазона при сохранении» → red | 2998/12xf/0 (wt B); ruff 0; mypy ok (180); bandit 0; tsc/lint/build 0; vitest 940 + флейк | `10c1504` | — | TP ≤ 100 % (решение DEV, принято); /code-review 3 находки |
| S8R-AUDIT-032 | HIGH | ✅ | `assert <LiveTrade …> is None` (13 из 15) | 27 backend | «apply_close без ветки short» → `999900.00 == 1000100`; «снят HOLD-guard»; «гейт CB без exit_trades» → red | 3035/10xf/0; ruff 0; mypy ok (180); bandit 0; tsc/lint/build 0 (итерация 1); vitest — пакет | `c29aa23` | — | Q5=b; /code-review 3 прохода (5 + 6 находок исправлены); DEV на Opus |
| S8R-AUDIT-068 | HIGH | ✅ | `CheckResult(blocked=False, …)` ×2; статус `Decimal('0') == Decimal('12')`; ревью: ложный `Drawdown 9.09%` по устаревшей цене, пик `1000000 == 1100000` | 9 тестов | «paper-only», «без свежести», «без commit», «pnl IS NOT NULL» → red | 2854/14xf/0 (wt B); ruff 0; mypy ok; bandit 0; фронт не менялся | `6fbe378` | — | миграция `f6a2c8e41d93` (down `e9e5c919fbbf`); /code-review: 3 находки исправлены |
| S8R-AUDIT-069 | HIGH | ✅ | `DID NOT RAISE ValidationError` ×4; гонка ×4 | тесты 069/100/гонки 10/10 + фронт 3 | «снять проверку», «без лока», «аудит до лока» → red | 2845/15xf/0 (wt B); ruff 0; mypy ok; bandit 0; tsc/lint/build 0; vitest 937 | `25e95f9` | — | вместе со 100; stopped — вопрос заказчику |
| S8R-AUDIT-100 | HIGH | ✅ в составе 069 (`25e95f9`) | `StaleDataError … 0 were matched` | см. 069 | «без лока и перехвата» → red | см. 069 | `25e95f9` | — | | | | | | | | |
| S8R-AUDIT-055 | HIGH | ✅ | `expected 'http://localhost:8000/api/v1' to be '/api/v1'`; `'ws://localhost:8000' not to contain 'localhost:8000'`; preflight `'CORS_ORIGINS' in ''` | 8 vitest + 14 preflight | «абсолютный дефолт» → red | 2800/16xf/0; ruff 0; mypy ok; bandit 0; tsc 0; lint 0; build ok (0× localhost:8000); vitest 934 | `c93e1ad` | — | dev-proxy Vite добавлен (его не было) | | | | | | | |
| S8R-AUDIT-078 | HIGH | ✅ | `воркеры живы после cancel: pids=[…]`; `4 > лимита 2`; `job осталась в реестре _tasks` | 6 passed ≈ 3 с | «terminate только при отмене» → red | 2806/16xf/0 (wt B); ruff 0; mypy ok; bandit 0; фронт не менялся | `a3bb857` | — | /code-review: 2 находки исправлены; gotcha-73 | | | | | | | |
| S8R-AUDIT-093 | HIGH | ✅ | `AttributeError: … no attribute 'set_calendar_service'`; `assert None is True` (7 failed) | 7 passed | исключение ISS не применяется → `is_trading_day(2024-04-27)` False; без сброса календаря в conftest → 1 failed | 2718/21xf/0 (wt B); ruff 0; mypy ok; bandit 0 | `bca0cec` | — | ⏸→решение оркестратора: источник `engines/stock.json` dailytable; Сб/Вс неторговые (вопрос заказчику); /code-review: изоляция тестов исправлена; gotcha-70 |
| S8R-AUDIT-089 | HIGH | ✅ | `Decimal('100.00') == Decimal('1000.00')`; купон `137.50 == 875.00` | 4 passed | «без множителя» → red | 2722/21xf/0 (wt B); ruff 0; mypy ok; bandit 0; фронт не менялся | `6012089` | — | /code-review чисто |
| S8R-AUDIT-090 | HIGH | ✅ | `assert True is False` (processed); `Decimal('1000.00') == Decimal('0.00')` | 5 passed | «без фильтра даты», «rollback → pass» → red | 2774/16xf/0 (wt B); ruff 0; mypy ok; bandit 0; фронт не менялся | `04fb5cb` | — | /code-review: 2 находки исправлены |
| S8R-AUDIT-091 | HIGH | ✅ | `ImportError: cannot import name 'corporate_action_warning'`; `(1000, Decimal('1.5')) == (10, Decimal('150'))`; `processed True is False` | 9 passed + 68 смежных | «ratio перепутан», «без фильтра opened_at» → red | 2784/16xf/0 (wt B); ruff 0; mypy ok; bandit 0; фронт не менялся | `d138ef1` | — | /code-review: 3 находки исправлены | | | | | | | |
| S8R-AUDIT-092 | HIGH | ✅ | `restore не использует os.replace`; `DID NOT RAISE BackupError` ×2; `--server-port` не распознан; нет `BACKUP_DIR` | tests/test_backup 42 passed | «без файлового лока», «WAL до подмены» → red | 2794/16xf/0 (wt B); ruff 0; mypy ok; bandit 0; фронт не менялся | `464b8cd` | — | /code-review: 4 находки исправлены; gotcha-72, ретро gotcha-19 | | | | | | | |
| S8R-AUDIT-101 | HIGH | ✅ | `AssertionError: вызовы вернули разные записи: 1 и 2`; topup `пополнений: 2, ожидалось 1`; выжившая `{1: False} == {2: True}` | 9 тестов, гонка 10/10 | «без лока и без UNIQUE» → red | 2815/16xf/0 (wt B); ruff 0; mypy ok; bandit 0; фронт не менялся | `2ee8f06` | — | миграция `b8e4d17c9a52` (down переставлен на `d4f1a9c2b7e0`); /code-review: 2 находки исправлены | | | | | | | |
| S8R-AUDIT-099 | HIGH | ✅ | `активная сессия [1] ссылается на удалённый счёт №1: старт=TradingSession, удаление=NoneType` (10/10) | 2 теста, 10/10 подряд | «без лока в delete_account» → red | 2817/16xf/0 (wt B); ruff 0; mypy ok; bandit 0; фронт не менялся | `82b2a2a` | — | развилка DEV: перепроверка счёта в `_create_session_locked` (порядок «удаление первым»); /code-review чисто | | | | | | | |
| S8R-AUDIT-008 | MEDIUM | ✅ | `assert 'active' in {'paused','suspended'}`; `DID NOT RAISE NotFoundError`; `['active'] == ['suspended']` | 14 + 2 backend | «ранний commit active», «временный сбой → paused», «освобождать пару» → red | 3050/10xf/1 (FIX-016); ветка после переноса 013: 3104/8xf/0; ruff 0; mypy ok (180); bandit 0 | `a374468` | — | временный сбой → suspended (развилка рецепта, решение оркестратора); /code-review 3 прохода; хвосты → S8R-FIX-018 |
| S8R-AUDIT-009 | MEDIUM | ✅ | `AssertionError: несверенную сессию подняли`; `TimeoutError` (restore висит) | 11 backend | «без повтора при BrokerTimeoutError» → red | 3115/8xf/0; ветка после переноса 037: 3134/8xf/0; ruff 0; mypy ok (181); bandit 0 | `706c183` | — | молчание брокера (030) → paused, а не старт без сверки (решение оркестратора); /code-review 3 прохода; хвосты → S8R-FIX-019 |
| S8R-AUDIT-010 | MEDIUM | ✅ | `поднято: [] / assert [] == [2]` | 11 backend | «убрать suspended из тира 1», «проигравший в paused до исхода победителя» → red | 3145/8xf/0; ruff 0; mypy ok (181); bandit 0 | `025f929` | — | UI-чеклист S8.30; /code-review 2 прохода |
| S8R-AUDIT-011 | MEDIUM | ✅ | `неполный ответ песочницы принят за расхождение / assert True is False` | 20 backend | «вернуть список», «не сбрасывать счётчик», «любая отсутствующая бумага = неполный» → red | 3166/8xf/0 (2:46); ветка после переноса 038/039: 3250/8xf/0; ruff 0; mypy ok (182); tsc 0 | `abb6e07` | — | развилка ФТ v3.6 п.(2): sandbox-only + подтверждение за 3 прохода (решение оркестратора); /code-review 3 прохода; S8R-FIX-020 |
| S8R-AUDIT-012 | MEDIUM | ✅ | `бар, переживший обрыв, отдан стратегии / assert 1 == 0` | 29 backend | «буферизовать», «не учитывать интервал обрыва», «не дозагружать», «listener игнорирует флаги» → red | 3279/8xf/0; ветка после переноса 040/041: 3378/8xf/0; ruff 0; mypy ok (186) | `e55d0aa` | — | модель — интервал обрыва (решение оркестратора после 2 ревью); S8R-FIX-022 |
| S8R-AUDIT-029 | MEDIUM | ✅ | `после «Стоп» открыто позиций: 1 (entry_price=['301.00000000'])` | 13 backend | «снять лок», «без shield», «однократный перехват отмены» → red | 3391/8xf/0; A после переноса 021: 3407/8xf/0 | `0ec78a5` | — | gotcha-77 `89ded45`; /code-review 3 прохода |
| S8R-AUDIT-076 | MEDIUM | ✅ | `blocked_amount 0.00 ≠ стоимость открытых позиций 9000.00` | 6 backend (5 прогонов без флейка) | «снять лок» по 4 путям, «лок после flush» → red | 3413/8xf/0; A после переноса 058/056: 3454/8xf/0 | `3e27737` | — | CB: commit паузы до закрытий (решение оркестратора); gotcha-78 `a799b52`; /code-review 2 прохода |
| S8R-AUDIT-027 | MEDIUM | ✅ | `percent считается от initial_capital: 100 лотов вместо 50` | 29 backend | «база = initial_capital», «лимит CB от initial_capital», «без поколения», «без резерва», «без освобождения адаптера» → red | 3483/8xf/0; A после переноса 057: 3512/8xf/0 | `47a2ce6` | — | модель «капитал сессии ∧ RUB счёта» — решение оркестратора в рамках Q5=a; /code-review 3 прохода |
| S8R-AUDIT-070 | MEDIUM | ✅ | `вердикты (большая, маленькая) = [False, True]` | 14 backend | «порог от капитала одной сессии», «override на класс» → red | 3526/8xf/0 | `b770a1e` | — | вопрос заказчику: sandbox+real одним классом; /code-review 2 прохода |
| S8R-AUDIT-071 | MEDIUM | ✅ | `CheckResult(blocked=False…)` при лимите 1 000 и лоте 10×300 | 14 backend + vitest | «signal.price для неизвестного режима», «temporary вместо паузы» → red | 3540/8xf/0; vitest 965; A после переноса 015: 3560/7xf/0 | `908bdd5` | `6128b9c52d8a` | гайд §7; /code-review 2 прохода |
| S8R-AUDIT-072 | MEDIUM | ✅ | `assert True is False` (`is_within_trading_hours(2026-11-04 12:00)`) | 435 в смежных наборах | «без календаря», «закрытия блокируются в выходные» → red | 3594/6xf/0 | `40e9c6e` | — | входы/закрытия разведены (решение оркестратора; вопрос заказчика 093 не решён в опасную сторону); /code-review 2 прохода |
| S8R-AUDIT-044 | MEDIUM | ✅ | «сетевой сбой выдан за отказ ключа: API-ключ отклонён T-Invest» | 58 тестов карточки | «исход неизвестен» в тексте ордера → 4 failed | 3671/5xf/0; ruff/mypy/bandit 0 | `63480e8` | — | /code-review 3 прохода (10 + 8 находок); находки S8R-FIX-030, 032 |
| S8R-AUDIT-045 | MEDIUM | ✅ | `TypeError: place_order() got an unexpected keyword argument 'figi'`; `NotFoundBrokerError … TMOS` | 29 тестов | 4 мутации → red | 3725/4xf/0; ruff/mypy/bandit 0 | `6fc6b94` | — | /code-review 3 прохода; выход старых сделок — чек-лист A6 |
| S8R-AUDIT-046 | MEDIUM | ✅ | `assert Decimal('1000000') == Decimal('100000')`; `('total', 12345678.12345679)` float | 10 + 2 + vitest 30 | «available=total», «баланс из кэша с резервом» → red | 4189/3xf/1 (тайминг-флейк под нагрузкой, отдельно зелёный); ruff/mypy/bandit/typecheck/lint/build 0 | `4b30058` | — | /code-review 2 прохода; e2e s5-account testid → balance-in-positions |
| S8R-AUDIT-047 | MEDIUM | ✅ | `DID NOT RAISE BrokerError` (direction=long); `KeyError: 'price'`; `'placed' == 'unknown'` | 55 тестов | «BUY if buy else SELL», «BrokerError вместо OrderNotSentError» → red | 4250/3xf/0; ruff/mypy/bandit 0 | `9914d88` | — | /code-review 2 прохода; поведение для пользователя не меняется (ФТ без правок) |
| S8R-AUDIT-062 | MEDIUM | ✅ | discover `assert [DiscoveredAccount('SB-NEW')] == []`; `accounts_seen 1 == 0`; старый токен в `_singletons` | 17 + vitest 19 | «skip existing», «только свой пользователь» → red | 4267/3xf/0; ruff/mypy/bandit/typecheck/lint/build 0 | `41b7949` | — | /code-review 2 прохода (9 находок) |
| S8R-AUDIT-063 | MEDIUM | ✅ | `datetime(…10:00, utc) == datetime(…07:00, utc)`; ISS-бары [16:00,17:00,18:00] в T-Invest-ряду; `DID NOT RAISE LotSizeUnavailableError` | тесты карточки | 6 мутаций → red | 4385/3xf/0; ruff/mypy/bandit 0; alembic round-trip | `dc6376c` | — | живой GET ISS (MSK); миграция `6f3a063d1c2e`; /code-review 2 прохода; S8R-FIX-041 |
| S8R-AUDIT-006 | MEDIUM | ✅ | grid `E TimeoutError` (48 с); бэктест с `while True` не завершался и за 1500 с | 6 + 1 skip (macOS) | 3 мутации → red | 4392/3xf/1s/0; ruff/mypy/bandit/typecheck/lint/build 0 | `79e1994` | — | gotcha-83 |
| S8R-AUDIT-077 | MEDIUM | ✅ | `AssertionError: (Decimal('7.0623'), Decimal('7.0623'))` (D = W) | 16 тестов | «всегда D» → red | 4408/3xf/0; vitest 1005; ruff/mypy/bandit/typecheck/lint/build 0 | `fc043bb` | — | |
| S8R-AUDIT-013 | MEDIUM | ✅ | `DID NOT RAISE WebSocketDisconnect`; `assert 200 == 401` (login); `['telegram','email'] == []` | 18 + 21 backend | «проверку jti → pass» → 6 red | 3087/8xf/1 (FIX-016, wt B); ruff 0; mypy ok (180); bandit 0; tsc/lint/build 0 | `e2ecb03` | — | разрыв открытых WS ⏸ → S8R-FIX-017; /code-review 3 прохода |
| S8R-AUDIT-037 | MEDIUM | ✅ | `ValueError: Аккаунт заблокирован`; 5× `assert 429 == 200`; п.2 ревью `assert 6 == 1` | 2 + 18 backend | «lockout в refresh», «убрать IP-потолок», «не сбрасывать счётчик» → red | 3107/8xf/0 (wt B); ruff 0; mypy ok (181); bandit 0 | `25143f8` (wt B, перенос в A — позже) | — | 127.0.0.1:80:80, доверие только nginx 172.28.0.10; Docker не запускался; /code-review 3 прохода |
| S8R-AUDIT-038 | MEDIUM | ✅ | `E assert 200 == 403` (8 failed) | 88 CSRF + 22 preflight | «fail-open», «Origin и без cookie», «refresh с double-submit» → red | 3174/8xf/0 (wt B); ruff 0; mypy ok (182); bandit 0; tsc/lint/build 0 | `16daf82` (wt B) | — | Dash-mount POST без X-CSRF-Token — проверить на стенде пакета; /code-review 3 прохода |
| S8R-AUDIT-039 | MEDIUM | ✅ | `assert 401 == 204`; фронт `expected false to be 'unavailable'` | backend + vitest (session/client/aiStream) | «logout требует access», «повтор и на 5xx» → red | 3191/8xf/0 (wt B); ruff 0; mypy ok (182); bandit 0; tsc/lint/build 0; vitest 952 | `1247c46` (wt B) | — | path refresh-cookie /api/v1/auth; E2E auth-hardening — на уровне пакета; /code-review 2 прохода |
| S8R-AUDIT-040 | MEDIUM | ✅ | `AttributeError: … 'COOKIE_SECURE'`; `{'access_token': {False}} != {True}` | 46 + 77 backend | «secure = not DEBUG», «дефолт auto в проде» → red | 3244/8xf/0 (wt B); ruff 0; mypy ok (183); bandit 0; tsc/lint/build 0 | `e914a72` (wt B) | — | fail-closed дефолт (решение оркестратора, см. 33); /code-review 2 прохода |
| S8R-AUDIT-041 | MEDIUM | ✅ | `ImportError: cannot import name 'secrets_rotation'` | 46 backend | «не перешифровывать AI», «is_current → False», «без _weak_secret_reason», «роль при импорте» → red | 3290/8xf/0 (wt B); ruff 0; mypy ok (186); bandit 0 | `5fc4fa9` (wt B) | — | гайд §6а «Ротация секретов»; /code-review 3 прохода |
| S8R-AUDIT-021 | MEDIUM | ✅ | `AssertionError: ✅ Telegram привязан…` (6-я попытка) | 16 backend | «снять лимит», «сброс при успехе» → red | 3306/8xf/0; B после синхронизации с A: 3394/8xf/0 | `b24e1ab` (wt B) | — | /code-review 2 прохода |
| S8R-AUDIT-058 | MEDIUM | ✅ | `DID NOT RAISE WebSocketDisconnect`; `'subscribed' == 'forbidden'`; AI `[200,200,200,200,403]` | backend + vitest | «снять лимит», «close_user вместо close_session», «без проверки отзыва» → red | 3426/8xf/0; vitest 964; B после синхронизации: 3439/8xf/0 | `35c3ba8` (wt B) | — | nginx `^~ /ws` чинит мультиплексор в проде; /code-review 3 прохода |
| S8R-AUDIT-057 | MEDIUM | ✅ | `assert set() == {'content-sec…'}`; `tax_download None == 'no-store'` | backend + vitest | «add_header в location статики» → red | 3477/8xf/0; vitest 964/965 (флейк S8R-FIX-005); B после синхр.: 3483 | `dcbcc12` (wt B) | — | CI nginx-config; /code-review 2 прохода |
| S8R-AUDIT-056 | MEDIUM | ✅ | `_favicon.ico` → location статики (5 failed) | 13 (test_nginx_conf) | «убрать ^~» → red | 3448/8xf/0 (wt B) | `64f114b` (wt B) | — | правка в одну строку; /code-review не требовался (nginx вне списка), smoke после деплоя |
| S8R-AUDIT-015 | MEDIUM | ✅ | `DID NOT RAISE IntegrityError`; `DROP TABLE trading_sessions — FOREIGN KEY constraint failed` | pragmas + delete FK + CB router | «убрать FK=ON», «убрать FK OFF в env», «удалять и running» → red | 3503/7xf/0 (wt B) | `f8bb8e5` (wt B) | — | консервативный 422 по stopped-истории (вопрос заказчика 069); gotcha-79; S8R-FIX-023 |
| S8R-AUDIT-004 | MEDIUM | ✅ | `alembic check → 255: removed table 'user_favorites' …` | 17 (+ процессный alembic check) | «env.py без user_favorites» → red | 3578/6xf/0 (wt B) | `112da39` (wt B) | — | новой ревизии нет; CI «Migrations drift» |
| S8R-AUDIT-003 | MEDIUM | ✅ | `AssertionError: без контекста расшифрован чужой токен: [1]` | 16 backend | «снять фильтр по пользователю», «real принимает sandbox-лот» → red | 3548/7xf/0 (wt B) | `797b2de` (wt B) | — | модель «свой пользователь, prod-first, sandbox-лот не authoritative» — решение оркестратора после ревью; /code-review 3 прохода |
| S8R-AUDIT-014 | MEDIUM | ✅ | `assert [200, 200, 200, 200] == [200, 200, 200, 429]` | 10 backend + vitest | «не инкрементировать при fallback» → red | 3589/6xf/0; vitest 965; B после синхр.: 3623/5xf/0 | `9afbcb4` (wt B) | `1d92db59f28d` | /code-review не обязателен (ai/), — в ревью пакета |
| S8R-AUDIT-020 | MEDIUM | ✅ | `TimeoutError` ожидания события новой сессии | 2 backend + 5 vitest | «убрать publish» → red | 3625/5xf/0; vitest 969/970 (флейк S8R-FIX-005) | `e25b8ef` (wt B) | — | /code-review не выполнен (пауза) — в ревью пакета |
| S8R-AUDIT-028 | MEDIUM | ✅ попутно с 001 (`c6bf111`): legacy-путь удалён | | | | | | | |
| S8R-AUDIT-054 | MEDIUM | ✅ | `assert 200 == 503` (health при disconnected БД); `/health/live` 404 | 15 backend + 11 vitest | `ready=True` → 4 failed | 3631/4xf/0; ruff/mypy/bandit/typecheck/lint/build 0 | `774bbb0` (wt B) | — | регресс HealthWidget при 503 найден DEV, исправлен в карточке; находка S8R-FIX-031 |
| S8R-AUDIT-051 | MEDIUM | ✅ | «настройки config.py не описаны в docs/env_vars.md: [AI_ALLOW_PRIVATE_PROVIDER_URLS, …]» | 13 тестов | новое поле Settings → red | 3644/4xf/0; ruff/mypy/bandit/typecheck/lint/build 0 | `63edfb0` (wt B) | — | `.env.example` не трогали; drain 10 с (gotcha-80); находка S8R-FIX-033 |
| S8R-AUDIT-053 | MEDIUM | ✅ | «обход прошёл анализатор: s = getattr(datetime, 'sys')» (удалён getattr) | 292 теста | exit/vars/pickle/__spec__/allow-list +sys → red | 3995/4xf/0 (разовый SIGABRT preflight[dash], S8R-FIX-024); ruff/mypy/bandit/typecheck/lint/build 0 | `f173824` (wt B) | — | только тесты; находка S8R-FIX-034 |
| S8R-AUDIT-065 | MEDIUM | ✅ | `DID NOT RAISE ValidationError` (×36); xlsx `assert 'f' == 's'` | 125 тестов + vitest | «без префикса ' в CSV» → 11 failed | 4120/3xf/0; vitest 974; ruff/mypy/bandit/typecheck/lint/build 0 | `c7ab8b4` (wt B) | — | gotcha-81; «_» в формате (CNYRUB_TOM); находка S8R-FIX-035 |
| S8R-AUDIT-066 | MEDIUM | ✅ | `'&lt;b&gt;x…' in '✅ Telegram привязан к аккаунту <b><b>x</b> & <a href=…>'` (6 из 7) | 7 тестов | «_safe_format_event_text без escape» → red | 4127/3xf/0; ruff/mypy/bandit/typecheck/lint/build 0 | `6703261` (wt B) | — | п.5 рецепта (экранировать подписи кнопок) не применён — решение оркестратора по Bot API (text кнопки не HTML); `P&L` с голым `&` в шаблонах /positions — Telegram принимает, не правилось |
| S8R-AUDIT-097 | MEDIUM | ✅ | `AssertionError: AsyncAdaptedQueuePool`; `OperationalError: database is locked [SQL: COMMIT]` | тесты карточки | «tax без повтора», «повтор без отката», «без processed» → red | 4143/3xf/0; ruff/mypy/bandit 0; бенчмарки N без деградации (заглушки без БД), прямой замер +0,65 мс/сессия | `3f85fac` (wt B) | — | NullPool (вложенные сессии); /code-review 2 прохода; S8R-FIX-036; ⚠️ E2E пакета — с новым пулом |
| S8R-AUDIT-094 | MEDIUM | ✅ | `misfire_grace_time вне [600, 3600] с: {refresh…: 1, detect…: 1, send_daily_stats: 1, sync_moex_calendar: 1}` | 2 теста | 2 мутации → red | 4514/3xf/1s/0; ruff/mypy/bandit/typecheck/lint/build 0 | `c9a5496` | — | |
| S8R-AUDIT-096 | MEDIUM | ✅ | `AssertionError: лоты прежнего отчёта года обязаны удаляться / assert 4 == 2`; `TaxLot has no attribute report_id` | 4 + round-trip | «не удалять прежние лоты» → red | зелёный, кроме 7 флейков test_f2 (UTC-дата, исправлено `deb9db8`); alembic check чист | `0da5c0a` | — | |
| S8R-AUDIT-095 | MEDIUM | ✅ | `{'MYFUND': 'share'} != {'MYFUND': 'etf'}`; `Decimal('100000.00') == Decimal('60000.00')` | 8 тестов | «раздельная база» → red | 4527/3xf/0; ruff/mypy/bandit/typecheck/lint/build 0 | `7c653c3` | — | Q6-095=a; конфликт с FIX-025 при сведении (market_data/service) разрешён оркестратором; S8R-FIX-043 |
| S8R-AUDIT-081 | MEDIUM | ✅ | `assert 200 == 409`; `DID NOT RAISE StrategyStatusConflictError`; `'tested' == 'live'` | тесты карточки + vitest 84 | «refusal=None», «без promote_draft», «без защиты parking» → red | 4213/3xf/1 (preflight[sh] под нагрузкой, S8R-FIX-024); alembic round-trip ok | `81aa3f7` (wt B) | — | решения заказчика 2026-09-30; миграция `2c7bd0443aa6`; ⚠️ E2E с запуском draft-стратегий → 409 |
| S8R-AUDIT-082 | MEDIUM | ✅ | vitest `expected "vi.fn()" to be called with…` (тишина при 422); pytest `AssertionError: []` | 5 парсер + 5 хук + store/CodePanel | 3 мутации → red | 4223/3xf/0; vitest 999; ruff/mypy/bandit/typecheck/lint/build 0 | `def57dd` (wt B) | — | S8R-FIX-038 |
| S8R-AUDIT-073 | MEDIUM | ✅ | `assert Decimal('0.00') == Decimal('-60000.00')`; `'circuit_breaker.triggered' != 'cb.triggered'` | 13 тестов | «trigger_value=0», «commit→flush», «без перечитывания» → red | 4237/3xf/0; ruff/mypy/bandit 0 | `1322329` (wt B) | — | /code-review 2 прохода; S8R-FIX-039 |
| S8R-AUDIT-083 | MEDIUM | ✅ | `RuntimeError: smtp exploded`; `'in_app' == 'in_app,telegram'`; `'warning' == 'error'` | 23 теста | «return_exceptions=False» → red | 4388/3xf/0; ruff/mypy/bandit/typecheck/lint/build 0 | `bc50928` (wt B) | — | gotcha-82; S8R-FIX-040 |
| S8R-AUDIT-084 | MEDIUM | ✅ | `assert 400 == 409`; `assert 400 == 200`; `assert 503 == 429` | 3 + переписанные | «extra=ignore» → red | 4393/3xf/0; vitest 1004; ruff/mypy/bandit/typecheck/lint/build 0 | `f25a772` (wt B) | — | CLAUDE.md кода: категория лимитера notifications_test — добавить в инварианты при сведении |
| S8R-AUDIT-085 | MEDIUM | ✅ | vitest: telegram_enabled включён без привязки; стор не откатывает; «Очистить все» без вопроса; backend `'sent' == 'failed'` | vitest + 3 backend | «готовность по тесту», «retry=True» → red | 4396/3xf/0; vitest 1012; ruff/mypy/bandit/typecheck/lint/build 0 | `d4222dc` (wt B) | — | тот же DEV, что 084 |
| S8R-AUDIT-086 | MEDIUM | ✅ | `KeyError: 'auth.login'`; `cb.triggered опубликован до записи pending_events` | 15 + р.2 | 3 мутации → red | 4419/3xf/0; ruff/mypy/bandit 0 | `2f84469` (wt B) | — | /code-review 2 прохода |
| S8R-AUDIT-087 | MEDIUM | ✅ | `ImportError: validate_ohlcv`; `no attribute consecutive_broker_rejections`; `assert 5 == 8` (остаток через 5 с) | 16 + 23 + 14 | 4 мутации → red | 4494/3xf/0; ruff/mypy/bandit 0; alembic round-trip | `f0243da` (wt B) | — | Q6: (а)/(д)/(е) — ФТ; /code-review 3 прохода; S8R-FIX-042 |
| S8R-AUDIT-005 | LOW | ✅ | `ValueError: No such index: 'idx_ai_user'` | test_audit_s8r_alembic 3 | «безусловный drop_index» → red | 4571/1s/2xf/0; ruff/mypy/bandit/typecheck/lint/build 0 | `6843a0a` | — | ветка s8r/fix-low (стек на MEDIUM), worktree wt-s8r-fixes-c |
| S8R-AUDIT-016 | LOW | ✅ | `чужой id → 403 {Нет доступа…}; несуществующий → 404` | 8 параметров | «проверка после выборки» → red | 4579/2xf/0; ruff/mypy/bandit/typecheck/lint/build 0 | `b334a35` | — | |
| S8R-AUDIT-017 | LOW | ✅ | `assert 422 == 202` (grid без strategy_id); прочее закрыто в 015 | 4 теста | «strategy_id из тела» → red | 4583/2xf/0; ruff/mypy/bandit/typecheck/lint/build 0 | `0ba87cb` | — | |
| S8R-AUDIT-018 | LOW | ✅ | `DID NOT RAISE InvalidProviderURLError` (CGNAT); тело ответа в verify; `201 in (403, 422)` | 21 тест | 3 мутации → red | 4604/1s/2xf/0; ruff/mypy/bandit/typecheck/lint/build 0 | `bde605d` | — | gotcha-84; S8R-FIX-046 |
| S8R-AUDIT-019 | LOW | ✅ | `app.strategy.code_generator вернулся`; сводка по чужой стратегии (22 failed) | guard 21 + 1 | 2 мутации → red | 4562/1s/1xf/0 (тестов меньше — удалены вместе с кодом), coverage 91 %; ruff/mypy/bandit/typecheck/lint/build 0 | `4f31507` | — | S8R-FIX-047 |
| S8R-AUDIT-022 | LOW | ✅ | RED не получен — угрозу закрыла 096 | 2 теста | «общее имя файла» → red | 4564/1s/1xf/0 | `55a964a` | — | только тест |
| S8R-AUDIT-023 | LOW | ✅ | `assert 200 == 403`; категория `general` | 2 + 38 | «get_current_user» → red | 4566/1s/1xf/0 | `e68870e` | — | Ticker вместо regex рецепта; `..` в Ticker — закрыть в 098 |
| S8R-AUDIT-031 | LOW | ✅ | filled без цены (`Decimal 0`); pending → closed; 1.005 → 1.00 | 10 + 4576 | «только is None», «без ветки cancelled» → red | 4576/1s/1xf/0 | `a28c2a9` | 2 | новый `app/common/money.py`; P&L «слепой» позиции → S8R-FIX-048 |
| S8R-AUDIT-042 | LOW | ✅ | `TypeError … 'context'`; `'sk-a...WXYZ' == '…WXYZ'`; `(16877 & 63) == 0`; токен в выводе telegram.ext/uvicorn.access | 48 + 50 | AAD header-only, по record.msg, широкий regex, без поэлементных args, без _create_private_file → red | 4627/1xf/0 | `9efb76e` | 3 | развилка AAD — п.38; CLI rebind; гайд §6а, чеклист A7 |
| S8R-AUDIT-043 | LOW | ✅ | `assert 0 == 1` (verify не вызван); 423; нет `cleanup_revoked_tokens`; `200 == 403`; `assert 1 == 4` | 5 файлов pytest + 6 vitest | без dummy-verify, без гейта, read-modify-write, отказ → 401 → red | 4658/1xf/0; vitest 1058 | `842854f` | 3 | ревью 086 п.7 («сбой журнала → 401») заменён на единый 503; resume — тоже под гейтом |
| S8R-AUDIT-048 | LOW | ✅ | нет `TInvestService`; naive vs aware `TypeError`; `'EXECUTED' == 'UNKNOWN'`; два sandbox-счёта; `['reopened','reopened']` | 78 (test_rate_limiter_registry + reopen) | кэш выкл., вся песочница в общую квоту, эпоха — время, account_id из объекта, сосед без сверки, потолок на все → red | 4759/1xf/0 | `5325063` | 3 | 059 перенесена cherry-pick → `a78e0c3`; пропуск сверки при ошибке чтения портфеля — подтверждено оркестратором |
| S8R-AUDIT-064 | LOW | ✅ | `httpx.ConnectError`; пагинация без cursor (5001 запрос); переподключение из-за битой свечи; `503 == 200` | 3 файла pytest + vitest стора | узкий except маппера, без _stopping, без проброса, 503 при кэше → red | 4887/1s/1xf/0; vitest 1067 | `f73085f` | 2+1 | parse_candles — уже 063, удаление счёта — уже 099 |
| S8R-AUDIT-049 | LOW | ✅ | «нет permissions»; actions не по SHA; нет dependabot.yml; группа push без run_id | test_ci_workflows 15 | убрать permissions, группа по ref → red | 5116/0 (р.1) | `29413f8`* + docs `d0abe74` | 1+1 | *detached A, перенос после 050; команды удаления — заказчику (раздел «Команды заказчику») |
| S8R-AUDIT-050 | LOW | ✅ | SDK закреплён тегом (10 failed); строки без `--hash` (9 failed) | test_deps_pinning 23 + CI 15 | вернуть тег, убрать greenlet, снять хэши → red | 5143/1s/0; vitest 1136; чистый venv --require-hashes — 5143 | `1fb3ad7`, `626b6fe` | 1+1 | хэши — из JSON API PyPI (pip-tools за прокси не справился); docker build — заказчик |
| S8R-AUDIT-052 | LOW | ✅ | чеклист: 300 `- [ ]` при «Итого 299»; README: «старт Sprint 9», плейсхолдер, `tsc --noEmit` | — (документация) | — | — | docs | — | выполнено оркестратором (владелец — документация); расхождение счётчика — S8.17 (9 пунктов, в таблице 8) |
| S8R-AUDIT-059 | LOW | ✅ | docs-пути не None; `DID NOT RAISE RuntimeError`; расширенные поля анониму; `200 == 401` | test_health 22+, test_main 11, test_config | расширенные анониму, без решения по подписи, без перехвата SQLAlchemyError → red | 4787/1xf/0 | `d62ad0d`* | 2+1 | *detached A, cherry-pick в s8r/fix-low после 048; решения — п.48 |
| S8R-AUDIT-060 | LOW | ✅ | `'http 403' == 'close 4401'` (18, реальный uvicorn); `(4401, '') == (4401, 'logout')`; 4 канала → 4 POST | test_ws_authz 18+15, nginx 39, vitest контроллера/хуков/баннера | close до accept, без reason, без общего refresh, logout без ожидания, без эпохи → red | 4835/1xf/0; vitest 1108 | `1e45465`* | 4 | *detached A, cherry-pick после 067; отступление — п.51 |
| S8R-AUDIT-098 | LOW | ✅ | `invalid choice 'revoke_admin'`; лишние 201; `('..', 200)`; `'garbage1'` принят | 6 файлов + vitest | без is_active, счёт отдельно, без Timeframe, str(exc) → red | 4907/1xf/0 (A) | `1e5e402`, `34add1c` | 2+3 | cherry-pick с конфликтами в ISS (064) — разрешены через iss_path |
| S8R-AUDIT-067 | LOW | ✅ | `assert 200 == 422` ×15; `DID NOT RAISE ValidationError`; «Достигнут лимит сделок за день: 0/0»; `assert 11 == 1` | готовый тест плана (xfail снят) + 3 файла | без ge=1, без нейтрализации, без отката override, без наследования часов → red | 4974/1s/0; vitest 1071 | `50ac6b8` | 3 | 060 перенесена cherry-pick → `60e32da` |
| S8R-AUDIT-079 | LOW | ✅ | `unexpected keyword 'max_points'`; экстремум выпадал; `assert 1335 == 2000` | 12 | равномерная сетка без экстремумов, extras=set() → red | 5033/0 | `df6f3e5` | 1+1 | старые кривые не пересчитываются (ФТ §4.5) |
| S8R-AUDIT-088 | LOW | ✅ | ТЗ §4: 37 несуществующих путей, 106 неописанных; §3: 13 таблиц расходятся | — (документация) | — | — | docs | — | выполнено оркестратором: реестр маршрутов выгружен из приложения (ветка s8r/fix-low), схема — из `Base.metadata` |

## Пакеты
| Пакет | Карточек ✅/⏸ | E2E | /code-review | Три оси | S-сценарии | PR | Мерж |
|---|---|---|---|---|---|---|---|
| BLOCKER | 3/0 из 3 | 171/1/0 (9,0 мин) | ✅ по карточкам (001 — два прохода) | треб. 0 / рецепт 0 / качество 1 low → `9f676aa` | — | [#28](https://github.com/SergoPipo/moex-terminal/pull/28), CI ✅ (backend 2711/19xf = локально; frontend, security-scan) | смёржен `15bb244` (2026-09-30) |
| HIGH | 25/0 из 25 (+ 028 в BLOCKER) | 172/0/0 (5,8 мин), свой стенд | ✅ по карточкам (024, 025, 032 — 3 прохода) | треб. 0 / рецепт 1 low (докстринг 080 → `3417a70`) / качество 0; гонки 10× по 46 — зелёные; pytest 3035/10xf/0, покрытие 88,46 %; vitest 941 | S-1/S-2/S-7 ✅ 2026-10-01 (на коде MEDIUM) | [#29](https://github.com/SergoPipo/moex-terminal/pull/29) (база `s8r/fix-blocker`, стек) | смёржен `c18dd92` (2026-10-01) |
| MEDIUM | 51/0 из 51 (+ S8R-FIX-025…028) | **172/0/0** (7,6 мин, свой стенд из A `ffc2a42`, чистая БД) | ✅ по карточкам (1–3 прохода) | в работе | — | — | — |
| LOW | 0/0 из 21 | | | | — | | |
| S8R-FIX (находки цикла) | 38/1⏸ (030–069 + 005; 050 ⏸; вне пакета 052, 058) | **175/0** (10,1 мин, свой стенд C) | ✅ по пачкам (торговля/брокер — до 5 проходов) | две группы: треб. 0+0 / рецепт 0+4 low / качество 7 low + 2 medium 8 low → REV-FBE/FFE | — | [#44](https://github.com/SergoPipo/moex-terminal/pull/44), CI ✅ после REV-CI3 | ✅ 2026-10-06 `48f6773`, CI develop ✅ |

### Гейт пакета MEDIUM (A `ffc2a42`, 2026-10-01, тихая машина)
pytest **4570 passed / 1 skipped / 3 xfailed / 0 failed** (315 с), coverage **91 %**; ruff 0; mypy Success (196); bandit M0/H0; alembic heads `7fe0fae4293c` (1); typecheck 0; lint 0; build ok; vitest **1017/1017** (143 файла). Миграции пакета: `1d92db59f28d` (014) → `2c7bd0443aa6` (081) → `6f3a063d1c2e` (063) → `bbb9a0908e89` (FIX-027) → `c7d2e5a19f84` (087) → `7fe0fae4293c` (096). Stack Gotchas 80–83, INDEX v40.

## Живой счёт #3
**2026-10-01, 15:14–15:18 МСК, код MEDIUM `ffc2a42` (включает HIGH), стенд из `wt-s8r-live` на копии рабочей БД (`Connection.backup`), в копии остановлены все сессии и выключены счета 2 и 4; корень НУЦ Минцифры — только процессу стенда (`GRPC_DEFAULT_SSL_ROOTS_FILE_PATH`), симлинк `.env` 15:14:19–15:16:21 и 15:16:26–15:18:46, снят.**
- Исходное состояние счёта #3: 99 994,072795 ₽, бумаг нет.
- Паритет: бэктест #59 версии 90 (SBER, D, 01.07–30.09.2026) — completed, `parity_signals_match=1` (исполнен в дочернем процессе — 006).
- **S-1:** сессия 7 (SBER, 1m, sandbox, 1 лот, «Тестовая» v90) → статус стратегии `live` (081) → вход по свече 15:16: ордер `ecf2a668-128c-454e-b2fc-ac7b10c1d913` (клиентский ключ `54d690f1-…`), buy 1 лот (лот песочницы — 1 шт.) по 275,35, FIGI `BBG004730N88`, комиссия 0,14 ₽ (`broker`); у брокера SBER 1 шт. (`source=strategy`), кэш 99 718,58512 ₽. «Стоп» 15:18:22 → встречный ордер, выход 275,51, P&L +0,16 ₽, комиссия итого 0,28 ₽; статус стратегии → `tested` (081).
- **S-2:** штатная остановка с открытой позицией → сессия `suspended`; рестарт → `restore_all restored=1 mismatched=0`, сессия `active`, сделка одна; 90 с после рестарта — второй позиции нет, расхождений нет.
- **S-7:** FIGI резолвится токеном счёта сессии (`figi_synced` → подписка стрима по тому же FIGI — 003/045); `70001` за прогон не встречался (повторов не понадобилось).
- Итог: счёт #3 в исходном состоянии (бумаг нет), кэш 99 993,957365 ₽ (−0,115 ₽ = P&L − комиссия). Ошибок в логе нет; одно предупреждение `instrument_cache_persist_failed … database is locked` во время бэктеста (best-effort запись кэша справочника) — S8R-FIX-045.
- PR #29 смёржен после сценариев: `c18dd92`.

## Решения, принятые самостоятельно
1. Worktree вынесены за пределы docs-репо (`../wt-s8r-fixes` от `Develop/` попадал внутрь `Test/`) — `git worktree move` в `/Users/sergopipo/Documents/Claude_Code/`.
2. Параллельный DEV (Q8=b) работает во втором worktree на detached HEAD; его коммит переносится `cherry-pick` в ветку пакета — лишних веток не создаётся (`branch -D` заблокирована).
3. S8R-AUDIT-100 перенесена в пакет HIGH к 069: рецепт требует «в одном коммите с 069».
4. Карточки 067/079/088 отсутствуют в строке порядка LOW (`prompt_s8r_audit_fixes.md` §4), но есть в таблице — поставлены в конец пакета LOW.
5. S8R-AUDIT-080 — в HIGH сразу после 026 (таблица §2: «сразу после 026 — общая схема риск-параметров»), хотя в строке порядка `prompt_s8r_fixes.md` §3 её нет.

6. Ветка `s8r/fix-high` ведётся **стеком от `s8r/fix-blocker`** (не от `develop` после мержа): Q1=d «всё подряд» несовместимо с ожиданием мержа по команде (Q3=a). Работа по HIGH (034) идёт параллельно с 001 во втором worktree; PR `s8r/fix-high` → `develop` будет показывать и коммиты BLOCKER до его мержа.
7. Футер коммитов: оба соавтора — `Claude Fable 5.1` (DEV-субагенты, как в промпте) и `Claude Opus 5.5` (оркестратор: приёмка, правки по /code-review, интеграция).
9. 034: порог длины `SECRET_KEY` — 32 байт по рецепту (ТЗ §7.3 обещал 64 символа, в коде это не проверялось никогда); ТЗ приведено к рецепту. ⚠️ Риск для стенда заказчика: при `DEBUG=false` и слабом ключе в `.env` backend теперь не стартует — `.env` исполнитель не читает, проверка за заказчиком.
8. Вне рецепта 002 по находке /code-review: Grid Search теперь вызывает `_validate_strategy_for_backtest` (раньше обходил проверку блоков).

10. /code-review 001, находка «риск-монитор live берёт SL/TP из сохранённого `generated_code`» (`trading/engine.py:~2295`) — дефект существовал до 001; передана в карточку S8R-AUDIT-080 (единая точка `RiskParams` для всех путей), в 001 не чинится.
11. /code-review 001: эндпоинт `POST /api/v1/sandbox/execute` (исполнение произвольного кода, UI не использует) снимается с регистрации — прямое следствие решения Q4-001=(a) «исполняется только код из IR».

12. Номера Stack Gotchas для подтверждённых кандидатов аудита берутся из резерва audit §6 (61–69), а не «следующий свободный» из `stack_gotchas/README.md`: промпт и аудит ссылаются на эти номера. Неподтверждённые номера останутся пропусками — перечень в итоговом отчёте.

13. 093: `get_trading_calendar` бил в `/iss/engines/stock/markets/shares/dates.json` — там нет календаря (DEV ⏸, проверено живым GET к ISS). Источник заменён на `/iss/engines/stock.json` `dailytable` (исключения `is_work_day`) — техническая часть рецепта. **Политика выходных не меняется** (Сб/Вс — неторговые): `timetable` ISS помечает все 7 дней рабочими из-за торгов выходного дня с 2025 — **вопрос заказчику** (ФТ §1.6/1.7, карточка 072).

14. Vitest в карточке снимается только при изменении `frontend/`; иначе — один раз на уровне пакета на тихой машине. Причина: два DEV параллельно гоняют полный vitest при load 18–25 → 1–8 таймаутов (S8R-FIX-005). typecheck/lint/build — по-прежнему в каждой карточке.
15. Находка DEV-007 «входной ордер с NOT_FOUND у брокера остаётся pending бессрочно» передана в карточку 024 (поиск ордера по клиентскому ключу, подтверждённое отсутствие → failed).

16. /code-review 091: отсечка `process_split` по `opened_at` (была записана как находка S8R-FIX-008 от DEV-090) чинится в 091 — эта карточка впервые делает путь сплита рабочим, без отсечки позиции пересчитывались бы повторно.
17. /code-review 024, п. 6: сделка, не ушедшая брокеру, при удалённом/деактивированном счёте → `failed` (брокерской правды больше не будет); при временной недоступности адаптера — остаётся `pending`.

18. Передаётся в карточку 025: движок при частичном исполнении встречного ордера закрывает сделку целиком без `_flag_partial_exit` (runtime-recovery флаг ставит) — учёт частичного закрытия как такового не реализован (контрольный /code-review 024).
19. 024: «ответ потерян» ≠ «ордера нет» подтверждается только спустя `UNKNOWN_OUTCOME_CONFIRM_AGE_SEC = 120 с` от последней отправки; детерминированный gRPC-отказ (INVALID_ARGUMENT и т. п.) = «не принят» сразу (иначе «недостаточно средств» занимал бы слот); активный частично исполненный ордер входа ждёт терминала, остаток отменяется recovery через 30 мин — решения DEV, приняты оркестратором.

20. 24–25.09: DEV-024 и DEV-078 дважды обрывались на лимите сессии модели Fable (HTTP 429); возобновлены с сохранённым контекстом, потерь работы нет.

21. 074: в `docker-compose.yml` добавлен `stop_grace_period: 60s` для backend — без него Docker убивает процесс через 10 с, и исправленный graceful shutdown (45 с) не успевает; Docker локально не запускался (как и раньше).

22. 069: удаление стратегии при живых сессиях → 409; судьба ОСТАНОВЛЕННЫХ сессий при удалении стратегии (запрет или отвязка истории через SET NULL + PRAGMA foreign_keys, карточка 015) — **вопрос заказчику**, текущее поведение не меняется.

23. 25.09: исчерпан лимит модели Fable для субагентов (HTTP 429 «You've reached your Fable limit»). Дальше DEV-субагенты — на **Opus 5.5**; футер коммитов таких карточек — только `Claude Opus 5.5`. Ревьюеры по-прежнему sonnet.

24. 080: предел тейк-профита 100 % вместо 500 % из рецепта — прежний предел редактора; для short (Q5-032=b) TP выше 100 % даёт цену ≤ 0. Принято.

25. 032 (Q5=b): лог HOLD «сигнал против направления без позиции» — DEBUG, а не warning из рецепта: warning был для варианта «только long» (`short_not_supported`), при short-варианте выход без позиции — штатное состояние после каждого закрытия. Флаг CB `block_shorts` (есть только в API, в UI нет) оставлен без изменения схемы: при значении по умолчанию `True` он запретил бы все short, что противоречит Q5=b; проверка стала защитным инвариантом — **вопрос заказчику**: удалить флаг или сделать его запретом short для real-режима.

26. Пакет HIGH, отступления от процедуры: (а) три оси ревью запущены одновременно (процедура — ≤ 2), ревьюеры сами дробили работу на под-агентов — нагрузка на машину в пределах, тесты под ревью не флейкали; (б) ось качества — новый ревьюер вместо продолжения прежнего (контекст прежнего устарел после 20+ карточек); с MEDIUM — продолжать ревьюера качества HIGH через SendMessage; (в) PR #29 открыт в `s8r/fix-blocker`, а не в `develop`: ветка стоит стеком на неcмёрженном #28, PR в develop показал бы и BLOCKER — после мержа #28 базу переключить.

27. CI PR #29 красный (DEBUG=false, прогон в 00:15 МСК): S8R-FIX-016 и 2 теста гонок 101 (sandbox без `DEBUG=True`). Блокирует гейт «CI зелёный» → исправлено `03398cf` во временном worktree `wt-s8r-fixes-c` на `s8r/fix-high` (A и B заняты DEV); в `s8r/fix-medium` — cherry-pick после DEV-008. Локальный прогон в режиме CI (фиктивные ключи из `ci.yml` в окружении) отклонён правилами разрешений — режим DEBUG=false проверяет только CI; впредь сверять CI до объявления пакета закрытым.

28. 008: развилка рецепта «статус при сбое start()» — временный сбой → `suspended` (пара сохраняется, повтор при следующем старте), постоянная причина (`NotFoundError`) → `paused`; внутри restore `suspended` считается торговавшей. MEDIUM идёт двумя потоками: A (`s8r/fix-medium`) — рантайм/CB, B (detached) — auth/периметр; коммиты B переносятся cherry-pick в A.

29. Паттерн: 008, 009, 013, 037 — по три раунда /code-review, находки одних классов (сбой записи в БД и инвариант active⇔listener, лишние/потерянные уведомления и их severity, тест в обход реального пути `_unary`, таймаут поверх commit, ресурсы от клиента без предела). Системная мера: в шаблон DEV (`dev_base.md`) добавлена обязательная «Самопроверка по типовым находкам код-ревью» (6 пунктов) — действует с 010/038. Сообщено заказчику в отчёте.
30. 037: nginx опубликован на `127.0.0.1:80:80` (cloudflared на хосте ходит в localhost по гайду §5.3; LAN-доступа гайд не предусматривает) — иначе клиент LAN подделывает `CF-Connecting-IP`. В гайде `service: http://127.0.0.1:80`.

31. Инцидент DEV-009: первая версия двух новых тестов не подменяла SDK-клиента для `start()` — возможна попытка gRPC к T-Invest с фиктивным токеном `fake_key` (не токен заказчика, ордеров нет); тест упал по страховке 1,3 с, исправлено в той же карточке. Правило: тесты restore/start глушат SDK-клиента.

32. 011: рецепт «неполный портфель → без паузы» отменял ФТ v3.6 п.(2) (закрытие позиции у брокера днём не ловилось бы, а у real-счёта — никогда). Решение оркестратора: только sandbox; сигнатура сбоя — «только деньги» (gotcha-56); фон подтверждает расхождение после 3 подряд неполных проходов (~45 мин); real — без повторов. ФТ v3.6 п.(2) сохранён с задержкой для песочницы.
33. 040: дефолт `COOKIE_SECURE=auto` ослаблял прод (Secure зависел от цепочки прокси) — решение оркестратора: без настройки прод всегда Secure, `auto` — только при DEBUG=true или явно.

34. 012: модель давности бара — по интервалу обрыва стрима, а не по часам (отсечка по часам ломала торговлю последнего бара каждого дня) и не по голому номеру поколения (штатные RST_STREAM теряли сигналы). Свечи публикуются сразу, бар переопубликуется после дозагрузки.
35. Перенос коммитов между потоками: B (auth) → A cherry-pick; конфликты `config.py` (011 ↔ 040 ↔ 041) — «обе стороны», гейт A после переноса 3378/8xf/0. Дальше B синхронизируется с вершиной A перед каждой новой карточкой.

36. S8R-FIX-025 (торги выходного дня, решение заказчика «по расписанию ISS»): у ISS нет признака «инструмент торгуется в выходные» (проверено живым GET 2026-09-30), окно торгов выходного дня по свечам SBER 26–27.09 — 10:00–19:00 MSK. Решение оркестратора: расписание (дни, часы) — ISS, допуск конкретной бумаги — флаг T-Invest `weekend_flag` (торговля и так только через T-Invest); нет флага — вход в выходной запрещён.
37. S8R-FIX-026: «скрывается из списка» (решение заказчика) — фильтр «Все» и окно запуска без архивных; первоначальная формулировка задания оркестратора («оставь как есть») была неверной — исправлено до коммита.
38. 042 (AAD, рецепт «AAD = broker_account:{id}» при «формат шифртекста не трогать»): формат (MAGIC+key_id, IV) не меняется; к AAD новых записей добавляется контекст «назначение:id владельца» (нет id — user_id + назначение); decrypt сначала с контекстом, при InvalidTag — прежний путь. Fallback безопасен: новая запись без своего контекста не расшифровывается ни в какой другой строке; старые записи привязки не получают до ротации (CLI ротации перешифровывает с контекстом). Реестры токенов — ключ-отпечаток вместо plaintext.
39. Ревью MEDIUM, ось «требование»: первый ревьюер пропал при перезапуске сессии оркестратора после групп 1–2 (18 карточек: OK, одна low → S8R-FIX-044); 2026-10-01 запущен второй на оставшиеся 33 карточки + S8R-FIX-027. Итог второго: 34 карточки — OK, находок 0 (ревьюер разделил работу на 2 под-агента — сверх лимита «≤ 2 одновременно» вместе с DEV-042, нагрузка в пределах). **Ось «требование» MEDIUM закрыта: 0 открытых находок.** Ось «качество» запущена 2026-10-01 (≤ 1 под-агента, без полного pytest).
40. Ревью MEDIUM, ось «качество» (2026-10-01): 54 позиции — OK, кроме: (high) `_extend_partial_entry` — на real при оценке первой части к ней прибавлялась кумулятивная фактическая комиссия ордера (задвоение) → факт заменяет оценку; (low) комментарий `csrf.py` расходился с кодом → исправить комментарий; (low) дубль паттерна «записать статус, при сбое suspended» в runtime ×3 — уже S8R-FIX-037. Правки high/low — DEV-REV-MQ в wt A до PR MEDIUM; после — повторная ось качества по diff правки.
41. Правка high из п.40, раунд 2 (код-ревью): правило «факт заменяет оценку, ноль на real — факт» отменено — маппер T-Invest отдаёт `executed_commission = 0`, пока удержание не начислено (0 ≠ факт), и прежняя правка затирала оценку нулём с источником broker. Новое единое правило комиссии входа (один helper для всех писателей): cumulative > 0 — факт по ордеру (max с записанным фактом, sandbox и real одинаково); 0/None — оценка по тарифу на весь оборот входа, broker-факт не уменьшается. 042 код-ревью: 10 находок (пропущенный decrypt в diag-скрипте, маскирование bot-token на итоговой строке и в stdlib-логгерах, chmod только своего каталога, файлы бэкапа 0600 с момента создания, перепривязка старых записей через CLI, обязательный account_id при записи и др.) — отданы DEV-042.
42. Правка high, раунд 3 (второй проход код-ревью, 9 находок): единое правило «0 = не начислено» само давало ложную оценку на тарифе без комиссии при отсутствии позднего перечитывания факта. Решение оркестратора: в PR MEDIUM — только устранение задвоения при довыполнении; первый fill — как в ffc2a42 (табличный тест); семантика нуля и перечитывание факта (вход и выход) — S8R-FIX-049 (medium).
43. Правка high закрыта: `f7fe66d` (s8r/fix-medium), код-ревью 3 прохода + итерация 4 (ветка «нет цены → пропорциональный объём», введённая оркестратором в раунде 3, оказалась ошибочной — маппер отдаёт цену 0, а не None; убрана). Таблица первого fill зелёная на ffc2a42. Гейт DEV: 4609/1s/3xf/0. Слияние develop `1e5b1ba` (FIX-029) в s8r/fix-medium — конфликт только импортов `main.py`, обе стороны сохранены.
44. PR #31 (MEDIUM): CI backend красный — `test_stop_session_race.py::test_repeated_cancel_like_anyio_still_stops` TimeoutError (4615 passed / 1 failed); тот же тест падал на `ca1d0bc` и `f7fe66d`, на `ffc2a42` прошёл — нестабилен, не связан с правкой комиссии. Причина (оркестратор): `_wait_through_cancels` под повторными отменами anyio вращает цикл на каждом тике; под coverage в CI защищённая запись `stopped` идёт 12 с > `fail_after(10)`. В проде — сжигание CPU до grace 155 с при разрыве клиента во время «Стоп». Исправление — ожидание в `anyio.CancelScope(shield=True)` (DEV-REV-CI); PR #31 не мёржится до зелёного CI.
45. Отступление оркестратора: ad-hoc промпты DEV-REV-MQ и DEV-REV-CI собраны из dev_042.md, и п.1 «Задач» (тест-нота) остался от 042; оба исполнителя работали по тексту своей карточки (вреда нет). Исправлено: `s8r-evidence/fixes/tools/adhoc.py` подменяет и п.1, и разделы карточки/решений. DEV-REV-CI р.1: вращение 3482 → 1 вызов `asyncio.wait`; код-ревью — механизм верен, раунд 2: один экран на весь цикл, `uncancel()` проглоченных нативных отмен, тесты без «пустого прохода».
46. DEV-REV-CI закрыта `756b48c`: один экран anyio на весь цикл ожидания. Пункт оркестратора «uncancel() проглоченных нативных отмен» снят по возражению исполнителя (проверено моделью): uncancel превращал чужую отмену под `asyncio.timeout` в TimeoutError — каждая перехваченная отмена настоящая и пробрасывается после записи; сценарии A/B/C закреплены тестом. Гейт 4621/1s/3xf/0; под coverage ×3 зелёный. CI PR #31 перезапущен.
47. **Пакет MEDIUM смёржен:** PR #31 → develop `7b2eaf3` (2026-10-01), CI PR 4622 passed / 3 xfailed / 0 failed, coverage-гейт пройден. CI на develop `7b2eaf3` — success. DEV-043 (LOW), раунд 4: атомарный счётчик неудач (параллельные попытки обходили блокировку — найдено вторым проходом код-ревью), гейт дисклеймера и на resume (решение оркестратора: resume — действие пользователя; restore — нет).
48. LOW: develop `7b2eaf3` влит в `s8r/fix-low` → `413fe47` (конфликты: `_extend_partial_entry` — взят helper REV-MQ, переведён на `quantize_money`; INDEX gotcha обе v41 → v42); гейт 4709/1s/1xf/0, vitest 1058. 052 выполнена оркестратором (документация). Параллельность: 048 — в C (брокерский слой); 059 — в A (detached `413fe47`, перенос cherry-pick'ом); 064 пересекается с 048 (multiplexer, broker/service) — строго после 048. 059 (решение оркестратора): аноним на `/health` — только status с прежним HTTP-кодом (healthcheck), расширенные поля — при валидной аутентификации; http-origin не-localhost в проде — warning, `*` — ошибка старта.
49. 048 — два решения оркестратора пересмотрены после 3-го прохода код-ревью: (а) «UNCHANGED с другим счётом → без сверки позиций» (раунд 3) неверно — ручное переоткрытие (кнопка, баланс) пишет новый счёт без сверки, фантомы старого счёта оставались бы открытыми; теперь сверка во всех ветках, идемпотентная по фактическим позициям текущего счёта; (б) единый потолок ожидания квоты 3 с превращал троттлинг в отказы (история, префетч, SL в песочнице) — потолок 10 с только для отправки ордеров, чтения без потолка, `read_worst_case_sec` обратно 38 с. Флейки: полный pytest DEV зависал дважды (043 ~40 %, 059 ~60 %) при параллельном чужом pytest → S8R-FIX-050; тест 043 «429 с Retry-After HTTP-датой» зависит от текущего времени → исправить в следующей карточке ветки C.
50. 2026-10-02: 049 (docs-часть) — `d0abe74`: `.gitignore` (s8r-evidence/, корневые *.png/*.crt, prompt_s8r_*), три prompt_s8r_* сняты с отслеживания (локально остаются), отчёты DEV 031/042/043/048/059/REV-CI/REV-MQ закоммичены (раньше пропускались). Код-часть 049 (workflows permissions/concurrency, Dependabot) — после переноса 060 (оба меняют ci.yml). Убраны свои временные worktree `wt-s8r-fixes-b` и `wt-s8r-live` (с `live_copy.db`). DEV-060 оборвался на раунде 3 (истёк OAuth) — возобновлён. Основной чекаут `Develop` у заказчика на старом develop `eb769cc` (origin — `7b2eaf3`) — не трогается, сообщить в отчёте.
51. Отступление: 060 — 4 прохода код-ревью (процедура — 2–3) и 5 раундов правок: контроллер переподключения WS + HTTP refresh оказался глубже рецепта (logout/refresh/reuse-detection, эпоха сессии, несколько каналов). Каждый проход давал реальные сценарии «выход на всех устройствах»; после раунда 5 — коммит без нового прохода, при регрессиях — карточка в пакет S8R-FIX. 067: фронт-часть неприменима (формы CB нет → S8R-FIX-055); раунд 2 — нейтрализация сохранённых значений вне диапазона в движке + `invalid_fields` в GET (решение оркестратора: старые бессмысленные лимиты не применяются, а не блокируют торговлю). 088 и 052 — оркестратором (документация).
52. 098 — решение оркестратора пересмотрено после 2-го прохода: лимит таймфреймов 20 + последовательный DELETE→POST во фронте породили гонки (тоггл туда-обратно, порядок, рассинхрон сторов) ради недостижимого из UI предела (8 значений Timeframe) → валидация значения по enum `Timeframe` на сервере, прежняя логика фронта, ошибки избранного раздельно по видам. FIX-054: при сбое опроса брокера баланс отдаёт `null`, а не 0 (иначе карточки показывают «0 ₽» вместо «—»). Флейк vitest `StrategyEditPageDelete` (таймаут 5 с под параллельной нагрузкой) — повторяется, см. S8R-FIX-005.
53. 098 — расширение «FIX-056: регистр как пришёл» (решение оркестратора) откатано после ревью: без канонизации тикера оно открывало обход проверки «один инструмент — одна сессия» на живом счёте (sber vs SBER) и десяток расхождений сравнений; S8R-FIX-056 переписана под системную канонизацию (пакет S8R-FIX). 079 — корзины min/max искажали ось времени графика → гибрид «равномерная сетка + экстремумы».
54. Пакет LOW — сдача (2026-10-02): все 20 карточек кода + 052/088 (docs) + S8R-FIX-054 закрыты; вершина `s8r/fix-low` = `626b6fe` (запушено). E2E на своём стенде (wt-s8r-fixes-c, чистая БД): **173 passed / 0 failed** (5,4 мин); стенд погашен. Ревью: ось «требование» — 20 карточек OK, находок 0; ось «рецепт» — 20 OK, 0 находок; ось «качество» — 1 medium (048: ручное переоткрытие sandbox-счёта без сверки позиций) → правка S8R-REV-LQ; код-ревью правки р.1 нашёл гонку (сверка по снимку портфеля до локов могла закрыть сделку, только что исполненную на новом счёте) → р.2: единая функция «переоткрыть + сверить», кандидат — только вход до момента переоткрытия (проверка под локом сделки), сервисные пути — фоновой задачей с повторами и паузой/пометкой при неудаче. PR LOW — после р.2.
55. REV-LQ (находка оси «качество» LOW): 3 прохода код-ревью, 4 раунда правок — механика «переоткрыть + сверить» оказалась глубже исходной находки (гонка со сделками нового счёта, выходы в полёте, единая обработка неудачи, фоновые задачи, дубли задач на счёт, порядок shutdown). Отступление: р.2 написан до RED (исполнитель снял RED задним числом на базе); р.3–4 — в порядке TDD. После р.4 — коммит без нового прохода; остаток, если будет, — карточкой S8R-FIX.
56. PR #32 (LOW) открыт 2026-10-02; первый CI — backend 1 failed (`test_retry_settings_clamped_not_fatal`, caplog пуст): alembic `fileConfig` в тестах миграций выключал существующие логгеры (`disable_existing_loggers=True` по умолчанию) — порядок-зависимый флейк, локально зелёный. Корень исправлен в `alembic/env.py` (S8R-REV-CI2), точечные обходы в тестах убраны; кандидат в Stack Gotchas — «caplog пуст только в полном прогоне». Повторный CI — зелёный (5171/0); **PR #32 смёржен → develop `2d460cf`**, CI develop — success; Dependabot отработал по всем 4 экосистемам (pyproject с git-SDK разбирается) и открыл 10 PR (#34–#43, среди них мажорные protobuf 4→7, openai 2→3, cryptography 46→50, vitest 4→5) — не мёржатся в цикле, решение заказчика.
57. Пакет S8R-FIX (2026-10-02): ветка `s8r/fix-findings` от develop `2d460cf` (worktree C), A — detached для параллели (cherry-pick). Порядок: medium 051, 056, 055, 049 → low по областям (торговля 048/041/042/037/039; брокер 030/032/053/045; бэктест 035/036/057; настройки/AI/health 031/033/034/046; фронт 038; налоги 043; WS 044; тесты 050); 052 — ждёт решения заказчика. Запущены 051 (C) и 056 (A).
58. S8R-FIX: закрыты 051 (`9dc3d50`, C), 056 (`7bbf291`+`0d3def4`, A), 055 (`4b538ac`, A); 049 — раунд 2 (раздельные колонки комиссии входа/выхода, миграция — корень находки ревью). Отступление: low-карточки — ПАЧКАМИ по области (один DEV, один коммит на пачку с перечнем карточек, RED/GREEN/мутация на каждую карточку) — 21 мелкая карточка, накладные расходы «карточка = DEV» несоразмерны. Пачка «бэктест» (035, 036, 057) — в A. Находка DEV-055: ФТ §12.4 cooldown «в параметрах стратегии» и в конфиге CB — какой действует, уточнить в ФТ.
59. S8R-FIX: 049 закрыта (`1a99d33`); в `s8r/fix-findings` перенесены 056 (`2fbdf08` — конфликт с 051 в мониторе/роутере алертов: монитор — версия 051, роутер — канонический тикер 056 + resolve_alert_figi 051; `581bf4f`) и 055 (`897e1b4`); цепочка миграций сведена `145d125` (7fe0fae4293c → fb26c08f99c8 → c04dc4f891a2; round-trip теста индекса — откат до 7fe0fae4293c). Пачка «бэктест» — р.2 в A. Находка: тест CB зависит от времени суток → S8R-FIX-060 (medium — флейк CI), до PR.

60. Пакет S8R-FIX — сдача (2026-10-06): ветка `s8r/fix-findings` (вершина `46f0f67`) → PR #44 → develop `48f6773`, CI develop ✅. Финальный гейт на `0f1e004`: pytest 5631/0, vitest 1281/0, ruff/mypy/bandit/typecheck/lint/build — 0, head `752ae89f7ec2`. Первый CI PR красный по двум причинам вне пакета → REV-CI3 (`46f0f67`): тест `test_old_env_with_dead_ai_limits_still_starts` зависел от DEBUG раннера (окружение важнее .env) — тест убирает DEBUG; Safety — новая CVE-2026-81726 в nltk ≤3.10.3, nltk — зависимость самого сканера safety (не приложения), исправленной версии нет → исключение в `safety_policy.yml` до 2026-12-31. Решения оркестратора за пакет: 045 закрыта через корень 064; фоновые/короткие busy_timeout-обходы откачены; 063 — источник stages[].price + проверка правдоподобия ответа PostOrder ±20 % (gotcha-33), консервативная оценка для CB — вопрос заказчику; 065 — outbox у всех издателей, `stop_grace_period` 90 с; 069 — «Дублировать» архивной разрешено (копия черновик); 055 — блок состояния CB над формой оставлен; воркеры vitest не ограничиваем. Ошибки оркестратора, исправленные ревью: дозапрос типа в горячем пути `ensure_figi_lookup` (043 р.1 п.5), удаление SELECT в `_entry_without_price` (TR р.2 п.9). Worktree A/B/C убраны.

## Новые находки (заведены в backlog, не чинились)
- S8R-FIX-024 — флейки полного прогона: SIGABRT fork+gRPC в preflight-тестах, разовое зависание гейта (medium).
- S8R-FIX-023 — хвосты 015: тесты без FK (162 падения при включении), audit_log SET NULL vs append-only (low).
- S8R-FIX-022 — хвосты 012: график не исправляет не-последний бар, бары в обрыве не публикуются (low).
- S8R-FIX-021 — хвосты 041: AAD без привязки к строке, alembic-путь от cwd, живой сервер по двум адресам (low).
- S8R-FIX-020 — фоновая сверка счетов последовательно, держит watchdog (medium, не регресс).
- S8R-FIX-019 — хвосты 009: `start()` в сети до yield, фоновая сверка на wait_for поверх записей (low).
- S8R-FIX-018 — хвосты 008: shutdown-доучёт частичного выхода без паузы, осиротевшая paused держит пару, три копии кода уведомлений (low).
- S8R-FIX-017 — хвосты 013: открытые WS не рвутся при logout, «Аккаунт деактивирован» не виден на экране входа, дубли кода аутентификации (low).
- S8R-FIX-016 — ✅ `03398cf` — `test_same_commission_and_net_pnl` падает после 23:50 МСК (реальное время vs проверка торговых часов закрытия; low; ревью HIGH).
- S8R-FIX-015 — направление сделки: inline-копии `in ("buy","long")` в 8 местах; paper-выручка по `volume_lots`; paper-просадка без unrealized (low; DEV-032).
- S8R-FIX-014 — нет эндпоинта корректного лота для формы запуска (low).
- S8R-FIX-013 — CB: пик только на входах; `DailyStat.peak_equity` мёртвая; дневной лимит без проверки свежести цены (low).
- S8R-FIX-012 — recovery частичного выхода: P&L на всю позицию при активном partially_filled; события после снятия слушателя теряются (low).
- S8R-FIX-011 — фронт не повторяет REST-подписку графика после reconnect WS; ключ стрима без токена (low).
- S8R-FIX-010 — лок `sandbox_recovery_user` вне «Порядка захвата»; `find_instrument` в `market_data` мимо дедлайна адаптера (low).
- S8R-FIX-009 — `shutdown()` job-менеджера без таймаута; grid в ожидании слотов без статуса (low).
- S8R-FIX-008 — корп. действия: «дата отсечки» = дата реестра в уведомлении; сплит без отсечки; расписание джобы ≠ ФТ (low).
- S8R-FIX-007 — UI не подписывает `LiveTrade.status='pending'` (low).
- S8R-FIX-006 — `nkd_entry/nkd_exit`: писателя нет, tax и корп. действия трактуют по-разному (low).
- S8R-FIX-005 — vitest `StrategyEditPageDelete` таймаутит в полном прогоне под нагрузкой машины (low).
- S8R-FIX-004 — Grid Search: имена параметров Bollinger `bollinger_*` vs `bb_*` (low; /code-review 001, было до фикса).
- S8R-FIX-003 — ТЗ §8.4: JSON-логи с ротацией не существуют; `dev.log` без ротации и с правами 644 (low).
- S8R-FIX-002 — `test_users_cli.py` вакуумен: CLI без ключей падает на импорте настроек (low; найдено DEV-034).
- S8R-FIX-001 — `SOURCE=volume`: live-интерпретатор считает по close, backtrader по volume (low; найдено DEV-002).

## Команды заказчику (S8R-AUDIT-049, собрано 2026-10-02; не выполнялись исполнителем)

Репо кода (`Develop/`), локальные ветки, слитые в `origin/develop` (кроме develop/main/s8r/audit):
```bash
cd ~/Documents/Claude_Code/Test/Develop
git branch -d docs/claude-md-stack-gotchas feat/chart-drawings-rev2 fix/fe-backtest-ui fix/fe-charts fix/fe-core-refactor fix/fe-network fix/fe-security fix/fe-ui-misc fix/s5r2-track4-relogin-401 p1/auth-hardening p1/be-trad-06 p1/wave2-backend p1/wave3-frontend pr10 s1/arch-fixes s1/auth s1/backend-init s1/db-models s1/frontend s1/infra s2/broker-settings s2/moex-iss s3/owner-review s5r/chart-hardening-and-closeout s6r/code-fixes s7/sprint-7 s8/sprint-8 s8r/fix-blocker s8r/fix-high s8r/fix-medium
git push origin --delete feat/chart-drawings-rev2 fix/s5r2-track4-relogin-401 p1/auth-hardening p1/be-trad-06 p1/wave2-backend p1/wave3-frontend s3/owner-review s5r/chart-hardening-and-closeout s5r/ci-cleanup s5r/closeout s5r/closeout-wave3 s5r/e2e-s4-fix s5r/live-runtime-loop s5r/real-positions s7/sprint-7 s8/sprint-8 s8r/acceptance-fixes-2026-07-26 s8r/backlog-cleanup-2026-08 s8r/backlog-fixes-2026-07-27 s8r/bug-23-interpreter s8r/bug-31-unified-codegen s8r/ci-green-2026-08 s8r/fix-blocker s8r/fix-high s8r/fix-medium s8r/fix-tls-root s8r/sandbox-account-reopen s8r/sandbox-review-fixes s8r/tail-fixes s8r/tax-and-commission-tail
git branch -f main origin/main
git stash list   # stash@{0} — страховка правки .env.example (2026-07-29); если правка проверена: git stash drop stash@{0}
git branch -D s8r/fix-tls-root   # локальная, смёржена (#30); -D — т.к. upstream удалён после push --delete
```
Пакет S8R-FIX смёржен (#44, 2026-10-06) — ветки пакетов больше не нужны (worktree A/B/C исполнитель убрал сам):
```bash
cd ~/Documents/Claude_Code/Test/Develop
git fetch origin && git checkout develop && git pull --ff-only   # стенд на старом develop eb769cc — обновить, когда удобно (Vite/uvicorn перезапустятся)
git branch -d s8r/fix-findings s8r/fix-low
git push origin --delete s8r/fix-findings s8r/fix-low
```

Репо документации (`Test/`):
```bash
cd ~/Documents/Claude_Code/Test
git log --oneline -1 worktree-agent-a8dcfbad   # fa6cf3d «docs(faq): инструкция миграции на VPS» — не в main
# если коммит не нужен:
git worktree remove .claude/worktrees/agent-a8dcfbad && git branch -D worktree-agent-a8dcfbad
# если нужен — cherry-pick в docs/backlog-006-strategy-builder, затем те же две команды
git push origin docs/s5r2-skeleton-track4   # ветка опережает remote на 1 — или удалить, если не нужна
```
После мержа PR LOW: вкладка GitHub → Insights → Dependency graph → Dependabot — проверить, что `backend/pyproject.toml` с git-зависимостью SDK разбирается без ошибки.

## Следующий шаг
**2026-10-19 (11:15 МСК) — ордерные сценарии M1–M9 на стенде-копии выполнены; чеклист 357 / 13 / 0; находка S8R-FIX-139.**

Решения заказчика (утро 19.10): `override_parity` — как 14.10; доступ для отложенных задач включит заказчик; рабочий
стенд — обновить вечером, после сценариев. Копия переведена на `develop` `70848d8`. Подробно — backlog «Цикл
2026-10-19», журнал `s8r-evidence/obs/stand/mscen.log`.

- ✅ 410, 427, 431 (M1: продажа вне терминала → пауза на ближайшей фоновой сверке, сделка открыта), 432 (после
  «Возобновить» — снова пауза с уведомлением), 411 (уведомление о том же расхождении — один раз на перезапусках),
  448 (M8), 450 (M9), 414/415/418/433 (M4/M5: ответ на закрывающий ордер задержан своим прокси дольше дедлайна —
  «исход неизвестен», после перезапуска сделка закрыта по факту брокера, статистика без задвоения, ложной паузы нет),
  346 (M6), 173 (Telegram, обработчики бота на копии), 77 (письмо — с `DEV_MODE=false`).
- ❌ 413 (M2) → **S8R-FIX-139** (medium): при сверке позиции берутся без справочника инструментов, бумага тикера
  сессии, купленная вручную, — `external` и молча пропускается; «возможно, куплено вручную» недостижимо. Решение
  заказчика: вариант (а) — FIGI тикеров сверяемых сессий из кэша резолва, или (б) — переписать пункт.
- Не проводились: M7 (381 — нужна утренняя сессия 07:00–10:00 МСК и узкий SL/TP), 382/417/493 (нужны отказ
  автозакрытия, код `70001`, резолв FIGI во время обрыва).
- **Ошибка исполнителя:** после «Закрыть все» сессии #19/#20 не остановлены — повторный вход (sandbox `a5e890fb…`
  buy SBER, закрыт `7950be26…`) и **4 письма вместо одного**. Почта на копии выключена, `DEV_MODE=true` вернул.
- **Брокеру ушло:** 13 ордеров по 1 лоту (список — backlog); на #3 только рубли 99 987 ₽.
- `broker_ops.py`: строка счёта — из `BROKER_OPS_DB` (по умолчанию рабочая БД); `obsstand.sh`: `OBS_DEV_MODE`,
  `OBS_EMAIL`, `OBS_GRPC_PROXY`, `OBS_WORKTREE`, `backend-stop`; `faultproxy.py`: режимы `trace`,
  `hold_resp_new:K:D`.
- Дальше: решение по FIX-139; обновление рабочего стенда до `70848d8` (вечером); наблюдения (доступ для задач).

**2026-10-19 (10:10 МСК) — субботняя позиция SBER закрыта в понедельник; FIX-050 ✅; отложенные задачи не работают (доступ организации).**

- **`s8r-weekend-close` (Сб 17.10 17:40 по Mac) не выполнилась:** запуск упал на старте — «Your organization has disabled
  Claude subscription access for Claude Code · Use an Anthropic API key instead, or ask your admin to enable access». Та же
  ошибка у `s8r-obs-daily` 18.10 (17.10 — успешно). Напоминания этой сессии (17:48 / 18:07 / 18:13) в субботу не
  сработали и пришли пачкой в понедельник. **Ежедневные наблюдения стенда-копии сейчас не идут** — нужен доступ
  (решение/настройка заказчика) либо замена (напоминание в живой сессии). Сам стенд жив (backend/frontend 200).
- **SBER #14 закрыта по плану «если окно пропущено — в понедельник до сценариев»:** брокеру ушёл **sell SBER 1 лот**
  на sandbox #3, ордер `093b1a6e-35d5-4fa2-b340-351e6eb82032`, исполнен 10:04:18 МСК по 287,30 (вход 287,56, Сб 11:52
  МСК, ордер `ac4c446d…`); сделка #59 «Закрыта», P&L −0,26 ₽ до комиссий, комиссия брокера 0,28 ₽ (0,14 + 0,14), чистый
  −0,54 ₽; уведомления «Сделка закрыта», «Все позиции закрыты», «Сессия остановлена»; сессия #14 `stopped`, стратегия #4 →
  `tested`. `connection_lost` в субботнем окне — только вызванные прокси-разрывателем (#2910, #2917), самопроизвольных нет.
- **«Выходной» (стр. 572, вторая половина)** не проверен — окно пропущено; следующая возможность — Сб 24.10 после 19:00 МСК
  или Вс до 10:00 МСК. Пункт остаётся `[~]`.
- **S8R-FIX-050 ✅:** контрольный прогон 17.10 13:28–13:48 — 3 раунда × 2 параллельных полных pytest (`develop` `70848d8`),
  6 из 6 — 6087 passed / 1 skipped, сторож и faulthandler не сработали; worktree `f050-a/b` убраны.
- **Ветки** FIX-134, FIX-135/136, FIX-137/138 (локальные и удалённые) удалены заказчиком.
- Чеклист **343 [x] / 27 [~] / 0 [ ]**. Дальше — ордерные сценарии M1–M9 на копии (план — запись 2026-10-16).

**2026-10-17 (14:20 МСК) — PR #76 смержен (FIX-137/138 ✅), `develop` `70848d8`; чеклист 343 / 27 / 0.**

- PR #76 смержен по команде заказчика по локальным гейтам (merge-коммит) → `develop` `70848d8`, дерево = ветка
  `cc26dbe`; S8.62 п.3/п.4 `[x]`; worktree `fix-137-138` убран. Ветку `s8r/fix-137-138-broker-status` (локальную и
  удалённую) удаляет заказчик. Рабочий стенд — `d48c6f3`, не обновлялся: в `develop` после него только PR #74–#76
  (тест, косметика, статус брокера); обновление стенда — по команде заказчика (перезапуск backend сбрасывает
  6-часовой таймер детекта корп. действий).
- Ветки FIX-134 (локальная и удалённая) и локальная FIX-135/136 удалены заказчиком.

**2026-10-17 (13:50 МСК) — PR #75 смержен; FIX-137/138 (вариант «а») — PR #76, проверено вживую; ФТ 4.7 / ТЗ 3.7; чеклист 341 / 29 / 0.**

- **PR #75** (FIX-135/136) смержен по команде заказчика → `develop` `7d9bbbe` (дерево = ветка `20fc977`); worktree
  `fix-135-136` убран; ветку `s8r/fix-135-136-ui-format` удаляет заказчик. Рабочий стенд не обновлялся.
- **FIX-137** — `/health.tinvest_connected` учитывает `connection_lost_reported` мультиплексора; **FIX-138** — тело
  `connection_lost` из `stream_loss_reason` («поток данных прерван, переподключение — автоматически [UNAVAILABLE]»),
  repr — только в логах; сбой разбора не роняет цикл переподключения (находка `/code-review`). Ветка
  `s8r/fix-137-138-broker-status` (`cc26dbe`) → **PR #76**; worktree `Develop/.claude/worktrees/fix-137-138`.
  Тесты: 8 новых, RED → GREEN, 2 мутации красные; гейты — 6087 passed / 1 skipped в обоих окружениях, покрытие
  92,31 %, ruff/mypy/bandit 0 (фронт не менялся).
- **Вживую на копии** (12:33–12:36 по Mac; backend из ветки фикса через прокси, `OBS_WORKTREE` — новая переменная
  `obsstand.sh`): `tinvest_connected` true → false на 31-й с обрыва (уведомление #2917 с новым текстом) → true
  после `connection_restored` (#2918); подвал «Брокер: настроен», виджет T-Invest OFFLINE, «До закрытия: 5ч 26м».
  Копия возвращена на код `s8r-obs` без прокси (12:36:08, обе сессии восстановлены); прокси остановлен.
- **ФТ 4.7 / ТЗ 3.7** — статус брокера при подтверждённом обрыве, текст `connection_lost`, `tinvest_connected`,
  косметика 135/136. **UI-чеклист** — раздел S8.62 (4 пункта), итог 368 новых пунктов.
- Мерж PR #76 — по команде заказчика.

**2026-10-17 (13:10 МСК) — решения заказчика по остаткам; обрыв связи на копии (стр. 574, 434 `[x]`); PR #74 смержен; косметика 135/136 в ветке.**

Решения заказчика (все — по рекомендациям, подробно — backlog «Цикл 2026-10-17»): PR #74 мержить;
обрыв связи на копии сегодня; FIX-011 — правило ТЗ; FIX-017 — закрыть без миграции; в работу —
стр. 344, косметика «6h»/«−0 ₽», письмо с копии в пн, контрольный прогон FIX-050. Гейт
`Develop/CLAUDE.md` не менять, деплой не делать — без изменений.

- **PR #74 смержен** (merge-коммит, 11:39 по Mac) → `develop` `875597e`; дерево совпадает с
  проверенной веткой `6485f00` (`9f32e5a`) → S8R-FIX-134 ✅, стр. 250 `[x]`. Рабочий стенд не
  обновлялся (правка только в тесте). Worktree `fix-134` убран; локальную и удалённую ветку
  `s8r/fix-134-sandbox-test-debug` удаляет заказчик.
- **FIX-011 ✅** — правило «один токен на процесс» уже записано в ТЗ §5.4.5, ТЗ не менялось.
  **FIX-017 ✅** — в копии рабочей БД от 16.10 одна привязка Telegram, дублей нет, новые гасит
  привязка (S8R-AUDIT-013).
- **Стр. 344** переписана по факту (песочница возвращает удержание) → `[x]`. ФТ/ТЗ этой
  оговорки не содержат.
- **Стр. 572** — в окне ✅ (12:31 МСК: «ТОРГИ ВЫХОДНОГО ДНЯ», «До закрытия: 6h 28m», API
  `weekend_session`); «Выходной» — проверка в 18:07 по Mac.
- **Обрыв связи на копии** — `s8r-evidence/obs/faultproxy.py` (HTTP CONNECT только к хостам
  T-Invest, 127.0.0.1:18911), backend копии через `grpc_proxy` (`OBS_GRPC_PROXY=… obsstand.sh
  backend`, новая `obsstand.sh backend-stop`). Фаза A (`cut` 75 с, 11:42:52–11:44:08 по Mac):
  `connection_lost` 12:43:24 МСК (32 с), `connection_restored` 12:44:54 → **стр. 574 `[x]`**.
  Фаза B (`blackhole_new` 11:55:01–12:01:32, `phase_b.py`, лог `stand/phase_b.log`): фоновая
  сверка 12:57:12 МСК — 6×10 с → «брокер не ответил за 60 с», сессия #14 активна; API 8–41 мс,
  свечи по WS без перерыва, `price_alert` (алерт создан в окне) через 2 с → **стр. 434 `[x]`**.
  Не берутся обрывом: 417 (только код `70001`), 493 (нет резолва FIGI в окне), 382/418 (нет
  ордеров). Backend копии возвращён без прокси в 12:02:58, обе сессии восстановлены; прокси
  остановлен. Ордеров брокеру не уходило.
- **Новые находки:** S8R-FIX-137 (подвал «Брокер: на связи» во время подтверждённого обрыва) и
  S8R-FIX-138 (сырой repr gRPC в тексте `connection_lost`) — ждут решения заказчика.
- **S8R-FIX-135/136** (ветка `s8r/fix-135-136-ui-format` от `875597e`, worktree
  `Develop/.claude/worktrees/fix-135-136`): «6ч 28м» вместо «6h 28m»; `roundForDisplay` — без
  «−0 ₽»/«−0.0%» на главной. RED → GREEN, мутация красная, `/code-review` — 6 находок (1
  исправлена, 5 — без правки). Гейты: pytest `DEBUG=true` 6079 passed / 1 skipped, окружение CI
  6079 passed / 1 skipped (покрытие 92,32 %), ruff/mypy/bandit 0, typecheck/build/lint 0, vitest 1487;
  скриншоты главной до/после (`s8r-evidence/obs/ui/S8.46-1-fix136-*.png`). Коммит `20fc977` → **PR #75**
  (CI красный из-за квоты Actions — джобы не стартуют); мерж — по команде заказчика.
- **Почта с копии (пн, стр. 77):** `OBS_EMAIL=1 obsstand.sh backend` не обнуляет SMTP; перед
  сценариями — email только для `all_positions_closed` у пользователя 1 (сейчас email выключен у
  всех событий), письмо придёт на «Закрыть все» в конце сценариев. Если SMTP в `.env` не задан —
  в журнале будет отказ отправки (файл `.env` не читается).
- Чеклист **339 [x] / 27 [~] / 0 [ ]** (обе копии одинаковы).

**2026-10-17 (12:00 МСК) — субботнее окно: sandbox SBER вошёл в 11:52 МСК; S8.42 п.2 закрыт; чеклист 335 / 31 / 0.**

- Коммиты по команде заказчика: FIX-134 `6485f00` → **PR #74** в develop (мерж — по команде, CI стоит);
  docs `48e757d`; `main` сведён заново `a179e11` (`--no-ff`, дерево = docs-ветка). Гейт
  `Develop/CLAUDE.md` не менялся (решение заказчика).
- Отложенная задача 08:35 сессию не создала — ошибка в `s8r-evidence/obs/weekend.py` (постраничный
  ответ `/trading/sessions` перебирался как список); исправлено. Повторный старт заблокирован сверкой
  движков (`parity_signals_diverged`, 12 расхождений, v103 SBER 1m) → по решению заказчика старт с
  `override_parity` (`weekend.py start --override-parity`; аудит #117/#118), как 14.10 с #7.
- **Сессия #14** sandbox SBER 1m на #3: сделка #59 buy 1 лот по 287,56, 11:52 МСК, ордер
  `ac4c446d-2e15-4543-9047-80acf6b6d1de` `filled`, комиссия 0,14 ₽ (брокер); `weekend_flag=True` →
  S8.46 п.2 — первая половина ✅ (вторая вживую не воспроизводится, тесты), пункт оставлен `[~]`.
  Стратегия #4 → `live`, бейдж «Боевая торговля» → **S8.42 п.2 `[x]`**.
- **Брокеру ушло:** buy SBER 1 лот (`ac4c446d…`). Закрытие (sell 1 лот) и стоп #14 — задача
  `s8r-weekend-close` в 17:40 по Mac (18:40 МСК). Paper-шорт LKOH #13 открыт 16.10 в 21:00 МСК:
  sell 1 лот по 6 045,50.

**2026-10-16 (20:30) — цикл «после мержа»: свод docs в `main`, перепроверка 14 пунктов, находка S8R-FIX-134, наблюдения поставлены.**

Решения заказчика (начало цикла): 1 — перепроверить «после мержа» и поправить учёт; 2 — ордерные
сценарии на **стенде-копии**, **пн 19.10 с 10:00 МСК**, параметры как 14.10 (GAZP и SBER по 1 лоту
рыночными, sandbox #3, сделки «вне терминала» — `broker_ops.py`); 3 — наблюдения: paper-шорт до
отсечки + sandbox SBER на субботнее окно, проверять в нужное время и отписываться; 4 — деплой не
делать, SMTP раньше работал (перепроверка — только если нужно), docs свести в `main`, закрыть без
мержа что можно, квоту CI ждать.

- **Docs:** `main` ← docs-ветка `--no-ff` (`33f2ef4`, дерево совпадает); PR #1 (30.04) закрыт без
  мержа — 158 строк уже в ветке. В `backlog.md` ✅ у 008/018/021 (остаток закрыт 13.10) и 100.
- **Стенд-копия** `s8r-evidence/obs/obsstand.sh reset|up|backend|down|status`: worktree
  `Develop/.claude/worktrees/s8r-obs` (`d48c6f3`), backend :8111, фронт :5284, копия рабочей БД
  через backup API (`mode=ro`), тестовый пароль только в копии (`obs_login.json` 0600), Telegram/почта
  выключены. API-вход — `obs_api.py`, UI-пробы — `obs_ui.cjs`, правки копии — `db_edits.log`.
- **Перепроверка** (`reports/s8r_checklist_recheck_2026-10-16.md`): 14 пунктов `[x]` — S8.1 п.6, S8.4 п.5,
  S8.7 п.5, S8.8 п.3, S8.11 п.4, S8.15-4 п.2, S8.15-6 п.3, S8.18 п.2, S8.17 п.8, S8.31 п.5, S8.32 п.3,
  S8.33 п.2, S8.50 п.5, S8.59 п.2. Чеклист **334 [x] / 32 [~] / 0 [ ]** (обе копии одинаковы).
  S8.42 п.2 — paper-часть ✅, sandbox → «Боевая» — со стартом sandbox-сессии.
- **S8R-FIX-134 (medium, тест):** в окружении CI (`DEBUG=false`) падает
  `test_sandbox_account_of_other_user_is_refused` — нет `patch(settings.DEBUG)`; локальный гейт с
  `DEBUG=true` это прячет, CI не идёт из-за квоты. Исправление в worktree
  `Develop/.claude/worktrees/fix-134` (ветка `s8r/fix-134-sandbox-test-debug`, `6485f00` → **PR #74** в develop, 2026-10-17):
  22 passed в обоих окружениях, мутация красная, полный прогон в окружении CI 6079 passed / 1 skipped,
  покрытие 92,33 %. Мерж — по команде заказчика. Предложение (заказчик: «гейт пока не менять»): в гейт `Develop/CLAUDE.md` добавить
  прогон с `DEBUG=false` (окружение CI).
- **Наблюдения (стенд-копия):** paper-сессия **#13** — шорт LKOH 1 лот, 1ч, стратегия #7 (копия #4,
  short, SL/TP 50 %) — держится до отсечки LKOH (посл. покупка 08.12.2026, реестр 09.12, 535 ₽;
  единственная объявленная будущая отсечка среди 38 ликвидных бумаг) → S8.61 п.1 и п.3.
  Отложенные задачи (время Mac, CEST): `s8r-weekend-start` Сб 17.10 08:35 — sandbox SBER 1m 1 лот на #3
  (`weekend.py start`, вход по первому субботнему бару) → S8.46 п.2 и остаток S8.42 п.2;
  `s8r-weekend-report` Сб 09:25; `s8r-weekend-close` Сб 17:40 — «Закрыть все» + стоп + отчёт;
  `s8r-obs-daily` ежедневно ~09:17 — стенд жив (поднимает backend), шорт держится, корп. действия,
  сплиты; после 09.12 — PASS/FAIL по S8.61 п.1/п.3. S8.46 п.2 «отказ по бумаге без торгов выходного
  дня» вживую не воспроизводится (нет свечей — нет сигнала), покрыто тестами.
- **Пн 19.10 с 10:00 МСК — ордерные сценарии на копии** (напоминание этой сессии на 09:03 по Mac):
  M1 FIX-124 — продажа GAZP вне терминала при позиции сессии → пауза ≤15 мин, сделка открыта,
  уведомление один раз и не повторяется после рестарта, «Возобновить» → снова пауза с уведомлением
  (S8.28 п.3/п.4, S8.29 п.3/п.8); M2 — бумага у брокера, неизвестная терминалу → «возможно,
  куплено вручную», без паузы (S8.28 п.6); M3 — нехватка бумаг у брокера останавливает сессию (S8.29 п.7);
  M4 — закрывающий ордер исполнился при остановленном backend → после рестарта «Закрыта» с фактом,
  дневная статистика один раз (S8.28 п.7/п.8); M5 — встречный ордер в полёте не считается расхождением (S8.29 п.9);
  M6 — paper и sandbox на одних данных → одинаковый чистый P&L (S8.22 п.7); M7 — сработавший уровень вне
  торговых часов (S8.25 п.5, позицию держать через 23:50); M8 — «Возобновить» отказывает при другой сессии
  на инструменте (S8.30 п.8, правка копии); M9 — пара конфликтующих сессий после рестарта (S8.30 п.10, правка копии);
  сценарий 3 п.4 (Telegram) — подписи sandbox в `/status`/`/positions` через обработчики бота на копии.
  Не берутся: S8.31 п.2 (real — ордера только заказчик), S8.30 п.3 (нужен второй торговый счёт),
  S8.22 п.5 (комиссию удерживает только real). Сначала — убедиться, что субботняя позиция SBER закрыта.
- **Рабочий стенд** не трогался: `d48c6f3`, сессия #2 LKOH paper, health 200.

**2026-10-16 (17:30) — репо документации переименован `SergoPipo/test` → `SergoPipo/moex-terminal-docs` (решение
заказчика).**

- `gh repo rename` (репо публичный; GitHub перенаправляет старые URL, пока на аккаунте не создан новый `test`).
- `origin` docs-репо → `git@github.com:SergoPipo/moex-terminal-docs.git`; `fetch` проходит, ветка
  `docs/backlog-006-strategy-builder` синхронна (`c0fd8f6`).
- Корневой `CLAUDE.md` — новое имя репо; память Claude (`feedback_two_repos`, `project_docs_branch_not_main`,
  `feedback_no_superpowers`, `MEMORY.md`). Упоминания `SergoPipo/test` в старых handoff (`Code_Review_Full_2026-07/
  P1_WAVE{2,3}_HANDOFF.md`) — история, не тронуты. Код-репо `moex-terminal` не затронут (ссылок на `test` нет).
- Попутно: Context7 (плагин) ждёт входа заказчика (Authenticate); Gmail/Higgsfield — коннекторы аккаунта claude.ai, к
  проекту не привязаны.

**2026-10-16 (17:18) — PR #73 смержен по команде заказчика → develop `d48c6f3`, стенд подтянут.**

- `gh pr merge 73 --merge` по локальному гейту (CI стоит — квота Actions; develop с момента гейта не менялся, `4028d83`).
- `git pull --ff-only` в `Develop/`: `4028d83 → d48c6f3`, изменился только `CLAUDE.md` — backend не перезапускался
  (`--reload` смотрит `.py`), таймер `detect_corporate_actions` не сброшен (детект ≈23:01 по Mac); health 200, 5173 200.
  Правка `.env.example` заказчика на месте.
- Заказчик удалил `Develop/backend/.venv.old-Test` и локальную ветку `s8r/rename-paths` (`-d`, коммит был в
  `origin/s8r/rename-paths`), затем удалённую ветку (`git push origin --delete s8r/rename-paths`). Хвостов
  переименования не осталось.

**2026-10-16 (17:15) — каталог переименован `Test` → `Moex-Terminal`, окружение восстановлено, стенд поднят.**

- Проверка: docs `88ad7be`, код `develop 4028d83` = `origin/develop`, по одному worktree, хуков нет; `.env` на старый путь
  проверил заказчик — пусто.
- Память Claude: `memory/` (27 файлов) скопирована в ключ `-Users-sergopipo-Documents-Claude-Code-Moex-Terminal`
  без перезаписи, путь в `feedback_two_repos.md` исправлен; история сессий осталась в старом ключе (не копировалась, 560 МБ;
  старый каталог не удалён). `~/.claude/settings.json` строки 93–94 (`additionalDirectories`) → новый путь (с подтверждения
  заказчика). Плагин `pyright-lsp` для новой папки уже был включён (запись от 16:43); запись для старого пути осталась —
  безвредна.
- venv backend: старый → `Develop/backend/.venv.old-Test` (удалит заказчик), новый — Python 3.11.15 по рецепту
  `Develop/CLAUDE.md` (lock с хэшами, SDK, editable, `pip check` 0, `verify_install` 0) + `bandit==1.9.4` (как был; в CI
  ставится отдельно). Пакеты, стоявшие в старом venv вне lock (`safety`, `RestrictedPython`, `nltk`, `Authlib` и пр.), не
  ставились — для гейта не нужны. Шебанги `uvicorn`/`alembic`/`pytest` — новый путь.
- Frontend: `pnpm install --frozen-lockfile` — шимы `.bin` перепривязаны (было 6 со старым путём, стало 0); кэш Vite
  чистый, оставлен.
- Пути: код — `Develop/CLAUDE.md` строка `V=…` (ветка `s8r/rename-paths`, `6ee41c7` → **PR #73** в develop; мерж — по
  команде заказчика, CI стоит); docs — `restart_dev.sh`, `start_ngrok.sh`,
  `deployment_guide.md` (2), `FAQ/vps_migration_guide.md`, `ui_check_teststand_guide.md`; локальный (не коммитится)
  `s8r-evidence/teststand/teststand.sh` (T/V/SRC; его worktree `wt-s8r-teststand` уже удалён). Остались 24 упоминания в
  9 исторических отчётах/журналах — не трогались.
- Стенд запущен 17:01 (вкладка Terminal, `bash scripts/start.sh`): `grpc_dns_resolver_configured resolver=native`
  (раньше `…_preset` — переменная была задана в shell; после S8R-FIX-093 не нужна), `scheduler_started`,
  `session_runtime_restore_complete restored=1` (LKOH paper #2), без WARNING/ERROR (кроме известного `UserWarning`
  protobuf `GetPrototype`); health 200, `localhost:5173` 200; ngrok-туннель жив (→ `localhost:8000`). Таймер
  `detect_corporate_actions` сброшен — следующий детект ≈23:01 по Mac.
- Smoke-гейт в worktree `s8r/rename-paths` от `origin/develop`: pytest **6079 passed / 1 skipped**, ruff/mypy 0,
  bandit medium+ 0, `pnpm typecheck` 0, vitest **1482** (192 файла).

**2026-10-16 (16:30) — подготовка к переименованию каталога `Test` → `Moex-Terminal` (решение заказчика).**

- Worktree агента `.claude/worktrees/agent-a8dcfbad` (апрель 2026, незакоммиченная правка
  `Спринты/Sprint_6/reports/DEV-7_report.md`, +31/−19) — по решению заказчика правка сохранена:
  `Спринты/Sprint_6/reports/DEV-7_report.agent-a8dcfbad-uncommitted.patch` и `.md` (патч проверен — применяется к
  `fa6cf3d` и даёт ровно сохранённый файл); worktree удалён, ветка `worktree-agent-a8dcfbad` удалена (уже в docs-ветке).
- `core.hooksPath` (абсолютный путь на стандартный `.git/hooks`, активных хуков нет) снят в обоих репо.
- Передача — `~/.claude/handoffs/handoff-20261016-1630.md`: шаги восстановления после переименования (память Claude,
  `~/.claude/settings.json`, плагин pyright-lsp, venv, `pnpm install`, пути в файлах, запуск стенда, smoke-гейт).
- Переименование делает заказчик: остановить стенд → закрыть сессии/терминалы в папке → `mv` → проверить `.env` → новая
  сессия в `Moex-Terminal`.

**2026-10-15 (22:23) — живая проверка S8R-FIX-129 и обновление стенда до `4028d83`.**

- Детект 22:23:15 (тикеров 3): SBER 37,64 ₽, реестр 20.07.2026 — проведён, остановленная paper #1 **+376,40 ₽**
  (10 лотов × лот 1; прежняя оценка «+3 764 ₽» считала лот 10 — ошибка оценки, не начисления); sandbox #3/#6 —
  `DailyStat` за день нет; уведомления пользователям 1 и 4 (владелец sandbox #6); WARNING/ERROR нет. S8.60 → [x]:
  **320 [x] / 46 [~] / 0 [ ]**.
- Стенд обновлён `833ac34` → **`4028d83`** (S8R-FIX-131/132/133): снимок `terminal.db.bak-2026-10-15-pre-4028d83`
  (integrity ok; 10 сессий / 45 сделок / 91 корп. действие), `git pull --ff-only`, перезапуск в 22:24 чистый,
  сессия #2 восстановлена, `/health` 200, фронт 200. Следующий детект — ≈04:24.
- Ждут: переименование каталога `Test` → `Moex-Terminal` (план — запись «после полудня»); CI GitHub (квота Actions).

**2026-10-15 (вечер) — S8R-FIX-133 (уведомление держателю шорта: «сумма спишется»; решение заказчика).**

- По команде заказчика: коммит `f352cb3`, **PR #72 смержен** по локальным гейтам → **develop `4028d83`** (дерево = проверенному);
  worktree и локальная ветка удалены. **Стенд — обновить до `4028d83` после детекта ≈22:23** (S8.60).
- Ветка `s8r/fix-133-short-notification` (от `585e297`): приписка «; по короткой
  позиции сумма спишется» владельцам коротких позиций, которым причитается дивиденд/купон. 1 тест, 3 мутации; гейт —
  pytest 6079 / 1 skipped, ruff/mypy/bandit 0. ФТ 4.6 / ТЗ 3.6, чеклист S8.61 +1 [~]: **319 [x] / 47 [~] / 0 [ ]**.

**2026-10-15 (вечер) — S8R-FIX-131 и S8R-FIX-132 по решениям заказчика (находки ревью 129).**

- 131 (`s8r/fix-131-short-dividend`, worktree `wt-s8r-131`): дивиденд и купон шорта — со знаком минус; 3 теста, мутации
  красные; купон добавлен по ревью. 132 (`s8r/fix-132-split-notify-stopped`, worktree `wt-s8r-132`): уведомление о сплите
  владельцу открытой позиции остановленной сессии; 2 теста, 4 мутации. Ревью (агент) — high нет, ветки без конфликтов.
  Гейт 132: pytest 6075 / 1 skipped; гейт 131: pytest 6076 / 1 skipped; ruff/mypy/bandit 0 в обеих. ФТ 4.5, ТЗ 3.5, чеклист S8.61 (+2 [~]):
  **319 [x] / 46 [~] / 0 [ ]**. По команде заказчика: коммиты `97146b5` (132) и `3fb35f6` (131), **PR #70 и #71 смержены**
  по локальным гейтам (CI — квота Actions) → **develop `585e297`**; гейт объединённого состояния: pytest **6078 / 1 skipped**, ruff/mypy/bandit 0; worktree 131/132
  и локальные ветки удалены. **Стенд — обновить после детекта ≈22:23** (S8.60, команда заказчика).
- По ходу: опечатка исполнителя в цикле создания worktree (zsh не разбивает `$c`) создала worktree по пути с пробелом и
  локальную ветку `refs/heads/origin/develop` — удалены сразу (`git update-ref -d` с проверкой SHA), удалённая
  `origin/develop` не затронута.

**2026-10-15 (вечер) — S8R-FIX-129 (дивиденды по позициям остановленных сессий; решение заказчика «делай»).**

- По команде заказчика: коммит `c8bf307`, **PR #69 смержен** по локальным гейтам (CI — квота Actions) → **develop
  `833ac34`**; рабочий стенд обновлён (снимок `terminal.db.bak-2026-10-15-pre-833ac34`, перезапуск в 16:23 чистый,
  сессия #2 восстановлена). **Следующий шаг — детект ≈22:23 по Mac:** проверить SBER 20.07.2026 → paper #1 (баланс
  111 619,44 до), одно уведомление, WARNING/ERROR нет; закрыть S8.60.
- Ветка `s8r/fix-129-dividends-stopped-sessions` (от `8f5ce74`): тикеры детекта —
  живые сессии и остановленные со сделками в начисляемых статусах (`detection_tickers`); уведомление о дивиденде/купоне —
  и владельцам позиций, которым он начислен (`accrual_recipient_user_ids`). TDD: 13 тестов на границе джобы, RED с
  фактическим выводом, 9 мутаций красные. Ревью (агент): 3 находки исправлены (регистр тикера — двойное начисление;
  граница «год»; сбой поиска получателей), сплит-уведомления и дивиденд короткой позиции — находками. ФТ 4.4, ТЗ 3.4,
  чеклист S8.60 (+1 [~]): **319 [x] / 44 [~] / 0 [ ]**. Гейт: pytest **6073 / 1 skipped**, ruff/mypy/bandit 0.
- Перед этим — docs `215fac1` (S8.59: Qwen проверен заказчиком, «Ключ валиден»).

**2026-10-15 (после полудня) — S8R-FIX-130 (AI-провайдер за прокси/VPN; решение заказчика — вариант «В»).**

- Заказчик не смог подключить OpenRouter `qwen/qwen3.8-flash`: «Проверить» → «Не удалось проверить ключ провайдера».
  Разбор: настройки провайдера #3 верные (тип `openrouter`, модель есть в публичном списке, адрес
  `https://openrouter.ai/api/v1`); правило Proxifier `openrouter.ai; *.openrouter.ai` с разрешением имён через прокси —
  DNS отдаёт заглушку `0.0.10.166`, SSRF-барьер её отвергает (`POST /providers/3/verify` 422, причина не показана).
- Ветка `s8r/fix-130-ai-proxy-placeholder` (от `b10c562`; коммит `9743097`, **PR #68 смержен** по команде заказчика по
  локальным гейтам → **develop `8f5ce74`**, CI не стартовал — квота Actions; worktree и локальная ветка удалены): заглушки прокси/VPN
  допустимы только для известных AI-провайдеров (точное имя), проверка и при соединении; `verify_provider` отдаёт
  причину; заглушка — только по https (находка ревью). TDD: 22 теста, RED с фактическим выводом, 12 мутаций красные;
  живой закреплённый клиент через Proxifier — HTTP 200. Ревью (агент): обхода нет, 2 находки исправлены, 1 оставлена. Миграции нет, фронт не менялся. ФТ 4.3, ТЗ 3.3, `deployment_guide` (строка флага), чеклист S8.59 (+2 [~]):
  **317 [x] / 45 [~] / 0 [ ]**. Гейт: pytest **6060 / 1 skipped**, ruff/mypy/bandit 0.
- Вместо флага `AI_ALLOW_PRIVATE_PROVIDER_URLS` (вариант «Б») — по команде заказчика стенд обновлён до `8f5ce74` после
  детекта дивидендов: снимок БД `terminal.db.bak-2026-10-15-pre-8f5ce74` (integrity ok; 10 сессий / 45 сделок / 90 корп.
  действий), `git pull --ff-only`, backend перезапустился сам в 15:31 (`restored=1`, без WARNING/ERROR), `/health` 200,
  фронт 200, код стенда принимает `openrouter.ai`.
- **Детект дивидендов (S8.58) — 15:30:15, всё по ожиданию:** LKOH 278 ₽ (реестр 04.05.2026) проведён — paper #2 +278,00 ₽
  (114 916,77 → 115 194,77), LKOH 535 ₽ (реестр 09.12.2026) ждёт реестра, 12.01.2026 вне окна; уведомления #2871/#2872
  с обеими датами. S8.58 → [x]: **318 [x] / 44 [~] / 0 [ ]**. SBER не детектирован (paper #1 остановлена) — находка,
  предложение S8R-FIX-129 ждёт решения заказчика.
- **Qwen проверен заказчиком** (рабочий стенд `8f5ce74`): «Ключ валиден», `POST /providers/3/verify` 200, `is_verified=1`,
  провайдер активен — S8.59 п.1 → [x]: **319 [x] / 43 [~] / 0 [ ]**.
- Ждут: решение по S8R-FIX-129 (дивиденды остановленных
  сессий); заказчик планирует переименовать каталог проекта `Test` → `Moex-Terminal` (план — в чате 2026-10-15:
  после остановки стенда; память Claude, venv, `pnpm install`, `core.hooksPath`, забытый worktree агента, пути в
  `Develop/CLAUDE.md` и скриптах); CI GitHub (квота Actions).

**2026-10-15 (день) — S8R-FIX-128 (по команде заказчика; лимит 3 оставлен — решение заказчика).**

- Ветка `s8r/fix-128-backtest-limit-ui` (worktree `wt-s8r-128` от `834a05b`, не закоммичено): «Запустить в фоне» без
  своего отказа по бейджу — лимит решает сервер, тост с его текстом; значки статусов в таблице бэктестов стратегии
  («×» «Отменён», «...» для очереди, полное название в подсказке). Пункт про давний баннер — без кода (объяснение в
  backlog). Тесты первым, 3 мутации красные; typecheck/build/lint 0, vitest **1482**; Playwright изолированно — 175 passed /
  1 failed (`auth-hardening`, нужен живой backend) / 6 did not run, как у 127; временные конфиги удалены. Backend не менялся.
- Вопрос заказчика «почему лимит 3»: Grid Search — одна задача из трёх, внутри до 1000 комбинаций на 9 процессах;
  после S8R-FIX-127 в лимит входят и обычные бэктесты. Решение заказчика — оставить 3.
- По команде заказчика: коммит `f951180`, **PR #67** смержен по локальным гейтам → **develop `b10c562`** (ветка от проверенного
  `834a05b`, только фронт — дерево develop совпадает с проверенным). Рабочий стенд: `git pull --ff-only`, backend не
  перезапускался (не менялся), Vite отдаёт новый код; на странице стратегии у #61 — «×» «Отменён». Worktree и локальная
  ветка удалены.

**2026-10-15 (утро) — мерж #65/#66 и обновление рабочего стенда (по команде заказчика).**

- Коммиты: `moex-terminal` `s8r/fix-122-dividends-tinvest` — `1970fc3` (**PR #65**), `s8r/fix-127-backtest-jobs` — `84ce5a3`
  (**PR #66**). CI GitHub не стартует (квота Actions, 0 проверок) — оба PR смержены по команде заказчика по локальным
  гейтам: `7e3bf6b`, `834a05b` — **develop `834a05b`**. Гейт объединённого состояния: pytest **6038 / 1 skipped**
  (5971 + 40 + 27), ruff/mypy/bandit 0, typecheck/build/lint 0, vitest **1478**.
- **Рабочий стенд обновлён** `e527a05` → `834a05b` (`git pull --ff-only`; миграций и зависимостей нет; правка
  `.env.example` заказчика сохранена). Перед этим — снимок БД через backup API
  `data/terminal.db.bak-2026-10-15-pre-834a05b` (integrity ok; 10 сессий / 45 сделок / 88 корп. действий; paper #1
  111 619,44 ₽, #2 114 916,77 ₽). Backend перезапустился сам (`--reload`, 09:30): `grpc_dns_resolver_preset native`
  (FIX-093), `session_runtime_restore_complete restored=1`, без WARNING/ERROR; `/api/v1/health` 200, фронт 200, Vite
  отдаёт новый код; сессия #2 (LKOH paper) активна; туннель ngrok не трогался. `GRPC_DNS_RESOLVER=native` в команде
  запуска больше не нужен (стенд пока запущен со старой командой — вреда нет).
- Первый детект корп. действий — по интервалу 6 ч от старта планировщика (~15:30): начислит paper #2 LKOH +278 ₽
  (реестр 04.05.2026), paper #1 SBER +3 764 ₽ (реестр 20.07.2026) и пришлёт уведомления — проверка S8.58.
- Уборка: worktree `wt-s8r-div`/`wt-s8r-127` и локальные ветки удалены (удалённые на GitHub остались).
- **Живая проверка 127 (2026-10-15, вход под sergopipo по паролю заказчика):** отмена обычного бэктеста #61 → «Отменён»;
  общий лимит — обычный при трёх фоновых → 422 и тост, строки бэктеста нет; ожидание лимита T-Invest сработало вживую
  (паузы 15/42 с). Оба пункта чеклиста → [x]: **317 [x] / 43 [~] / 0 [ ]**. Находки — S8R-FIX-128 (low): текст и счёт
  фронтового лимита фоновых, статус «?» у отменённого бэктеста в таблице стратегии; давний баннер расхождения GAZP.
  Созданы бэктесты проверки #61–#65 (остаются в списке).
- Ждут: S8.58 — после первого детекта (~15:30); CI GitHub — решение заказчика по квоте Actions; вопрос заказчика о лимите 3.

**2026-10-14 (ночь) — S8R-FIX-122 и S8R-FIX-127 по решениям заказчика готовы (ветки от `1454c16`, не закоммичено).**

- **S8R-FIX-122 (дивиденды T-Invest) — готово.** `wt-s8r-div`, ветка `s8r/fix-122-dividends-tinvest`. Живое чтение
  `GetDividends` на sandbox #3 (только чтение, `s8r-evidence/checkrun/live/dividends_probe.py`) подтвердило находку DEV:
  отсечка дивиденда/купона стояла на торговый день раньше (T+1). Решения заказчика: чинить в этой ветке; прошлые
  дивиденды начислять (paper #2 LKOH +278 ₽, paper #1 SBER +3 764 ₽ при первом детекте после обновления). Ревью — 4
  находки исправлены (сверка непроведённых с брокером, недоступный счёт не глушит прогон, перебор владельцев, пустая
  валюта). 10 мутаций красные. Гейт: pytest **6011 / 1 skipped**, ruff/mypy/bandit 0.
- **S8R-FIX-127 (обычный бэктест через `BacktestJobManager`) — готово.** `wt-s8r-127`, ветка `s8r/fix-127-backtest-jobs`.
  Три раунда: DEV (реализация) → ревью 1 (6 находок, в т.ч. тяжёлая — поздняя отмена навсегда занимала место в лимите) →
  DEV-127R → ревью 2 дельты (7 находок, живую job самовосстановление не трогает) → DEV-127R. Все находки исправлены тестом
  первым, мутации красные. Финальный гейт: pytest **5998 / 1 skipped**, ruff/mypy/bandit 0, typecheck/build/lint 0,
  vitest **1478**; Playwright — изолированный прогон (свой Vite :5299, свой `cacheDir`, прокси и `PW_API_URL` на пустой
  :8999 — рабочий стенд не затрагивается): **175 passed / 1 failed / 1 skipped / 6 did not run** — упал только
  `auth-hardening.spec.ts` (по своему описанию требует живой backend с чистой БД; в изолированном прогоне backend
  намеренно не поднимался → `ECONNREFUSED :8999`, 6 тестов спеки не запустились); ветка auth и e2e не трогает.
  Прошлый цикл: 174 / 2 failed.
- Документы: ФТ **4.2** (§4.1, §6.4, §19.10), ТЗ **3.2** (§5.12, §11.4), backlog (раздел «Решения заказчика 2026-10-14
  (поздний вечер)»), чеклист: S8.58 → [~] (новый текст уведомления), Сценарий 4 + 2 пункта [~] (отмена обычного
  бэктеста, общий лимит) — проверка после мержа: **315 [x] / 45 [~] / 0 [ ]**.
- Уборка: временные БД проб ревьюеров в `$TMPDIR` удалены; временные конфиги Playwright в `wt-s8r-127/frontend` удалены,
  :5299 свободен; кэш Vite прогона — в scratchpad сессии.
- Дальше: предложить заказчику коммиты (ветки названы) → push → PR → мерж по команде по локальным гейтам → обновление
  стенда (`git pull` + перезапуск; миграций нет) → живая проверка: уведомления и начисления дивидендов LKOH/SBER,
  отмена обычного бэктеста и общий лимит.

**2026-10-14 (поздний вечер) — мерж и последние решения.**

- **PR #64 и #63 смержены** по команде заказчика по локальным гейтам (CI GitHub не стартует): `37b0aa8`, `1454c16` —
  **develop `1454c16`**. Гейт объединённого состояния: pytest 5971 / 1 skipped, ruff/mypy/bandit 0, typecheck/build/
  lint 0, vitest 1459. Миграций и зависимостей между `e527a05` и `1454c16` нет — рабочий стенд (ещё `e527a05`)
  обновляется `git pull` + перезапуск, по команде заказчика. Worktree `wt-s8r-fix`/`wt-s8r-react` и локальные ветки
  убраны (удалённые ветки на GitHub остались).
- **Причина блокировки CI найдена:** исчерпана бесплатная квота Actions приватного репо (2 000 мин/мес) — с 1 октября
  ~2 600 мин: CI 82 запуска × 26–38 оплачиваемых мин (задание `backend` — 19–29 мин), nightly 9 × 10–13. Снять — только
  заказчику в Billing (бюджет Actions / платёжные данные); предложено сократить расход в репо (ускорить backend-job,
  фильтры путей, nightly 1×/день) — ждёт команды.
- Решения заказчика: **S8R-FIX-127 — (а)** обычный бэктест через `BacktestJobManager` (ветка
  `s8r/fix-127-backtest-jobs`, worktree `wt-s8r-127`); **дивиденды — (а)** `GetDividends` T-Invest (ветка
  `s8r/fix-122-dividends-tinvest`, worktree `wt-s8r-div`) — в работе, DEV-агенты.

**2026-10-14 (вечер) — коммиты, PR и решения заказчика.**

- Коммиты по команде заказчика: `moex-terminal` `s8r/fix-checklist-run` — `caf448f` + `d2d81b8` (**PR #63**),
  `s8r/fix-093-grpc-resolver-import` — `9a1e7c6` (**PR #64**); `test` `docs/backlog-006-strategy-builder` — `5a290d5`
  и следующий. Все ветки запушены. **CI GitHub снова не запускается** (биллинг: «recent account payments have failed
  or your spending limit needs to be increased», все задания падают за 2 с) — мерж #63/#64 по локальным гейтам ждёт
  команды заказчика (как #60–#62 2026-10-13); auto-merge в репозитории выключен.
- Решения заказчика исполнены: пауза Dependabot не нужна (правки откатаны, worktree и ветка удалены); причина
  входа/выхода в бэктесте не реализуется (ФТ §19.9, S8.12 п.4 закрыт, «Причина: —» убрана); лимит T-Invest в бэктесте —
  автоповтор с ожиданием; мелочи сценария — исправлены (ревью блока: 1 находка — исправлена); инструкция ротации
  ключа для нативного стенда — `deployment_guide.md` §6а. Уборка заказчиком выполнена (worktree, ветки, кэши, стенды,
  файлы входа). Чеклист: 316 [x] / 42 [~] / 0 [ ].
- Ждут решения: источник дивидендов; S8R-FIX-127 (отмена обычного бэктеста — маршрута нет); мерж #63/#64 по локальным
  гейтам. Финальный гейт ветки: pytest 5966 / 1 skipped, ruff/mypy/bandit 0, typecheck/build/lint 0, vitest 1459.

**2026-10-14 — тот же цикл: ордерный сценарий, ревью дельты, чеклист.**

1. **Ордерный сценарий на sandbox #3 завершён** (рабочий стенд, параметры согласованы, `DEV_MODE=false` на время
   сценария): сверка снимает пометку сама (S8.28 п.5, S8.29 п.4), две сессии одного счёта без ложного расхождения,
   FIGI без брокера — один `figi_unresolved` и повтор через 15 мин (S8.35), 3-НДФЛ 2026 с GAZP, Telegram (`/balance`
   с заблокированной заявкой 270.13 → 0.00, `/status`, `/positions`, тестовое сообщение, «Все позиции закрыты»),
   закрытие из карточки с маркером на графике, «Запустить заново» (новая #11). Итог счёта — только рубли.
2. **Находки сценария:** S8R-FIX-124 (high, пауза при исчезнувшей позиции) — исправлена до этой записи;
   S8R-FIX-125 (цена в «Сделка исполнена» / «Позиция открыта») — исправлена; S8R-FIX-126 (перезапуск сессии с
   override сверки движков) и мелочи текстов — в работе; шум журнала watchdog'а, обрезанная подпись маркера,
   баннер давно снятого расхождения — записаны, не чинятся. На develop подтверждён дефект подписей режима в
   Telegram (sandbox = «Paper»/«real») — уже исправлен S8R-FIX-117 в ветке.
3. **Ревью дельты — два раунда** (8 находок, все исправлены DEV-4 тестом первым): общий счётчик неполных проходов
   у ручной и фоновой сверки, пауза сверкой без `session.paused`, ручная сверка единственной остановленной сессии
   принимала «только деньги» за ответ, своя/закрытая/гонка в привязке sandbox-счёта (общий лок `sandbox_recovery`),
   курс ЦБ через полночь, `/balance` заголовки, изоляция `test_backup_cli`.
4. **Чеклист:** обе копии `ui_checklist_s8.md` — 315 [x] / 42 [~] / 1 [ ] (журнал сценария —
   `s8r-evidence/checkrun/live/scenario_log.md`, локальный). ФТ **v4.1** / ТЗ **v3.1** (2026-10-14) — поведение цикла (094…126, 093), шапки и таблицы истории сходятся; обход для
   S8R-FIX-126 в ФТ §6.3 описан по коду, вживую не проверялся.
5. **Дальше:** гейт после FIX-126; ФТ/ТЗ; предложить заказчику ветки и сообщения коммитов по обоим репо; команды
   уборки (worktree, стенды-копии, кэши Vite, слитые ветки); решения заказчика — источник дивидендов, причина
   входа/выхода в бэктесте, «Нет рыночных данных» при лимите T-Invest, нужна ли пауза Dependabot (Actions снова
   работает).

**2026-10-13 (вечер) — цикл «обновление стенда + сплошная проверка UI-чеклиста S8.1–S8.58».**

Решения заказчика в начале цикла (такт 1): стенд обновить до `e527a05` с миграцией;
S8.1–S8.46 проверить заново (не принимать по прогону 2026-07-26), ошибки — чинить и
перепроверять; пункты с живыми событиями — эмулировать Playwright'ом; опции — «всё, что
возможно»; по ходу: прокси для тестовых стендов убрать, чтобы работало как в продакшене.

1. **Стенд обновлён** — подробности в backlog.md (раздел цикла): снимок
   `data/terminal.db.bak-2026-10-13-pre-e527a05`, `a854e13` → `e527a05`, venv по lock,
   `pnpm install`, миграция `a7c1d9e2b4f8`, туннель и webhook не перезапускались.
2. **S8R-FIX-093** (high) — фикс 083 не действовал: grpc читает `GRPC_DNS_RESOLVER` при
   импорте, а шаг корней в lifespan импортировал grpc раньше. Ветка
   `s8r/fix-093-grpc-resolver-import` (worktree `wt-s8r-react`), TDD + /code-review (6
   находок исправлены), pytest 5868 passed. Не закоммичено — ждёт команды. Рабочий стенд
   до мержа запущен `GRPC_DNS_RESOLVER=native bash scripts/start.sh`.
3. **Вопрос заказчика о прокси**: проекту прокси не нужен, переменные из `~/.zshrc`;
   на брокер не влияет (11 с с прокси и без). OpenRouter — единственный адрес
   `https://openrouter.ai/api/v1`; правило Proxifier `openrouter.ai; *.openrouter.ai:443`.
4. **Стенды-копии для сплошной проверки**: `s8r-evidence/checkrun/checkstand.sh <n>
   reset|up|down|status` — worktree `wt-s8r-check` (develop, detached), копия БД из
   снимка через backup API + `alembic upgrade head`, тестовые пароли только в копии
   (`stand_credentials.json`, 0600), ключи брокера — `MOEX_ENV_FILE` на `.env` рабочего
   стенда (файл не читается), Telegram и почта выключены, `DEV_MODE=true`, прокси сняты
   (решение заказчика), свой `cacheDir` Vite и `CORS_ORIGINS` на стенд (gotcha-46).
   Обвязка Playwright — `wt-s8r-check/frontend/e2e-check/`. Четыре QA-агента по
   промпту `prompt_s8r_checklist_qa.md` (пакеты qa1–qa4, стенды 1–4), отчёты —
   `reports/s8r_checklist_run_qa<N>.md`. Брокерские ордера, Telegram и эмуляции — оркестратор.

**Предыдущая запись:**
**2026-10-13 — цикл «закрыть все незакрытые замечания» (по команде заказчика).**

Объём подтверждён заказчиком: все открытые карточки, включая хвосты
сентябрьского аудита. Решение по **S8R-FIX-089** — вариант **(а)**, общий хук.
Ветка `s8r/fix-tails-ui` (worktree `wt-s8r-react`), построена поверх
`s8r/fix-080-react-compiler-rules` (PR #58) с влитой веткой 085 (PR #56) —
её `mergeSessionKeepingStats` понадобился карточке 088. PR по ветке будет
включать коммиты #56 и #58, пока те не смержены.

**Бухгалтерия (коммит `9955865` в репо документации).** Разбор показал, что
восемь «открытых» карточек на самом деле закрыты: **070, 071** реализованы и
подтверждены прогоном S8.56 (9 из 9) против живого sandbox-счёта, **082, 083,
084** в develop (PR #52), **085** в PR #56, **077** и **081** закрыты решениями
заказчика. В ФТ дописаны 082–085 — они меняли видимое поведение, а в документ
не попали.

**Закрыто кодом (12 карточек).**

- **088** — починено в КОРНЕ, а не третьей копией обходного приёма во фронте:
  `TradingService._session_card_response` отдаёт pause/resume/stop ту же
  заполненную карточку, что `get_session`. Обходной приём 085 снят — после
  корневого фикса он отбрасывал бы свежие числа. Пропажа позиции была опаснее
  потери статистики: на паузе SL/TP не работают.
- **087** — две половины: `close_failure_reason` не отдаёт пустую строку
  (запасной текст называет класс исключения — иначе бесполезен при разборе), а
  фронт берёт причину проверкой на пустоту: `??` ловит только `null`.
- **089** (а) — хук `useAsyncData`: один эффект загрузки вместо восьми
  рукописных копий, 8 тестов. Перенесены шесть компонентов; `RiskSettingsPage`
  (поколения запросов против гонки повторных «Повторить») и `StrategyEditPage`
  (наполняет десяток полей, трогает ref'ы) оставлены осознанно.
- **007** — бейдж «ожидает брокера» и подпись вместо прочерка: `pending` входит
  в `OPEN_POSITION_TRADE_STATUSES`, то есть деньги уже в рынке.
- **014** — карточка инструмента отдаёт настоящий лот (`ensure_lot_size`, не
  единица из ISS), форма запуска предупреждает, когда суммы не хватает на лот.
- **011** (фронт) — `useWsAuthGeneration()` на `useSyncExternalStore`:
  после переподключения WS повторяется REST-подписка на стрим. Разрыв дольше
  60 с снимал серверную подписку, и это выглядело как вставший рынок.
- **022** (фронт) — `upsertLiveCandle` правит бар в любом месте окна, а не
  только последний: бар, переживший обрыв, приходит исправленным уже не
  последним и молча отбрасывался.
- **001** — `SOURCE=volume` считается по объёму и в live: parity давал
  `count_mismatch`, и «виноватым» выглядел бэктест.
- **004** — имена параметров Bollinger сведены к кодогену IR: «Применить
  параметры» из Grid Search давало 422 на том, что сам же grid и перебирал.
- **015** (2 из 3) — inline-копии проверки направления заменены на
  `is_long_direction`; выручка paper-закрытия — по лотам позиции.
- **010** — лок `sandbox_recovery_user` описан в порядке захвата; три прямых
  вызова `find_instrument` переведены под дедлайн адаптера.
- **013** — пик equity поднимается на каждой закрытой свече (между сигналами
  максимум не фиксировался, и лимит просадки срабатывал позже, чем настроен);
  мёртвая `daily_stats.peak_equity` удалена миграцией `a7c1d9e2b4f8`.

**Закрыто кодом — вторая половина цикла (14 карточек, те же сутки).**

- **002** — `test_users_cli` проходил вакуумно: подпроцесс падал на валидаторе
  настроек, а утверждением было «код возврата не ноль», то есть регресс
  `grant_admin` тестами не ловился вовсе. Теперь проверяется ЭФФЕКТ в БД.
- **003** — ТЗ §8.4 описывал подсистему логов, которой нет; раздел приведён к
  факту, а правам лога добавлены `umask 077` + `chmod 600` (в лог попадают
  идентификаторы счетов и тексты ошибок брокера).
- **006** — семантика `nkd_entry/nkd_exit` зафиксирована до появления писателя:
  «на ОДНУ облигацию, ₽». Купонный учёт и налоговая база разошлись бы в размер
  позиции, причём молча — сейчас обе величины нули.
- **008** — уведомление называет обе даты: «закрытие реестра DD.MM.YYYY,
  ex-date DD.MM.YYYY». Переименовать колонку нельзя — `ex_date` входит в ключ
  дедупа. Остаток (нужна ли сплиту отсечка по дате) — решение по ФТ §6.4.
- **009** — `SHUTDOWN_WAIT_SEC = 10` на остановку job-менеджера (отмена
  корутины не прерывает расчёт в потоке, а дозапись под write-локом SQLite
  тянулась до `busy_timeout`); grid без слотов стоит в `queued` с событием
  `waiting`, а не в «running 0 %».
- **012** — оба дефекта закрыты более поздней работой; закреплены инвариантными
  тестами (выход — только по терминальному состоянию ордера; инциденты — через
  outbox по владельцу, живой слушатель не нужен).
- **015** (третий пункт) — paper-просадка учитывает открытую позицию тем же
  источником цены, что sandbox/real: защита включается во время убытка.
- **017** — один декодер access-токена на три входа (REST, WS, админка) и один
  текст отказа бота вместо пяти копий. `detail` 401 на экране входа
  осознанно не делался: противоречит S8R-AUDIT-043 (оракул перечисления).
- **018** — shutdown-recovery паузит и `suspended`-сессии (частичный выход,
  доученный на остановке, иначе терялся); осиротевшая сессия → `stopped`, иначе
  она держала пару «счёт+тикер» навсегда.
- **019** — `RESTORE_TOTAL_BUDGET_SEC = 45` общим дедлайном вместо суммы
  частных: при молчащем брокере старт мог превысить `start_period` healthcheck
  и уйти в петлю перезапусков.
- **020** — портфели счетов в периодической сверке греются параллельно под одним
  бюджетом (тем же приёмом, что на старте): последовательный проход на N счетах
  держал watchdog стримов и доучёт ордеров.
- **021** (частично) — `alembic/env.py` нормализует путь БД: `upgrade head` из
  корня репозитория создавал свой файл и «успешно» мигрировал его.
- **023** — тесты идут с `PRAGMA foreign_keys=ON`. Карточка предсказывала 162
  падения — фактически 5 ошибок teardown из-за цикла strategies↔strategy_versions.
- **024** — `GRPC_ENABLE_FORK_SUPPORT=0` до импортов grpc, `faulthandler_timeout`
  в `addopts`: SIGABRT в preflight-тестах и вечное ожидание при зависании.
- **079** — `@tanstack/react-table` 9.3.1: `tableFeatures` + фабрики
  `create*RowModel`, `table.state`, `getAllCells`, `sortFn`. Подавление
  `react-hooks/incompatible-library` больше не нужно.

**`/code-review` по правилу CLAUDE.md** (правки в `app/trading/`,
`app/circuit_breaker/`, `app/market_data/`) — xhigh по диффу второй половины
цикла, **2 находки, обе исправлены здесь же**; обе — об одном: состав
бумажного equity после S8R-FIX-015 оказался записан в трёх местах по-разному.

- **Пик бумажного портфеля не поднимался нереализованной прибылью.** Третий
  пункт 015 добавил нереализованную часть в `current`, а пик продолжал
  поднимать только денежный учёт `PaperTradingEngine` (`balance + blocked` на
  закрытии сделки). Прибыль открытой позиции опускала просадку, но пика не
  поднимала — и когда та же прибыль уходила, просадка считалась от величины,
  которой капитал уже не был: лимит снова срабатывал позже, чем настроен, а
  режимы (у sandbox/real пик берётся от equity С нереализованным) снова
  расходились — ровно то, что карточка устраняла.
- **Экран показывал другую просадку, чем движок.** `CircuitBreakerService.get_status`
  (блок «текущее состояние» над формой риск-менеджмента, ФТ §12.4) считал
  бумажную просадку по старой формуле и давал 0 % там, где движок уже видел
  просадку.

Исправлено в корне, а не двумя правками формул: состав бумажного equity сведён
к одной функции `paper_equity_by_session` (симметрично `broker_equity_by_session`),
пик — к `paper_peak_equity` (симметрично `broker_peak_equity`); её используют
проверка просадки, `update_peak_on_bar` и статус. 4 новых теста
(`test_paper_peak_with_unrealized.py`), мутации «пик paper не растёт», «статус
по деньгам», «equity без нереализованного», «цена сигнала игнорируется» — все
красные. Структурные тесты 015 (`inspect.getsource`) переписаны на
поведенческие: они сломались от самого переноса арифметики в хелпер, ничего при
этом не проверив, — и это же объясняет, почему расхождение состава пережило
прошлый прогон.

**Третья находка — из самого прогона, не из ревью.** Полный pytest дал
`test_job_goes_to_error_with_reason`: `assert 'done' == 'error'` при зелёном
изолированном прогоне (3 из 3). Причина не в тесте: `_run_backtest_task` ловит
ошибку внутри, помечает `backtests.failed` и не пере-бросывает, а runner job'а
узнавал о провале, **перечитывая запись из БД**. Между пометкой и чтением есть
зазор — пометка идёт через `persist_with_retry`, у неё ветка «последний рубеж»
на свежей сессии и ветка «даже она не помогла, подберёт реконсиляция», — и
пока запись не видна, runner возвращает результат как при успехе: job → `done`,
пользователю уходит «Бэктест завершён», хотя бэктест упал. Это ровно тот
дефект, который закрывал BUG-31, но закрывал по следствию (статус в БД), а не
по причине. Теперь задача возвращает текст ошибки значением, и решение
принимается по нему; перечитывание записи осталось запасной проверкой и
источником метрик. 3 теста, обе мутации («решение не по возвращённому
значению», «задача не возвращает причину») — красные. Фоновый путь
`create_backtest` возвращаемое значение игнорирует, как и раньше.

**Гейты на конец цикла (2026-10-13):** backend полный pytest
**5839 passed / 1 skipped / 1 failed** — `test_backup_cli` отказывается делать
restore, пока на :8000 слушает стенд заказчика (известный артефакт, не
регресс); frontend typecheck/lint 0, build 0, vitest **1376 passed**;
Playwright по таблицам v9 — **7 из 7** (`e2e/s8r-fix-079-tables-v9.spec.ts`).

**Проверка Playwright (запрос заказчика «проверить это при помощи Playwright»).**
Полный набор e2e прогнан на своём vite :5273 из ветки `s8r/fix-tails-ui`
(стенд заказчика :5173/:8000 не трогался): **176 passed / 6 failed / 1 skipped**
из 183 за 12,4 мин. Падают те же 6 сценариев
`auth-hardening.spec.ts` — они по условию работают на ЖИВОМ backend с
изолированной БД `data/e2e_auth.db` и `PW_ORIGIN`, равным адресу фронта; своего
backend я не поднимал (команда с экспортом `SECRET_KEY`/`ENCRYPTION_KEY` была
отклонена permission-правилом, обходить не стал). Эта область закрыта backend-
тестами: 534 теста auth/ws/admin зелёные в полном прогоне, включая новый тест
единого декодера токена из 017.

**Сплошная сверка остатка backlog (2026-10-13).** После закрытия карточек
прошёлся по всем незакрытым заголовкам файла — ни одна не оказалась работой,
которую можно сделать сейчас:

- семь карточек раздела «Переносится в Sprint_8_Review» (`S8R-SR-*`) закрыты
  ещё волнами W4/W5, заголовки просто не обновляли: `PERF-BASELINE` —
  `pytest-benchmark` + `tests/test_performance/test_benchmarks.py` +
  `Sprint_8/perf_baseline_w5.md`; `COV-MARKET-DATA` 83 % и `COV-STRATEGY` 97 %
  (цель 80 %); `MULTICURRENCY-TOGGLE` — переключатель RUB/USD, mock-курс
  заменён курсом ЦБ РФ 2026-08-10; `PLAYWRIGHT-NIGHTLY-RERUN` —
  `playwright-nightly.yml` на моках; `TEST-EVENT-DELIVERY-FIX-FIXTURES` — в
  тестах доставки событий не осталось ни `skip`, ни `xfail`;
  `DOCKER-COMPOSE-VALIDATE` — сборка на Mac mini, это заказчик (pre-deploy A8);
- `S8R-FIX-040` закрыта S8R-AUDIT-086 (outbox по владельцу входил в её рецепт);
- `S7R-SESSION-RERUN-PAYLOAD-BROKEN` закрыта в S8: «Запустить заново» передаёт
  режим, счёт и таймфрейм, конфликт пары показывается до запуска (визуальные
  пункты — UI-чеклист S8.31, ждут живого стенда);
- `S8R-FIX-078` заблокирована апстримом: `protobuf` не поднять, пока SDK
  T-Invest требует `<5.0.0` (`ResolutionImpossible`), Dependabot #38 закрыть
  как неприменимый.

Обе находки ревью заведены карточками **S8R-FIX-091** (состав бумажного equity)
и **S8R-FIX-090** (провал бэктеста через БД) с отметкой «закрыта в том же
цикле» — чтобы в истории остался не только фикс, но и класс ошибки.

**Такт 2 — решения заказчика 2026-10-13 (после мержей) и их исполнение.**

- **021 (AAD) — «делать сейчас, с бэкапом».** Работы не потребовалось:
  привязка шифртекста к записи закрыта ещё S8R-AUDIT-042
  (`encrypt(context="broker_account:<id>")`, `is_bound`, CLI
  `rebind-encrypted-secrets`), и все писатели в `app/` контекст передают.
  Снял копию рабочей БД через `sqlite3.Connection.backup` (gotcha-19), прогнал
  на ней `python -m app.cli rebind-encrypted-secrets` — **«All secrets are
  already bound to their records — nothing to rebind»**. Копия и её снимки
  удалены, рабочая БД не менялась, стенд не гасился. Правило закреплено
  тестом-инвариантом: ни одного `encrypt()` без `context=` в `app/` (мутация
  «писатель забыл контекст» — красная). Остаётся опцией заказчика
  `rotate-encryption-key`: он закрывает старые снимки БД, в которых ключи
  лежат ещё без привязки (гайд §6а).
- **023 (FK `audit_log`) — «оставить как есть».** Поведение зафиксировано
  тестом (удаление пользователя с аудитом обрывается, журнал цел), схема БД не
  меняется. Карточка закрыта решением.
- **008 (отсечка сплита) — «отсечка по ex-date».** Проверка показала, что она
  действует с S8R-AUDIT-091: `process_split` берёт только позиции с
  `opened_at < 00:00 МСК` даты сплита. Различие с дивидендами не в наличии
  отсечки, а в источнике даты — у сплита это ISS `tradedate` (первый день в
  новом масштабе), у выплат ex-date считается от даты реестра по календарю.
  Записал это в ФТ §6.4, карточку закрыл.
- **018 (третий пункт) — сведено.** Все три пути уведомления о сессии идут
  через `_notify_session`. Копии расходились в обработке ошибок: у расхождения
  позиций её не было вовсе, и сбой канала доставки обрывал проход сверки ПОСЛЕ
  записи пометки. 7 тестов, мутации «сбой доставки рушит вызывающего» и
  «уведомление без владельца» — красные.
- **Dependabot — «#53 смержить, #54/#55 разобрать и починить».** #41 закрыт как
  выполненный (react-table 9.3.1 пришёл с S8R-FIX-079; автобамп падал именно
  потому, что v9 — другой API), #38 — как неприменимый (SDK T-Invest держит
  `protobuf<5.0.0`, `ResolutionImpossible`). Причины падений: у **#55** это
  были те самые 20 ошибок `react-hooks/set-state-in-effect` — карточка
  S8R-FIX-080, уже смержена, значит PR нужно пересобрать от свежего develop;
  у **#54** Dependabot бампит пины в `pyproject.toml`, но не умеет
  пересобирать `requirements*.lock` с хэшами (S8R-AUDIT-050), и
  `verify_install` видит расхождение — нужен прогон `scripts/lock_deps.sh`.
- **`docker compose build` (pre-deploy A8)** — на этой машине docker не
  установлен; остаётся на Mac mini у заказчика.

**Dependabot закрыт целиком (2026-10-13).** #53 смержен; #41 и #38 закрыты как
выполненный и неприменимый; #55 Dependabot закрыл сам (плагины eslint пришли с
#58); #54 закрыт как замещённый — те же пять пинов сделаны в **PR #61** вместе
с пересборкой lock (`scripts/lock_deps.sh`): sqlalchemy 2.1.4, cryptography
50.0.2, openai 3.26.1, ruff 0.16.10, pypdf 6.20.0, транзитивно truststore
0.10.5. Набор проверен на ОТДЕЛЬНОМ venv (общий не трогал, чтобы стенд не
поехал на новых версиях до мержа): `pip check` чист, `verify_install` OK,
полный pytest **5839 passed / 1 skipped**. Попутно починен сам тест
`test_sdk_sha_has_no_copies_outside_pyproject_and_lock`: он пропускал venv по
ИМЕНИ каталога (`.venv`), а установленный пакет кладёт sha SDK в свои
`METADATA`, поэтому любой второй venv рядом ронял проверку; теперь окружение
определяется по `pyvenv.cfg`, мутация «третья копия sha» по-прежнему красная.

**⚠️ CI GitHub Actions не запускается с 12:23 UTC 2026-10-13.** Все джобы падают
за 2–3 с с аннотацией «The job was not started because recent account payments
have failed or your spending limit needs to be increased»: прогоны #443, #444,
#445, включая CI на `develop` после мержа #59 и оба открытых PR (#60, #61).
Нужен Billing & plans у заказчика; по правилу «мерж при зелёном CI» мерж этих
двух PR оставлен ему. Локальные гейты по обоим зелёные.

**Стенд заказчика** (:8000 / :5173) на момент 12:30 UTC не слушает порты,
процессов `uvicorn`/`vite` в системе нет. Исполнитель его не останавливал —
гасил только свой dev-сервер на :5273 (по PID из `lsof` этого порта). Поднять:
`cd Develop && sh scripts/start.sh`.

**Запуск стенда после перезагрузки Mac mini (2026-10-13, по просьбе заказчика)
вскрыл шесть дефектов в самих скриптах запуска — карточка S8R-FIX-092, PR #62.**
Стенд и туннель поднялись и работают: backend :8000, frontend :5173, сессия #2
(LKOH 5m) восстановлена, туннель `https://unedited-floral-hardcover.ngrok-free.dev`,
Telegram webhook зарегистрирован, backend отвечает через туннель.

Что мешало и как закрыто:

- `sh scripts/start.sh` падал «syntax error near unexpected token `>`»: скрипт
  bash-only (process substitution для дублирования лога), и сообщение не
  называло ни интерпретатор, ни то, что скрипт исполняемый. Оба скрипта теперь
  перезапускают себя bash'ем — это важно не только для удобства: `sh` в Docker
  и на Ubuntu — dash, он не понимает ни process substitution, ни массивы.
- ngrok не поднимался из-за `HTTP_PROXY`/`HTTPS_PROXY` в профиле оператора:
  бесплатный план агента через прокси не работает (`ERR_NGROK_9009`). Скрипт
  печатал только «ngrok не запустился за 10 секунд», причина оставалась в
  `/tmp/ngrok.log`. Теперь прокси снимается для агента, причина печатается с
  прямой подсказкой, а `NGROK_KEEP_PROXY=1` оставлен как осознанный выбор.
- Webhook регистрировался, даже когда backend не запущен — Telegram стучал в
  туннель, за которым никого нет, и сообщения терялись молча. Теперь проверка
  `/api/v1/health` до регистрации и проверка пути через туннель после.
- Отказ Telegram возвращал код 0: сбой выглядел успехом и для оператора, и для
  автоматизации. Теперь ненулевой код на любом сбое.
- Настройки читались `export $(grep … | xargs)` — значение с пробелом,
  кавычкой или `#` разрывалось. Переведено на тот же парсер, что в `start.sh`.
- Агент ngrok умирал вместе с терминалом, хотя скрипт — разовая операция.
  Теперь `nohup` + `disown`; проверено, что туннель переживает закрытие таба.

Добавлен режим `start.sh --check` (только preflight, ничего не поднимает) и 14
тестов со стабами ngrok/Telegram/backend (`test_start_scripts.py`) — раньше
скрипты запуска были непокрыты вовсе. Мутации «прокси не снимается», «backend
не проверяется», «отказ Telegram с кодом 0», «guard убран» — красные.

**Мерж по локальным гейтам (решение заказчика 2026-10-13).** CI у GitHub
недоступен до следующего месяца (план не позволяет частые прогоны), поэтому
#60, #61 и #62 смержены на локальных гейтах. Перед мержем собрано ОБЪЕДИНЁННОЕ
состояние трёх ветвей и прогнано целиком: ruff 0, mypy 0 (207 файлов),
bandit 0, pytest **5863 passed / 1 skipped**, фронт typecheck/lint/build 0,
vitest 1376 passed. Падает только `test_backup_cli` — CLI отказывается делать
restore, пока на :8000 живёт поднятый стенд (известный артефакт).

Этот прогон и оправдал себя: он поймал регрессию от правки скриптов, которой не
было видно на отдельной ветке. Тест S8R-FIX-003 ищет в `start.sh` подстроки
`umask 077` и `tee -a logs/dev.log` и требует, чтобы umask шёл раньше, — а
комментарий bash-guard'а цитировал команду записи лога дословно, и цитата в
шапке файла стала «первым tee». Починено двумя правками: комментарий больше не
цитирует команду, а тест перестал учитывать комментарии (он сторожит порядок
команд, не текст пояснений). Мутации «umask → 022» и «umask после tee» — обе
красные. **develop `e527a05`, открытых PR не осталось.**

**Осталось у заказчика:** решение по AAD-перешифровке (021) и по FK `audit_log`
(023); нужна ли сплиту отсечка по дате (008, ФТ §6.4); свести три копии
резолва владельца в уведомлениях (018); правка `app/cli/secrets_rotation.py`
(закрыта permission-правилом); мерж PR #56/#57/#58 и ветки `s8r/fix-tails-ui`;
10 PR Dependabot.

**2026-10-12 (ночь) — S8R-FIX-080 закрыта целиком: 20 из 20 мест правил React-компилятора.**

Ветка `s8r/fix-080-react-compiler-rules`, worktree `wt-s8r-react` (от `892fbe7`, свои
`node_modules`), **не закоммичено** — ждёт команды заказчика. Версии плагинов возвращены на
`^7.1.1` / `^0.5.7`, `pnpm lint` зелёный.

**Главное про само правило (чтобы не тратить время заново).** `react-hooks/set-state-in-effect`
смотрит интерпроцедурно: если из тела эффекта вызывается функция, которая где-то внутри зовёт
`setState`, это ошибка — независимо от того, что присваивания стоят после `await`. То есть
«вынести загрузчик рядом и звать его из эффекта» правило НЕ удовлетворяет (проверено на
`AISettingsPage`, `RiskSettingsPage`, `VersionsHistoryDrawer` — сначала сделали именно так, lint
остался красным). Рабочая форма одна: загрузчик **чистый** (возвращает данные, состояние не
трогает), а присваивания лежат в самом эффекте, лексически после `await`, внутри инлайновой
async-IIFE. Сброс состояния для кнопок «Повторить»/«Обновить» остаётся в обработчиках — там
правило не действует.

**Вторая форма — «update state during rendering»** (сравнение с предыдущим значением) для
сброса при смене пропа. Две ловушки, обе поймались прогоном, не рассуждением:
- сравнивать можно только **стабильное** значение. В `LaunchSessionModal` сравнение ссылки на
  `userStrategies` из селектора стора дало «Too many re-renders» — селектор отдаёт новый массив
  на каждый вызов (в тестовых моках — всегда, и настоящий zustand identity не обещает). Сломалось
  8 тестовых файлов сразу; лечится сравнением **содержимого** (ключ из тикеров инструментов);
- во время рендера **нельзя трогать сторонний стор** (React 19 — ре-рендер чужого компонента).
  Поэтому в `SessionDashboard` местное состояние ушло в рендер, а `clearError` остался эффектом —
  но через **`useEffectEvent`**, не через deps: `clearError` из селектора нестабилен, в deps он
  запускал бы эффект на каждый рендер и затирал только что показанную пользователю ошибку
  (ровно то, что чинила S8R-FIX-052). Ref «последнего значения» тоже не годился — его запись во
  время рендера и была одной из 20 находок.

**Разбор по файлам.** Пять мест закрыты в прошлой сессии (`BalanceWidget`,
`ActivePositionsWidget`, `HealthWidget`, `TimeframeSelector`, `CustomToolbox`), пятнадцать —
сейчас:
- *выборка на монтировании:* `AISettingsPage` (чистая `fetchProvidersData` + `reloadProviders`
  для обработчиков), `RiskSettingsPage` (`fetchConfig` возвращает `{gen, data, error}` —
  поколения запросов сохранены), `VersionsHistoryDrawer` (чистая `fetchVersions`, сброс
  «загрузка» — сравнением по ключу `opened ? strategyId : null`; побочно пропал кадр со старым
  списком и неактивным спиннером);
- *сброс при смене пропа:* `SparklineWidget` (спиннер стал начальным состоянием —
  `useState(() => Boolean(selectedTicker))`, иначе первый кадр показывал «Нет данных за 24 ч»),
  `ProfileSettingsPage`, `SessionDashboard` (см. выше), `ChartPage` (сброс WS-канала переехал в
  сравнение `channelKey = ticker:timeframe` — на один коммит раньше, чем делало тело эффекта;
  синхронность тут обязательна, иначе живые свечи старого ТФ попадают в массив нового и Daily
  «мигает»);
- `StrategyEditPage` — самая крупная перестройка. `status` и `updatedAt` оказались чисто
  производными (их больше никто не выставлял) и стали обычными вычислениями; имя (поле
  редактируемое) применяется сравнением в рендере; **наполнение формы переехало в путь
  загрузки**: `strategyStore.fetchStrategy` теперь возвращает стратегию (`Promise<StrategyDetail
  | null>`), и редактор наполняется её ответом, а не реакцией отдельного эффекта на появление
  `currentStrategy` в сторе. Render-phase для этого блока не годился: внутри и `setDescription`
  из `useAIChatStore` (сторонний стор), и чтение `initialLoadDoneRef`/`templateAppliedRef`
  (ref'ы в рендере — другое правило). Побочное улучшение, которое стоит знать: редактор больше
  не наполняется оставшейся в сторе стратегией прошлой страницы (раньше при повторном заходе
  форма сначала заполнялась чужими данными, и `initialLoadDoneRef` закрывал повторное
  наполнение свежими);
- `CandlestickChart` — 4 × `react-hooks/refs` + 1 × `immutability`. Все четыре ref'а «последнего
  значения» (`sequentialModeRef`, `onChartReadyRef`, `candlesRef`, `rebuildMarkersRef`)
  обновляются в одном **`useLayoutEffect`**. Почему именно layout: data-эффект №2 читает
  `rebuildMarkersRef.current()` после `setData`, а layout-эффекты выполняются раньше любого
  passive-эффекта коммита — обычный `useEffect` отдал бы коллбэк на коммит старше. Вызов
  `useTradeMarkers` при этом поднят выше эффектов: `immutability` требует, чтобы запись в ref
  стояла лексически ДО читающего эффекта. Для хука перенос безопасен — его собственный эффект не
  рассчитывает на готовый график, а дожидается серии циклом `for (let i = 0; i < 40; i++)`.

**Гейты ветки:** typecheck 0, build 0, lint 0, **vitest 1341 passed** (baseline 1335 + 6 новых).
**Шесть новых тестов, четыре мутации поймано:** ключ канала без таймфрейма → падение;
`clearError` в deps → 2 падения; `loading` без начального значения → падение; описание из стора
вместо ответа `fetchStrategy` → падение. Файлы: `SessionDashboard.sessionReset.test.tsx` (новый,
3 теста), `ChartPage.channelReset.test.tsx` (новый), по одному тесту в `SparklineWidget.test.tsx`
и `StrategyEditPage.test.tsx`. `backend/` не затрагивался.

**Скриншоты (требование карточки) — `s8r-evidence/s8r-fix-080/`.** Снимались на временном стенде
**без БД и без пароля заказчика**: свой vite на :5273 из worktree + Playwright с моками
`e2e/fixtures/api_mocks.ts` (`injectFakeAuth`) — рецепт из `playwright.config.ts` для CI.
Дашборд: все четыре виджета с данными (Баланс 1 372 500 ₽ со sparkline и дневной дельтой,
«Состояние систем», «Активные позиции», SBER 295,1 +5.39 %). График: Д (60 свечей, overlay OHLC,
объём, источник T-Invest) и 1ч **после переключения таймфрейма** — свечи, объём, sequential-index
режим, новый overlay. Два артефакта мока, не продукта, стоит записать: точка истории баланса —
`{ts, total_value, trading_pnl}` (форма `{date, total}` ронял виджет на `toLocaleString`, поймал
ErrorBoundary), и свечи для intraday обязаны лежать в 07:00–20:59 UTC — `ChartPage.filteredCandles`
режет внесессионные, и набор «всё на T00:00» давал пустой график на 1ч. Временные vite, спека и
конфиг Playwright удалены, порт 5273 свободен, стенд заказчика :8000/:5173 не затрагивался.

**`/code-review` по правилу CLAUDE.md** (правки в `components/trading/`, PR > 10 файлов) —
xhigh, 9 находок, 5 починены в том же цикле, 1 закрыта тестом, 3 вынесены карточкой.

- **Главная (починено).** Наполнение формы редактора зависело только от успешного ответа
  `fetchStrategy`: при недоступном backend редактор оставался пустым, хотя в сторе лежала ЭТА же
  стратегия, — и «Сохранить» записало бы версию с пустыми блоками. Добавлен откат на кэш стора со
  сверкой `id` (чужую стратегию не берём), доступ к стору — через `useEffectEvent`. Два теста,
  обе мутации (снять сверку `id`, снять откат) поймались.
- **Регрессия от самого фикса (починено).** Перенос `useTradeMarkers` выше эффектов поменял
  порядок passive-эффектов: эффект загрузки сделок стал выполняться ДО создания графика, видел
  `seriesRef.current === null` и уходил в паузу 250 мс — маркеры опаздывали на четверть секунды
  при каждом открытии. В `useTradeMarkers` первые 8 итераций ожидания сделаны по 10 мс, общий
  бюджет ~10 с сохранён.
- **Починено также:** `setDescription` убран из зависимостей эффекта, который сбрасывает
  `initialLoadDoneRef` и перечитывает стратегию (нестабильная ссылка затирала бы несохранённые
  правки; тот же приём, что для `clearError`); сняты две недостижимые ветки «тикер пуст» в
  `SparklineWidget`; дописаны ФТ и ТЗ.
- **Закрыто тестом:** `CandlestickChart.refSync.test.tsx` — data-эффект №2 зовёт `rebuildMarkers`
  текущего рендера. **Поправка к собственному обоснованию:** мутация `useLayoutEffect` →
  `useEffect` тест НЕ роняет — блок объявлен выше читающего эффекта, а passive-эффекты идут в
  порядке объявления, так что обычного эффекта тоже хватило бы. `useLayoutEffect` оставлен как
  страховка от перестановки эффектов в файле, комментарий в коде переписан по факту.
- **Вынесено карточкой S8R-FIX-089** (low, M, нужно решение): приём «загрузка данных в эффекте»
  теперь вручную повторён в 8 компонентах (правило запрещает звать загрузчик из тела эффекта, но
  не запрещает спрятать весь приём в свой хук). Копии уже расходятся: `setError(null)` при успехе
  делают две из восьми. Рефакторинг задевает дашборд, настройки и редактор сразу — на фазе
  приёмки это отдельный цикл, не попутная правка. Туда же — дублирующийся блок «разобрать ответ»
  в `RiskSettingsPage`/`AISettingsPage` и пересборка ключа подсказок в `LaunchSessionModal`.
- **Замечено, не правил:** в ФТ нет карточек **082–085** прошлого цикла, включая видимые
  пользователю 084 (`/balance` по всем счетам) и 085 (карточка сессии после «Стоп»). Это работа
  другого цикла — формулировки стоит согласовать с заказчиком.

Гейты после правок ревью: typecheck/build/lint 0, **vitest 1344 passed** (baseline 1335 + 9
новых тестов).

**Что ждёт заказчика:** команда на коммит ветки `s8r/fix-080-react-compiler-rules` и PR; мерж
PR #56 (085) и #57 (086); пуш репозитория документации (коммит `1479798` не запушен); обновление
стенда до `892fbe7` (`git pull --ff-only`, после чего проверяема телеграм-половина S8.49 п.2);
Dependabot #53/#54/#55; карточки 087 и 088 не исправлялись.

**2026-10-12 — причина «брокер не отвечает» найдена; прогон на живом брокере; мерж #50/#51.**

**Главное за день: «молчание брокера» — это резолвер DNS внутри gRPC, а не брокер.**
Симптом держался с ~05.10 и блокировал четыре пункта чеклиста; «виноватыми» успели побывать
выходные, закрытая биржа, sandbox-API и Proxifier. gRPC не пользуется системным резолвером —
по умолчанию работает c-ares, и на этой машине он разрешал имя T-Invest за **10–11 с** при
дедлайне unary-вызова 10 с: каждый вызов умирал за секунду до готовности канала. Стрим свечей
жил, потому что мультиплексор держит один долгий канал, а адаптер создаёт новый на каждый вызов.
Замеры одного процесса: TCP 0,01 с · TLS через openssl мгновенно · UDP к DNS 0,01 с ·
gRPC-канал **по IP 0,08 с** · **через DNS 11,15 с** · с `GRPC_DNS_RESOLVER=native` **0,04 с**.
Проверено и отвергнуто: подмены трафика нет (цепочка — подлинная НУЦ Минцифры), Proxifier TLS
не перехватывает, корень НУЦ в бандле стенда есть (сверено по SHA-256 `d26d2d02…`, 131 сертификат).
После перезапуска стенда с `GRPC_DNS_RESOLVER=native` всё ожило за 6 секунд: `broker_candles_loaded
count=203`, балансы и позиции 200 OK, футер «Брокер: на связи».

**Прогон на живом брокере (решения заказчика 7-а, 8-б, 9 «все пункты», 10-б).**
- **S8.49 п.1 ✅** — четыре карточки с реальными числами (Всего 1 378 549,75 ₽ и т. д.).
- **S8.49 п.2 ✅ (API)** — заявка SBER 1 лот по **258 ₽** на счёте #3 (`d05ec40c-…`):
  `blocked` 0 → **258,129 ₽** (258,00 + 0,129 комиссии 0,05 %), `available` 99 993,957 → 99 735,828,
  `total` не изменился; после отмены обе величины вернулись. Те же числа отдаёт
  `GET /broker/accounts/balances`. **Отступление:** 250 ₽ брокер отклоняет —
  `[30099] price is outside the limits`, дневной коридор 257,86–306,15 ₽; лот SBER = 1 акция
  (значит «100 лотов» дали бы 25 800 ₽, а не 250 000). **Телеграм-половина недостижима** —
  S8R-FIX-084.
- **S8.53 п.3 ✅** — у paper-сессии #1 была открыта Long 11 лот @ 284,58; «Стоп» →
  `risk_monitor_trade_closed exit_price=284.39 pnl=-2.09 reason=session_stop trade_id=53` →
  `telegram_notification_sent` ×3, доставку заказчик подтвердил. Вариант «сессия на паузе»
  не покрыт — такой сессии на стенде нет. Попутно найден S8R-FIX-085.
- Осталось: S8.51 п.2 (нужна sandbox-сделка без цены входа), S8.54 п.4, раздел S8.56 (нужен код #50 на стенде).

**Раздел S8.56 пройден целиком (9 из 9), на копии данных стенда.** Свой backend :8100 поверх копии
рабочей БД (backup API) + временный vite :5273 с прокси на него; пароль заменён **только в копии**,
рабочая БД не менялась. Состояние населено осознанно: сделке 42 проставлены `close_reason =
closed_outside_terminal`, `exit_price`/`pnl` = NULL (ровно то, что теперь порождает код), сессии #3 —
пометка расхождения; сделка 43 оставлена контролем и подтвердила, что legacy-закрытия пометки не
получают. Ключевое: **сверка шла против живого sandbox-счёта**, не против мока —
`manual_reconcile_done result=match` и `result=mismatch` в логе. Доказательства: кнопка есть только у
остановленной брокерской сессии (у paper — только «График» и «Удалить»); при совпадении плашка
исчезает без перезагрузки и фильтр списка не сбрасывается, в журнале действий `session.reconciled
{"mismatch_cleared": true}`; при расхождении текст обновляется на фактический («SBER: у брокера
позиции нет, у нас открыто 3 шт (сделки №54)») вместе со временем пометки; при `TINVEST_UNARY_TIMEOUT_SEC
=0.01` — «Не удалось сверить позиции: брокер не ответил…» и пометка на месте; удаление счёта — 409
с нужным текстом, после сверки — 204; в журнале сделок «P&L не определён» с подсказкой «Позиция
закрыта вне терминала…» и комиссией 12,50 ₽; payload уведомления — `pnl_text = 'не определён (позиция
закрыта вне терминала)'`, `pnl = None`; в CSV — раздел «Сделки без подтверждённой цены» со строкой
`05.08.2026;SBER;buy;песочница;33;295.02;—;12.50` при `Комиссии:;0` (подтверждает решение по вопросу 5);
балансы и `/health` до и после сверки совпали посимвольно. **Оговорки:** работающей sandbox/real-сессии
на стенде нет — отсутствие кнопки проверено на paper (остановленной и активной); доставленное
Telegram-сообщение не снималось (стенд копии с `DEV_MODE=true`, чтобы не слать заказчику тестовые
уведомления) — проверен production-код сборки payload. Временный стенд погашен, копия БД и
`vite.config.s856.ts` удалены, порты свободны, стенд заказчика цел.

**Остаток UI-чеклиста в зоне цикла (S8.47–S8.56): 17 пунктов.** Из списка прогона открыты
**S8.51 п.2** (нужна sandbox-сделка без цены входа) и **S8.54 п.4** («Закрыть позицию и пауза»);
S8.49 п.2 помечен `[~]` — API-часть пройдена, телеграм-часть станет проверяемой после мержа PR #52
(S8R-FIX-084). Остальные 14 — вне списка прогона (аутентификация, риск-менеджмент, редактор стратегии).

**2026-10-12 (вечер) — 085, 086 и остаток прогона; работа разделена на три потока по просьбе заказчика.**
Два независимых потока отданы субагентам (фикс 085 и документация 086), прогон чеклиста оркестратор
вёл сам — он требует браузерной панели, которая одна.

**S8R-FIX-085 — сделано** (ветка `s8r/fix-085-session-card-stats`, worktree `wt-s8r-085`, симлинк
`node_modules`). Корень оказался на backend: `total_trades`/`winning_trades`/`losing_trades`/
`current_pnl`/`active_position_*` — не колонки `trading_sessions`, их считает `_fill_session_card_data`
только в `get_session`/`get_sessions`, а `PATCH /sessions/{id}/stop` отдаёт
`SessionResponse.model_validate(session)` с дефолтами схемы (0/null) — спред ответа поверх карточки их
и затирал. Решение: `mergeSessionKeepingStats` (переносим всё, кроме вычисляемого) **плюс** перезапрос
ОДНОЙ сессии `getSession(id)` — остановка реально меняет статистику, и «сохранить прежние числа» дало
бы правдоподобную, но неверную карточку; `fetchSessions()` не зовём, это регрессия S8R-FIX-070.
Сбой перезапроса не поднимает `error`: остановка-то удалась. 5 новых тестов, включая проверку, что
«Сделок: 0» не мелькает ни на один кадр (через `store.subscribe`). RED до фикса — 4 failed.
Мутации: снятие защиты статистики → 2 failed, снятие перезапроса → 2 failed. Гейты: typecheck 0,
build 0, lint 0, **vitest 1340 passed** (проверено оркестратором независимо; одно падение
`BacktestResultsPage` в полном прогоне — флейк по таймауту под нагрузкой, отдельным прогоном 6/6).
**Поправка к baseline:** 1335, а не 1332 — число выросло на PR #52, в промптах было устаревшее.

**S8R-FIX-086 — сделано.** Новый раздел **§7а** гайда деплоя «Обновление dev-стенда (голый venv,
без Docker)»: когда нужен полный путь, почему падает наивный `pip install -r requirements.lock`
(lock с хэшами + git-зависимость SDK → `--require-hashes` отвергает файл целиком), полная
последовательность по рецепту `ci.yml`/`Dockerfile.backend`, перезапуск с миграциями и штатность
сообщения `requires tinkoff, which is not installed` (с предупреждением, что код возврата ≠ 0 рвёт
цепочку `&&`). Короткая версия — в `Develop/CLAUDE.md` рядом с гейтами. Побочно замечено:
в «Ссылках» гайда битый путь `../Develop/INSTALL.md` (файл в `Develop/backend/`) — не правили.

**S8.51 п.2 (9-в) ✅** — на сделке с `entry_price IS NULL` production-код сборки payload даёт
`pnl_text = 'не определён (брокер не сообщил цену входа)'`, `pnl = None`, `commission = '7.25'`.
Причина выбрана верная, не «закрыта вне терминала»: приоритет в `_undetermined_text` различает случаи.

**S8.54 п.4 (9-д) — наполовину, вторая половина оказалась дефектом.** Главное работает: активная
sandbox-сессия с открытой позицией, закрытие провалилось (`close_all_positions_done count=0 failed=1`)
→ сессия осталась `active`, **пауза не поставлена** — ровно то, ради чего делался код-ревью р.4 п.7
карточки 052. Но причина пользователю не показана: `POST /close-all` вернул
`{"result":"not_closed","reason":"","error_code":null}` — пусто, модалка закрылась, уведомления нет.
Корень в `engine.py:~8014`: `getattr(close_exc, "detail", None) or str(close_exc)` — исключение без
текста даёт пустую строку. Путь общий, не особенность сценария. Заведена **S8R-FIX-087** (medium),
пункт помечен `[~]`.

**Заведена S8R-FIX-088** (medium, находка исполнителя 085): у `pauseSession`/`resumeSession` тот же
дефект, что у 085, но хуже — карточка теряет не только статистику, но и открытую позицию
(«Нет позиции»), и именно в момент, когда позиция остаётся без защиты SL/TP.

Попутно подтверждено: восстановление сессии при старте с мёртвым брокером честно ставит её на паузу
(S8R-AUDIT-009). Временные стенды погашены, копии БД и конфиги удалены, стенд заказчика цел.

**Мерж.** #50 → `develop` `7e167b9`; #51 → `a854e13`; **#52 (082/083/084) → `892fbe7`** (CI 5758 passed, сходится с локальным 5756 + skipped + `test_backup_cli`). #51 обновлён от develop (`8a4b104`) — обе ветки
правили `test_close_stopped_session_broker_check.py`, поэтому не «как есть», а с прогоном:
локально **5747 passed / 1 skipped**, единственное падение — `test_backup_cli` (артефакт занятого
:8000, стенд гасить нельзя). CI #51: frontend/nginx/security ✅, backend считается.

**Новые карточки 081–085.** 081 (канал на каждый вызов) — решение заказчика **(б)**: ограничиться
083 + 082, торговый контур перед деплоем не переписывать. 082, 083, 084 — сделаны в ветке
`s8r/fix-082-084-grpc-dns-balance` от `develop` `7e167b9`: новый модуль `grpc_env.py` с
`ensure_grpc_dns_resolver()` в lifespan рядом с корнями (значение оператора не трогается);
сообщение таймаута больше не утверждает, что до брокера дошли (+ подсказка в лог, поправлены
докстринг `BrokerTimeoutError` и текст в `sandbox_recovery.py`); `/balance` показывает **все**
активные счета блоками (решение заказчика **084-а**), причём сбой одного счёта больше не прячет
остальные — раньше любое исключение уводило в общий `except` и вместо балансов приходило одно
«❌ Ошибка». 9 новых тестов, 3 мутации пойманы, ruff/mypy/bandit 0/0/0 (207 файлов).

**Решение заказчика по правилам (2026-10-12):** ордера на sandbox-счёте #3 исполнитель выставляет
сам, параметры согласуются в чате; реальный счёт #2 и будущие реальные счета — только заказчик;
токен не читается (ключ расшифровывается в памяти через `decrypt_broker_key`). Записано в
корневой `CLAUDE.md` и в память проекта. **Docker (вопрос 6):** на машине нет, ставить будем
после завершения работ и правок — A2/A8 остаются открытыми до этого.

**2026-10-10 (ночь) — цикл по решениям заказчика: 077 (а), 070 (а), 071 (а), Dependabot (в), лицензии (а).**
Две параллельные ветки от `origin/develop` `6b197c6`, оба worktree свои:
`s8r/fix-070-071-pnl-reconcile` (`wt-s8r-fixes`, общий venv, symlink node_modules) и
`s8r/deps-bump-2026-10` (`wt-s8r-deps`, **свой** venv и свои node_modules — чтобы не тронуть
работающий стенд заказчика на :8000/:5173).

**S8R-FIX-071 (а).** Новая колонка `live_trades.close_reason` (миграция `fd3492233bc0`, nullable
String(40), round-trip тест) и три константы причин (`closed_outside_terminal`,
`…_by_user`, `sandbox_account_reopened`) в `UNDETERMINED_CLOSE_REASONS`. Колонка, а не вывод из
полей: состояние «цена входа есть, цены и ордера выхода нет» принадлежит ещё и legacy-закрытиям
paper до W8 — вывод по полям переписал бы историю (журнал, налоги, оценку CB по старым строкам).
`_close_trade_without_order` больше не подставляет `exit_price = entry_price` и `pnl = 0`:
`pnl`/`pnl_pct` NULL, причина записана, `DailyStat` получает `trades_closed +1` и
`realized_pnl = −commission` без выдуманного нулевого `pnl`. Признак `is_pnl_undetermined`
учитывает причину; `unpriced_cb_losses` читает колонку (иначе CB не увидел бы убытка — проверено
тестом: оценка −52 500 ₽ срабатывает дневным лимитом); налоговый раздел «без подтверждённой цены»
подхватывает сделку автоматически; в payload `trade.closed` — новый текст
`PNL_UNDETERMINED_OUTSIDE_TEXT` («не определён (позиция закрыта вне терминала)»), в журнале UI —
своя подсказка вместо «брокер не сообщил цену выхода». Шесть тестов старого поведения
(`pnl == 0`, `exit_price == entry_price`) переписаны под решение заказчика.

**Миграция проверена на копии рабочей БД** (копия — backup API, источник `mode=ro` без
`immutable`, gotcha-19; рабочая БД не менялась, копия удалена): `606214dc5ccf → fd3492233bc0`
проходит, `alembic check` чистый, `integrity_check` ok; у всех 40 существующих сделок
`close_reason` = NULL, две исторические закрытые сделки с `pnl = 0` остались как были — задним
числом признак на них не распространяется (ровно то, ради чего выбрана колонка, а не вывод из полей).

**S8R-FIX-070 (а).** `POST /api/v1/trading/sessions/{id}/reconcile` → `TradingService.reconcile_with_broker`
→ `SessionRuntime.reconcile_session_with_broker`: тот же `_reconcile_broker_positions`, но с
`pause_from=()` и явным исходом (`ReconcileOutcome.result`: `match` / `mismatch` / `not_read` /
`no_account`) — возвращаемый `bool` не отличал «сошлось» от «портфель не прочитан», и кнопка
показывала бы молчание брокера как успех. Бюджет — `POSITION_RECONCILE_TIMEOUT_SEC` (молчащий
брокер не держит HTTP-запрос). Коды: 422 paper/без счёта, 409 `session_is_live`,
409 `reconcile_not_done` (пометка остаётся), 409 `account_credentials_missing`.
`BrokerService._assert_account_not_in_use` теперь держит счёт при неснятой пометке у
sandbox/real-сессии (в `sessions[]` — признак `position_mismatch`, в тексте отказа — подсказка про
сверку); прежнее решение «не держать» было обосновано тем, что снять пометку было нечем.
UI — кнопка «Сверить с брокером» в карточке остановленной брокерской сессии (`SessionCard`),
экшен `tradingStore.reconcileWithBroker`. Поле исхода названо `result`, а не `status`: гейт
`test_fix_037` запрещает присваивания `*.status = …` вне единого setter'а статуса сессии.

**Собственный `/code-review` (high) по диффу — 5 находок, 4 исправлены в том же цикле:**
(1) `reconcileWithBroker` звал `fetchSessions()` без параметров — список перечитывался без
фильтров страницы; теперь карточка правится на месте, как в `stopSession` (3 теста стора);
(2) исход `no_account` отдавался как «брокер не ответил» → свой код `account_credentials_missing`
(тест); (3) докстринги обещали 400 там, где проект отдаёт 422; (4) ручная сверка не попадала в
журнал действий → `AUDIT_SESSION_RECONCILED` (`session.reconciled`, тест). (5) **Не фикс, а
следствие, принятое осознанно:** сделка, закрытая вне терминала, теперь без `exit_price`, поэтому
`_build_lots` её пропускает — вместе с её комиссией, которая раньше уменьшала налоговую базу.
Сделка целиком уходит в раздел «Сделки без подтверждённой цены» (вне базы и итогов), как все
прочие сделки «P&L не определён» (S8R-FIX-063). Отражено в ФТ.

**Гейт ветки `s8r/fix-070-071-pnl-reconcile`:** pytest **5747 passed / 1 skipped**, +17 новых
тестов (baseline был 5730 passed); единственное падение — `tests/integration/test_backup_cli.py::
test_backup_create_then_restore_recovers_data`, артефакт среды: CLI отказывается восстанавливать
БД, пока на 127.0.0.1:8000 слушает рабочий стенд заказчика («a server is listening on
127.0.0.1:8000 — stop the backend before restore»), гасить стенд нельзя. ruff / mypy / bandit —
0 / 0 / 0. Фронт: typecheck 0, lint 0, build 0, vitest **1332 passed** (168 файлов).
E2E — **175 passed / 1 skipped** (`s7r-chart-drawings-fix` — штатный `test.skip(CI)`), то есть
ровно baseline, прогон на своём стенде :8100/:5273.

**S8R-FIX-077 (а).** Пункт S8.49 п.2 переформулирован в обеих копиях `ui_checklist_s8.md`:
`blocked` для sandbox проверяется через `GET /broker/accounts/balances` и Telegram `/balance`
(параметры заявки — SBER 100 лотов по 250 ₽), UI-часть — на реальном счёте с торговыми правами,
когда он появится. Решение 2026-04-28 (sandbox только в «Торговле») не отменялось.

**Лицензии (а).** `docs/licenses.md`: статус «требует решения» → «утверждено 2026-10-10 для
self-hosted single-user», с явной оговоркой об обратимости (обязательства возникают при
распространении) и следствием «не углублять связность с `backtrader`».

**Dependabot (в) — применено 9 PR из 11, два вынесены карточками.**
Применены: #33 (`pnpm/action-setup` + SHA-пин в двух workflow), #34 (18 пакетов backend:
`sqlalchemy` 2.0.49→2.1.1, `uvicorn` 0.44→0.54, `alembic` 1.18.4→1.20.0, `ruff` 0.15.10→0.16.9,
`TA-Lib` 0.6.8→0.8.1, `pandas` 3.0.2→3.0.6, `python-telegram-bot` 22.7→22.8, `grpcio` 1.80→1.84,
`pytest` 9.0.3→9.1.1 и др.), #35 `cryptography` 46.0.7→50.0.1, #36 `anyio` 4.13→4.15.1,
#37 `openai` 2.38→3.20, #39 (18 пакетов frontend: `react` 19.2.4→19.3.0, `vite` 8.0.1→8.3.1,
`@playwright/test` 1.59→1.63, `lightweight-charts` 5.1→5.2.1 и др.), #40 `@eslint/js`,
#42 `vitest` 4.1.1→5.0.2, #43 `@types/node` 24→26. Lock-файлы перегенерированы
(`scripts/lock_deps.sh`), `pnpm-lock.yaml` обновлён.
Не применены: **#38 `protobuf` 4.25.9→7.36.2** — физически невозможно (SDK T-Invest требует
`<5.0.0`, апстрим на `main` всё ещё `0.2.0-beta117`): `pip-compile` → `ResolutionImpossible`,
заведена **S8R-FIX-078**; **#41 `@tanstack/react-table` 8→9** — не бамп, а переписанный API
(`useTable`, `create*RowModel`, генерик `TFeatures`; 32 ошибки `tsc` в двух таблицах, поведение
сортировки/пагинации проверяется глазами), заведена **S8R-FIX-079**.
**Что пришлось починить из-за мажоров:** (1) `ruff` 0.16.9 расширил набор правил «по умолчанию»
→ 5329 замечаний (FURB157 2292, DTZ001 551, UP017 396, RUF100 380, I001 352, BLE001 302 …):
набор зафиксирован в `pyproject.toml` (`[tool.ruff.lint] select = ["E4","E7","E9","F"]`) — гейт
не должен молча менять требования при обновлении линтера; расширение набора — отдельная задача.
(2) SQLAlchemy 2.1 сделал генерики `Row`/`Select` вариадическими → 16 ошибок mypy в 4 файлах
(аннотации `Select[tuple[_T]]` → `Select[_T]`, `Row[tuple[A, B]]` → `Row[A, B]`, типизация
распаковки строки в `market_data/service.py`, пропуск NULL `figi` в `price_alert_monitor`).
(3) SQLAlchemy 2.1 запрещает commit при открытом курсоре соседней сессии на том же соединении —
два теста держали одну in-memory БД и закрывали sandbox-сделки параллельно; в проде этого нет
(файловая БД идёт с `NullPool`, S8R-AUDIT-097), поэтому поправлена обвязка тестов
(`SESSION_STOP_CLOSE_CONCURRENCY = 1`), не продакшен-код. (4) `vitest` 5 регистрирует матчеры
jest-dom только через `@testing-library/jest-dom/vitest` → 1358 ошибок TS2339 снялись одной
правкой `src/test/setup.ts`. (5) `TA-Lib` 0.8.1 ставится готовым wheel (в нём своя C-библиотека);
для linux/arm64 и x86_64 wheel'ы cp311 на PyPI есть, значит версия Debian-пакета `libta-lib0` в
образе на сборку не влияет. `bandit` в lock не входит ни до, ни после (CI ставит его отдельно).

**Гейт ветки `s8r/deps-bump-2026-10`:** pytest **5729 passed / 1 skipped** (единственное
падение — тот же `test_backup_cli` из-за занятого :8000), ruff / mypy / bandit — 0 / 0 / 0
(`bandit` поставлен в venv ветки отдельно, как в CI: в lock он не входит ни до, ни после).
Фронт: typecheck 0, lint 0, build 0, vitest **1327 passed** (168 файлов).
E2E — **175 passed / 1 skipped**, то же, что baseline, на всех новых версиях (vite 8.3.4,
react 19.3, Playwright 1.63, vitest 5.0.3). Единственная оговорка: после бампа Playwright нужен
`npx playwright install chromium` (1.63 просит новые бинарники — первый прогон дал 172 мгновенных
падения «Executable doesn't exist»); CI ставит браузеры сам, в гайде деплоя шага не требуется,
образ фронта E2E не гоняет.

**⚠️ Побочный эффект прогона E2E, о котором заказчик должен знать.** Первый прогон E2E шёл против
своего vite на :5273, но его прокси по умолчанию ведёт на :8000 — то есть на **работающий стенд
заказчика**. Шесть auth-тестов (`e2e/auth-hardening.spec.ts`) пытались залогиниться служебной
учёткой `e2e_auth`, которой на стенде нет: по логу стенда — `POST /auth/setup` 4× **403**
(регистрация закрыта после первого пользователя, S8R-AUDIT-033 сработала — пользователь НЕ создан),
`POST /auth/login` 3× 401, затем 429 от лимитера (30 login + 14 setup). **Данные стенда не
изменены:** мутирующих запросов, кроме auth, в логе нет; блокировка «5 попыток / 15 минут» висит
на записи пользователя, а `e2e_auth` в БД отсутствует — учётка заказчика не затронута.
Транзитный эффект: на минуту-две лимитер логина был исчерпан. После этого поднят отдельный стенд
(`scratchpad/e2e_stand.sh`: свой backend :8100 с чистой БД `data/e2e_local.db`, `DEV_MODE=true`,
`MOEX_ENV_FILE=""`, временный `vite.config.e2e.ts` с прокси на :8100, временный
`playwright.e2e-local.config.ts`) — прогон повторён против него. Временные конфиги не коммитятся.


**Обновлено 2026-10-06.** Цикл фиксов аудита S8R завершён: PR #28–#32 (аудит S8R-AUDIT-001…101) и #44 (пакет находок S8R-FIX-030…069 + 005) смёржены в develop (`48f6773`), CI develop ✅. Открыто только то, что ждёт заказчика: S8R-FIX-052 (409 при удалении счёта со stopped-сессией с открытыми позициями), S8R-FIX-058 (фьючерсы в поиске инструментов), вопрос по 063 (консервативная оценка сделки без цены для дневного лимита CB), 10 PR Dependabot #34–#43 (мажорные обновления), лицензии GPL/LGPL (`docs/licenses.md`), `docker compose build` (чеклист A8). Команды заказчику — раздел выше. Стенд заказчика `Develop` — на старом develop `eb769cc`, не трогался.
**2026-10-08 — обновление стенда заказчика (по команде «продолжай по плану»):** стенд не запущен (процессов uvicorn/vite нет); открытых позиций sandbox/real нет (A6 не нужен); схема рабочей БД `a7b8c9d0e1f2`. Пробный прогон миграций на копии (backup) — 15 ревизий до `752ae89f7ec2`, integrity ok, НО `alembic check` → в `ai_provider_configs` нет 4 колонок, которые S8R-AUDIT-004 вернула в модель → после обновления AI ломается. Заведена S8R-FIX-072 (high): идемпотентная ревизия «добавить, если нет»; DEV-FIX-072 в C, ветка `s8r/fix-ai-config-columns`. 072 смёржена (PR #46, develop `68ceb58`). **Стенд обновлён (2026-10-08, по команде заказчика):** снимок рабочей БД `data/terminal.db.bak-2026-10-08-pre-s8r-update` (backup API, integrity ok, 0600); `git pull --ff-only` → `68ceb58` (правка `.env.example` и stash не тронуты); `pnpm install --frozen-lockfile`; `alembic upgrade head` — 16 ревизий → `606214dc5ccf`, `alembic check` чистый, integrity ok; `rebind-encrypted-secrets --server-stopped` — 3 счёта, 2 AI-конфига (снимок `backups/backup_2026-10-08_082441.sqlite`), `chmod 700 backups`; по команде заказчика остановлены sandbox-сессии №3 (счёт #3) и №6 (счёт #4) через `set_session_status` (позиций не было, ордеров нет). Стенд не запускался — запуск за заказчиком; paper-сессии №1/№2 (suspended) восстановятся при старте. Мой снимок `mode=ro` обновил служебный `terminal.db-shm` (данные не менялись). Sandbox-сессий, которые восстановятся при старте, в рабочей БД нет (№3 и №6 остановлены).
**2026-10-10 — состояние на передачу в Claude Desktop:** S8.47 п.3–5 (дисклеймер) — ✅ заказчик на тестовом стенде (на рабочем стенде дисклеймер у `sergopipo` ещё не принят — окно появится один раз). S8.49 п.3: заказчик смотрел виджет «Баланс» на главной («−1 518 ₽») — это не «Счёт»; найден дефект виджета → **S8R-FIX-073** (low; без активных сессий виджет показывает накопленный P&L как баланс; решение заказчика 2026-10-10 — вариант (а): «Нет активных сессий» + P&L отдельной подписью, к исполнению). «Счёт» на стенде — все суммы `null` → «—» (ответ ~40 с). Инструкция по оставшимся ⚙️ — `Sprint_8_Review/ui_check_teststand_guide.md` (значения сверены с API стенда: Win Rate 50 %, средняя −22,05, Суммарный P&L −54,62). `restart_dev.sh` (по команде заказчика): `DEV_MODE="${DEV_MODE:-true}"` (Telegram — `DEV_MODE=false ./restart_dev.sh`), ожидание корректного завершения до 60 с перед `kill -9`. `teststand.sh reset` перезаписывает копию на месте (без накопления копий; после правки не перезапускался — стенд был в работе у заказчика). **Дальше:** заказчик проходит ⚙️ по инструкции и остальные пункты S8.48–S8.54 на рабочем стенде → отметки в обоих ui_checklist → фикс 073 (а), решения по 070/071, Dependabot #34–#43, лицензии (A8) → Docker-сборка на Mac mini (A2/A8) → деплой. После чеклиста — убрать тестовый стенд: `teststand.sh down`, `git -C Develop worktree remove ../../wt-s8r-teststand --force` (там правка vite.config.ts — только proxy на 8100). Незакоммичено в репо Test: этот файл, backlog.md (073), ui_checklist ×2, ui_check_teststand_guide.md, restart_dev.sh — коммит по команде заказчика.
**2026-10-10 — цикл S8R-FIX-074 + S8R-FIX-073 (по команде заказчика «делаем (а) вместе с 073», ветка `s8r/fix-account-page-errors`).** Worktree `wt-s8r-account` от `origin/develop` `68ceb58`, симлинк `frontend/node_modules`, `.env` не симлинковался. RED до правок: 9 vitest + 5 pytest. **074** (`3c92d29`): в `stores/accountStore.ts` один `error`/`loading` на шесть загрузок разнесён на посекционные `{balances,positions,operations,tax}Loading/Error`; `pages/AccountPage.tsx` держит полноэкранный Alert только за `balancesError`, ошибки позиций — в блоке «Позиции», ошибки операций и налоговых отчётов — НАД своими таблицами; тексты через `getApiErrorMessage`, тем же проходом на хелпер переведены оставшиеся 9 файлов (17 замен, паттерн `err.message` вне `apiError.ts` и тестов больше не встречается). **073** (`9485676`): `BalanceHistoryPoint.active_capital` (капитал активных на дату сессий, `total_value = active_capital + trading_pnl`; отдаём явно, чтобы фронт не реконструировал состояние вычитанием) + ветка «Нет активных сессий» с подписью «Накопленный P&L: …» в `BalanceWidget.tsx`. `/code-review` (high) по своему диффу: **2 находки** — ошибка скачивания отчёта убирала таблицу с кнопками «Скачать», ошибка операций убирала фильтр дат и пагинацию (повторить запрос нечем); обе исправлены в том же цикле (Alert над содержимым) и закрыты тестами. **Гейт G1:** pytest 5730 passed / 1 skipped / 0 failed; vitest 1320/0; ruff/mypy/bandit 0/0/0; typecheck/lint/build 0/0/0; E2E 175 passed / 1 skipped (`s7r-chart-drawings-fix` — штатный `test.skip(CI)`) / 0 failed. Новых тестов: 6 backend, 13 vitest, 1 E2E. **Живая проверка** на копии БД тестового стенда (свой стенд :8101/:5274, пароль заменён только в копии `verify.db`, копия удалена): «Счёт» — четыре карточки «—», ошибки в своих блоках с текстом сервера «T-Invest не принял API-ключ…»; главная — «Нет активных сессий» и «Накопленный P&L: −1 518 ₽». **PR #47 смёржен 2026-10-10 — develop `0945edd`**, CI PR ✅ (backend 5731 passed, сходится с локальным 5730 passed + 1 skipped), CI develop ✅ (22m23s). Документы: ФТ §7.3/§19.5 + история версий, ТЗ §4/§11.12 + история версий, UI-чеклист — новый раздел **S8.55** (5 пунктов, итого 335), обе копии синхронны.

**2026-10-10 (вечер) — прогон остатка на РАБОЧЕМ стенде (по команде заказчика).** Перед началом: бэкап рабочей БД `data/terminal.db.bak-2026-10-10-pre-live-checklist` (backup API, `mode=ro` без `immutable` — под WAL; integrity ok, 0600, 5 сессий / 3 счёта). Стенд поднят `DEV_MODE=false ./restart_dev.sh`; **рабочий чекаут `Develop/` обновлён заказчиком `68ceb58 → 6b197c6`** (`pull --ff-only`, миграций между ревизиями нет) — до обновления заказчик видел на «Счёте» старый 503-алерт на всю страницу, то есть ровно S8R-FIX-074 на старом коде, не регрессию.

**Подтверждено на боевых данных:** S8R-FIX-074 — экран «Счёт» при неотвечающем брокере цел, четыре карточки «—», ошибки внутри блоков; S8R-FIX-075 — футер показывает зелёное «Брокер: на связи» при живом стриме (на тестовом стенде та же плашка серая «Брокер: настроен») — обе ветки фикса проверены вживую.

**Telegram.** Команды боту идут через вебхук на ngrok-туннель `unedited-floral-hardcover.ngrok-free.dev`; туннель был не поднят, `getWebhookInfo` показывал `404 Not Found` и очередь из апдейтов. Запуск упал с **ERR_NGROK_9040** («we do not allow agents to connect from your IP») — блокировка ngrok по IP, не дефект скрипта. Заказчик завернул ngrok через Proxifier (адреса: `*.ngrok-agent.com` порты 443 и **80** — `crl.ngrok-agent.com` по HTTP для агента ≥ 3.10.0; агент 3.37.6, конфиг v3, можно и `proxy_url` в секции `agent`), после чего `./start_ngrok.sh` поднял туннель. Проверено: `POST .../telegram/webhook` снаружи отдаёт 403 без секрета (путь жив), в логе `POST … 200 OK` с IP Telegram `91.108.5.116`. Побочно: `ngrok diagnose` врёт при работе через прокси — формирует `https://[IPv4]:443` и падает на `invalid IP-literal` (ERR_NGROK_8002), это его собственный баг, сеть ни при чём.

**Результаты пунктов.** S8.49 `/balance` — **частично**: канал и обработчик работают, при недоступном брокере бот отвечает «❌ Ошибка получения баланса: Брокер T-Invest не ответил за 10 с (получение портфеля)» (по-человечески, без нулей и стектрейса); строка «Заблокировано под заявки» требует отвечающего брокера. S8.49 п.2 — **не выполнен**: (1) sandbox-API T-Invest не отвечает на unary (5+ попыток подряд, таймаут 10 с; реальный счёт #2 ответил один раз: всего 1 394 055,80 ₽, доступно 7 818,30 ₽, blocked 0), отправка ордера — такой же unary; (2) структурно пункт непроверяем — заведена **S8R-FIX-077**. S8.51 п.2 и S8.53 п.3 — не выполнены: первому нужна sandbox-сделка без цены входа, второму — открытая позиция, а у обеих активных paper-сессий (#1 SBER 1h, #2 LKOH 5m) позиций нет; сессии заказчика не трогались, хотя разрешение «любую» было дано. Параметры заявки согласованы на будни: **SBER 100 лотов по 250 ₽** (цена заметно ниже рынка — заявка должна повиснуть, потом отмена и проверка возврата).

**Стенд оставлен работать** по команде заказчика: uvicorn :8000, vite :5173, ngrok — все под `nohup`, переживают закрытие браузера и терминала; `DEV_MODE=false`, то есть Telegram продолжает слать реальные уведомления. Остановка: `lsof -ti tcp:8000 -sTCP:LISTEN | xargs kill ; lsof -ti tcp:5173 -sTCP:LISTEN | xargs kill ; pkill -f "ngrok http"`.

**2026-10-10 — остаток UI-чеклиста пройден оркестратором (по замечанию заказчика «ты можешь пройти сам»).** Заказчик справедливо указал, что ручная работа по стенду мне по силам. Прогон сделан в браузере на **отдельной копии** данных стенда (`verify.db` из `teststand.db`, свой пароль только в копии, порты :8101/:5274) — стенд заказчика :5273 не затрагивался и остался свежим. **Пройдено:** S8.52 п.4 (уведомление «blocks_json: неизвестные типы блоков: teststand_unknown_block…», копия не создана, баннера нет); S8.53 п.1 (выход «≈ 7 080,00 ₽ (не подтверждена)», «—» без оценки, «P&L не определён» ×3, комиссии), п.2 (позиция 3 лота, P&L −120,30 ₽), п.4 (xlsx скачан и прочитан: раздел «Сделки без подтверждённой цены» с SBER «≈ 265 (не подтверждена)» и тремя LKOH; SBER 250→280 в основной таблице +297,35 ₽; база 0 — сальдирование с убытками песочницы, решение Q3=b); S8.54 п.1 (перечень №7 GAZP / №8 LKOH / №9 SBER; отказ по сессии), п.2 («Закрыто: 0 из 1 … Встречный ордер не отправлен»), п.3 (диалог → «Позиция №53 отмечена закрытой… Ордер не отправлялся», P&L 0 — известная S8R-FIX-071, после чего сессия удалилась); S8.55 п.4 («Ошибка скачивания отчёта» над таблицей, кнопки «Скачать» на месте), п.5 («Нет активных сессий» + «Накопленный P&L: −1 518 ₽»), п.7 («Данные не получены — см. сообщение выше»). Не проверено (стендом не берётся): серый маркер «≈» на графике сессии (без брокера свечи не грузятся), Telegram-пункты, S8.49 п.2, S8.54 п.4. **Находка по оснастке, не по продукту:** `seed.py` останавливал сессии временем `now`, а `AccountService._initial_at(d)` считает сессию активной на дату, пока `stopped_at.date() >= d`, — поэтому сразу после `reset` виджет «Баланс» показывал сумму и S8.55 п.5 был невоспроизводим. `seed.py` исправлен (остановка позавчерашним числом + подчистка ранее остановленных сегодня), стенд заказчика пересобран — «остановлены сегодня: 0». Свой стенд и копия БД убраны. `ui_check_teststand_guide.md` закрыт: остался только список непроверяемого.

**2026-10-10 — S8.52 п.3 и S8.55 п.6 пройдены; способ обрыва связи для Safari.** Заказчик работает в **Safari**, где в Web Inspector нет ни троттлинга, ни режима Offline (это только Chrome DevTools), а выключение Wi-Fi стенду на `127.0.0.1` нипочём — прежняя подсказка в гайде («Network → Offline») была под Chrome и не подходила. Прошли иначе: по команде заказчика остановлен **только backend `:8100`** (vite `:5273` и открытая страница живы), заказчик нажал «Сохранить» → модалка «Код устарел» → «Сгенерировать и сохранить» → ошибка; backend поднят обратно, повторное «Сохранить» снова дало модалку «Код устарел», новой версии в истории нет, в шапке прежняя v4 — **S8.52 п.3 ✅**. Попутно выяснилось и исправлено в гайде: постоянной пометки «код устарел» в редакторе нет (это модалка при «Сохранить», оранжевый тост после сохранения и предупреждение в окне запуска бэктеста — `StrategyEditPage.tsx:751`, `useStrategySave.ts:441`), а генерация кода идёт через backend (`generateCode(id, blocks)`), AI в ней не участвует — прежняя формулировка «генерация локальная» неточна. **S8.55 п.6 ✅** — на скриншоте редактора со стенда `6b197c6` футер показывает серое «Брокер: настроен» вместо зелёного «Брокер: подключён» (S8R-FIX-075 в работе). Гайд сокращён: убраны S8.52 п.3 и строка п.6 из таблицы S8.55, в «Перед началом» записан приём «гаси/поднимай» для обрыва связи с backend. Осталось: S8.52 п.4, S8.53, S8.54, S8.55 пп.4, 5, 7.

**2026-10-10 — отмечены S8.50 п.4 и S8.51; стенд пересобран на `6b197c6`; инструкция сокращена.** Заказчик проверил на стенде: **S8.50 п.4** (подсветка «вне допустимого диапазона — исправьте») и **S8.51** пп.1, 3, 4 («P&L не определён» в журнале; Win Rate 50 %, средняя −22,05 ₽, суммарный P&L −54,62 ₽; legacy paper без пометки) — отмечены в обеих копиях чеклиста; S8.51 п.2 (Telegram) на стенде не воспроизводится, остаётся на рабочем стенде с `DEV_MODE=false`. Стенд переведён `0945edd` → **`6b197c6`** (тот же приём: свой `stash push`/`pop` для `vite.config.ts`, чужой stash не тронут), `reset` + `up`, фиксы 075/076 на стенде сверены по исходникам. `ui_check_teststand_guide.md` сокращён по команде заказчика: пройденные разделы (S8.49 п.3, S8.50 п.4, S8.51) убраны, в шапке — список закрытого и оставшегося, S8.55 переписан под оставшиеся пп.4–7 (налоговый отчёт, виджет «Баланс», футер, пустое состояние операций), заголовок — «S8.52–S8.55». Непройденным остаётся: S8.52 п.3–4, S8.53, S8.54, S8.55 пп.4–7, Telegram-пункты и S8.49 п.2 (нужен живой sandbox #3).

**2026-10-10 — цикл S8R-FIX-075 + S8R-FIX-076 (по команде заказчика «делай а, в той же ветке»).** Та же ветка `s8r/fix-account-page-errors` (ветка на origin была удалена после мержа #47 — локальная подтянута `--ff-only` до `0945edd` и запушена заново), worktree `wt-s8r-account`. RED до правок: 6 vitest. **075** — `StatusFooter` опрашивает `/health` в своём 60-секундном цикле и ведёт три состояния плашки: «Брокер: не подключён» (счёта нет) / «на связи» (зелёный, `tinvest_connected === true`) / «настроен» (серый, счёт есть, стрима нет). Развилка решена оркестратором и вынесена в отчёт: чистый вариант (а) красил бы футер тревожным при каждом простое без активных сессий (подписок тогда не существует), поэтому «нет стрима» показывается нейтрально, а зелёный остаётся обещанием живой связи; ошибка `/health` и отсутствие поля = «связь не подтверждена». Побочно: на главной `/health` теперь опрашивают двое (футер и `HealthWidget`) — запрос лёгкий, readiness. **076** — проп `loadFailed` у `OperationsTable` (`AccountPage` передаёт `operationsError !== null`), пустая строка получила `data-testid="operations-empty"` и текст «Данные не получены — см. сообщение выше»; `PositionsTable` не трогалась (там Alert рисуется вместо таблицы). **Гейт:** vitest 1327/0 (+7 новых), typecheck/lint/build 0/0/0, E2E 175 passed / 1 skipped / 0 failed. `backend/` не затрагивался — pytest/ruff/mypy/bandit локально не перегонялись, их прогоняет CI. Коммит `3590ab7`, **PR #48 смёржен 2026-10-10 — develop `6b197c6`**, CI PR ✅ 4/4 (backend 24m43s), CI develop ✅ (26m32s). Документы: ФТ §7.3/§19.5 + история версий, ТЗ §4/§6 + история версий, UI-чеклист S8.55 расширен до 7 пунктов (итого 337).

**2026-10-10 — S8.49 п.3 ПРОЙДЕН на пересобранном стенде; заведены 075 и 076.** Заказчик открыл «Счёт» на стенде с `0945edd`: четыре карточки «—», страница целиком на месте, сообщения внутри блоков «Позиции» и «История операций» с русским текстом сервера («Брокер T-Invest не ответил за 10 с (получение портфеля/операций)» — в этот раз таймаут, не `UNAUTHENTICATED`), под сообщением в истории операций доступны поля «С»/«По». Отмечены: **S8.49 п.3** и **S8.55 пп.1–3**; остаются S8.55 п.4 (сбой скачивания отчёта) и п.5 (виджет «Баланс» на главной), Telegram `/balance` в S8.49 — без бота не проверить. Две новые находки на том же экране: **S8R-FIX-075** (low) — футер держит зелёное «Брокер: подключён», хотя брокер не отвечает и `HealthWidget` показывает T-Invest OFFLINE: `StatusFooter` берёт `hasBrokerConnection = accounts.some(is_active)`, то есть «счёт настроен», а не «связь есть»; нужно решение заказчика (брать `tinvest_connected` из `/health` / переименовать плашку / три состояния). **S8R-FIX-076** (low, XS) — под красным сообщением таблица операций пишет «Нет операций за выбранный период»: пустое состояние не знает про ошибку, «не удалось узнать» и «операций нет» показываются одновременно.

**2026-10-10 — тестовый стенд пересобран на фикс (по команде заказчика).** `teststand.sh down`; worktree `wt-s8r-teststand` переведён `68ceb58` → **`0945edd`** (правка `frontend/vite.config.ts` proxy→8100 сохранена через свой `git stash push`/`pop`, чужой stash с `.env.example` не трогался); зависимости между ревизиями не менялись — `pnpm install` не нужен; `teststand.sh reset` (копия из бэкапа, integrity ok, head `606214dc5ccf`, seed: 6 stopped-сессий, счета 5/6) и `up` — backend 200, frontend 200, http://127.0.0.1:5273. Код на стенде сверен: `active_capital` в `app/account/schemas.py`, «Нет активных сессий» в `BalanceWidget.tsx`, `positionsError` в `AccountPage.tsx`. Вход — под учёткой заказчика, проверку S8.49 п.3 и S8.55 выполняет он; инструкция `ui_check_teststand_guide.md` обновлена (пометка «провален» снята, добавлен раздел S8.55 из 5 пунктов).

**2026-10-10 (вечер) — S8.49 п.3 на тестовом стенде: ПРОВАЛ, заведена S8R-FIX-074 (medium).** Заказчик открыл «Счёт» (`/account`) → через ~2 с вся страница заменилась красным Alert «Ошибка / Request failed with status code 503», карточек «—» не появилось. Разбор по `s8r-evidence/teststand/backend.log`: `GET /broker/accounts/6/{positions,operations}` → 503 за ~2 с (`broker_positions_endpoint_failed` / `broker_operations_endpoint_failed`, `detail` = «T-Invest не принял API-ключ: ключ истёк… [UNAUTHENTICATED 40003]» — фиктивный ключ стенда), тогда как `GET /broker/accounts/balances` → 200 с суммами `null` только через ~30 с (три `tinvest_unary_deadline_exceeded` по 10 с). Причина — не стенд: в `stores/accountStore.ts` один общий `error` на шесть загрузок, а `AccountPage.tsx:177` при `error` рендерит Alert вместо всей страницы; плюс `catch` использует `err.message` (сырой axios), а не `getApiErrorMessage` — миграция на хелпер из S9-BG-BACKTEST-503-UX сделана частично (10 файлов остались на старом паттерне). Тот же симптом уже видели в S8R-ACCEPTANCE (4 падающих E2E `s5-account.spec.ts`) — тогда поправили мок, причину нет. Пункт S8.49 п.3 на стенде непроходим до фикса (поведение «—» подтверждено только на уровне API). В чеклисте пункт оставлен неотмеченным. Решение по исполнению — за заказчиком.

**2026-10-08 — тестовый стенд для пунктов ⚙️ UI-чеклиста (по команде заказчика):** worktree `wt-s8r-teststand` (detached `68ceb58`), копия БД из бэкапа `terminal.db.bak-2026-10-08-pre-s8r-update` (backup API, `immutable=1` — рядом с бэкапом файлов не появляется; рабочая БД не открывалась), миграции → `606214dc5ccf`; без `.env` заказчика (`MOEX_ENV_FILE=""`, свои SECRET/ENCRYPTION_KEY, DEV_MODE=true); все ключи брокеров и AI в копии заменены фиктивными (брокер отвечает UNAUTHENTICATED/тайм-аут — заявки никуда не уходят); все сессии копии остановлены; добавлены счета «ТЕСТ песочница»/«ТЕСТ реальный» и 6 сессий «ТЕСТ: …» (052 ×2, частичный вход 3/10, сделки без цены, налоговый отчёт, legacy paper), CB `max_drawdown_pct=150`, стратегия «ТЕСТ: дублирование с ошибкой». Порты 8100 / http://127.0.0.1:5273 (другой хост — cookie не пересекаются со штатным стендом). Управление `s8r-evidence/teststand/teststand.sh reset|up|down`. Проверено через API: 409 удаления счёта (перечень 3 сессий) и сессии; «Закрыть позиции» → `broker_position_unverified` без ордера; «Позиция закрыта у брокера» → закрыта без ордера (P&L 0 — открытая 071), затем удаление сессии 204; «P&L не определён» у 3 сделок, legacy paper — нет; итог P&L сессии −30 (без неопределённых); позиция 3 лота; CB `invalid_fields=[max_drawdown_pct]`; дублирование → 422 с текстом сервера. После проверки стенд пересоздан. Не воспроизводятся на стенде: S8.49 п.2 (лимитная заявка — нужен живой sandbox #3), Telegram-пункты (бота нет), S8.54 п.4 (активная sandbox-сессия без брокера при старте сама встаёт на паузу), путь «закрыта вне терминала» по портфелю брокера. S8.52 п.3 — сбой генерации проще вызвать DevTools → Network → Offline (генерация кода локальная, AI не участвует — прежняя подсказка «отключить AI-провайдера» неверна). `restart_dev.sh`: штатный запуск рабочего стенда годится, но `DEV_MODE=true` полностью гасит Telegram/Email (S8R-ACCEPTANCE/AUDIT-084), а `kill -9` через 1 с обрывает graceful shutdown (бюджет до ~60 с) — предложено заказчику.
**2026-10-07 — цикл 063 + 052 (по решениям заказчика):** ветка `s8r/fix-cb-unpriced-account-delete` от develop `48f6773` (worktree C), A — detached для параллели. DEV-FIX-063 (часть 2: консервативная оценка сделки без цены для CB — только убыток, сдвиг 1 % на сторону против позиции, без оценки — уведомление) в A; DEV-FIX-052 (409 при удалении счёта со stopped-сессией с открытыми позициями) в C. Закрыты: 052 (`cb19c72`, 4 прохода ревью / 6 раундов — главное: сверка с портфелем брокера перед ордером у неуправляемой сессии, при неопределённости — отказ без ордера, ручное «Позиция закрыта у брокера»), 063 ч.2 (`fd74766`, 2 прохода / 2 раунда), стыковка при сведении — уведомление 063 в пути «закрыта вне терминала» (`aefa87d`). Заведены 070 (снять пометку расхождения у остановленной сессии), 071 (P&L 0 у закрытых вне терминала). Гейт G1: pytest 5719/0, vitest 1307/0, ruff/mypy/bandit/typecheck/lint/build — 0; E2E 175/0. PR #45 смёржен 2026-10-08 (develop `bb5fa88`), CI develop ✅; worktree A и C убраны. Команда заказчику: `git branch -d s8r/fix-cb-unpriced-account-delete && git push origin --delete s8r/fix-cb-unpriced-account-delete`.
Флейки: S8R-FIX-005 и 068 закрыты; 050 — ⏸ со сторожем зависания (при рецидиве сторож назовёт тест).
Правило гейтов: `-o faulthandler_timeout=240`, лог в файл; bandit — `-ll -c .bandit`.
Инструменты: `s8r-evidence/fixes/tools/` (compose.py + card.sh; для FIX-карточек — cfg с полем CARD из backlog; adhoc.py — находки ревью/CI; stand.sh — E2E-стенд; review_compose.py + rcfg_*.json — ревью по осям).
В конце цикла: убрать worktree A и C; команды заказчику — раздел «Команды заказчику» выше (плюс удаление `s8r/fix-low` после мержа).
