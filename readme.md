# RedWaist — Telegram Mini App

A static, JSON-driven Telegram Mini App: watch tasks unlock videos. No backend, no database.

## Files

```
/
├── index.html       # the app users open in Telegram
├── admin.html        # admin panel for editing videos/tasks/settings
└── data/
    └── data.json      # all content (videos, tasks, settings)
```

## 1. Deploy (GitHub Pages)

1. Push this folder to a GitHub repo (public or private, either works for Pages).
2. Repo → **Settings → Pages** → Source: `main` branch, root folder.
3. Your app will be live at `https://USERNAME.github.io/REPO/index.html`.
4. Put your `.mp4` files anywhere reachable by URL — easiest is the same repo under `videos/`, referenced via
   `https://raw.githubusercontent.com/USERNAME/REPO/main/videos/xyz.mp4`. Large files or heavy traffic should
   use real video hosting instead of raw GitHub URLs (GitHub isn't meant for that).

## 1b. Deploy (Vercel)

1. `vercel` CLI or the Vercel dashboard → import the repo/folder as a static project (no build step needed).
2. Vercel serves `index.html` and `admin.html` at the project's root URL automatically.

## 2. Telegram Mini App setup

1. Talk to **@BotFather** → `/newapp` (or `/newbot` first, then attach a Mini App).
2. Set the Web App URL to your deployed `index.html` URL (e.g. `https://USERNAME.github.io/REPO/index.html`).
3. Add a menu button or inline "Open App" button pointing at that URL — Telegram loads it inside its own WebView
   and injects `window.Telegram.WebApp`, which `index.html` already reads (first name, user id, etc.).
4. `admin.html` is **not** meant to be linked from the bot — open it directly in a normal browser when you need
   to edit content. Don't expose it as a Mini App button.

## 3. Editing content (JSON workflow)

**Local / Download mode (recommended):**
1. Open `admin.html` in a browser (locally or via a private URL).
2. Click **Load JSON** to import your current `data/data.json` (or start from the bundled sample).
3. Edit videos, tasks, and settings in the form, or edit raw JSON directly under the "Raw JSON" tab.
4. Click **Download JSON**, replace `data/data.json` in your repo with the downloaded file, commit, and push
   (or drag-and-drop into GitHub's web UI).

**GitHub mode (optional, use with care):**
- Lets `admin.html` read/write `data/data.json` directly via the GitHub Contents API, using a
  Personal Access Token you paste into the page.
- **This token lives only in the browser tab's memory** — it is not saved anywhere by the app. But because
  `admin.html` is a public static file, anyone who can load that page could, in principle, view its source or
  intercept what you type. Only use this mode on a private/trusted device, ideally with a fine-grained token
  scoped to just this one repo, and revoke it when you're done. Do not link this page publicly.

## 4. Video unlock flow

1. User opens a video card — sees required task count (from `requiredTasks` in the video's JSON).
2. User taps each task; the app opens `targetUrl` and marks the task complete in the browser's `localStorage`.
3. Progress bar/ring updates: `1/3`, `2/3`, `3/3`.
4. Once `requiredTasks` is met, the video unlocks: the built-in player and **Download** button appear.

## 5. Important limitations of a static, JSON-only architecture

- **No real accounts or server-side verification.** `localStorage` tracks progress per browser/device only.
  Clearing browser data, using a different device, or editing localStorage via devtools resets or fakes progress.
  There is no way to prevent this without a backend.
- **Task completion is not verified.** The app trusts that opening a task's link means the task was done — it
  can't confirm someone actually watched an ad or joined a channel. This is inherent to a backend-less design.
- **Editing content requires a manual publish step.** Because browser JS can't write files back to a static
  host, every content change goes through: edit in `admin.html` → download/commit `data.json` → redeploy (GitHub
  Pages/Vercel redeploy automatically on push, typically within a minute).
- **GitHub-token mode is inherently less safe than a real backend.** Treat it as a convenience for trusted,
  single-admin use — not a substitute for authenticated server-side admin access.
- **Video hosting via raw GitHub URLs is a stopgap.** Fine for small test files; for real traffic/size, move
  `videoUrl`/`downloadUrl` to dedicated video hosting or object storage (still just a URL change in `data.json`,
  no code changes needed).

## 6. Customizing appearance

- `settings.accentColor` in `data.json` recolors the app's chili-red accent live (index.html reads it into a CSS
  variable — no rebuild needed).
- `settings.appName`, `settings.logo`, and `settings.featuredVideo` are also live-editable through the same
  file.
