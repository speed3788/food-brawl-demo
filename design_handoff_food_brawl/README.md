# Handoff: Food Brawl (Winner picks dinner)

## Overview

Food Brawl settles the "where are we eating tonight" argument with a race. The room
picks 3-6 foods, each person backs one or more, one of several mini-games runs for
under 20 seconds, and the winning food becomes dinner. The result card deep-links to
Maps, DoorDash and Uber Eats.

The whole thing runs on one device with no backend, no accounts and no network calls
except the restaurant lookup. Three mini-games are built and playable:

| # | Game | Format | Mechanic |
|---|------|--------|----------|
| 1 | Corgi Derby | horizontal, 5 lanes | timed auto-race with per-runner wobble |
| 2 | Frog Jump | vertical board, left-to-right travel | frogs hop pad to pad across open water |
| 4 | Walnut Drop | vertical | Pachinko physics through a fresh peg board each race |

Games 3 (River Race) and 5 (Monkey Tree) are defined in the games list but marked
`built: false` — they appear as "COMING SOON" in the games shed.

## About the design files

**The files in this bundle are design references written in HTML.** They are a working
prototype of the intended look, copy and behaviour — not production code to copy into a
shipping app.

The job is to **recreate this design in a real app environment**. There is no existing
codebase, so the implementer picks the framework. See "Recommended target" below for the
recommendation and the reasoning; the implementer should feel free to disagree, but should
keep the constraints in "Product rules that must not change".

The prototype is a single self-contained HTML file whose logic is a plain JavaScript class.
The game loops, the physics, the pond generator, the daily draw and the storage schema are
all worth porting close to line-for-line — they are tuned, and re-deriving them wastes time.
The markup and inline styles are worth rebuilding idiomatically in the target framework.

## Fidelity

**High fidelity.** Colours, typography, spacing, copy and interactions are final and should
be matched. Every value below is exact. The artwork is the client's own and ships as-is.

## Recommended target

**Vite + React + TypeScript, wrapped with Capacitor for iOS and Android.**

Why:

- The prototype's logic is already a React-style class with state and a render map. Porting
  it to React is close to mechanical; a rewrite in SwiftUI or Compose would mean rebuilding
  three tuned game loops twice, once per platform.
- Capacitor gives real native app packaging (App Store and Play Store), plus native StoreKit
  and Play Billing through plugins, plus `window.open` handling for app-scheme deep links.
- The app is offline-first with a single storage key, which is exactly what a WebView-based
  wrapper is good at. There is no scroll-heavy content, no long lists, no video.

Practical alternative if the implementer prefers native: React Native with
`react-native-skia` or plain `Animated` for the three boards. Expect the game loops to need
retuning against a non-DOM renderer.

What must be replaced when leaving the prototype:

1. **Pro entitlement.** In the prototype `pro` is a boolean in local storage, trivially
   flipped. It must become a StoreKit 2 / Play Billing entitlement check on launch and on
   restore. Never trust the persisted flag as the source of truth.
2. **Restaurant lookup.** `spotsFor(id)` returns hard-coded invented restaurants. It is the
   single seam where Google Places (Nearby Search + Place Details + Photos) goes. Everything
   downstream already handles the empty-results case.
3. **Deep links.** The prototype shows the URL it would fire in a sheet rather than
   navigating. On device it should open the app scheme and fall back to the web URL.

## Product rules that must not change

These are the client's explicit constraints. They are load-bearing for the product's feel.

1. **Zero backend.** Everything in one local storage key (`foodbrawl.v1`). No accounts, no
   cloud sync, no live service.
2. **Taps are cosmetic in every game.** The big button emits sparks and counts cheers. It
   never affects speed or outcome. This keeps a shared screen fair.
3. **Every race finishes in under 20 seconds**, enforced on a wall clock, not just simulated
   time. Each game has a hard-stop failsafe that declares the current leader.
4. **Pro = no ads plus pick tonight's game.** Nothing else goes behind the paywall. Both
   tiers are ad-free today, so in practice Pro is only the game pick. One payment, no
   subscription.
5. **Max 6 foods in a field, max 5 guests plus the owner.**
6. **All artwork is the client's own**, in `art/`. Never substitute drawn or generated art.
   If something is missing, use a labelled placeholder and say what is needed.
