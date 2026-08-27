# Food Brawl — project notes

## Files
- `Food Brawl Demo.dc.html` — the live source. Edit this.
- `Food Brawl Demo - standalone-src.dc.html` — mirror of the source with fonts/art inlined as base64 via `A('key')`. **Every gameplay/UI change must be applied to both**, then rebuilt to `Food Brawl Demo.html` (the offline single file).
- `Food Brawl.dc.html` — the original turn-1 options/exploration doc. Leave alone.

## Game formats
Games are either **horizontal** or **vertical**, and the format decides the board markup, the preview crop, and the peek panel.

- **Horizontal** (Corgi Derby, Frog Jump, River Race): wide track art, lanes stacked vertically, runners move left→right. Preview uses a stretched fit (`100% auto`, `top center`) because the art is short and wide.
- **Vertical** (Walnut Drop, **Monkey Tree**): tall board art, one shared playfield, movers travel top↔bottom. **Do not squeeze tall art to fit a short preview frame.** Crop it instead: oversize the background (`~132% auto` in the 150px card preview, `~190% auto` in the 72px games-tab thumb) and anchor to the end the action happens at — `center bottom` for Walnut Drop's finish line. Monkey Tree runs Down to Up, so it should anchor `center top` instead, framing the treetop the monkeys are climbing toward.

Preview compositions live in one place: `shotFor(n)` in the logic (card preview + picker rows) and the `thumbCss`/`thumbSize`/`thumbPos` ternaries in `gameGroups` (games-tab thumbs).

## Constraints the user has set
- Zero backend. Everything in one `localStorage` key (`foodbrawl.v1`). No accounts, no cloud, no live service.
- Taps are cosmetic in every game — sparks and hype only, never a speed boost.
- Races finish in under 20 seconds, enforced on a wall clock, not just simulated time.
- Pro = no ads + pick tonight's race. Nothing else goes behind the paywall.
- Max 6 in a field, max 5 guests plus the owner.
- All artwork is the user's own, in `art/`. Never draw substitutes — use a labelled placeholder and say what's needed.
