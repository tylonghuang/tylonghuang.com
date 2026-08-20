# tylonghuang.com — personal site

One-page personal hub for Long Hoang Nguyen (PhD researcher, AI accountability).
Custom Hugo theme built from scratch in `layouts/`. No external theme.

The page's job: say only what Google Scholar and LinkedIn can't. Everything else links out.

## Hard constraints (never violate)

- No JS frameworks, no bundlers, no npm, no Tailwind, no CSS frameworks, no icon fonts.
- Vanilla Hugo templates + one CSS file + minimal vanilla JS (only what the spec below requires).
- No web fonts unless explicitly requested later. System font stack, with a CJK-capable fallback chain so 龙 always renders (e.g. system-ui first, then "PingFang SC", "Hiragino Sans GB", "Noto Sans CJK SC", sans-serif).
- All colors via CSS custom properties. Dark mode via `prefers-color-scheme` only — no toggle, no JS theming.
- Deployment: Netlify, linked to this repo, build configured in the Netlify dashboard (hugo build, publish dir `public/`, `HUGO_VERSION=0.165.0` env var). By choice there is no `netlify.toml` and no CI workflow in the repo — do not create either, and do not touch DNS/domain settings.
- Do not re-scaffold Hugo or rewrite `hugo.toml` wholesale.
- `themes/` stays empty.

## Page structure (single page, no navigation)

Only `content/_index.md` renders as a page.

Hero, split layout:
- Left: the origami figure (see The figure). On mobile (< ~700px) it stacks on top, scaled so it takes at most ~1/3 of viewport height.
- Right: text block:
  1. Greeting line: `Hi, I am Long 龙`
     - 龙 gets the accent color and a subtle affordance (dotted underline or similar). Clicking/tapping it toggles a small inline reveal, one line: "lóng = dragon, the literal meaning of my name." Must work on touch (no title-attribute-only tooltip). Keyboard accessible.
  2. One research sentence in plain language, e.g. "I research how to establish meaningful AI accountability across information systems, computer science, and law."
  3. Two–three short bio sentences max: research affiliation (KIT and TUM, linked but without formal titles — those live on LinkedIn), the languages/cultures line, and one personal detail. The last sentence ties the hero figure to origami and must name it via `{{ $figure }}`, never a hardcoded word, so copy and image can't drift apart.
- Link list, directly after bio, plain labeled text links (no icons), one per line or a tight inline row:
  Scholar · ORCID · LinkedIn · GitHub · Email
  URLs live under `[params.links]` in `hugo.toml`. Email stays obfuscated with the `[AT]` pattern — ask before changing.
- Footer: nothing except a small imprint/legal line placeholder (comment it in the template, content TBD).

Explicitly excluded (do not add): publication list, news/updates, CV download, photo/avatar in the layout, separate about/contact pages, hamburger/nav menus, social icon rows.

Blog: a `posts` section must exist in the templates (list + single), but no link or section renders anywhere unless at least one non-draft post exists. Zero visible trace of it until then.

## Visual style

- Quiet, typographic, near-monochrome. White/warm off-white background, near-black ink.
- One accent color, used ONLY for the 龙 character and the hub link list. Inline prose links (e.g. KIT / TUM in the bio) stay body-colored with a muted underline — the accent is a signal, not a link marker. Define `--accent` even if muted; a desaturated vermillion or warm dark gray. Keep it easy to change.
- CSS variables minimum set: `--bg`, `--ink`, `--ink-muted`, `--accent`. Dark mode redefines these under `prefers-color-scheme: dark` (dark warm gray background, off-white ink, accent adjusted for contrast).
- Minimal CSS overall. No hover animations, no transitions on links beyond color, no shadows, no rounded cards. When in doubt, less.
- Target: whole page fits in roughly one viewport on desktop.

## The figure (core feature)

The hero holds one hand-drawn origami figure. Which one is a site param, not a hardcoded asset:

```toml
[params]
  figure = "dragon"   # or "crane"
```

`layouts/home.html` loads `assets/{figure}_clean.svg`, tags the wrapper `origami--{figure}`, and derives the facing. The bio's closing noun renders from the same variable, so the copy can never name a figure the page isn't showing. Both SVGs stay in the repo; switching is one word.

