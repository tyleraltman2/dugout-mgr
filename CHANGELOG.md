# Salt DugoutMgr — Changelog

A record of product iterations built through active use and real-game testing with a youth travel baseball coaching staff.

---

## v1.0 — Foundation
*Initial build*

- Roster management: add/remove players with name and jersey number
- Basic lineup tab with per-inning position assignment (P, C, 1B, 2B, 3B, SS, LF, CF, RF, SIT)
- Batting order with drag-to-reorder (desktop drag and touch drag support)
- SAT and IP columns tracking innings sat and innings played up to current inning
- AB tracking per player with +/− controls
- 6-inning navigation with prev/next arrows
- Inning completion dots (gold = current, green = complete, amber = partial, gray = empty)
- Copy Previous Inning button for quick assignment reuse
- Game setup: Regular and Cup game types
- Cup game sitting tracker panel enforcing "everyone sits once before anyone sits twice"
- Basic pitch count tracking with +/− controls and tap-to-enter modal
- Pitcher/catcher conflict detection (41+ pitches can't catch; 4 catcher innings can't pitch)
- Rest day calculation: 1–4 days based on pitch count thresholds
- Remove-from-mound enforcement (no re-entry rule)
- Game history tab with per-player defensive innings, AB, and sitting totals
- Firebase Firestore real-time sync across multiple coaching devices
- localStorage persistence as offline fallback

---

## v1.1 — Sync Stability
*Addressing multi-device sync reliability*

- Switched from ES module Firebase SDK to compat SDK — resolved silent failures on iOS Safari and Chrome mobile
- Added 4-second polling fallback alongside real-time Firestore listener
- Implemented `_updatedAt` timestamp-based conflict resolution in `mergeRemoteState`
- Removed `suppressRemoteUpdate` and `isSyncing` flags that were blocking legitimate syncs
- Changed localStorage key to `dugout_v3` for new schema version
- Fixed sync dot indicator (green = live, amber = local only)

---

## v1.2 — Tournament Game Types
*Major feature expansion for travel tournament compliance*

**Perfect Game (10u) mode:**
- Daily pitch limit: 75 pitches max
- Daily out limit: 18 outs max
- 10+ outs in a day triggers 2-day rest requirement
- No 3 consecutive days pitching rule
- No 3 games in same day rule
- Event pitch max: 100p (2–4 day events) or 140p (5+ day events), selectable at game start
- Violations shown as warning chips (not hard blocks) with confirm-to-proceed

**Training Legends (10u) mode:**
- Innings-based limits only (1 pitch = 1 inning pitched)
- Dynamic limit by game number: 6 innings through game 4, +1 per game after
  - Game 5 = 7 inn, Game 6 = 8 inn, etc.

**Tournament architecture:**
- `activeTournament` object tracks cumulative pitcher totals across games within a tournament
- Cross-game pitch/out/inning totals per pitcher
- Day tracking for consecutive-day and same-day rest rule enforcement
- Tournament saved to `state.tournaments[]` on completion

**New TOURN tab:**
- Active tournament dashboard with per-pitcher eligibility badges
- Dual progress bars (daily pitches + event pitches) for PG
- Inning progress bar for TL
- Game log showing per-game pitching stats
- End Tournament button saves to completed tournaments list
- Past tournaments section with cumulative totals, deletable

**Game setup additions:**
- PG options: event length selector, game #, day #, tournament name
- TL options: game #, tournament name
- Season toggle hidden for PG and TL games

---

## v1.3 — UI Polish & Bug Fixes
*Based on first round of real-game testing*

**Bug fixes:**
- Fixed soccer ball emoji (⚽) displaying instead of baseball (⚾) in header and game type options
- Fixed "Early Season · Max 65p" pitch limit label incorrectly displaying during Perfect Game games
  - Root cause: `renderAll()` was calling `selectGameType()` on every Firebase sync, overwriting user's game type selection
  - Fix: `selectGameType()` no longer writes to `state` — DOM only; `state.gameType` set from DOM at `startGame()` time
- Fixed Perfect Game game incorrectly starting as Regular game
  - Root cause: Firebase syncing back stale `gameType: 'regular'` state after user selected PG but before tapping Start
  - Fix: `startGame()` reads game type from DOM selection, not from `state.gameType`
