# 14 Days of Interactions

A two-week challenge: one interaction, designed and shipped every day. Vanilla HTML/CSS/JS — no build step, no framework. GSAP where it's the right tool, native Web Animations API where it isn't.

**Live gallery:** `https://kanishksingh3012.github.io/Interactions-2026/` — enable it under Settings → Pages → Deploy from a branch → `main` / `/docs`.

## Structure

- **`interactions/`** — pure source for each day. No panel, no debug tooling — just the component as you'd drop it into a real project.
- **`docs/`** — served live via GitHub Pages. Each day gets a demo page (same interaction, plus a tweak panel for exploring the parameters where one makes sense), and `docs/index.html` is the gallery linking to all of them.

Source and demo intentionally duplicate the interaction's code rather than sharing a file — each one stays fully self-contained and portable (copy one file, it works standalone), at the cost of needing to re-copy a fix into both if one comes up after publishing.

## Days

| Day | Interaction | Source | Demo |
|---|---|---|---|
| 01 | Magnetic Button | [source](interactions/day-01-magnetic/source-code.html) | [demo](https://kanishksingh3012.github.io/Interactions-2026/day-01-magnetic/demo.html) |
| 02 | Dots with Labels | [source](interactions/day-02-pagination-dots/source-code.html) | [demo](https://kanishksingh3012.github.io/Interactions-2026/day-02-pagination-dots/demo.html) |