7. **The almanac never stores restaurant names**, ratings or addresses — Google Places terms
   do not permit retaining them. It stores food name, timestamp, field size, tap count, game
   number and a `found` boolean only.

## Screens and views

The app is a single-column phone layout, 390px design width. Every screen is a flex column.
Standard screen padding is `66px 26px 24px` (the top value clears the status bar and the
floating screen title). The setup screen uses `74px 26px 40px`. The race screen uses
`56px 0 22px` because its board is edge-to-edge with its own 14px side margin.

### 1. Setup (first run only)

**Purpose:** name yourself and pick an ingredient avatar. Written to the device, never sent.

- Logo image `art/logo.png`, 236px wide, centred, with the tagline "Winner picks dinner!"
  under it in Patrick Hand.
- Text input, full width, `rgba(255,255,255,.42)` fill, 1px `#cbbb95` border, 3px radius,
  placeholder "Bradley".
- Ingredient grid: `repeat(3, 1fr)`, 10px gap, 6 options from `AV`. Each cell is a 52px-tall
  contain-fit image plus an 11px italic caption. Selected cell gets a 1px `#2b271f` ring and
  a `rgba(87,118,75,.12)` fill.
- Primary button: `#2b271f` fill, `#f7f0dd` text, 16px padding, 4px radius, Patrick Hand 26px,
  label "Start Brawl!".
- Footnote, 12.5px italic `#7d7460`: "Saved to your device only. No account, no cloud,
  nothing to log into."
- If the name is left blank, submitting opens the "Shall we call you Bradley?" sheet instead
  of proceeding.

### 2. Home

**Purpose:** the launch pad. Shows tonight's game, the current field, and recent results.

- Header row: 48px circular avatar (1px `#cbbb95` border, white-ish fill, ingredient image
  inside), then a greeting ("welcome back" once there is history, otherwise "good evening")
  and the user's name in Patrick Hand 30px. Right side shows the settled-race count under a
  9.5px mono "SETTLED" label.
- **Tonight's race card:** 1px `#cbbb95` border, 6px radius, `rgba(255,255,255,.42)` fill,
  18px padding, with a decorative `#57764b` blob at 10% opacity clipped to the top-right
  corner (`border-radius: 50% 0 50% 0`).
  - 9.5px mono "TONIGHT'S RACE" label, then the game name in Patrick Hand 30px.
  - A 150px-tall preview frame, 5px radius, overflow hidden, showing the game's art. The
    composition per game lives in `shotFor(n)` — see "Preview compositions" below.
  - Top-left ribbon on the preview: 8.5px mono, cream on `rgba(43,39,31,.75)`, showing how
    the game was chosen ("your pick", "drawn from your 3 starred", "drawn from all games").
  - Bottom-right button: "Change" for Pro, or a locked state for free users.
  - Game description, 13px italic `#6b6250`.
  - Field chips: pill `999px`, 1px `#d6c8a4`, 13px, each with a 7px diamond swatch in the
    food's tone colour (a square rotated 45° with `border-radius: 50% 0 50% 0`).
  - Primary button "To the starting line", Patrick Hand 28px.
- **Lately:** up to 3 history rows. Each is a 52px mono timestamp column, then the winning
  food in Patrick Hand 20px plus an italic detail line, separated by a 1px dotted rule
  `rgba(120,95,50,.3)`. A link to "the whole almanac" sits in the section header.
- Empty state: 1px dashed `#cbbb95`, 6px radius, 20px padding, centred 13.5px italic: "The
  almanac is empty. Win a race and it starts keeping score."

### 3. Filters ("Set the rules")

**Purpose:** budget, distance and the field of foods that can run.

- Budget: 4 equal buttons `$`, `$$`, `$$$`, `$$$$`, 13px vertical padding, 3px radius.
  Selected is `#2b271f` on cream; unselected is `rgba(255,255,255,.4)` with a `#cbbb95` border.
- Distance: a range input (accent `#a8781c`), 1-25 miles, with the ends labelled "next
  street" and "worth the drive".
- Who's running: three chip groups under 9.5px mono headers CATEGORY, FOOD, YOURS. Chips are
  `999px` pills, 9px 15px padding, 15px text, with the tone diamond. Selected chips invert to
  `#2b271f` on cream.
