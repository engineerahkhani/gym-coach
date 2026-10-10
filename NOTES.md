# Pocket Coach (مربی جیبی) — Project Notes

Context file for continuing work in a new chat. Attach `~/Developer/gym-coach` and point Claude here.

## What this is
Single-file offline PWA (`index.html`, ~430 KB, fonts embedded) for one person's gym program. Persian, RTL, built for iPhone X width (375 px). Data lives only in the browser's localStorage (key `pocketcoach_v1`); backup/restore via JSON copy in Settings.

- Live: https://engineerahkhani.github.io/gym-coach/
- Repo: git@github.com:engineerahkhani/gym-coach.git
- Deploy: push to `main` → GitHub Pages. Bump `CACHE` in `sw.js` (`pocketcoach-vN`) on every release or phones keep the old version. On the phone, close and reopen the app twice after a deploy.
- Claude's cloud session cannot push (no network/SSH from the device bridge). Claude writes files + commits; the user runs `git push`. If git leaves lock files: `rm -f .git/index.lock .git/HEAD.lock .git/objects/maintenance.lock`.
- No Persian comments in code. Commit messages in English.

## Person (as of 2026-10-07)
- Male, 178 cm, 85.5 kg, waist 92, abdomen (navel) 100. BMI ~27, abd/height 0.56 (target < 0.5 → ~89 cm).
- Goals in order: smaller belly, strength/agility, muscle/shape, consistency. Worried about face getting gaunt → slow loss (0.4–0.6 kg/week).
- No injuries, no meds, no supplements. Gets migraines when under-eating / meals too far apart.
- Bikes to the gym: downhill there, 20–30 min uphill back (or walks). Gym at ~14:00. Sleeps ~00–08.
- Procrastination is the main risk, not fitness. Short 20-min session exists for low-motivation days.
- Home cooking by spouse; rice at lunch, bread at dinner; oil/butter heavy; 1 egg/day; water ~2 glasses/day (too low).

## Training program (v11)
3 gym sessions/week (rotating A → B → C, app picks next) + 1 swim/tennis + rest. Uphill bike ride logged as "رکاب تا باشگاه".

- **A**: barbell squat 3×8–10, bench press 3×8–10, barbell row 3×8–12, plank 3×30–45s, farmer walk 2×30–40s. Finisher: dead hang 2×20–40s, hanging knee raise 2×8–12.
- **B**: dumbbell RDL 3×8–12, dumbbell OHP 3×8–12, lat pulldown 3×8–12, lunge 2×8–10/leg, dead bug 3×8–10. Finisher: hanging leg raise 2×6–10, crunch 2×12–20.
- **C**: hip thrust 3×10–12, incline DB press 3×8–12, seated cable row 3×10–12, side plank 3×20–30s/side, DB curl 2×10–12, lying triceps ext 2×10–12. Finisher: chin-up negatives 2×3–5, dead hang 2×.
- Warm-up (4-item checklist inside the session): 5 min bike/treadmill, 10 bodyweight squats, shoulder circles + hip hinges, one light set of first exercise.
- Short version: first 3 exercises, 2 sets, no finisher.
- Leg volume was reduced (lunge 3→2, goblet squat removed, bike interval removed) because of the uphill ride home. Squat/RDL/hip thrust kept.

## Progression rules (in `suggest()`)
- Each exercise has a rep range. All sets at top of range → add `inc` kg (2.5 DB / 5 barbell & machines). Two sets below bottom → subtract.
- RIR ("در توان", 0/1/2/3+) per set: all 3+ → add weight even if reps not at top; two sets at 0 and reps low → reduce.
- Deload weeks 4, 8, 12 (counted from first session date): suggestions ×0.9, rounded to 2.5. Banner on Today + Live.
- Bodyweight/timed exercises progress by reps/seconds (`inc`).
- Warm-up sets auto-suggested at 50%×8 and 75%×4 of working weight.
- Plate calculator for barbell exercises (bar 20, lying triceps bar 10). Plates: 20/15/10/5/2.5/1.25.

## Food log (v13)
Tap-to-log with household units, no calorie counting in the UI. `FOODS` catalog (~45 Persian home foods: name, unit, category, step 0.5 for rice/tuna/baguette, hidden rough kcal/protein per unit used only in export). Tapping an item adds one unit at the current time; same item tapped again within 10 min merges into the previous entry. Category chips (default "پرتکرار" = top 12 used in last 30 days, padded from `FOOD_DEFAULT`). Free-text entries still possible. Stored in `nutrition[date].meals[{t,id,q}|{t,txt}]`; legacy `log[]` is migrated into `meals` in `load()`. Checklist kept but collapsed under a `<details>`. `foodDate` (null = today) selects the day being edited: day arrows in the food header, the week strip, and the 6-week calendar in History are all `data-fday` buttons (past and future allowed); entries added to another day get time 12:00.
Monthly export (food tab, bottom card): Jalali month picker → CSV (UTF-8 BOM, one row per entry: jalali/gregorian date, weekday, time, meal slot from time, food, qty, unit, category, ~kcal, ~protein, gym session/activity that day, weight if measured, flags, checklist ✓/✗). `foodExport()` uses Web Share with a File on iOS, download fallback, or clipboard copy. Send the file to Claude at month end for review.

