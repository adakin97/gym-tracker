# Gym Tracker App — Context

> **Last updated:** 2026-08-18

---

## Pick up here (next session)

**2026-08-18 session: nutrition paused, morning posture block added. Changed locally, NOT committed or
pushed** (Alex said deployment can wait), so the live site is still the 14 Aug build.

**1. Nutrition tracking is paused behind one flag, not deleted.** Alex is using MyFitnessPal for food, so
every calorie surface is gated off `const NUTRITION_ENABLED = false;` (declared just above `KCAL_TARGET`).
Flip it to `true` to restore everything. What the flag hides: the fuel strip, the Gain tab's Maintenance
check / Today's fuel / food buttons / quick-add / repeat-yesterday / daily note / last-7-days strip (all
moved out of `gainHeader` into a separate `nutritionCards` template), the Recipes nav button, the rest-day
calories-needed note, the workout-complete banner's calories-owed stat, and the History tab's food and note
cards. `loadRecipes()` is also skipped so the `STARTER_RECIPES` seeding does not write to Firestore while
paused. **No Firestore data was deleted** — `nutrition` and `recipes` documents are untouched and will
reappear when the flag flips. Weight, trend, rate, verdict, projection, graphs and all workout tracking are
deliberately unaffected: only food logging was paused, not weight tracking.

**2. Morning posture block (new), built to the AthleanX rounded-shoulders protocol.** Alex supplied the
protocol transcript and asked to follow it, so the block is **staged and ordered**, not a flat list: mobilise
the thoracic spine, then stretch what is tight, then strengthen what is weak. The order is the point of the
protocol (a restricted t-spine is treated as the upstream cause, so shoulder work comes last), and the card
renders the stages numbered with the reasoning under each heading.

Kept **out of the `WORKOUTS` object on purpose**: low-load daily habit work, and folding it into a session
would corrupt volume totals, History and the progressive-overload nudge. Constants `POSTURE_STAGES` (the
staged structure), `POSTURE` (flattened for ticking), `POSTURE_REQUIRED`, `POSTURE_HABIT`. Firestore
collection `users/{uid}/posture/{date}` as `{ date, done: [names] }`.

| # | Stage | Exercise | Dose | Image (`imgId`) |
|---|---|---|---|---|
| 1 | Mobilise t-spine | Prone Roll-Back | 5 each side | none |
| 1 | Mobilise t-spine | Foam Roller Extension **(optional)** | 60 sec | `Rhomboids-SMR/0.jpg` (foam roller under upper back, exact match) |
| 2 | Stretch tight | Doorway Slide | 30-45 sec each side | none |
| 2 | Stretch tight | Pitcher's Stretch | 30-45 sec each side | none |
| 3 | Strengthen weak | Band Face Pull to Overhead | 2 x 10 slow | `Face_Pull/0.jpg` |
| 3 | Strengthen weak | High-to-Low Band Pull-Apart | 2 x 10 slow | `Band_Pull_Apart/0.jpg` |

**Images: 3 of 6.** free-exercise-db has no prone thoracic roll-back, no doorway stretch and no pitcher's
stretch; the near misses (`Side-Lying_Floor_Stretch`, `Chest_And_Front_Of_Shoulder_Stretch`) describe
different movements once you read their instructions, and a wrong picture on a card whose job is showing the
movement is worse than none. Those three render a dashed placeholder via `postureThumb()`, which is also the
`onerror` fallback for the three real images. **Open option: draw inline SVG diagrams, or embed photos Alex
supplies.**

**The optional item is a real design decision.** The foam-roller drill needs a foam roller and a med ball,
and it is unknown whether Alex owns either. It is flagged `optional: true`, badged "optional" in the UI, and
**excluded from `POSTURE_REQUIRED`**, so the target is 5/5 rather than 6/6. Without that, a missing foam
roller would make completion unreachable and the streak would never start. `postureDoneCount()` counts
required items only.

**Habit reinforcement** from the protocol (tennis or lacrosse ball between the shoulder blades while sitting)
renders as a footer note rather than a tickable row, because it is an all-day habit, not a morning rep, and
making it tickable would break the meaning of 5/5.

Behaviour: shows every day including rest days; **hidden on past dates**; tap a row to tick, tap the header
to collapse/expand; **collapses to a single "Posture done today" line** once the five required are done;
streak counts consecutive fully-completed days and **does not break on an unfinished today**. Functions:
`userPosture()`, `loadPosture()`, `togglePosture()`, `togglePostureOpen()`, `postureDoneCount()`,
`postureStreak()`, `postureThumb()`, `renderPosture()` (called at the top of `renderToday()`). State:
`allPosture`, `postureOpen`.

