# MEMORY.md — Habit Tracker

Lessons learned, key decisions, and things to remember for future sessions.

---

## Native App Decisions

### React Native / Expo port — started and abandoned
- **What happened:** Rachel asked to port the HTML app to iOS using Expo React Native. A full plan was presented and approved. The Expo scaffold was created (`create-expo-app@latest`, SDK 56) and dependencies installed, but Rachel changed her mind before any source code was written. The scaffold was deleted cleanly.
- **Why abandoned:** Rachel changed her mind mid-session — reason not given. No code was lost.
- **Key finding:** Expo SDK 56 uses a new template structure — files live under `src/app/` (not `app/`), and uses `NativeTabs` from `expo-router/unstable-native-tabs` instead of the classic `Tabs`. This is worth knowing if we revisit the native app.
- **Next time:** Before scaffolding, ask Rachel to confirm she is fully committed to the native app direction and understands it is a multi-session effort. Don't scaffold until she's sure.

---

## Process Lessons

### Always plan before implementing
- **Lesson:** Claude started implementing a feature mid-conversation without showing a plan first. Wasted tokens and led to rework.
- **Rule established:** Always present a plan and wait for confirmation before writing any code on complex tasks.
- **When this matters most:** Any feature touching multiple parts of the file (data model + streak logic + UI + render).

### Break big features into reviewable steps
- **Lesson:** Large features (e.g. per-habit targets) touch many areas of the file at once. Reviewing a plan upfront catches problems before code is written.
- **Rule:** For any feature that changes the data model, state, helpers, AND UI — plan it out section by section first.

### Commit before making major changes
- **Lesson:** Always have a clean commit to roll back to before attempting a large or risky change.
- **Applied:** Dark mode was a good example — having a clean commit made the full revert safe and easy.

---

## Technical Decisions

### Single-file architecture — keep it that way
- **Decision:** Stay with one `habit-tracker.html` file. No build tools, no npm, no bundler.
- **Reason:** Opens directly in mobile browser, no setup needed. Rachel is learning and simplicity matters.
- **Implication:** All CSS, JS, and HTML live together. Scroll to navigate; use line number comments to orient.

### `render()` rebuilds everything — lean into it
- **Decision:** Don't try to do targeted DOM updates. Call `render()` after every state change.
- **Reason:** Simpler mental model. The file is small enough that full re-renders are instant.

### Migration pattern in `load()` — always backfill
- **Decision:** When adding a new field to the data model, always add a backfill migration in `load()`.
- **Pattern:**
  ```js
  if (h.newField === undefined) { h.newField = defaultValue; dirty = true; }
  ```
- **Why:** Existing localStorage data won't have the new field. Without migration, old habits silently break.
- **Fields migrated so far:** `completedDates` → `dates`, `colorIdx`, `type`, `target`, `note`.

### Reuse the bottom sheet pattern for new UI
- **Decision:** All modal/overlay interactions use the same bottom sheet pattern (`ep-sheet` style).
- **Elements so far:** `#ep-sheet` (emoji picker), `#settings-sheet` (data management), `#target-sheet` (target edit), `#detail-sheet` (habit detail).
- **Pattern:** sheet element + overlay element + `show` class toggle + close on overlay tap.

### Use `Map<period, count>` not `Set<period>` for weekly logic
- **Decision:** When checking if a weekly period is "done", count completions and compare to target — don't just check presence.
- **Before:** `Set<weekStart>` — any completion = done.
- **After:** `Map<weekStart, count>` — `count >= target` = done.
- **Applies to:** `calcStreakWeekly`, `calGridHtml` rest-day logic, weekly grid `weekMet`.

### Monthly habit type removed
- **Decision:** Removed `'monthly'` type entirely. Only `'daily'` and `'weekly'` remain.
- **Why:** User doesn't track habits on a monthly basis — the type added complexity without value.
- **Migration:** Any existing `'monthly'` habits are auto-converted to `'daily'` in `load()`.
- **Removed:** `calcStreakMonthly`, monthly calendar logic, monthly filter tab, monthly CSS.