## Design (v13): Liquid Glass
Translucent layered surfaces over a soft colored backdrop (3 radial blobs on `body`). `.card`, `nav.tabs`, `.toast`, header capsules use `backdrop-filter` (`--blur`) + inset top highlight (`--hl`) + a gradient specular rim (`.card::before` with mask-composite). Header and tab bar are `position:fixed` and float over content (`main` has top/bottom padding for them); the header has a progressive blur via `mask-image`. Controls are capsules; inputs/steppers/segments sit in `--well` with `--lined` borders. `prefers-reduced-transparency` falls back to `--solid`. Tokens: `--surface`/`--surface2` are now rgba, so inline styles that reference them stay translucent automatically.

## Nutrition (simple rules, no counting)
13-item daily checklist, tri-state (✓ / ✗ / blank), "good day" = 9+. Day flags: migraine, severe hunger, ate out. Free-text food log per day for later review. History tab shows 28-day adherence % per item (lowest first) and migraine vs adherence averages.

Core four: protein every meal (2 eggs breakfast, palm-size lunch/dinner); eat protein+veg first, rice/bread last, stop at "not hungry" (lunch ≤ 1 heaped spoon rice, dinner ½ baguette); halve oil/butter; water 2–2.5 L. Meals ≤ 4 h apart, mandatory pre-gym snack 12:30 (banana/2 dates + yogurt or milk), main lunch after gym ~15:30–16:00. Keep 2 dates/day (migraine prevention). Keep tea count constant. Carb reduction stepwise: week 1 only plate order + protein; from week 2 rice to one spoon.

Targets: 0.4–0.6 kg/week; weigh weekly (same day, fasted), waist/abdomen every 2 weeks (Today tab shows a due-date card). Two weeks no change → half a spoon less rice; faster than 0.8/week → add a bread or fruit. Migraine ↑ → add protein snack, restore rice.

## App structure (index.html)
- CSS tokens at top; dark-first, light via `data-theme`; fonts Vazirmatn (default), Sahel, Shabnam embedded as base64 @font-face; switch in Settings. Theme setting dark/light/system.
- State `S`: `profile{height,age}`, `settings{sound,wake,restScale,font,theme}`, `measures[]`, `sessions[]` (each: date, w, short, min, vol, ex[{id,rating,sets[{w,r,rir,done}]}], note), `activities[]` ({date,type: swim|tennis|ride|rest}), `nutrition{date:{itemKey:true|false|null, migraine, hungry, cheat, log[{t,txt}]}}`, `live` (in-progress workout, survives reload).
- Tabs: today, lib (exercise library + detail), food, body (measures + chart), hist (calendar, muscle sets/week, PRs + e1RM, nutrition adherence, food log review, sessions), set.
- Animation: `POSES[id]=[poseA,poseB]`, forward-kinematics stick→human renderer (`fk`, `drawPose`), `CAPS` captions per phase. Props: bar, db, kb, barhip, cable, cableup, barup, bike; benches 1/2/3.
- Live workout: stepper rows (−/+ weight and reps, prefilled from suggestion/last session, weight copies down from set 1), RIR chips after a set is logged, PR toast and badge, rest timer (ring + beep + ±15 s), hold timer for timed exercises, warm-up checklist on first exercise, "machine busy" swap to `alt`, per-exercise satisfaction rating 1–5.
- Render keeps scroll position unless the view changed. `touch-action: manipulation` everywhere; all inputs ≥ 16 px to stop iOS zoom.
- Service worker: cache-first with background refresh.

## Version log
v1 stick figures · v2 human figures, Vazirmatn, icons, ratings, bike commute · v3 plate calc, warm-ups, PR/volume, RIR, muscle sets, deload · v4 font switcher · v5 theme · v6 stepper set rows · v7 nutrition tab · v8 tri-state checklist, flags, adherence history · v9 food log · v10 keep scroll · v11 no double-tap zoom, hold timer, finishers, warm-up in session, Today reorder · v12 no input zoom, measurement reminder, weight-only entries · v13 tap-to-log food with household units + monthly CSV export, Liquid Glass restyle · v14 edit any day's food from the calendar / week strip.

## Ideas not done (deliberately)
Calendar reminders (.ics), progress photos, Apple Watch, social features, 12-week block periodization beyond deload weeks, Peyda font (couldn't fetch).
