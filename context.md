# Project Context

## Current Status
Single-file HTML habit tracker — deployed to GitHub Pages at https://rachelhfteoh.github.io/habit-tracker/. Cards redesigned, weekly tab removed, app simplified for consistent mobile/web experience.

## In Progress
Nothing in progress.

## Up Next
1. Review the app on iPhone after latest changes and fine-tune any remaining visual inconsistencies

## Backlog
- Browser reminders — daily push notification per habit
- Native iOS app port — direction TBD (see MEMORY.md)

## Completed — Archive
✅ Core habit tracking — add, delete, complete habits
✅ Data model — localStorage, migration pattern in load()
✅ Streak calculation — daily and weekly with targets
✅ Auto-emoji + manual emoji picker override
✅ Per-habit colour palette — 8 colour PALETTE cycles
✅ Home tab — week strip, filter tabs, habit cards
✅ Calendar tab — shared grid, habit picker, month nav, type-aware day cells
✅ Stats tab — built then removed (data not meaningful after type split)
✅ Weekly tab — built then removed (not valuable enough to keep)
✅ Visual redesign — Plus Jakarta Sans, glassmorphism, gradients
✅ Mobile responsive — media query for ≤480px
✅ Data backup & restore — export/import JSON via ⋯ menu
✅ Drag-to-reorder habits — long-press SortableJS
✅ Toast notifications — confirmations for key actions
✅ Habit frequency types — Daily/Weekly only (Monthly removed)
✅ Per-habit targets — weekly custom target count (1–7×/week)
✅ Home week strip navigation — tap day, swipe weeks
✅ Dark mode — attempted and fully reverted (see MEMORY.md)
✅ Habit detail sheet — tap card → edit name, type, target, note; delete in sheet
✅ GitHub Pages deployment — renamed to index.html, live at github.io URL
✅ PWA meta tags — full-screen on iPhone home screen, safe-area-inset nav fix
✅ Long name truncation fix — min-width:0 fix across home, calendar views
✅ Habit name editing — editable in detail sheet with validation
✅ Card redesign — two-column layout (emoji+name left, complete+streak right), delete moved to sheet
✅ Card compacting — reduced padding, font sizes, button sizes for tighter layout
✅ Weekly tab removed — fully deleted (HTML, CSS, JS, nav button)

## Architecture Reference
- Single file: index.html (no build tools)
- All state in module-level JS variables
- render() rebuilds entire UI on every state change
- Data persisted to localStorage key 'ht_v1'
- External deps: SortableJS 1.15.3 (CDN), Google Fonts Plus Jakarta Sans (CDN)
- Deployed: https://rachelhfteoh.github.io/habit-tracker/