**Superseded:** the first version of this block (same day) was four flat exercises of my own selection
(pull-apart, face pull, overhead Y-raise, banded chin tuck). The protocol replaced it. The Y-raise and chin
tuck are gone; the protocol reaches the lower traps through the overhead finish on the face pull instead.

**Verified:** JS syntax clean (`node --check` on the extracted script blocks); posture logic unit-tested in
node against planted data (streak 3/4/2 across a gap, collapsed-when-done, 4 rows when expanded, 2/4
counter, hidden on past dates — all pass). Re-tested after the protocol rewrite: 3 stages, 6 items, 5 required, an optional-only tick counts 0, the streak survives the roller never being done, it still collapses at 5/5, the stages render in protocol order, 6 rows, habit footer present — all pass. Card states rendered in a real browser against the live stylesheet and screenshotted three times, the last to confirm the CDN thumbnails load and the "optional" badge does not wrap. **Not tested against real Firestore data or on a phone.**

**Service worker bumped `gym-v3` -> `gym-v4`** (both `sw.js` and the `?v=` in `index.html`) so phones do not
serve the stale build when this does ship.

**Firestore rules:** the existing `users/{uid}/{document=**}` wildcard already covers the new `posture`
collection, so no rule change is needed.

**Still outstanding from 17 Aug and before:** the app has never been verified signed in on an actual phone;
the arms-trained-1x/week gap in `WORKOUTS` was identified and deliberately not changed; Alex's private
thyroid + coeliac panel result is still worth checking back on. Note `generate_icon.py` is sitting untracked
in this folder, which breaks the standing "no build scripts in project folders" rule.

---

**2026-08-17 session:** Audited `WORKOUTS` against hypertrophy research (frequency, volume, rest periods,
exercise selection) at Alex's request — frequency/exercises/rest all check out against the literature; the
one gap found is arms trained 1x/week (3 sets each) vs 2x/week for every other muscle group. Recommended
adding a 2nd arm exercise into the other upper session but **made no code changes** — Alex didn't ask for
the edit, and this stays low-priority per the hardgainer brief's own sequencing (see below). Also added 4
new recipes to the Recipes tab live in Firestore via the app's own UI (not just seed data): 1000-Cal Whey
Smoothie, Chocolate Avocado Flaxseed, Chocolate PB Greek Yogurt, Coffee Whey — sourced from a Healthline
roundup, filtered to whey-only (no new protein powders to buy). Confirmed Google sign-in and a live
Firestore write both work from a desktop Chrome session — **still not verified on an actual phone.**

**State: LIVE.** Committed (`4ebf6a1`) and pushed to `main` on 2026-08-14 evening. GitHub Pages rebuilds
from `main`, so adakin97.github.io/gym-tracker is now running everything below rather than the 4 May
version. Firestore rules confirmed as a genuine `users/{uid}/{document=**}` wildcard, so `nutrition` and
`recipes` are already covered, no rule change was needed. Service worker cache bumped `gym-v2` → `gym-v3`
(both the cache name in `sw.js` and the `?v=` on the registration string in `index.html`) so phones don't
keep serving the stale cached build.

**Not yet done: verifying it signed in on an actual phone.** Everything so far was checked with stubbed
data in a browser preview harness, never against real Firestore data. Do this first next session. Two
things to expect there, not bugs: the maintenance-calibration card and the trend/rate will both say "not
enough data yet" until there are several real weigh-ins and logged food days.

**Two full sessions of work went into this, summarised below.**

**2026-08-13 (session 1): the Gain tab rewrite.** Trend weight (7-day mean), regression-based kg/week
rate, a verdict that converts the shortfall against the 0.6-0.8 target band into a calories-per-day
instruction, goal-date projection capped at 52 weeks, daily calorie/protein tracking, 16 one-tap food
presets, quick-add, 7-day intake strip. New Firestore collection `nutrition`.

**2026-08-14 (session 2): tailoring for a chronic hardgainer specifically**, on Alex's stated failure mode
("forgetting to eat enough") and no-push-notifications constraint. Five UI changes plus two new features:

1. **Fuel strip** — one line (`1,750 / 3,350 cal · 76/139g protein`), always visible above every tab, not
   just on Gain. The point: you already open this app to check the workout, for an unrelated reason, so
   the eating status rides along for free instead of hiding behind a tab switch. `renderFuelStrip()`,
   wired into the load sequence and every food mutation.
