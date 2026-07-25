# Agentless onboarding — interaction notes & open questions

Companion notes for the Guardian **Add single node (agentless)** flow in the
`essentials` prototype. Covers the happy path, validation, error states, and
what is backend-confirmed vs. concept-only, so engineering can scope the build
and the client can sign off the interaction design.

Prototype: https://burkotaras-dot.github.io/guardian-demo/essentials/

---

## 1. End-to-end flow

```
Add node → choose method (agentless) → select / create Connection Manager
         → select / create credential → enter node details
         → Test connection & continue → connectivity result
         → Add node → Node added → Run first scan / Schedule / View node
```

View sequence (prototype ids):
`s-method` → `s-agentless` → `s-connect-test` → `s-node-added` → `s07` (scan) → `results`.

The three-step top stepper reads **Add node · Connect · Scan · Results**.
`s-agentless`, `s-connect-test` and `s-node-added` all sit under step 2
("Connect"); the scan animation (`s07`) is step 3.

---

## 2. Add-node form (`s-agentless`) — validation

The primary button **Test connection & continue** stays **disabled** until the
form is complete. Validation (`alValidate`) re-runs on every relevant change:

- typing in **node name, hostname/IP, port, username, password, key**
  (`oninput`);
- selecting or changing the **Connection Manager**;
- switching credential mode (**Direct** ↔ **Credential Vault**);
- switching direct-auth type (**Password** ↔ **SSH key**);
- picking a credential from the Vault.

**Required to enable the button:** node name · hostname/IP · a Connection
Manager · a valid credential set.

- *Direct + Password:* username **and** password.
- *Direct + SSH key:* username **and** key.
- *Credential Vault:* a selected vault credential.

While incomplete, a hint under the button spells out exactly what is missing,
e.g. *"Add a hostname or IP and credentials to test the connection."*

**Duplicate node name.** If the entered name matches an already-monitored node
(case-insensitive), an inline error appears under the name field —
*"A node named 'X' is already monitored. Choose a unique name, or open the
existing node instead."* — and the submit button stays disabled.

**OS-aware defaults.** The selected OS drives the connection defaults:
Windows → WinRM, port **5985**, password-only (SSH-key tab hidden);
Linux/macOS → SSH, port **22**, password or key.

---

## 3. Connectivity test (`s-connect-test`) — states

The test runs three sequential checks, each with a live spinner → result icon:

1. **Reachability** — can the Connection Manager open a connection to the host.
2. **Authentication** — does the credential sign in.
3. **Read access** — can the account read the configuration Guardian needs.

On success all three turn green and a **Ready to scan** banner appears with a
single primary action **Add node →**.

### Error / edge states (all demoed via the on-screen scenario switcher)

| State | Fails at | Result | Actions |
|---|---|---|---|
| **Host unreachable** | Reachability | "Can't reach X" — host down, wrong address/port, or firewall | Retry test |
| **Connection timeout** | Reachability | "Connection timed out" after 30 s — firewall/route blocking | Retry test |
| **CM offline** | Reachability | "Connection Manager offline" — CM hasn't checked in | View Connection Manager · Retry test |
| **Bad credentials** | Authentication | "Authentication failed" — credential rejected | Edit credentials · Retry test |
| **Insufficient permissions** | Read access | "Insufficient permissions" (warning, not hard fail) — signed in but can't read config | Edit credentials · Retry test |

- **Retry test** re-runs all three checks from the top with the same inputs.
- **Edit credentials** returns to the form (`s-agentless`) with **all entered
  data preserved** — the user fixes the credential and re-tests.
- **View Connection Manager** deep-links to the CM list to bring the relay back
  online.
- **← Back to edit** (always present in the footer) returns to the form with
  data preserved.

### Data preservation

Going back from the test — via Back, Edit credentials, or Retry — never clears
the form. Node name, host, port, OS, CM selection and credential all persist.

---

## 4. Success (`s-node-added`)

Confirmation screen summarising the created node (name, address + protocol, OS,
Connection Manager, credential) and three next-step CTAs:

- **Run first scan now** → starts the first scan immediately (step 3).
- **Schedule scan** → schedule it for later / recurring.
- **View node details** → node configuration, history, assigned policies.
- **+ Add another node** → restart the flow from method selection.

The new node name is added to the "already monitored" set, so immediately
adding a second node with the same name trips the duplicate check.

---

## 5. Backend-confirmed vs. concept-only

**Concept / prototype-only (needs engineering confirmation):**

- The three-check sequence and its **timing** (spinner → result) are simulated
  on the client; real checks, ordering and durations are not yet defined.
- The 30 s timeout figure and the "42 ms" reachability latency are placeholder
  copy.
- The scenario switcher exists **only** to demo states — it is not a product
  control.
- Error copy and remediation guidance are drafts for review, not final strings.
- "Read access" is modelled as: local admin (Windows) / sudo read (Linux). The
  exact permission set Guardian requires is an open question (see below).

**Interaction decisions we are proposing (for sign-off):**

- Disabled primary button + explicit "what's missing" hint, rather than
  letting the user submit and fail.
- Duplicate-name caught **before** the test, not after.
- Errors keep the user on the test screen with targeted actions
  (Retry / Edit credentials / View CM) rather than dumping them back to a blank
  form.
- Single **Add node** commit step **after** a green test, so a node is only
  created once connectivity is proven.

---

## 6. Open technical questions

1. **Read-access permission set** — what exactly must the agentless account be
   able to read per OS (Windows / Linux / macOS)? This defines the
   "Insufficient permissions" check and the guidance we show.
2. **Test timing & timeout** — are the three checks run in this order backend-
   side? What real timeout applies, and is it per-check or overall?
3. **Node created before or after test?** — prototype creates the node only
   after a successful test. Confirm the backend supports "test, then create",
   or whether a node record must exist first (draft/unverified state).
4. **Partial success** — if reachability + auth pass but read access is denied,
   should the node be creatable in a degraded/"connected, not scan-ready"
   state, or blocked entirely? Prototype currently blocks (warning + fix path).
5. **Duplicate detection scope** — dedupe on node name only, or also on
   host/IP? What's authoritative?
6. **Credential re-validation** — when a credential is rejected mid-onboarding,
   do we revalidate just this node, or flag the credential everywhere it's
   used?
7. **CM offline during Add Node** — can the user proceed by switching to another
   CM inline, or must the original CM be restored first?
8. **Scheduling** — is "Schedule scan" from the success screen the same
   scheduler as elsewhere, and what cadences are supported at Essentials tier?

---

## 7. For the client review

Ready to show: **CM setup · CM management (list + detail) · Credential Vault ·
Add single node agentlessly · happy path · main error states · open questions.**

Decisions to confirm with the client:
- Is this onboarding flow ready to hand off to engineering, or does it want
  one more UX pass?
- Which of the open questions above are backend-limited (must be answered
  before build) vs. product choices we can settle in design?
