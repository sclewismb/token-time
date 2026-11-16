# Handoff: Kids Token Time

## What this is
A single-file web app (`index.html`, no build step, no dependencies except Google Fonts) that tracks a family token system for two kids, Matilda (8) and Brooks (6). The kids earn tokens for jobs and spend them on treats and weekend iPad/YouTube time. The system started Mon Nov 16, 2026. The app was built and published as a claude.ai artifact; this handoff is for moving it to a real repo and hosting it.

## Rules the app enforces
- Routine jobs are 1 token. Other jobs are 2 tokens. Counter jobs can be done several times a day.
- Treat: 5 tokens, max 1 per day, any day.
- iPad or YouTube: 10 tokens per hour, Saturday and Sunday only.
- Tokens do not carry over. The week starts Monday and the app keys all data by the Monday date, so it resets itself.
- Spending is blocked when the weekly balance is too low. Un-checking a job is blocked if it would push the balance below 0 (shows a toast).

## Jobs (see `JOBS` array at the top of the script)
Fields: `id`, `g` (group: morning, day, night, bonus), `e` (emoji picture), `t` (title), `s` (optional subtitle), `v` (token value), `count` (true = stepper), `only` (kid id) and `days` (0=Mon to 6=Sun).

- Morning (1 each): stay in room until clock is green, get dressed, brush hair, brush teeth (morning), make bed.
- During the day (1 each): pack backpack, unpack backpack, no fighting, dishes to sink, clothes in laundry basket, hang up jacket, clean up after each activity (counter, 1 per activity), hockey prep (Matilda only, Sat and Sun: water bottle, equipment in bag, stick).
- Bedtime (1 each): brush teeth (night), go to bed nicely.
- Bonus (2 each): garbage bins to curb and back, vacuum, try a new food, help someone else (counter, 2 each time).
- Special: Brooks only, Fridays: no call from the principal and no school incidents = 5 tokens (id `school`).
- "From Mom and Dad": free-form extra tokens per day (+1, +5, -1), stored as `extra`.

## Data model
```
state = { matilda: { days: { "0": { jobs:{id:true|number}, treat:bool, hours:number, extra:number }, ... "6": {...} } },
          brooks:  { same } }
```
- Day index 0 to 6, Monday to Sunday. `fix()` normalizes missing fields; `peek()` reads without creating days; `dd()` reads and creates.
- Totals: `earnedDay`, `spentDay`, `totals(kid)` (earned, spent, left).
- Persistence: always mirrored to `localStorage` under `tt-<kid>-<weekStartISO>` and `tt-kid`.
- On claude.ai the page also syncs through the artifact `db` capability: doc path `tokens/<kid>-<weekStartISO>`, `set()` on every change (queued, one write at a time) and `onSnapshot` to pull changes. Snapshots are ignored for 3 seconds after a local write and while writes are pending, to stop stale echoes overwriting new taps.

## Known issues and decisions
- Outside claude.ai there is no `window.claude`, so `connect()` does nothing and data is per-browser only. Two iPads will not sync. Fixing that needs a small backend or a hosted DB (for example Firebase, Supabase or a tiny serverless function); the `save()` and `connect()` functions are the only places to change.
- An early bug (only the first tap responded) was most likely the db returning day objects without `jobs`, which made render throw. Fixed by `fix()` and `peek()`, plus the snapshot guard. It was not reproduced or confirmed on a device, so test taps across several jobs after any change.
- The "From Mom and Dad" section has no parent lock. The kids can tap it. A simple PIN gate would fix that if wanted.
- Manifest and icons added Nov 16, 2026 (`manifest.webmanifest`, `icons/`: coin-with-star on the Matilda purple, 32/180/192/512 px, the 512 doubles as maskable). Not yet tested on a real iPad home screen.
- Tapping five jobs in a row was checked in a desktop browser on Nov 16, 2026 and every tap counted.
- Single light theme by design (bright, kid-oriented). No dark mode.
- Emoji are used as job pictures on purpose, so non-readers can recognize jobs.

## Related files outside the code
- Spreadsheet tracker (claude.ai Sheets artifact "Kids Token Tracker", one tab each for Matilda and Brooks) has the same jobs and totals but is separate and does not sync with the app. It has no "From Mom and Dad" row.
- Source of truth for the rules, in plain language: the project doc `token-jobs.md` in the claude.ai Project "Kids tokens".

## Suggested next steps
1. Push the local git repo to GitHub and turn on Pages (steps in README.md).
2. Add to the iPad home screen from Safari and confirm the icon and full-screen launch.
3. Decide whether cross-device sync matters; if so, add a small backend and replace `save()` and `connect()`.
4. Optional: parent PIN for the "From Mom and Dad" section, a weekly history view, and sounds or an animation when a treat or iPad hour is bought.
