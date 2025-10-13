# Changelog

## Simplified 2-Day Cycle (Latest)

### Changes
- **Cycle simplification**: Changed from `["A", "B", "A'", "B'"]` to `["A", "B"]`
  - Pattern A: Squat, Overhead Press, Pullup
  - Pattern B: Deadlift, Bench, Row
  - Simpler rotation: just alternates A/B on training days
  - `getCyclePosition()` now uses `% 2` instead of `% 4`
- **Test fixes**: Updated all tests to match new 2-day cycle
- All 152 tests now passing ✓

## Variable-Length Rep Patterns

### Major Changes

**Rep System Overhaul:**
- Replaced global `low_reps`/`high_reps` with per-lift rep pattern arrays
- Each lift now has its own variable-length rep pattern
- Patterns cycle using modulo arithmetic: `reps = array[(trainingDay - 1) % array.length]`

**Default Configuration:**
```javascript
"rep_scheme": {
  "Squat": [5],                            // Flat 5s (consistent)
  "Overhead Press": [5, 6, 5, 7, 5, 6, 5, 8],  // 8-workout DUP cycle
  "Pullup": [5, 6, 5, 7, 5, 6, 5, 8],          // 8-workout DUP cycle
  "Deadlift": [5],                         // Flat 5s (consistent)
  "Bench": [5, 6, 5, 7, 5, 6, 5, 8],          // 8-workout DUP cycle
  "Row": [5, 6, 5, 7, 5, 6, 5, 8]             // 8-workout DUP cycle
}
```

**Benefits:**
- **Squat/Deadlift**: Consistent 5 reps for heavy compound movements
- **Upper body**: 8-workout Daily Undulating Periodization pattern
- **Flexibility**: Users can create any pattern length (1 to 50+ values)
- **Simple UI**: Comma-separated text input per lift

**Code Changes:**
- `getRepsForExercise(lift, trainingDayNumber)` - Now uses modulo indexing
- Removed `alternate_reps` flag from `sets_reps` config
- Stats table hardcoded to 5 reps (provides consistent baseline)

**Validation:**
- Each lift must have non-empty array
- All values must be positive integers (1-50 range)
- Per-lift error messages with field highlighting

**Settings UI:**
- Text inputs for rep patterns (one per lift)
- Hint text: `e.g., '5' for flat or '5,6,5,7' for variation`
- Parses comma-separated values to integer arrays
- Red border + inline error message for invalid input

**Documentation Updated:**
- `spec.md`: Updated goal description, JSON structure, formulas, settings section
- `README.md`: Updated "How It Works" and "Customization" sections
- Both files now reflect variable-length rep pattern system

### Breaking Changes

⚠️ **Old localStorage configs will fail validation** (acceptable since unreleased)

Old format:
```javascript
"rep_scheme": {
  "low_reps": 5,
  "high_reps": 7
}
```

New format:
```javascript
"rep_scheme": {
  "Squat": [5],
  "Overhead Press": [5, 6, 5, 7, 5, 6, 5, 8],
  // ... etc for all lifts
}
```

Users with old configs must click "Reset to Defaults" to get new structure.
