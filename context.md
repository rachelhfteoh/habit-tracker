# Project Context

## Current Status
Single-file HTML habit tracker — deployed to GitHub Pages. PWA with home screen icon, safe-area insets fixed, calendar and home interactions polished this session.

## In Progress
Nothing in progress.

## Up Next
1. Browser reminders — daily push notification per habit

## Backlog
- Native iOS app port — direction TBD (see MEMORY.md)

## Completed — Archive
✅ Core habit tracking — add, delete, complete habits
✅ Data model — localStorage, migration pattern in load()
✅ Streak calculation — daily and weekly with targets; resets every Jan 1
✅ Auto-emoji + manual emoji picker override
✅ Per-habit colour palette — 8 colour PALETTE cycles
✅ Home tab — week strip, filter tabs, habit cards
✅ Calendar tab — shared grid, habit picker grouped by type, month nav, type-aware day cells
✅ Stats tab — built then removed (data not meaningful after type split)
✅ Weekly tab — built then removed (not valuable enough to keep)
✅ Visual redesign — Plus Jakarta Sans, glassmorphism, gradients
✅ Mobile responsive — media query for ≤480px
✅ Data backup & restore — export/import JSON via ⋯ menu
✅ Drag-to-reorder habits — long-press SortableJS
✅ Toast notifications — confirmations for key actions
✅ Habit frequency types — Daily/Weekly only (Monthly removed)
✅ Per-habit targets — weekly custom target count (1–7×/week)
✅ Home week strip navigation — tap day, swipe weeks, ‹ › buttons
✅ Dark mode — attempted and fully reverted (see MEMORY.md)
✅ Habit detail sheet — tap card → edit name, type, target, note; delete in sheet
✅ GitHub Pages deployment — renamed to index.html, live at github.io URL
✅ PWA meta tags — full-screen on iPhone home screen, safe-area-inset top+bottom fix
✅ Long name truncation fix — min-width:0 fix across home, calendar views
✅ Habit name editing — editable in detail sheet with validation
✅ Card redesign — two-column layout (emoji+name left, streak+complete right), delete moved to sheet
✅ Card compacting — reduced padding, font sizes; streak left of done button in a row
✅ Weekly tab removed — fully deleted (HTML, CSS, JS, nav button)
✅ Calendar picker grouped — Daily and Weekly sections with labels
✅ Calendar day colours — all day numbers black, no grey-out
✅ Calendar tap to mark done — all past days clickable including pre-creation dates
✅ Yearly streak reset — streaks cap at 365, reset to 0 on Jan 1 each year
✅ PWA apple-touch-icon — 180x180 black/purple icon, correct meta tag

## Architecture Reference
- Single file: index.html (no build tools)
- All state in module-level JS variables
- render() rebuilds entire UI on every state change
- Data persisted to localStorage key 'ht_v1'
- External deps: SortableJS 1.15.3 (CDN), Google Fonts Plus Jakarta Sans (CDN)
- Deployed: https://rachelhfteoh.github.io/habit-tracker/
- Icon: apple-touch-icon-v2.png (renamed to bust iOS cache)
