# Barbell A/B Daily Planner — Prompt Specification

## Goal

Build a one-year strength training planner that:
- Uses a **linear–log progression curve** to calculate daily weights.
- Alternates between **Day A** (Squat, Overhead Press, Pullup) and **Day B** (Deadlift, Bench, Row).
- Trains Monday–Friday, rests Saturday–Sunday.
- Has an **every-8th-week deload** (weeks 8, 16, 24, etc.).
- Alternates between low and high reps in a 4-day cycle: A (low reps), B (low reps), A' (high reps), B' (high reps).
  - Rep counts are configurable (default: 5 for low, 7 for high)
  - **Exception**: Squat and Deadlift always use low reps (never alternate).
- Displays the **current day's lifts** and the **next training day's lifts** below for quick reference.
- On rest days, shows "Rest" with the next two training days displayed below.
- **Mobile-optimized**: Designed for phone portrait mode with minimal, polished styling that looks great on small screens.

The webpage will be fully static — a single `index.html` file that performs all calculations locally via embedded JSON and JavaScript.

---

## Functional Description

1. **Auto-Compute Daily Data**
   - On page load, calculate days elapsed since start_date (Day 1 = start_date).
   - Determine if today is a training day (Mon-Fri) or rest day (Sat-Sun).
   - Determine the current training day number and which day type it is (A, B, A', or B').
   - Calculate weights for each lift using the linear–log curve and Epley formula.
   - Display current day info and the next training day below.

2. **Daily Display**
   - **Training Day Title**: `Week N • <date>`
   - **Rest Day Title**: `Week N • <date> • Rest Day`
   - For each lift on training days:
     ```
     Squat — 2×5 — Total: ### lb • Plates/side: #.# lb
     ```
     or for pullups:
     ```
     Pullup — 3×5 — Add: #.# lb (System: BW + Add)
     ```
   - On rest days, show "Rest" message and display the next two training days below.
   - On training days, display the next training day in the same format below.

3. **Rounding & Precision**
   - Each lift rounds independently to its configured increment.
   - The `round_plates_per_side_to_lb` value represents the rounding increment for plates on ONE SIDE of the bar.

4. **Plate Math**
   - Uses a standard plate inventory (e.g., 45/25/10/5/2.5).
   - Greedy fill algorithm for plate assignment.

5. **Deload Logic**
   - Every 8th week (weeks 8, 16, 24, 32, 40, 48), multiply all weights by `deload_factor` (default 0.80).
   - Resume curve progression afterward (not a reset).

---

## JSON Configuration

Embedded directly in the HTML inside:
```html
<script type="application/json" id="program-config">…</script>
```

Example structure:

```json
{
  "user": {
    "name": "Stephen",
    "bodyweight_lb": 175,
    "start_date": "2025-09-15",
    "total_weeks": 52,
    "training_days": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"]
  },
  "bar_weight_lb": 45,
  "rep_scheme": {
    "low_reps": 5,
    "high_reps": 7
  },
  "schedule": {
    "pattern": {
      "A": ["Squat", "Overhead Press", "Pullup"],
      "B": ["Deadlift", "Bench", "Row"]
    },
    "cycle": ["A", "B", "A'", "B'"]
  },
  "sets_reps": {
    "Squat":   {"sets": 2, "alternate_reps": false},
    "Deadlift":{"sets": 2, "alternate_reps": false},
    "Overhead Press":{"sets": 3, "alternate_reps": true},
    "Bench":  {"sets": 3, "alternate_reps": true},
    "Row":    {"sets": 3, "alternate_reps": true},
    "Pullup": {"sets": 3, "alternate_reps": true, "type": "weighted_bodyweight"} 
  },
  "round_plates_per_side_to_lb": {
    "Squat": 1.25, 
    "Deadlift": 1.25, 
    "Overhead Press": 0.25, 
    "Bench": 0.25, 
    "Row": 0.25, 
    "Pullup": 1.25
  },
  "progression": {
    "alpha": 0.07,
    "deload_every_weeks": 8,
    "deload_factor": 0.80
  },
  "targets": {
    "goal_bodyweight_multiple": {
      "Squat": 1.50,
      "Overhead Press": 0.75,
      "Pullup": 0.33,
      "Deadlift": 2.00,
      "Bench": 1.50,
      "Row": 1.00
    },
    "start_as_percentage_of_goal": {
      "Squat": 0.50,
      "Overhead Press": 0.50,
      "Pullup": 0.00,
      "Deadlift": 0.50,
      "Bench": 0.40,
      "Row": 0.60
    }
  },
  "plate_inventory_per_side_lb": [45, 25, 10, 5, 2.5, 1.25, 1, 0.75, 0.5, 0.25]
}
```