- Field counter, 12.5px italic: "N of 6 · three minimum".
- Save button: label and colour switch on validity — "Save the rules" on `#2b271f` at 3+
  foods, "Pick three or more" on `#8f8874` below that. Toggling off is blocked at 3; toggling
  on is blocked at 6.

### 4. Paddock

**Purpose:** say who is backing which food before the race.

- Explainer with a 2px `#cbbb95` left rule: "Tap a runner to say who's backing it. One person
  can back several, and two people can share one."
- One row per food: 1px border, 5px radius, 10px 11px padding. Left side is a 52x42 contain
  image of the food's corgi art plus the food name in Patrick Hand 20px and an italic silks
  line ("brick and oat", "honey and cream" — each food's `silks` string). Right side is a
  dashed `999px` chip showing 17px avatar thumbnails of everyone backing it, or "+ back" when
  empty. Rows with a backer get a `0 2px 0 rgba(43,39,31,.22)` shadow.
- Backer summary line, 12.5px italic, counts backed runners and nudges when some are open.
- Primary button "To the starting line".
- Tapping a row opens the claim sheet (see Sheets).

### 5. Race

**Purpose:** run the game. One of three boards renders based on `gameN`.

- Header: headline in Patrick Hand 30px, clock in Patrick Hand 26px `#a8781c`. The headline
  changes by phase: pre-start ("Under starter's orders" / "Balls in the hopper" / "On the lily
  pads"), first two seconds ("And they're off" / "And they drop" / "And they hop"), then a
  live leader line, then "Photo finish" on completion.
- **Corgi Derby board:** `art/track.png` at `100% auto`, `top center`. A 64px spacer, then one
  66px lane per runner with a 1px dotted bottom rule. Runners sit at `top: 58%` inside a lane
  inset `left: 34px; right: 26px`, as a centred flex column of backer plate, name plate, and a
  54x36 sprite. Plates are `rgba(247,240,221,.86)`, 2px radius; backer in Patrick Hand 15px,
  food name in 11px italic.
- **Frog Jump board:** 14px side margin, height 540px, `art/frogjump_board.png` at `cover`,
  `top center`, water colour `#8aacb4` behind it. Lily pads are 34x19 absolutely positioned
  divs, `translate(-50%,-50%)`, `z-index` = rounded y. Frogs are flex columns anchored
  `translate(-50%,-100%)` with `top = y + 5 - hop`, so the sprite's feet land on the pad
  surface; `z-index` = rounded y + 200. Sitting sprite 34x23, jumping sprite 40x31.
- **Walnut Drop board:** 346x520, `art/board_background.png`, pegs and bumpers as
  absolutely positioned sprites, balls as rotating sprites with name and backer plates.
- Spark particles: 8-12 per tap, `50% 0 50% 0` radius, colours `#a8781c` and `#57764b`,
  gravity 60px/s², life 0.5-0.75s, array capped at 42.
- Big button, full width, Patrick Hand: `#57764b` before the start ("Start Race" / "Drop the
  Balls" / "Start Hopping"), `#2b271f` during the race ("Let's Go!"), `#8f8874` once won.
  Meta line above it shows the field size pre-race and the cheer count during.

### 6. Result card

**Purpose:** announce the winner and hand off to a food app.

- "the winner is" in mono, then the food name in Patrick Hand at display size with its art.
- Winner blurb from the per-game blurb list, e.g. "by a whisker, out of 5 in the field".
- Backer line: names of everyone who backed it, or "nobody backed it, but you're still
  eating it".
- Restaurant list: up to 4 invented spots with rating and distance. **This is the Places
  seam.** Header carries the required "POWERED BY GOOGLE" attribution, and a query line
  showing what was asked for: `PIZZA · ≤6MI · $$ · +PHOTOS`.
- Photo strip: empty frames labelled "PLACES PHOTO" — the real build fills these from photo
  references.
- Three deep-link buttons per spot: Maps, DoorDash, Uber Eats.
- No-results state: "No restaurants nearby. Run it again!" plus "Run it again without
  <food>" (which bars that food from the next race) and "Widen the rules instead".

### 7. Almanac

**Purpose:** the record of settled dinners.

- STANDINGS: up to 5 foods by win count, each with a proportional bar (`pct` of the top
  winner) in the food's tone colour.
- EVERY RACE: full history, newest first, capped at 40. Each row is a mono timestamp, a
  32px avatar, the winning food in Patrick Hand, and a detail line. **The avatar reflects the
  game played** — a frog for Frog Jump, a food ball for Walnut Drop, a corgi for Corgi Derby
  (`almanacImg(name, g)`). Entries recorded before this field existed fall back to a corgi.
- Empty state: "Nothing pressed in here yet. Run a race and it starts keeping score."

### 8. Potting shed (settings)

**Purpose:** everything stored on the device, editable.

- YOU: name and ingredient avatar.
- THE ROOM: up to 5 guests, each with a name and an auto-assigned ingredient. Add field plus
  a limit notice at 5.
- CUSTOM FOODS: add up to 20. Also lists built-in foods that have been hidden, with a "Bring
  back" action.
- PRO: purchase entry point, or "Pro is on" with a demo-only undo.
- THIS DEVICE: a storage summary line naming exactly what is stored and its rough size, and
  "Clear everything and start over".

### 9. Games shed

**Purpose:** browse the games, star favourites, and (Pro) pick tonight's.

- Grouped by `GAME_GROUPS`: "Racing & Track Matches" and "Physics & Gravity Drops".
- Each row: 72px art thumb, game name in Patrick Hand, direction and description, a star
  toggle (`★`/`☆`, `#a8781c` when starred), and a READY TO PLAY / COMING SOON badge.
- Starring biases the free daily draw; if nothing is starred, the draw uses every built game.
- Tapping a built game opens a 5-second looping sample ("5-SECOND SAMPLE · LOOPING").

### Sheets (modals)

All sheets are bottom-anchored panels over a scrim, with a "Close" or "Done" action.

- **Claim sheet** — "Who's on <food>?" Toggle any subset of the room; multiple people per
  food allowed.
- **Name confirm** — "Shall we call you Bradley?" with "Yes, that's me" and "Let me type my
  own".
- **Pro paywall** — "One payment. You pick tonight's game." Three benefit lines, "Buy for $1",
  "Maybe later".
- **Deep link** — shows the URL that would fire, plus which app it opens and its web
  fallback, and "Back to the card".
- **Game peek** — the 5-second looping sample from the games shed.

## Interactions and behaviour

### Race loops

Each game has its own `requestAnimationFrame` tick. All three clamp `dt` so a stalled frame
cannot teleport anything, and all three track both simulated time and wall time.

- **Corgi Derby** (`tick`): each runner gets `dur = raceSeconds * (0.86..1.14)` plus two
  sine wobbles (`0.055*sin(1.9t + φ1) + 0.032*sin(4.3t + φ2)`) damped as it approaches the
  line. Default `raceSeconds` is 15 and is exposed as a tweakable prop. Failsafe: past
  `raceSeconds + 5` the current leader is declared.
- **Frog Jump** (`tickFrog`): each frog waits `WMIN..WMAX` seconds, then picks a random
  unoccupied pad within `REACH` that is forward of it, and hops with `sin(πt)` arc height
  `ARC`. Pads are single-occupancy, tracked in a `Set`. From `RAMP_AT` seconds the reach and
  speed ramp by `RAMP_C` per second so no frog can stall. Reaching `FIN` sends it to the bank
  and finishes. Failsafe at `HARD_STOP` 18s simulated / `WALL_STOP` 19s wall.
- **Walnut Drop** (`tickPlinko`): gravity 200px/s², restitution 0.42, peg bounce 0.9, terminal
  velocity clamps on both axes, plus a funnel force toward `FUNNEL_TO` after the ramp point so
  balls converge on the finish. Failsafe at 18s/19s.

Tuning constants live in the `FROG` and `PLINK` objects and should be ported verbatim:

```
FROG  = {W:374, H:540, REACH:58, JUMP:0.50, ARC:26, MINSEP:28, CR:48, SIDE:0.35,
         BRIDGE_CAP:10, WMIN:0.1, WMAX:1.0, RAMP_AT:13, RAMP_C:0.6,
         HARD_STOP:18, WALL_STOP:19, Y0:176, Y1:527, FIN:302, INSET:15}
PLINK = {W:346, H:520, R:11, PEG:6, BUMP:11, G:200, REST:0.42, BOUNCE:0.9,
         VY_MAX:32, VX_MAX:80, FINISH:458, ROWS:11, RAMP_AT:9, RAMP_C:2,
         HARD_STOP:18, WALL_STOP:19, FUNNEL_TO:130, FUNNEL_K:1.9}
```

### Pond generation (Frog Jump)

Worth reading the source directly — it took several passes to get right.

`POND_BAND` is a hand-sampled table of the water's left and right edge at 14 y values, taken
from `art/frogjump_board.png`. `bandAt(y)` interpolates it; `inWater(x, y)` adds a 15px inset
so pads never overlap the bank. `buildPond()` then:

1. Places 6 start pads hugging the left waterline, vertically spaced so nameplates never
   stack.
2. Scatters pads on a jittered 6x10 grid across the whole water body, skipping about 24% of
   cells so open water survives. `MINSEP` 28px rejection keeps them from clumping.
3. Guarantees each start pad has at least 3 forward options within reach.
4. Bridges true dead ends only, capped at `BRIDGE_CAP` additions, so the pond stays open
   rather than turning into a carpet.

If the board art is ever redrawn, `POND_BAND` must be re-sampled.

### Backgrounding

`requestAnimationFrame` stops when the app backgrounds, which would leave the wall-clock
failsafe ready to fire the instant the user returns. A `visibilitychange` handler pauses the
loop and rebases `_t0` by the elapsed gap on return, so a race resumes where it left off.
Port this — on a real phone it fires constantly.

### The daily draw

Free users get one game per day, drawn by `drawDaily(favs)` from a seed derived from the UTC
date, weighted to starred games (or all built games if none are starred). Same day, same
device, same game. Pro users bypass it entirely via `chosen`.

### Storage and validation

One key, `foodbrawl.v1`, holding: `name`, `avatar`, `budget`, `distance`, `picked`, `people`,
`claims`, `customs`, `hiddenFoods`, `history`, `pro`, `favs`, `chosen`.

Everything read from storage passes through `sanitize()` before it reaches state: it clamps
numeric ranges, drops unknown or hidden food ids from the field, tops the field back up to
three, strips claims that point at deleted people or unpicked foods, enforces the 40/20/5
caps, and rejects a `chosen` game that is not built. **Keep this pattern.** Any new persisted
key should be validated in the same place. Writes are wrapped in try/catch and toast on
failure rather than silently losing the almanac.

### Input limits

Custom food and guest names are capped at 24 characters, must contain at least one letter or
number in any script (`/[\p{L}\p{N}]/u`), are dedupe-checked case-insensitively, and get a
collision-proof id (`Date.now()` base36 plus a counter).

## Design tokens

### Colour

| Token | Hex | Use |
|-------|-----|-----|
| Paper | `#d9d0bc` | app background, under two subtle overlays |
| Ink | `#2b271f` | primary text, primary buttons, selected chips |
| Cream | `#f7f0dd` | text on ink, nameplate fills |
| Gold | `#a8781c` | accent, links, clock, stars, range thumb |
| Gold hover | `#8a6110` | link hover |
| Moss | `#57764b` | go button, positive accents |
| Border | `#cbbb95` | standard 1px border |
| Border soft | `#d6c8a4` | chip borders |
| Border dash | `#bdae8c` | dashed rules |
| Muted 1 | `#6b6250` | italic body |
| Muted 2 | `#7d7460` | footnotes |
| Muted 3 | `#8a8069` | summary lines |
| Mono label | `#9b8f73` | uppercase micro labels |
| Faint | `#b0a488` | disabled glyphs, remove × |
| Spent | `#8f8874` | disabled buttons, finished state |
| Pond | `#8aacb4` | water behind the frog board |
| Track | `#a89a6f` | ground behind the derby track |

Body background is `#d9d0bc` plus
`radial-gradient(120% 80% at 50% 0%, rgba(255,255,255,.5), transparent 70%)` and
`repeating-linear-gradient(96deg, rgba(120,95,50,.05) 0 3px, transparent 3px 9px)` — a faint
paper tooth. Card fills are `rgba(255,255,255,.42)`, or `.5` for chips.

Food tone colours come from each entry's `tone` field in `CU`; custom foods cycle
`CUSTOM_TONES = ['#7d5ba6','#2f6f6a','#9d4a4a','#5c7a2f','#a8781c','#8c4a33','#4f6f6a','#6b5340']`.

### Typography

- **Patrick Hand** (Google Fonts) — display. Screen titles 30px, card titles 24-30px, buttons
  26-28px, nameplates 14-20px. Never below 14px.
- **Newsreader** (Google Fonts, 300-600 plus italic) — body serif, default for `html, body`,
  with `Georgia, serif` fallback. Body 13-15px; helper text is italic at 12.5-13.5px.
- **ui-monospace / Menlo** — micro labels only. 8.5-9.5px, uppercase,
  `letter-spacing: .05em` to `.12em`, colour `#9b8f73` (or cream on dark ribbons).

Both webfonts must be bundled locally in the native build — no CDN dependency in an
offline-first app.

### Spacing, radius, shadow

- Screen padding `66px 26px 24px`; section gaps 16-24px; row gaps 9-14px.
- Radius: 3px inputs and small buttons, 4px primary buttons, 5-6px cards, 2px nameplates,
  `999px` chips. The decorative leaf shape is `border-radius: 50% 0 50% 0`.
- Shadows are flat and offset, never blurred soft: `0 2px 0 rgba(43,39,31,.22)` and
  `0 3px 0 rgba(43,39,31,.2)`. Sprites use `drop-shadow(0 2px 3px rgba(43,39,31,.4))`.
- Dotted rules: `1px dotted rgba(120,95,50,.3)`. Dashed dividers:
  `repeating-linear-gradient(90deg, #cbbb95 0 5px, transparent 5px 10px)`.
- Scrollbars are hidden (`*::-webkit-scrollbar { width: 0; height: 0 }`).

### Preview compositions

Card previews and picker thumbs are composed in one place — `shotFor(n)` for the 150px card
preview and picker rows, and the `thumbCss`/`thumbSize`/`thumbPos` ternaries for the 72px
games-tab thumbs. Format decides the crop:

- **Horizontal games** (Corgi Derby, Frog Jump, River Race): stretched fit, `100% auto`,
  `top center` — the art is short and wide.
- **Vertical games** (Walnut Drop, Monkey Tree): do not squeeze tall art into a short frame.
  Oversize it (`~132% auto` in the 150px preview, `~190% auto` in the 72px thumb) and anchor
  to the end where the action is: `center bottom` for Walnut Drop's finish line,
  `center top` for Monkey Tree's canopy.

## Assets

All artwork in `art/` is the client's own and ships as-is. 55 files:

- `logo.png` — wordmark, used at 236px wide.
- `corgi-1..10.png` — runner sprites, also reused as generic food avatars via `IMG` and
  `CORGI_POOL`.
- `track.png`, `finish_line_zone.png` — Corgi Derby board.
- `frogjump_board.png` (748x1080, the source of `POND_BAND`), `frogjump_preview.png`,
  `lily_pad_1..4.png`, `frog_sit_1..6.png`, `frog_jump_1..6.png` — Frog Jump.
- `board_background.png`, `obstacle_peg.png`, `obstacle_peg_2.png`, `obstacle_peg_3.png`,
  `food_ball_1..6.png` — Walnut Drop.
- `ing-tomato.png`, `ing-porcini.png`, `ing-garlic.png`, `ing-thyme.png`, `ing-onion.png`,
  `ing-leek.png` — the six ingredient avatars.
- `leaf-a..h.png` — decorative leaves.
- `boost_spark_effect.png` — tap spark sprite.

Missing art, needed before those games ship: River Race board and boats, Monkey Tree trunk
and monkey sprites.

## Files in this bundle

- `reference/Food Brawl Demo.dc.html` — the design source. Human-readable markup and logic,
  loads art from `art/`. **Read this one.**
- `reference/support.js` — runtime helper the prototype's markup layer needs. Not part of the
  design; do not port.
- `art/` — all 55 artwork files, ready to drop into the new project.
- `APP_STORE_LAUNCH.md` — store submission checklist, IAP setup, Places API setup.
- `GITHUB_SETUP.md` — getting this into a repository, written for a first-time user.
- `CLAUDE_CODE_FIRST_PROMPT.md` — paste this into Claude Code to start.

A fully self-contained playable build (all art inlined, opens offline in any browser) exists
in the project as `Food Brawl Demo.html`. It is 19MB and is excluded from this bundle; ask for
it separately if you want to hand a playable copy to someone.
