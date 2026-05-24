# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Session Rules

### Commit message conventions
- **Mid-session commits** (automatic, during work): always use `"<description> — mid-session checkpoint"`. This applies to all commits made proactively during a session — e.g. before `/compact`, after a feature lands, or any other automatic commit.
- **End-of-session commits**: only use `"end of session"` in a commit message when the user explicitly types `/end` or `"end of session"`. Never use this wording for any other commit.

### Before every `/compact`
Update both `CLAUDE.md` and `context.md` to reflect the latest changes, then commit both files using the mid-session checkpoint format before compacting.

### At the end of every session (`/end`)
Always confirm all 3 of the following before closing:
  1. ✅ All changes committed to git
  2. ✅ `context.md` updated with what was completed and what's still left to do
  3. ✅ `CLAUDE.md` updated to reflect any architecture, data model, or helper changes

Never end a session without completing all 3.
Use the `/end` custom command (`.claude/commands/end.md`) to automate all steps.

---

## Project files

Every session should keep these 3 files up to date:
- `CLAUDE.md` — rules and architecture
- `CONTEXT.md` — current state and next steps
- `MEMORY.md` — lessons learned and key decisions

## Projects in this directory

| Project              | Type                        | Entry point                      |
| -------------------- | --------------------------- | -------------------------------- |
| `index.html`         | Single-file HTML/CSS/JS app | Open directly in browser or via GitHub Pages |


---

## index.html (formerly habit-tracker.html)

A zero-dependency habit tracker — all HTML, CSS, and JavaScript in one file. No build step.

**To develop:** open `index.html` in a browser and reload after edits.
**Deployed at:** https://rachelhfteoh.github.io/habit-tracker/

### Data model

Persisted to `localStorage` under key `'ht_v1'`:

```js
{
  habits: [{
    id: string,          // uid()
    name: string,
    dates: string[],     // ISO date strings "YYYY-MM-DD" of completed days
    createdAt: string,   // "YYYY-MM-DD"
    colorIdx: number,    // index into PALETTE (0–7)
    emoji: string,       // may be absent; falls back to getHabitEmoji(name)
    type: string,        // 'daily' | 'weekly' (default: 'daily') — monthly removed
    target: number,      // completion target per period (default 1; daily ignores it)
    note: string         // personal "why" note (default '')
  }]
}
```

Migration in `load()` backfills missing `colorIdx`, `dates`, `type`, `target`, and `note` fields automatically. Also converts any legacy `'monthly'` type → `'daily'`.

### Architecture

All state lives in module-level JS variables; `render()` is a single function that rebuilds the entire UI on every state change.

Key state variables:
- `currentView` — `'home' | 'calendar'`
- `vYear`, `vMonth` — calendar tab month being viewed
- `selectedCalHabit` — habit ID selected in the calendar tab
- `pendingEmoji`, `pickerTarget` — emoji picker state (`pickerTarget` is `'add'` or a habit ID)
- `pendingType` — type selected in the add-form type picker (`'daily'` | `'weekly'`)
- `pendingTarget` — target count for the habit being added (default 1; reset after add)
- `homeFilter` — `'all' | 'daily' | 'weekly'` — filters home habit list AND sets `pendingType`
- `homeWeekOffset` — week nav offset for home week strip (0 = current week, -1 = last week, etc.)
- `selectedHomeDate` — ISO date string of the day selected in the home week strip (default: today)
- `detailId` — habit ID currently open in the detail sheet (null when closed)
- `detailName`, `detailType`, `detailTarget`, `detailNote` — working state for the detail sheet while open

