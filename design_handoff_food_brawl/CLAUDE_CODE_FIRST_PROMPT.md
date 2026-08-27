# First prompt for Claude Code

Copy everything below the line into Claude Code, in an empty folder that also contains this
handoff bundle.

---

I'm porting a finished HTML prototype into a real mobile app. The design bundle is in this
folder: read `design_handoff_food_brawl/README.md` first, completely, then
`design_handoff_food_brawl/reference/Food Brawl Demo.dc.html`.

The app is Food Brawl: a room picks 3-6 foods, a 20-second mini-game runs, and the winning
food is dinner. Three games are built. There is no backend and no accounts — one local
storage key holds everything.

Please do this in order, and stop after each step so I can look:

**Step 1 — Scaffold.** Set up Vite + React + TypeScript, and Capacitor for iOS and Android.
Bundle the Patrick Hand and Newsreader fonts locally rather than loading them from Google.
Copy `design_handoff_food_brawl/art/` into the project's assets. Confirm `npm run dev` and
`npx cap run ios` both work before moving on.

**Step 2 — Port the logic.** Move the prototype's game logic across close to line-for-line:
the three race loops, the pond generator and its `POND_BAND` table, the physics constants,
`drawDaily`, `sanitize`, and the storage schema. Put it in plain TypeScript modules with no
React in them, and write tests for `sanitize` and for the three failsafes (every race must
declare a winner inside 20 seconds of wall time, including with an empty or single-entry
field). Do not retune any constant without telling me.

**Step 3 — Build the screens.** Recreate the nine screens and five sheets described in the
README, matching the colours, type and spacing exactly. Keep the visual language: flat offset
shadows, dotted rules, mono micro-labels, Patrick Hand display type. No new colours.

**Step 4 — Native seams.** Three things the prototype fakes:
- Pro entitlement must become a real StoreKit 2 / Play Billing check on launch and on
  restore. The persisted `pro` boolean is not trustworthy. One non-consumable product,
  `$0.99`, unlocking exactly one thing: picking tonight's game.
- `spotsFor(id)` must call Google Places Nearby Search, honouring the budget and distance
  filters, with Place Details and Photos for the result card. Keep the "POWERED BY GOOGLE"
  attribution and keep the almanac free of restaurant names — Places terms don't allow
  storing them.
- The deep-link sheet should open real app schemes with web fallbacks instead of displaying
  the URL.

**Step 5 — Ship prep.** Work through `design_handoff_food_brawl/APP_STORE_LAUNCH.md` with me.

Rules that are not negotiable, from the client:
- Zero backend, no accounts, no cloud sync.
- Taps during a race are cosmetic. They never affect the outcome.
- Every race finishes in under 20 seconds on a wall clock.
- Pro is one payment and unlocks only the game pick. No subscription, no ads either tier.
- Max 6 foods per race, max 5 guests plus the owner.
- All artwork is the client's own, in `art/`. Never generate or substitute art. If something
  is missing, use an obvious labelled placeholder and tell me what's needed.

Ask me questions before guessing on anything.
