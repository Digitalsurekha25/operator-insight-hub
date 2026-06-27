# ARIT V6 — Phase 1 Build Plan

## Phase 1 scope (this turn)
1. **Clone the existing tool** from `Maheshonlinelife/stake-0m-smart-flow-8-newbornbaby` into this project, replacing the placeholder TanStack starter. All existing files preserved verbatim — no edits to prediction engines, Wheel/Table Intelligence, Adviser, Agreement, Bankroll, SRIRANGAM, V5.1/5.2/5.3.
2. **Spin tab — Bulk Results Entry (30+)** with enable/disable toggle.
3. **Module 9 — Session Bootstrap & Bulk Import** (the engine powering #2).
4. **Module 1 — Live Decision Coach 2.0** (always-on Inside + Outside panels).

Modules 2–8 and 10–13 are deferred to later phases per your choice.

---

## Step 1 — Bring in the existing tool
- Clone the GitHub repo into a temp dir, then copy its source (`src/`, `index.html`, `package.json`, configs, public assets, etc.) into this project root, overwriting the TanStack starter shell.
- Run `bun install` so its declared deps resolve.
- Quick smoke check: dev server boots, original tabs render unchanged.
- Storage stays **local only** (localStorage / IndexedDB) — no Cloud, no backend.

## Step 2 — Spin tab Bulk Results panel
Add a collapsible "Bulk Old Results" card at the top of the Spin tab.

Controls:
- **Enable Bulk Mode** toggle (persisted in `arit_v6_ui.bulkEnabled`). When off, the panel collapses to a single-line button — original Spin UI is fully untouched.
- **Textarea** accepting flexible input: commas, spaces, tabs, newlines all valid separators. Min 30 entries enforced before "Import" enables; soft warning below 30.
- **Mode selector** (radio): `Replace` · `Append` · `Bootstrap`.
- **Use historical data in analytics** toggle (independent of import — lets the operator turn imported history on/off without re-importing).
- **Validate** button: parses input, shows preview (`Accepted: N · Rejected: M · Duplicates kept: K`), lists rejected tokens with reason (not 0–36, non-numeric, OCR noise like `O→0`, `l→1` auto-fix suggestions).
- **Import** button: commits via the Module 9 engine.

UI lives in a new `src/components/v6/BulkResultsPanel.tsx`, mounted into the existing Spin tab via the smallest possible insertion point — no edits to spin logic.

## Step 3 — Module 9 engine (`ARIT_V6.bootstrap`)
New file `src/lib/arit_v6/bootstrap.ts`. Pure functions, no prediction side-effects.

- `parseBulk(text): { accepted: number[], rejected: {token,reason}[] }` — strips OCR noise, validates 0–36.
- `importHistory(numbers, mode)`:
  - `Replace` → overwrites `arit_v6_history.imported`.
  - `Append` → appends to imported, live session continues.
  - `Bootstrap` → seeds `imported` array AND marks `liveStartIndex` so live session starts at the next push. Imported and live remain logically separate but analytics can union them.
- `setHistoryEnabled(bool)` — flips `arit_v6_history.enabled`; analytics readers respect this flag.
- Emits via existing `ARIT_V2.notify()` — never creates predictions.

Persisted under `localStorage["ARIT_V6_HISTORY"]` and `localStorage["ARIT_V6_UI"]`.

## Step 4 — Module 1: Live Decision Coach 2.0
New component `src/components/v6/LiveDecisionCoach.tsx` replacing the visual region of the current coach (existing data sources untouched — V6 reads them).

Two always-rendered panels:

**Inside Bets**
Status (READY/WAIT/MONITOR/DEFENSIVE) · Top Numbers · Best Street · Best Double Street · Best Corner · Best Split (gated by existing splits-enabled flag) · Agreement · Confidence · Prediction Age · Expected Hit Window · Historical Average Window · Current Lifetime · Reason · Recommendation.

**Outside Bets**
Best Dozen · Best Column · Best Outside Bet · Best Color · Best Odd/Even · Best High/Low · Agreement · Confidence · Prediction Age · Historical Resolution · Rotation Status · Reason · Recommendation.

Every field renders a meaningful placeholder (e.g. `"Awaiting 5 more spins"`, `"No active prediction"`) — never blank. Selector logic lives in `src/lib/arit_v6/coach.ts`, reading existing prediction state read-only.

## Step 5 — Namespace & technical guardrails
- All new code under `ARIT_V6.*` (`window.ARIT_V6` global mirror so original code can read it but never the other way).
- All new files under `src/lib/arit_v6/` and `src/components/v6/`.
- Zero edits to existing prediction, wheel, table, adviser, agreement, bankroll, SRIRANGAM, V5.x files.
- Reuses `ARIT_V2.notify()` for all toast/log surfaces.
- UI prefs only in `arit_v6_ui`; history only in `arit_v6_history`.

## Step 6 — Verification
- Boot dev server, confirm original tabs render identically.
- Paste 35 mixed-format numbers + a couple of invalids → validate preview correct → import in each mode → confirm history persists across reload and the enable/disable toggle hides imported data from analytics readers.
- Confirm Coach 2.0 shows every field with a non-empty value on a cold session and on a live session.

---

## Deferred to later phases
Modules 2 (Source Performance), 3 (Current Table Strategy), 4 (Independent Outside Analytics), 5 (Lifecycle), 6 (Resolution Windows), 7 (Fast Shift), 8 (Session Readiness), 10 (Operator Workspace), 11 (Continue/Stop), 12 (Session Learning), 13 (Discipline). Each will be its own append-only ARIT_V6 module in a follow-up turn.