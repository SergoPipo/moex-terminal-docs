# DEV-1 — фиксы по сплошной проверке UI-чеклиста S8 (2026-10-14)

## 1. Реализовано
- **S8R-FIX-102 (high) — PASS.** Бар sequential-серии несёт время свечи в `customValues.barTime` (`setData` и live-`update`). Точка рисунка хранит время свечи; при отрисовке номер бара ищется по времени в текущем массиве; правее последнего бара — экстраполяция шагом ТФ; левее окна — не рисуется до дозагрузки. Старые точки (год `t` < 2000) — по `logical`, как раньше; drag привязывает их к свече. Drag, vline, копия, конец long/short — в барах серии. Дневной ТФ не тронут.
  RED на HEAD: 16/17 (`t`=-10788000 мс; дозагрузка `120≠420`; окно `200≠120`; вне окна `50≠null`; live-бар без `barTime`). Мутации 6/6 пойманы.
- **1.2 qa2 Н-6 — PASS.** Корень: `autoScale:false` ставится один раз и не снимается; смена ТФ/тикера при свечах в кеше не размонтирует график и шла веткой дозагрузки → ось «Д» (245–312 → 236–330) оставалась на 1ч. Новый набор (тикер:ТФ) = первая загрузка: `autoScale:true` → начальный вид → фиксация в rAF. RED: `expected [] to deeply equal ['autoScale:true','autoScale:false']`; мутации 2/2.

## 2. Файлы
Изменены: `frontend/src/components/charts/{primitives/coords.ts, sequentialIndex.ts, CandlestickChart.tsx, DrawingsLayer.tsx}`. Новые: `primitives/__tests__/coords.fix102.test.ts`, `__tests__/CandlestickChart.fix102.test.tsx`, `__tests__/CandlestickChart.datasetSwitch.test.tsx`.

## 3. Тесты
Новые 20/20; зона charts+marketDataStore+hooks 37/326 (baseline 34/306); весь vitest 188/1441; `pnpm typecheck` 0, `pnpm lint` 0.
Живая проверка, стенд 2 (свой Vite :5290 vs штатный :5275 = develop), 5м: develop — после дозагрузки линия ушла из вида, `t=1969-12-31T21:06:49Z`; фикс — концы на тех же свечах, `t=2026-10-13T15:30Z/18:20Z`. Н-6: доля высоты свечей 1ч после перезагрузки — develop 0.11, фикс 0.45. Доказательства: `s8r-evidence/checkrun/results/dev1/{develop,fix,spec}/`.

## 4. Integration points
`sequentialCandleData` ← `candlesToSequential`/`CandlestickChart`; `seq*` ← `coords.ts`; `offsetPointByBars`, `getBarWidthSec(chart, series)` ← `DrawingsLayer`. NOT CONNECTED нет.

## 5. Контракты
API не менялся; в intraday `t` — время свечи, `logical` — номер бара при клике.

## 6. Проблемы / TODO
- Точка левее окна скрывает фигуру до дозагрузки (как маркеры BUG-28 и «Д»).
- Точка «в будущее» при ночном разрыве ляжет на бар перед ним.
- Вне зоны: устарел комментарий `DrawingPoint` в `api/chartDrawingsApi.ts` («источник правды — logical»).
- Удалить вручную (`rm -r` запрещён): `s8r-evidence/checkrun/dev1-vite-cache/`.
- Node локально 20.20.1.
- Стенд 2: удалены рисунки SBER 5m.

## 7. Применённые gotchas
24, 46, 60.

## 8. Новые gotchas (предложение)
- Дополнить 24: номер бара — не идентичность; время бара — в `customValues`.
- 61: `autoScale:false` переживает `setData` нового набора; при свечах в кеше график не перемонтируется.

## 9. Плагины
context7 не авторизован → API по `typings.d.ts` и исходнику lightweight-charts; typescript-lsp → `pnpm typecheck`; playwright — `@playwright/test`; TDD — `mattpocock-skills:tdd`.
