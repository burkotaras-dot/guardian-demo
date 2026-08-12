# Guardian UX Consistency — Week 1 Review

For: Stephen Earl, Connor Conway · From: Bambuk · 14 Aug 2026

Week 1 scope was *review feedback → consistency themes → prioritised behavioural UX backlog*.
This is the short version; the full working document is `CONSISTENCY-BACKLOG.md`.

---

## What we found

We audited the current Guardian across eight categories (navigation, action placement, confirmation,
edit/save, empty states, terminology, status communication, error handling) and combined the result
with Connor's GWB-6638 report. **38 items** in one list.

The single most useful finding is not any individual defect — it is where the defects are:

> Where a rule was already written down, we found **one** violation.
> Where the standard was empty, we found **22 of 24**.

Connor's document confirms this independently from the other side. Where the rule existed (Back
button placement), the delivered implementation matched it without asking. Where it did not
(theme and button treatment), the team had to raise a question. Two independent sources, one cause.

This is why we prioritised the items that fill empty sections of the standard, rather than the most
visible defects.

---

## What we prioritised

| Bucket | Items | Meaning |
|---|---|---|
| Ship this package | 17 | goes into code |
| Standard only | 18 | already correct in code — costs documentation, not implementation |
| Future backlog | 3 | next package |

P1 — 6 · P2 — 19 · P3 — 13.

Almost half the list is *Standard only*, and that is a positive result: 14 of Connor's items are
already implemented correctly. They need writing down, not rebuilding.

The Work Package named a risk: *"consistency work becomes too broad — many observations, little
shipped improvement."* Our control is the ratio — of 38 items, **17** go to code, and they close
**three of five** systemic problems.

---

## What we shipped

Two P1 blocks, chosen deliberately as working references for the new rules — a standard with no
working example in the product does not survive handoff.

**One toast component.** Two parallel systems merged into one, with variants `success · error ·
warning · info`. Colour, icon and duration are properties of the variant, not of the message text.

The important part: five validation errors were being shown in a **green success toast with a
checkmark** — an error message that looked like a confirmation. This was the clearest example of
"Guardian behaves differently for similar tasks." The inversion is gone.

**No native browser dialogs.** `Discard unsaved changes?` used `window.confirm` — the only system
dialog in a product with a fully custom modal system. Replaced. No `window.confirm` remains.

**Honest status:** those five validations now have the right colour but still the wrong carrier —
the rule requires an inline message next to the field. Flagged as partially fixed in the standard,
not reported as closed.

---

## Guardian Interaction Standards

`GUARDIAN-DESIGN-SYSTEM.md` now holds **32 rules, 19 of them new** — the entire behavioural half:
loading and long-running operations, empty states, validation, action result, error, success,
warning, action placement and order, terminal action labels, one outcome — one control, step naming,
exit vs skip, message grammar, confirmation, edit/save, progressive disclosure, wizard chrome.

Each rule has four parts: **Rule · Reference · Examples · Exceptions**. We added `Examples`
deliberately — a rule without a worked example reads as a declaration and does not survive being
handed to someone else.

---

## Your five decisions — agreed

All five questions in §5 of GWB-6638: **yes** to each.

- No sub-stages inside the stage indicator.
- `Next` always advances the indicator.
- Side panel and stage number persist across all stages.
- `Back` is always top-left, in one style.
- This becomes the format for every future wizard.

None of them conflicts with the existing standard; four of the five directly confirm it. They are
now written up as the *Wizard chrome* pattern, with the delivered GWB-6638 flow as the reference.

---

## What we need from you

**Stephen — one item is blocking.**

*Theme and button treatment (CN-01): is the canon the application theme, or the wireframe colours?*
Development delivered against the application theme. Until this is settled, every new screen guesses.
Our recommendation: **the application theme is canon**; wireframe colours are a mock artefact. If the
decision goes the other way, that is a separate product-wide theming ticket, not part of this package.

Two lighter ones: whether badge consolidation (54+ usages, dark-mode regression risk) belongs to the
next package — we recommend yes; and whether finishing the implementation slice takes priority over
starting the Week 3 AI and roles work.

**Connor — two scope questions.**

Does "one outcome — one control" apply to all wizards or only the AWS flow? The prototype has two
more instances of the same pattern. And is the three-stage naming final, or will Run Scan / Results
return later — this decides whether the stepper needs to be extensible.

**Design — two defects in the canonical button set.**

There is no neutral button type: `Cancel` and `Skip` next to `Primary` would put two green controls
side by side. And `Text button` has no dark variant — `#059669` on the dark canvas is low contrast,
and the same problem has already reproduced in the tab component.

---

## Next

Finish the remaining implementation blocks (each has ready guidance with exact locations and
acceptance criteria), continue the canonical button migration, then Week 3 as planned:
Enterprise AI surfacing and roles & permissions.

Nothing from the Week 3 scope — Einstein, Felix, Jupiter, AI surfacing, roles & permissions, new
screens or broad redesigns — was started this week, as agreed.
