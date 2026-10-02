## DEV-FIX-BT отчёт — код-ревью, раунд 2 (S8R-FIX-035/036/057)
Статус: ✅ готово к коммиту (worktree `wt-s8r-fixes`, ничего не закоммичено)

### Правки
1. **035**: `canonical_ticker(..., fallback=)`; rerun канонизирует после проверки стратегии и `_check_code_not_stale`; если канон не найден, остаётся сохранённый тикер (`SiZ6`).
2. **035**: `_check_backtest_period` — общий хелпер для create/run-async/grid/rerun (`date_to > date_from`, ≥ 30 дней). Ошибка → 422 до создания записи.
3. **035**: запись создаёт `_running_backtest(params)` — тот же хелпер у create. Текст 422 по-русски: подписи полей, ошибка модели — общая фраза.
4. **036**: `persist_with_retry_fresh` (своя сессия на каждую попытку) для INSERT и статуса job. Общий цикл `_retry_on_locked`, у обоих хелперов есть `on_retry`; логи повторов во всех трёх местах.
5. **057**: `_wall_clock_index` — смешанные пояса и aware+naive переводятся в «время на часах» бара, бенчмарк не падает. NaN close IMOEX считается днём без свечи (ffill). Раньше NaN ронял весь бэктест на `EquityPoint` (`finite_number`). Поведение закреплено тестом и докстрингом.
6. `_with_canonical_ticker` заменил четыре копии в роутере.

### Тесты
- RED: `'SIZ6' != 'SiZ6'`, `'позже даты начала' in 'Период … не менее 30 дней'`, `commission_pct: Input should be…`, `PendingRollbackError … database is locked`, `imoex_benchmark_load_failed`, `finite_number`.
- GREEN: 035 — 16, 036 — 8, 057 — 6.
- Мутация п.1 (без `fallback`) → `'SIZ6' != 'SiZ6'`.
- Мутация п.4 (одна сессия на все попытки) → `PendingRollbackError`.
- Замер: 2.895 → 0.415 с, md5 кривой не изменился.

### Гейты
- pytest: 5248 passed, 1 failed (вне карточек, см. ниже).
- ruff 0, mypy Success (200), bandit 0.
- typecheck 0, lint 0, build ok; vitest — фронт не менялся.

### Находки
- `test_peak_persists_when_signal_skipped_out_of_hours` падает после 22:00 MSK: окно `now+1ч…now+2ч` переходит через полночь. Circuit Breaker не трогал; в 20:55 полный прогон был зелёным.
- ТЗ, строка 947 (`/rerun`), и ТЗ §5.6 — текст правки из раунда 1. Дополнительно: «конец периода позже начала» во всех точках входа бэктеста.
