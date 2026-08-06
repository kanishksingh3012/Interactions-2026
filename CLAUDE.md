# Interactions-2026 — 14 Days of Interactions

A two-week challenge: one interaction, designed and shipped every day, posted to LinkedIn/X. Vanilla HTML/CSS/JS — no build step, no framework. Use GSAP (via CDN script tag) when it's the right tool for the animation; the native Web Animations API (`element.animate()`) is a fine alternative when it isn't — pick whichever fits the interaction, don't force GSAP by default.

Repo: `github.com/kanishksingh3012/Interactions-2026`
Live gallery: `https://kanishksingh3012.github.io/Interactions-2026/`

## Repo structure

- `interactions/day-0N-slug/source-code.html` — pure component code. No tweak panel, no debug UI. This is the "here's the code" file — fully self-contained, works if opened alone.
- `docs/day-0N-slug/demo.html` — same interaction, live on GitHub Pages, with an optional tweak panel for exploring parameters. Not every day needs a panel — only add one if the interaction has meaningful tunables, and its rows are hand-written per day (content varies too much to share/generate).
- `docs/index.html` — gallery landing page. Data-driven: add `{ day: N, title: '...', slug: 'day-0N-slug' }` to the `days` array in its `<script>` block; the grid auto-fills "Coming soon" cards for the rest.
- `README.md` — one table row per day, linking source + live demo.
- GitHub Pages is already configured: Settings → Pages → Deploy from branch → `main` / `/docs`. Redeploy after a push isn't instant — it can take anywhere from ~30s to several minutes depending on load; a 404 or stale page checked immediately after pushing doesn't mean something's broken, it may just mean the build hasn't finished yet. Verify with `curl -sI <url>` after a short wait, or check ground truth at `api.github.com/repos/<owner>/<repo>/deployments` → latest deployment's `/statuses`.

## Why source and demo duplicate code instead of sharing a file

Each HTML file is intentionally fully self-contained — no cross-file `<script src>` between `interactions/` and `docs/`. Any single file can be copied out, pasted into a gist, or opened alone and it just works. Tradeoff, accepted deliberately: if a bug is found after publishing, it needs fixing in both copies.

## Naming convention

