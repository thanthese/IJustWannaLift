# Code Review Findings

## Questions Answered

### 1. Is `cycle` still used?
**YES** - The `cycle` array is still actively used to determine workout patterns (A/B rotation).

Location: `config.schedule.cycle = ["A", "B"]`

Used in:
- `getCyclePosition(trainingDayNumber)` - Returns position in 2-day cycle
- `renderTrainingDay()` - Uses cycle to determine which pattern (A or B) to show
- Pattern lookup: `const cycleDay = config.schedule.cycle[cyclePosition]`
- Then extracts pattern: `const pattern = cycleDay.replace("'", "")` → gets "A" or "B"
- Finally looks up lifts: `const lifts = config.schedule.pattern[pattern]`

**Conclusion**: Keep `cycle` - it's essential for alternating between workout days.

**Note**: Simplified from `["A", "B", "A'", "B'"]` to just `["A", "B"]` for cleaner rotation.

### 2. Is `alternate_reps` still used?
**NO** - The `alternate_reps` flag is completely unused legacy code.

Location: In `sets_reps` config for each lift
- ~~`"Squat": {"sets": 2, "alternate_reps": false}`~~
- ~~`"Overhead Press": {"sets": 3, "alternate_reps": true}`~~
- etc.

**Not referenced anywhere** in the codebase after migration to variable-length rep arrays.

**Action Taken**: Removed all `alternate_reps` flags from config.

## Changes Made

### 1. Removed `alternate_reps` from config (index.html)
```javascript
// Before:
"sets_reps": {
  "Squat": {"sets": 2, "alternate_reps": false},
  "Bench": {"sets": 3, "alternate_reps": true},
  // ...
}

// After:
"sets_reps": {
  "Squat": {"sets": 2},
  "Bench": {"sets": 3},
  // ...
}
```

### 2. Created Unit Tests (test.html)

Updated test file with comprehensive tests for variable-length rep arrays:

**Test Coverage:**
- ✓ Flat rep arrays (length 1): Squat, Deadlift always return 5
- ✓ Variable rep arrays (length 8): Full cycle through [5,6,5,7,5,6,5,8]
- ✓ Modulo wrapping: Day 9 wraps to index 0
- ✓ All lifts with patterns: OHP, Pullup, Bench, Row
- ✓ Edge cases: Large training day numbers (365+)
- ✓ Modulo arithmetic validation

**Updated test configuration:**
- Migrated from `{low_reps: 5, high_reps: 7}` to per-lift arrays
- Removed `alternate_reps` from test config
- Updated `getRepsForExercise()` to use modulo indexing

**Test file opened in browser** - check results there!

## Summary

1. **`cycle`** - ✓ Still needed, determines A/B workout pattern rotation
2. **`alternate_reps`** - ✗ Removed, replaced by variable-length rep arrays
3. **Tests** - ✓ Created comprehensive unit tests for new rep pattern system

The system now has:
- Clean config without legacy fields
- Full test coverage for variable-length rep arrays
- Documentation (spec.md, README.md, CHANGELOG.md) all updated

## Update: Simplified to 2-Day Cycle

### Changes Made
1. **Cycle simplified**: `["A", "B", "A'", "B'"]` → `["A", "B"]`
   - Removes the prime notation (A'/B')
   - Cleaner alternation: just A, B, A, B, ...
   - Still achieves the same goal: alternating between two workout patterns

2. **Updated `getCyclePosition()`**: `% 4` → `% 2`

3. **All tests passing**: 152/152 tests now pass ✓
   - Fixed "Rep Cycle" tests that were using old cycle logic
   - Fixed "Cycle Wrap" tests for 2-day rotation
   - Updated all documentation

### Why This Works
- The **cycle** determines which lifts you do (A pattern vs B pattern)
- The **rep arrays** determine how many reps (independent of cycle)
- With 2-day cycle:
  - Day 1: Pattern A (Squat, OHP, Pullup) - reps determined by rep arrays
  - Day 2: Pattern B (Deadlift, Bench, Row) - reps determined by rep arrays
  - Day 3: Pattern A again
  - Day 4: Pattern B again
  - etc.

### Benefits
- Simpler to understand
- Same functionality (still alternates A/B patterns)
- Less confusing notation (no primes)
- Cleaner code (`% 2` vs `% 4`)
