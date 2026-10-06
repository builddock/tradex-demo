# Tradex — instructions for Claude sessions

**Read this first.** Since 2026-09-28 the architecture is a **configurable SaaS e-commerce platform**
(`docs/SAAS_ARCHITECTURE_CHANGE.md`, decision `D-227`), not a single-business system:

- **One store codebase**, installed once per customer: every customer gets a dedicated installation in an
  environment the customer owns, installed and operated by the owner, sharing nothing with other customers except
  code and releases (`docs/DEDICATED_DEPLOYMENT_2026-10-05.md`, `D-287`, `D-288`). A store's e-commerce category, features, terminology, look and
  behaviour come from **configuration**, compiled into a fast runtime artefact. Never fork the codebase, never
  create a second application for a category.
- **A separate Configurable Root Admin platform** (`root-admin/`, its own portal, codebase, database and
  identity realm) creates, configures and deploys stores. It holds no customer business records and is never
  needed for a store's normal use (`D-287`).
- **Store users must never see the SaaS layer** — no other store, no category list, no template list, no
  configuration internals, no platform vocabulary.
- **Tradex is store #1**: vertical pack `VP-electronics`, template `TPL-forge`. Everything in `docs/meeting.txt`,
  the blueprint and the proposals is the specification of *that store*, and still fully authoritative for it.

- **Every feature is a switch.** ~230 features are grantable or withheld per store — including each
  communication channel (WhatsApp, email, SMS, web chat, push) and each automation. The store's own admin manages
  only what the root admin **delegated** to them.

Architecture: `plan/19-saas-platform.md` (tenancy, configuration, capabilities, channels, packs, templates),
`plan/20-root-admin.md` (the control plane) and **`plan/21-feature-map.md`** (which capability governs which
screen, tab, section, endpoint and entity). Read `19` §1–§2 before touching anything, and `21` before building
any screen.

## Repository map
| Path | What | Rule |
|---|---|---|
| `docs/` | Source requirements: `SAAS_ARCHITECTURE_CHANGE.md` (SAAS — authoritative for the architecture), `meeting.txt`, `ERP_Ecommerce_Implementation_Blueprint.md` (BP), two proposal PDFs | Read-only sources of truth |
| repo root `*.html`, `assets/` | Clickable store mockup v0.1 (32 screens) — the UI reference, published by GitHub Pages | Do not modify unless the user asks |
| `root-admin-mockup/` | Configurable Root Admin mockup (12 screens, own assets) | **Internal.** Never link it from any client-facing page (`D-249`) |
| `plan/` | Implementation plan, **`tasks/`** (the task tracker: one file per task, `D-296`), `TASKS.md` (generated index of every task), `STATE.md` (progress), `DECISIONS.md` (open questions) | Update tracking files every session |
| `frontend/`, `backend/`, `root-admin/`, `infra/`, `tests/` | Implementation code (created during Phase 1) | Layout: `plan/00-conventions.md` §11 |

## When the user says "Continue implementation"
(also "continue", "next task", "resume" — in any session.) Follow **`plan/14-continuation-protocol.md`** exactly;
the step-by-step form is in **`plan/tasks/README.md`**. In short:
1. Inspect the repo (`git status`, implementation folders), read `plan/STATE.md`, run `python3 plan/tools/status.py`
   (then `--check` and `--warnings`), and read `plan/19-saas-platform.md` §1–§2.
2. Reconcile the task files in `plan/tasks/` with the actual code and tests (resume `IN_PROGRESS` work first; never
   redo `COMPLETED` work).
3. Pick the next eligible task (all dependencies `COMPLETED`, no `OPEN` decision). If it needs a decision, **ask the
   user** with the documented options — never assume. `D-227`–`D-253` are already DECIDED; do not re-open them (on 2026-10-05 `D-287` amended several of them through
   change control — the "Amended by D-287" notes are authoritative).
4. Mark it `IN_PROGRESS` in its own file (`plan/tasks/<stage>/<task-id>.md`), implement only that task, run its tests.
5. Always — without being asked — update the task file (status + evidence), run `python3 plan/tools/tasks_sync.py`
   (refreshes dependents and the `plan/TASKS.md` index), update `STATE.md` (snapshot, logs), confirm
   `python3 plan/tools/status.py --check` shows 0 issues, then stop at a logical point and report.

## When the user asks "what next" or "status"
Answer from `plan/STATE.md` §0.1 (*Where we are and what's next*) checked against `python3 plan/tools/status.py`:
where the project is, the next steps in order, the next task and what it waits for, the decisions waiting for the
user, and what to say to continue. Change nothing (`plan/14-continuation-protocol.md` → *Other user requests*).

## Non-negotiable rules
- **Source fidelity:** build only what `docs/`, a mockup, or a `DECIDED` entry in `plan/DECISIONS.md` supports.
  Mockup sample values (prices, names, day counts, ₹ limits, product names of couriers/accounting tools, store
  names and colours in the root admin mockup) are not requirements. Gaps → add a decision to `DECISIONS.md` and ask.
- Use the IDs in `plan/00-conventions.md` (modules M##, entities E-*, pages P-*/P-R##, roles R-*, APIs API-*,
  tasks T-*, capabilities CAP-*, packs VP-*, templates TPL-*, config keys CFG-*, terminology TT-*).
- Never start Phase 2/3 (`LATER`) work, never skip dependencies, never duplicate an existing module/component —
  search the codebase first.
- Money is never floating point (BP §8.1); never trust client-sent price, role, stock, seller or refund-approval
  fields (BP §17.4); payments/refunds/orders must be idempotent (BP §10.2–10.3).
- **SaaS rules (`plan/14-continuation-protocol.md` → SaaS section):** `store_id` on every store-scoped table from
  its first migration · zero configuration I/O on the request path · a declared capability on every store-facing
  route and component, disabled = **404** not 403 · terminology tokens instead of hard-coded concept words, theme
  tokens instead of hard-coded visual values · nothing store-facing may reveal the platform · no import across the
  `root-admin/` boundary · two stores (one test installation) in every integration and E2E run, and two installations
  for the independence suite `TS-SAAS-INST` · nothing shared between customers' installations except code and
  releases, and no customer business record in the Root Admin (`D-287`).
- **The client-facing mockup is the reference store** (`D-279`): it shows a store with **every feature enabled**
  and is deliberately not gated. It will not tell you what to gate — `plan/21-feature-map.md` will.
- The user commits to git themselves unless they ask you to commit. Work happens on branches and reaches `main` only
  through a reviewed pull request (`D-295`).
- **Production publishes only the applications** (`D-294`): no mockup, plan or internal folder in any production
  deployment.
- Developer dashboard: `python3 plan/tools/dashboard.py` — a local, read-only view of the tracker (`D-293`).

## Plan index
`plan/README.md` lists every plan file and what it covers.