- kebab-case, zero-padded day number, no spaces or special characters: `day-01-magnetic`, not `Day 1 - Magnetic`. (This repo previously broke on a space+colon in a local path — don't repeat that.)
- Keep the `interactions/` and `docs/` slug identical for the same day so they're trivially correlated.

## Daily workflow

1. Lock the concept **and the final slug together**, before any file exists (`day-0N-slug`, kebab-case, matching the naming convention above). This is the one decision that prevents a rename later — get it right here, not after something's built.
2. Design pass in Claude.ai (chat/artifacts): describe the concept, get back one self-contained standalone HTML file. Iterate on look/feel here — regenerating the whole file is cheap before it's "real" repo code.
3. Hand the standalone file + slug to Claude Code. It checks behavior against intent, confirms any fixes with you before writing them, implements, and verifies — real browser (golden path + edge cases) plus math-simulation checks for anything animation-driven (the preview pane can't be trusted for rAF/WAAPI timelines; see lessons below).
4. Split into `interactions/day-0N-slug/source-code.html` (clean) and `docs/day-0N-slug/demo.html` (+ panel if warranted, + a "View source" link back to that day's file on GitHub).
5. Add the day to `docs/index.html`'s `days` array and a row to `README.md`.
6. Ship via `git add`/`commit`/`push` from the terminal — not GitHub's web upload UI. One atomic commit for the whole day (folders, index, README together); no partial states. Claude always states what's about to be pushed and waits for your OK before pushing.
7. Don't judge deploy success from a check made right after pushing — see the Pages timing note above. Re-check after a short wait, or confirm via the deployments API.

## Renaming a published day

Should be rare now that the slug is locked in step 1 above. If it still comes up: a rename touches 5 places — both folder names, the slug in `docs/index.html`, the README row, and inside the demo file (`<title>`, heading, "View source" link). With git-based shipping this is just a normal commit (`git mv` + edit references + push) — it lands atomically, no manual delete-then-reupload dance.

## Technical lessons learned (Day 1: Magnetic Button)

- GSAP must own `transform` exclusively on an animated element — nothing else (CSS, other JS) should ever write to it, or an unrelated update will snap the element back to its start position mid-animation.
- Animate independent visual channels (e.g. `x/y` for position vs `scale` for a press effect) as separate `gsap.to()` calls — `overwrite:'auto'` only cancels conflicting tweens on the *same* property, so they won't fight each other.
- When re-tweening toward a moving target every animation frame, skip the re-tween if the target barely moved (e.g. < 0.5px) — otherwise a fresh tween every frame with `overwrite:'auto'` causes visible jitter.
- Damping a *direction* (angle) frame-to-frame: don't lerp the x/y vector and renormalize — it mathematically locks up when the target is ~180° from the current direction (interpolating antipodal vectors stays collinear, so renormalizing snaps right back to a fixed point that never rotates). Lerp the angle itself instead (`atan2`, wrap to [-π, π], lerp, convert back with `cos`/`sin`) — always takes the shortest rotational path.
- `requestAnimationFrame` (and GSAP's own ticker, which depends on it) fully pauses when a tab/pane is backgrounded. Don't trust visual screenshots in a sandboxed/headless preview to verify rAF-driven animation — verify the math directly instead (mock `gsap.to` to record calls, invoke the tick function manually with controlled inputs).

## Technical lessons learned (Day 2: Spring Pagination)

- Web Animations API: `fill: 'backwards'` does **not** persist the final value after an animation ends (only `'forwards'`/`'both'` do). Don't depend on fill at all — write the destination to inline style *before* calling `.animate()` and let the keyframes describe only the journey. Otherwise the element snaps back to its last inline value the moment the animation finishes.
- Before starting a replacement animation, re-read the element's live rendered position (`new DOMMatrixReadOnly(getComputedStyle(el).transform).m41`) and start from there — otherwise interrupting a transition jumps to the logical position first.
- For a stretch-and-settle "rubber band" indicator, animate **centre + width**, not left + width. With left+width the spring lands on a segment that doesn't move when travelling backward (so only one direction springs), and moving the spring onto width squashes the element to a sliver on long jumps. Centre carries the spring; width is a pure outward bulge.
- When a spring on one channel can't be expressed as per-segment CSS easings on another, sample the curves in JS (~60 keyframes, played back with `easing: 'linear'`). A ~20-line `cubic-bezier()` evaluator (Newton-Raphson to invert x(t), then read y) keeps existing `cubic-bezier`-based tuning parameters meaningful.
- **Testing animations without a real browser:** the preview pane freezes rAF/WAAPI timelines, so a tween that plays fine and then snaps at the end is indistinguishable from a correct one. Two checks that do catch it: (1) force completion with `el.getAnimations().forEach(a => a.finish())` then assert the committed geometry; (2) monkey-patch `el.animate` to capture the generated keyframes and assert on their start/end values, overshoot extremes, and min/max size.
- The preview pane may refuse to re-execute a document even with `force: true` or a cache-busting query — JS state set by earlier test calls can survive "reloads". When initial-load state is in question, read it from the source file, not the live pane.
- A spring's overshoot lives on the pill's *centre*, but what's rendered is centre ± half the width — and if width is still bulged when centre overshoots, the two combine into a leak far past where the animation should visually stop (measured: 77px leak past a 16px dot on a 3-step jump, unclamped). Fix without discarding the spring: build each frame from two independent edges instead of centre+width directly. Pass the trailing edge through raw (it never leaked). Clamp only the leading edge to `destinationEdge ± smallBounce`. Rebuild centre/width from those two clamped edges for the actual keyframe — symmetric for both directions and every step count for free, since it's a per-frame clamp, not a different easing curve.
- To get a numeric before/after on an overshoot fix (rather than eyeballing it), pull the exact frame-generation formulas into a standalone Node script and simulate across cases (both directions, 1-step and multi-step) — assert on concrete values like max edge overshoot and final rest position, not just "looks right." Caught the 77px leak this way before ever opening a browser.

## Design conventions used so far (not mandatory for every future day)

- Fonts: Space Grotesk (headings/labels) + JetBrains Mono (mono/panel UI), via Google Fonts.
- Day 1 palette: warm neumorphic beige (`#dcd5c4`) / dark (`#221f19`), gold accent `#d9a441`.
- If a day has a tweak panel, the floating dark-glass panel style from Day 1 (see its CSS) is there to reuse for series consistency — but it's a preference, not a rule. Some interactions may call for a different visual language entirely.