### Stats tab removed
- **Decision:** Stats tab (`currentView === 'stats'`) removed entirely.
- **Why:** User felt the data wasn't meaningful after habit types were split into daily/weekly.
- **Removed:** `renderStatsView`, `calcBestStreak`, all `.sts-*`, `.trend-*`, `.hb-*` CSS, nav button.

### Target chip removed — detail sheet is the single place for target editing
- **Decision:** Removed the `N×` target chip from weekly habit cards and its dedicated target sheet (`#target-sheet`).
- **Why:** The habit detail sheet already lets you edit the target. Having two paths (chip → target sheet, AND detail sheet) was redundant and confusing.
- **Removed:** `.target-chip` CSS, `#target-sheet` HTML, `targetEditId`/`sheetTargetValue` state vars, `showTargetSheet`/`closeTargetSheet`/`changeSheetTarget`/`saveTargetEdit` functions.
- **Still present:** The CSS classes `.target-sheet-stepper`, `.target-adj-btn`, `.target-sheet-count`, `.target-sheet-unit`, `.target-save-btn` — these are reused by the detail sheet stepper. Do NOT remove them.

### Habit detail sheet — single place to edit type, target, note
- **Decision:** Tapping a habit's name opens a detail bottom sheet with type toggle, target stepper, and "why" note.
- **Why:** Consolidates settings into one place rather than scattered chips/sheets on the card.
- **Tap zone:** Only the habit name `<div>` is the trigger — existing buttons (emoji, complete, delete, chips) are unaffected. Used `event.stopPropagation()` to prevent card-level conflicts.

### CSS specificity — source order matters for equal-specificity selectors
- **Bug:** `.ws-dot.selected` was defined before `.ws-dot.past` in the stylesheet. Both have the same specificity (two classes). `.ws-dot.past` (later in file) overrode `.ws-dot.selected` for past days, so the selected purple dot never appeared on past dates.
- **Fix:** Move `.ws-dot.selected:not(.today)` to after `.ws-dot.past` and `.ws-dot.future` in the CSS.
- **Lesson:** When styles aren't applying, check source order before assuming a specificity hierarchy issue.

### Swipe gestures instead of arrow buttons for navigation
- **Decision:** Home week strip uses swipe (touchstart/touchend + mousedown/mouseup) instead of `‹ ›` buttons.
- **Reason:** Cleaner mobile UI; no extra buttons cluttering the week strip.
- **Threshold:** 40px minimum swipe distance to avoid accidental triggers.
- **Note:** Weekly tab still uses `‹ ›` buttons — different context, different treatment is fine.

### Filter tabs are dual-purpose
- **Decision:** The All / Daily / Weekly filter row on home does two things: (1) filters the visible habit list, and (2) sets `pendingType` for the next habit to be added.
- **Why:** Reduces UI complexity — one row does both jobs. If you're looking at weekly habits, you probably want to add a weekly one.
- **Edge case:** "All" filter keeps whatever `pendingType` was last set; it doesn't reset it.

---

## Features That Were Tried and Reverted

### Dark mode — reverted entirely
- **Attempted:** Full dark mode toggle with CSS variables for background, cards, text, nav.
- **Problem:** The pastel gradient habit cards (which use hardcoded PALETTE colours) clashed badly with the dark background. Calendar day cells became hard to read. The per-habit gradient colours are too light for dark mode without a redesign of the entire palette.
- **Decision:** Reverted all dark mode code completely. Not worth the complexity until the colour system is rethought.
- **If revisiting:** Would need to either (a) redesign PALETTE with dark-mode-safe colours, or (b) add a separate dark PALETTE and swap at render time.

---

## UI/UX Decisions

### Today's dot is always dark purple; selected non-today gets light purple
- **Rule:** Two visual states on the week strip:
  - Today (regardless of selection): dark purple gradient dot — always anchors the user to now.
  - Selected non-today: light purple (`#ede9fe` background, `#7c3aed` text) — clearly selected but distinguishable from today.
