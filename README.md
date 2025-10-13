# I Just Wanna Lift

A simple, deterministic strength training planner that runs in your browser. No apps, no tracking, no decisions -- just show up and lift what it says.

## Philosophy

**Time-efficient** • Short sessions you can do every day after work  
**Strength-focused** • Prioritize steady, long-term strength gains  
**Balanced hypertrophy** • Moderate-to-good muscle growth without excess fatigue  
**Sustainable** • Can be run for years with minimal burnout  
**Refreshing** • Leaves you energized and clear-headed, not wiped out  
**Dependable** • Designed so you rarely miss a rep—built on consistency and momentum

## How It Works

- **A/B Split**: Alternate between Day A (Squat, Overhead Press, Pullup) and Day B (Deadlift, Bench, Row)
- **5 Days/Week**: Train Monday–Friday, rest Saturday–Sunday (configurable)
- **Linear-Log Progression**: Smooth daily increases via logarithmic curve—weights feel easy early, challenging later
- **2-Year Program**: Progress for 730 days, then automatically maintain strength forever
- **Deload Weeks**: Built-in recovery every 8 weeks (82% intensity)
- **Rep Alternation**: 4-day cycle rotates between 5 and 7 reps (except squat/deadlift stay at 5)

## Your Progression

Starting from Day 1, you'll gradually build to these Year 2 targets:

| Exercise | Year 2 Goal | As Bodyweight Multiple (175 lb) |
|----------|-------------|----------------------------------|
| Squat | 332 lb | 1.90×BW |
| Deadlift | 415 lb | 2.37×BW |
| Bench Press | 246 lb | 1.41×BW |
| Overhead Press | 135 lb | 0.77×BW |
| Barbell Row | 171 lb | 0.98×BW |
| Weighted Pullup | +72 lb | 0.41×BW |

*Year 1 targets are ~18% lower, giving you room to progress in Year 2.*

## Setup

1. Open `index.html` in any browser (no build required)
2. Click "Settings" at the bottom to customize:
   - **User Data**: Name, bodyweight, start date
   - **Goals**: Target strength for each lift (as bodyweight multiples)
   - **Starting Strength**: Where you begin (as % of goal)
   - **Training Schedule**: Which days you train (default: Mon–Fri)
   - All other parameters (reps, plates, progression curve, etc.)
3. Settings are automatically saved to your browser's localStorage
4. Bookmark the page on your phone
5. Show up and lift

**Note**: Settings are stored locally in your browser. If you clear browser data, you'll lose your settings. Write down your key values (bodyweight, start date, goals) as backup.

## Daily Use

The app shows:
- **Today's workout** (if training day) or "Rest" (if rest day)
- **Tomorrow's preview** so you can plan ahead
- Weight to load on the bar, plates per side, sets × reps
- **Progress tables** showing milestones at 0, 6, 12, 18, 24 months
- **Strength gains** showing percentage increases over time

No logging required. The date drives everything.

## Customization

Click **Settings** at the bottom of the page to customize all parameters:

- **User Data**: Name, bodyweight, start date
- **Goal Strength**: Target 1RM for each lift (as bodyweight multiples)
  - Example: `1.60` means 1.6× your bodyweight
- **Starting Strength**: Where you begin (as decimal percentage of goal)
  - Example: `0.60` means you start at 60% of your goal weight
- **Rep Scheme**: Low reps (default 5) and high reps (default 7)
- **Training Schedule**: Check which days you want to train
- **Plate Inventory**: Comma-separated list of available plates per side
- **Progression Parameters**: Alpha (curve shape) and deload factor

All changes are **live-updated** and **automatically saved** to localStorage. Invalid values are highlighted in red with error messages.

### Reset to Defaults

The "Reset to Defaults" button restores all hardcoded values and sets the start date to today. This clears your localStorage and cannot be undone.

## The Math

Weight progression uses a **linear-log curve**:

```
W(t) = W_start + (W_goal - W_start) × ln(1+αt) / ln(1+αT)
```

Where:
- `t` = days elapsed (0-730)
- `T` = 364 days (defines curve shape)
- `α` = 0.0105 (calibrated for smooth daily progression)
- `W_start` = 40-60% of your goal (depending on exercise)
- `W_goal` = Your Year 1 target (as bodyweight multiple)

The formula **clamps at Day 730**, so after 2 years you maintain that strength indefinitely.

Working weight for sets uses the **Epley formula**:

```
WorkingWeight = 1RM / (1 + reps/30)
```

Deload weeks (every 8 weeks) multiply working weight by 0.82.

## Long-Term Use

**Years 1-2**: Progressive overload via the logarithmic curve  
**Year 3+**: Automatic maintenance mode (same weights forever)

No modifications needed. The program transitions from progression → maintenance automatically after Day 730.

*Optional Year 3 adjustment*: Change `training_days` to 3 days/week for lifelong maintenance with less time commitment.

## Testing

Run `test.html` in a browser to verify:
- 90+ tests across 22 categories
- Date calculations, deload logic, progression curve
- Visual tables showing week-by-week progression

All tests are config-driven and adapt when you change goals.

## Files

- `index.html` - The main app (single-file, no dependencies)
- `test.html` - Comprehensive test suite
- `spec.md` - Technical specification and design decisions
- `notes.md` - Development notes

## License

Do whatever you want with it. Lift heavy, stay consistent, get strong.

---

*"The best program is the one you actually run."*
