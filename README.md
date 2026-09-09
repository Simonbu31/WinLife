# WinLife

Turn your life into a video game. WinLife is a personal habit tracker styled as an RPG character sheet — complete daily habits and long-term goals to earn XP, build streaks, and level up.

**Web app:** https://simonbu31.github.io/WinLife/
**App Store:** [WinLife on the App Store](https://apps.apple.com/app/winlife/id6797948771)

---

## What it does

- **Daily Routines & Content Output** — define your own daily habits, split into two sections (e.g. personal routines vs. things you ship/create). Tap to complete, earn XP, build a streak.
- **Streaks** — every habit tracks a running streak. Miss a day and it resets; a "yesterday" toggle lets you backfill a day you forgot to log without losing progress.
- **Missions** — longer-term goals grouped into custom columns (e.g. Mind / Body / Business). Some are simple one-shot completions, others have sub-goals or a progress bar.
- **XP & Leveling** — every action earns XP on an exponential leveling curve, with a level-up animation when you cross a threshold.
- **Onboarding** — new users are walked through a short 5-step flow: name, domains of mastery, goals per domain, content/output habits, then daily routines. Skip any step (or all of it) and go straight to the defaults if you'd rather. A one-time coach-marks walkthrough after onboarding shows the tap/swipe gestures on the real habit list.
- **Offline-first** — the app works fully offline with no account. Optionally sign in to sync progress across devices.
- **Daily reminders** *(iOS app only)* — an optional daily notification (defaults to 12:00 noon), plus a smart streak-protection alert at 18:00 if a streak longer than 3 days is about to break.

## Tech stack

- **Frontend:** Vanilla JavaScript, single HTML file (`index.html`) — no framework, no build step
- **Auth & sync:** [Supabase](https://supabase.com) (email/password auth, Postgres + Row Level Security for cloud saves) — entirely optional, the app is offline-first and fully usable without an account
- **Hosting (web):** GitHub Pages, deployed straight from `main`
- **iOS app:** [Capacitor](https://capacitorjs.com) wraps the same `index.html` into a native iOS shell via Swift Package Manager (no CocoaPods). Native notifications via `@capacitor/local-notifications`.
- **Fonts:** DM Serif Display, DM Sans, DM Mono (Google Fonts)

## Project structure

```
index.html          # the entire web app — HTML, CSS, and JS in one file
privacy.html         # privacy policy (linked from the App Store listing)
mobile/               # Capacitor wrapper for the iOS app
  capacitor.config.json
  ios/App/            # Xcode project
  www/                 # Capacitor's web asset source (synced from index.html — not the source of truth)
```

`index.html` at the repo root is the single source of truth for all app logic. The copies inside `mobile/www/` and `mobile/ios/App/App/public/` are build artifacts, resynced from the root file before every iOS release — they are not edited directly.

## Running it locally

No build step, no dependencies. Just open the file:

```bash
git clone git@github.com:Simonbu31/WinLife.git
cd WinLife
open index.html          # macOS
# or serve it, e.g.: python3 -m http.server
```

To develop the iOS app:

```bash
cd mobile
npm install
npx cap sync ios
open ios/App/App.xcodeproj
```

## Deployment

**Web (GitHub Pages):** push to `main` — GitHub Pages redeploys automatically. No build step.

**iOS (App Store):**
1. Edit `index.html` at the repo root (the source of truth)
2. Copy it into `mobile/www/index.html`
3. `cd mobile && npx cap sync ios` to propagate the change into the Xcode project
4. Archive, validate, and distribute via Xcode Organizer (or `xcodebuild archive` from the CLI)
5. Submit the build for review in App Store Connect

## License

No license file yet — all rights reserved by default. Reach out to the author before reusing any part of this project.
