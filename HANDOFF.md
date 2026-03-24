# GPP-30min Workout Timer App — Claude Code Handoff

## What This Is

A single self-contained HTML file (`GPP-30min.html`) that runs a 7-day rotating workout timer. No dependencies, no build step, no server. User downloads the file, taps it on their phone, it opens in the browser and runs.

The user is Dylan — a physical therapist and business owner with zero coding background. All communication about this project should be in plain English.

---

## Architecture

**Single file.** Everything lives in one `.html` file: CSS in a `<style>` block, all workout data and app logic in a single `<script>` block. No external JS/CSS except two Google Fonts (Outfit + JetBrains Mono).

**Key sections in the file (top to bottom):**

1. **CSS** — Dark theme, color-coded by exercise category (green = strength, blue = cardio, yellow/amber = rest). Intensity days color-coded (red = high, amber = medium, green = low).
2. **HTML skeleton** — Day selector, day header, workout preview list, active workout display, completion screen, controls. Sections toggle via `.hidden` class.
3. **Workout data construction** — Helper functions (`straightSets`, `superset`, `circuit`, `cardioBlock`, `intervals`, `transition`) build step arrays. Each day is an IIFE that pushes to the `DAYS` array. The last step of every day is a cardio "finisher" whose duration is calculated as `1800 - used` to guarantee exactly 30:00.
4. **App logic** — Timer tick, display updates, controls (start/pause/skip/reset), audio beeps, wake lock, localStorage for last-used day.

---

## Data Model

Each day in `DAYS[]`:
```
{
  name: "Day 1",
  title: "Squat + Push",
  subtitle: "Quads · Chest · Triceps · Glutes",
  intensity: "high" | "med" | "low",
  steps: [ ...array of step objects ]
}
```

Each step:
```
{
  cat: "strength" | "cardio" | "rest",
  name: "Barbell Back Squat",
  detail: "5 reps @ RPE 8",
  duration: 50,          // seconds
  block: "Heavy Squat",  // optional, groups steps in preview list
  round: "Set 2/4"       // optional, shown during active workout
}
```

**Critical constraint:** Every day's steps must sum to exactly 1800 seconds. The finisher cardio block at the end auto-fills the remainder. If you add/remove/change step durations, the finisher absorbs the difference. If the finisher goes negative, the day is over 30 minutes and you need to shorten something.

---

## The 7-Day Program

| Day | Intensity | Emphasis | Key Exercises |
|-----|-----------|----------|---------------|
| 1 | HIGH | Squat + Push | Back Squat, DB Floor Press, KB Goblet Squat, Push-Up |
| 2 | MED | Hinge + Pull | Barbell RDL, Cable Row, Cable Reverse Curl, KB Swing, Leg Curl |
| 3 | LOW | Movement + Core | KB Goblet Squat (slow), Cable OH Tricep Ext, Pallof Press, Dead Bug, Calf Raise, Cable Hip Ad/Ab, Roman Chair |
| 4 | HIGH | Vertical Press + Pull | Barbell OHP, Pull-Up, DB Lateral Raise, Med Ball Slam, Landmine Row |
| 5 | MED | Single Leg + Arms | Bulgarian Split Squat, Leg Ext/Curl, Cable Curl, Cable Pushdown |
| 6 | LOW | GPP Conditioning | Calf Raise, KB Clean, Hanging Knee Raise, Band Lateral Walk, Cable Face Pull, Plank Shoulder Tap |
| 7 | MED | Carry + Core + Unilateral | Farmer Carry, Single Leg RDL, Cable Woodchop, Pallof Press, Med Ball Slam, KB Snatch, Plyo Inverted Row, GHR, DB Wrist Curl/Extension |

**Muscle group spacing was carefully audited.** No muscle group hits on consecutive days, including across the Day 7 → Day 1 cycle wrap. If swapping exercises, check the adjacency map below.

---

## Muscle Group Adjacency Map (current)

| Group | Days Hit |
|-------|----------|
| Quads | 1, 3, 5 |
| Hamstrings | 2, 5, 7 |
| Chest/Push | 1 |
| Back/Pull | 1, 2, 4, 7 |
| Shoulders | 4 |
| Biceps | 2, 5 |
| Triceps | 3, 5 |
| Core | 3, 6, 7 |
| Calves | 3, 6 |
| Hip Ad/Ab | 3 |
| Glute Med | 6 |
| Rear Delt | 1, 4, 6 |
| Grip/Forearm | 2, 7 |

---

## How to Swap Exercises

1. Find the exercise string in the `<script>` block (search by exercise name)
2. Replace the `name`, `detail`, and optionally `dur` fields
3. If you change `dur`, the finisher auto-adjusts — but verify the finisher doesn't go negative
4. To validate all days: run the file in a browser and check the console for warnings like `Day X total: Ys (expected 1800s)`
5. Check the adjacency map above — make sure the muscle group you're adding doesn't already appear on the adjacent day(s)

---

## Helper Function Reference

- `straightSets(name, detail, sets, workSec, restSec, block)` — classic sets with rest between
- `superset(exercises[], rounds, restBetweenRounds, block)` — alternating exercises, 8s transition between, rest between rounds
- `circuit(exercises[], rounds, restBetweenEx, restBetweenRounds, block)` — multiple exercises in sequence
- `cardioBlock(name, detail, duration, block)` — single timed cardio segment
- `intervals(workName, workDetail, workDur, restName, restDetail, restDur, rounds, block)` — work/rest intervals
- `transition(dur)` — short rest labeled "Transition · Set up next block"

---

## UI Features

- **Day selector** across top with intensity color dots
- **Workout preview** before starting (scrollable list of all exercises)
- **Active display** shows: countdown timer (color-coded), total remaining, progress bar, current exercise with name/detail/round, up-next preview
- **Controls**: Start, Pause/Resume, Skip, Reset
- **Audio**: beeps at transitions, countdown beeps at 3-2-1, triple beep on completion
- **Wake lock**: prevents screen sleep during workout
- **localStorage**: remembers last selected day

---

## Equipment Available

Walking pad (12% incline, 3.8 mph max), squat rack, cable stack, barbell + plates, kettlebells, dumbbells, bench, pull-up bar, jump rope, med ball, yoga mat, landmine attachment, power bands, leg extension machine, leg curl machine, Roman chair, GHR, slant board, stair stepper.

**Constraints**: No rotational med ball throws (can only slam). All other equipment is fair game.

---

## Known Limitations / Future Work

- No progressive overload tracking — it's a timer, not a logbook
- No rep counting — user self-manages reps within the time window
- Fonts require internet on first load (Google Fonts) — could be embedded for true offline
- Wake lock API not supported on all browsers — screen may sleep on older devices
- No sound toggle — beeps always play if audio context is available
