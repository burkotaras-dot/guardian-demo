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

**Dark mode needs no overrides.** Colour comes from tokens (`--bdg-{colour}-txt|-bg|-bd`,
light `index.html:165`, dark `index.html:2642`), so `body.dark` badge rules are obsolete and
were deleted. Do not reintroduce them — and do not set badge text to `#fff` in dark: the canon
uses the coloured text token.

**Layout rule:** a row must contain **exactly one** object with a shape (the pill). A value never
gets a border/background — otherwise data-vs-metadata hierarchy collapses and colour is encoded
several times over.

Legacy family class names (`.badge-*`, `.drift-stat`, `.intg-status`, `.type-pill`, …) are kept as
selectors inside the canonical block, so markup and JS class logic stay untouched.

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
