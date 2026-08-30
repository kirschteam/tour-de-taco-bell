# Tour de Taco Bell

A mobile web app for tracking a one-day driving tour (one car, Tesla Model Y) of every Taco Bell in the Omaha/Council Bluffs metro: 30 stops plus 2 Supercharger stops, 66 menu items, 5 riders. The goal is to visit every store and collectively order and finish the whole menu along the way — one classic representative of each item family (variant duplicates like the chicken/steak/cheese quesadillas collapse to the classic, here the Chicken Quesadilla; shell/seasoning twists like the Doritos Cheesy Gordita Crunch and the Tajín items count as their own items), with every multi-size item ordered at its smallest size (16 oz fountain drinks, regular freezes/fries, 2 pk Cinnabon Delights). Menu refreshed 2026-08-29 against the tacobell.com menu API for an Omaha store: the discontinued Crispy Chicken Avocado Ranch line, Pepper Jack Steak Nacho Fries, Jalapeno Citrus packet, Enchirito, and Rockstar Refresca are out; the Tajín line (Taco, Cheesy Gordita Crunch), the Decades menu (Naked Chicken Chalupa, Meximelt, Beefy Crunch Burrito, Caramel Apple Empanada), and Pineapple Citrus Energy are in — still 66 items, now with that store's prices (~$219 + tax for the full board). The original 107-item pull lives in `items.json`. Race day: 2026-08-30.

**Stretch goal — the full menu.** 44 optional ⭐ stretch items: the 38 variants the dedupe cut (bacon/steak breakfast versions, supreme tacos, zero-sugar and dirty sodas, the black-bean line, etc.) plus the 6 items the store launched after the original pull (Fresca and Melty Steak Burritos, Chicken Enchilada Nacho Fries, Strawberry Tropics Energy, Pineapple Citrus Energy Freeze, Tajín Pineapple Strawberry Freeze). They don't count toward the core goal, the completion %, or the plan, but they log like any other item and feed a separate stretch tracker on the Race tab (all 110 items eaten, +13,265 cal, +~$162 if the group clears the whole board). Verified against the live store menu 2026-08-30 — only still-sold items made the cut (the 10 discontinued ones, e.g. Dragonfruit Berry Refresca and the Crispy Chicken line, stay out).

## The app

Everything lives in `index.html`: a single self-contained page (no build step, no framework) designed to run on phones. Tabs:

- **Race**: live progress, per-rider standings, pace vs. the 11 AM breakfast cutoff, completion % (weighted 70% items / 30% stops).
- **Menu**: all 110 items (66 core + 44 ⭐ stretch, stretch rows dashed) grouped by category with calories/macros, searchable with a ⭐ Stretch filter, tap to claim/log an item for a rider.
- **Route**: the 32 stages in order (30 Taco Bells + 2 ⚡ charge stops) with done/skip status and Google Maps directions links.
- **Map**: Leaflet map of all stops with live status markers.
- **Plan**: the pre-computed order sheet, per stop and per rider.

### The plan

Item-to-stop and item-to-rider assignments are precomputed in the `PLAN` and `PLAN_RIDER` constants:

- Breakfast items and coffee/OJ at stops 1-4 (breakfast ends ~11 AM).
- All caffeinated drinks placed by stage 19, the Rockstar energy drink early (stage 5).
- Stops calorie-balanced to roughly 570-750 cal each, max 2 drinks per stop.
- Riders balanced to ~4,060 cal (3,880-4,200 total / 470-630 drink) and 12-14 items each.

### The route

Stop order minimizes total elapsed time: OSRM driving matrix over home, the rider pickup, all 30 stores and both Tesla Superchargers, 2-opt/or-opt with Sunday opening-hour time windows (5310 S 108th opens 10 AM Sunday; 4810 S 72nd and 3150 Dial Dr open 9 AM). The optimized order waits zero minutes at any window.

Day plan: leave 10901 Leavenworth St 8:00 AM, pick up riders at 609 N 43rd St ~8:17, first stop 3855 Dodge St 8:22. Then Florence, Council Bluffs (Supercharge at Metro Crossing ~9:43, ~25 min), south Omaha, Bellevue, Papillion/La Vista, Millard, Gretna, Elkhorn, the W Center Rd pair (Supercharge at 90th & Center ~2:41 PM), up the 72nd/90th/108th corridor, finish 168th & Maple ~4:57 PM. ~4.2 h driving total including the home leg. The two charge stops appear in the app as ⚡ stages (6 and 24) with no food assigned.

Plan constraints are positional against the Taco Bell sequence (breakfast at stages 1-4, done ~9:40; caffeine early-to-mid route), so they hold under the new order. Per-stop Sunday hours and the projected arrival at every stage live in `stop-verification-2026-08-29.md`.

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
