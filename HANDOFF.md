# Handoff: Kids Token Time

## What this is
A single-file web app (`index.html`, no build step, no dependencies except Google Fonts) that tracks a family token system for two kids, Matilda (8) and Brooks (6). The kids earn tokens for jobs and spend them on treats and weekend iPad/YouTube time. The system started Mon Nov 16, 2026. The app was built and published as a claude.ai artifact; this handoff is for moving it to a real repo and hosting it.

## Rules the app enforces
- Routine jobs are 1 token. Other jobs are 2 tokens. Counter jobs can be done several times a day.
- Treat: 5 tokens, max 1 per day, any day.
- iPad or YouTube: 10 tokens per hour, Saturday and Sunday only.
- Tokens do not carry over. The week starts Monday and the app keys all data by the Monday date, so it resets itself. Past weeks stay viewable and editable through the week bar and "Looking back".
- Spending is blocked when the weekly balance is too low. Un-checking a job is blocked if it would push the balance below 0 (shows a toast).

## Jobs (see `JOBS` array at the top of the script)
Fields: `id`, `g` (group: morning, day, night, bonus), `e` (emoji picture), `t` (title), `s` (optional subtitle), `v` (token value), `count` (true = stepper), `only` (kid id) and `days` (0=Mon to 6=Sun).

- Morning (1 each): stay in room until clock is green, get dressed, brush hair, brush teeth (morning), make bed, pack backpack (moved from During the day on Nov 17, 2026).
- During the day (1 each): unpack backpack, no fighting, dishes to sink (counter since Nov 16, 2026, 1 per trip; old `true`/`false` values are migrated to 1/0 in `fix()`), clothes in laundry basket, hang up jacket, clean up after each activity (counter, 1 per activity), flush the toilet (counter, 1 per flush, added Nov 22, 2026), hockey prep (Matilda only, Sat and Sun: water bottle, equipment in bag, stick).
- Brooks only, school days (Mon to Fri): have a good second break at school = 3 tokens (id `break`, added Nov 22, 2026).
- Bedtime (1 each): brush teeth (night), go to bed nicely.
- Bonus (2 each): garbage bins to curb and back, vacuum, try a new food, help someone else (counter, 2 each time).
- Special: Brooks only, Fridays: no call from the principal and no school incidents = 5 tokens (id `school`).
- "From Mom and Dad": free-form extra tokens per day (+1, +5, -1), stored as `extra`.

## Data model
```
store = { "matilda-2026-11-16": { days: { "0": { jobs:{id:true|number}, treat:bool, hours:number, extra:number }, ... "6": {...} } },
          "brooks-2026-11-16":  { same }, ... one entry per kid per week }
state = { matilda: store["matilda-<viewed week>"], brooks: store["brooks-<viewed week>"] }
```
- Week history (added Nov 22, 2026): `store` holds every week, keyed `<kid>-<Monday date>` (same ids as localStorage `tt-…` and the Firestore docs). `curWeekKey` is this week; `weekKey` is the week on screen, changed by `setWeek()`; `bindWeek()` points `state` at it. The week bar (‹ › and "This week") moves between weeks; › and "This week" are disabled on the current week, and ‹ stops at `FIRST_WEEK` (2026-11-16) or the oldest week with entries. Past weeks are fully editable. "Looking back" lists up to 12 weeks newest first (weeks with entries plus the current and viewed week) with each kid's earned, spent and left; tapping one opens it. The "today" dot and the jar's next-goal line only show on the current week. The TV scoreboard always shows the current week.
- Day index 0 to 6, Monday to Sunday. `fix()` normalizes missing fields; `peek()` reads without creating days; `dd()` reads and creates.
- Totals: `earnedDay`, `spentDay`, `totals(kid, weekData?)` (earned, spent, left); the optional week data defaults to the viewed week.
- Persistence: every week is mirrored to `localStorage` under `tt-<kid>-<weekStartISO>`, plus `tt-kid`, `tt-view`, `tt-tv` and `tt-family`. All `tt-<kid>-<date>` keys are loaded into `store` on start.
- Sync (added Nov 16, 2026, replacing the claude.ai `db` sync): Firebase Firestore, docs `families/<family code>/weeks/<kid>-<weekStartISO>`. Since Nov 22, 2026 one `onSnapshot` listens to the whole `weeks` collection (rules allow `list`), so past weeks and Looking back stay live. `save()` diffs the viewed week against its last synced copy and sends only changed leaf fields with `updateDoc` (FieldPath segments), so two people tapping different jobs don't clobber each other; a week with no doc yet is created with `setDoc` merge. After the first server answer, weeks that exist only in this device's localStorage are uploaded once. Offline cache is IndexedDB via `persistentLocalCache`.
- The family code is a 32-character random secret, read from `#family=` in the URL or pasted via the sync label, and stored in `localStorage`. It is not in the repo; the full link is in the git-ignored `family-link.txt`. `firestore.rules` only allows get/list/create/update under a code of 20+ characters, with kid-week doc names and a `days` map. No deletes.
- The manifest has no `start_url` on purpose, so an iPhone saved to the home screen from the family link keeps the code.
- The app reloads when it returns to the foreground on a new day, so the week resets even if it was never closed.

## Known issues and decisions
- Firebase project `token-time-f2bfe` (owner's Google account, Spark free plan, Firestore Standard edition, `(default)` database in `nam5`) was set up Nov 16, 2026 and `firestore.rules` is published. Tested with two browser windows on a throwaway family code: taps appeared in the other window within about a second without refreshing, near-simultaneous taps on different jobs were both kept, and the rules denied short codes, other collections, listing families, bad doc names, extra fields and deletes. Not yet tested on real iPhones/iPads.
- If `firestore.rules` changes, paste it into Firebase console → Firestore → Rules and publish (no CLI is set up).
- Monday Nov 16, 2026 entries from the claude.ai version were imported into the real family's Firestore docs (Matilda and Brooks each: earned 15, spent 5 on a treat, 10 left). The earlier `2026-11-09` test week was not imported.
- A test doc set exists under `families/synctest-throwaway-0001-abcdefgh` (emptied). It can be deleted from the console.
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
1. Done: hosted at https://sclewismb.github.io/token-time/ from github.com/sclewismb/token-time.
2. Done: Firebase sync is live.
3. Open the family link on each iPhone/iPad in Safari, then Add to Home Screen, and check a tap shows up on another device.
4. Optional: parent PIN for the "From Mom and Dad" section, a weekly history view, and sounds or an animation when a treat or iPad hour is bought.
