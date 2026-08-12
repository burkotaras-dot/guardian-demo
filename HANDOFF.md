# Guardian Essentials — Handoff & Concept Pack

Переглядний пакет для інженерної команди та клієнта. Збирає всі Essentials-flows в одному місці:
що це за екран, чому саме так (rationale), у якому він статусі готовності, і які лишились відкриті питання.

> **Як читати.** Розділ 2 — інвентар усіх flows. Розділ 3 — interaction notes по кожному flow.
> Розділ 4 — reusable-патерни (звірятися при побудові нових екранів). Розділ 5 — open questions
> (потребують підтвердження клієнта/backend). Кожен flow має **статус готовності** (легенда нижче).
>
> Прототип: single-file `index.html`. Live: https://burkotaras-dot.github.io/guardian-demo/essentials/
> Дизайн-система (детальні правила): `GUARDIAN-DESIGN-SYSTEM.md`.

## Легенда статусів

| Позначка | Значення |
|---|---|
| 🟢 **Ready to build** | Дизайн і поведінка узгоджені, немає невідомих — можна імплементувати. |
| 🟡 **Needs technical confirmation** | Візуально готово, але треба підтвердити backend-поля / API / логіку. |
| 🔵 **Concept only** | Демонструє напрям/ідею; не фінальний дизайн, потрібне рішення. |
| ⚪ **Deferred** | Свідомо відкладено за межі Essentials scope. |

---

