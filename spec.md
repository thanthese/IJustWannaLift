# Barbell A/B Daily Planner 

## Program Goals

- **Time-efficient**: Short sessions you can do every day after work.
- **Strength-focused**: Prioritize steady, long-term strength gains.
- **Balanced hypertrophy**: Moderate-to-good muscle growth without excess fatigue.
- **Sustainable**: Can be run for years with minimal burnout.
- **Refreshing**: Leaves you energized and clear-headed, not wiped out.
- **Dependable**: Designed so you rarely miss a rep—built on consistency and momentum.

## Goal

Build a one-year strength training planner that:
- Uses a **linear–log progression curve** to calculate daily weights.
- Alternates between **Day A** (Squat, Overhead Press, Pullup) and **Day B** (Deadlift, Bench, Row).
- Trains Monday–Friday, rests Saturday–Sunday (configurable).
- Has an **every-8th-week deload** (weeks 8, 16, 24, etc.).
- Uses **per-lift rep patterns** that cycle through variable-length arrays:
  - Each lift has its own rep array (e.g., `[5]` for flat reps, `[5,6,5,7,5,6,5,8]` for 8-workout cycles)
  - Reps index using modulo: `reps = repArray[(trainingDayNumber - 1) % repArray.length]`
  - Squat/Deadlift typically use `[5]` (flat), upper body uses longer patterns for variety
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
    "Squat": [5],
    "Overhead Press": [5, 6, 5, 7, 5, 6, 5, 8],
    "Pullup": [5, 6, 5, 7, 5, 6, 5, 8],
    "Deadlift": [5],
    "Bench": [5, 6, 5, 7, 5, 6, 5, 8],
    "Row": [5, 6, 5, 7, 5, 6, 5, 8]
  },
  "schedule": {
    "pattern": {
      "A": ["Squat", "Overhead Press", "Pullup"],
      "B": ["Deadlift", "Bench", "Row"]
    },
    "cycle": ["A", "B"]
  },
  "sets_reps": {
    "Squat":   {"sets": 2},
    "Deadlift":{"sets": 2},
    "Overhead Press":{"sets": 3},
    "Bench":  {"sets": 3},
    "Row":    {"sets": 3},
    "Pullup": {"sets": 3, "type": "weighted_bodyweight"} 
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
    "alpha": 0.0105,
    "deload_every_weeks": 8,
    "deload_factor": 0.82
  },
  "targets": {
    "goal_bodyweight_multiple": {
      "Squat": 1.65,
      "Overhead Press": 0.75,
      "Pullup": 0.33,
      "Deadlift": 2.20,
      "Bench": 1.20,
      "Row": 0.90
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

Calculate the projected 1RM based on **calendar days** elapsed since the start date:

$$
W(t) = W_{start} + (W_{goal} - W_{start}) \cdot \frac{\ln(1+\alpha t)}{\ln(1+\alpha T)}
$$

Where:

* `t` = calendar days elapsed since start date (0-indexed: day 0 = start, day 1 = second day, ..., day 364 = goal)
* `T` = 364 (so that days 0-364 span 365 days total, with day 364 reaching 100% of goal)
* `α` = 0.0105 (curvature constant - calibrated to match weekly progression curve with α=0.07 when T=50 weeks)
* Clamp `t` between `[0, T]`

**Alpha Calibration Note:**
The alpha value of 0.0105 for a day-based system (T=364 days) produces a similar progression curve shape to alpha=0.07 with a week-based system (T=50 weeks). This ensures smooth daily progression while maintaining the same overall curve characteristics.

**Week Number Calculation:**
* Week number = `floor(daysElapsed / 7) + 1`
* Based on **calendar days**, not training days
* Example: Days 0-6 = Week 1, Days 7-13 = Week 2, Days 28-34 = Week 5

**Deload Weeks:**
* Deload occurs every 56 calendar days (8 weeks)
* Check: `daysElapsed >= 7 AND floor(daysElapsed / 56) > floor((daysElapsed - 7) / 56)`
* This means deload happens during weeks 9, 17, 25, 33, 41, 49 (starting at days 56, 112, 168, etc.)
* First week (days 0-6) can never be a deload week

**Note:** Weight progression is based on calendar days, NOT training days. The weights increase with time whether you're in the gym or resting.

### 4. Working Weight Conversion (Epley Formula)

Convert the 1RM to working weight based on the number of reps for that day:

$$
WorkingWeight = \frac{W(t)}{1 + \frac{r}{30}}
$$

* `r` = reps per set for this specific training day
  - Determined by indexing into the lift's rep pattern array
  - Formula: `reps = rep_scheme[lift][(trainingDayNumber - 1) % rep_scheme[lift].length]`
  - Example: `[5]` always returns 5 reps
  - Example: `[5,6,5,7,5,6,5,8]` cycles through 8 different workouts, then repeats
  - Squat/Deadlift typically use `[5]` (flat/consistent)
  - Upper body lifts use longer arrays for Daily Undulating Periodization (DUP)
  - The A/B cycle is based on **training days** (Mon-Fri), not calendar days
* If deload week → `WorkingWeight *= deload_factor` (0.82)
* Round to nearest increment for that lift.

### 5. Plate Math

```
PlatesPerSide = (WorkingWeight - bar_weight_lb) / 2
```

Then fill from largest to smallest plate in the available inventory.

### 6. Day Mapping

* Calculate days elapsed: `days_elapsed = current_date - start_date` (calendar days, 0-indexed)
* Determine day of week: check if `current_date` day name is in `config.user.training_days` array
  - Default training days: Monday, Tuesday, Wednesday, Thursday, Friday
  - If not in training_days → Rest Day
* Calculate training day number: count only days where `isTrainingDayISO()` returns true since start_date
* Determine cycle position: `cycle_index = (training_day_number - 1) % 2`
  - 0 → Day A
  - 1 → Day B
* Calculate week number: `week = floor(daysElapsed / 7) + 1` (based on calendar days, not training days)
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
- `getCyclePosition()` - test both positions (0→A, 1→B)
- `getWeekNumber()` - test week boundaries (day 1-5 = week 1, day 6-10 = week 2)

**2. Weight Progression Calculations**
- `compute1RM()` - test linear-log curve at key points:
  - **All exercises at Day 0**: should equal start weight (start_percentage * goal)
  - **All exercises at Day 364**: should equal goal weight (bodyweight_multiple * bodyweight)
  - Day 182 (mid): verify ~2/3 progress due to log curve
  - Examples for 175 lb bodyweight with updated goals:
    - Squat: Start 144.38 lb (50% of 288.75), Goal 288.75 lb (1.65x BW)
    - Deadlift: Start 192.5 lb (50% of 385), Goal 385 lb (2.20x BW)
    - Bench: Start 84 lb (40% of 210), Goal 210 lb (1.2x BW)
    - OHP: Start 65.63 lb (50% of 131.25), Goal 131.25 lb (0.75x BW)
    - Row: Start 94.5 lb (60% of 157.5), Goal 157.5 lb (0.90x BW)
    - Pullup: Start 0 lb (0% of 57.75), Goal 57.75 lb (0.33x BW)
- Verify alpha parameter (0.0105) produces smooth day-by-day progression

**3. Working Weight (Epley Formula)**
- `computeWorkingWeight()` - test conversions:
  - 200 lb 1RM at 5 reps → 200/(1+5/30) ≈ 171.43 lb
  - 200 lb 1RM at 7 reps → 200/(1+7/30) ≈ 162.16 lb
  - With deload (0.82): multiply result by 0.82 → 140.57 lb
- Test edge cases: 0 reps, very high reps

**4. Rounding & Precision**
- Test rounding to various increments:
  - 0.25 lb: 100.1→100, 100.13→100.25, 100.24→100.25
  - 1.25 lb: 57.4→57.5, 57.6→57.5, 58.7→58.75
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
  - Takes calendar days as parameter (not week number)
  - Weeks 9, 17, 25, 33, 41, 49 return true (days 56+, 112+, 168+, etc.)
  - Weeks 7, 8, 15, 16 return false
  - Week 1 (days 0-6) always returns false
  - Deload check: `daysElapsed >= 7 AND floor(daysElapsed/56) > floor((daysElapsed-7)/56)`

**7. Rep Pattern Arrays**
- `getRepsForExercise()` - test modulo cycling:
  - Squat: returns 5 for all training days (flat array `[5]`)
  - Bench: cycles through `[5,6,5,7,5,6,5,8]` pattern
  - Verify wrapping after array length (day 9 returns same as day 1)
  - Test large training day numbers (365+)

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

## Settings & Persistence

### User-Editable Configuration

All JSON configuration values should be editable through a collapsible settings UI at the bottom of the page.

**UI Design:**
- Collapsible accordion section (starts collapsed by default)
- Section title: "Settings" or "Configuration"
- Persist collapse/expand state to localStorage
- Fields ordered by frequency of use:
  1. User data (bodyweight, name, start_date)
  2. Goal values (bodyweight multiples for each lift)
  3. Start percentages (for each lift)
  4. Rep patterns (comma-separated for each lift)
  5. Sets configuration
  6. Rounding increments
  7. Progression parameters (alpha, deload settings)
  8. Plate inventory
  9. Training schedule

**Input Types:**
- **Name**: Text input
- **Bodyweight**: Number input (validate > 0)
- **Start Date**: HTML date picker (`<input type="date">`)
- **Goal multiples**: Number input (decimal format, e.g., "1.60")
  - Label hint: "e.g., 1.60 = 1.6× bodyweight"
- **Start percentages**: Number input (decimal format, e.g., "0.60")
  - Label hint: "e.g., 0.60 = 60% of goal"
- **Rep patterns**: Text input, comma-separated values (e.g., "5, 6, 5, 7" or just "5")
  - Label hint: "e.g., '5' for flat or '5,6,5,7' for variation"
  - Parses to integer array: `[5, 6, 5, 7]`
  - Array length is flexible (1+ values)
  - Uses modulo to cycle: `reps = array[(trainingDay - 1) % array.length]`
- **Plate inventory**: Text input, comma-separated values (e.g., "45, 25, 10, 5, 2.5")
- **Training schedule**: 7 checkboxes (Mon, Tue, Wed, Thu, Fri, Sat, Sun)

**Validation:**
- Real-time validation on every keystroke (for workingConfig)
- Invalid values highlighted in red (border or text color)
- Error message displayed near/over the invalid field
- Common validations:
  - Numbers must be positive where applicable
  - Decimals in valid range (e.g., start_percentage between 0 and 1)
  - Date must be valid (YYYY-MM-DD format)
  - Rep patterns must parse to array of positive integers (1-50 range)
  - Plate inventory must parse to valid number array
  - At least one training day must be selected

**Behavior:**
- Two-config pattern: `workingConfig` (temporary edits) + `config` (active display)
- Edit workingConfig freely without triggering re-renders
- "Save Changes" button validates and copies workingConfig → config
- Invalid state: Re-render settings to show all red borders + error messages
- On save success: Scrolls to stats table (shows progression impact)

**Persistence:**
- Save button approach (explicit save action)
- Storage key: `workout-config` (single key for entire config object)
- Collapse state key: `workout-settings-collapsed`
- Load from localStorage on page load
- If no saved config exists, use hardcoded defaults

**Reset Functionality:**
- "Reset to Defaults" button at bottom of settings section
- Restores all hardcoded default values from embedded JSON
- Does NOT set start_date to today (uses value from JSON)
- Clears localStorage
- Confirms before resetting (simple confirmation dialog)

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