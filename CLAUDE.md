# Set Ledger

A workout tracker PWA. The entire app is one file — `index.html` holds all markup, CSS,
and JavaScript (a single IIFE, no build step, no dependencies, no framework). `sw.js`
caches it for offline use; `manifest.webmanifest` and `icons/` make it installable.

## Working agreement

**REQUIRED: No code changes without Derek's consent first.** Propose what will change and
why, and wait for approval before editing any file. Investigating, reading code, and
answering questions never need approval — edits always do.

## Deploying

Hosted on GitHub Pages from this repo's `main` branch (root folder). Deploy = commit and
push; Pages rebuilds in about a minute. Phones pick up a new version on the *second*
launch (the service worker background-fetches on the first).

**REQUIRED: bump `CACHE = "set-ledger-vN"` at the top of `sw.js` in every commit that
changes `index.html`** — without the bump, installed phones keep serving the old cached
copy indefinitely.

## UI conventions

- The tab bar stays at exactly four tabs: Log, History, Progress, Routines. Do not add a
  fifth. Rarely-used surfaces go behind the circled-i info button in the header (see the
  `showHelp()` / `#panel-help` pattern) or nest inside an existing tab.
- Small per-item editors (e.g. the exercise note) open in the native `<dialog>` via
  `showModal()` — see `showNotes()` / `#noteDialog`. Values that change on every
  keystroke are patched in place by id (see `updateTallies()` / `updateVolumes()`),
  never by re-rendering, so the focused field keeps focus.
- Rendering is manual: mutate `state`, then call `render()` (or the tab-specific
  `renderX()`). DOM is built with the `el()` helper — no innerHTML for dynamic content.
- Match the existing style: condensed uppercase headings, `.card` containers, `.btn`
  variants, and accessible names/roles on interactive elements.

## Data rules

- All user data lives in localStorage (`setledger.v1`); last-backup time in
  `setledger.lastBackup`. Nothing syncs, and no user data may ever enter this repo.
- Backups are full JSON snapshots. **Restore must merge by ID and never delete** —
  restoring an old backup must not wipe newer sessions. CSV export is analysis-only.
- Sessions and routines are identified by `id` and versioned by `updatedAt`; when
  merging, the newer `updatedAt` wins.

## Verifying changes

No test suite. Minimum bar: extract the inline scripts and syntax-check with
`node --check`, and sanity-test any storage/merge logic with a standalone node script
before pushing — the deploy target is someone's phone with real training history.
