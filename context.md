# Project Context

## Current Status
Single-file HTML habit tracker — v1.0 nearly done, 1 feature remaining.

## In Progress
Nothing in progress — ready to start next feature.

## Up Next
1. Browser reminders — daily push notification per habit (deferred to native app phase; skip for HTML version)

## Backlog
- Port habit-tracker.html logic into Expo React Native 
  as a native app (longer term, not committed to yet)

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

## Architecture Reference
- Single file: habit-tracker.html (no build tools)
- All state in module-level JS variables
- render() rebuilds entire UI on every state change
- Data persisted to localStorage key 'ht_v1'
- External deps: SortableJS 1.15.3 (CDN), 
  Google Fonts Plus Jakarta Sans (CDN)