- Fixed TOURN tab showing "No active tournament" during active PG/TL game
  - Root cause: Firebase overwriting `activeTournament` after `startGame()` saved it
  - Fix: `renderTournament()` reconstructs tournament object on-the-fly from `currentTournMeta` if `activeTournament` is null during active game
- Removed dangerous delayed `save()` in `startGame()` that was overwriting Firebase with stale game state

**Improvements:**
- Season section (Early/Late toggle) now hidden by default, only visible for Regular and Cup game types
- `renderAll()` calls `selectGameType()` on load to sync season section visibility with saved state
- Page always initializes with Regular game type selected regardless of previous session state

---

## v1.4 — History Tab Redesign
*Driven by specific coach use case: tracking position reps across games*

**Problem identified:** Coaches needed to track cumulative innings per player per position — specifically to ensure multiple players all got reps at contested positions like shortstop across a season.

**Solution: expandable game cards with position breakdown**

- Tap any game card in History to expand inline
- "Innings by Position" section shows one row per position used in that game
- Player chips with inning counts, color coded: green = heavy reps, gold = some reps
- Zero-inning players excluded from position chips (only show players who actually played)
- SIT row uses "Option 4" logic: shows summary line ("9 players sat 1 inn") instead of individual chips; only players who sat 2+ innings get a red chip; flags players who sat 0 innings in amber
- Tap again to collapse
- Pitch summary in card header shows only players who actually pitched (zero-pitch players excluded)
- History records now include batting order and tournament metadata

---

## v1.5 — Roster Drag & Drop
*Quality of life improvement for batting order management*

- Added drag-to-reorder functionality to the Roster tab (mirrors Lineup tab behavior)
- Touch drag support for mobile
- Reordering the roster automatically updates `battingOrder` — order set in Roster becomes default batting order for all new games
- Reordering during an active game does not affect current game's batting order

---

## v1.6 — Tournament Continuation Flow
*Reducing friction for multi-game tournament management*

**Problem identified:** When starting a new game within an existing tournament, coaches had to manually re-type the tournament name and remember the correct game number — creating risk of data entry errors affecting pitcher eligibility calculations.

**Solution: smart tournament continuation picker**

- When PG or TL is selected, app auto-detects any in-progress tournament of that type
- Shows "In-Progress Tournament" banner with tournament name, games played, and event details
- Two options: "✓ Continue This" (pre-fills fields) or "+ New Tournament" (blank form)
- **Continue mode:** game # auto-increments to next unplayed game; day # pre-filled from last game with amber warning note
- **Day # confirmation:** warning displays affected pitchers with rest requirements on selected day — coach must consciously verify before starting
- Game # dropdown only shows unplayed games when continuing (e.g. after 2 games, dropdown starts at Game 3)
- Retry timing on Firebase sync ensures in-progress tournament populates even on slow connections
- `pgUserEdited` / `tlUserEdited` flags prevent Firebase sync from overwriting manual field changes
- Modes reset cleanly when tournament is ended

---

## v1.7 — PG Rules Display
*Surfacing compliance rules at point of decision*

**Problem identified:** Coaches needed quick reference to PG pitching rules during games without leaving the app.

**Option A (static rules) + Option C (contextual per-pitcher warnings):**

- PG pitching rules block added to TOURN tab tournament card:
  - Daily limit: 75 pitches OR 18 outs
  - Rest: 10+ outs requires 2 days rest
  - Consecutive days: cannot pitch 3 days in a row
  - Same day: cannot pitch in 3 games in one day
- Same rules block added to PITCH tab when a PG game is active; hidden for Regular/Cup/TL games
- Pitcher status cards in TOURN tab now show contextual warning chips:
  - **2-DAY REST** (amber) — with last game's out count as context
  - **3RD CONSEC DAY** (red) — with specific days pitched shown
  - **MAX GAMES/DAY** (red) — with game count shown
  - **EVENT LIMIT** (red) — with pitch total shown
- Today's running totals (pitches + outs) shown on pitcher card during active game
- Eligibility badge combines all four rules into single ELIGIBLE / INELIGIBLE status

---

## v1.8 — Bug Fixes & Rule Corrections
*QA pass after first tournament weekend*

**Rule corrections:**
- Rest threshold corrected from `>= 9 outs` to `> 9 outs` (10+ outs) throughout — affected 4 locations: tournament pitcher card, pitch tab warning, game setup day warning, in-game alert
- Rest alert in `adjustOuts` changed to fire only at exact 9→10 crossing, not on every increment above 9

