# Project Context

## What it is
A single-file HTML habit tracker (`habit-tracker.html`) built for mobile browser use. No build tools, no dependencies — open in browser and it works. There is also a `MyApp/` folder which is an untouched `create-expo-app` Expo/React Native boilerplate (2 tab screens, no custom logic yet).

---

## habit-tracker.html — Architecture

**Core pattern:** All state lives in module-level JS variables. One `render()` function rebuilds the entire UI on every state change. Data is read from and written to `localStorage` (`key: 'ht_v1'`) on every interaction via `load()` / `save()`.

**External dependency:** SortableJS 1.15.3 (CDN) — used for long-press drag-to-reorder habit cards. Google Fonts (Plus Jakarta Sans) for typography.

**Data model:**
```js
{
  habits: [{
    id: string,           // uid() = Date.now().toString(36) + random
    name: string,
    dates: string[],      // ISO "YYYY-MM-DD" strings of completed days
    createdAt: string,    // "YYYY-MM-DD"
    colorIdx: number,     // index into PALETTE (0–7), assigned as habits.length % 8
    emoji: string,        // stored if manually picked; falls back to getHabitEmoji(name)
    type: string,         // 'daily' | 'weekly' | 'monthly' (default: 'daily')
    target: number        // completion target per period (default 1; daily ignores it)
  }]
}
```

**State variables:**
```js
let currentView      = 'home';    // 'home' | 'weekly' | 'calendar' | 'stats'
let vYear, vMonth                 // calendar tab month being viewed
let wkOffset         = 0;        // 0=current week, -1=last week (weekly tab nav)
let homeWeekOffset   = 0;        // week nav on home strip (0=current week)
let selectedHomeDate = todayStr(); // day selected in home week strip
let selectedCalHabit = null;     // habit ID selected in calendar tab
let expandedHabits   = new Set(); // habit IDs with inline calendar open (home tab)
let habitMonths      = new Map(); // per-habit { year, month } for inline calendar nav
let pendingEmoji     = null;     // emoji chosen in picker before habit is saved
let pendingType      = 'daily';  // type chosen in type picker before habit is saved
let pendingTarget    = 1;        // target count chosen before habit is saved
let homeFilter       = 'all';    // 'all' | 'daily' | 'weekly' | 'monthly' — home list filter
let pickerTarget     = null;     // 'add' | habitId — emoji picker callback
let targetEditId     = null;     // habitId being edited in target sheet
let sheetTargetValue = 1;        // current value shown in target edit sheet
let _sortable        = null;     // SortableJS instance for home list drag-to-reorder
```

**Key helpers:**
- `getWeekDays(weekOffset=0)` — returns 7 ISO strings Mon–Sun
- `getWeekStart(ds)` — returns the Monday ISO string for the week containing `ds`
- `getHabitEmoji(name)` — keyword regex map; fallback is `defaults[(name.charCodeAt(0)||0) % defaults.length]`
- `calcStreak(dates)` — daily streak: anchors on today or yesterday, walks backwards
- `calcStreakWeekly(dates, target=1)` — consecutive weeks where completions ≥ target
- `calcStreakMonthly(dates, target=1)` — consecutive months where completions ≥ target
- `calcStreakByType(habit)` — dispatches to the correct streak function, passing `habit.target`
- `streakUnit(habit)` — returns `'d'` / `'w'` / `'m'` based on `habit.type`
- `calcBestStreak(dates)` — all-time best consecutive day streak
- `load()` — includes migration: `completedDates` → `dates`, missing `colorIdx`/`type`/`target` backfilled
- `isPerfect(habit, week, td)` — defined but unused (PRF badge was removed)
- `initSortable()` — creates/recreates SortableJS instance on `#home-list`; uses `filter: '.habit-section-label'` and data-id–based reorder
- `showToast(msg)` — shows a brief floating notification (3s)
- `cycleHabitType(id)` — cycles a habit's type Daily → Weekly → Monthly → Daily, saves, re-renders, shows toast
- `setHabitType(type)` — sets `homeFilter` + `pendingType`, resets `pendingTarget`, shows/hides target picker row, re-renders
- `updateTargetPickerRow()` — shows/hides target picker row and updates its label/button states
- `changePendingTarget(delta)` — +/− for pending target in add form
- `showTargetSheet(id)` — opens target edit bottom sheet for an existing habit
- `closeTargetSheet()` — closes target edit sheet
- `changeSheetTarget(delta)` — +/− buttons inside target edit sheet
- `saveTargetEdit()` — saves sheet target value to habit, closes sheet, shows toast
- `selectHomeDay(ds)` — sets `selectedHomeDate`, re-renders
- `changeHomeWeek(dir)` — navigates home week strip by ±1 week, updates `selectedHomeDate`
- `initWeekStripSwipe()` — attaches swipe (touch + mouse) listeners to week strip for week navigation

