# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **single-file HTML prototype** for Guardian — a security compliance/drift-detection product. All CSS, HTML, and JavaScript live in `index.html` (~27 000 lines). There are no build steps, no package managers, and no dependencies beyond Google Fonts.

The canonical icon set lives in the repository at **`Assets/icons/`** (`Property 1={Name}.svg` —
Right/Left/Up/Down/Plus/Loading, plus `new/` and `then/` action icons). Use these for any arrow,
plus or spinner; inline them with `stroke="currentColor"` so they follow the theme.

---

# ⭐ ГОЛОВНІ ПРАВИЛА — читати перед будь-якою правкою

Це продуктовий прототип із **готовою дизайн-системою**. Майже все вже має канонічний компонент.
**Нічого не вигадувати з нуля** — спершу знайти наявний патерн у файлі й переюзати його.
Джерело істини: `GUARDIAN-DESIGN-SYSTEM.md` + Figma-файл `eXz33Qu7v58JjOiatTptAE`.

### 0. Процес
1. Відповідати **українською**.
2. **Не комітити й не деплоїти без прямого прохання** користувача.
3. Після кожної правки — **перевірити в прев'ю** у світлій і темній темі та показати
   **виміряні значення** (`getComputedStyle` / `getBoundingClientRect`), а не «має працювати».
   Ніколи не просити користувача перевірити самому.
4. Дали Figma-URL → викликати `get_design_context` і брати **літеральні значення** з макета.
   Токен підставляти, тільки якщо він точно дорівнює значенню макета **в обох темах**.

### 1. Кнопок рівно 6 типів (DS §6.5, Figma `2988:70531`)
`Primary` · `Secondary` · `Text button` · `Third` · `Back` · `Only icon`.
Будь-яка нова кнопка — один із цих шести. Нових не вигадувати.
`Only icon` — єдиний нейтральний тип: квадрат 32×32, radius **8** (виняток із загального r10), гліф 16×16.

**Кнопка в «рядку/картці з діями» (полоски з кнопками) — завжди зелена контурна (`.action-cta`):**
- light: bg `rgba(0,175,115,0.05)`, border `#9DE0C9`, hover bg `rgba(0,175,115,0.15)` + border `rgba(0,175,115,0.35)`
- dark: bg `rgba(0,175,115,0.15)`, border `rgba(0,175,115,0.35)`, текст білий, стрілка `#00AF73`

Стрілки/плюс/спінер — **тільки з `Assets/icons/`**, інлайном зі `stroke="currentColor"`, 12×12.

