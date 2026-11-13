# Coach starter kit: setup instructions for Claude

You are setting up a personal endurance coach for the person you're talking to (the **athlete**). This kit came from another athlete who uses the same system. Nothing in it is about your athlete, and none of that athlete's data is here or should be looked for. Build everything new in **your athlete's own account**.

The system has four parts. Build them in this order.

1. **Intake.** Learn who the athlete is.
2. **Training plan:** a Claude doc.
3. **Coach app:** a Claude artifact with its own database, built from `coach-app.html` in this folder.
4. **Weekly update:** a scheduled task every Sunday evening that reviews the week, adjusts the plan, loads workouts onto the watch, updates the app and sends a report.

Tell the athlete in one line what you're about to do, then start the intake.

---

## 1. Intake

Ask in two or three short rounds, not one long form. Use multiple-choice questions where your interface offers them.

- **About them:** first name, age, weight, home city and its altitude, job or study schedule (busy weeks, exams, travel).
- **Sport and history:** runner, cyclist or triathlete; years training; recent races and times; current weekly volume; longest recent run or ride; injuries in the last year.
- **Goals:** target races with dates and goal times (A/B/C if they like), plus any non-race goal such as FTP.
- **Availability:** days per week, time per day, which days can hold long sessions, and gym access and habits.
- **Equipment:** watch brand and model; chest strap or wrist HR; bike power meter or smart trainer; Zwift or similar.
- **How they want it:** how strict, and preferences such as workout length or an easy Monday.

**Data from the watch.** If they use **Garmin** and Claude can run tools on their computer (the Claude desktop app linked to this conversation), offer to connect Garmin.
- Add a local Garmin Connect MCP server to their Claude desktop app. Several open-source ones exist. Pick a maintained one that can list activities, read splits, HR zones, HRV and sleep, and create and schedule workouts.
- Walk them through installing it. **They** sign in to Garmin themselves; never ask for or type their password.
- Then pull their last 3–6 months of activities to see their real volume, intensity distribution (time in HR zones), long runs and race results.

Without Garmin, plan from what they tell you and from their app check-ins.

## 2. The training plan (Claude doc)

Write their plan as a Claude doc titled "<Name>'s Training Plan: <main goal>". Use the evidence-based structure below and fit every number to them.

- **How the plan is built:** phases toward each race (base → build → specific → taper) with dates.
- **Training zones:** heart-rate zones from their threshold (LTHR) or max HR, marked as estimates until tested. Plan a 30-minute run time trial to set LTHR and an FTP test if they ride. Altitude note if they live high.
- **Intensity discipline:** about 80% of time easy, with a hard ceiling on easy days (e.g. easy HR at most about 84% of LTHR). Quality sessions have a pace band first and an HR ceiling second. Most amateurs run their easy days too hard; check their data for this and say so if you see it.
- **Volume rules:** increase weekly volume 5–10% at a time, with a down week every 3–4 weeks; no single run more than 10% longer than their longest in the last 30 days; never raise volume and intensity in the same week; at most 3 hard days a week, never 3 in a row.
- **Week-by-week table** for the current block: km/hours, key session, long session.
- **Strength:** 1–2 sessions a week (heavy compound lifts plus plyometrics, built up gradually), placed after hard days, not before.
- **Fuelling** for long sessions, and race-day pacing.
- **Safety:** red flags (pain that changes the stride or lasts 3+ days, bone pain, chest pain, fever) mean stop and see a professional. Illness rules. How to return after missed time.
- **Checkpoints:** test or race dates with "on track" and "re-target" numbers.

Show them a summary and adjust it before moving on.

## 3. The Coach app (Claude artifact)

