# Тиждень 1 — Підсумок (13–17 липня 2026 + закриття 20 липня)

## Ціль тижня

Сформувати УЗГОДЖЕНУ базу Essentials: інвентар клієнтських wireframes, порівняння з
old Guardian Demo, наскрізний flow, пріоритетні journeys, перший polish pass. П'ятничний
вихід = «agreed scope slice + first-pass consolidated wireframe direction».

## Що зроблено

### Аналіз та планування (Пн–Вт)

1. **Повний інвентар** 68 екранів клієнтського wireframe — кожен flow пройдений,
   стани задокументовані.
2. **Порівняння client vs old Guardian Demo** — таблиця відсутніх екранів/станів
   по 6 потоках (Agentless, CM, Credential Vault, CSV import, Scan Schedule, Reports).
3. **Наскрізний Essentials-флоу** (один happy path):
   ```
   Перший вхід → вибір підключення → додати вузол → креденшели →
   розклад скану → перший скан → результат → baseline → drift →
   розслідування → реконсиляція або export
   ```
4. **9 пріоритетних journeys** у порядку показу клієнту.
5. **Рекомендація по Scan History** — взяти каркас Гарета (baseline → drift зв'язка),
   підтягнути наш патерн таблиці/детальки.

### Уніфікація (Вт, достроково)

6. **Термінологія node/device** — канонічний об'єкт = node; хедери, кнопки, nav-лейбли,
   онбординг, стат-підказки уніфіковані. Задеплоєно на клієнтський лінк (b8f1925 + c30d37f).

### Reports — глибока робота (Ср, випередила план)

7. **3-рівнева модель фільтрів:**
   - Scope filters (Environment, Node group, Benchmark, Date) → перераховують весь звіт
   - Report type (Summary / Failed-checks detail / Full) → видимість блоків
   - Local table filters (Severity, Policy, пошук) → лише таблиця Failed checks
8. **Детерміноване перерахування** `REP_SCOPE` + `renderReport()` — KPI, context,
   обидві таблиці, export state з одного джерела. Node group реально перераховує звіт
   (підгрупи сумуються до All node groups).
9. **5 UX-покращень** Reports: групування хедера, ясність станів, hover-ряди як лінки,
   порожній стан, узгодження контролів до 32px.

### Клієнтські рішення (Ср)

10. **Reports = повний implementation-ready flow** (CSV + PDF export), НЕ лише концепт.
11. **AWS OIDC + scan-failure troubleshooting** — align з Foundry + реальними
    можливостями бекенду.

### Polish pass (Нд 20, закриття)

12. **Навігація:** канонічні ← Back кнопки на j3-scan-complete, j3-results, v-s05.
13. **Кольори:** v-s01 «Add from Cloud» blue → green (= сусідня картка); v-s-cm-setup
    demo-банер blue → neutral.
14. **Dead buttons:** обидва «Retry scan» на v-scan-failed підключені (loading →
    toast → node-detail).
15. **Коміти:** `ad093b6` (Reports recompute) + `92f5f5c` (polish pass).

## Що НЕ зроблено (план vs факт)

| Заплановане (Чт–Пт) | Статус |
|---|---|
| Empty/loading/success/error стани (концептуальний рівень) | Частково (retry scan); решта = нова функціональність, перенесено |
| Scan History рекомендація (формальна) | Є усна (Вт, п.5); письмова не оформлена |
| Клієнтське рев'ю (Пт) | Не відбулось — тижневий слот вичерпався |
| Формальний скоуп Тижня 2 | Формується зараз (нижче) |

## Причина зсуву

Reports (area 9 у task plan) за планом мала бути FINAL (тижні 3–4), але клієнтське
рішення «Reports = повний flow» підняло пріоритет. Глибока перебудова фільтрів (Ср)
зайняла слот Чт–Пт. Рішення правильне — тепер Reports має робочу архітектуру, а не
заглушку, — але формальне закриття тижня зсунулось на Нд.

## Артефакти

- Гілка: `essentials` (2 ahead of origin)
- Статуси: `docs/status/2026-07-13.md` … `2026-07-17.md`
- Пам'ять: `essentials-task-plan.md`, `essentials-decisions.md`,
  `essentials-prototype-lessons.md`

---

# Тиждень 2 — Скоуп (21–25 липня 2026)

## Ціль

Connection Manager setup + Credential Vault + onboarding flow detail (areas 3, 4, 5
з task plan). Це логічне продовження: Тиждень 1 = інвентар + flow + Reports base;
Тиждень 2 = глибокий UX трьох ключових interaction flows.

## Денний план

### Понеділок 21 — Connection Manager: стани та список

- CM list page: таблиця з 5 станами (Installed / Connected / Offline / Update required /
  Error) + status badges + last-seen timestamp.
- Кожен стан = свій рядок у таблиці (реалістичні дані).
- Actions per CM: Edit / Retry / Update / Disable / Remove.
- Порожній стан: «No Connection Managers — install your first one».

### Вівторок 22 — Connection Manager: add-new flow + troubleshooting

- Add CM flow: deployment location → prerequisites → install instructions →
  connectivity test → readiness check → success.
- Error states: missing prereqs, connectivity failure, installed-but-not-scan-ready.
- Troubleshooting panel: діагностика + retry.

### Середа 23 — Credential Vault

- Vault list: збережені креденшели, де використані (node count), тип (WinRM / SSH /
  OIDC), last-validated.
- Create credential flow: type picker → fields → validate → save.
- Edit / replace / delete (з попередженням «used by N nodes»).
- Pick existing credential під час add-node.
- Secret Server vs Guardian Vault = two modes (base vs PRO), clear upgrade path
  WITHOUT blocking base flow.

### Четвер 24 — Onboarding detail (agentless single node)

- Повний refined flow: вибір OS/env → pick CM → pick/create creds → address +
  params → reachability check → connection errors → «ready to scan» confirmation.
- Missing states з інвентарю Вт (14): reachability-check, помилка з'єднання,
  «не готовий до скану».
- Зв'язка з CM list і Credential Vault (cross-flow navigation).

### П'ятниця 25 — Polish + закриття + рев'ю

- Перший polish pass по CM + Vault + onboarding (навігація, кольори, стани).
- Scan History рекомендація — оформити письмово.
- Week-2 summary.
- Підготовка до клієнтського рев'ю (якщо заплановане).

## Що НЕ входить у Тиждень 2

- AWS OIDC flow (потребує підтвердження Foundry-паттерну)
- ServiceNow / CSV import (Тиждень 3)
- Integrations IA (Тиждень 3)
- Reports: Environment / Benchmark / Date recompute (Тиждень 3–4)
- Pixel-perfect UI / final clickable prototype

## Залежності

- CM troubleshooting: потрібен від команди список failure causes, які бекенд реально
  детектить (клієнтське рішення Ср 15).
- Credential Vault: підтвердити, чи AWS OIDC = Foundry pattern reuse.
- Обидві — питання до клієнта, можуть заблокувати деталі (fallback = placeholder +
  annotation «pending backend confirmation»).