2. **Rest-day screen** now shows the real calories-still-needed number instead of "eat well, sleep well".
   Rest days have no post-workout hunger cue, which makes them the easiest day to under-eat without
   noticing.
3. **Workout-complete banner** gained a third stat: calories still owed today, alongside volume and
   duration. Post-workout is the single highest-attention moment the app gets, and hunger often drops
   right after training, which is the worst possible time to also forget the calories owed.
4. **"Repeat yesterday"** — a dashed one-tap button above the food grid, shown only when yesterday has
   entries, that clones them onto today (editable after). The common day for someone who forgets to eat
   is not exotic, it's the same shake and the same lunch most days; this cuts the typical day's logging
   cost from N taps to 1. `repeatYesterdayUI()`.
5. **Recipes tab** (5th nav item) — a saved, editable shortlist of go-to high-calorie meals/shakes with a
   "how you make it" note. Logging a recipe calls the same `addFood()` as the quick-add buttons, so a
   recipe and a logged food are indistinguishable once saved. Seeded with 3 starters on first load
   (`STARTER_RECIPES`) so the tab isn't empty; new Firestore collection `recipes`.
5. **Daily notes for the food-intolerance investigation.** A "How did today feel?" textarea saved onto the
   *same* nutrition doc as that day's food entries (`note` field added to `users/{uid}/nutrition/{date}`),
   deliberately, so food and symptoms sit together rather than in separate logs — that pairing is the
   whole point if the goal is spotting a pattern. Surfaced again in History: any day with food or a note
   now shows both, even rest days that would otherwise not appear.

**Verified:** JS syntax clean (`node --check`); gain maths tested against synthetic weight series with
planted rates (recovers 0.7 kg/wk under +/-1.2kg daily noise); all new UI (fuel strip, Gain tab, Recipes
tab, note field) rendered in-browser with stubbed data via a combined preview harness. **Not yet tested
against real Firestore data or on an actual phone.**

**Research brief completed 2026-08-14, saved to `../Hardgainer-Research-2026-08-14.md`.** Six sourced
findings (NEAT variability, training volume, surplus size, post-workout appetite suppression, self-report
unreliability, and when to see a GP). **Headline result: training design is very unlikely to be the
primary cause.** The evidence points at intake execution (which today's app changes already target) and
opens a genuine medical question (coeliac/thyroid/GI red flags match Alex's exact pattern: years of
apparently adequate effort with minimal result). **`WORKOUTS` left unchanged for now** — the brief
explicitly recommends against revising the training program before intake execution and the medical
question are addressed. **Maintenance calibration — built 2026-08-14.** `maintenanceCalibration()` back-calculates real maintenance
from actually-logged intake (10+ logged days in the last 28) against the real rate of gain from
`gainRate()`, using the same 7,700 kcal/kg constant already in the app. Rendered as a new card in the Gain
tab, between the hero and today's fuel: shows the real number, compares it to `FORMULA_TDEE` (2,682, the
original Mifflin-St Jeor estimate, kept only as a reference point not ground truth), and gives a
recommended target range for 0.6-0.8 kg/wk once ready. Gated on both enough logged days AND enough
weigh-ins independently — neither substitutes for the other. **Verified against a planted scenario** (true
maintenance 2,900, logged 3,350/day for 28 days): recovered maintenance within 40 kcal of the planted
value. Not-ready state shows a progress bar toward the 10-day threshold rather than nothing.

**Not built, and Alex has chosen not to build it:** the per-100g calorie-anchor estimator for unknown
home-cooked meals. He's using MyFitnessPal externally to look up a meal once, then it goes into the
Recipes tab as a one-tap preset from then on. Photo-based calorie estimation (AI vision) remains a real
future option but needs a Firebase Cloud Function as a backend to keep an API key server-side — real
infrastructure, not a UI change, so it's parked rather than started.

**Alex is getting bloods done privately** (thyroid + coeliac panel, ~£60-80 via Medichecks or similar) per
Section 6 of the research brief, rather than waiting on an NHS GP referral. Worth checking back on this.

**First things next session, in order:**

1. **Verify signed in on the phone** — the one item that never got checked before shipping (see the LIVE
   note at the top).
2. **Log real food and weigh-ins for a couple of weeks**, so the maintenance-calibration card and the
   trend/rate leave their "not enough data yet" states and start saying something real.