## Зміст
1. [Огляд](#1-огляд)
2. [Інвентар flows](#2-інвентар-flows)
3. [Interaction notes](#3-interaction-notes) — ⏳ заповнити (пункт 2 пʼятниці)
4. [Reusable patterns](#4-reusable-patterns) — ⏳ (пункт 4)
5. [Open questions](#5-open-questions) — ⏳ (пункт 5)
6. [End-to-end перевірка](#6-end-to-end-перевірка) — ⏳ (пункт 6)

---

## 1. Огляд

**Що таке Essentials.** Полегшена версія Guardian: підключення джерел вузлів (agentless / ServiceNow / CSV),
збір конфігурацій, сканування на відповідність політикам, фіксація baseline, виявлення drift і звітність.
Фокус — **простота підключення й зрозуміла звітність**, без важкої аналітики.

**Продуктова історія (happy path):**
Підключити вузли → налаштувати доступ (Credentials / Connection Manager) → просканувати →
зафіксувати baseline → бачити drift → звіт / реконсиляція через ITSM.

---

## 2. Інвентар flows

Повний перелік Essentials-flows, що входять у concept pack. Статус і нотатки — у розділі 3.

### A. Onboarding / підключення вузлів
| # | Flow | Ключові екрани | Патерн |
|---|---|---|---|
| A1 | Add single node (agentless) | `v-s-agentless` → connect-test → node-added | Full-page (виняток) |
| A2 | Import from ServiceNow CMDB | `#cmdb-modal` (connect → browse → credentials → importing → complete) | Overlay modal |
| A3 | Import from CSV | `#csv-modal` (upload → review → importing → complete) | Overlay modal |
| A4 | Connection Manager setup | `cm-wizard` (create group → install & connect → active) | Overlay modal |

### B. Доступ і джерела
| # | Flow | Ключові екрани | Патерн |
|---|---|---|---|
| B1 | Credentials | list → New/Edit credential | Overlay modal |
| B2 | Integrations | list → connect wizard (icx) | Overlay modal |
| B3 | ITSM capabilities (J4 ServiceNow) | `j4-connect` → match → recon active → summary | Full-page flow |
| B4 | Data / field mapping | `fm-mapping-wizard` | Overlay modal |
| B5 | Cloud connect (J3 AWS) | `j3-connect` → inventory → scan → results | Full-page flow |

### C. Сканування, baseline, drift
| # | Flow | Ключові екрани | Патерн |
|---|---|---|---|
| C1 | Scan Schedules | list → set/edit schedule | Overlay modal |
| C2 | Scan History | node-detail History / scans list | Inline / detail |
| C3 | Baseline (set / active / compare) | node-detail baseline model | Inline / detail |
| C4 | Detected Drift | `v-all-devices` detected tab → change-detail | Full-page + detail |

### D. Звітність
| # | Flow | Ключові екрани | Патерн |
|---|---|---|---|
| D1 | Reports | type → scope → filters → preview → export | Компактний (без wizard) |

---

## 3. Interaction notes

Поведінка кожного flow + стани + статус готовності. Стани позначено: **E**mpty · **L**oading ·
err**O**r · **S**uccess.

### A. Onboarding / підключення вузлів

**A1 · Add single node (agentless)** — 🟡 *Needs technical confirmation*
- Full-page onboarding (виняток із modal-правила): три секції — Target (host/OS) · Connection Manager · Credentials.
- CM-picker фільтрується за сумісністю протоколів (WinRM/SSH); одноопційний credential-toggle прихований.
- Валідація: submit заблокований, поки не заповнені обовʼязкові поля + inline req-hint («Add a node name, a hostname…»).
- Connect-test engine: сценарії success / auth-fail / unreachable з відповідними станами.
- Стани: **L** connect-test spinner · **O** три типи помилки конекшену · **S** «node added» екран.
- *Confirm:* реальні коди помилок конекшену від бекенду; чи всі три сценарії відповідають продукту.

**A2 · Import from ServiceNow CMDB** — 🟡 *Needs technical confirmation*
- Overlay modal (`#cmdb-modal`), 3-крокний stepper: Connect → Select nodes → Credentials → (importing) → complete.
- Browse: список класів/вузлів із CMDB, вибір для імпорту; імпортовані падають у Detected tab.
- Стани: **L** connecting/importing spinner · **O** «Connection failed» (circle-info, «Try again →») · **S** центрований «N nodes added to Detected» + міні-статистика (Linux/Windows).
- *Confirm:* які CMDB-класи/поля реально доступні; чи застосовуються default credentials при імпорті.

**A3 · Import from CSV** — 🟢 *Ready to build*
- Overlay modal (`#csv-modal`), той самий gm-* shell + 3-крокний stepper: Upload → Review → Import → complete.
- Download-template, upload, попередній перегляд колонок/рядків перед імпортом.
- Стани: **L** importing · **O** невалідний файл/колонки · **S** центрований complete.
- Формат CSV фіксований шаблоном — низький ризик невідомих.

**A4 · Connection Manager setup** — 🟢 *Ready to build*
- Overlay modal (`cm-wizard`), 2 кроки: Create group → Install & connect.
- Create: назва групи (валідація red-border + auto-clear). Install: API key + URL + installer (Windows/Linux) + очікування check-in.
- Стани: **L** «Waiting for a CM to check in…» · **S** центрований «Connection Manager active» (уніфіковано цього тижня).
- Проміжний «Group created» рядок — свідомо лишений (середина flow, не термінал).

### B. Доступ і джерела

**B1 · Credentials** — 🟢 *Ready to build*
- List + New/Edit credential (overlay modal, преференс клієнта «як Flow»).
- Валідація: red-border `#FB3C44` + focus + auto-clear на першому вводі.
- Empty-states: no-match (reset) / true-empty (filled CTA «+ New credential»).
- *Note:* Secret Server / зовнішні сховища — toast-success (доречно per-context).

**B2 · Integrations** — 🟡 *Needs technical confirmation*
- List + connect wizard (`icx-wizard`, overlay).
- Статус-вокабуляр: Available → Connect → Connected → Attention → Error → Disabled.
- Capability-бейджі окремим набором: Enabled / Disabled / Issue.
- *Confirm:* повний перелік доступних інтеграцій та їхніх capability для Essentials.

**B3 · ITSM capabilities (ServiceNow / J4)** — 🔵 *Concept only*
- Full-page flow: Connect → Match Nodes → Recon Active → Summary (nav 3 кроки).
- Демонструє реконсиляцію detected drift проти approved Change Requests.
- Стани: **L** connecting/syncing · **O** «Connection Failed» (icon-row + errors, «Try again →») · часткові збіги (partial-match) / no-CRs.
- *Note:* deep reconciliation-логіка — на dev, НЕ у прод-scope Essentials.

**B4 · Data / field mapping** — 🔵 *Concept only*
- Overlay modal (`fm-mapping-wizard`): маппінг зовнішніх полів на поля Guardian.
- *Confirm:* які transformations реально підтримуються (напр. rename / split / default).

**B5 · Cloud connect (AWS / J3)** — 🔵 *Concept only*
- Full-page flow: Connect AWS → Sync Nodes → Run Scan → Results (nav 4 кроки, лічильники уніфіковано «of 4»).
- Стани: **L** connecting/scanning · **O** «Connection Failed» (IAM/permission errors, «Try again →») / no-resources · **S** results.
- *Note:* OIDC для AWS — deferred (за межі 4-го тижня).

### C. Сканування, baseline, drift

**C1 · Scan Schedules** — 🟢 *Ready to build*
- List + set/edit schedule (overlay modal).
- Empty-states: table-level (0 груп/env) + per-row «No scan schedule set» (Set-кнопка) — це коректний per-row індикатор, не плутати з table-empty.

**C2 · Scan History** — 🟡 *Needs technical confirmation*
- Список сканів на node-detail; узгоджений із header/baseline-моделлю.
- Навігація: scan result → drift → report.
- *Confirm:* які поля історії/метадані скану доступні з бекенду.

**C3 · Baseline (set / active / compare)** — 🟡 *Needs technical confirmation*
- Вибір скану як baseline, позначка активного baseline, порівняння сканів.
- *Confirm:* модель зберігання baseline; чи baseline per-node чи per-policy.

**C4 · Detected Drift** — ⚪ *Deferred (dev, не прод-scope)*
- Detected tab у Nodes + change-detail з before/after evidence, CR-станами, audit-нотатками.
- *Note:* виявлення drift показане; глибока reconciliation-робота лишається на dev.

### D. Звітність

**D1 · Reports** — 🟢 *Ready to build*
- Компактний flow (свідомо БЕЗ покрокового wizard): type → scope → filters → preview → export.
- Точки входу: Detected Drift / Scan History / Baseline / Policies.
- Типи: що змінилось / що відхилилось від baseline / що потребує розслідування.
- Стани: **E** empty · **L** loading · no-results · **S** export success / **O** export error. Export = green primary; CSV + PDF.
- Reporting свідомо легкий (не аналітичний продукт).

---

## 4. Reusable patterns

Будь-який новий екран звіряється з цими патернами, а не з десятком попередніх. Детальні
навігаційні правила (з Reference + Exceptions) — у `GUARDIAN-DESIGN-SYSTEM.md`; тут — зведення
для handoff.

### 4.1 Navigation (див. `GUARDIAN-DESIGN-SYSTEM.md` §1)
- **Add / Create / Connect / Import → overlay modal** через `openModal(opts)`. Виняток — agentless (full-page).
- **Back (full-page)** — icon-only `.back-btn` 32×32, зелений tint; handler `navBack()`. Не саморобні «← Back».
- **Close (modal)** — `gm-x` (×) + клік по backdrop + `Esc`; `confirmDirty` за незбережених змін.
- **Steppers** — єдиний `gm-stepper` у модалці (опція `steps`); без паралельних full-page step-nav.
- **Modal content is frameless** — контент лежить на `gm-body`, без вкладених «карток-рамок».

### 4.2 Success screen (canonical)
- Іконка: **56×56 коло**, ghost-green фон `rgba(37,167,96,0.1)`, 24–28px SVG-галочка `stroke #25A760`.
- Заголовок 20px/800 `--text-default`, **без знаку оклику** («Policy created», не «Policy created!»).
- Підзаголовок 12–14px `--text-sec`; опційна міні-статистика; CTA — filled `btn-primary`.
- Reference: CMDB complete · Policy Create · CM Group «Connection Manager active».

### 4.3 Error screen (canonical)
- Іконка: ghost-red коло `rgba(251,60,68,0.08)`, SVG **circle-info**, `stroke #FB3C44` (не circle-X, не `#E92D38`).
- Рядок-підсумок помилок: flex icon + `<span 13/700 #FB3C44>N errors found</span>`.
- Кнопка повтору: **«Try again →»** (мала «a»).
- Reference: J3 connect-failed · CMDB · J4.

### 4.4 Wizard step names
- Лічильник кроку в бейджі **дорівнює** кількості кроків у top-nav (J3 = «of 4», J4 = «of 3»).
- Назви кроків — дієслова-імперативи («Connect», «Match Nodes»), консистентні між собою.

### 4.5 Empty-state
- 44×44 icon-circle (`--bg-canvas`) + 20px SVG `stroke --text-muted 1.8` + title 14/600 `--text-default` + subtitle 12px `--text-muted` + дія.
- Два варіанти: **no-match** (outlined green reset «↺ Clear/Reset») · **true-empty** (filled `btn-primary` CTA).
- Reference: Reports (`rc/rb/ri-empty-state`).

### 4.6 Validation
- **Full-page** (`alValidate`): disabled-submit + inline warn + req-hint зі списком того, що бракує.
- **Modal** (`cmCreateGroup`): red-border `#FB3C44` + focus + **auto-clear на першому вводі**.

### 4.7 Search / text input focus
- Реплікує `.form-input:focus`: `border-color var(--green1)` + `box-shadow 0 0 0 3px var(--green-dim)`.
- Для інлайн-полів — через `onfocus`/`onblur` з `var()` (theme-safe).

### 4.8 Green-outlined action button (row/strip)
- Стандартна action-кнопка для будь-якого «рядок/картка + кнопки» (View/Edit/Review/Reconcile…).
- Light: bg `#F3FAF6`, border `rgba(16,185,129,0.32)`, стрілка `--green1`. Dark: bg `rgba(0,175,115,0.15)`, текст білий.
- Завжди через CSS-клас під scope рядка/картки, НЕ інлайн.

### 4.9 Dark-mode parity
- CSS `body.dark …{ … !important }` + JS `_syncInlineTheme(isDark)` (regex-ремапінг відомих світлих hex → dark).
- Токени/`var()` інлайн — theme-safe. Нові інлайн-світлі hex на chrome → додавати мапінг у `_syncInlineTheme`.
- Column-headers таблиць у dark = `#063135` (не чорний). Бейджі в dark: текст `#fff`, іконка = колір border.

### 4.10 Terminology / status vocab
- Integration status: Available → Connect → Connected → Attention → Error → Disabled.
- Capability badges (окремо): Enabled / Disabled / Issue.
- Severity: coloured **dot** + текст-лейбл на нейтральному контролі (не заливати весь control кольором).

---

## 5. Open questions

Потребують підтвердження клієнта / backend перед тим, як 🟡/🔵-flows перейдуть у 🟢 ready-to-build.
Колонка «Блокує» вказує, який flow розблокується відповіддю.

### 5.1 Доступні backend-поля
| # | Питання | Блокує |
|---|---|---|
| Q1 | Які CMDB-класи та поля реально доступні для імпорту з ServiceNow? | A2 CMDB |
| Q2 | Які метадані/поля скану доступні для Scan History (час, тривалість, ініціатор, результат)? | C2 History |
| Q3 | Повний перелік доступних інтеграцій та їхніх capability для Essentials? | B2 Integrations |

### 5.2 Стани конекшену
| # | Питання | Блокує |
|---|---|---|
| Q4 | Реальні коди/типи помилок конекшену агентлес-вузла (auth-fail / unreachable / timeout / …)? | A1 Agentless |
| Q5 | Які помилки повертає CMDB/ServiceNow-конект (щоб error-екран відповідав продукту)? | A2, B3 |
| Q6 | Логіка check-in Connection Manager: таймаут очікування, retry, стан «частково підключено»? | A4 CM setup |

### 5.3 Логіка валідації
| # | Питання | Блокує |
|---|---|---|
| Q7 | Правила унікальності імен (node / credential / CM group / policy) — case-sensitivity, scope? | A1, B1, A4 |
| Q8 | Обовʼязкові поля для кожного типу credential (user/pass, key, token, vault-ref)? | B1 Credentials |

### 5.4 Частота синку
| # | Питання | Блокує |
|---|---|---|
| Q9 | Як часто синкати вузли з ServiceNow/CMDB — manual / scheduled / webhook? Чи налаштовується? | A2, B3 |
| Q10 | Дефолтний розклад сканування та допустимі інтервали для Scan Schedules? | C1 Schedules |

### 5.5 Mapping transformations
| # | Питання | Блокує |
|---|---|---|
| Q11 | Які трансформації полів реально підтримуються (rename / split / concat / default / lookup)? | B4 Field mapping |
| Q12 | Чи застосовуються default credentials автоматично при імпорті вузлів? | A2 CMDB |

### 5.6 Baseline та поведінка при падінні скану
| # | Питання | Блокує |
|---|---|---|
| Q13 | Модель baseline: per-node чи per-policy? Скільки baseline можна тримати, як позначається активний? | C3 Baseline |
| Q14 | Підтверджена поведінка при падінні скану: які причини бекенд розрізняє, які рекомендації валідні? | C2, C4 |
| Q15 | Звʼязок scan result ↔ baseline ↔ detected drift ↔ report — чи збігається з реальною моделлю даних? | C2–C4, D1 |

> **Принцип:** error/fail-стани в прототипі побудовані ТІЛЬКИ з відомої по Guardian інформації —
> без вигаданих рекомендацій. Відповіді на Q4–Q6, Q14 дозволять зробити їх точними.

---

## 6. End-to-end перевірка

Прогін повного Essentials flow у preview (2026-08-08), light + dark. Метод: програмна навігація
по всіх views/модалках + перевірка рендеру (view видимий і не порожній) + консоль.

### Результат: ✅ пройдено, помилок немає

**Основні екрани (рендер OK, light + dark):**
Dashboard · All Nodes · Policies · Results · Detected Drift · Reports · Credentials · CM Groups · Integrations.

**Flow-екрани (рендер OK):**
Add node (agentless) · node-detail · J3 connect (AWS) · J4 connect (ServiceNow) · change-detail · policy-detail.

**Nodes під-таби (OK):** Managed · Detected · Peer.

**Модалки (відкриваються, контент рендериться):**
CM setup (`cm-wizard`) · New credential (`cred-edit-overlay`) · CMDB import (`#cmdb-modal`) · CSV import (`#csv-modal`).

**Консоль:** 0 errors у light і dark протягом усього прогону.

**Спот-чек цього тижня (окремо верифіковано скріншотами):** success-екрани (Policy created, CM «Connection
Manager active») · error-екрани (CMDB, J4) · step-лічильники (J3 «of 4», J4 «of 3») — усі коректні в обох темах.

### Не виявлено
Тупиків, битих переходів, порожніх (0×0) views, помилок консолі.

---

## Підсумок пакета
Concept pack зібрано: інвентар 14 flows (розд. 2) → interaction notes зі статусами (розд. 3) →
reusable patterns (розд. 4) → 15 open questions з привʼязкою до flows (розд. 5) → e2e-перевірка (розд. 6).
Наступний крок після відповідей клієнта на Q1–Q15: перевести 🟡/🔵-flows у 🟢 ready-to-build.
