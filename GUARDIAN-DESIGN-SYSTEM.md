# Guardian Design System Rules

Єдине джерело правди для всіх екранів Guardian. Кожне правило складається з **чотирьох** частин:

- **Rule** — що є стандартом.
- **Reference** — екран/компонент, який є еталоном (canonical example). Один, не список.
- **Examples** — де правило вже застосоване коректно; де порушене.
- **Exceptions** — коли правило можна порушити (і чому).

> Мета: будь-який новий екран звіряється з цим документом, а не з десятком попередніх. Якщо екран не відповідає правилу і не підпадає під зафіксований Exception — він неправильний.

Файл: `index.html` (single-file prototype). Посилання на рядки — орієнтовні, звіряти перед правкою.

**Розділи 1–4** описують візуальні правила (як виглядає). **Розділи 5–11 — Guardian Interaction
Standards** — описують поведінкові правила (як реагує). Поведінкові додано 12.08.2026 за підсумком
UX consistency audit (`CONSISTENCY-BACKLOG.md`) і входу розробки GWB-6638.

## Зміст
1. [Navigation](#1-navigation) — ✅ заповнено
2. [Page layout](#2-page-layout) — ✅ dark canvas
3. [Forms](#3-forms) — ✅ dropdown hover + selected state
4. [Tables](#4-tables) — ✅ table headers
5. [States](#5-states) — ✅ loading · empty · validation · action result · error · success · warning
6. [Buttons](#6-buttons) — ✅ placement · order · terminal labels · один результат — один контроль · **6.5 канонічний набір з Figma** · 6.6 сегменти й таби
7. Cards — ⏳
8. KPI — ⏳
9. Spacing — ⏳
10. [Terminology](#10-terminology) — ✅ назви кроків · вихід vs пропуск · граматика повідомлень
11. [Interaction flows](#11-interaction-flows) — ✅ confirmation · edit/save · progressive disclosure · wizard chrome
12. [Badges](#12-badges) — ✅ **канонічний компонент з Figma** · одна геометрія · 6 кольорів · 3 варіанти
13. [Неклікабельні елементи](#13-неклікабельні-елементи) — ✅ поділ чипів за поведінкою · **13.1 `.code-tag`**

---

## 1. Navigation

### 1.1 Add / Create / Connect → overlay modal

**Rule.** Будь-який flow додавання/створення/підключення сутності (Add / Create / Connect / Import) відкривається як **overlay-модалка** через уніфікований shell `openModal(opts)`, а НЕ як окрема full-page `view`.

Один shell дає безкоштовно: `gm-overlay` затемнення, `gm-card`, `gm-head` із заголовком+підзаголовком, `gm-x` close (×), опційний `gm-stepper`, `gm-body`, `gm-foot` із кнопками. Опції: `id · title · subtitle · size · steps · buttons(kind/label/id/disabled) · bodyHtml · footerLeftHtml · dismissable · confirmDirty`.

**Reference.** **CM Groups → Create group** — `openModal('cm-wizard', …)`. `openModal` визначено в `index.html:11596`. Інші коректні приклади: New/Edit credential, Integrations wizard, Edit scan schedule, Policy Create.

**Import sub-pattern (reference).** **CMDB Import** — ✅ сконвертований в overlay (`#cmdb-modal`, gm-* shell + 3-крокний `gm-stepper`, driver `cmdbShowPanel()`), тепер є **canonical reference для всіх import-flows**; **CSV Import** — ✅ conform-нутий під нього (`#csv-modal`, той самий gm-* shell + 3-крокний stepper Upload→Review→Import, driver `csvShowPanel()`).

**Add Node sub-pattern (reference).** Figma `Guardian UI Design System based on Mantine v5.10`, component frame `3165:10694` (reference modal instances `3154:27407` Light / `3154:27537` Dark). Detected-OS wizard має desktop shell `600 × 537px`: header `74px`, stepper `54px`, content `346px`, footer `63px`. Усередині: detected-OS block `550 × 60px`, full-width Primary action `550 × 44px`, OS cards `269 × 95px` з gap `14px`; від нижнього краю карток до footer divider завжди `18px`. Content Primary використовує лише канонічний `.btn-primary` з варіантами `.btn-primary--large.btn-primary--arrow`: padding `13px 24px`, gap `8px`, `DM Sans 600 14px`, gradient `#10B981 → #059669`, radius `10px`, shadow `0 4px 7px rgba(16,185,129,.30)`. Не підміняти його footer-компонентом `.gm-btn-primary`. Картка завжди має вертикальний left-aligned layout: inset `14px`, іконка `20 × 20px`, title під іконкою через `8px`, optional subtitle через `2px`. Канонічні light-theme стани беруться з Figma card components `3154:26383`: **Default** (`3150:26310`, `3150:26328`) — surface `#F8FAFC`, border `1px solid #E2E8F0`, icon `#334155`; **Hover** (`3150:26312`, `3150:26352`) — surface `#EDF9F4`, border `1px solid rgba(16,185,129,.60)`, icon `#10B981`, без shadow. Канонічні dark-theme стани беруться з Figma `3154:27718`: **Default** (`3154:27720`, `3154:27729`) — surface `#123336`, без видимого border, icon `#6EE7B7`, title `#FFFFFF`, subtitle `#94A3B8`; **Hover** (`3154:27741`, `3154:27735`) — surface `rgba(16,185,129,.10)`, border `1px solid rgba(16,185,129,.60)`, ті самі text/icon colors, без shadow. Чотири кроки: `Add node → Connect → Scan → Results`; канонічний footer містить `Back`. На вузьких екранах fixed-height вимикається й компоненти складаються вертикально. Світла й темна теми мають однакову геометрію; змінюються лише surface/border/text tokens.

**Add Node source cards.** Source of truth — Figma `3145:24091`. Кожна desktop-картка має точну зовнішню висоту `94px`, `padding: 14px 20px`, content row `64px`, border `1px`, radius `14px`, icon container `44px` і column gap `16px`; вертикальні внутрішні відступи завжди `14px` зверху та знизу. У Dark за Figma `3145:24948` дві нижні default-картки — `Bulk import from ServiceNow` та `Import from CSV` — мають surface `#123336` і не мають видимого бордера. Верхні source-картки зберігають семантичні green/blue borders; на hover нижні картки можуть показувати відповідний semantic border.

**Add Node / Step 2 method cards.** Sources of truth — Figma `3164:10368` Light та `3164:10455` Dark. Після **Add Manually** крок 1 завжди показує detected OS / OS cards; лише після вибору ОС крок 2 показує `Install an agent` та `Agentless scan`. Не міняти ці екрани місцями. Method grid `268px 268px`, gap `14px`; card `268 × 204px`, padding `20px`, radius `14px`, icon container `44px` / icon `22px` / radius `12px`, icon `#3FC878` на `rgba(63,200,120,.10)`, title offset `14px` (`DM Sans 700 14/18`), description offset `4px`, meta offset `14px` (`600 12/16`). Light Default — surface `#F8FAFC`, border `1px solid #E2E8F0`, description `#475569` `11/18`; Light Hover — surface `#EDF9F4`, border `1px solid #10B981`, без shadow. Dark Default — surface `#123336` без видимого border, description `#CBD5E1` `12/18`; Dark Hover — success surface/border без shadow. Від нижнього краю карток до footer divider завжди рівно `18px` у двох темах. Method-step shell `600 × 433px` (body `242px`); Back повертає до вибору ОС.

**Exceptions.**
- **Add node agentlessly після вибору методу** (`v-s-agentless` та connect-test) — поки full-page. Обґрунтування: складний багатосекційний onboarding (Target / CM / Credentials) + connect-test engine. Source choice, OS і method вже перенесені в overlay; detected-OS geometry звірена з Figma `3165:10694`.
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

Усі іконки закриття модальних вікон використовують один компонент **Close button** (`.gm-x`; legacy Add Node `.add-node-close` наслідує той самий контракт). Геометрія: control `28×28px`, icon `18×18px`, stroke `1.5px`, radius `6px`, без border. Хрестик будується з двох діагоналей із round line-cap; текстові символи `×` як видима іконка не використовуються.

- Light Default: прозорий фон, icon `#475569`.
- Light Hover: фон `#F1F5F9`, icon `#0F172A`.
- Dark Default: прозорий фон, icon `#CBD5E1`.
- Dark Hover: фон `rgba(255,255,255,.06)`, icon `#F8FAFC`.
- Компонент лишається нативною `<button type="button">`, має описовий `aria-label`, видимий `:focus-visible` і не застосовується до remove/clear-кнопок усередині полів, тегів чи рядків.

**Reference.** Figma `Guardian UI Design System based on Mantine v5.10`, node `3147:10053` (`Close button`: Default / Hover; Light + Dark). Поведінка закриття — `openModal` requestClose.

**Exceptions.** Full-page create/onboarding (agentless) не має `gm-x` — вихід через Back/Cancel у хедері. Це наслідок full-page винятку 1.1.

### 1.4 Steppers (multi-step flows)

**Rule.** Кроки багатокрокового flow показує ЄДИНИЙ stepper — `gm-stepper` всередині `openModal` (опція `steps`). НЕ тримати паралельно окремі full-page step-nav елементи для того самого flow. Візуальний стандарт — Figma `Guardian UI Design System based on Mantine v5.10`, node `3006:2013` (`Counter`).

Канонічна геометрія та токени:

- контейнер: `display:flex`, `gap:12px`, `padding:16px 24px 0`; кожен крок однаково розтягується (`flex:1 0 0`);
- крок: висота `38px`, контент центрований, gap `8px`, нижній padding `14px`, нижня лінія `2px`;
- номер: `22×22px`, круг `radius:11px`, DM Sans Bold `11px`;
- label: DM Sans `12px`; active — Semibold, non-active у dark — Medium;
- light active: line/circle `#10B981`, circle text `#FFFFFF`, label `#475569`;
- light non-active: line/circle border `#E2E8F0`, circle background `#FFFFFF`, number/label `#94A3B8`;
- dark active: line/circle `#10B981`, circle text `#FFFFFF`, label `#F8FAFC`;
- dark non-active: line `#04363A`, circle transparent із border `rgba(255,255,255,.20)`, number `rgba(255,255,255,.70)`, label `#64748B`;
- завершений крок використовує active-палітру та check замість номера; клік назад дозволений тільки по завершених кроках;
- для accessibility контейнер має `role="list"`, а поточний крок — `aria-current="step"`.

**Reference.** `openModal` `steps` → `gm-stepper` (`index.html`, `openModal()`); Figma node `3006:2013`.

**Examples.** Policy Create, CM Groups Create, Integrations Setup, Field Mapping, CMDB Import і CSV Import використовують той самий `.gm-stepper` без локальних перевизначень.

**Exceptions.** Full-page onboarding, який явно зафіксований як виняток у §1.1, може мати власну навігацію кроками; він не є modal wizard і не наслідує `.gm-stepper` автоматично.

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

## 2. Page layout

### 2.1 Main background

**Rule.** Основний фон усіх сторінок у темній темі — `#031516`. Єдине джерело значення — primitive-токен `--p-dark-canvas`, який через `--bg-canvas` / `--bg-page` застосовується до `body`, content-area та всіх основних page surfaces. Не задавати локальний основний фон сторінки окремими hex-значеннями.

**Reference.** Усі основні views застосунку: Dashboard, All Nodes, Scans, Detected Drift, Policies, Reports, Scan Schedules, CM Groups, Credentials та Integrations.

**Exceptions.** Картки, таблиці, sidebar, модальні вікна, dropdown-панелі, hover/selected-стани та інші підняті або інтерактивні surfaces використовують власні семантичні токени й не прирівнюються до основного фону сторінки.

### 2.2 Table row hover

**Rule.** Усі рядки даних у всіх таблицях застосунку мають єдиний hover-фон через семантичний токен `--table-row-hover`: `#F8FAFC` у світлій темі та `rgba(255,255,255,0.05)` у темній. Не задавати локальні hover-кольори, інлайн `onmouseover` або окремі hex/rgba для конкретної таблиці. Native `<table>` покриваються глобально; нові grid-таблиці повинні використовувати клас `.table-row`.

Hover змінює лише поверхню всього рядка. Текст, іконки, баджі та вкладені кнопки не успадковують окремий hover від рядка й зберігають власні інтерактивні стани.

**Reference.** All Nodes → Managed / Detected; MACBOOK-TARAS → Assigned policies / Scan history; CM Groups; Credentials; Scan Schedules; Reports.

**Exceptions.** Заголовки таблиць, footer/summary-рядки, empty states, disabled placeholders та нетабличні картки/списки не отримують table-row hover.

---

## 3. Forms

### 3.1 Dropdown / select trigger hover

**Rule.** Усі інтерактивні тригери dropdown/select у світлій і темній темах на `hover` отримують лише рамку `rgba(16,185,129,0.32)` (`#10B981` при 32% opacity). Фон і колір тексту hover не змінюються, якщо компонент не має окремо затвердженої поведінки. Єдиний CSS-токен — `--dropdown-hover-border`; не дублювати значення кольору локальними правилами.

Стандарт автоматично охоплює native `select` і reusable-компоненти `.csel-btn`, `.ms-btn`, `.dk-form`, `.tag-filter-btn`, `.scan-menu-trigger`, `.pe-sev-btn`, `.cvp-btn`. Новий bespoke-тригер dropdown обов’язково отримує клас `.dropdown-trigger`.

**Reference.** Policies → `Filter by tag`; All Nodes → `All groups` / `All environments`; Scans → `Filters` / `Export`; Detected Drift → `Newest first` / `Group: None`.

**Exceptions.** Disabled controls не мають hover. Overflow/action menus (три крапки), primary action-кнопки, tag-add popovers і спеціально кольорові domain actions зберігають власну button-семантику — це не form dropdown/select triggers. Hover рядків усередині відкритого меню регулюється окремо й не належить до цього правила.

### 3.2 Dropdown option hover

**Rule.** Усі звичайні рядки всередині відкритих dropdown/select-меню у світлій темі на `hover` використовують фон `#EDF9F4` через єдиний токен `--dropdown-option-hover`. У темній темі токен дорівнює `rgba(255,255,255,0.06)`. Нові dropdown-рядки отримують клас `.dropdown-option`; не задавати hover-фон локальними inline handlers або окремими hex/rgba значеннями.

**Reference.** Scans → меню **Add check to policy**; hover на рядку `Windows 11 Security Baseline`.

**Exceptions.** Destructive/danger options зберігають червоний hover. Disabled options не мають hover. Native `<option>` може використовувати системний стиль браузера й не гарантує кастомний hover.

### 3.3 Dropdown selected option

**Rule.** Поточне вибране значення у value-bearing dropdown завжди має видимий selected-state: фон `var(--dropdown-option-hover)` (`#EDF9F4` у світлій темі, `rgba(255,255,255,0.06)` у темній), контрастний основний текст і зелену галочку там, де компонент має single-select indicator. Недостатньо позначати вибір лише зеленим кольором тексту. Reusable `.csel` автоматично використовує `.csel-opt.sel`; bespoke-компоненти мають ставити `.is-selected` або власний задокументований active-клас. Multi-select рядки отримують той самий фон через checked-state. Action/overflow menus і Export не мають selected-state, бо не зберігають поточного значення.

**Reference.** Scan Schedules → Edit schedule → `Frequency`, `Hour`, `Minute`; All Nodes → `All groups` / `All environments`; Detected Drift → sort/group/time; Policies → `Filter by tag`; CM Groups → `Status` / `Availability`.

**Exceptions.** Action/overflow menus, Export і create-new rows не мають selected-state. У **Scans → Filters** вибрані рядки `Sort by` / `Show` не мають постійної фонової заливки: стан уже однозначно показують radio/checkbox; `var(--dropdown-option-hover)` зʼявляється лише під час hover. Це локальний виняток `#results-filter-panel`, його не переносити на інші dropdown-и. Native `<option>` може залишатися системним; для повністю контрольованого selected-state використовувати reusable `.csel`.

### 3.4 Search text

**Rule.** Усі search-поля використовують спільні theme-aware токени `--search-text` і `--search-placeholder`; локальні кольори для окремих search-полів заборонені. У світлій темі введений текст і placeholder мають колір `#475569` з `opacity:1`. У темній темі введений текст — `#96A6BD`, placeholder — `rgba(150,166,189,0.60)` з `opacity:1`. Нове поле пошуку має використовувати семантичний `type="search"` або клас `.app-search-input`; legacy-поля додатково покриваються за `id`/placeholder зі словом `search`.

**Reference.** Global Search, All Nodes, Detected Drift, Policies, CM Groups, Credentials, CI browser.

**Exceptions.** Звичайні form inputs та combobox без функції пошуку не підпадають під це правило.

### 3.5 Radio selection cards у модальних вікнах

**Rule.** Усі radio-вибори всередині `gm-overlay` використовують один компонент `.gm-radio-card` (або legacy `.radio-item`, який у модалці наслідує ті самі токени). Геометрія: `min-height:63px`, `padding:14px`, `gap:12px`, `border:1px`, `border-radius:10px`; radio — `15×15px`, внутрішня крапка — `7×7px`. Заголовок — DM Sans Semibold `13px`, опис — DM Sans Regular `11px` із верхнім відступом `2px`.

Light default: фон `#F1F5F9`, border `#E2E8F0`, radio border `#94A3B8`, title `#0F172A`, description `#475569`. Light active: фон `rgba(16,185,129,.12)`, border `rgba(16,185,129,.60)`, radio/dot `#059669`. Dark default: фон `rgba(255,255,255,.05)`, border `rgba(255,255,255,.10)`, title `#FFFFFF`, description `#CBD5E1`. Dark active зберігає той самий фон і змінює border на `rgba(16,185,129,.40)` та radio/dot на `#059669`.

Вкладений залежний dropdown належить активній картці: `.gm-radio-dropdown` зʼявляється тільки для checked / `.is-selected` стану. Не дублювати active-стилі inline або JavaScript-ом; стан виводиться через `:checked` / `.is-selected`.

**Reference.** Figma `Guardian UI Design System based on Mantine v5.10`, node `3018:4218` (`RadioButton`: Default, Active, Active + Dropdown; White + Dark).

**Examples.** Policy Create → Type; CMDB Import → Authentication; CMDB Import → Connection Manager picker.

**Exceptions.** Scans → Filters використовує компактні inline radio/checkbox у popover і має окремий selected-state без постійної фонової заливки (§3.3). Severity selectors у Add/Edit Check лишаються компактними кольоровими pills: це взаємовиключний severity-control, а не content selection card. Full-page agentless onboarding не є модальним flow і не успадковує `.gm-overlay`-правило автоматично.

### 3.6 Text inputs і textarea у модальних вікнах

**Rule.** Усі text-like поля всередині `gm-overlay` використовують єдиний modal-input pattern. Звичайний input має висоту `38px`, горизонтальний padding `14px`, border `1px`, radius `10px`; textarea — `min-height:74px`, padding `10px 14px`. Шрифт поля — DM Sans Regular `13px`. Label — DM Sans Semibold `11px`, uppercase, колір `#94A3B8`, відступ до поля `6px`; required-asterisk — `#FB3C44`. Helper text — DM Sans Regular `11px`, `#94A3B8`, відступ зверху `6px`; character counter — `10px`, `#657387`.

Light default / filled: фон `#F1F5F9`, border `#E2E8F0`, основний текст `#0F172A`, placeholder `#657387`. Light hover/focus: фон `#FFFFFF`, border `rgba(16,185,129,.60)`, ring `0 0 0 3px rgba(16,185,129,.12)`. Dark default / filled: фон `rgba(255,255,255,.05)`, border `rgba(255,255,255,.10)`, основний текст `#F8FAFC`, placeholder `#657387`. Dark hover/focus зберігає темний фон, використовує border `rgba(16,185,129,.40)` і той самий зелений ring.

Стиль застосовується централізовано до text/email/password/url/number/date/time inputs, textarea, `.form-input` та legacy `.icx-input` у shared modal shell. Не дублювати кольори, геометрію чи focus-ring локальними правилами або inline-стилями. Disabled field зберігає структуру компонента, має знижену opacity і не показує hover/focus-ring.

**Reference.** Figma `Guardian UI Design System based on Mantine v5.10`, node `3023:4716` (`Input block`: Input / Description; Default, Hover, Activated; White + Dark).

**Examples.** Policy Create; Add/Edit Check; Edit Credential; Create/Edit CM Group; Incident actions; CMDB/CSV Import; Integrations Setup — усі text-like поля в цих модалках наслідують один `.gm-overlay`-контракт.

**Exceptions.** Dropdown/select triggers регулюються §3.1–3.3; search-поля — §3.4; radio cards — §3.5. Компактні inline-редактори всередині mapping/data-grid (`.fm-edit-input`) можуть мати меншу висоту й padding, бо підпорядковуються геометрії клітинки. Full-page onboarding та auth-форми не є модальними вікнами й не успадковують правило автоматично.

### 3.7 Tags у модальних вікнах

**Rule.** Усі семантичні теги всередині `gm-overlay` використовують компонент `.gm-tag`; legacy `.tagpick-chip` у модалці автоматично наслідує той самий контракт. Tag — це компактний інтерактивний pill для вибору класифікації, а не status badge чи filter button. Label має відступ `8px` до групи; `.gm-tag-group` / `.tagpick` використовує flex-wrap і gap `6px`. Геометрія tag: horizontal padding `12px`, internal gap `6px`, border `1px`, radius `20px`; текст — DM Sans Regular `12px`. Default не має іконки; Active має галочку `12×12px` у light / `11×11px` у dark, DM Sans Semibold `12px` і видимий selected-state. Інтерактивний tag використовує `aria-pressed`; `.is-active`, `.is-on` та `[aria-pressed="true"]` є еквівалентними active-станами.

Light Default: висота `26px`, фон `#F1F5F9`, border `#E2E8F0`, текст `#475569`. Light Active: висота `28px`, фон `rgba(16,185,129,.12)`, border `#10B981`, текст `#0F172A`, галочка `#059669`. Dark Default: висота `28px`, фон `rgba(255,255,255,.05)`, border `rgba(255,255,255,.18)`, текст `#CBD5E1`. Dark Active: висота `28px`, фон `rgba(0,175,115,.15)`, border `rgba(0,175,115,.50)`, текст `#F8FAFC`, галочка `#00AF73`.

Усі нові modal tag selectors будуються через `.gm-tag` / `.gm-tag-group`; активність не оформлюється inline-стилями та не позначається лише кольором тексту. Видалення вже вибраного tag може виконуватися кліком по active-tag, але візуальним індикатором вибору залишається галочка, а не символ `×` у назві.

**Reference.** Figma `Guardian UI Design System based on Mantine v5.10`, node `3023:13076` (`Tag`: Default / Active; Light + Dark).

**Examples.** Create New Policy → Policy tags; Edit Check → Tags. Обидва modal flows використовують однаковий default/active компонент у світлій і темній темах.

**Exceptions.** Status/severity badges (`.bdg`), severity selectors, filter pills, removable free-form chips, code/literal tags (`.code-tag`) та dropdown options мають іншу семантику й не наслідують `.gm-tag`. Посилання `Manage approved tags` є окремою текстовою дією; воно не є tag-компонентом і може бути відсутнє в конкретному flow.

### 3.8 Upload block у модальних вікнах

**Rule.** Завантаження файла всередині `gm-overlay` будується як один компонент `.gm-upload-block`: `.gm-upload-title`, інтерактивна `.gm-upload-dropzone`, helper `.gm-upload-helper` та альтернативний selected-state `.gm-upload-file`. Не оформлювати dropzone inline-стилями й не змінювати кольори через `onmouseover` / `onmouseout`. Dropzone має підтримувати click, Enter / Space, drag-over і drop; hover та drag-over використовують той самий візуальний стан.

Геометрія Default: ширина `100%`, min-height `145px`, padding `32px 20px`, dashed border `2px`, radius `14px`. Upload icon — `28×28px`, відступ до тексту `10px`; primary text — DM Sans Semibold `13px`; meta — DM Sans Regular `11px`, margin-top `3px`. Заголовок секції — DM Sans Bold `13px`, відступ до dropzone `10px`. Helper розташований під dropzone або selected-file row з margin-top `8px`, DM Sans Regular `11px`, line-height `16.5px`.

Light Default: прозорий фон, border `rgba(100,116,139,.45)`, primary `#0F172A`, secondary/helper `#475569`, icon `#94A3B8`. Light Hover / Drag-over: фон `rgba(16,185,129,.12)`, border `rgba(16,185,129,.60)`. Dark Default: прозорий фон, той самий neutral dashed border, primary `#F8FAFC`, secondary `#CBD5E1`, helper `#94A3B8`, icon `#94A3B8`. Dark Hover / Drag-over: фон `rgba(16,185,129,.10)`, border `rgba(16,185,129,.60)`.

Selected-file row замінює dropzone, але зберігає title та helper. Геометрія: padding `10px 14px`, gap `10px`, solid border `1px rgba(16,185,129,.60)`, radius `10px`, file icon `16×16px`; filename — Bold `12px`, metadata — Regular `12px`, Remove — Semibold `12px`. Light background `rgba(16,185,129,.12)`, text `#065F46`; Dark background `rgba(16,185,129,.10)`, file text/icon `#CBD5E1`; Remove в обох темах `#059669`.

**Reference.** Figma `Guardian UI Design System based on Mantine v5.10`, node `3071:11173` (`Upload block`: Default / Hover / Selected; Light + Dark).

**Example.** CSV Import modal → Upload step. Dropzone, selected-file row та helper text наслідують один `.gm-upload-*` контракт у світлій і темній темах.

**Exceptions.** Кнопка завантаження готового звіту, attachments list, інтеграційні source cards і звичайні file links не є Upload block. Full-page onboarding може використовувати цей компонент лише явно; правило автоматично діє тільки всередині `gm-overlay`.

### 3.9 Tab block у модальних вікнах

**Rule.** Взаємовиключні режими всередині `gm-overlay` оформлюються єдиним сегментованим `.gm-tab-block`; кожен пункт — `.gm-tab`. Legacy `.toggle-group` / `.toggle-btn` у модалці автоматично наслідує цей контракт. Контейнер має висоту `32px`, border `1px`, overflow hidden та щільно зʼєднані сегменти без gap. Кнопка має внутрішню висоту `30px`, horizontal padding `14px`, DM Sans Semibold `11px`; сусідні кнопки розділяє border `1px`. Активність позначається `.active`, `.is-active` або `aria-selected="true"`; `role="tablist"`, `role="tab"` і синхронний `aria-selected` є обовʼязковими для нових tab blocks.

Опційний заголовок `.gm-tab-label` розташовується на `8px` вище від контейнера: DM Sans Bold `9px`, uppercase, tracking `0.6px`, колір `#94A3B8`.

Light: container radius `8px`, border/divider `#E2E8F0`, default background `#F9FAFC`, default text `#475569`; active background `rgba(0,175,115,.10)`, active text `#15803D`. Dark: container radius `10px`, border/divider `#04363A`, default background `rgba(255,255,255,.05)`, text `#FFFFFF`; active background `rgba(0,175,115,.15)`, text `#FFFFFF`.

**Reference.** Figma `Guardian UI Design System based on Mantine v5.10`, node `3045:5350` (`Tab block`: Active / Inactive; Light + Dark).

**Examples.** Integrations Setup → Authentication; CMDB Import → Windows/Linux credential source та authentication method. Усі ці modal flows використовують однакові розміри, dividers і theme states.

**Exceptions.** Верхньорівневі таби сторінок, navigation tabs, table filters, status/severity pills, radio cards та full-page segmented controls не є modal Tab block і не успадковують це правило. Для них діють власні компоненти; scoped-селектори `.gm-overlay` не повинні змінювати їхню геометрію.

---

## 4. Tables

### 4.1 Table header background

**Rule.** Усі шапки таблиць у світлій темі використовують єдиний фон `#F1F5F9` через токен `--table-header-bg`. У темній темі цей токен дорівнює `#063135`. Шапка не має нижнього бордера (`border-bottom:0`), щоб не утворювати подвійну лінію разом із межею першого рядка. Не задавати локальні кольори чи нижні бордери шапок таблиць. Для grid-таблиць використовувати `.col-thead` (або наявні canonical класи `.compare-thead` / `.drift-thead`); HTML-таблиці з `<thead>` покриваються глобальним правилом автоматично.

**Reference.** **Scan Schedules** — шапка таблиці `Node group / Status / Frequency / Time / Last run / Next run`.

**Exceptions.** Групові розділювачі, toolbar-рядки, bulk-action bars, subtotal/footer rows і заголовки карток не є шапками колонок та не використовують `--table-header-bg`.

---

# Guardian Interaction Standards

Розділи 5–11. Описують **поведінку**, а не оформлення: коли система питає, коли повідомляє, коли
показує прогрес, як називає дії. Джерело — UX consistency audit 11–12.08.2026 (38 позицій) і
підтвердження з боку розробки (GWB-6638, Connor Conway, 31.07.2026).

**Принцип, спільний для всіх правил нижче:** однакові дії поводяться однаково. Якщо два екрани
виконують дію одного класу, користувач має отримати той самий тип відповіді, тими самими словами,
в тому самому місці екрана.

---

## 5. States

### 5.1 Loading та довгі операції

**Rule.** Будь-яка операція, що триває довше ~400 мс, показує **прогрес, а не порожній екран**.
Обовʼязковий мінімум: індикатор активності + текстовий рядок, що саме зараз відбувається.
Операція, яка може тривати понад ~10 с, додатково дає користувачеві **вихід без скасування** —
кнопку «Continue in Background» — і явно повідомляє, **де зʼявиться результат**, коли він буде готовий.

Прогрес описується конкретно («Scanning us-east-1…»), а не абстрактно («Loading…»).

**Reference.** **Node Discovery** у флоу підключення AWS — спінер + рядок поточного статусу +
«Continue in Background» + фраза про те, що знайдене зʼявиться під **Detected Nodes**.
Реалізовано в GWB-6638 (CN-10) і є еталоном для всіх довгих операцій.

**Examples.**
- ✅ `j3-syncing`, `cmdb-importing`, connect-test у agentless-onboarding — покроковий чекліст із поточним станом.
- ❌ Операції, що показують лише спінер без тексту, не кажучи, чого чекати і як довго.

**Exceptions.** Операції коротші за ~400 мс індикатора не отримують — миготіння спінера гірше за
його відсутність. Фонові автооновлення (sync, polling) не блокують екран і не показують прогрес;
вони позначаються лише часом останньої синхронізації.

### 5.2 Empty states

**Rule.** Порожній список ніколи не буває просто порожнім. Канонічний блок: **icon-circle 44×44**
+ заголовок **14px / 600** + підпис **12px** + дія. Дія залежить від типу порожнечі — і типів рівно два:

| Тип | Причина | Дія |
|---|---|---|
| **no-match** | фільтр/пошук нічого не знайшов | reset-посилання «Clear filters» |
| **true-empty** | сутностей не існує взагалі | filled `btn-primary` CTA, що створює першу |

Плаский центрований рядок тексту без іконки, заголовка й дії — не empty state.

**Носій — клас, не inline.** Розмітка нового empty state не пишеться від руки:

```html
<div class="es">                       <!-- .es.es-sm — компактний варіант у панелі/картці -->
  <div class="es-ico"><svg …></svg></div>
  <div class="es-title">…</div>
  <div class="es-sub">…</div>
  <div class="es-act"><button …></button></div>
</div>
```

Клас `.es` тримає всі канонічні значення (`index.html:598`), тому новий стан не потребує **жодного**
нового CSS. Ставити `font-size`/`color` інлайн поверх `.es-*` заборонено.

**Reference.** `.es` (`index.html:598`). Робочі зразки: `nd-attrs-empty` (no-match + `Reset filters`),
`cmd-nodes-empty` (true-empty + `Assign nodes`), `intg-list` порожній (true-empty + `Add integration`).
Історичні inline-екземпляри `rc-empty-state` / `rb-empty-state` / `ri-empty-state` (Reports) несуть ті
самі значення й лишаються валідними, але новий код пишеться на `.es`.

**Examples.**
- ✅ Credentials, Nodes, Scan Schedules, CM Groups — приведені до канону 07.08.2026.
- ✅ `nd-attrs-empty`, `nd-policies-empty`, `.intg-empty`, `.id-caps-empty` (×3), `cmd-nodes-empty`,
  `report-empty` — переведені на `.es` 13.08.2026. `report-empty` знято з 16px/700 + 13px на 14/600 + 12px.
- ✅ no-match отримав робочий reset: `ndResetAttrFilters()` / `ndResetPolFilters()` чистять і пошук,
  і статус-фільтр. Reset без обробника — не reset.

**Exceptions.** Порожня комірка або порожній під-блок усередині заповненого рядка (напр. «—» у колонці)
не є empty state і не отримує іконки з CTA. Порожній стан всередині dropdown-меню або поповера
обмежується одним рядком тексту — `icon-circle 44×44` у контейнері шириною 240px непропорційний.
Чинний виняток: `recon-node-empty` («No matching nodes» у поповері Node scope) свідомо лишений
text-only.

### 5.3 Validation

**Rule.** Помилка валідації показується **біля поля, яке її спричинило**, а не глобально.
Два шасі — за типом контейнера:

- **Full-page форма:** кнопка submit **disabled**, поки форма невалідна, + текстовий hint, чого бракує.
  Користувач не має натискати, щоб дізнатися про помилку.
- **Модалка:** submit активний; при натисканні невалідні поля отримують червону рамку `#FB3C44` + фокус
  на першому невалідному, і рамка **автоматично знімається** при першому вводі в це поле.

Валідаційна помилка **ніколи не показується як toast** — toast не вказує на поле й зникає раніше,
ніж користувач встигне його виправити.

**Носій — хелпер, не копіпаст.** Обидва шасі зведені до трьох функцій (`index.html`, поруч із
`showToast`), щоб рамка й авто-очищення не переписувались на кожному call-site:

| Функція | Призначення |
|---|---|
| `vRequire(ids)` | Модалка. `ids` — id або масив id обовʼязкових полів. Ставить рамку `#FB3C44` на порожні, фокус на перше, знімає рамку при першому вводі. Повертає `true`, якщо є порожні. |
| `vFormError(host, msg)` | Помилка рівня форми: inline-блок `.v-form-err` угорі `host` (напр. `.gm-body` модалки). `msg = null` знімає блок. Знімається і сам — при першому `input`/`change` у формі. |
| `vMarkField(el)` | Позначити одне поле (нестандартна умова, не «порожнє»). |

`gmGoStep` очищає `.v-form-err` при кожній зміні кроку — помилка одного кроку не переїжджає
на наступний. Full-page шасі має два класи-носії: `.v-submit:disabled` і `.v-req-hint`.

**Reference.** Full-page — `alValidate` (agentless onboarding) і `ssValidate` (Configure Secret Server).
Модалка — `vRequire` у `icxNext` (крок 2, Instance URL); `cmCreateGroup` — історичний зразок.

**Examples.**
- ✅ `credSave`, CM group edit — auto-clear рамки при вводі (додано 07.08.2026).
- ✅ **Закрито 13.08.2026.** Пʼять валідацій, що показувались у **зеленому success-toast із галочкою**,
  12.08 переведені на варіант `error`, а 13.08 — на inline-шасі. По одному шасі на форму:
  - `ssSaveConfig` (`Instance URL and username are required`) → full-page: `ss-submit` disabled +
    hint `ss-req-hint` («Add the instance URL and a username to test the connection.»).
  - `icxNext` крок 2 (`Enter the instance URL to continue`) → `vRequire('icx-url')`: рамка + фокус
    + auto-clear.
  - `icxNext` крок 1 (`Select a provider to continue`) → `vFormError`: провайдер обирається плиткою,
    поля не існує. Знімається у `icxSelectProvider`.
  - `_fmNext` крок 0 (`Select at least one field`) і крок 1 (`Fix blocking errors before proceeding`)
    → `vFormError`: обидві стосуються набору галочок / результату тесту, а не поля.
  У продукті не лишилось жодної валідації, що виводиться через `showToast`.

**Exceptions.** Помилка, що стосується форми в цілому, а не конкретного поля (напр. «Connection test
failed», «Select a provider»), показується inline-блоком `.v-form-err` угорі форми — але теж не toast.

### 5.4 Action result (toast)

**Rule.** Результат дії, що не змінює екран, повідомляється **одним toast-компонентом** із варіантами
`success · error · warning · info`. Колір та іконка — властивість **варіанта**, не тексту повідомлення.

- Один компонент на весь застосунок. Двох паралельних систем не існує.
- Гліфи `★ ✓ ⚠` не пишуться всередині рядка повідомлення.
- Toast повідомляє про **результат**, а не про **процес** (процес — це 5.1) і не про **валідацію** (це 5.3).
- Дія, наслідок якої видно на екрані (рядок зник, бейдж змінився), toast не потребує.

**Reference.** `showToast(message, variant)` — ✅ реалізовано 12.08.2026. Варіант керує фоном
(`#success-toast[data-variant]`), іконкою (`_TOAST_ICON`) і тривалістю (`_TOAST_MS`: success/info 2800 мс,
warning/error 3800 мс — помилка має жити довше, бо на неї треба відреагувати).

| Варіант | Фон | Іконка |
|---|---|---|
| `success` | `#166534` | галочка |
| `error` | `#991B1B` | коло з × |
| `warning` | `#92400E` | трикутник |
| `info` | `#0F172A` | коло з i |

Фон задається **CSS-правилом за `[data-variant]`**, не inline — інакше `_syncInlineTheme` мутує його
й ламає колір. Фон однаковий у light і dark: toast — суцільна темна поверхня.

**Examples.**
- ✅ Одна система. `scToast` тепер тонкий аліас → `showToast(msg, 'info')`; окремий одноразовий div
  із hardcoded `#0F172A` видалено.
- ✅ Захисні розгалуження `typeof scToast === 'function' ? … : showToast(…)` знешкоджені — обидва імені
  ведуть в один компонент.
- ✅ Гліфи винесено з тексту у варіант: `'★ Baseline set to …'` → `showToast('Baseline set to …', 'success')`;
  `'✓ … revalidated'` → `'success'`; `'⚠ … still failing'` → `'warning'`.
- ✅ Зворотна сумісність: `showToast(msg)` без варіанта = `success`, тому ~25 наявних викликів не змінювались.

**Exceptions.** Незворотна дія над persisted-сутністю може показати toast із **undo** — тоді тривалість
збільшується, бо користувач має встигнути відреагувати. Це єдиний випадок, коли toast несе дію.

### 5.5 Error

**Rule.** Помилка завжди має **червону** семантику і завжди дає **шлях відновлення**. Носій обирається
за масштабом:

| Масштаб | Носій |
|---|---|
| Поле форми | inline під полем (5.3) |
| Дія на екрані | toast варіанта `error` (5.4) |
| Крок флоу не вдався | inline-блок у місці кроку + кнопка retry |
| Сторінка/дані недоступні | повноекранний error screen із поясненням і дією |

Текст помилки називає **причину** і **наступну дію**. «Something went wrong» без дії — не помилка,
а тупик.

**Reference.** Connect-test у agentless-onboarding — фейл показує деталь причини й дозволяє
виправити ввід і перетестувати **не виходячи з кроку** (той самий патерн, що CN-03 у GWB-6638).

**Examples.**
- ✅ AWS Stage 1 у реалізації: account ID можна виправити й перетестувати всередині Stage 1.
- ✅ **Виправлено 12.08.2026 — але лише семантику кольору.** Пʼять помилок валідації показувались
  у **зеленому** success-toast із галочкою: помилка виглядала як підтвердження успіху. Було
  найгрубішим порушенням семантики кольору в продукті; переведені на варіант `error`
  (`#991B1B` + коло з ×). ⚠️ **Носій усе ще неправильний** — за 5.3 валідація має бути inline
  біля поля, а не toast. Не вважати пункт закритим.

**Exceptions.** Немає. Помилка, показана не червоним, — дефект.

### 5.6 Success

**Rule.** Успіх підтверджується **найтихішим достатнім способом**. Ієрархія, згори вниз:

1. Зміна на екрані сама по собі (рядок додався, статус змінився) — нічого більше не потрібно.
2. Toast варіанта `success` — якщо результат не видно.
3. Success-екран — **лише** для завершення багатокрокового флоу, з чіткою наступною дією.

Текст: `<Сутність> <дієслово в past tense>` — «Policy saved», «Credential updated». Слово
**«successfully» не використовується**: носієм успіху є сам компонент, а не текст.

**Reference.** Success-екран завершення onboarding — підсумок + одна очевидна наступна дія.

**Examples.**
- ❌ Неузгоджена граматика: «Policy saved **successfully**» (8472) поруч із «Assignment saved» (8700),
  «Draft saved» (10507), «Credential saved» (12054), «Mapping activated **successfully**» (10538).

**Exceptions.** Незворотні або дорогі операції (видалення інтеграції, імпорт сотень нод) можуть
отримати success-екран навіть поза багатокроковим флоу — користувачеві потрібне однозначне
підтвердження, що операція завершилась.

### 5.7 Warning

**Rule.** Warning — це стан, що **не блокує**, але змінює рішення користувача: «дія спрацює, але
наслідок може бути не той, якого ти очікуєш». Показується **до** дії, а не після.

Відмінності: **error** = не вийшло; **warning** = вийде, але зважай; **info** = просто контекст.
Якщо повідомлення не змінює поведінку користувача — це не warning, а шум.

**Reference.** Impact preview у `credDelete` (12160) — перед видаленням показує, **скільки нод
залежать** від цього credential. Користувач бачить наслідок до підтвердження.

**Examples.**
- ✅ `credDelete` — еталон «destructive with impact preview».
- ✅ **Виправлено 12.08.2026.** `'⚠ … still failing'` — warning був закодований гліфом у тексті;
  тепер `showToast(…, 'warning')`, гліф і колір бере варіант.

**Exceptions.** Warning не використовується для повідомлення про успішний результат із застереженням
(«Imported 8 of 10») — це `info` із деталізацією, не warning.

---

## 6. Buttons

### 6.1 Розміщення та порядок дій

**Rule.** Головна дія екрана або кроку — **праворуч**, вторинні — ліворуч від неї, у тому самому ряду.
Порядок: `[вторинні …, головна]`, головна завжди остання. Дві головні дії в одному ряду не існують.

- **Overlay-модалка:** дії в `gm-foot`; Back — у `.gm-foot-left`, основна група — у
  `.gm-foot-right`. `Cancel` або supportive secondary стоїть ліворуч від Primary у правій групі.
- **Full-page екран:** головна дія праворуч у хедері, навігація назад — ліворуч (див. 1.2).
- Destructive-дія займає позицію головної, але має `kind:'destructive'`.

**Reference.** `openModal` footer — `.gm-foot-right`, кнопки рендеряться в порядку масиву `opts.buttons`.

**Examples.**
- ✅ Усі перевірені call-sites `openModal` дотримуються `[Cancel/secondary, …, primary/destructive]`.
- ⚠️ API `openModal` приймає довільний `buttons[]` і **не валідує порядок** — правило тримається на
  дисципліні. Контракт зафіксовано тут; при додаванні нової модалки звіряти явно.

**Exceptions.** Toolbar-кнопки над таблицею та row-level actions не є footer-діями і живуть за
власним правилом 6.4.

#### Канонічний footer модального вікна

**Source of truth:** Figma `eXz33Qu7v58JjOiatTptAE`, component set `3023:5088`.

Footer відділений від body однією верхньою лінією: Light `#E2E8F0`, Dark
`rgba(255,255,255,0.06)`. Геометрія незмінна між темами: `padding:10px 24px 12px`, мінімальна
висота `54px`; усі кнопки `h32`, `radius:10px`, `DM Sans 600 12px`. Gap усередині лівої групи —
`8px`, між двома правими кнопками — `10px`.

Дозволені лише чотири композиції:

1. одна Primary праворуч;
2. Back ліворуч + Primary праворуч;
3. Back ліворуч + supportive Secondary і Primary праворуч;
4. лише Back ліворуч.

Back у footer — це `Third`, а не нейтральна прямокутна кнопка: квадрат `32×32` зі стрілкою +
підпис **поза рамкою**, gap `8px`. У `openModal` використовувати `backButton`, не вставляти
`← Back` вручну через `footerLeftHtml`. Primary: gradient `#10B981 → #059669`, horizontal padding
`20px`, білий текст, стрілка праворуч, shadow `0 4px 7px rgba(16,185,129,.30)`. Supportive
Secondary: Light `rgba(0,175,115,.05)` / `#9DE0C9`; Dark `rgba(0,175,115,.15)` /
`rgba(0,175,115,.35)`.

`Cancel` / `Close` залишаються нейтральними, бо вони не мають читатися як друга позитивна дія.
Warning і destructive дії зберігають семантичний колір; геометрія footer-кнопки лишається
канонічною. Довільний HTML у `footerLeftHtml` дозволений лише для контекстної допоміжної дії
(наприклад, Test), але не для Back або основних submit-дій.

### 6.2 Лейбли термінальних дій

**Rule.** Слово на кнопці описує, **що станеться**, а не де ти в інтерфейсі. Три різні дії — три різні слова:

| Лейбл | Коли | Що робить |
|---|---|---|
| **Save changes** | редагування наявної сутності | зберігає зміни, лишає користувача в контексті |
| **Done** | завершення флоу, де вже все збережено | лише закриває, нічого не зберігає |
| **Confirm** | підтвердження вибору перед виконанням | фіксує вибір, далі йде дія |

Не використовувати «Apply», «OK», «Submit» — вони не кажуть, що станеться.

**Reference.** Policy Edit — `Save changes` при виході з full-page редагування.

**Examples.**
- ✅ «Save changes» — policy-edit (8472), policy-assign (8700), edit-check (8997).
- ⚠️ «Done» — CM wizard finish (11487), CI picker (19041): перевірити, що там справді нічого не зберігається.
- ⚠️ «Confirm» — j3 node match (16349, 16355, 16372): це підтвердження вибору, лейбл коректний.

**Exceptions.** Доменні дієслова сильніші за загальні: «Reconcile», «Approve», «Disconnect»,
«Run scan» лишаються як є — вони конкретніші за «Confirm» і не підпадають під таблицю вище.

### 6.3 Один результат — один контроль

**Rule.** Два контролі, що ведуть до одного результату, ставити поруч заборонено — користувач обере
не той. Якщо на кроці вже є «Continue», окремий «Skip» **не потрібен**: Continue має проходити далі
й без вибору.

Вихід із флоу і пропуск кроку — **різні** дії й не називаються одним словом (див. 10.2).

**Reference.** **Node Inventory** у GWB-6638 — Skip прибрано, Continue проходить далі з порожнім вибором.
Обґрунтування розробки: *«two controls for one outcome invite the wrong one»* (CN-12).

**Examples.**
- ❌ `index.html:16288` — «Skip for now» стоїть поруч із «Continue to Scan Setup» (16290), причому
  Continue веде на наступну стадію, а Skip викидає на dashboard. Різні наслідки, схожі лейбли.

**Exceptions.** Справді необовʼязковий крок (напр. вибір credential, коли він не потрібен) може мати
явний пропуск — але тоді на кроці **немає** окремого Continue, пропуск і є проходженням далі.

### 6.4 Кнопка дії в рядку/смузі

**Rule.** Будь-який рядок списку або смуга-картка з кнопками дії (`View`, `Edit`, `Review`,
`Reconcile`, …) використовує канонічний **`Secondary`** із 6.5 — єдиний тип для цього патерну app-wide.
Власних кольорів це правило **не задає**: значення беруться з 6.5 і живуть у базових класах.

Новий рядок-із-кнопкою не потребує **жодного** нового CSS — достатньо класу. Скоупити під клас
рядка/картки можна лише **геометрію** (min-width, вирівнювання); колір, розмір і стани
перевизначати заборонено. Ніколи inline — `_syncInlineTheme` мутує inline-фони.

**Reference.** Бази `.btn-secondary` (~1153) і `.action-cta` (~2114) — обидві на значеннях 6.5
`Secondary s=32`. Зразок вживання: Dashboard → «Needs your attention».

**Examples.**
- ✅ 12.08.2026 застосунок переведено на 6.5 за один прохід: **54** екземпляри `.action-cta` +
  `.btn-secondary` дають ОДНУ computed-сигнатуру в кожній темі.
- ✅ Видалено як дублікати: `.action-row .action-cta`, `.node-row .btn-secondary`, `.remed-risk-cta`,
  `.remed-back-results-cta`, `.remed-post-risk-cta`, кольори `.schedule-action-cta` і `.detected-row-cta`.
- ⚠️ Інлайнові `font-size` / `padding` переважують канон — при міграції знято ~49 таких атрибутів.
  Після будь-якої зміни базового класу перевіряти інлайни.

**Exceptions.** Destructive row-action (напр. `Disconnect`) зберігає червону семантику. Icon-only
overflow-меню (три крапки) — це `Back` із 6.5, а не кнопка дії в рядку.

### 6.5 Канонічний набір кнопок (Figma) — ЄДИНЕ ДЖЕРЕЛО

**Rule.** В застосунку існує **рівно пʼять** типів кнопок: `Primary`, `Secondary`, `Text button`,
`Third`, `Back`. Створюючи будь-що нове, брати кнопку **звідси** — не вигадувати власну й не копіювати
старі inline-стилі. Джерело: Figma «Guardian UI Design System based on Mantine — v5.10», секція `Buttons`
(fileKey `eXz33Qu7v58JjOiatTptAE`, node `2988:70531`). Оновлено з макета 2026-08-12 (**4-та ревізія**).

**Спільне для всіх типів:** шрифт `DM Sans` **600** (SemiBold), `font-size:12px`, `border-radius:10px`,
текст центрований. Gap між текстом та іконкою — `4px`.

#### Primary — `Thema=Both` (однаковий у light і dark)

| Стан | Значення |
|---|---|
| default | bg `linear-gradient(161.24deg, #10B981 0%, #059669 100%)` · text `#FFFFFF` · shadow `0 4px 7px rgba(16,185,129,0.3)` |
| hover | bg `linear-gradient(65.09deg, #2DCE94 3.4%, #17BE85 86.55%)` · shadow без змін |
| disable | = default + `opacity:0.4` |
| loading | = default + `gap:4px` + спінер **після** тексту: бокс `12px`, гліф `8px` |

Padding `10px 20px`. Розміри: **M = 36px**, **s = 32px**. Іконки: `None / Left / Right`.

#### Secondary — light і dark ЗАДАНІ ОКРЕМО, лише розмір **s (32px)**

Повна матриця: Light + Dark × Default / Hover / Disable / Loading × None / Left / Right.

| Тема | Стан | Значення |
|---|---|---|
| Light | default | bg `rgba(0,175,115,0.05)` · border `1px #9DE0C9` · text `#334155` |
| Light | hover | bg `rgba(0,175,115,0.15)` · border `1px rgba(0,175,115,0.35)` · text `#334155` |
| Dark | default | bg `rgba(0,175,115,0.15)` · border `1px rgba(0,175,115,0.35)` · text **`#FFFFFF`** |
| Dark | hover | bg `rgba(0,175,115,0.20)` · border `1px rgba(0,175,115,0.41)` · text **`#FFFFFF`** |
| Light/Dark | disable | = свій default + `opacity:0.4` |
| Light/Dark | loading | = свій default + `gap:4px` + спінер **після** тексту: бокс `8px`, гліф `8px` |

Padding `10px 20px`, height `32px`.
⚠️ Light `default` має **суцільний** `#9DE0C9`, а `hover` — прозорий `rgba(0,175,115,0.35)`. Візуально
на білому це майже те саме (`0.35` over white ≈ `#A6DFCC`), але це два різні способи задати одне.

#### Text button — `Thema=Both`, без розміру

Без фону, бордера й падінгу. Height `16px`. Text `#059669`, hover `#10B981`.
Права іконка: бокс `8px`, гліф `6px`, gap `4px`.
Стани: Default / Hover / Disable / Loading (іменування в бібліотеці **виправлено**).
У `loading` спінер (бокс `8px`) стоїть **між текстом і стрілкою**: `text · ↻ · →`.

#### Third — icon-square + підпис ЗЗОВНІ

Це **не** кнопка з іконкою — це квадрат `32×32` з іконкою **плюс окремий текст поруч**, gap `8px`.
Патерн «назад»: `[←] Button text`, де рамка охоплює **тільки стрілку**.

| Тема | Стан | Бокс | Підпис |
|---|---|---|---|
| Light | default | bg `rgba(0,175,115,0.05)` · border `1px rgba(0,175,115,0.35)` | `#334155` |
| Light | hover | bg `rgba(0,175,115,0.15)` · border `1px #10B981` | `#334155` |
| Dark | default | bg `rgba(0,175,115,0.05)` · border `1px rgba(0,175,115,0.35)` | **`#FFFFFF`** |
| Dark | hover | bg `rgba(0,175,115,0.15)` · border `1px #10B981` | **`#FFFFFF`** |
| Light/Dark | disable | = свій default + `opacity:0.4` | — |

⚠️ **Тема впливає ЛИШЕ на колір підпису/гліфа.** Сам бокс (`bg .05` / `border .35`) ідентичний
у light і dark. Тому `body.dark … border-color` для `Third` і `Back` писати не треба — такий
оверрайд є багом.

Іконка: бокс `10px`, гліф `7.5px` (ліва стрілка = `rotate(180deg)`). Radius `10px`.
Підпис: `DM Sans 600 12px`, **поза** рамкою. Стану `loading` немає.

#### Back — той самий квадрат, але БЕЗ підпису

Окремий варіант для «голої» кнопки повернення: квадрат `32×32`, всередині лише стрілка, тексту немає.
Бокс ідентичний до `Third`; стани Default / Hover / Disable, Light + Dark.

| Тема | Стан | Значення |
|---|---|---|
| Light | default | bg `rgba(0,175,115,0.05)` · border `1px rgba(0,175,115,0.35)` · radius `10px` |
| Light | hover | bg `rgba(0,175,115,0.15)` · border `1px #10B981` |
| Dark | default | bg `rgba(0,175,115,0.05)` · border `1px rgba(0,175,115,0.35)` |
| Dark | hover | bg `rgba(0,175,115,0.15)` · border `1px #10B981` |
| Light/Dark | disable | = свій default + `opacity:0.4` |

Іконка: бокс `10px`, гліф `7.5px`, `rotate(180deg)`. Стану `loading` немає.
**Коли що:** є підпис («Back to nodes») → `Third`; лише стрілка в шапці → `Back`.

**Reference.** Figma-токени (3-тя ревізія — перейменовані з `new button/*` на **`new buttons/*`** і тепер
семантично-нейтральні): `bg/teal-light-bg` `#00AF730D` (0.05) · `bg/teal-hover-bg` `#00AF7326` (0.15) ·
`bg/teal-dark-hover-bg` `#00AF7333` (0.20) · `border/teal-border` `#00AF7359` (0.35) ·
`border/teal-dark-hover-border` `#00AF7369` (0.41) · `border/mint-border` `#9DE0C9` ·
`bg/teal-accent` `#00AF73` · `bg/emerald-500` `#10B981` · `bg/emerald-600` `#059669` ·
`bg/white` `#FFFFFF` · `bg/slate` `#334155`. Тип: `button/Button text` = DM Sans SemiBold 12/100.

**Examples.**
- ✅ Шрифт збігається з застосунком — `DM Sans` уже базовий (`index.html:233`).
- ✅ `Text button` → наявний `.action-arrow` (8×8 viewBox) сумісний за геометрією.
- ✅ `Third` / `Back` → наявний патерн back-кнопки (1.2).
- ✅ **Міграція виконана 12.08.2026.** Старі значення (light bg `#F3FAF6`, border
  `rgba(16,185,129,0.32)`) у коді більше не існують: бази `.btn-secondary` і `.action-cta`
  переписані на 6.5, дублікати видалені. 6.4 більше не тримає власної таблиці значень і
  посилається сюди.
- ✅ `.btn-outline` (Cancel / Skip / нейтральні дії) взяв **форму** канону — h32 · r10 · 12/600 ·
  gap 4 · `padding:0 14px` · `:disabled opacity .4` — але лишив нейтральні кольори, бо в наборі
  немає нейтрального типу (див. дефект 8). 36 екземплярів → одна сигнатура.

**Exceptions / відомі дефекти макета** (перевірити з дизайнером до міграції):
1. **`Text button` = `Thema=Both` з текстом `#059669`.** Це єдиний тип, який НЕ отримав темного варіанта.
   `#059669` на нашому темному полотні (`#00262A`) дає низький контраст. Потрібен dark-варіант
   (кандидат — `#10B981` або `#9DE0C9`); до рішення в dark використовувати світліший зелений.
2. У `Secondary` **немає розміру M** (у Primary є M і s).
3. У `Third` і `Back` **немає** стану `loading`.
4. Бордер `Third` / `Back` = `0.909px` — артефакт масштабування, застосовувати **1px**.
5. Градієнти Primary мають різний кут (default 161.24°, hover 65.09°) — помітний зсув напрямку
   на hover, а не лише зміна кольору. Підтвердити, чи навмисно.
6. `Secondary` Light: `default` — суцільний `#9DE0C9`, `hover` — прозорий `rgba(0,175,115,0.35)`.
   Два різні способи задати майже однаковий колір; варто звести до одного.
7. Дрібне: в інстансі `Text button` підпис лишився «Dark thema» замість «Button text»;
   у dark-рядках `Secondary` третій ряд підписаний «left icon», хоча стрілка справа.
8. **У наборі немає нейтрального типу.** Усі пʼять — зелені або градієнтні. `Cancel`, `Skip`, `Close`
   поруч із `Primary` дали б два зелені контроли підряд, тому для них довелось лишити нейтральний
   `.btn-outline` поза каноном. Потрібен шостий тип або явний дозвіл на нейтральний варіант `Secondary`.

**Виправлено в 4-й ревізії (12.08.2026):** `Third` і `Back` у Dark мають **той самий** бордер
`rgba(0,175,115,0.35)`, що й у Light — суцільний `#00AF73` був помилкою макета. Тема змінює лише
колір підпису/гліфа.

**Виправлено в 3-й ревізії:** `Secondary` Dark — текст став **білим**, а hover отримав власні значення
(`0.20` / border `0.41`), тож ховер у темній темі більше не «мертвий»; токени перейменовані
на нейтральні `new buttons/*`.

**Виправлено в 2-й ревізії:** стани `Text button` більше не звуться `Variant=Frame 1`; `Secondary`
отримав повну матрицю станів та іконок для обох тем; `Third` отримав Light+Dark; додано `Back`.

### 6.6 Сегментовані контроли й таби

**Rule.** Це **два різні компоненти**, і плутати їх не можна. Вибір робиться не за виглядом, а за тим,
**що саме перемикається**:

| Компонент | Коли | Ознаки |
|---|---|---|
| **Таби** — `.policies-tabs` / `.pol-tab` | перемикають **фільтр-вимір сторінки**: взаємовиключні зрізи одного списку, у кожного свій обсяг | підкреслення активного + **лічильник у баджі**, стоять на горизонтальній лінії |
| **Сегментований контрол** — `.seg` / `.seg-btn` | перемикає **режим або джерело** всередині тулбару; обсяг не показується | одна зʼєднана доріжка з роздільниками, компактна, **без лічильників** |

Сегментований контрол — це ОДНА доріжка. Окремі плаваючі пігулки сегментованим контролом не є.

**Таби.** Смуга `h42`, `border-bottom:1px var(--border)`. Таб `13px/500` → активний `600`,
`border-bottom:2px` кольору тексту, `margin-bottom:-1px` (лягає на лінію смуги). Бадж `.pol-tab-count`
`22×22`, `radius:999px`, `11px/700`.

**Сегментований контрол.** Доріжка: `h32` · `radius:10px` · `var(--bg-surface)` · `1px var(--border)` ·
`overflow:hidden`. Сегмент: `padding:0 14px` · `12px/600` · `border-right:1px var(--border)`
(немає в останнього). **Драбинка тінтів = тінти `Secondary` своєї теми, нових кольорів не вигадано:**

| Стан | Light | Dark |
|---|---|---|
| спокій | прозорий · `var(--text-sec)` | прозорий · `var(--text-sec)` |
| hover | `rgba(0,175,115,0.05)` = Secondary **default** | `rgba(0,175,115,0.15)` = Secondary **default** |
| вибраний | `rgba(0,175,115,0.15)` = Secondary **hover** · `#334155` | `rgba(0,175,115,0.20)` = Secondary **hover** · білий |

`.toggle-group` / `.toggle-btn` — **форм-варіант того самого контролу**: ті самі правила, плюс
`display:flex` + `flex:1`, щоб доріжка займала ширину form-group. Клас активного — `.active`.

JS **не фарбує** сегменти інлайн — лише `classList.toggle` + `aria-pressed`. Інлайн-фон мутує
`_syncInlineTheme`.

**Reference.** Сегментований контрол — `#node-filter-btns` (All / Local / AWS). Таби — `#results-pills`
на Scans.

**Examples.**
- ✅ 12.08.2026 пʼять незалежних реалізацій зведено до однієї: `#node-filter-btns`,
  `#detected-source-btns`, `#peer-group-btns`, `.rep-seg`, `.toggle-group` — **7 доріжок / 17 сегментів**.
  Клас `.rep-seg-btn` і мертвий `.dash-seg` видалені; прибрано ~30 рядків інлайн-перефарбування в JS.
- ✅ Detected Drift: фільтри — це фільтр-вимір із лічильниками, тому **таби**, а не `.seg`.
- ❌ Колір активного таба в dark заданий **лише для `.remed-tabs`** (`#00AF73`); решта груп
  успадковують `var(--green2)` = `#059669` — той самий низький контраст на темному полотні,
  що й дефект 1 у 6.5.
- ❌ Білий текст у баджі активного таба патчиться **пʼятьма** id-скоупленими правилами
  (`#v-policies`, `#nodes-tabs`, `#results-pills`, `#sched-scope-tabs`, `#drift-filter-btns`)
  поверх базового `#00AF73`. Якщо виняток діє для всіх груп — це не виняток, а базове правило.

**Exceptions.** Таб без лічильника допустимий, якщо обсяг зрізу неможливо порахувати дешево.
Зворотне — сегментований контрол із лічильниками — не допускається: лічильник є ознакою табів.

---

## 10. Terminology

### 10.1 Назва кроку = заголовок сторінки кроку

**Rule.** Крок у stepper і заголовок сторінки, на яку він веде, називаються **дослівно однаково**.
Розбіжність робить індикатор недостовірним: користувач читає одну назву в навігації і іншу в контенті.

Кількість кроків у stepper дорівнює кількості реальних стадій. Стадія, якої немає в stepper, не існує.

**Reference.** AWS-інтеграція в реалізації GWB-6638 — три стадії:
**Connect AWS · Node Discovery · Node Inventory** (CN-08).

**Examples.**
- ❌ `index.html:3944-3950` — `#j3-step-nav` показує вайрфреймові чотири стадії
  `Connect AWS · Sync Nodes · Run Scan · Results`, тоді як сторінки називаються **Node Discovery** (16070)
  і **Node Inventory** (16196), а `Node Inventory` у stepper відсутній узагалі.

**Exceptions.** Скорочення назви в stepper допустиме, якщо повна не вміщується — але лише усіканням
із збереженням початку («Node Discovery» → «Discovery»), не синонімом.

### 10.2 Вихід із флоу vs пропуск кроку

**Rule.** Це дві різні дії, і вони не називаються одним словом.

| Дія | Лейбл | Результат |
|---|---|---|
| Вийти з флоу | **Exit** | флоу покинуто, користувач повертається на попередній екран |
| Пропустити необовʼязковий крок | **Skip** | флоу триває, крок лишається незаповненим |

Одне формулювання на кожну дію в межах усього продукту.

**Reference.** `Exit` у stepper-хедері онбордингу.

**Examples.**
- ❌ Пʼять формулювань однієї дії: «Skip for now» (16288, 19702, 19737), «Skip for now — return to
  Dashboard» (16968), «Skip — return to Dashboard» (20290), «Skip — show results →» (19147),
  «Skip» (15483, 16364, 16381). Частина веде на dashboard, частина — далі по флоу; лейбл не дає
  передбачити результат.

**Exceptions.** Немає.

### 10.3 Словник статусів інтеграції

**Rule.** Життєвий цикл інтеграції описується закритим набором:
**Available → Connect → Connected → Attention → Error → Disabled**. Інших слів не вводити.

Статуси **capability** — окремий закритий набір: **Enabled / Disabled / Issue**. Два набори не змішуються.

**Reference.** Мапа `INTG_STATUS` (~9277).

**Examples.** ✅ `notset` → «Available» (узгоджено із заголовком секції «Available integrations»
і кнопкою `Connect`); ✅ `attention` → «Attention».

**Exceptions.** Немає.

### 10.4 Граматика повідомлень

**Rule.** Повідомлення про результат: `<Сутність> <дієслово в past tense>`. Без «successfully»,
без крапки в кінці, без гліфів у тексті.

**Reference.** «Credential updated».

**Examples.** ✅ «Assignment saved», «Draft saved». ❌ «Policy saved successfully» (8472),
«Mapping activated successfully» (10538).

**Exceptions.** Повідомлення з числами конкретизує обсяг: «8 nodes imported» — коректно.

---

## 11. Interaction flows

### 11.1 Confirmation

**Rule.** Підтвердження вимагають **persisted-сутності**. Під-елементи ще незбереженої форми
видаляються миттєво, без питання.

| Що видаляється | Підтвердження |
|---|---|
| Збережена сутність (policy, credential, node group, saved view, інтеграція) | **так** — модалка |
| Тег/рядок/елемент усередині форми, яку ще не збережено | **ні** — миттєво |

Діалог підтвердження — завжди `openModal` size `sm`, кнопки `[Cancel, <Дієслово> (destructive)]`.
Дієслово на кнопці називає дію («Delete policy»), не «OK». Де наслідок неочевидний — додається
impact preview (5.7).

**Нативний `window.confirm()` не використовується.** Він не стилізується, не темизується,
не локалізується і виглядає як системна помилка, а не як частина продукту.

**Reference.** `confirmDiscard()` — канонічний діалог «Discard unsaved changes?»
(size `sm`, `[Keep editing, Discard changes (destructive)]`), ✅ реалізовано 12.08.2026.
Для видалення з impact preview — `credDelete` → `credConfirmDelete`.

**Examples.**
- ✅ `checkDelete`, `policyDelete`, `policyEditDelete`, `cmConfirmRemove` — `openModal` + `kind:'destructive'`.
- ✅ Миттєво й правильно (під-елементи незбереженої форми): `ssRemoveFolder`, `pdRemoveTag`,
  `peRemoveTag`, `ecRemoveValueCheck`, `ciRemove`.
- ✅ **Виправлено 12.08.2026.** `confirmDirty` у `openModal` викликав нативний
  `window.confirm('Discard unsaved changes?')` — єдиний браузерний діалог у продукті з повністю
  кастомною модальною системою. Замінено на `confirmDiscard()`. У кодовій базі не лишилось
  жодного `window.confirm`.
- ❌ `deleteSavedView` — persisted-сутність зникає миттєво, без підтвердження і без toast. Відкрито.

**Exceptions.** Дія з доступним undo (5.4) може обійтися без модалки — undo замінює підтвердження.
Одне з двох, не обидва.

### 11.2 Edit / save

**Rule.** Шасі редагування визначається контейнером, і воно фіксоване:

| Контейнер | Ліворуч | Праворуч | Cancel |
|---|---|---|---|
| **Full-page edit** | `.back-btn` (1.2) | `Save changes` | **немає** — back і є скасуванням |
| **Overlay-модалка** | — | `[Cancel, Save changes]` у `gm-foot` | є |

Дубльований `Cancel` поруч із back на full-page — надлишковий контрол (порушує 6.3).
Незбережені зміни при виході → підтвердження за 11.1.

Редагування наявної сутності відбувається **там, де вона живе** — не через повторний прохід
майстра створення. Якщо сутність налаштовується майстром, її сторінка керування має ті самі
секції, що й майстер, і дозволяє перевірити зміну на місці (retest).

**Reference.** Сторінка керування AWS-інтеграцією в GWB-6638 (CN-13) — редагування scan config,
retest і delete в одному вікні, тим самим пʼятикроковим чеклістом, що й у майстрі.

**Examples.** ✅ policy-edit, policy-assign — стандартизовано 31.07.2026: лівий `← Back` +
`Save changes`, дубльований правий `Cancel` прибрано.

**Exceptions.** Inline-редагування одного поля (перейменування на місці) не має ані back, ані
`Save changes` — Enter зберігає, Esc скасовує.

### 11.3 Progressive disclosure

**Rule.** Система не робить широкий незворотний вибір за користувача. Там, де операція має обсяг
(що саме сканувати, які ресурси імпортувати, які ноди взяти під керування), користувач **обирає
явно**, а не отримує «все» за замовчуванням.

Вибір подається **згрупованим за змістом**, а не пласким списком, і показує лічильник обраного.
Обовʼязково пояснюється, що станеться з **необраним**.

**Reference.** **Node Discovery** у GWB-6638 (CN-07) — типи ресурсів згруповані за сервісом
(Compute / Database / Storage / Identity & Security / Networking / Management & Monitoring),
з лічильником `2/4` і `Select all` на групу. Необране лишається під **Detected Nodes**.

**Examples.**
- ✅ Node Inventory — перелік знайденого з фільтром за типом і per-item вибором (CN-11).
- ✅ CM-picker в agentless-onboarding — фільтрація за сумісністю протоколів замість повного списку.

**Exceptions.** Обовʼязкові технічні передумови (регіони, без яких флоу не працює) можуть бути
попередньо вибрані — але лишаються видимими й редагованими.

### 11.4 Wizard chrome

**Rule.** Усі багатокрокові майстри Guardian мають однакове шасі. Пʼять правил, підтверджених
розробкою (GWB-6638 §5) і погоджених Bambuk 12.08.2026:

1. **Індикатор стадій не містить під-стадій.** Під-стадія робить індикатор недостовірним — він
   перестає казати, де ти.
2. **`Next` завжди рухає індикатор на наступну стадію.** Кнопка, що відкриває новий екран, лишаючи
   індикатор на місці, ламає очікування, яке цей індикатор задав.
3. **Ліва панель і номер стадії присутні на всіх стадіях.** Зникнення панелі на середині читається
   як вихід із флоу.
4. **`Back` завжди top-left, в одному стилі** (`.back-btn`, 1.2). Навігація не рухається між кроками
   одного флоу.
5. **Кожен екран флоу — нумерована стадія.** Сторінок поза нумерацією, з яких немає вороття,
   не існує. Помилка виправляється **всередині стадії**, без виходу з неї.

**Reference.** Флоу підключення AWS у реалізації GWB-6638 — три стадії
**Connect AWS · Node Discovery · Node Inventory**, верифіковано end-to-end на реальному AWS-акаунті.

**Examples.**
- ✅ Stage 1 «Connect AWS» — розділено на нумеровані секції: що вводить користувач / що генерує
  Guardian / верифікація (CN-09). Єдиний ввід клієнта — 12-значний account ID.
- ✅ Кожна картка на Integrations відкриває сторінку **свого** акаунта, тому delete застосовується
  до правильного (CN-14).
- ❌ Прототип `#j3-step-nav` (3944-3950) — чотири вайрфреймові стадії замість трьох (10.1);
  саморобна текстова `← Back` на `j3-results` (16744) замість `.back-btn` (1.2).

**Exceptions.** Overlay-майстри використовують `gm-stepper` і кнопки `Back`/`Next` у `gm-foot`
замість лівої панелі та `.back-btn` — правила 1, 2, 5 діють, правила 3 і 4 замінюються
еквівалентами модального шасі (1.3, 1.4).

---

## 12. Badges

### 12.1 Канонічний бадж (Figma) — ЄДИНЕ ДЖЕРЕЛО

**Rule.** У застосунку існує **один** компонент баджа: **одна геометрія · 6 кольорів · 3 варіанти**.
Будь-який новий бадж — це `.bdg` плюс клас кольору; власних розмірів, радіусів і відтінків не
вигадувати. Джерело: Figma `eXz33Qu7v58JjOiatTptAE`, секція `Badge` (node `2988:70572`).

**Геометрія — однакова для всіх кольорів і обох тем**

| Властивість | Значення |
|---|---|
| padding | `2px 7px` |
| radius | `999px` (пігулка) |
| border | `1px solid` |
| шрифт | DM Sans **700**, `9px`, line-height `normal` |
| letter-spacing | `1px` |
| регістр | **UPPERCASE** |
| крапка | `5×5px`, radius `2.5px`, колір = `currentColor`, gap `4px` |

**Три варіанти**

| Клас | Figma | Що це |
|---|---|---|
| `.bdg` | `No dot` | базова пігулка |
| `.bdg.bdg--dot` | `Dot` | пігулка з крапкою |
| `.bdg.bdg--quiet` | `Dot+text` | **не пігулка**: крапка + текст, без фону, рамки й padding |

**Шість кольорів** — `.bdg--red · --orange · --blue · --green · --gray · --purple`.
Значення живуть **тільки** в токенах `--bdg-{колір}-txt / -bg / -bd` (`index.html:165` light,
`index.html:2642` dark). Схема однакова в обох темах: bg `0.05`, border `0.24`, окремий hex тексту.

| Колір | Light текст | Dark текст | Семантика |
|---|---|---|---|
| Red | `#CF454D` | `#E66A71` | збій, блокує |
| Orange | `#B86410` | `#D98A3A` | потребує уваги |
| Blue | `#2663EB` | `#5C9DFF` | інформація, нижчий пріоритет |
| Green | `#059669` | `#34D399` | успіх |
| Gray | `#475569` | `#CBD5E1` | нейтральне, неактивне |
| Purple | `#7054BD` | `#9B83DD` | класифікація (не статус) |

**Severity** мапиться так (рішення 13.08.2026): `critical` → Red · `high` і `medium` → **обидва
Orange** · `low` → Blue · `pass` → Green. High і medium ділять колір свідомо — вони й раніше мали
один текстовий колір `#B45309`, канон просто зробив це чесним.

**Rule (пігулка vs тихий маркер).** Пігулку отримує те, що керує наступною дією: **статус** або
**severity**. Усе інше — значення атрибута, тип зміни, допоміжний маркер — це варіант `Dot+text`.
У рядку має бути **рівно один** об'єкт із формою; якщо їх два і більше, ієрархія зникає.

**Reference.** Токени `badge/{color}/text · /background · /border` у Figma; у коді — блок
`BADGE — КАНОНІЧНИЙ КОМПОНЕНТ` (`index.html:1239`).

**Examples.**
- ✅ **Міграція виконана 13.08.2026.** 18 родин (`.type-pill`, `.badge`, `.drift-stat`, `.drift-type`,
  `.intg-status`, `.id-cap-badge`, `.cm-status`, `.cred-badge`, `.intg-card-tag`, `.intg-cap`,
  `.rc-chg`, `.sf-st`, `.cred-source-pro`, `.csv-badge`, `.rpt-badge`, `.usedby-tag`,
  `.node-kpi-badge`, `#pd-status-badge`) плюс ~30 інлайн-одноразовиків зведено сюди.
- ✅ Легасі-імена класів **лишені як селектори**, розмітку й JS-логіку класів не переписували —
  геометрія й палітра беруться з канону.
- ✅ Тема тепер повністю в токенах, тому **всі** `body.dark` оверрайди баджів стали мертвими
  й видалені. Новий бадж не потребує жодного рядка dark-CSS.
- ✅ `.drift-type` (тип зміни) і значення before→after у `v-node-detail` — варіант `Dot+text`,
  тому в рядку лишається одна пігулка: severity або статус реконсиляції.
- ❌ Було: `Enabled › Disabled` як залиті зелена/червона коробки поруч із баджем `CRITICAL` —
  три однорангові форми, колір закодовано тричі (позиція + severity + заливка).

**Exceptions.**
- **`.csv-badge`, `.rc-chg`, `.drift-type`** використовують `Dot+text`, а не пігулку — за правилом
  «один об'єкт із формою в рядку».
- **Не баджі й під канон не підпадають:** `.conn-status` (панель стану), `.step-badge` (шасі
  майстра), `.auto-banner` (банер), лічильники (`.pol-tab-count`, `.step-dot`, `.gm-step-num`),
  усі клікабельні чипи (`.pill`, `.recon-node-chip`, `.drift-meta-pill`, `.tagpick-chip`,
  `.ac-sev-pill`, `.hh-toggle`) та літеральні значення (`.code-tag`) — див. §13.
- **Іконок канон не має.** Severity-баджі, що раніше несли SVG, тепер мають крапку `5px`.
  Гліфи `✓ ✗ ★ ⚠` у тексті баджа не пишемо — форма й колір уже несуть значення.

**Відомі дефекти макета** (нормалізовані при імплементації, підняти з дизайнером):
1. **Blue Dark border** у файлі лишився світлим `rgba(38,99,235,0.24)`, хоча текст і фон перейшли
   на `#5C9DFF` — недотягнута змінна. Нормалізовано на власний hue.
2. **Green Light bg = 10%**, у решти 5%. Нормалізовано до 5%.
3. **Orange Dark 7%/26% і Green Dark 7%/28%**, решта dark — 5%/24%. Нормалізовано.
4. **Канон дає 6 кольорів на ~10 семантичних вимірів.** Мапінг severity вирішено (див. вище),
   але при появі нових статусів колір доведеться ділити далі.

---

## 13. Неклікабельні елементи

Чипоподібних прямокутників у продукті три різновиди, і плутанина між ними — головне джерело
питань «а це що за бадж?». Розрізняти треба **за поведінкою**, а не за виглядом:

| | клікабельний? | що це |
|---|---|---|
| `.bdg` (§12) | ні | **статус / класифікація**, яку присвоїла система |
| `.pill`, `.tagpick-chip`, `.recon-node-chip`, `.hh-toggle`, `.drift-meta-pill`, `.ac-sev-pill` | **так** | контрол — фільтр, перемикач, вибір |
| `.code-tag` (13.1) | ні | **літеральне значення**, яке користувач вводить або звіряє |

Ні `.bdg`, ні `.code-tag` не мають hover, focus, cursor:pointer і не реагують на клік.
Якщо елемент щось робить при кліку — він не належить до цього розділу, а до §6 Buttons / §3 Forms.

### 13.1 Літеральне значення — `.code-tag`

**Rule.** Значення, яке користувач **вводить або звіряє символ у символ** (назва колонки CSV,
ServiceNow CI id, шлях до папки секретів), подається як `.code-tag`: моноширинний, **регістр
збережено**, прямокутник `radius:5px` — свідомо НЕ пігулка, щоб не читатись як статус.

| | `.bdg` (§12) | `.code-tag` |
|---|---|---|
| шрифт | DM Sans 700, 9px, **UPPERCASE** | DM Mono 400, 11px, **регістр збережено** |
| форма | пігулка `999px`, `2px 7px` | `5px`, `2px 8px`, `--bg-subtle` + `--border` |
| приклад | `CRITICAL`, `UNRECONCILED`, `CONNECTED` | `node_group`, `CI0043221`, `\Personal Folders\Guardian` |

Канон баджів сюди **не застосовується свідомо**: 9px UPPERCASE перетворив би `node_group` на
`NODE_GROUP`, а це рядок, який має потрапити в CSV дослівно. Тут регістр — вміст, не стиль.

Модифікатор `.code-tag--req` = обовʼязкова колонка: червона рамка + червоний текст.
Червоний береться з `--bdg-red-txt`/`--bdg-red-bd`, **а не з `--status-danger-txt`** — той
визначений лише для світлої поверхні (§5.6), а чип живе в обох темах.

**Reference.** `index.html:1348`.

**Examples.** CSV-імпорт «Columns» (9 чипів) · ServiceNow CI id у `v-node-detail`. Інлайн-`<code>`
у тексті ділить із чипом шрифт через глобальне правило `code { font-family:'DM Mono' }`.

**Exceptions.** Моноширинний текст у **комірці таблиці** (`CI0043221` у CMDB-мапінгу, `web-prod-01`
у колонці нод) лишається простим текстом без фону й рамки: колонка вже є контейнером, чип додав би
другу коробку в кожну комірку.

**Відкрите питання.** Візуальна близькість `.code-tag` до `.bdg` (обидва — обведена коробка
з фоном) свідомо лишена: спробу зняти форму повністю (моно-текст із розділювачем `·`) розглянуто
й **відхилено**. Якщо блок «Columns» знову читатиметься як другий ряд баджів — повертатись сюди.

---
