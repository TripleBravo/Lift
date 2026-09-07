# Workout Tracker — Design Spec

Draft v2. Nothing here is code. Mark up anything that's wrong.

---

## 1. Vocabulary

| Term | Meaning |
|---|---|
| **Cycle** | The repeating loop. Length = the deload week number. |
| **Week** | Position inside a cycle. Also one session for that lift. |
| **TM** | Training Max. The number all core percentages are computed from. |
| **Core lift** | Percentage-driven. App computes the weight. |
| **Accessory lift** | Flat numbers. You type everything. |

---

## 2. Core lift

### Setup fields

| Field | Example |
|---|---|
| Name | Squat |
| Day of week | Monday |
| Training max | 400 lb |
| Starting percent | 65% |
| Scaling percent | 5% |
| Deload week | 4 (cycle is 4 weeks long) |
| Mesocycle increase | 5% |

### Set structure

Always **3 sets**. Reps are **5 / 5 / AMRAP**. Only the third set takes a typed rep count.

The scaling percent does double duty — it advances the base week to week, *and* steps up the three sets within a session.

```
base_percent(cycle, week) = starting_percent
                          + (cycle − 1) × mesocycle_increase
                          + (week  − 1) × scaling_percent

set_percent(n)            = base_percent + (n − 1) × scaling_percent    n = 1, 2, 3
```

Deload week ignores all of this: **3 sets at 50% of TM**.

Worked out for the example above:

| | Set 1 | Set 2 | Set 3 (AMRAP) |
|---|---|---|---|
| C1 W1 | 65% · 260 | 70% · 280 | 75% · 300 |
| C1 W2 | 70% · 280 | 75% · 300 | 80% · 320 |
| C1 W3 | 75% · 300 | 80% · 320 | 85% · 340 |
| C1 W4 | 50% · 200 | 200 | 200 |
| C2 W1 | 70% · 280 | 75% · 300 | 80% · 320 |
| C2 W2 | 75% · 300 | 80% · 320 | 85% · 340 |
| C2 W3 | 80% · 320 | 85% · 340 | 90% · 360 |

The overlap between C2 W1–W2 and C1 W2–W3 is **intentional** — a deliberate ramp, tuned by the mesocycle increase at lift creation.

### Rounding

All computed weights round **down to the nearest 5 lb**. Applies to working weights and to the TM.

---

## 3. TM progression

The TM never moves on a schedule. It moves only when you beat it.

1. The **final core set is always AMRAP**, deload weeks included. You enter reps completed.
2. Estimated 1RM by Epley: `weight × (1 + reps / 30)`.
3. If it beats the current TM, a popup fires immediately:

   > **New Max 447** (capped 420)

   First number is the raw Epley estimate. Parenthetical appears only when the cap bites.
4. The new TM is **staged, not applied**. It sits pending through the rest of the cycle and takes effect when the next cycle begins.
5. Multiple breaks in one cycle: the **highest** estimate wins.
6. **Cap:** never more than **+5%** in one cycle. `new_TM = min(best_estimate, current_TM × 1.05)`, floored to nearest 5.
7. **No floor.** A bad AMRAP does nothing. TMs only fall when you edit them by hand.

TM is editable manually at any time.

### At rollover
Both dials move together. A staged TM applies **and** the base percentage climbs by the mesocycle increase in the same cycle. Breaking your TM therefore produces a larger jump than a normal rollover — 400 → 420 with base 65% → 70% takes set 1 from 260 to 294. Intended.

---

## 4. Session structure

One session = **one core lift** plus its accessories.

