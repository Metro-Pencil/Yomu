# Yomu — a private manga reader PWA

Everything lives in your browser's own storage on your device. Nothing is
uploaded anywhere — there's no backend at all, which is why this can run
straight off GitHub Pages for free.

## What it does

- **Upload** `.zip` / `.cbz` (optionally password-protected) or `.pdf` chapter
  files. Drag-and-drop or pick from the file browser.
- Reads each **file name** to work out the series and chapter number
  automatically (handles things like `One Piece - Chapter 1050.zip`,
  `Naruto_070.5.cbz`, `[Group] Chainsaw Man - Ch 12 (2024).zip`, `Berserk Ch
  364.zip`, etc).
- **Next chapter** jumps to the next-highest chapter number for that series —
  it copes fine with gaps (e.g. you have 1, 2, 4 but not 3).
- Remembers **exactly where you left off**, per chapter and per series, and
  shows a "Continue reading" shelf on the library home screen.
- **Export/repack** any chapter or an entire series back into a `.zip`, with
  an optional new password.
- **Delete** chapters or whole series, with confirmation.
- Paged or long-strip (webtoon-style) reading modes, page slider, keyboard
  shortcuts, double-tap zoom, auto-hiding chrome.
- Dark and light themes (plus "match system"), a storage-usage view, and a
  "keep this persistent" button so the browser is less likely to clear your
  library under storage pressure.
- Installable as a PWA (Android/desktop Chrome & Edge "Install app"; iOS
  Safari "Add to Home Screen"), and works offline after the first visit.

## Deploying to GitHub Pages

1. **Unzip** this archive somewhere on your computer. You should see
   `index.html`, `manifest.json`, `service-worker.js`, an `icons/` folder and
   a `vendor/` folder — don't rename or nest these, GitHub Pages needs to see
   `index.html` at the root of what it serves.
2. Create a new **public** repository on GitHub (any name — e.g. `yomu`).
3. Upload the *contents* of the unzipped folder to the repo root — either:
   - On the repo's GitHub page: **Add file → Upload files**, then drag in
     `index.html`, `manifest.json`, `service-worker.js`, the whole `icons`
     folder and the whole `vendor` folder, and commit; or
   - With git: `git add . && git commit -m "Add Yomu" && git push`
4. In the repo, go to **Settings → Pages**. Under "Build and deployment",
   set **Source: Deploy from a branch**, branch **main**, folder **/ (root)**,
   then **Save**.
5. GitHub gives you a URL like `https://yourname.github.io/yomu/` — it takes
   a minute or two to go live the first time. Open it.
6. On your phone: open that URL in Chrome (Android) or Safari (iOS), then use
   **"Install app"** / **"Add to Home Screen"** so it launches full-screen
   like a native app.

That's it — no build step, no server, no config. Every path in this project
is written relatively so it works whether it's served from a domain root or
from a GitHub Pages *project* URL (`username.github.io/repo-name/`).

## Notes on how it works

- **Storage**: page images are kept in IndexedDB; your library list and
  reading progress are kept in `localStorage`. Both are scoped to the exact
  URL you install from, so if you ever move to a different repo name/URL your
  library won't follow automatically.
- **Password-protected archives**: handled by
  [zip.js](https://gildas-lormeau.github.io/zip.js/), loaded from a CDN at
  runtime (needs internet the first time; cached by the service worker for
  offline use afterward). The password is only ever needed once, at upload
  time — after that the pages are stored unlocked in your own IndexedDB, so
  re-reading or jumping to the next chapter never asks again.
- **PDF chapters** are rendered with [pdf.js](https://mozilla.github.io/pdf.js/),
  which is bundled directly in `vendor/pdfjs/` so that feature works fully
  offline with no CDN dependency.
- Filename parsing is a best-effort heuristic. If a file doesn't match a
  recognizable "name + chapter number" pattern it's filed under its full name
  as a one-chapter series — you can rename the series afterward from its menu.

## A known limitation

There's currently no way to manually move a chapter to a different series or
correct a misdetected chapter number from within the app — if a file gets
parsed wrong, the simplest fix is to delete it and re-upload with a clearer
file name. Happy to add an edit option for this if it turns out to matter in
practice.
