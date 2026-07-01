# Betting Hospital — Consolidated Feature Plan

All work stays inside `public/arit.html`. No existing module, engine, storage key, or analytic is removed unless listed here. Everything additive is namespaced under `window.ARIT_V6_*` and lives in its own IIFE so it can be toggled/removed cleanly.

Note: `Srirangam` tab does not exist in the current build — nothing to remove. Confirming with you before I skip it.

---

## 1. Rebrand — "Betting Hospital" Demo Tool

- `<title>` → `Betting Hospital — Demo Tool`.
- Header logo (`.logo` at line 486) → `Betting Hospital <em> — Demo Tool</em>`.
- All "Om Namo — S8" text/references removed from header, footer, exports, and report titles.
- No functional changes.

## 2. Global Font — Sora

- Add `<link rel="preconnect">` + Sora import in `<head>`.
- Set `--font-body` and `--font-head` CSS variables to `"Sora", system-ui, sans-serif` (fallback preserved).
- Every existing text keeps its size/weight; only the family changes.

## 3. Live Decision Coach 2.0 — Enable / Disable / Collapse + Improvements

- New header row on the Coach 2.0 card:
  - Collapse/expand chevron (state stored in `ARIT_V6_UI.coachCollapsed`).
  - Enable/disable switch (state in `ARIT_V6_UI.coachEnabled`). When disabled the card collapses to a 1-line stub and `renderCoach()` short-circuits — zero CPU.
- Improvements (still read-only, still using the 5-tier source ladder):
  - New "Why" strip under status — 3-4 bullet reasons pulled from the same signals the status engine already scores (Confidence, Agreement, Master Verdict, Readiness, Shift, Discipline). Same format as the Operator Insight Center's "Explain WHY" (see §10) so both stay consistent.
  - Delta chips on Top Numbers / Best Dozen / Best Column showing change vs previous render (↑ new, = same, ↓ dropped).

## 4. Settings Warning Notification

- On tool open (and on tab switch back to Spin), evaluate the settings snapshot against a small ruleset ("required toggles present", "confidence threshold in sane range", "history window ≥ N", etc.).
- If mismatches found, show a persistent yellow banner at top of the tool: `⚠ Check Settings — wrong settings can negatively affect predictions. [Review]`.
- `[Review]` opens the existing Settings tab.
- Dismissible per-session (not per-mismatch) so it re-appears next open until fixed.

## 5. Floating Adviser Mini-Window

- New draggable, minimizable popup (bottom-left by default).
- Content: Adviser Decision (PLAY / CAUTION / WAIT), Primary/Secondary/Offset current picks, hit-rate last 10, confidence bar.
- Reads directly from existing Adviser state — no recompute.
- Minimize → collapses to a small pill showing just the Decision word + color.
- Position + minimized state persisted in `ARIT_V6_UI.adviserMini`.

## 6. Table Analytics (V5.2) — Reposition

- Move the V5.2 launcher/panel to bottom-left of the tool.
- Panel opens as a bottom-left docked drawer (max-height 60vh, scrollable), never overlapping the Adviser Mini (§5) or Workspace button (existing bottom-right).
- No changes to what V5.2 renders.

## 7. V5.2 — Decision Validation & Wheel-vs-Table Agreement: Last 5 / Last 10

- Add a filter row (`All | Last 10 | Last 5`) inside both sections (lines ~11232 and ~11239).
- Same computations, sliced to the last N validated events. Renders side-by-side counts so you can compare "recent vs all".

## 8. Soft Hit Notifications + Notification Center

- Replace the current pop-up toast burst with a small always-present "Hit Feed" pill (top-right, next to logo).
- Each hit appends a subtle 1-line entry (color-coded by source) plus a very short soft sound (optional, off by default).
- Click the pill → expands into a Notification Center panel listing last 50 hits with source, spin #, time. Minimize/maximize state in `ARIT_V6_UI.notifCenter`.
- `ARIT_V2.notify` intercepted so nothing else in the codebase has to change.

## 9. Wheel Tab — Half-Wheel Overlay Correction

- Fix label + slicing so `HALF A (Green — Pos 0–23)` and `HALF B (Blue — Pos 10–26)` are computed against wheel-position order (not number order), matching the intended overlap band.
- Update the overlay renderer at lines ~7594 and ~7632 to use the corrected slices and re-color the tiles accordingly. Legend updated to match.

## 10. Execution Tab — Live Event Timeline Filters

- Add filter chips above the timeline (line ~9510): `All | Last 5 | Last 10 | Hits only`.
- Each chip recomputes hit rate + source breakdown for that window and shows a compact header line: `Window: Last 10 — Hits 6/10 (60%) · Top: Adviser Primary 3, Zone 2`.
- Purely a view filter, no engine changes.

## 11. Header — Quick Spin Entry (works from any tab)

