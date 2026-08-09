# Guardian Design System Rules

Єдине джерело правди для всіх екранів Guardian. Кожне правило складається з трьох частин:

- **Rule** — що є стандартом.
- **Reference** — екран/компонент, який є еталоном (canonical example).
- **Exceptions** — коли правило можна порушити (і чому).

> Мета: будь-який новий екран звіряється з цим документом, а не з десятком попередніх. Якщо екран не відповідає правилу і не підпадає під зафіксований Exception — він неправильний.

Файл: `index.html` (single-file prototype). Посилання на рядки — орієнтовні, звіряти перед правкою.

## Зміст
1. [Navigation](#1-navigation) — ✅ заповнено
2. Page layout — ⏳
3. [Forms](#3-forms) — ✅ dropdown hover
4. Tables — ⏳
5. States — ⏳
6. Buttons — ⏳
7. Cards — ⏳
8. KPI — ⏳
9. Spacing — ⏳
10. Terminology — ⏳

---

## 1. Navigation

### 1.1 Add / Create / Connect → overlay modal

**Rule.** Будь-який flow додавання/створення/підключення сутності (Add / Create / Connect / Import) відкривається як **overlay-модалка** через уніфікований shell `openModal(opts)`, а НЕ як окрема full-page `view`.

Один shell дає безкоштовно: `gm-overlay` затемнення, `gm-card`, `gm-head` із заголовком+підзаголовком, `gm-x` close (×), опційний `gm-stepper`, `gm-body`, `gm-foot` із кнопками. Опції: `id · title · subtitle · size · steps · buttons(kind/label/id/disabled) · bodyHtml · footerLeftHtml · dismissable · confirmDirty`.

**Reference.** **CM Groups → Create group** — `openModal('cm-wizard', …)`. `openModal` визначено в `index.html:11596`. Інші коректні приклади: New/Edit credential, Integrations wizard, Edit scan schedule, Policy Create.

**Import sub-pattern (reference).** **CMDB Import** — ✅ сконвертований в overlay (`#cmdb-modal`, gm-* shell + 3-крокний `gm-stepper`, driver `cmdbShowPanel()`), тепер є **canonical reference для всіх import-flows**; **CSV Import** — ✅ conform-нутий під нього (`#csv-modal`, той самий gm-* shell + 3-крокний stepper Upload→Review→Import, driver `csvShowPanel()`).

**Exceptions.**
- **Add node agentlessly** (`v-s-agentless` та супутні) — full-page. Обґрунтування: складний багатосекційний onboarding (Target / CM / Credentials) + connect-test engine, не вміщується коректно в модалку.
- Нові винятки додавати сюди ЛИШЕ з явним обґрунтуванням і рішенням user.

**Порушники, які треба привести до правила:**
- ✅ **CMDB / ServiceNow Import** — БУВ 7 full-page views (`v-cmdb-connect` → `-connecting` → `-connect-failed` → `-browse` → `-credentials` → `-importing` → `-complete`). **Сконвертований в один overlay** (`#cmdb-modal`): усі 7 екранів тепер `.cmdbm-panel` всередині gm-* shell, `go('cmdb-*')` перехоплюється → `cmdbShowPanel()`; hand-rolled back-кнопки прибрані, мертвий `updateCMDBNav` видалено. Тепер = canonical import reference.
- ✅ **CSV Import** — БУВ 4 full-page views (`v-csv-import` → `-preview` → `-importing` → `-complete`). **Сконвертований в один overlay** (`#csv-modal`): 4 екрани → `.csvm-panel` всередині gm-* shell, `go('csv-*')` перехоплюється → `csvShowPanel()`; hand-rolled back-кнопки прибрані (у т.ч. дубльована back у footer csv-preview). Conform-иться під CMDB.

### 1.2 Back button

**Rule.** Кнопка «назад» на full-page екранах — це icon-only компонент **`.back-btn`**: 32×32, `border-radius:10px`, зелений tint (`bg rgba(0,175,115,0.05)`, `border rgba(0,175,115,0.35)`), стрілка-іконка 10×10 (viewBox `0 0 8 8`), hover → `bg rgba(0,175,115,0.15)` + border `#10B981`. Handler — **`navBack()`** (або доменний `checkDone()` для check-flows). Dark-safe.

НЕ робити саморобні back-кнопки з inline-стилями (`background:none;border:none` + текст «← Back» / «← Cancel»).

**Reference.** `.back-btn` CSS — `index.html:2257`. Канонічне вживання (~20 екранів): node-detail, compare-nodes, policy-detail, всі Reports views, CM group detail тощо (напр. 4512, 8277, 17075).

**Exceptions.**
- **Overlay-модалки НЕ мають `.back-btn`** — там повернення між кроками робить `gm-stepper` + кнопки `Back`/`Next` у `gm-foot`. Це коректно, не плутати з full-page back.
- **Agentless wizard** свідомо використовує текстовий `btn-outline` «← Back» / «← Back to edit» — full-page onboarding-виняток (див. 1.1). Допустимо, але з переходом CMDB/CSV на overlay бажано звести і його до `.back-btn`, якщо лишиться full-page.

**Порушники — ✅ усунені** (конвертацією CMDB/CSV в overlay, див. 1.1):
- ✅ CMDB — саморобні back/cancel у connect/loading/browse прибрані; навігація тепер через `gm-stepper` + gm-x/backdrop/Esc.
- ✅ `csv-preview` дубльована back-кнопка (хедер + футер) — прибрана; лишилась навігація через stepper.
- ✅ Плутанина «Back» vs «Cancel» у CMDB/CSV — знята разом із хедерними full-page кнопками.

### 1.3 Close

**Rule.** Overlay-модалки закриваються трьома шляхами (усі дає `openModal`): `gm-x` (×) у `gm-head`, клік по затемненню `gm-overlay`, клавіша `Esc`. За наявності незбережених змін — `confirmDirty` показує «Discard unsaved changes?». `dismissable:false` вимикає backdrop/Esc для критичних кроків.

**Reference.** `openModal` requestClose — `index.html:11633`.

**Exceptions.** Full-page create/onboarding (agentless) не має `gm-x` — вихід через Back/Cancel у хедері. Це наслідок full-page винятку 1.1.

### 1.4 Steppers (multi-step flows)

**Rule.** Кроки багатокрокового flow показує ЄДИНИЙ stepper — `gm-stepper` всередині `openModal` (опція `steps`). НЕ тримати паралельно окремі full-page step-nav елементи для того самого flow.

**Reference.** `openModal` `steps` → `gm-stepper` (`index.html:11616`).

**Технічний борг — ✅ усунено (2026-08-07):**
- ✅ `cm-step-nav`, `pc-step-nav`, `icx-step-nav` — три мертві top-nav step-блоки ВИДАЛЕНО з DOM (CM/PC/ICX тепер overlay-модалки зі своїм `gm-stepper`; `s-cm-setup` = лише прихований DOM-паркінг для вузлів модалки, `go()` туди не навігує). Прибрано і 6 JS-рядків-тоглів у двох nav-функціях. `cmSetStep` лишений як безпечний no-op (`if(!el)continue`). Верифіковано: навігація по всіх flows чиста, j3/j4 stepper на місці, консоль без помилок.
- `updateCMDBNav()` — вже раніше видалено (grep не знаходить).

### 1.5 Modal content is frameless

**Rule.** Контент всередині модалки лежить БЕЗПОСЕРЕДНЬО на `gm-body` — БЕЗ вкладених «карток-рамок» (`background:var(--bg-surface)` + `border` + `box-shadow` + `border-radius`), які обгортають секцію контенту. `gm-card` вже є єдиною рамкою модалки; будь-яка друга рамка всередині створює «вікно у вікні». Секції розділяти вертикальним ритмом (`margin`/`padding`) або тонкою лінією `border-top:1px solid var(--border)`, а не вкладеною карткою.

**Reference.** **Policy Create modal** (`openPolicyCreate`, `pc-step-1`) — поля лежать плоско на `gm-body` через `.form-group`/`.form-label`/`.form-input`, без карток-обгорток. Тепер conform-нуті: **CMDB Import** (`#cmdb-modal`) і **CSV Import** (`#csv-modal`) — 13 зовнішніх карток-обгорток сплющено (connect-форма, connecting/failed/importing/complete центровані блоки, browse class-table, credentials групи, csv template/upload/preview).

**Exceptions (це НЕ «рамки» — лишати).**
- **Radio / selection tiles** — bordered плитки вибору (Type-плитки в Policy Create, CM-picker items). Це контрол, не обгортка.
- **Info callouts** — підказки/нотатки з tint-фоном.
- **KPI stat chips** — короткі метрики-чипи (напр. «22 ready · 2 need attention»).
- **Toggle panels / inline accordion drawer** — розкривні секції (CMDB inline node-view accordion).
- **Dropzone** та **csv-col chips** — це елементи вводу/даних, не рамки-обгортки.

**Порушники — ✅ усунені** (сплющено разом із конвертацією 1.1): CMDB connect/connecting/failed/browse/credentials/importing/complete + CSV template/upload/preview/importing/complete.

---

## 3. Forms

### 3.1 Dropdown / select trigger hover

**Rule.** Усі інтерактивні тригери dropdown/select у світлій і темній темах на `hover` отримують лише рамку `rgba(16,185,129,0.32)` (`#10B981` при 32% opacity). Фон і колір тексту hover не змінюються, якщо компонент не має окремо затвердженої поведінки. Єдиний CSS-токен — `--dropdown-hover-border`; не дублювати значення кольору локальними правилами.

Стандарт автоматично охоплює native `select` і reusable-компоненти `.csel-btn`, `.ms-btn`, `.dk-form`, `.tag-filter-btn`, `.scan-menu-trigger`, `.pe-sev-btn`, `.cvp-btn`. Новий bespoke-тригер dropdown обов’язково отримує клас `.dropdown-trigger`.

**Reference.** Policies → `Filter by tag`; All Nodes → `All groups` / `All environments`; Scans → `Filters` / `Export`; Detected Drift → `Newest first` / `Group: None`.

**Exceptions.** Disabled controls не мають hover. Overflow/action menus (три крапки), primary action-кнопки, tag-add popovers і спеціально кольорові domain actions зберігають власну button-семантику — це не form dropdown/select triggers. Hover рядків усередині відкритого меню регулюється окремо й не належить до цього правила.

---
