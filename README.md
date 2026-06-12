# Salt DugoutMgr

A mobile-first web app for youth baseball coaches to manage lineups, track pitch counts, and enforce tournament pitching compliance rules in real time.

Built to solve a real problem: during travel baseball tournaments, coaches juggle complex pitching rules across multiple games and days, while also tracking playing time and batting order for 11+ players. Spreadsheets and paper don't cut it on a dugout bench.

**Live app:** [dugout-mgr](https://tyleraltman2.github.io/dugout-mgr)

---

## Features

### Lineup Management
- Per-inning position assignment for up to 11+ players
- Drag-to-reorder batting order (touch-friendly on mobile)
- Sitting tracker with Cup game compliance enforcement
- Inning completion dots showing assignment progress
- Copy previous inning assignments with one tap

### Pitch Tracking
- Real-time pitch count with +/− controls and tap-to-enter modal
- Rest day calculations for regular and cup games
- Pitcher/catcher conflict detection (pitch count vs. catcher innings)
- Remove-from-mound enforcement (no re-entry rule)

### Tournament Mode — Perfect Game (10u)
- Daily pitch limit: 75 pitches or 18 outs (whichever comes first)
- 10+ outs in a day triggers 2-day rest requirement
- No 3 consecutive days pitching enforcement
- No 3 games in same day enforcement
- Event pitch max: 100p (2–4 day events) or 140p (5+ day events)
- Cross-game cumulative tracking within active tournament

### Tournament Mode — Training Legends (10u)
- Innings-based pitching limits
- Dynamic limit by game number: 6 innings through game 4, +1 per game after
- Cross-game inning totals tracked across tournament

### Tournament Dashboard
- Per-pitcher eligibility status with contextual warning chips
- Visual progress bars for daily and event pitch totals
- Game log showing pitching stats for each completed game
- In-progress tournament continuation flow with smart game/day pre-fill

### History Tab
- Expandable game cards with innings-by-position breakdown
- Position totals per player (e.g. who played SS and for how many innings)
- Bench tracking with outlier flagging (players who sat 2+ innings)
- Pitch summary per game

### Multi-Coach Sync
- Real-time sync across up to 5 devices via Firebase Firestore
- Live/offline sync status indicator with manual sync button
- Conflict resolution via timestamp-based merge

---

## Tech Stack

- **Frontend:** Vanilla HTML, CSS, JavaScript — single file, no build step
- **Database:** Firebase Firestore (real-time sync)
- **Hosting:** GitHub Pages
- **Design:** Mobile-first, portrait orientation, dark theme

---

## Background

This app was built iteratively over several weeks based on direct feedback from coaches using it during real games and tournaments. The feature set grew from basic lineup tracking to full tournament compliance tooling through continuous user testing in the dugout.

Key product decisions along the way:
- Chose a single HTML file over a framework to eliminate build complexity and enable easy deployment via drag-and-drop
- Firebase compat SDK (not ES modules) chosen specifically for mobile browser compatibility
- Tournament rules encoded precisely per Perfect Game and Training Legends official rulebooks
- History tab redesigned from a simple list to an expandable position-breakdown view based on a specific coach use case: tracking which players got reps at shortstop across games

---

## Status

Actively used by coaching staff. Ongoing development based on in-season feedback.
