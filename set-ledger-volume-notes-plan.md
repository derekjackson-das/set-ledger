# Set Ledger: Set Volume & Exercise Notes — Project Plan

**Created:** 2026-09-17

## Elevator Pitch
Two additions to the Log tab of Set Ledger, Derek's single-file workout PWA: a live
weight × reps volume readout for every set (with a per-exercise total), and a per-exercise
note that carries forward from workout to workout so setup details and cues are always at
hand without cluttering the card.

## Problem Statement
- **Volume is invisible below the session level.** The Log tab shows a session-wide
  Volume tally, but there is no way to see what a single set or a single exercise
  contributed. Derek wants to see "30 × 10 = 300 lb" on the row and "700 lb" for the
  exercise as he logs.
- **There is nowhere to write anything down.** Set Ledger stores only weight, reps and a
  top-set mark. Things like machine seat position, grip width, or "left knee tight last
  time" have to live in Derek's head. Because the app is used on a phone mid-workout, any
  notes surface has to cost almost no screen space.

## Proposed Solution

### 1. Per-set volume column and exercise total (Log tab)
- Move the column labels out of each set row into a single **header row** at the top of
  the exercise card's set list: set number, `LB`, `REPS`, `VOL`. This is visual only, not
  an HTML table.
- Remove the inline `lb` unit and `×` glyph from every row; that reclaimed width holds a
  new read-only **volume cell** (weight × reps, monospace, tabular numbers, right-aligned).
- Add an **exercise total** (sum of volume over filled sets) in the card footer, on the
  same line as "+ Add set", right-aligned.
- Volume cell and total update **in place on every keystroke**, the same way
  `updateTallies()` patches the session tallies today, so a full re-render never steals
  focus from the field being typed in.
- Sets with a missing weight or reps show `—` instead of a number.

### 2. Per-exercise notes behind a button, in a modal
- A small **notes button** joins the exercise card header (between the "best 40×10"
  hint and the remove ×). It shows a filled/dotted state when a note exists so Derek can
  see at a glance which exercises have something written.
- Tapping it opens a **native `<dialog>` shown modally** (focus trap, Escape to close,
  backdrop, focus returns to the button on close). It contains the exercise name, one
  textarea, a "Clear note" button, and "Done". The textarea autosaves on input, matching
  the app's everything-autosaves convention; there is no separate Save/Cancel.
- **Notes follow the exercise.** The note is stored on the session's exercise entry
  (`entry.note`). When a session starts from a routine, or an exercise is added
  mid-session, the entry's note is pre-filled with the most recent note logged for that
  exercise name (case-insensitive match, same rule `lastPerformance()` uses). Derek can
  keep it, append to it, or clear it and write fresh. Whatever is in the field is what
  that session records.
- Notes ride along in backups automatically (full snapshot) and merge-by-ID restore needs
  no change; older entries simply have no `note` field.

### 3. History tab
- The expanded session table gains a **Total** column with each exercise's volume.
- An exercise's note (if any) appears as a muted line under its name in the same row.

### 4. Docs and deploy
- Help page (`renderHelp()`) and README gain short entries for volume and notes.
- `sw.js` CACHE bumps from `set-ledger-v4` to `set-ledger-v5` in the same commit.

## Goals & Success Metrics
- Every set row on the Log tab shows its volume, updating as Derek types, with no lost
  focus or caret jumps.
- Each exercise card shows a correct total (sum of filled sets only).
- Set rows fit without wrapping or horizontal overflow at 375 px wide (iPhone standard)
  and are checked at 320 px (iPhone SE).
- A note written on leg press today appears pre-filled the next time leg press is logged
  from any routine or open session.
- Screen reader users hear an unambiguous name for every weight, reps and volume cell,
  and for the notes button (including the exercise name).
- Existing sessions from before this change render and restore unchanged (verified with a
  standalone node script against a synthetic backup).
- Deployed to GitHub Pages and confirmed on Derek's phone on second launch.

## Stakeholders
- **Derek Jackson** — owner, sole user, and approver of every code change (per the
  project working agreement: no edits without consent).
- **Claude** — implements under that agreement, proposing each change before editing.

## Functional Requirements

- **Must have:**
  - Header row per exercise card with `#`, `LB`, `REPS`, `VOL` labels; inline `lb`/`×`
    removed from set rows.
  - Read-only volume cell per set: `weight × reps`, formatted with the existing `fmt()`
    helper; `—` when either value is empty.
  - Exercise total in the card footer, live-updated with the volume cells.
  - Notes button on the exercise card header with a visible "has note" state and an
    accessible name of the form "Notes for Leg press".
  - Modal `<dialog>` with textarea (autosave to `entry.note`), "Clear note", "Done".
  - `lastNote(name, excludeId)` lookup that returns the most recent non-empty note for an
    exercise name across `allSessions()`; used when starting a session and when adding an
    exercise mid-session.
  - History detail: Total column and note line per exercise.
  - `sw.js` CACHE bump to v5.
  - Help page and README updated.

