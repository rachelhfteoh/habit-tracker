# Project Context

## Current Status
Single-file HTML habit tracker — core features complete, 
3 features remaining before v1.0 is done.

## In Progress
Nothing in progress — ready to start next feature.

## Up Next
1. Dark mode toggle
2. Habit notes (jot a note when marking done, visible on calendar)
3. Browser reminders (daily push notification per habit)

## Backlog
- Port habit-tracker.html logic into Expo React Native 
  as a native app (longer term, not committed to yet)

## Completed — Archive
✅ Core habit tracking — add, delete, complete habits
✅ Data model — localStorage, migration pattern in load()
✅ Streak calculation — daily, weekly, monthly with targets
✅ Auto-emoji + manual emoji picker override
✅ Per-habit colour palette — 8 colour PALETTE cycles
✅ Home tab — week strip, filter tabs, habit cards, inline calendar
✅ Weekly tab — habit x day grid, week nav, 4 summary stats
✅ Calendar tab — shared grid, habit picker, month nav, type-aware day cells
✅ Stats tab — 4 headline cards, last 8 weeks bar chart, per-habit breakdown
✅ Visual redesign — Plus Jakarta Sans, glassmorphism, gradients
✅ Mobile responsive — media query for ≤480px
✅ Data backup & restore — export/import JSON via ⋯ menu
✅ Drag-to-reorder habits — long-press SortableJS
✅ Toast notifications — confirmations for key actions
✅ Habit frequency types — Daily/Weekly/Monthly, type-aware logic
✅ Per-habit targets — weekly/monthly custom target count
✅ Home week strip navigation — tap day, swipe weeks
✅ Dark mode — attempted and fully reverted (see MEMORY.md)

## Architecture Reference
- Single file: habit-tracker.html (no build tools)
- All state in module-level JS variables
- render() rebuilds entire UI on every state change
- Data persisted to localStorage key 'ht_v1'
- External deps: SortableJS 1.15.3 (CDN), 
  Google Fonts Plus Jakarta Sans (CDN)