### 2. Форми і контролі
- **Інпут/textarea/select на картці:** light bg `#F1F5F9` (`--bg-subtle`), focus → білий;
  **dark bg `rgba(255,255,255,0.05)`**, border `rgba(255,255,255,0.2)`, focus bg `rgba(255,255,255,0.07)`
  + зелена рамка. ⚠️ У dark **НЕ** `--bg-subtle` (#00262A) — темніше за картку, виглядає провалено.
- **Чекбокси і радіо стилізовані глобально** — новим нічого не задавати.
  ⚠️ Ніколи не писати світле правило без `body:not(.dark)` — протече в темну тему білою плямою.
- **Severity-дропдауни:** нейтральне поле + **кольорова крапка** біля тексту. Ніколи не заливати
  все поле кольором severity.
- **Кнопка Back / `.btn-outline` у light** завжди має білий фон `var(--bg-surface)`, не `transparent`.

### 3. Таблиці
Хедер таблиці в **dark = `#063135`**, ніколи не чорний. Це глобальне правило з `!important`
(`index.html:737`) — на нових `th` просто **не задавати `background`**, канон застосується сам.

### 4. Стани й значення
- Статус-бокси (current↔expected, before↔after) — **нейтральна поверхня + кольорова крапка**.
  Ніколи не заливати червоним/зеленим/жовтим.
- **У рядку рівно ОДИН обʼєкт із формою** (пігулка). Значення форми не отримує → `.bdg--quiet`.
- Порожні списки — канонічний empty-state: коло-іконка 44×44 + заголовок 14/600 + підпис 12px
  + дія (скинути фільтр / CTA).

### 5. Модалки
**Усі Add/Create/Connect — це overlay-модалка через `openModal()`**, ніколи не нова повносторінкова
верстка. Оболонка одна на застосунок (600px, r12, header · stepper · body · footer) — змінюється
**лише вміст слота**. Деталі — розділ «Modal footer actions» нижче + DS §14.

### 6. Темна тема
Ніколи не ставити фон, що залежить від теми, **інлайном** — `_syncInlineTheme` перезаписує інлайнові
фони. Тільки CSS-клас + `body.dark` override.

---

## Preview & Deploy

**Local preview:** Claude Code has a built-in preview server. Use `preview_list` to find the active port (typically 8080, name `guardian-demo`).

### ⛔ CRITICAL: two different pages live in one repository

GitHub Pages serves the **`public`** branch, and `public` mirrors `main`'s tree. That tree contains
**two independent pages**:

| URL | file in the served tree | what it is |
|---|---|---|
| `https://burkotaras-dot.github.io/guardian-demo/` | **root `index.html`** | the **client-facing** page |
| `https://burkotaras-dot.github.io/guardian-demo/essentials/` | **`essentials/index.html`** | **our prototype** |

Work from the `essentials` branch deploys to **`essentials/index.html` only**.
**Never write to the root `index.html`** — it is a different product surface and the client reads it.

**How to tell a clean root from a contaminated one.** The client page has exactly **6 sidebar items**:
`dashboard · all-devices · results · changes · policies · integrations`. The prototype has 10.
If `sb-reports`, `sb-scan-schedule`, `sb-cm-list` or `sb-credentials` appear in the root — or any of
`nd-iss-`, `bdg--quiet`, `gm-add-node-wizard`, `gm-agentless-testing`, `btn-primary--s` — the prototype
has leaked into it.

```
curl -s https://burkotaras-dot.github.io/guardian-demo/ | grep -oE 'id="sb-[a-z-]+"'
```

Clean-root reference blob = **`4e5a9cf3`** (14 211 lines). Verify the blob, not a commit —
`git rev-parse main:index.html` must print `4e5a9cf3…`. It was last restored by `15ed43d`, which
took its content from `1985c3b`; both hashes float around in notes, the blob is the only stable check.

**Deploy an `essentials` change — always through a worktree**, because the `essentials` working tree
usually holds uncommitted WIP that `git checkout main` would drag onto prod:
```
git worktree add /tmp/gd-deploy main
cd /tmp/gd-deploy
git show essentials:index.html > essentials/index.html          # ONLY paths under essentials/
git diff --stat -- index.html                                    # MUST be empty: root untouched
git commit -am "deploy(essentials): ..." && git push origin main
git checkout public && git reset --hard main && git push --force-with-lease origin public
cd - && git worktree remove /tmp/gd-deploy --force
```

- **There is no CI.** `.github/workflows/deploy.yml` exists on `essentials` only, is not on `main`,
  and never runs. GitHub Pages serves the **`public` branch directly** — the `push origin public`
  line above *is* the deploy. Do not wait for an Actions run and do not go looking for a workflow.
- `main` is a staging mirror, not a deploy source; it fast-forwards normally. `public` diverged from
  history, so it needs `--force-with-lease`.
- **Verify against the live URL, not the branch** — `curl -s <url> | wc -l` plus a marker `grep`.
  A correct git blob does not prove the page is serving it.
- **Before reverting the root, inspect the target's contents.** "The state before my mistake" is not
  the same as "a correct state" — `d529b58` predates the leak commits yet already carried 4 prototype
  sections.

The older `dev` branch and its cherry-pick chain are **historical**; `dev` is not a deploy source and
is not used by this workflow.

**Quick prototype navigation (browser console):**
```js
// Skip login
Array.from(document.querySelectorAll('a')).find(l => l.textContent.includes('Skip to prototype')).click()
// Enable dark mode
document.body.classList.add('dark'); _syncInlineTheme && _syncInlineTheme();
// Navigate to any view
showView('v-policy-edit')  // or 'policies', 'results', 'dashboard', etc.
```

## Architecture

### `index.html` structure (in order)

1. **`<style>` block** — all CSS, including a CSS custom-property design token system (`--p-*` primitives, semantic tokens), component styles, and `body.dark {}` dark mode overrides.
2. **`<body>` block** — 55 `div.view[id="v-*"]` elements, all hidden by default; `showView()` reveals one at a time.
3. **`<script>` block** — all JavaScript: view routing, filter logic, bulk operations, dark mode sync.

### View routing

`showView(name)` is the single entry point for navigation. It calls `showOnly(name)` (which hides all views then sets `#v-{name}` to `display:flex`), updates sidebar active state, updates the top-nav label via `LABELS[name]`, and handles special cases (J3/J4 step navs, onboarding nav).

View groups:
- **Onboarding:** `v-s01`, `v-s02`, `v-s05`, `v-s07`
- **Core app:** `dashboard`, `results`, `remediation`, `changes`, `all-devices`, `policies`
- **Detail/edit:** `node-detail`, `compare-nodes`, `policy-detail`, `policy-edit`, `policy-create`, `policy-assign`, `add-check`, `edit-check`, `change-detail`, `change-approve`
- **Jira J3 flow:** `j3-connect` → `j3-connecting` → `j3-connected` → `j3-syncing` → `j3-inventory` → `j3-scan-config` → `j3-scanning` → `j3-scan-complete` → `j3-results`
- **Jira J4 flow:** `j4-connect` → `j4-connecting` → `j4-connected` → `j4-match` → `j4-active` → `j4-summary`
- **CMDB flow:** `cmdb-connect` → `cmdb-connecting` → `cmdb-browse` → `cmdb-importing` → `cmdb-complete`

### Nodes view tabs

`v-all-devices` contains three sub-tabs switched by `setNodesTab(tab)`:
- `managed` — Managed Nodes table
- `detected` — Detected Drift table
- `peer` — Peer Comparison / Consensus view

### Dark mode

Two-layer system:
1. **CSS:** `body.dark .selector { ... !important }` overrides. Uses `!important` throughout due to inline-style specificity conflicts.
2. **JS:** `_syncInlineTheme(isDark)` scans every `[style]` element and regex-replaces known light background hex values with dark equivalents. Theme persisted to `localStorage` key `guardian-theme`.

**Critical gotcha:** Writing any `el.style.X = value` causes the browser to re-serialize the entire inline style attribute (hex → rgb). Always guard with `if (el.style.X !== newValue)` before writing. `_syncInlineTheme` already matches both hex and rgb forms for known colors.

### Badges — canonical component

Single component from Figma (`eXz33Qu7v58JjOiatTptAE`, section `2988:70572`), implemented at
`index.html:1239`. **One geometry · 6 colours · 3 variants.** Any new badge is a `.bdg` —
never a hand-rolled inline chip. Full spec: `GUARDIAN-DESIGN-SYSTEM.md` §12.

Variants:
- `.bdg` — pill (no dot)
- `.bdg.bdg--dot` — pill + 5px dot in the text colour
- `.bdg.bdg--quiet` — **not a pill**: dot + text only, no bg/border/padding. Used for a *value*
  or a quiet type marker (`.drift-type`, before→after values).

Colours: `.bdg--orange | --purple | --red | --green | --gray | --blue`.

Severity mapping: critical → Red · high + medium → Orange · low → Blue · pass → Green.

### Add Node modal flow

The Add Node detected-OS flow follows Figma component frame `3165:10694` (modal instances
`3154:27407` Light / `3154:27537` Dark): `openAddNodeWizard()` uses an
600×473px desktop shell (74px header, 54px stepper, 282px body, 63px footer), with a
550×60px detected block and two 269×95px OS cards separated by 14px. The primary action lives
**inside** the detected block, right-aligned — the strip states what we found and the button acts
on that same statement, so they are one object rather than two stacked ones.
Keep exactly 18px between the bottom of the OS cards and the footer divider.
Light OS cards follow Figma `3154:26383`: default `#F8FAFC` / `#E2E8F0`, hover
`#EDF9F4` / `rgba(16,185,129,.60)` with a `#10B981` icon and no hover shadow.
Dark OS cards follow Figma `3154:27718`: default surface `#123336` with no visible border;
hover surface `rgba(16,185,129,.10)` with `rgba(16,185,129,.60)` border; icons remain
`#6EE7B7`, titles white, subtitles `#94A3B8`, and no shadow.
Add Node source cards follow Figma `3145:24091`: exact desktop height 94px, 14px vertical and
20px horizontal padding, a 64px content row, 1px border, 14px radius, 44px icon, and 16px column gap.
In the dark Add Node source-choice modal (Figma `3145:24948`), only the two lower default cards
(ServiceNow and CSV) use `#123336` with a transparent border; keep the semantic hover borders.
After Add Manually, step 1 is always detected OS / OS selection; step 2 is the method choice.
Never swap them. Method cards follow Figma `3164:10455`: 268px wide, 14px apart, 20px padded,
14px radius, with 44/22px icon geometry. Cards are 204px high; keep
exactly 18px from their bottom edge to the footer divider. The resulting method shell is 600×433px.
Dark method cards use `#123336` with no visible default border.
Its Light source is Figma `3164:10368`: default `#F8FAFC` / `#E2E8F0`, hover
`#EDF9F4` / solid `#10B981`, no shadow; descriptions are `#475569` at 11/18.
The in-strip content action is DS §6.5 **Primary, size s** — `.btn-primary.btn-primary--s.btn-primary--arrow`
(32px, `padding:0 20px`, DM Sans 600/12, radius 10, gradient + `0 4px 7px rgba(16,185,129,.3)`).
Do not substitute the modal-footer `.gm-btn-primary` component, and do not restore the old
full-width `.btn-primary--large` action.
Its canonical four-step `gm-stepper` is Add node → Connect → Scan → Results. Do not route these selection screens back to the legacy full-page `v-s02`, `v-s-method`, `v-s-agentless` or `v-s-connect-test` views. Choosing `Agentless scan` opens the canonical 600px Connect modal (`#agentless-setup-overlay`, Figma `3171:12282`) with Target, Connection Manager Group and Credentials sections; continuing replaces it with the `Testing connection` state in the same overlay (Figma `3173:13780`, 600 × 609px). Preserve the same stepper and modal footer throughout.

**Dark mode needs no overrides.** Colour comes from tokens (`--bdg-{colour}-txt|-bg|-bd`,
light `index.html:165`, dark `index.html:2642`), so `body.dark` badge rules are obsolete and
were deleted. Do not reintroduce them — and do not set badge text to `#fff` in dark: the canon
uses the coloured text token.

**Layout rule:** a row must contain **exactly one** object with a shape (the pill). A value never
gets a border/background — otherwise data-vs-metadata hierarchy collapses and colour is encoded
several times over.

Legacy family class names (`.badge-*`, `.drift-stat`, `.intg-status`, `.type-pill`, …) are kept as
selectors inside the canonical block, so markup and JS class logic stay untouched.

### Modal footer actions — canonical component

Use `openModal()` and `.gm-foot`; source of truth is Figma component set `3023:5088` and
`GUARDIAN-DESIGN-SYSTEM.md` §6.1. Footer geometry is `padding:10px 24px 12px`, buttons are 32px high,
10px radius, DM Sans 600/12. Back belongs on the left via `backButton` and renders as the canonical
Third pattern `[32px arrow square] Back`; never hand-code `← Back` in `footerLeftHtml`. Primary and
optional supportive secondary belong in `.gm-foot-right`, with Primary last. `Cancel`/`Close` remain
neutral; warning/destructive buttons keep their semantic colour.

Credential selection inside modal wizards must use the custom dropdown pattern from Figma `3171:13726`
(status dot + two-line credential identity + canonical status badge). Do not use a native `<select>`
popup for these fields; it cannot match the product styling across browsers or themes.

**The only other chip family is `.code-tag`** (`index.html:1348`) — a *literal value* the user types
or verifies (CSV column name, ServiceNow CI id). DM Mono 11px, case preserved, radius 5px. The badge
canon deliberately does not apply: 9px UPPERCASE would turn `node_group` into `NODE_GROUP`, and the
case is significant. `.code-tag--req` = required field. Mono text inside a **table cell** stays plain
text — the column is already the container. `.code-tag` is **not** documented under Badges: it lives in
DS **§13 Неклікабельні елементи / §13.1**, because it is not a badge.

### Key JS function namespaces

| Prefix | Domain |
|---|---|
| `pe*` | Policy Edit (peInitB, peOpenSevPick, pePickSev, peToggleSec, etc.) |
| `pc*` | Policy Create (pcGoStep, pcTypeChange, pcFinishCreate, etc.) |
| `setNodesTab` | Nodes tab switching |
| `applySelectionConsensus` / `clearSelectionConsensus` | Bulk consensus comparison |
| `_syncInlineTheme` | Dark mode inline-style sync |
| `showView` / `showOnly` | View routing |

### Policy Edit custom severity dropdown

Native `<select>` elements are replaced with `.pe-sev-btn` divs. Popup: `#pe-sev-pick`. Dark mode styles applied via `body.dark .pe-sev-btn[data-val="..."]` CSS attribute selectors.

### Consensus Comparison

Triggered from Managed Nodes when 3+ nodes are selected (toolbar button `#node-action-consensus`). Calls `applySelectionConsensus(selectedNames)` after navigating to `setNodesTab('peer')`. Outlier rows are identified by structure: exactly 3 div children, badges in the 3rd cell — do NOT filter by `parentElement === card`.


## Перевірка змін

**Тестів, лінтера і typecheck у проєкті НЕМАЄ** — немає `package.json`, немає збірки.
Не пропонуй `npm run typecheck / test / lint`: ці команди не існують і ніколи не існували.

Замість них єдина перевірка — **виміряти результат у прев'ю**:
1. `preview_list` → знайти сервер `guardian-demo`, `preview_eval` → `window.location.reload()`.
2. Застосунок стартує на екрані логіну — спершу клікнути посилання `Skip to prototype`,
   інакше `showView()` і модалки не працюють.
3. Дійти до зміненого екрана **тим самим шляхом, що й користувач** (клікаючи), а не лише
   викликом функції.
4. Зняти реальні значення через `getComputedStyle` / `getBoundingClientRect` — у **світлій і
   темній** темі (`document.body.classList.add('dark'); _syncInlineTheme(true)`).
5. `preview_console_logs` — має бути порожньо.
6. Показати користувачу **виміряні числа й скріншоти**, а не «має працювати».

⚠️ `getComputedStyle` у тому ж `preview_eval`, де перемикається тема, віддає **застарілі**
значення — читати наступним викликом.

## Git

Commit messages — Conventional Commits: feat/fix/chore/refactor/test/docs.
Без крапки в кінці заголовку. Тіло коміту — якщо зміна не очевидна.

**Ніколи не комітити й не деплоїти без прямого прохання користувача.**

## Працюємо вдвох — один файл на двох

У проєкті двоє людей, обидва через Claude Code, обидва мають право запису.
Весь прототип — **один файл на 27 000 рядків**, тому git не зможе автоматично злити паралельні
правки.

- **Перед стартом роботи завжди `git pull`.** Якщо `git status` показує, що гілка позаду
  `origin/essentials` — спершу підтягнути, тільки потім правити.
- **Після коміту одразу `git push origin essentials`.** Незапушена робота невидима для другого
  і майже гарантує конфлікт.
- Робоча гілка одна — **`essentials`**. Нових гілок не створювати без прохання.
  ⚠️ На свіжому клоні `git checkout essentials` **падає** з
  `fatal: 'essentials' could be both a local file and a tracking branch` — бо в дереві `main` лежить
  тека `essentials/`. Використовувати **`git switch essentials`**.
- Якщо все ж виник конфлікт у `index.html` — **не вирішувати його самостійно й не робити
  `--force`**. Показати користувачу, які саме блоки розійшлись, і спитати, чию версію лишити.
