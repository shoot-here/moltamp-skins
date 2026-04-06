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

Want a user-toggleable effect? Declare `--effect-youreffect: 1;` in `:root`. It auto-appears in Settings > Effects as a toggle with intensity slider.

Add labels in `skin.json`:
```json
{
  "effects": {
    "youreffect": { "label": "Your Effect", "description": "What it does" }
  }
}
```

## Testing Your Skin

Before submitting:

- [ ] Load it in MOLTamp (Settings > Skins > Import)
- [ ] Right-click context menus work on all panels
- [ ] Resize handles work (vibes, panels)
- [ ] Permission dialog is visible and clickable
- [ ] Looks good at different window sizes
- [ ] Toggle panels on/off in Settings > Layout

## Using AI

You can use ChatGPT, Claude, Codex, etc. to generate skins. See the "For AI-generated skins" section in [SKINNING.md](SKINNING.md) for a ready-to-paste prompt. MOLTamp's preflight system fixes common AI mistakes automatically.

## Code of Conduct

Be respectful. Share your creativity. Help others succeed with their skins.

<br/>

<div align="center">

<sub><a href="https://moltamp.com">moltamp.com</a> &nbsp;&middot;&nbsp; <a href="SKINNING.md">Skinning Guide</a> &nbsp;&middot;&nbsp; <a href="README.md">Back to README</a></sub>

</div>
