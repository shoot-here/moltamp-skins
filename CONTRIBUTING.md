<div align="center">

<a href="https://moltamp.com">
  <img src=".github/assets/logo.png" alt="MOLTamp" width="56"/>
</a>

<br/><br/>

# Contributing to MOLTamp Skins

**Thanks for making a skin.** Here's how to get it into the repo.

</div>

<br/>

## Quick Version

1. Fork this repo
2. Create `skins/your-skin-id/` with `skin.json`, `theme.css`, and optionally `assets/`
3. Add a `preview.png` screenshot
4. Open a PR

## Skin Structure

```
skins/your-skin/
  skin.json       <- Manifest
  theme.css       <- All styles
  assets/         <- Images (optional)
  preview.png     <- Screenshot (required for PR)
```

## Guidelines

MOLTamp's validator checks these automatically. If something's off, it'll tell you what to fix.

1. **All colors in `:root` as CSS variables.** Use `var()` in selectors. Never hardcode hex/rgb.
2. **Override the contract vars.** `--t-*` for terminal, `--c-chrome-*` for chrome.
3. **Custom vars use `--skin-*` prefix.**
4. **No `background` on `.moltamp-vibes`.** Use `skin.json` vibes slots for GIFs.
5. **Every `::before`/`::after` needs `pointer-events: none`.**
6. **No external URLs, `@import`, or JS.**
7. **Assets under 5MB each, 20MB total.** PNG, JPG, WebP, GIF, SVG, AVIF only.

## Custom Effects

Want a user-toggleable effect? Declare `--effect-youreffect: 1;` in `:root`. It auto-appears in Settings > Effects as a toggle with intensity slider. The slider range is 0% to 200%, where 1 = 100% (your intended default).

Add labels in `skin.json`:
```json
{
  "effects": {
    "youreffect": { "label": "Your Effect", "description": "What it does" }
  }
}
```

## Naming Conventions

- **Skin IDs** are `lowercase-kebab-case` — e.g., `neon-horizon`, `deep-space`, `ice-nine`. Must match `^[a-zA-Z0-9_-]+$`.
- **Display names** are Title Case — e.g., "Neon Horizon", "Deep Space", "Ice Nine". Set this in the `name` field of `skin.json`.
- **Skin folder name** must match the `id` in `skin.json`.

## Preview Screenshots Required

Every PR submission must include a `preview.png` screenshot in your skin's folder. This is how people browse skins before trying them.

- **Recommended size:** 800x500 pixels
- **Show the skin in action:** panels open, some terminal content visible, effects active
- **Capture the full window** — title bar, vibes, panels, statusbar
- Take the screenshot in MOLTamp with a realistic session, not an empty terminal

## AI-Generated Skins Welcome

If you used ChatGPT, Claude, Codex, or another AI to generate your skin, that's great. AI-generated skins are welcome and treated the same as hand-crafted ones. The only requirements are:

- The skin passes validation (the validator checks this automatically)
- You've tested it visually in MOLTamp and it looks intentional
- Effects are properly gated with `--effect-*` variables
- It has a `preview.png` screenshot

See the "For AI-generated skins" section in [SKINNING.md](SKINNING.md) for a ready-to-paste prompt block that gives the AI all the rules upfront. MOLTamp's preflight system also auto-fixes common AI mistakes, so generated skins work safely even when the AI misses a rule.

## Testing Your Skin

Before submitting:

- [ ] Load it in MOLTamp (Settings > Skins > Import)
- [ ] Right-click context menus work on all panels
- [ ] Resize handles work (vibes, panels)
- [ ] Permission dialog is visible and clickable
- [ ] Looks good at different window sizes
- [ ] Toggle panels on/off in Settings > Layout
- [ ] Effects work at 0%, 100%, and 200% intensity

## Review Criteria

When maintainers review your PR, they're looking for:

1. **Validation pass** — the skin loads without errors. Warnings are OK if they're acknowledged.
2. **No security issues** — no external URLs, no path traversal, no attempts to target protected UI.
3. **Visual quality** — the skin looks intentional and polished. It doesn't have to be complex, but it should look like a finished product, not a half-started experiment.
4. **Effects properly gated** — every animation, glow, and filter is controlled by an `--effect-*` variable. Users must be able to disable any visual effect.
5. **Contract variables overridden** — the skin actually changes the look. A submission that uses all default contract values isn't a skin, it's the template.
6. **Preview screenshot included** — `preview.png` at 800x500 showing the skin in action.

We'll work with you if something needs adjustment. The goal is to get your skin in, not to gatekeep.

## Code of Conduct

Be respectful. Share your creativity. Help others succeed with their skins.

<br/>

<div align="center">

<sub><a href="https://moltamp.com">moltamp.com</a> &nbsp;&middot;&nbsp; <a href="SKINNING.md">Skinning Guide</a> &nbsp;&middot;&nbsp; <a href="README.md">Back to README</a></sub>

</div>