---

## Views

**Home tab:**
- **Week strip** (Mon–Sun dots) at top — tap any past/today dot to select it (purple fills selected dot; today shows light purple tint when not selected); swipe left/right on the strip to navigate previous/future weeks (`homeWeekOffset`)
- Add form: emoji pick button + text input + submit
- **Filter/type picker row** (All / ☀️ Daily / 📅 Weekly / 🗓 Monthly) — filters the habit list AND sets type for new habit
- **Target picker row** — appears below filter row when Weekly or Monthly is selected; `− N × / week +` stepper sets `pendingTarget` for the new habit
- Stats row (3 cards): Done Today / This Week % / Best Streak — "Done Today" count reflects the selected day (`selectedHomeDate`)
- Habit cards list — when "All" is selected: grouped into sections (☀️ Daily / 📅 Weekly / 🗓 Monthly); when a type is selected: shows only that type's habits
- Each habit card has: emoji icon (tap to change emoji), name, streak badge (unit: d/w/m), **type chip** (☀️/📅/🗓 — tap to cycle type), **target chip** (e.g. `3×` — tap to open target edit sheet; weekly/monthly only), complete button (✓/+ — targets `selectedHomeDate`), calendar toggle button, delete button (×)
- Inline expandable calendar per card, with its own month nav (`habitMonths` map)
- Long-press any card to drag-reorder within and across sections (SortableJS, 400ms delay on touch)
- ⋯ menu button (top-right) opens the Data Management bottom sheet

**Weekly tab:**
- 🏠 home button in page header
- Week nav bar with `‹ ›` buttons (next disabled when `wkOffset >= 0`)
- Habit × day grid: emoji + name column, then 7 × 26px square cells (`.wk-sq`) — each row uses its habit's gradient colour
  - `done` = solid accent colour, `missed` = translucent dark, `future` = near-transparent, `today-open` = grey with purple outline
  - **`rest`** = very faint (weekly/monthly habits: days where the weekly/monthly target was already met by another completion that period)
- 4 summary stats with coloured gradient backgrounds: % Met (red), Best Day (blue), Total Done (green), Best Streak (amber)
- Background: soft lavender-to-blue gradient

**Calendar tab:**
- 🏠 home button in page header
- Scrollable horizontal emoji bubble picker row at top — tapping swaps the calendar below
- Selected habit name + streak badge info bar (streak shown in correct unit d/w/m)
- Legend (Done / Missed / Today / Upcoming)
- Month summary chips — **type-aware**:
  - Daily: ✓ N done / ✗ N missed / N% rate / N days left
  - Weekly: ✓ Nw done / ✗ Nw missed / N% rate / Target: N×/wk
  - Monthly: ✓ N/target done OR ✗ N/target done OR N/target so far…
- Single `.cal-card` with gradient background, `--accent` set — contains 7-column `cal-grid`
- Day cells: **type-aware** — for weekly/monthly habits, days where the period completion count ≥ target show as `.rest` (neutral grey) instead of `.missed` (red)

**Stats tab (4th nav tab — bar chart icon):**
- 🏠 home button in page header
- **4 headline cards (2×2 grid):**
  - 🏆 Best Streak Ever (longest consecutive day run, any habit)
  - ✅ Total Completions (all habits, all time)
  - 🎯 Most Consistent (habit name + completion rate since creation)
  - 📅 Active Days (unique days with ≥1 completion)
