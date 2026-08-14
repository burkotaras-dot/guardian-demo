# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **single-file HTML prototype** for Guardian — a security compliance/drift-detection product. All CSS, HTML, and JavaScript live in `index.html` (~12 000 lines). There are no build steps, no package managers, and no dependencies beyond Google Fonts.

## Preview & Deploy

**Local preview:** Claude Code has a built-in preview server. Use `preview_list` to find the active port (typically 8080, name `guardian-demo`).

### ⛔ CRITICAL: `dev` is NOT a deploy source — it holds work the client must NOT see

`dev` contains **Detected Drift / Reconciliation** work that is explicitly **NOT approved for production**. Only **dashboard** work is approved for prod. `dev` and `main` therefore **intentionally diverge**, and `main` is **not** a fast-forward of `dev`.

- ❌ **NEVER** run `git merge dev` / `git merge dev --ff-only` into `main` or `public`. It drags every un-deployed `dev` commit (all the Detected Drift work) onto prod, where the client will see it. (This bug happened 2026-06-26 and required a force-push recovery.)
- ✅ Deploy a dashboard change by **cherry-picking only that commit** onto `main`.

**Deploy chain (cherry-pick, per-commit):**
```
# on dev: commit the dashboard change as usual, then deploy ONLY that commit:
git checkout main   && git cherry-pick -x <dashboard-sha>
git checkout public && git reset --hard main          # public mirrors main's tree
git push origin main && git push --force-with-lease origin public
git checkout dev
```

- `main` push triggers the GitHub Actions (`deploy.yml`) Pages deploy; **`public` is the served branch**.
- `public` diverged from history, so it needs `--force-with-lease`. `main` is normally a clean fast-forward of itself (no force) once it only carries cherry-picked dashboard commits.
- Last drift-free prod baseline = commit `d08d0e2`; its Detected Drift view is the "old" version the client is allowed to see.

GitHub Pages serves the `public` branch at https://burkotaras-dot.github.io/guardian-demo/

GitHub Actions (`deploy.yml`) auto-deploys to Pages on every push to `main`.

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
Its canonical four-step `gm-stepper` is Add node → Connect → Scan → Results. Do not route these first selection screens back to
the legacy full-page `v-s02` / `v-s-method` views. The complex agentless setup remains a
documented full-page exception after the method has been chosen.

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


## Команди перевірки (запускати після змін)
- `npm run typecheck` — TypeScript перевірка
- `npm run test` — всі тести
- `npm run lint` — ESLint
## Правило
Перед будь-яким комітом всі три команди мають пройти успішно.
Якщо є помилки — виправ їх перш ніж комітити.
Після цього CC сам запускає перевірки — без твого нагадування.
Коміти
Базовий сценарій
> Закомітуй зміни. Використовуй Conventional Commits.
CC робить: git status → git diff → формує повідомлення → git add → git commit.
Результат: feat(auth): add JWT refresh token rotation — не "fix stuff".
Атомарні коміти
> Зроби два окремих коміти:
  1. Зміни в хуку useProductFilters
  2. Зміни в компоненті ProductList
  Кожен коміт має проходити typecheck самостійно.
CC стейджить по частинах і перевіряє після кожного. Кожен коміт компілюється — не все разом в кінці.
Швидкий коміт без сесії
git diff --staged | claude -p "Write a Conventional Commits message for these changes"
CLAUDE.md — конвенція
## Git
Commit messages — Conventional Commits: feat/fix/chore/refactor/test/docs.
Без крапки в кінці заголовку. Тіло коміту — якщо зміна не очевидна.