1. Download the template's exact text with your shell from https://raw.githubusercontent.com/fonsi43/fonsi-coach/main/starter/coach-app.html and save it as their file, e.g. `coach.html`. Use the raw file: a page summary won't work. Change nothing except the `<title>` (e.g. "<Name> Coach").
2. Publish it as an artifact with capabilities `{"db": {}, "user": {}}`.
3. Seed its database with the ArtifactData tool, writing the documents below in one batch. **The app reads these exact shapes:**
   - `coach/profile`: `{name, races:[{name, date "YYYY-MM-DD", goal}], pastRaces:[{date, label}]}`. Races are upcoming, soonest first; the first one gets the big countdown card. pastRaces mark the fitness chart.
   - `coach/plan`: the current week, `{weekStart (Monday, YYYY-MM-DD), phase, plannedKm, days:[{date, items:[{id, sport run|bike|strength|test|swim, title, target, status planned|done|partial|missed|swapped|optional, result}]}]}`.
     - Write all 7 days, with `items: []` on rest days.
     - `id` like "w1116a".
     - Put minutes in the title with an apostrophe, e.g. "Easy 40' + strides".
     - Write `target` as structure steps separated by " · ", e.g. "WU 10' + 25' HR 140–152 · 6 strides · CD 5'". The app draws the workout-shape bar from that.
   - `coach/weekly`: `{rows:[{week (Monday), phase, plannedKm, km, runs, longRun, bikeH, swimKm, strength, easyPct, easyHR, ef, note}], updated, note}`.
     - Past weeks come from their watch history; future weeks have only plannedKm.
     - easyPct is the % of run time at easy HR.
     - ef = speed in m/min divided by average HR, on outdoor easy runs.
   - `coach/pmc`: `{updated, method, days:[{d, load, ctl, atl, tsb}]}`, one entry per day of history.
     - load = that day's training load. Rides with power: duration_s × NP × IF / (FTP × 3600) × 100. Runs: hours in HR zones Z1–Z5 × 30/50/70/90/115. Strength: 30 per hour. Anything else: 45 per hour.
     - ctl += (load − ctl) × (1 − e^(−1/42)).
     - atl += (load − atl) × (1 − e^(−1/7)).
     - tsb = yesterday's ctl − atl.
   - `coach/summary`: `{updated, focus (one sentence), intensityScore 0–10 or null, intensityNote}`.
   - `coach/goals`: `{races:[{name, date, target, sub}], checkpoints:[{when, test, onTrack, retarget, result}], paces:[{effort, hr, pace, use}], ftp:[{date, test, ftp, wkg, note}], shoes:[{shoe, use, km, note}]}`.
   - `coach/report`: `{date, title, headline, happened[], changes[], todo[]}`.
   - The athlete writes `checkins/<date>` (`{date, sleep, legs, energy (1–5), pain, painWhere, note, savedAt, workouts:{<item id>:{done, fit plan|short|over, rpe, at}}}`) and `requests/<auto>` (`{workoutId, date, title, kind move|skip|easier|note, note, status, reply, createdAt}`) from the app.
4. Read back `coach/plan` once to confirm it saved.
5. Tell them how to open it on their phone: open the link in the Claude app or in Safari while signed in to Claude.

**Optional standalone phone app.** For a real full-screen home-screen app with no Claude around it, the same app runs on GitHub Pages against a **private** GitHub repository that holds the same documents as JSON files (`coach/plan.json` and so on). They can use the hosted copy at https://fonsi43.github.io/fonsi-coach/ with **their own** private data repository and a fine-grained token limited to that repository (Contents: read and write). The token stays on their phone. They can also fork https://github.com/fonsi43/fonsi-coach and enable Pages on the fork. Only set this up if they ask; it also needs GitHub connected to Claude for the weekly update to write there.

## 4. The weekly update (scheduled task)

Create a scheduled task for **Sunday evening in their time zone**. Pick a minute that isn't on the hour. If it uses their computer (Garmin), mark it as needing that computer. Its prompt must be a complete standalone instruction, because each run starts fresh. Include:

- **Who the athlete is:** goals, races, zones, availability and the rules from the plan. Link the plan doc and the app.
- **Step 1: check the Garmin connection.** If it fails, ask the athlete to sign in themselves. Never handle passwords.
- **Step 2: read the week's check-ins and requests** from the app. Apply the safety rules first.
- **Step 3: review the week in the watch data.** Sessions done vs planned; easy days kept easy; quality sessions vs pace band and HR ceiling; long runs; RPE; HRV, resting HR and sleep; fitness, fatigue and form.
- **Step 4: adjust next week from the plan.**
  - Cut back on clear fatigue signals: HRV low plus another signal, form below −30, a poor check-in, illness or pain.
  - Progress only when the check-in and the data agree.
  - Respect the volume rules and their requests.
- **Step 5: load next week's structured workouts** onto the watch: warm-up, main set, cool-down; HR ranges for easy runs; pace in the description for reps. Also write Zwift `.zwo` files for bike sessions if they use Zwift. Never delete workouts the coach didn't create.
- **Step 6: update the app documents.** Fill the finished week's row in weekly, append the pmc days, write next week's plan, the summary, the report and the goals results, and mark handled requests done with a reply. Read the documents first and write with their versions.
- **Step 7: send the coaching report.** Cover the week in numbers, feedback, a recovery read, next week day by day and what changed, strength, and three to five focus cues. Make it specific, honest and encouraging.

Tell the athlete which approval setting the task got. Offer to run it once now so they see a first report.

## Ground rules

- **Use their data only.** If a link or file seems to contain another person's data, don't open it.
- **Health:** you're a coach, not a doctor. For red-flag symptoms, tell them plainly to see a professional.
- **Keep it simple for them.** Plain language, no jargon without a short explanation, and one question at a time when something is unclear.
