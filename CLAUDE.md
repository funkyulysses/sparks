# Spark — project context

Personal quick-capture journal, styled like a private Twitter/X timeline,
entirely client-side. Single self-contained HTML file — no build step, no
framework, no server, no accounts. Everything lives in the browser's
`localStorage` on-device.

## Where the files live

This project ended up in three synced locations (some sprawl from a
mid-project move — see "Known quirks" below):

1. **`~/Documents/web-projects/spark-site/spark.html`** — the working
   master copy on this Mac. Edit this one.
2. **`~/Documents/web-projects/spark-site/index.html`** — identical copy
   of the above, kept in sync manually. This is the exact file that gets
   uploaded to GitHub Pages.
3. **`~/Documents/web-projects/spark-site/sw.js`** — the offline service
   worker (see Deployment). Bump the `CACHE` constant inside it whenever
   `spark.html` changes, or returning visitors can get served a stale
   cached copy.
4. **`~/Library/Mobile Documents/com~apple~CloudDocs/Spark/spark.html`**
   — a copy in iCloud Drive, kept in sync manually. This is also where
   the *original* file ended up after it got moved/renamed mid-project
   (outside this tool's involvement).

**Workflow: after any edit, copy `spark.html` → `index.html` and → the
iCloud Drive copy, so all three stay identical.** Plain `cp`, no build
step exists.

There is no local git repo for this project.

## Deployment

Live on GitHub Pages, in a repo the user created via the GitHub web UI —
there's no `gh` CLI or stored credentials on the original Mac, so every
deploy is a manual drag-and-drop upload, not a `git push`. URL pattern:
`https://<their-username>.github.io/<repo>/` — confirm the exact URL
with the user; it isn't recorded here.

**To ship a change:** update `spark-site/index.html` (and `sw.js` if it
changed), then have the user go to the GitHub repo → **Add file →
Upload files** → drag both in → Commit. Pages auto-redeploys in ~1
minute.

On the user's iPhone, Spark is installed via **Add to Home Screen** from
Safari pointed at that GitHub Pages URL — it runs as a standalone app,
not inside a Safari tab (this matters: standalone launches and a plain
Safari tab of the same URL use *separate* localStorage buckets).

## Design system — extend this, don't reinvent it

- **Palette**: native-Apple system tokens (grouped-background grays,
  hairline borders, no drop shadows in dark mode). Default accent is
  system blue (`#007aff` / `#0a84ff` dark), but it's **user-selectable**
  via a swatch picker (Blue/Purple/Pink/Red/Orange/Yellow/Green/
  Graphite) — it drives tags, links, buttons, the pinned-note tint.
  Stored in `localStorage['spark.accent']`.
- **The gold sparkle (`#f5a623` / `#ffcf3f`) is the fixed brand mark** —
  logo, favicon, apple-touch-icon, the corner-burst flourish, the
  empty-state icon. It is deliberately independent of the user's accent
  choice and should never be recolored to match it.
- **Typography**: system font stack only (`-apple-system, ...`), no
  custom/Google font. Deliberate choice for a native feel, not a
  placeholder waiting to be replaced.
- **Icons**: hand-drawn inline SVGs in an SF-Symbols-like style
  (magnifying glass, share tray, appearance half-circle, pin), not an
  icon font or library.
- **Motion**: a shared `--ease` (smooth) and `--spring` (overshoot)
  cubic-bezier token pair. Every animation respects
  `prefers-reduced-motion` — keep doing that on anything new.

## What's already built — don't re-suggest these as new features

Quick-add capture, `#tags` with autocomplete (Tab/arrows/click to
complete; a plain Enter still saves the literal text if you haven't
navigated the suggestions), `**bold**`, `` `code` ``, auto-linked URLs,
threaded replies, pin, edit, copy, undo-toast delete (no blocking
`confirm()` dialogs anywhere in the app), search, tag-filter chips, date
separators in the timeline (Pinned / Today / Yesterday / weekday /
date), JSON export/import, a backup-nudge banner (shows after 5+ notes
and 14+ days since the last export, snoozes a week when dismissed),
light/dark theme plus the accent picker, an auto-growing composer
(caps around 400px/50vh, then scrolls internally), a touch-vs-desktop
aware Enter key (mobile: Enter makes a new line, the Capture button is
the only "send"; desktop: Enter saves, Shift+Enter for a new line), a
corner sparkle-burst flourish on every capture, a custom favicon +
apple-touch-icon (white background, gold sparkle — replacing a stale
black-gradient icon left over from the very first version of the file),
and an offline-capable service worker (stale-while-revalidate; it's a
no-op on a local `file://` copy and only takes effect once actually
hosted over https).

**Most recent structural change**: the composer is no longer a box
sitting at the top of the page. It's a floating frosted-glass pill
anchored near the bottom (`#floatBar` / `#pillFace`) — tapping it (or
the `n` shortcut, or Reply/Edit on a note) morphs it into the full
compose sheet with a dimming scrim behind it (`#scrim`); Escape or
tapping the scrim collapses it back to the pill.

## Explicitly considered and declined — don't re-propose without new info

- **Photos/drawings as attachments.** Technically possible, but risky:
  `localStorage` caps around ~5MB total, and a couple of uncompressed
  photos could blow that and put text notes at risk too. Not built. If
  ever revisited, the real fix is migrating storage to IndexedDB (or
  routing just large attachments through the Cache API the service
  worker already uses) — not a quick patch, a real piece of work.
- **A native/App Store app.** The user doesn't want to pay Apple's
  $99/year Developer Program for this. Home Screen web app + service
  worker is the agreed ceiling.
- **Cloud sync / a backend.** Explicitly against the "runs only on my
  device" premise this whole project is built on. Don't propose it.

## Known quirks on the original Mac — re-verify on a different machine

- No `gh` CLI, no Homebrew installed — GitHub interactions there are
  manual web-UI only, not `git push`.
- iOS Simulator needs `sudo xcode-select -s
  /Applications/Xcode.app/Contents/Developer` once before it's usable —
  needs the user's password, was never run.
- The Claude in Chrome extension was never connected in this project's
  sessions — browser testing used the sandboxed Browser-pane preview
  instead, which itself blocks Service Worker registration (a false
  negative, already diagnosed as an environment limit, not a code bug).
  Verify real offline behavior on an actual device or a properly
  connected browser, not that pane.

## Working style this user has responded well to

- Keep suggestions to 3-4 max, prioritized, with one clear top pick —
  not an exhaustive feature dump. Simplicity wins over scope creep
  unless they explicitly ask for more.
- Actually test changes in a live browser before calling them done —
  this user has caught real bugs that way (an invalid CSS `font`
  shorthand that silently broke the composer's typeface; an Escape-key
  path that only worked in reply/edit mode, not for a plain new
  capture) that a code read-through alone missed.
- Be upfront *before* anything becomes reachable outside the user's own
  device (hosting, a public repo). They care specifically about this
  staying private/local in spirit even though it's technically hosted
  on GitHub Pages now — flag that trade-off explicitly rather than
  assuming it's fine.
