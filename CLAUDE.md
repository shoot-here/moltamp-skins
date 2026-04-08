# MOLTamp Skins -- AI Contributor Guide

Community skin repository for MOLTamp. Skins are pure CSS themes -- no JavaScript.

## Repository Structure

```
moltamp-skins/
├── skins/              <- Each subfolder is a complete skin
│   └── <skin-id>/
│       ├── skin.json   <- Manifest (required)
│       ├── theme.css   <- All styles (required, 100KB max)
│       └── assets/     <- Images, GIFs, fonts (optional)
├── screenshots/        <- Skin preview images
├── SKINNING.md         <- Full specification (read this first)
├── CONTRIBUTING.md     <- PR guidelines
└── README.md           <- Overview + skin catalog
```

## Critical Rules -- NEVER Violate These

1. **All colors as CSS variables in `:root`.** Never hardcode hex/rgb in selectors.
2. **Contract variables are the API.** `--t-*` (terminal), `--c-chrome-*` (chrome), `--effect-*` (effects).
3. **No backgrounds on `.moltamp-vibes`.** Vibes art uses `skin.json` GIF slots.
4. **Every `::before`/`::after` needs `pointer-events: none`.** Always.
5. **Every animation/glow/filter must be gated with `--effect-*` variables.** No ungated effects.
6. **No external URLs, no `@import`, no `javascript:`, no `expression()`.** CSS only, zero network.
7. **No targeting Moltamp internal UI.** Hard block -- skin won't load. Stick to the documented `.moltamp-*` classes.
8. **Custom variables use `--skin-*` prefix.** E.g., `--skin-panel-bg`, `--skin-overlay`.
9. **Never use `*` (universal selector) with display, opacity, pointer-events, or position.**
10. **Never set `overflow: hidden` on `.moltamp-shell`, `.moltamp-deck`, `.moltamp-titlebar`, or `.moltamp-statusbar`.**
11. **Don't reference reserved internal chrome variable namespaces.** Several `--*` namespaces are reserved for Moltamp's internal UI and are hard-blocked by the validator. Keep all custom variables under the documented `--skin-*`, `--t-*`, `--c-*`, or `--effect-*` prefixes. You'll see a clear error on import if you stray.
12. **Don't try to hide forbidden tokens behind CSS escapes.** The validator normalizes escape sequences (both hex `\6e` and identity `\n` forms) before pattern matching, so escape tricks will still be caught. Write CSS normally.

## When Generating a New Skin

1. Read `SKINNING.md` fully -- it's the single source of truth
2. Start from an existing skin (copy a `skins/<id>/` folder)
3. Set ALL contract variables in `:root` -- don't leave defaults
4. Gate EVERY effect with `--effect-*` variables (0 = off, 1 = default, 2 = max)
5. Include a `defaultLayout` in `skin.json` with themed tab names and curated widgets
6. Include vibes GIF slots in `skin.json` if using banner art
7. Test mentally: would this break at any panel size? Would right-click menus work?

## When Modifying an Existing Skin

- Preserve the skin's identity and artistic vision
- Don't add effects without gating them
- Don't change the skin ID (breaks user settings)
- Keep `theme.css` under 100KB

## Common AI Mistakes

- Using `background` on `.moltamp-vibes` (use skin.json GIF slots instead)
- Forgetting `pointer-events: none` on pseudo-elements
- Creating ungated animations (must use `opacity: var(--effect-*)` or `calc(... * var(--effect-*))`)
- Using `.moltamp-titlebar * { }` -- this breaks buttons. Target `.moltamp-titlebar > span:first-child` for logo
- Putting hardcoded colors in selectors instead of `:root` variables
- Setting `overflow: hidden` on containers (clips context menus)
- Using `flex-direction: column` on `.moltamp-deck` (rotates entire layout)
- Not including `engine: "1.0"` in skin.json
- Targeting Moltamp internal UI elements (hard block, skin won't load — stick to documented `.moltamp-*` classes)
- Referencing a reserved internal `--*` variable namespace (validator rejects on import with a clear error)
- Trying to hide a forbidden name behind CSS escape sequences — the validator normalizes both hex (`\6e`) and identity (`\n`) escapes before matching, so escape tricks don't work
- Mentioning a protected name in a code comment and assuming it will trip the validator — comments are stripped before matching, so honest doc comments are safe
- Applying `transform` to `.moltamp-shell` (breaks `position: fixed` children)

## File Size Limits

- `theme.css`: 100KB max
- Individual asset: 5MB max
- Total assets per skin: 20MB max
- Supported images: PNG, JPG, WebP, GIF, SVG, AVIF
- Supported fonts: WOFF2 (preferred), WOFF, TTF, OTF

## skin.json Required Fields

```json
{
  "id": "my-skin",
  "name": "My Skin",
  "version": "1.0.0",
  "engine": "1.0"
}
```

## Effect Gating Quick Reference

```css
/* Pseudo-element: gate with opacity */
.panel::after { opacity: var(--effect-my-effect); }

/* Box-shadow: gate with calc */
box-shadow: inset 0 0 calc(20px * var(--effect-glow)) var(--color);

/* Filter in keyframes: neutralize at 0 */
filter: brightness(calc(1 + 0.2 * var(--effect-pulse)));

/* Animated background: gate with calc on opacity */
opacity: calc(0.25 * var(--effect-art));
```

## Effect Variable Range

All `--effect-*` variables range from `0` to `2`:
- `0` = Off (0%)
- `1` = Skin author's intended default (100%)
- `2` = Max boost (200%)
