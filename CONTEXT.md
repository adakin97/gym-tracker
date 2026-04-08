# Gym Tracker App — Context

> **Last updated:** 2026-04-08

---

## Overview

Single-file PWA gym tracker. All HTML, CSS, and JS inline in `index.html`.

- **Live URL:** https://adakin97.github.io/gym-tracker
- **GitHub:** https://github.com/adakin97/gym-tracker (branch: main)
- **Local:** `C:\Users\alexd\Claude_Projects\Gym 2026 Claude\gym-app\`

---

## Tech Stack

| Layer | Tech |
|---|---|
| Frontend | Single-file PWA (vanilla HTML/CSS/JS, no framework) |
| Auth | Firebase compat SDK v9.22.0 — Google sign-in only |
| Database | Firebase Firestore |
| Hosting | GitHub Pages |
| Images | free-exercise-db (GitHub raw CDN) |

---

## Firestore Structure

```
users/{uid}/logs       — workout sets (exercise, exIdx, date, setIdx, weight, reps, workout)
users/{uid}/weights    — body weight entries (date, weight)
users/{uid}/weekPlans  — per-week schedule overrides keyed by mondayStr (YYYY-MM-DD)
                         each doc: { 0: null|wkName, 1: ..., ..., 6: ... }
                         default plan (Mon=Upper A, Tue=Lower A, Thu=Upper B, Sat=Lower B) used when no doc exists
