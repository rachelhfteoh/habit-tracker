# Project Context

## Current Status
Single-file HTML habit tracker — v1.0 complete. Native app direction is TBD (see MEMORY.md).

## In Progress
Nothing in progress.

## Up Next
1. Decide on native app approach — React Native/Expo was started then abandoned this session; reconsider in next session

## Backlog
- Browser reminders — daily push notification per habit (handle in native app)
- Native iOS app port — direction TBD

## Completed — Archive
✅ Core habit tracking — add, delete, complete habits
✅ Data model — localStorage, migration pattern in load()
✅ Streak calculation — daily and weekly with targets
✅ Auto-emoji + manual emoji picker override
✅ Per-habit colour palette — 8 colour PALETTE cycles
✅ Home tab — week strip, filter tabs, habit cards, inline calendar
✅ Weekly tab — habit x day grid, week nav, 4 summary stats
✅ Calendar tab — shared grid, habit picker, month nav, type-aware day cells
✅ Stats tab — built then removed (data not meaningful after type split)
✅ Visual redesign — Plus Jakarta Sans, glassmorphism, gradients
✅ Mobile responsive — media query for ≤480px
✅ Data backup & restore — export/import JSON via ⋯ menu
✅ Drag-to-reorder habits — long-press SortableJS
✅ Toast notifications — confirmations for key actions
✅ Habit frequency types — Daily/Weekly only (Monthly removed)
✅ Per-habit targets — weekly custom target count (1–7×/week)
✅ Home week strip navigation — tap day, swipe weeks
✅ Dark mode — attempted and fully reverted (see MEMORY.md)
✅ Target picker row removed — manage targets via habit detail sheet instead
✅ Habit detail sheet — tap habit name → edit type, target, and "why" note in one place
✅ Bug fixes — calendar/weekly sync for retroactive completions; correct streak unit in calendar tab; removed redundant target sheet

## Architecture Reference
- Single file: habit-tracker.html (no build tools)
- All state in module-level JS variables
- render() rebuilds entire UI on every state change
- Data persisted to localStorage key 'ht_v1'
- External deps: SortableJS 1.15.3 (CDN), 
  Google Fonts Plus Jakarta Sans (CDN)