**Naming.** `.origami` is the figure wrapper (it holds either one — never call it `.crane`). `.glyph` is the 龙 character in the greeting, which is a different thing entirely; keep them apart.

**Current figure: the dragon.** It carries the name — 龙 means dragon — so it needs no sentence justifying its presence. The crane did, which is why it lost.

Assets — stroke-only paths, `stroke="currentColor"`, round caps/joins, `pathLength="1"` on every path. **Do not regenerate or "optimize" the path data:**

| file | viewBox | faces | paths |
|---|---|---|---|
| `dragon_clean.svg` | `0 0 636 278` | right | `body`, `wing-near`, `wing-far`, `tail`, `neck`, `horn`, `leg-rear`, `leg-front`, `crease-1`…`crease-8` |
| `crane_clean.svg` | `0 0 784 430` | left | `body`, `neck`, `beak`, `tail`, `creases` |

- Inline the SVG into the template (Hugo `resources.Get` + `.Content`), never `<img>` — CSS must reach the paths.
- **Facing is per-figure, not a constant.** The figure sits left of the text and must face it, so the crane (drawn facing left) gets mirrored via `origami--flip` and the dragon (drawn facing right) must not be. `$flip` derives this from `$figure`; do not hardcode either way.
- The figure inherits `color: var(--ink)` so dark mode is automatic.
- Creases render lighter/thinner than the rest: `stroke: var(--ink-muted)`, stroke-width 2 vs 3.

### Draw-in animation

Stroke draw-in via `stroke-dasharray`/`stroke-dashoffset` with `pathLength="1"` on every path. Pure CSS keyframes; per-path duration/delay via custom properties, scoped under `.origami--{figure}`.

Crane reference timing (an approved prototype — keep these values):

| path    | duration | delay |
|---------|----------|-------|
| body    | 1.6s     | 0.2s  |
| neck    | 1.0s     | 1.7s  |
| beak    | 0.55s    | 2.6s  |
| tail    | 1.0s     | 3.1s  |
| creases | 1.2s     | 4.0s  |

The dragon's timings live in `main.css` and end at ~5.1s, matching the crane's 5.2s. Three rules govern any new figure:

- **`pathLength="1"` normalizes every path, so duration must track the path's real length** — otherwise a long stroke and a short one sharing a duration draw at wildly different speeds. Scale durations to hold px/s roughly constant; the crane's average is the reference. More paths should not mean a proportionally longer animation.
- **`stroke-dashoffset` walks a multi-subpath `<path>` sequentially, not simultaneously.** Split into separate elements when parts should move together (the dragon's two legs read as a glitch otherwise). Keep them combined, or stagger explicitly, when a ripple is wanted (the dragon's creases sweep head-to-tail on purpose).
- Overlap the strokes: each starts before the previous lands. No gaps.

- Easing: `ease-out` per stroke.
- Plays once per browser session: tiny inline JS sets a `sessionStorage` flag once the last path finishes; if the flag exists on load, a class skips straight to the fully drawn state (no animation, no flash of an empty figure). The flag is set by counting animation ends across `.origami path` — never by watching one named path, which breaks when the figure changes.
- `prefers-reduced-motion: reduce`: no animation ever, figure renders fully drawn.
- After the animation, the figure is static. No loops, no hover motion, no scroll effects, no page-transition effects.

## Favicon

- Keep existing `static/favicon.ico` (龙) as legacy fallback.
- Add an SVG favicon: the 龙 character as text in an SVG, with an internal `<style>` + `prefers-color-scheme` media query so the glyph is dark on light and light on dark. Wire both into the head with correct `rel`/`type` order (SVG first, ico fallback).

## Metadata

- Write a real meta description (one sentence about Long and AI accountability research). The old site shipped the theme's default description — that class of mistake must not recur.
- Sensible `<title>`, `lang` attribute, og:title/description. No analytics, no tracking, no third-party requests of any kind.

## Working agreements

- Small, focused commits.
- If the spec is ambiguous, ask; do not invent features.
