# Tour de Taco Bell

A mobile web app for tracking a one-day bike race across every Taco Bell in the Omaha/Council Bluffs metro: 30 stops, 107 menu items, 5 riders. The goal is to visit every store and collectively order and finish the entire menu along the way. Race day: 2026-08-30.

## The app

Everything lives in `index.html`: a single self-contained page (no build step, no framework) designed to run on phones. Tabs:

- **Race**: live progress, per-rider standings, pace vs. the 11 AM breakfast cutoff, completion % (weighted 70% items / 30% stops).
- **Menu**: all 107 items grouped by category with calories/macros, searchable, tap to claim/log an item for a rider.
- **Route**: the 30 stops in order with done/skip status and Google Maps directions links.
- **Map**: Leaflet map of all stops with live status markers.
- **Plan**: the pre-computed order sheet, per stop and per rider.

### The plan

Item-to-stop and item-to-rider assignments are precomputed in the `PLAN` and `PLAN_RIDER` constants:

- Breakfast items and coffee/OJ at stops 1-4 (breakfast ends ~11 AM).
- All caffeinated drinks placed by stop 18, Rockstar energy drinks early (stops 5-6).
- Stops calorie-balanced to roughly 1,080-1,280 cal each, max 2 drinks per stop.
- Riders balanced to ~7,450 cal and 21-22 items each.

### The route

Stop order minimizes total drive time: OpenRouteService driving matrix over all 30 stores, 2-opt/or-opt optimization, free start, finish fixed at 168th & Maple. Result: ~3h 43m / ~104 mi (the old Council Bluffs-first order was 4h 11m / ~119 mi). Starts in Bellevue, sweeps west through Papillion/Millard/Gretna/Elkhorn, back east through midtown to Council Bluffs, then the northern arc to the finish. Plan constraints (breakfast at stops 1-4, caffeine by stop 18) are positional, so they hold under the new order.

### Sync

State syncs across all riders' phones through Firebase Realtime Database over plain REST + SSE (no Firebase SDK). Offline writes queue in `localStorage` and flush when the connection returns. Setting `DB_URL` to an empty string in the config block at the top of `index.html` switches to solo mode (localStorage only, single device).

## Hosting

Static hosting; the repo is served via GitHub Pages. Deploy = push to `master`. To point at a different race, change `DB_URL` / `DB_PATH` in the config block.

## Other files

- `items.json`: the menu item list as structured data (id, category, name, variant).
- `taco-bell-unique-items.xlsx`: source spreadsheet the item list was built from.
- `pyproject.toml`: placeholder Python project config; there is no Python code, the app is pure HTML/JS.

## Local development

Open `index.html` in a browser. That's it. It hits the live Firebase database by default, so blank out `DB_URL` first if you don't want test taps polluting race state.