3. **Blood test outcome** — Alex is getting a private thyroid + coeliac panel done (~£60-80, Medichecks or
   similar) per Section 6 of the hardgainer research brief. Worth checking back on this; a real finding
   there would outrank everything else in the ranked list.
4. **Read back the hardgainer research** (`../Hardgainer-Research-2026-08-14.md`) once there's real data
   to weigh it against, and decide whether the training program (`WORKOUTS`) needs changing at all — the
   brief's own conclusion is that it's very unlikely to be the primary cause, so don't revise it
   speculatively before the intake and medical questions are further along.

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
users/{uid}/nutrition  — one doc per day keyed YYYY-MM-DD:
                         { date, kcal, protein, entries: [{ name, kcal, protein, t }], note }
                         entries are append-only; kcal/protein are recomputed from entries on every write
                         note (added 2026-08-14) is free text: how the day felt, for the food-intolerance log
users/{uid}/recipes    — saved go-to meals (added 2026-08-14): { name, kcal, protein, method }
users/{uid}/cardio     — cardio sessions keyed by date (YYYY-MM-DD): { date, type ('Swim'|'Run'), duration (mins) }
users/{uid}/weekPlans  — per-week schedule overrides keyed by mondayStr (YYYY-MM-DD)
                         each doc: { 0: null|wkName, 1: ..., ..., 6: ... }
                         wkName can be 'Upper A'/'Lower A'/'Upper B'/'Lower B'/'Swim'/'Run' or null (rest)
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

### Gain Tab (was "Weight", reframed 2026-08-13)

**The app's centre of gravity.** At 189cm and 63kg the binding constraint is not programming, it is eating
3,350 cal a day consistently. The tab is built around that, and the workout tracking is now the secondary
half of the app.

- **Trend weight** — trailing 7-day mean, shown large. A single weigh-in swings a kilo on water and gut
  content; the trend is the honest signal. Raw latest weigh-in shown small beneath it.
- **Rate of gain** — least-squares slope over the trend across the last 21 days, in kg/week. Regression
  rather than (last − first) so one odd weigh-in at either end cannot swing it. Verified against synthetic
  series: recovers a planted 0.7 kg/wk exactly, even with ±1.2kg daily noise.
- **Verdict + instruction** — compares the rate against the 0.6–0.8 kg/week target band and states what to
  do. Shortfall is converted to calories via 7,700 kcal/kg: gaining 0.3 kg/wk against a 0.6 target means
  eating ~330 more cal/day. This is the feedback loop the app previously lacked entirely.
  States: `good` / `slow` / `bad` (losing) / `fast` (above ceiling, cut calories) / `unknown` (<4 points).
- **Projection** — date of hitting 70kg at current rate. **Capped at 52 weeks**: a near-stalled trend
  projects an absurd date, which is discouraging noise, so past a year it says the rate is the thing to fix.
- **Today's fuel** — calories and protein against 3,350 / 139g, with "cal left" and hours remaining
  (assumes last food ~22:00).
- **One-tap food buttons** — 16 presets from the nutrition plan plus calorie-dense staples. Deliberately
  taps, not typing: the `Meal_Planner.xlsx` weekly tracker was built and never filled in once, which is the
  evidence that a typed food log does not survive contact with real life. Plus a quick-add for cal/protein.
- **Last 7 days strip** — bar per day, green at ≥95% of target, amber partial, empty for nothing logged.
- **Daily note ("How did today feel?")** — added 2026-08-14 for the food-intolerance investigation. Saved
  as `note` on the same `nutrition/{date}` doc as that day's food entries, deliberately, so food and
  symptoms are reviewed together rather than in separate logs. `saveDayNoteUI()`.
- Weight logging, goal progress, chart and recent entries all retained below.

Constants: `KCAL_TARGET 3350`, `PROTEIN_TARGET 139`, `RATE_MIN 0.6`, `RATE_MAX 0.8`, `KCAL_PER_KG 7700`.
Key functions: `trendSeries()`, `gainRate()`, `gainVerdict()`, `projectGoalDate()`, `addFood()`,
`removeFoodEntry()`, `quickAddUI()`, `saveDayNoteUI()`, `renderGain()` (was `renderWeight`).

### Fuel strip (added 2026-08-14)
One line, always visible above every tab (not just Gain): `1,750 / 3,350 cal · 76/139g protein`. The
reasoning: the app is opened to check the workout, for an unrelated reason, so the eating status should
ride along rather than hide behind a tab switch. Tapping it jumps to the Gain tab. `renderFuelStrip()`,
called on load and after every food mutation (`addFood`, `removeFoodEntry`).