- **Should have:**
  - Accessible names for inputs preserve the existing "last time N" context (see Open
    Questions for the aria approach).
  - Notes dialog textarea gets focus on open; Done/Escape return focus to the notes button.
  - Top-set highlighting (`is-top` row styling) continues to work unchanged with the new
    layout.

- **Nice to have:**
  - `notes` column in CSV export.
  - Per-set volumes listed in History (currently only the exercise total is planned there).

## Non-Functional Requirements
- **Single file, no dependencies.** All changes live in `index.html` (plus the one-line
  `sw.js` bump); no build step, no framework, DOM built with `el()`, no innerHTML.
- **Style match.** Condensed uppercase header labels, `.card` containers, existing `.btn`
  variants, monospace tabular numbers for the volume cell to match `.detail td.sets`.
- **Accessibility.** Every interactive element keeps a name and role; the dialog uses the
  native element so focus management is correct by default. Run the accessibility review
  agent on the changed markup before deploy.
- **Performance.** Volume updates patch existing DOM nodes by id; no re-render on input.
- **Data safety.** No schema migration. Restore-merge semantics untouched. No user data
  enters the repo (test fixtures are synthetic).
- **Verification bar (from CLAUDE.md).** Extract the inline script and `node --check` it;
  standalone node test for `lastNote()` and volume/total math against old-shape and
  new-shape session data.

## Out of Scope
- Notes on the Routines tab (routine templates do not carry notes; carry-forward is by
  exercise name from logged sessions).
- Per-set notes or a session-level note.
- Editing notes from the History tab (read-only there).
- Per-exercise volume charts on the Progress tab.
- CSV changes (listed as nice-to-have only).
- Any fifth tab. Notes live in a dialog; nothing joins the tab bar.

## Risks & Assumptions
- **Row width on small phones.** Even with `lb` and `×` removed, a row holds a set
  number, two inputs, the volume cell, the "top" toggle and a delete button. Estimated
  fit at 375 px is tight (roughly 350 px of a 351 px content width). Mitigation: shrink
  the "top" toggle padding or the weight/reps inputs by a few pixels; verify at 375 and
  320 px before approval.
- **Carry-forward copies notes into every session.** With pre-fill at session start, a
  note written once will appear on every later session of that exercise in History, even
  if Derek never opened the dialog that day. This is the intended "persistent" behaviour
  but it means History can't distinguish "wrote this today" from "carried over". See Open
  Question 1 for the alternative.
- **Renaming an exercise breaks the chain.** Carry-forward matches by exercise name, same
  as the grey "last time" placeholders. A renamed exercise starts with no note.
- **Assumes `<dialog>` support.** Native modal dialogs have been in iOS Safari since 15.4
  (2022) and in all current desktop browsers. No polyfill planned.
- **Assumes per-exercise totals should sum filled sets only**, ignoring rows with an
  empty weight or reps, mirroring how `sessionVolume()` behaves today. [TBD — not
  discussed explicitly; treated as the natural default.]

## Phases / Rough Timeline
Each phase is one proposal → approval → edit → check cycle; all phases can ship in a
single commit or be split, Derek's call.

1. **Volume column.** Header row, remove inline `lb`/`×`, volume cell per set,
   `updateVolumes()` live patching, CSS for the new column. Check widths at 375/320 px.
2. **Exercise total.** Footer total on the Log tab; Total column in History detail.
3. **Notes.** `entry.note` field, `lastNote()` lookup, pre-fill in `startSession()` and
   the add-exercise path, notes button with state, `<dialog>` markup and `showNotes()`
   handler, note line in History detail.
4. **Wrap-up.** Help page and README text, CACHE bump to v5, `node --check`, standalone
   node test of `lastNote()` and totals, accessibility review, commit, push, confirm on
   phone.

## Decisions (resolved 2026-09-17)
1. **Carry-forward mechanism.** Copy the last note into `entry.note` when the session
   starts (and when an exercise is added mid-session). History shows exactly what Derek
   saw that day.
2. **Column label wording.** `VOL` for the per-set column; "Total 700 lb" in the footer;
   "Volume" stays the name of the session-level tally.
3. **Accessible naming of inputs.** Keep the existing `aria-label` on each input (it
   carries the "last time N" hint). The header row is visual reinforcement, `aria-hidden`.
4. **Notes button glyph.** Pencil (✎), with a dot state when a note exists.
5. **Inline note preview.** None. The note is only visible in the dialog and in History.

## Deferred
- **320 px row overflow.** At 320 px viewport width (iPhone SE class) a set row overflows
  its card by about 52 px, cutting off the "top" toggle and delete button. Measured
  2026-09-17 on both the committed v4 build and the phase 1 build: identical overflow, so
  it predates this work. Fix later, likely by stacking the toggle/delete under the inputs
  or shrinking the inputs below 360 px.

## Open Questions
None outstanding. Build proceeds phase by phase, each edit approved before it is made.
