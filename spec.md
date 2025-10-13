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
   - Every 8th week (weeks 8, 16, 24, 32, 40, 48), multiply all weights by `deload_factor` (default 0.82).
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
    "deload_factor": 0.82
  },
  "targets": {
    "goal_bodyweight_multiple": {
      "Squat": 1.50,
      "Overhead Press": 0.6,
      "Pullup": 0.25,
      "Deadlift": 2.00,
      "Bench": 1.20,
      "Row": 0.85
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

## Testing Plan

A separate `test.html` file will validate all core calculations using a minimal vanilla JavaScript test framework.

### Test Framework Structure

```javascript
function assert(description, expected, actual, tolerance = 0.01) {
  const pass = (typeof expected === 'number') 
    ? Math.abs(expected - actual) < tolerance
    : JSON.stringify(expected) === JSON.stringify(actual);
  return { pass, description, expected, actual };
}

function runTests() {
  // Collect all test results
  // Display in table format: pass/fail, description, expected, actual
  // Summary: X passed, Y failed
}
```

### Test Coverage

**1. Date & Day Calculations**
- `getDaysElapsed()` - test with known start/end dates
- `getTrainingDayNumber()` - test weekends are skipped, multi-week spans
- `isRestDay()` - verify Saturday/Sunday return true, weekdays false
- `getCyclePosition()` - test all 4 positions (0→A, 1→B, 2→A', 3→B')
- `getWeekNumber()` - test week boundaries (day 1-5 = week 1, day 6-10 = week 2)

**2. Weight Progression Calculations**
- `compute1RM()` - test linear-log curve at key points:
  - Week 0 (start): should equal start weight
  - Week 26 (mid): verify interpolation
  - Week 51 (end): should equal goal weight
  - Example: 175 lb BW × 1.5 goal = 262.5 lb, 50% start = 131.25 lb
- Verify alpha parameter affects curve shape

**3. Working Weight (Epley Formula)**
- `computeWorkingWeight()` - test conversions:
  - 200 lb 1RM at 5 reps → 200/(1+5/30) ≈ 181.82 lb
  - 200 lb 1RM at 7 reps → 200/(1+7/30) ≈ 175.68 lb
  - With deload (0.82): multiply result by 0.82
- Test edge cases: 0 reps, very high reps

**4. Rounding & Precision**
- Test rounding to various increments:
  - 0.25 lb: 100.1→100, 100.13→100.25, 100.24→100.25
  - 1.25 lb: 57.4→57.5, 57.6→57.5, 57.7→58.75
- Test negative values (shouldn't occur but validate)

**5. Plate Math (Greedy Algorithm)**
- Test plate combinations:
  - 100 lb/side → [45, 45, 10]
  - 57.5 lb/side → [45, 10, 2.5]
  - 2.75 lb/side → [2.5, 0.25]
- Test impossible weights (e.g., 0.1 lb with 0.25 lb min plate)
- Test zero and negative values

**6. Deload Week Detection**
- `isDeloadWeek()` - verify:
  - Weeks 8, 16, 24, 32, 40, 48 return true
  - Weeks 7, 9, 15, 17 return false
  - Week 1, Week 52 return false

**7. Rep Scheme Selection**
- `getRepsForExercise()` - test alternation:
  - Squat: returns 5 for all cycle positions (0,1,2,3)
  - Overhead Press: returns 5,5,7,7 for positions (0,1,2,3)
  - Verify `alternate_reps: false` always uses low_reps
  - Verify `alternate_reps: true` alternates properly

### Test Data

Use consistent test configuration:
- Bodyweight: 175 lb
- Start date: 2025-09-15 (Monday)
- Low reps: 5, High reps: 7
- Alpha: 0.07
- Deload factor: 0.82

### Expected Output

Test results displayed in a simple table:
```
✓ getDaysElapsed: 2025-09-15 to 2025-09-16 = 1 day
✓ compute1RM: Squat week 0 = 131.25 lb
✗ computeWorkingWeight: Expected 181.82, Got 182.00
...
Summary: 45 passed, 2 failed
```

### Implementation

- Create `test.html` as standalone file
- Copy/import core calculation functions from `index.html`
- Run all tests on page load
- Display results in clean, readable format
- Color-code passes (green) and failures (red)

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
