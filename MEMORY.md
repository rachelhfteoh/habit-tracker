# MEMORY.md — Habit Tracker

Lessons learned, key decisions, and things to remember for future sessions.

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
- **Fields migrated so far:** `completedDates` → `dates`, `colorIdx`, `type`, `target`.

### Reuse the bottom sheet pattern for new UI
- **Decision:** All modal/overlay interactions use the same bottom sheet pattern (`ep-sheet` style).
- **Elements so far:** `#ep-sheet` (emoji picker), `#settings-sheet` (data management), `#target-sheet` (target edit).
- **Pattern:** sheet element + overlay element + `show` class toggle + close on overlay tap.

### Use `Map<period, count>` not `Set<period>` for weekly/monthly logic
- **Decision:** When checking if a weekly/monthly period is "done", count completions and compare to target — don't just check presence.
- **Before:** `Set<weekStart>` — any completion = done.
- **After:** `Map<weekStart, count>` — `count >= target` = done.
- **Applies to:** `calcStreakWeekly`, `calcStreakMonthly`, `calGridHtml` rest-day logic, weekly grid `weekMet`/`monMet`.

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
- **Decision:** The All / Daily / Weekly / Monthly filter row on home does two things: (1) filters the visible habit list, and (2) sets `pendingType` for the next habit to be added.
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
- **`isPerfect()` is defined but unused.** The PRF badge was removed. Don't accidentally reference it.
- **Calendar week starts Sunday, but week strip and weekly tab are Mon–Sun.** `CAL_LABELS = ['Su','Mo','Tu','We','Th','Fr','Sa']`. Don't "fix" this — it's intentional for the calendar grid layout.
- **`target` field is ignored for daily habits.** Don't add target UI to daily habit cards. The data field exists (defaulting to 1) but daily logic never reads it.
