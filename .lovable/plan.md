# Live Decision Coach 2.0 — Source-Priority Fix

Keep every other module exactly as-is. Only the Phase 1 Coach internals inside `public/arit.html` change.

## Why Coach 2.0 looks "broken"

The current `snapshot()` reads fields that mostly don't exist in normal Spin entry:
- `execAnalytics.tableAnalytics` — never defined anywhere
- `window.envState` — declared inside an IIFE, not on `window`
- `execSnap` / `execLifetime` — only populated when an active execution prediction is open

So with 100+ results, most rows stay on "Awaiting prediction" and status stays weak — even though ARIT and V6 already compute the real answers.

## Strict source priority (no skipping)

Every Coach field must walk this ladder top→bottom and stop at the first source that returns a real value:

```text
1. Existing ARIT Engine        (buildExecution, ARIT_V2._state, settings, DOM)
        ↓
2. Existing Execution Analytics (execAnalytics: tiers, predLog, regimeHistory,
                                  lifetimes, activePlan, lastHitAnalysis,
                                  scoreHistory)
        ↓
3. Existing V6 Modules         (ARIT_V6_PHASE2.snapshot, ARIT_V6_PHASE3.snapshot,
                                  ARIT_V6_PHASE4.masterVerdict)
        ↓
4. Execution State             (execSnap, execLifetime, execStats)
        ↓
5. Fallback Display Helper     (isolated, marked fallback-only, last resort)
```

Reused values must NOT be recomputed. The Coach never re-implements: Agreement, Source Performance, Resolution Analytics, Continue/Stop, Behaviour Engine, Active Play Plan.

## Field → Source Map (drives the implementation)

| Coach Field | Primary | Secondary | Fallback |
|---|---|---|---|
| Top Numbers | `buildExecution().gold/silver/clusterZone` | `execAnalytics.activePlan` numbers | recent spin frequency (last 30) |
| Best Street / D-Street / Corner / Split | `buildExecution()` tier rows / existing table-analytics DOM | V6 Phase 3 strategy | inside-set frequency last 30 |
| Best Dozen / Column | `buildExecution()` / existing dozen-col DOM | `ARIT_V6_PHASE3.outside` percentages | last-20 dozen/column majority |
| Best Color / Parity / High-Low | existing outside DOM (`renderOutside`) | `ARIT_V6_PHASE3.outside` | last-20 majority |
| Agreement | `buildExecution()` row source flags (Predict/Wheel/Adviser) | — | — (never compute) |
| Confidence | `buildExecution().conf` | `ARIT_V6_PHASE4.masterVerdict` vote ratio | sample-size heuristic only |
| Strength | `execAnalytics.regimeHistory` (existing regime block) | `ARIT_V6_PHASE2.lifecycle` | — |
| Prediction Age / Expected Window / Lifetime | `execLifetime` / `execSnap.age` / `execSnap.life` | — | — |
| Historical Avg Window | `ARIT_V6_PHASE2.resolutionWindow.avgRepeat` | — | — |
| Historical Resolution | `ARIT_V6_PHASE2.resolutionWindow.avgZone` | — | — |
| Rotation Status | `ARIT_V6_PHASE3.shift` (STABLE/DRIFT/SHIFT) | `execAnalytics.regimeStreaks` | — |
| Continue/Stop hint | `ARIT_V6_PHASE2.continueStop` | `ARIT_V6_PHASE4.masterVerdict` | — |
| Discipline hint | `ARIT_V6_PHASE3.discipline` | — | — |
| Recommendation | derived ONLY from above reused signals | — | generic guidance |
| Status (READY/MONITOR/WAIT/DEFENSIVE) | composite of reused values (conf + master verdict + readiness + shift) | — | sample-size gate |

## Improvement 1 — Coach Data Source Map (dev console report)

On Coach boot (and on demand) print one table to the console:

```text
[ARIT V6 Coach] Data Source Map
┌──────────────────────────┬──────────┬─────────────┬────────────┬───────────┐
│ Field                    │ Primary  │ Secondary   │ Fallback   │ Status    │
├──────────────────────────┼──────────┼─────────────┼────────────┼───────────┤
│ topNumbers               │ ARIT     │ ExecAnalyt. │ Freq30     │ Connected │
│ bestDozen                │ ARIT     │ V6_P3       │ Last20Maj  │ Connected │
│ continueStop             │ V6_P2    │ V6_P4       │ —          │ Connected │
│ rotation                 │ V6_P3    │ ExecAnalyt. │ —          │ Fallback  │
│ … (one row per field)    │          │             │            │           │
└──────────────────────────┴──────────┴─────────────┴────────────┴───────────┘
```

Rules:
- Printed once on first successful render via `console.table(...)`.
- Status values: `Connected` (primary returned), `Secondary`, `Fallback`, `Missing`.
- Toggle via `window.ARIT_V6_COACH_DEBUG = false` to disable, or `localStorage.ARIT_V6_COACH_DEBUG='0'`.
- Re-print on demand: `window.ARIT_V6.coach.printSourceMap()`.

## Improvement 2 — Fallback Guard (never silent)

Every Coach field goes through a single `pick(field, ladder)` resolver:

- Tries sources in priority order.
- Records `{ field, sourceUsed, value, isFallback }` into `window.ARIT_V6.coach._lastSources`.
- Tags fallback values with an internal flag `__fallback:true`.
- In the rendered UI, fallback values get a small subtle `ƒ` marker + tooltip `Fallback display value — primary source unavailable`. No layout change.
- A warning is logged once per session listing which fields are currently on fallback, so future engine work can replace them cleanly.
- Public introspection:
  - `ARIT_V6.coach.sources()` → current source map snapshot
  - `ARIT_V6.coach.fallbacksInUse()` → list of fields currently falling back

This guarantees we never confuse fallback display values with real analytics.

## Status engine (reuses existing signals only)

```text
score = 0
+2 if buildExecution().conf >= 6
+2 if Agreement present (>=2 sources align)  ← from ARIT, not recomputed
+2 if masterVerdict == 'go'                  ← from V6 Phase 4
-3 if masterVerdict == 'stop'
+1 if V6_P3.readiness.ready
-2 if V6_P3.shift.pill == 'stop'
-1 if V6_P3.discipline.pill in ('warn','stop')

>= 5 → READY
2..4 → MONITOR
0..1 → WAIT
< 0  → DEFENSIVE
```

If the source ladder yields nothing for confidence/agreement (cold start), status falls back to a pure sample-size gate (`<5` WAIT, `<15` MONITOR, else READY-lite) and is flagged as fallback in the source map.

## Files touched

- `public/arit.html` — only the Phase 1 Coach 2.0 block (`snapshot()`, `statusOf()`, `renderCoach()`, and a small helper section for `pick()` + source-map logging). The bulk-import panel, all of Phase 2–5, ARIT V3 coach, execution engine, Active Play Plan, and every other module are left untouched.

## Verification

1. Load tool, paste 100+ results via Bulk Import.
2. Open DevTools console → confirm `[ARIT V6 Coach] Data Source Map` table prints once.
3. Confirm Coach 2.0 rows show real values (dozen/column/top numbers/agreement/confidence) instead of "Awaiting prediction".
4. Call `ARIT_V6.coach.fallbacksInUse()` — should be a short list, not "everything".
5. Confirm no behavior changes in Phase 2/3/4/5 cards, Workspace, Alert Bar, Exports, V3 coach, or original ARIT execution analytics.