- **Completion — Last 8 Weeks** bar chart: 8 vertical bars, oldest left → this week right; this week highlighted pink→purple; % label above each bar; date label below
- **Habit Breakdown** list: one card per habit showing 🔥 current streak (correct unit), 🏆 all-time best day streak, ✓ total completions

---

## Key design decisions

| Decision | Detail |
|---|---|
| Font | `Plus Jakarta Sans` (Google Fonts, weights 400–900) — loaded via CDN |
| Body background | `linear-gradient(150deg, #fdf4ff, #eef2ff, #f0fdfa)` fixed |
| Mobile-first sizing | Calendar day cells: `width: min(40px, 100%); aspect-ratio: 1` (fills column, caps at 40px). Weekly squares: fixed 26px |
| Per-habit colour | `--accent` CSS var on parent; children use `var(--accent, fallback)`. Richer 8-pair PALETTE with more saturated gradient stops |
| Missed days = solid red | `background: #ef4444` — hardcoded, must NOT inherit `--accent` |
| Rest days (weekly/monthly) | `background: rgba(0,0,0,0.04)` — neutral, not red; shown when period target already met by another completion |
| Done days = solid accent | `background: var(--accent)` |
| Glassmorphism elements | Week strip, wk-nav, month-nav-bar, cal-habit-info use `rgba(255,255,255,0.75–0.82)` + `backdrop-filter: blur()` + `border: 1px solid rgba(255,255,255,0.95)` |
| Bottom nav active state | Color `#7c3aed` + pill background (`rgba(124,58,237,0.1)`, 44×34px, border-radius 12px) via `::before` pseudo-element |
| Shared emoji picker | Single `#ep-sheet` bottom sheet + `#ep-overlay`. `pickerTarget` tracks `'add'` or habit ID |
| Settings sheet | Separate `#settings-sheet` + `#settings-overlay` — opened by ⋯ button on home header |
| Calendar week starts Sunday | `CAL_LABELS = ['Su','Mo','Tu','We','Th','Fr','Sa']`, but week strip and weekly tab are Mon–Sun |
| No PRF/perfect badge | Removed — `isPerfect()` still defined but unused |
| Drag reorder | SortableJS with `delay: 400, delayOnTouchOnly: true`. `filter: '.habit-section-label'` prevents section labels from being dragged. `onEnd` uses DOM query of all `[data-id]` cards to rebuild habits array in new order, then re-renders after 80ms |
| Media query | `@media (max-width: 480px)` scales down page headers, calendar nav, picker items, weekly rows, habit cards |
| Habit type | Stored as `type: 'daily' | 'weekly' | 'monthly'`. Migration adds `'daily'` to existing habits. Type picker shown in add form; type chip (☀️/📅/🗓) on each card lets user change type without deleting history |
| Habit target | Stored as `target: number` (default 1). Daily ignores it. Weekly/monthly: "done" means completions ≥ target for the period. Set in add form (target picker row) or via N× chip on card (opens target sheet) |
| Streak per type | Daily = consecutive days; Weekly = consecutive weeks where completions ≥ target; Monthly = consecutive months where completions ≥ target. Badge unit changes to d/w/m accordingly |
| Home week strip | Tappable days — selected day gets full purple dot; today shows light purple tint when not selected. Swipe left/right to navigate weeks. ✓ button targets selected day |
| Home filter tabs | All / Daily / Weekly / Monthly — filters list AND sets pendingType for new habits. Target picker row appears for Weekly/Monthly |

---

## Current state