### History Tab
- Grouped log entries by date → exercise → sets
- Delete entire session via × button on each session card
- **Food + daily note, added 2026-08-14** — any date with logged food or a saved note now shows a card
  with that day's calories/protein, the food list, and the note in italics underneath, even on rest days
  that would otherwise not appear in History at all. This is what makes the intolerance pattern findable:
  scanning several days of food-plus-symptoms in a row, not just today's.

### Recipes Tab (new, added 2026-08-14)
5th nav item. A saved, editable shortlist of go-to high-calorie meals — shakes, easy cooked meals — each
with a "how you make it" note. **Logging a recipe calls the same `addFood()` as the quick-add food
buttons**, so a recipe and a logged food are indistinguishable once saved; the tab is really just a
personal, reusable extension of the Gain-tab food presets. Seeded with 3 starters
(`STARTER_RECIPES`: mass gainer shake, loaded oats, rice/chicken/oil) on first load if the user has never
saved anything, so the tab is useful immediately rather than empty. New Firestore collection `recipes`.
Functions: `loadRecipes()`, `addRecipeUI()`, `deleteRecipe()`, `logRecipe()`, `renderRecipes()`.

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

**Current exercise lists (5 per session as of 2026-05-04):**

| Upper A (Strength) | Upper B (Hypertrophy) |
|---|---|
| Barbell Bench Press 4×6-8 (120s) | Incline Dumbbell Press 4×10-12 (90s) |
| T-Bar Rows 4×6-8 (120s) | Lat Pulldown 4×10-12 (90s) |
| Dumbbell Shoulder Press 3×6-8 (90s) | Dumbbell Shoulder Press 3×10-12 (90s) |
| Pull-ups 3×6-8 (90s) | Face Pulls 3×15-20 (60s) |
| Dumbbell Curl 3×8-10 (60s) | Cable Tricep Pushdowns 3×12-15 (60s) |

| Lower A (Strength) | Lower B (Hypertrophy) |
|---|---|
| Barbell Squat 4×6-8 (120s) | Deadlift 4×6-8 (120s) |
| Machine Deadlift 4×6-8 (120s) | Bulgarian Split Squats 3×10-12 (90s) |
| Static Lunges 3×10-12 (60s) | Leg Curls 3×12-15 (60s) |
| Calf Raises 4×12-15 (45s) | Leg Press 3×15-20 (90s) |
| Plank 3×45s (45s) | Calf Raises 4×15-20 (45s) |

Rest values in brackets are the configured rest timer durations.

### Exercise Library

`EXERCISE_LIBRARY` array contains ~94 curated exercises from free-exercise-db, merged with workout exercises into `ALL_EXERCISES` (deduped). Used for the swap overlay.

### Cardio Sessions

`CARDIO_TYPES = ['Swim', 'Run']`. When a day is planned as Swim or Run (via replan sheet or badge picker), the Today tab renders a simple duration-input card instead of exercise cards. Duration logged in minutes to Firestore `users/{uid}/cardio/{date}`. Cardio sessions shown in History tab and date strip.

### Bodyweight Exercises