Key helpers:
- `getWeekDays(weekOffset)` — returns 7 ISO date strings Mon–Sun for a given offset
- `getWeekStart(ds)` — returns the Monday ISO string for the week containing `ds`
- `getHabitEmoji(name)` — keyword-based emoji auto-assignment
- `calcStreak(dates)` — current daily streak (consecutive days ending today/yesterday)
- `calcStreakWeekly(dates, target)` — consecutive weeks where completions ≥ target; uses `Map<weekStart, count>`
- `calcStreakByType(habit)` — dispatches to correct streak fn, passing `habit.target`
- `streakUnit(habit)` — returns `'d'` or `'w'`
- `cycleHabitType(id)` — cycles Daily→Weekly→Daily, saves, re-renders
- `setHabitType(type)` — sets `homeFilter` + `pendingType`, re-renders
- `showHabitDetail(id)` — opens detail sheet; populates name, type, target, note fields
- `closeHabitDetail()` — closes detail sheet
- `setDetailType(type)` — toggles Daily/Weekly in detail sheet, shows/hides target row
- `changeDetailTarget(delta)` — +/− target stepper in detail sheet (1–7)
- `saveHabitDetail()` — saves name, type, target, note from detail sheet; validates name non-empty; shows "Saved ✓" toast
- `deleteHabitFromSheet()` — closes sheet, confirms, deletes habit and all history
- `changeHomeWeek(dir)` — navigates home week strip by ±1 week, updates `selectedHomeDate`
- `initWeekStripSwipe()` — attaches swipe (touch + mouse) listeners to week strip for week navigation
- `selectHomeDay(ds)` — sets `selectedHomeDate`, re-renders
- `PALETTE[colorIdx]` — `{ grad, accent }` pairs; `--accent` CSS var drives per-habit child coloring

### Views

**Home** — Week strip (Mon–Sun dots) at top; tap any past/today dot to select it. Swipe left/right to navigate weeks. Filter/type row (All / ☀️ Daily / 📅 Weekly) filters list and sets `pendingType`. Habit cards grouped into Daily / Weekly sections (or flat when filtered). Each card: two-column layout — left (emoji + name, wraps freely), right (complete button on top, streak badge below). Tap anywhere on card (except complete button) opens detail sheet. Long-press to drag-reorder (SortableJS).

**Calendar** — top emoji bubble picker selects active habit; single shared calendar grid. Day cells are type-aware: for weekly habits, days in a week where completions ≥ target show as `.day-cell.rest` (neutral) not red. Month summary chips adapt to daily/weekly logic.

**Habit Detail Sheet** — slides up when card is tapped. Contains: name input (editable, validated non-empty), type toggle (Daily/Weekly), target stepper (weekly only, 1–7×/week), "Why is this habit important to me?" textarea. Save + Delete buttons at bottom. Delete closes sheet first, then confirms before deleting.

### Styling conventions

- Per-habit color: set `--accent` on a parent container; child elements reference `var(--accent, <fallback>)`.
- Missed days: solid red `#ef4444` — must NOT inherit from `--accent`.
- Rest days (weekly period met): `background: rgba(0,0,0,0.04)` — neutral grey.
- Done days: solid `var(--accent)`.
- Mobile-first sizing: `width: min(40px, 100%); aspect-ratio: 1` for calendar day cells.
- Card layout: `.card-body` (flex row) → `.card-left` (emoji + name, wraps) + `.card-right` (complete btn + streak, stacked column).
- PWA: `viewport-fit=cover` + `apple-mobile-web-app-capable` + `apple-mobile-web-app-status-bar-style: black-translucent`. Nav uses `env(safe-area-inset-bottom)`. App padding-bottom uses `calc(96px + env(safe-area-inset-bottom, 0px))`.

---

## MyApp (Expo React Native)

Standard `create-expo-app` scaffold — file-based routing via `expo-router`.

### Commands (run from inside `MyApp/`)

```bash
npm install          # install deps
npx expo start       # start dev server (scan QR for Expo Go, or press i/a for simulator)
npx expo start --ios
npx expo start --android
npx expo start --web
npm run lint         # ESLint via expo lint
```

### Architecture

- **Routing**: file-based via `expo-router`. `app/(tabs)/` holds tab screens; `app/_layout.tsx` wraps with `ThemeProvider` + `Stack`.
- **Path alias**: `@/` maps to the project root (configured in `tsconfig.json`).
- **Theming**: `Colors` from `constants/theme.ts` + `useColorScheme` hook. Light/dark values accessed as `Colors[colorScheme].tint` etc.
- **Platform-specific files**: `.ios.tsx` suffix used for iOS-only implementations (e.g. `components/ui/icon-symbol.ios.tsx` vs `icon-symbol.tsx`).
- **Icons**: `IconSymbol` wraps SF Symbols on iOS and a fallback on other platforms.
- **Haptics**: `HapticTab` component wraps tab bar buttons for tactile feedback.
