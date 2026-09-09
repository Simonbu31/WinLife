# Life RPG — Project Briefing

## What this is
A personal habit tracker styled as a video game character sheet. Single HTML file, vanilla JS, no frameworks. Hosted on GitHub Pages.

**Live URL:** https://simonbu31.github.io/WinLife/
**Repo:** https://github.com/Simonbu31/WinLife
**File:** `index.html` (everything is in this one file)

---

## Tech stack
- **Frontend:** Vanilla JS, single HTML file — no build step, no frameworks
- **Auth + Sync:** Supabase (email/password auth, cloud save)
- **Hosting:** GitHub Pages
- **Fonts:** DM Serif Display, DM Sans, DM Mono (Google Fonts)

### Supabase config
- **Project URL:** `https://lanfehcvxcfejtpyhtpg.supabase.co`
- **Anon key:** in the HTML file as `SUPABASE_KEY`
- **Tables:**
  - `saves` — one row per user, stores player/habits/log/missions/setup as jsonb
  - `feedback` — bug reports from users (message, user_id, created_at)

---

## Architecture

### Single file structure (top to bottom)
1. `<head>` — Supabase CDN, Google Fonts, `<style>` block
2. `<body>` — all HTML (auth overlay, onboarding overlay, game UI, modals)
3. `<script>` — all JS

### JS module layout (inside the script tag)
```
SUPABASE CONFIG + SYNC (sync.push, sync.pull)
DEFINITIONS LOADER (getDefinitions, getHabitDefs)
KEYS (localStorage key constants)
DEFAULT_COLUMNS / DEFAULT_SECTION_LABELS (fallback labels)
SHARED HELPERS (slugify, renderList - used by onboarding and the editors)
EDIT SETUP (getEditableSetup - staged copy of habits/missions/columns/sectionLabels)
XP LOGIC (exponential scaling: floor(100 * level^1.5))
HABIT LOGIC (complete, skip, unskip, streaks, yesterday backfill)
MISSION DEFINITIONS (hardcoded year goals)
HABIT DEFINITIONS (hardcoded daily habits)
STORAGE (loadState, saveState, loadMissions, saveMissions)
RENDERING (renderXP, renderHabits, renderHabitCard, renderMissions, renderStats)
ANIMATIONS (showXPPopup, showSkipToast, showBonusToast, showLevelUp)
SWIPE LISTENERS (left = undo, right = skip)
HANDLE CLICK (complete habit on tap)
YESTERDAY TOGGLE (backfill yesterday's habits)
MISSIONS RENDERING + INTERACTION
PAGE NAVIGATION (tab bar: Habits / Missions)
AUTH (runAuth — email/password, sign up/in, offline mode)
ONBOARDING (runOnboarding — 5 screens: name, domains, goals, output, routines)
COACH MARKS (maybeShowCoachMarks — first-use gesture walkthrough, once)
EDIT HABITS / EDIT GOALS (openHabitsEditor, openGoalsEditor - post-onboarding editors)
MIDNIGHT RESET
INIT (wires everything together)
ENTRY POINT (async IIFE — checks Supabase session, routes to auth or game)
```

---

## Key features

### Habits page
- **Daily Routines** (10 XP each) and **Content Output** (15 XP each)
- Tap to complete → XP popup, card slides to bottom
- Swipe left on completed card → undo button
- Swipe right on incomplete card → skip for today (breaks streak)
- Swipe left on skipped card → unskip (restores streak)
- **Yesterday toggle** — small pill button to backfill yesterday's habits
- **All-habits bonus** — +20 XP when every habit across both sections is done

### XP + Leveling
- Exponential scaling: `xpForLevel(n) = floor(100 * n^1.5)`
- Level 1→2: 100 XP, Level 2→3: 283 XP, Level 3→4: 520 XP, etc.
- XP bar shows progress within current level

### Missions page
- 3 columns: Life Goals / Fitness Goals / Music Goals (or user-defined domains)
- Tap to complete → 500 XP, card slides to bottom, permanently done
- "Finish Psychology Bachelor" has sub-goals (50 XP each)
- "Release 20 songs" has a progress bar (tap = +1, auto-completes at 20/20)

### Auth + Sync
- Supabase email/password auth shown on first load
- "Play offline" skips auth
- On sign in: pulls cloud save → merges to localStorage
- On every save: writes to localStorage instantly + pushes to Supabase in background
- Offline-first: localStorage is always source of truth
- Sign out in ⚙ settings