Exercises in `['Plank', 'Hanging Leg Raises', 'Dead Bug']` are flagged as `noWeight`: weight input is hidden (sends 0), previous best / overload nudge hidden. Plank uses "mins" label instead of "reps". Note: Hanging Leg Raises and Dead Bug were removed from Lower B in session 22 (May 2026); only Plank remains in active workouts.

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
| 2026-08-18 | **Nutrition paused behind `NUTRITION_ENABLED = false`** (Alex moved to MyFitnessPal): fuel strip, Gain-tab food cards, Recipes tab, rest-day and complete-banner calorie stats, and History food/note cards all gated off rather than deleted; recipe seeding skipped so nothing writes to Firestore while paused; no Firestore data removed. **Added the morning posture block, built to the AthleanX rounded-shoulders protocol Alex supplied**: three ordered stages (mobilise the thoracic spine, stretch pec minor/subscapularis, strengthen mid-back and lower traps), six items of which the foam-roller drill is optional and excluded from the 5/5 target so a missing roller cannot kill the streak, plus the tennis-ball desk habit as a footer note. Kept out of `WORKOUTS` so it never enters training volume. Tick-off, collapse-when-done, streak that tolerates an unfinished today. An earlier four-exercise version from the same session was superseded by the protocol. New collection `users/{uid}/posture/{date}`. Each row carries a 54px thumbnail from free-exercise-db where an honest match exists (`Rhomboids-SMR`, `Face_Pull`, `Band_Pull_Apart`, 3 of 6); the prone roll-back, doorway slide and pitcher's stretch have no match in the database and render a dashed placeholder, which is also the `onerror` fallback. Service worker bumped `gym-v3` → `gym-v4`. Logic unit-tested; card states screenshotted in-browser including the loaded thumbnails. **Not committed or pushed.** |
| 2026-08-14 | **Shipped.** Committed `4ebf6a1` and pushed to `main`; live at adakin97.github.io/gym-tracker. Confirmed the Firestore rules are a genuine `users/{uid}/{document=**}` wildcard (Alex pasted them from the console), so `nutrition` and `recipes` needed no rule change. Service worker cache bumped `gym-v2` → `gym-v3` in both `sw.js` and the `index.html` registration string, so phones stop serving the stale cached build. This is the two-session Gain-tab/hardgainer rebuild described above going live for the first time. |
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
| 2026-08-13 | **Reframed from workout tracker to weight-gain tracker.** Weight tab became the Gain tab and is now the app's centre of gravity, on the reasoning that at 189cm/63kg the constraint is eating, not training, and the app tracked training exhaustively while doing nothing about food. Added: 7-day trend weight, regression-based kg/week rate, a verdict that converts the shortfall against the 0.6–0.8 target band into a calories-per-day instruction (7,700 kcal/kg), goal-date projection capped at 52 weeks, daily calorie/protein tracking against 3,350/139g, 16 one-tap food presets, quick-add, and a 7-day intake strip. New Firestore collection `nutrition`. Maths verified against synthetic series with planted gain rates (recovers 0.7 kg/wk under ±1.2kg daily noise); view verified in-browser with stubbed data. **Not yet committed or pushed.** |
| 2026-05-04 | Service worker cache bumped to `gym-v2`; registration changed to `sw.js?v=2` to force browser re-fetch of cached sw.js on next visit. |
| 2026-05-04 | Rest timers updated per exercise type: 120s compound strength (Squat, Deadlift, Bench, T-Bar Row), 90s compound hypertrophy (Incline Press, Lat Pulldown, DB Shoulder Press, Bulgarian Split Squats, Leg Press), 60s isolation, 45s core/calves. All sessions reduced to 5 exercises: Upper A removed Cable Tricep Pushdowns; Lower A removed Leg Press; Upper B removed Dumbbell Flyes and Hammer Curls; Lower B removed Dead Bug and Hanging Leg Raises. Cardio logging added: Swim and Run are selectable session types in the replan sheet and badge picker. Cardio view in Today tab shows a duration input; logs saved to Firestore users/{uid}/cardio/{date}. Cardio sessions shown in History tab and date strip. Weekly plan bug fix: badge picker changes now persist to Firestore weekPlan immediately (selectWorkout and new selectCardio are async); past-date strip uses logged workout name not scheduled plan; getWeekPlan null-handling made robust with ?? instead of falsy chaining. |

---

## Pending / Ideas

**Decided against push notifications (2026-08-14):** Alex wants passive, fast logging, not nagging. The
fuel strip, rest-day note and complete-banner stat exist specifically as the alternative — surface the
number at moments the app is already opened, rather than interrupt.

**Next, in priority order:**
- **Read the hardgainer research brief (opened 2026-08-14)** and decide what changes to the `WORKOUTS`
  object itself — training volume, frequency, exercise selection — not just the tracking UI. This may be
  the highest-value item once it lands; the tracking fixes forgetting to eat, but a program mismatch is a
  separate failure mode entirely.
- **Trend line on the weight chart** — currently plots raw weigh-ins only; the 7-day trend and the target
  trajectory from 63 to 70 should both be drawn on it.
- **"Repeat yesterday" one-tap food log** — agreed 2026-08-14, not yet built. Clone yesterday's entries in
  one tap, editable after. Reduces the common day's logging cost from N taps to 1, which matters directly
  for the "forgetting to eat" failure mode.
- Confirm the Firestore rules cover `nutrition` and `recipes` (likely already, if `users/{uid}/**` is a
  wildcard — see the "Pick up here" block above).


- Personal records (PR) tracking — highlight new max weight on set log
- Workout streak counter — consecutive weeks hitting all 4 workouts
- Notes per exercise — form cues / session feel
- Volume trend in graphs — total volume per session as second chart
- Nutrition tracker tab
- Export logs as CSV