---

## Calculation Formulas

### 1. Goal 1RM

```
W_goal(lift) = goal_bodyweight_multiple[lift] * bodyweight_lb
```

> For pull-ups: this represents **additional weight** beyond bodyweight.

### 2. Start 1RM

```
W_start(lift) = start_as_percentage_of_goal[lift] * W_goal(lift)
```

### 3. Linear–Log Progression

Calculate the projected 1RM for a given week:

$$
W(t) = W_{start} + (W_{goal} - W_{start}) \cdot \frac{\ln(1+\alpha t)}{\ln(1+\alpha T)}
$$

Where:

* `t` = current week (0-indexed: week 1 = t=0, week 2 = t=1, etc.)
* `T` = total weeks - 1
* `α` = curvature constant controlling how front-loaded progress is
* Clamp `t` between `[0, T]`

### 4. Working Weight Conversion (Epley Formula)

Convert the 1RM to working weight based on the number of reps for that day:

$$
WorkingWeight = \frac{W(t)}{1 + \frac{r}{30}}
$$

* `r` = reps per set for this specific training day
  - For Day A and B: use `low_reps` (unless exercise has `alternate_reps: false`)
  - For Day A' and B': use `high_reps` (unless exercise has `alternate_reps: false`)
  - Exercises with `alternate_reps: false` (Squat, Deadlift) always use `low_reps`
* If deload week (weeks 8, 16, 24, 32, 40, 48) → `WorkingWeight *= deload_factor`
* Round to nearest increment for that lift.

### 5. Plate Math

```
PlatesPerSide = (WorkingWeight - bar_weight_lb) / 2
```

Then fill from largest to smallest plate in the available inventory.

### 6. Day Mapping

* Calculate days elapsed: `days_elapsed = current_date - start_date` (where start_date is Day 1)
* Determine day of week: if Saturday or Sunday → Rest Day
* Calculate training day number: count only Mon-Fri days since start_date
* Determine cycle position: `cycle_index = (training_day_number - 1) % 4`
  - 0 → Day A (low reps)
  - 1 → Day B (low reps)  
  - 2 → Day A' (high reps)
  - 3 → Day B' (high reps)
* Calculate week number: `week = floor((training_day_number - 1) / 5) + 1`
* On rest days, display "Rest" and show the next two training days

---

## Implementation Notes

* Single file (`index.html`) — all logic inline.
* JavaScript-only (no frameworks).
* **Mobile-first styling**:
  - Optimized for phone portrait mode (320px - 428px width)
  - Minimal, polished design with good typography
  - Clear visual hierarchy with appropriate spacing
  - Touch-friendly (no hover-dependent features)
  - Large, readable font sizes (minimum 16px for body text)
  - High contrast for readability in various lighting
  - Efficient use of vertical space
  - System fonts for fast loading
* Core functions:
  * `parseConfig()`
  * `getDaysElapsed(startDate, currentDate)`
  * `getTrainingDayNumber(startDate, currentDate)` - counts only Mon-Fri
  * `isRestDay(date)` - checks if Sat/Sun
  * `getCyclePosition(trainingDayNumber)` - returns A, B, A', or B'
  * `getWeekNumber(trainingDayNumber)`
  * `isDeloadWeek(weekNumber)` - checks if week 8, 16, 24...
  * `getRepsForExercise(lift, cyclePosition)` - returns low_reps or high_reps based on alternation
  * `compute1RM(lift, weekNumber)`
  * `computeWorkingWeight(oneRM, reps, isDeload)`
  * `plateMath(workingWeight, barWeight, plateInventory)`
  * `renderTrainingDay(dayType, trainingDayNumber)`
  * `renderRestDay(nextTrainingDays)`