### Onboarding (for new users / friends)
- 5 screens, reordered in v1.2 to follow a Future-Authoring-style flow (domains first, then goals, then habits): name → domains of mastery → goals per domain → content output (yes/no gate) → daily routines
- Domains: Mind / Body / Business / Relationships / custom (max 4)
- Goals-per-domain screen reuses the same add-goal UI/shape as Settings → Edit Goals (`{id, label, column, xp:500}`); forward-only flow, no back button, every screen has Skip
- Content Output screen is an in-place yes/no gate: "No" skips straight to Daily Routines, "Yes" reveals the existing add-habit form on the same screen
- Saved to `lifeRPG_setup` in localStorage (shape unchanged by the v1.2 reorder: `{name, habits, missions, columns}`)
- If `lifeRPG_setup` exists → skip onboarding, go straight to game
- `getDefinitions()` reads from setup if present, falls back to hardcoded defs
- Auth screen ("Welcome back." vs "Welcome.") is conditioned on local `lifeRPG_setup` presence (`KEYS.coachmarks` unrelated - see below) so first-time visitors aren't told "back"
- First-use coach marks: a 4-step Next-button-driven walkthrough (tap/swipe-left/swipe-right/+20 bonus) with a spotlight highlighting the real habit row, shown once immediately after onboarding completes, gated by `lifeRPG_coachmarks_seen` in localStorage (same once-only pattern as `REMINDER_KEY`)

### Settings (⚙ icon top right)
- Edit Habits → opens editor to add/delete habits and rename the two section headings
- Edit Goals → opens editor to add/delete goals and rename the goal columns
- Export progress → downloads JSON backup
- Import progress → loads JSON backup, reloads page
- Sign out
- Report a bug → submits to Supabase `feedback` table

### Edit Habits / Edit Goals (post-onboarding editing)
- Two modals, opened from Settings, independent of the onboarding wizard (never touches `lifeRPG_player`, so editing never resets XP/level)
- Both are add/delete only for now (no renaming or XP-editing of individual habits/goals) with an explicit Save/Cancel — nothing writes to storage until Save is tapped
- Edit Habits also lets you rename the "Daily Routines" / "Content Output" section headings (stored as `lifeRPG_setup.sectionLabels`, falls back to `DEFAULT_SECTION_LABELS` if unset)
- Edit Goals also lets you rename the goal columns in place (mutates `col.label` directly in the staged column list)
- `getEditableSetup()` is the shared entry point both editors use to get a full, deep-cloned, always-populated staging copy of `{name, habits, missions, columns, sectionLabels}` to mutate before Save
- The whole modal box scrolls as one unit (title + list + Save/Cancel together) rather than trying to pin a header/footer around an inner scroll region — that inner-scroll approach was tried first and was unreliable across browsers, so don't reintroduce it

---

## Architecture rules (never break these)
- **Never patch a bug by removing a feature**
- **Habit definitions live in code** (not localStorage) as the hardcoded fallback — editing habits never corrupts saved data
- **Offline-first** — localStorage is always written first, Supabase is background sync
- **No non-ASCII characters in JS comments** — causes SyntaxError in some browsers (use ASCII dashes, not em dashes; no backticks in comments)
- **No duplicate HTML element IDs** — causes null reference errors
- **Single file** — keep everything in index.html, no external JS files

---

## localStorage keys
```
lifeRPG_player   — { level, totalXP, currentXP, name }
lifeRPG_habits   — { habitId: { streak, lastCompleted } }
lifeRPG_log      — { "YYYY-MM-DD": { habitId: true | "skipped" | { skipped: true, prevStreak: N } } }
lifeRPG_missions — { missionId: { done, progress } }
lifeRPG_setup    — { name, habits, missions, columns, sectionLabels } (null habits/missions = use hardcoded; sectionLabels = { routines, content })
```

---

## Simon's current data
Simon's hardcoded habit/mission definitions are in the file. His backup can be imported via ⚙ → Import. His Supabase account is `burkhardt.simon@gmx.de`.

---

## Deployment
```bash
# Every update:
cp ~/Downloads/life-rpg-5.html ~/Downloads/Life_RPG/index.html
cd ~/Downloads/Life_RPG
git add . && git commit -m "update" && git push
```
SSH is set up, no password needed.

---

## Roadmap (not built yet)
- Bug report agent — reads feedback table, proposes fixes via Claude API, Simon reviews
- Character attributes (Mind / Body / Business / Connection) per habit
- Quest board / weekly goals
- Skill unlocks after X-day streaks
- PWA wrapper (installable, offline)
- App Store consideration (would need Capacitor or Swift rewrite)
- Rename/XP-edit of individual existing habits and goals (Edit Habits/Edit Goals currently only support add + delete)

---

## Common gotchas
- **`lifeRPG_setup` must exist** for onboarding to be skipped — new browsers need a backup import or fresh onboarding
- **Supabase 406 error on first login** is normal — means the saves table has no row yet for this user; it creates on first habit completion
- **Browser cache** — after pushing to GitHub, do Cmd+Shift+R to see the update
- **Non-ASCII in JS comments** has caused repeated SyntaxErrors — always use plain ASCII in comments
- **Duplicate HTML IDs** have caused repeated null reference errors — always check with grep before shipping