- **Why:** User needs to always know where "today" is, even when they've navigated to another day.

### "Done Today" label stays as-is even when a past day is selected
- **Decision:** The stat card always says "Done Today" even when `selectedHomeDate` is a past day.
- **Reason:** The label reflects the card's purpose (track today's completions), not the selected day. Changing the label was more confusing than helpful.

### Header date always shows today, not the selected day
- **Decision:** The home tab header always shows the current date. It does not update to reflect `selectedHomeDate`.
- **Reason:** Keeps the user oriented to the present. The week strip itself clearly shows which day is selected.

---

## Things to Watch Out For

- **SortableJS + re-render timing:** After a drag-reorder, there's an 80ms delay before `render()` is called. This is intentional — SortableJS needs to finish its own DOM cleanup first. Don't remove this delay.
- **Calendar week starts Sunday, but week strip is Mon–Sun.** `CAL_LABELS = ['Su','Mo','Tu','We','Th','Fr','Sa']`. Don't "fix" this — it's intentional for the calendar grid layout.
- **`target` field is ignored for daily habits.** Don't add target UI to daily habit cards. The data field exists (defaulting to 1) but daily logic never reads it.
- **`note` field is set via the detail sheet only.** It is not shown on the habit card itself. Don't add it to the card render.
- **localStorage data is browser-specific.** Always open in Safari — data saved in Safari won't appear in Chrome and vice versa.
- **`done` check must precede `isPre` check in calGridHtml.** If the user retroactively marks a day done, `isPre = true` would shadow the `done` state. Always check `done` first.
- **`min-width: 0` is required on flex children that use text-overflow: ellipsis.** Without it, the element won't shrink below its natural content width, so ellipsis never triggers even with `overflow: hidden` set. Applies to `.card-name`, `.wk-name`, `.cal-habit-name`.
- **Card onclick + SortableJS:** Adding `onclick` to the card div is safe because SortableJS uses `delay: 400, delayOnTouchOnly: true`. A tap (< 400ms) fires onclick; a long-press starts the drag and SortableJS prevents the click from firing. Don't remove the delay from the Sortable config.

---

## Technical Decisions (this session)

### Weekly tab removed — card-level delete instead
- **Decision:** Removed the entire weekly tab (HTML, CSS, JS, nav button). Added Delete to the detail sheet instead of keeping a × button on cards.
- **Why:** Rachel felt the weekly tab wasn't valuable anymore after the card redesign removed quick-glance per-day data. Consolidating delete into the sheet reduces card clutter.
- **Removed:** All `wk-*` CSS, `view-weekly` HTML, `wkOffset` state, weekly render block, `wk-prev-btn`/`wk-next-btn` event listeners.

### Card redesign — two-column layout, tap-to-open-sheet
- **Decision:** Cards now have: left column (emoji + name, wraps freely), right column (complete button + streak badge, stacked). Tapping anywhere on the card (except complete button) opens the detail sheet.
- **Why:** Cleaner look, less chrome. The old card had 5+ interactive elements (emoji picker, name, streak, type chip, cal toggle, delete). Now just 1 (complete button); everything else goes through the sheet.
- **Emoji editing:** Removed from card entirely. Emoji is now display-only. Not added to sheet — Rachel chose option B (drop emoji editing for now).
- **Card onclick + stopPropagation:** Complete button uses `event.stopPropagation()` to prevent the card click from firing when marking done.

### GitHub Pages deployment
- **Decision:** Renamed `habit-tracker.html` → `index.html` so the URL is clean: `https://rachelhfteoh.github.io/habit-tracker/`.
- **Why:** GitHub Pages serves `index.html` as the default — no filename needed in the URL, cleaner for iPhone home screen.
- **PWA setup:** `viewport-fit=cover` is required for `env(safe-area-inset-bottom)` to work in standalone mode. Without it, the safe area inset returns 0 even on iPhone X+.