```
┌─ Header ──────────────────────────────┐
│ Squat            0:00:04    ⏱  📅  ⋮  │
├───────────────────────────────────────┤
│ Cycle# 2 · Week# 3 · TM 400           │
├───────────────────────────────────────┤
│ ☐  Warm up                            │  ← checkbox only, no data
├───────────────────────────────────────┤
│ ▸ Squat — Main                        │  ← computed, collapsed
├───────────────────────────────────────┤
│ ▸ Calf raises                         │  ← accessories, from template
│ ▸ Good morning                        │
│ ▸ Leg curl                            │
├───────────────────────────────────────┤
│ Mon Sep 07, 2026                      │
│         [ SKIP ]  [ SAVE & CLOSE ]  ⊕ │
└───────────────────────────────────────┘
```

- Everything **collapsed until tapped**. One block expanded at a time.
- Expanded block shows `Set# | − reps + | − lb +` with a completion circle per row.
- **Computed weights are editable.** The app suggests; you override without touching the TM.
- Per-block controls: **history · add set · remove set · note**, plus **delete** on accessories. No per-block save.
- **SAVE & CLOSE is the only commit.** In-progress state must survive closing the app.
- ⊕ adds an accessory to this session.

### Warm up
A checkbox. No weights, no sets, no computation. A reminder not to go in cold.

### Navigation
- **SKIP** — logs nothing, advances to the next week.
- **Restart cycle** — replays the current cycle from week 1 at the same percentages. For coming back after missing a chunk without landing on a heavy week cold.

Percentages are allowed to exceed 100% of TM. No cap, no warning. If it happens, it means missed sessions or a genuine plateau, and the fix is manual — restart the cycle or drop back to an earlier one.

---

## 5. Accessory lifts

| Field | |
|---|---|
| Name | typed |
| Weight | typed |
| Reps | **per set** — e.g. 17 / 10 / 10, not one number |
| Note | optional, per session |

Accessories **attach to their core lift** and persist as its template. Next session pre-fills with last session's values. Edits stick going forward.

On **deload weeks**, accessory set count is halved, rounded down. 3 sets → 1.

*(Parked for later: a "sessions at this weight" badge.)*

---

## 6. History

History icon on any block opens a modal over the session:

```
History

Thu Sep 03, 2026
17 x 225.0
10 x 225.0
10 x 225.0
taking it easy to see how calves feel

Mon Jul 27, 2026
25 x 155.0
...
```

Reverse chronological, full scrollback, every set listed. That day's note sits at the bottom of its entry. Notes are per-exercise-per-session.

---

## 7. Home screen

A **status board**, not a to-do list. Says where you stand, not what's next. ⊕ starts a workout.

- Core lifts as columns.
- Each shows its own **independent cycle/week position** (C2:W3, C1:W2) — lifts drift out of sync rather than advancing together.
- Row 1: **Training max**. Row 2: **Estimated max**.
- Latest workout, dated, at the bottom.
- Accessories don't appear here.

### Visual direction
Dark grey on black, single red accent, oversized numerals. Readable at arm's length with a sweaty phone mid-set.

---

## 8. Platform

**Installable web app (PWA)** on the Pixel 10 Pro.

- Needs a real URL to install and store data — free static hosting (GitHub Pages or Netlify), set up once.
- Open once in Chrome → "Add to Home screen" → launches like an app, works offline in the gym.
- Data lives on-device in browser storage.
- **Manual export/import** of the full log as a JSON file. Browser storage is wiped by clearing site data, so this is the real backup. App nags if an export hasn't happened recently.

---

## 9. Data model (sketch)

```
Lift
  id, name, dayOfWeek
  trainingMax, startingPercent, scalingPercent,
  deloadWeek, mesocycleIncrease
  currentCycle, currentWeek
  pendingTM            ← staged, applied at cycle rollover
  bestEstimateThisCycle
  accessoryTemplate[]  ← name, weight, reps[]

Session
  id, liftId, date, cycle, week, tmUsed
  warmupChecked
  coreSets[]           ← setNumber, percent, weight, targetReps, actualReps, done
  accessories[]        ← name, weight, sets[{reps, weight, done}], note
  coreNote
  status               ← completed | skipped | in-progress
```

---
