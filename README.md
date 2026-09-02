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

## What lives where

Your workouts are stored in the browser's local storage on that device. They never travel
to the host, and a public repository does not expose them — but they also don't sync
between devices, and clearing browsing data wipes them.

Use **Routines → Backup** regularly:

- *Save backup file* / *Copy backup* writes everything out as JSON.
- *Restore from a backup* reads it back — this is also how you move your history over from
  the synced Claude version (export JSON there, paste it here).

This build ships with your two routines and no logged sessions, so nothing about your
training goes into a public repository. Restore a backup to bring your history in.

## Updating the app later

Replace the files, and bump `CACHE = "set-ledger-v1"` at the top of `sw.js` to `v2`, `v3`,
and so on. Without that bump, phones keep serving the cached copy. Your logged data is
untouched by updates.