```

---

## Current Features (as of 2026-03-11)

### Today Tab
- **Date selector** — 7-day Mon–Sun strip; shows workout name + ✓ on days with logs; future workout days at 0.5 opacity, rest days at 0.25
- **Workout badge dropdown** — tappable badge shows current workout (e.g. "Upper A ▾"); opens picker with all 4 workouts + rest; scheduled workout has "✓ Scheduled" tag
- **Collapsible workout overview card** — shows exercise list with muscle group + sets×reps; auto-opens when switching workout
- **Exercise cards** — image from free-exercise-db, muscle group, sets×reps, tempo, rest
- **Per-set logging** — ○ button per set; logs weight+reps to Firestore; tap ✓ again to un-log
- **Progressive overload nudge** — shows `→ try X kg` next to prev best weight
- **Rest timer** — floating countdown pill with shrinking bar; vibrates on completion
- **Workout complete banner** — centred full-card modal when all sets done; shows total volume + session duration; tap to dismiss
- **Workout duration timer** — starts on first logged set (`workoutStartTime`); resets on workout/date change
- **Rocket animation** — full-screen space animation on workout complete (4s flight, 70 stars, auto-dismisses after 4.8s)
- **Exercise swap** — tap swap icon to open bottom sheet; search all exercises; shows default badge; clears on workout/date change

### History Tab
- Grouped log entries by date → exercise → sets
- Delete entire session via × button on each session card

### Weight Tab
- Log bodyweight with date picker
- History list with delete (×) button per entry

### Graphs Tab
- **Exercise progress chart** — dropdown of all logged exercises; SVG line chart with gradient fill; shows peak / latest / progress (kg) / sessions stats
- **Body weight chart** — SVG line chart; shows start / current / gained / to goal stats; dashed green goal line at 70 kg

---

## WORKOUTS Object

```js
const WORKOUTS = {
  1: { name: 'Upper A', focus: 'Strength',    exercises: [...] },  // Monday
  2: { name: 'Lower A', focus: 'Strength',    exercises: [...] },  // Tuesday
  4: { name: 'Upper B', focus: 'Hypertrophy', exercises: [...] },  // Thursday
  6: { name: 'Lower B', focus: 'Hypertrophy', exercises: [...] },  // Saturday
};
```

Each exercise: `{ name, muscle, sets, reps, tempo, rest, imgId }`

### Exercise Library

`EXERCISE_LIBRARY` array contains ~94 curated exercises from free-exercise-db, merged with workout exercises into `ALL_EXERCISES` (deduped). Used for the swap overlay.

### Bodyweight Exercises

Exercises in `['Plank', 'Hanging Leg Raises', 'Dead Bug']` are flagged as `noWeight`: weight input is hidden (sends 0), previous best / overload nudge hidden. Plank uses "mins" label instead of "reps".

### Overload Nudge

Pull-ups use reversed overload logic (`prev.weight - 2.5` instead of `+ 2.5`) since they use an assisted machine where lower weight = harder.

---

## Key State Variables

| Variable | Purpose |
|---|---|
| `selectedDate` | Date currently being logged for (YYYY-MM-DD) |
| `scheduledDow` | Scheduled workout DOW for selected date |
| `selectedDow` | Active workout day of week |
| `workout` | Current workout object |
| `allLogs` | All Firestore log entries |
| `allWeights` | All Firestore weight entries |
| `swappedExercises` | `{exIdx: {name, imgId, muscle}}` — cleared on workout/date change |
| `extraSets` | Net set delta per exercise index (can be negative for removed default sets) |
| `workoutStartTime` | Timestamp (ms) of first set logged this session; null if not started |
| `summaryOpen` | Whether overview card is expanded |
| `selectedGraphEx` | Selected exercise in Graphs tab |
| `_restInterval`, `_restTotal` | Rest timer state |
| `_swapExIdx` | Currently open swap sheet index |

---

## Constants

```js
const WEIGHT_GOAL = 70;  // kg — used for body weight graph goal line
```

---

## Key Patterns

- `switchTab(tab)` — shows/hides `.view` divs, triggers render for history/weight/graphs
- `renderToday()` — main render; called after auth, date change, workout change
- `clearSwaps()` — clears `swappedExercises`, cancels rest timer, hides complete banner
- `buildProgressChart(data, options)` — generic SVG line chart (gradient fill, dots, labels, optional goal line)
- `filterSwap(query)` — filters and sorts swap overlay exercises; same muscle group as current exercise shown first, then alphabetical
- All inline `onclick=` handlers exposed via `window.fn = fn`
- Dynamic iOS notch: `padding: calc(env(safe-area-inset-top) + 8px) ...` + `viewport-fit=cover`

---

## Session Log

| Date | Changes |
|------|---------|
| 2026-02-26 | Initial build: PWA, Firebase auth, exercise cards, per-set logging, history view (localStorage at first) |
| 2026-02-26 | Migrated to Firestore; bodyweight tracker tab added |
| 2026-03-03 | Day flexibility (selectedDate + date strip); exercise swap bottom sheet |
| 2026-03-03 | Scheduled workout highlight in picker; "Default" badge in swap sheet |
| 2026-03-03 | Bug fix: swap now clears on workout/date change (clearSwaps) |
| 2026-03-03 | Compact header: removed workout row, converted to badge dropdown |
| 2026-03-03 | Progressive overload nudge; rest timer (countdown + vibrate); workout complete banner |
| 2026-03-03 | iOS safe-area padding; weight entry delete; set un-log toggle |
| 2026-03-03 | Weekly schedule labels in date pills (workout name + ✓ on logged days) |
| 2026-03-03 | Collapsible workout overview card (replaces summary bar) |
| 2026-03-11 | Graphs tab: exercise progress SVG chart + body weight chart; fixed uncommitted push from prior session |
| 2026-03-12 | PWA icon (dumbbell, 512×512 PNG); haptic feedback (iOS no-op); history session delete; progress bar fix (extraSets can go negative); rocket launch animation on workout complete; workout duration timer; centred completion banner |
| 2026-03-25 | Fixed Firestore security rules. Replaced open test-mode rules (expired 28 Mar) with proper auth-scoped rules: only authenticated users can read/write their own users/{uid}/** data. Applied via Firebase console. |
| 2026-03-24 | 8 exercise replacements based on real-world feedback (Barbell Curl to EZ-Bar Curl, Barbell Bench to Chest Press Machine, Tricep Dips to Tricep Pushdown, Romanian Deadlift to Leverage Deadlift, Leg Curl to Seated Leg Curl, Barbell Row to Seated Cable Row, Incline DB Press to DB Bench Press, Leg Press to Hack Squat). Pull-ups overload nudge reversed for assisted machine. Plank tracked in mins. No weight input for bodyweight exercises (Plank, Hanging Leg Raises, Dead Bug). Added exercise library (~94 exercises) for swap overlay. Swap suggestions sorted by muscle relevance (same muscle group first). |
| 2026-04-08 | Weekly workout replanning feature. Workouts are now planned per-week rather than fixed by day-of-week. "↻ replan week" button below date strip opens a Mon–Sun bottom sheet with dropdowns (Rest / Upper A / Lower A / Upper B / Lower B) and a 0/4 counter. Plans saved to Firestore weekPlans/{mondayStr}. Default schedule (Mon/Tue/Thu/Sat) used when no custom plan exists. All 7 days editable including past days so skipped workouts can be marked as rest retroactively. |
| 2026-04-08 | Added `sw.js` service worker (network-first) so PWA home screen icon auto-updates on push. |
| 2026-04-08 | Past-date workout review fix. selectDate for past dates now uses the logged workout name (not the schedule) so switching workouts mid-session is reflected when looking back. Log entries now store exIdx (exercise slot index); swaps are reconstructed on past-date views by comparing logged exercise name against the template at that index. Old logs without exIdx still work — swaps just won't restore for those sessions. |

---

## Pending / Ideas

- Personal records (PR) tracking — highlight new max weight on set log
- Workout streak counter — consecutive weeks hitting all 4 workouts
- Notes per exercise — form cues / session feel
- Volume trend in graphs — total volume per session as second chart
- Rest timer customisation — per-exercise rest time
- Nutrition tracker tab
- Export logs as CSV
