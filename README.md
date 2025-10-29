# I Just Wanna Lift

A simple strength training planner that runs in your browser. No apps, no tracking, no decisions—just show up and lift.

**[→ Try the live demo](https://thanthese.github.io/IJustWannaLift/)**

## Philosophy

**Refreshing** • Leaves you energized and clear-headed, not wiped out  
**Time-efficient** • Short sessions you can do every day after work  
**Strength-focused** • Prioritize steady, long-term strength gains  
**Balanced hypertrophy** • Moderate-to-good muscle growth without excess fatigue  
**Sustainable** • Can be run for years with minimal burnout  
**Dependable** • Designed so you rarely miss a rep—built on consistency and momentum

## How It Works

- **A/B Split**: Alternate between Day A (Squat, Overhead Press, Pullup) and Day B (Deadlift, Bench, Row)
- **5 Days/Week**: Train Monday–Friday, rest Saturday–Sunday (configurable)
- **Linear-Log Progression**: Smooth daily increases via logarithmic curve—weights feel easy early, challenging later
- **2-Year Program**: Progress for 730 days, then automatically maintain strength forever
- **Deload Weeks**: Built-in recovery every 8 weeks (82% intensity)
- **Variable Rep Patterns**: Each lift has a customizable rep pattern that cycles automatically
  - Squat/Deadlift: Flat 5s (consistent heavy work)
  - Upper body: 8-workout cycles for Daily Undulating Periodization (e.g., 5,6,5,7,5,6,5,8)

## Your Progression

Starting from Day 1, you'll gradually build to these Year 2 targets:

| Exercise | Year 2 Goal | As Bodyweight Multiple (175 lb) |
|----------|-------------|----------------------------------|
| Squat | 280 lb | 1.60×BW |
| Deadlift | 350 lb | 2.00×BW |
| Bench Press | 201 lb | 1.15×BW |
| Overhead Press | 114 lb | 0.65×BW |
| Barbell Row | 149 lb | 0.85×BW |
| Weighted Pullup | +53 lb | 0.30×BW |

*The progression curve spans 2 years (730 days), reaching these goals at the end of Year 2, then maintaining indefinitely.*

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

## The Math

Weight progression uses a **linear-log curve**:

$$W(t) = W_{\text{start}} + (W_{\text{goal}} - W_{\text{start}}) \times \frac{\ln(1+\alpha t)}{\ln(1+\alpha T)}$$

Where:
- $t$ = days elapsed (0-730)
- $T$ = 364 days (defines curve shape)
- $\alpha$ = 0.0105 (calibrated for smooth daily progression)
- $W_{\text{start}}$ = 40-60% of your goal (depending on exercise)
- $W_{\text{goal}}$ = Your Year 1 target (as bodyweight multiple)

The formula **clamps at Day 730**, so after 2 years you maintain that strength indefinitely.

Working weight for sets uses the **Epley formula**:

$$\text{WorkingWeight} = \frac{\text{1RM}}{1 + \text{reps}/30}$$

Deload weeks (every 8 weeks) multiply working weight by 0.82.

## Long-Term Use

**Years 1-2**: Progressive overload via the logarithmic curve  
**Year 3+**: Automatic maintenance mode (same weights forever)

No modifications needed. The program transitions from progression → maintenance automatically after Day 730.

*Optional Year 3 adjustment*: Change `training_days` to 3 days/week for lifelong maintenance with less time commitment.

## Customization

Click **Settings** at the bottom of the page to customize all parameters:

- **User Data**: Name, bodyweight, start date
- **Goal Strength**: Target 1RM for each lift (as bodyweight multiples)
  - Example: `1.60` means 1.6× your bodyweight
- **Starting Strength**: Where you begin (as decimal percentage of goal)
  - Example: `0.60` means you start at 60% of your goal weight
- **Rep Patterns**: Comma-separated rep pattern for each lift
  - Example: `5` for flat reps every workout
  - Example: `5, 6, 5, 7, 5, 6, 5, 8` for an 8-workout DUP cycle
  - Patterns automatically cycle using modulo arithmetic
- **Training Schedule**: Check which days you want to train
- **Plate Inventory**: Comma-separated list of available plates per side
- **Progression Parameters**: Alpha (curve shape) and deload factor

All changes are **live-updated** and **automatically saved** to localStorage. Invalid values are highlighted in red with error messages.

### Reset to Defaults

The "Reset to Defaults" button restores all hardcoded values and sets the start date to today. This clears your localStorage and cannot be undone.

## Testing

**[→ Run the test suite](https://thanthese.github.io/IJustWannaLift/test.html)**

Run `test.html` in a browser to verify:
- 152+ tests across 22 categories
- Date calculations, deload logic, progression curve, rep patterns
- Visual tables showing week-by-week progression

All tests are config-driven and adapt when you change goals.

## Files

- `index.html`—The main app (single-file, no dependencies)
- `test.html`—Comprehensive test suite
- `spec.md`—Technical specification and design decisions

## License

Do whatever you want with it. Lift heavy, stay consistent, get strong.

---

*"The best program is the one you actually run."*