- New button in header next to the (rebranded) logo: `⚡ Quick Entry`.
- Opens a small floating popup with the same input + primary buttons as the Spin tab (Enter Spin, Undo, Clear last).
- Delegates to existing `doEnterSpin()` etc. — zero duplicated logic.
- Popup is draggable, minimizable (collapses to a pill showing the last spin number). Position/state in `ARIT_V6_UI.quickEntry`.

## 12. Operator Insight Center (persistent alert bar)

Read-only interpretive layer over existing analytics. Nothing new is predicted.

- Persistent thin bar under the header (auto-collapsed) that only expands when a meaningful change is detected.
- Bar content when expanded (example): `OPERATOR INSIGHT · Gold resolved in 2 · Zone strongest · Adviser Primary agrees with Wheel · Execution Confidence ↑ [View Details]`.
- `[View Details]` opens a dedicated Insight Center modal with these sub-panels — each panel reads from existing state (no recompute of primary metrics):

  1. **Source Changes** — one row per source (Gold, Silver, Bronze, Zone, **Lifetime**, **Adviser Primary**, **Adviser Secondary**, **Adviser Offset**, Wheel, Table, Active Play Plan) with: Working (Y/N), Hit-in-N, Avg window, Trend arrow.
  2. **Lifespan Tracker** — for Lifetime + each Adviser stream: Open Spin #, Hit Spin #, Lifetime spins, Avg / Best / Worst. This fixes the current gap in SOURCE FLOW TRACKER (line ~6770) which only surfaces Gold/Silver/Bronze/Zone.
  3. **Adviser Tracker** — Primary / Secondary / Offset: Predictions, Hits, Avg Window, Current Trend.
  4. **Wheel Direction Health** — Predicted direction vs actual hit direction over last N; mismatch counter + advisory (`Reduce Wheel Confidence` / `Wheel recovered`). Purely advisory text.
  5. **Synchronization Matrix** — grid of Wheel ↔ Adviser, Wheel ↔ Table, Adviser ↔ Zone, Active Plan ↔ Gold, Continue/Stop ↔ Behaviour, each showing Match / Opposite / Weak.
  6. **Sudden Change Detection** — flags: Behaviour changed, Wheel Strong→Weak, Adviser disagrees, Outside accuracy dropping, Resolution increasing.
  7. **Explain WHY** — reason list for the current top-level state (READY / WAIT / STOP), same bullet format as Coach 2.0's Why strip (§3).

- **Popup priority** — the persistent bar only auto-expands (and pushes a single soft toast via §8 pipeline) on level ≥ WATCH:
  - INFO (silent, log only): recovered sources, minor deltas.
  - WATCH: sources diverging, wheel/adviser disagreement.
  - WARNING: lifetime increasing, sync matrix majority-weak.
  - ACTION: Continue/Stop flips to STOP, discipline STOP.
- Snooze + "don't show INFO" toggles stored in `ARIT_V6_UI.insight`.

Implementation: new IIFE `ARIT_V6_INSIGHT` that samples the same snapshots Phase 2/3/4 already expose plus small readers into existing Adviser/Execution/Wheel state. No writes into any engine.

## 13. Prediction-Change Popup

- Whenever a top-line prediction source changes (Gold pick swaps, Adviser Primary changes, Active Play Plan swaps method, Wheel direction flips), a single non-blocking popup surfaces: `New: use <source> — <pick> · reason: <one-line>`.
- Uses the same soft notification pipeline from §8 (so it participates in the notification center history, no toast storms).
- Debounced (one popup per source per 3 spins).

---

## Files touched

- `public/arit.html` only — rebrand, font swap, Coach 2.0 header controls + Why strip, Settings banner, Adviser Mini IIFE, V5.2 launcher relocation + Last 5/10 filters, Notification Center + soft-hit interception, half-wheel overlay fix, Live Event Timeline filters, Quick Entry header popup, Operator Insight Center IIFE, Prediction-Change popup.
- `src/routes/index.tsx` title/description → Betting Hospital.

## Verification checklist (I will run after build)

1. Title + header show "Betting Hospital"; no "Om Namo" strings remain (`grep`).
2. Sora font applied on every tab (screenshot of Spin / Wheel / Execution / V5.2).
3. Coach 2.0 collapses, disables, and re-enables; state survives reload.
4. Settings banner appears on load and disappears after fixing the flagged setting.
5. Adviser Mini drag + minimize; content matches Adviser tab.
6. V5.2 opens bottom-left; does not overlap Adviser Mini or Workspace button.
7. Last 5 / Last 10 filters produce different counts vs All.
8. Multi-hit burst produces one soft entry per hit in Notification Center, no toast spam.
9. Half-wheel overlay tiles match the corrected Pos ranges.
10. Timeline filter chips change hit-rate header line.
11. Quick Entry from Execution tab appends a spin visible in Spin tab.
12. Operator Insight bar stays collapsed on INFO, auto-expands on WATCH+, modal shows all 7 sub-panels populated.
13. Prediction-Change popup fires once per real change, not per render.

## Open question

- You said "Remove Srirangam Tab" but the current build has no Srirangam tab. Should I ignore this line, or is it a tab you expected to see and want me to hunt for under a different name?