**Fully implemented and working:**
- Add / delete habits
- Quick-complete from home card — targets the selected day in the week strip (not always today)
- Toggle any past date from inline or calendar tab grid
- Streak calculation — daily (consecutive days), weekly/monthly (consecutive periods where completions ≥ target)
- Auto-emoji by keyword + manual emoji picker override
- Per-habit colour palette (cycles through 8 richer colours)
- Home inline expandable calendar with per-habit month nav
- Weekly tab with per-habit coloured rows, week navigation, and 4 stats
- Calendar tab with shared grid + habit picker row + month nav + responsive day cells
- Month summary chips — type-aware and target-aware (daily/weekly/monthly)
- Bottom sheet emoji picker (works for both add flow and editing existing habits)
- Missed days shown as solid red; rest days (period completions ≥ target) shown as neutral grey
- 🏠 Home button on Weekly, Calendar and Stats tab headers
- **Visual redesign:** Plus Jakarta Sans font, richer gradients, glassmorphism, colored shadows, active nav pill, polished micro-interactions
- **Mobile responsive:** Media query for ≤480px, responsive calendar day cells
- **Data backup & restore:** Export habits as JSON / import from JSON file — via ⋯ menu on home screen
- **Drag-to-reorder habits:** Long-press a habit card to drag it to any position (SortableJS); works with type-grouped sections
- **Toast notifications:** Brief confirmation messages for backup/restore and type/target changes
- **All-time Stats tab:** 4 headline cards + Last 8 Weeks bar chart + per-habit breakdown with correct streak units
- **Habit frequency type:** Daily / Weekly / Monthly — filter tabs on home, type chip on each card, habits grouped by type, all logic type-aware
- **Per-habit targets:** Weekly/monthly habits have a custom target count (default 1); set in add form via target picker row; edit on card via N× chip → target sheet; all streak/calendar/rest-day/summary logic respects the target
- **Home week strip navigation:** Tap any day to select it (purple dot follows); swipe left/right to navigate weeks; ✓ button marks the selected day

**Git:** Initialised. Commits so far:
1. `Initial commit - Habits Tracker original version before redesign`
2. `Update CLAUDE.md and context.md with session rules and latest progress`
3. `Redesign UI and add backup, drag-reorder features`
4. `Add all-time stats tab and habit frequency types (Daily/Weekly/Monthly)`
5. `Update CLAUDE.md to reflect current architecture and features`
6. `Add end-of-session checklist rule to CLAUDE.md`
7. `Add /end custom command for end-of-session checklist`
8. `Add home week strip navigation, day selection, filter tabs, and per-habit targets`
9. `Update CLAUDE.md and context.md — end of session`
10. `Add MEMORY.md and update /end command commit conventions — mid-session checkpoint`

**Custom commands:** `.claude/commands/end.md` — type `/end` to automatically commit all changes, update both `context.md` and `CLAUDE.md`, and confirm all 3 are done. Mid-session commits use `"— mid-session checkpoint"`; only `/end` uses `"— end of session"`.

**Project files:** Three required files now in place — `CLAUDE.md` (rules & architecture), `CONTEXT.md` (current state & next steps), `MEMORY.md` (lessons learned & key decisions).

**Global CLAUDE.md:** Created at `~/.claude/CLAUDE.md` with Rachel's global preferences, project standards, design standards, and session rules. Applies across all projects.

**MyApp (Expo):** Untouched boilerplate. Has 2 tabs (Home, Explore), `ThemedText`/`ThemedView` components, `useColorScheme` hook, `Colors`/`Fonts` constants, `ParallaxScrollView`, `HapticTab`, `IconSymbol`. Nothing custom built yet.

---

## What's left / in progress

Features agreed to build (in order):

| # | Feature | Status |
|---|---|---|
| 1 | Data backup & restore (export/import JSON) | ✅ Done |
| 2 | Drag-to-reorder habits (long-press) | ✅ Done |
| 3 | All-time stats view (longest streak ever, total completions, most consistent habit, 8-week trend chart) | ✅ Done |
| 4 | Per-habit frequency type (Daily / Weekly / Monthly — with type-aware streak, calendar, and grouping) | ✅ Done |
| 5 | Simple trend chart (last 8 weeks completion rate) | ✅ Done (included in Feature 3) |
| 6 | Dark mode toggle | ⬜ Pending |
| 7 | Habit notes (jot a note when marking done, visible on calendar) | ⬜ Pending |
| 8 | Browser reminders (daily push notification per habit) | ⬜ Pending |

Longer-term (not committed to):
- Port `habit-tracker.html` logic into `MyApp` Expo project as a native app