**Bug fixes:**
- Fixed `gamesInDay` incorrectly incrementing for non-pitchers
  - Root cause: `pd.innings` tracks defensive innings from lineup tab; was being used alongside `pd.count` and `pd.outs` to determine pitching appearances
  - Fix: only `pd.count > 0 || pd.outs > 0` counts as a pitching appearance
- Fixed `daysPitched` same root cause — non-pitchers were being marked as having pitched on a given day, causing false consecutive-day violations
- Fixed "2-day rest required" warning showing for pitchers with only 9 outs in previous game
  - Root cause: `totalOuts` (cumulative across all tournament games) was being used for rest check instead of last game's outs only
  - Fix: added `getLastGameOuts()` helper that finds pitcher's outs from most recent completed game; all rest checks now use this
- Fixed "2-day rest" warning appearing when starting a new tournament after ending a previous one
  - Root cause: `pgMode` was still set to `'continue'` from ended tournament; `showPgDayWarning` was reading stale `activeTournament` data
  - Fix: `endTournament()` now resets `pgMode`, `tlMode`, `pgUserEdited`, `tlUserEdited` to defaults; `showPgDayWarning` bails if `pgMode !== 'continue'`
- Fixed day # dropdown reverting to pre-filled value when user manually selects a different day
  - Root cause: 800ms and 2000ms retry timers firing after user changed dropdown, re-running `pgSelectContinue()` which reset the value
  - Fix: `pgUserEdited` flag set on any manual field change; all retry timers and Firebase sync refreshes check flag before re-populating

---

## v1.9 — Pitch Tab & Tournament Tab Polish
*Presentation improvements based on in-game usage*

- Pitcher status in TOURN tab now sorted by total pitches (PG) or innings (TL) descending — heaviest-used pitchers float to top
- Game log in TOURN tab now only shows players who actually pitched — zero-pitch players excluded
- Game log entries sorted by pitch count descending within each game
- "Total outs" context on pitcher cards now correctly shows last game's outs, not cumulative tournament total

---

## v1.10 — Multi-Coach Sync Fix
*Critical fix for secondary coach devices*

**Problem identified:** Second coach opening the app on his device saw "NO GAME" even though game was active on head coach's device. Sync dot showed green/LIVE, meaning Firebase was connected but state wasn't being applied.

**Root cause:** `mergeRemoteState` had guards designed to protect the active coach's device from stale Firebase bouncebacks. These same guards were blocking secondary coaches from ever receiving state because:
1. Their localStorage from a prior session had a newer `_updatedAt` than the current Firebase state
2. `!state.gameActive && remote.gameActive` guard fired — their local state said "no game," Firebase said "game active" — blocked

**Fix:** All `mergeRemoteState` guards now only activate if `lastSaveTime > 0` (meaning this device has called `save()` during the current session). Fresh loads (secondary coaches, new devices, page refreshes) always accept whatever Firebase delivers with no guards applied.

**Additional improvements:**
- Sync dot is now a tappable button for manual sync
- Sync status labels added: SYNCING... / LIVE / SYNCED / LOCAL
- Firebase fetch and listen callbacks now update sync label on success
- `manualSync()` function forces fresh Firebase pull with user feedback

---

## v1.11 — History Bug Fix
*Critical fix: saved games not appearing in History tab*

**Problem:** Tapping "End Game & Save" confirmed the save dialog but games were not appearing in the History tab.

**Root cause:** `historyExpanded` variable (used by the expandable history cards) was accidentally dropped during a prior rebuild. `renderHistory()` threw an uncaught `ReferenceError: historyExpanded is not defined` on every call, crashing silently and preventing the tab from rendering. History data was saving correctly to localStorage and Firebase — the display layer was broken.

**Confirmed via console logging:** localStorage history length incremented correctly on save; crash occurred at `renderHistory()` → `renderAll()` → `endGame()`.

**Fix:** Restored `let historyExpanded = {}` declaration.

---

## Development Notes

This app was built entirely through iterative conversation with AI (Claude), with the product direction, requirements, QA, and user feedback provided by the coaching staff using it in real games. The development process itself mirrors modern product management: ship fast, get real feedback, fix what breaks, improve what's good enough.

Total development time: ~6 weeks of active iteration alongside a live tournament season.
