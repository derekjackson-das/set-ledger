# Set Ledger — installable app

Four files that turn Set Ledger into a real app on your phone: its own icon, no browser
chrome, and it opens with no signal.

```
index.html              the whole app
manifest.webmanifest    name, icon and "open like an app" settings
sw.js                   service worker — caches the app for offline use
icons/                  192px, 512px and Apple touch icons
```

Chrome will only install a page served over **https**, which is why opening the file
directly (`file://…`) gets you "not secure" and no install option. Put these files on any
https host and the install prompt appears.

---

## Deploy on GitHub Pages (free, about five minutes)

1. Create a new repository — call it `set-ledger`.
2. Upload the contents of this folder to the repository root (`index.html` at the top
   level, `icons/` beside it). Drag-and-drop into the GitHub web uploader works.
3. **Settings → Pages → Build and deployment**, set Source to *Deploy from a branch*,
   branch `main`, folder `/ (root)`. Save.
4. Wait a minute, then open `https://<your-username>.github.io/set-ledger/` on your phone.

Everything uses relative paths, so it works in a subfolder — no configuration needed.

Any other static host works the same way: Netlify, Cloudflare Pages, Vercel, or web space
you already have. Just keep the four items together in one directory.

## Install it on the phone

- **Android / Chrome:** open the URL, then `⋮` → **Install app** (or *Add to Home screen*).
  You get an icon that launches without browser chrome.
- **iPhone:** you must use **Safari** — Chrome on iOS can't install web apps. Open the URL
  in Safari, tap Share → **Add to Home Screen**.

After the first load the service worker has cached everything, so it opens in airplane mode.

---

## Using the app

Four tabs: **Log**, **History**, **Progress**, **Routines**.

### Log a workout

- Tap a routine to start it, or **Start an open session** to build the workout as you go.
  An open session can be saved as a routine when you finish, so it's an easy way to create one.
- Each set has a weight and reps field. The grey placeholder in an empty field is **what you
  did last time** for that same set of that same exercise — beat it or match it.
- Tap the circle at the end of a set row to mark it as your **top set**; it's highlighted in
  the log, flagged in History, and drives the Progress chart.
- Adding an exercise suggests names from everything you've logged before, so spellings stay
  consistent and your history for that exercise stays connected.
- The running **volume / sets / exercises** tallies update as you type. When you're done,
  **Finish session** files it under History; **Discard** throws it away.
- One session can be in progress at a time, and it survives closing the app — reopen and
  it's still there on the Log tab.

### Set / rest timer

While a session is open, a timer bar sits in the header — start it for a set or a rest
period, pause it, reset it. It works off real timestamps, so switching apps or locking the
phone never drifts it.

### History and Progress

- **History** lists every finished workout — tap one to see all sets (top sets marked ◉).
- **Progress** charts one exercise at a time: top-set weight and estimated one-rep max
  (Epley: weight × (1 + reps ÷ 30)), plus total volume per session. Charts appear once
  you've logged an exercise in a couple of sessions.

### Routines

Build and edit routines on the Routines tab. Deleting a routine never touches the sessions
you already logged from it.

## Keeping your data safe

Your log lives in the browser's local storage on the phone — nothing syncs anywhere. The
app has several layers of protection, but **the backup file is the only copy that is truly
yours**: it lives in your Files/Downloads, outside the browser, where restarts, cleared
browsing data, and even uninstalling can't touch it.

- **After every finished session** the app asks *"Would you like to back up this workout?"*
  — one tap saves the whole log as a JSON file.
- **The header shows a running count** of sessions logged since your last backup. Tap it to
  jump to the backup buttons. It disappears once you're backed up.
- **Backups are full snapshots.** Every backup contains your entire history, so you only
  ever need the most recent file.
- **Restore merges — it never deletes.** Restoring a backup adds whatever the phone is
  missing and keeps everything already there, so restoring an old file can't wipe new
  sessions. Where a session exists in both, the more recently edited copy wins.
- **The app asks the OS for persistent storage** on startup, which tells iOS/Android not to
  evict its data under disk pressure or after inactivity. This helps, but it's a request,
  not a guarantee — hence the backups.
- **Save CSV** exports every set as a spreadsheet row if you want to analyze your training
  elsewhere. (CSV is for analysis only — restore needs the JSON backup.)

Worth knowing on iPhone: the home-screen app and the Safari tab keep **separate storage**.
If your data seems to have vanished, check the other one before assuming it's gone — and
always log in the same place.

## Privacy

Your workouts never travel to the host, and a public repository does not expose them — the
repo holds only the app itself. This build ships with two starter routines and no logged
sessions. Restore a backup (Routines → *Restore from a backup*) to bring history in from
another device, or to move over from the synced Claude version (export JSON there, paste it
here).

## Updating the app later

Replace the files, and bump `CACHE = "set-ledger-v1"` at the top of `sw.js` to `v2`, `v3`,
and so on. Without that bump, phones keep serving the cached copy. Your logged data is
untouched by updates.
