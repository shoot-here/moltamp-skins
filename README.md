<div align="center">

<img src="https://moltamp.com/images/og-image.png" alt="MOLTamp — Skinnable Shell for AI Terminals" width="100%"/>

<br/><br/>

# Moltamp Skins

**Community CSS themes for the Moltamp cockpit shell — vibe coding for Claude Code, Codex CLI, Gemini CLI, and Aider.**

[![License: MIT](https://img.shields.io/github/license/shoot-here/moltamp-skins?style=flat-square&color=4d9fff&labelColor=08080a)](LICENSE)
[![Skins](https://img.shields.io/badge/skins-11-ff6b4d?style=flat-square&labelColor=08080a)](#featured-skins)
[![Spec](https://img.shields.io/badge/spec-SKINNING.md-4d9fff?style=flat-square&labelColor=08080a)](SKINNING.md)
[![Gallery](https://img.shields.io/badge/gallery-moltamp.com-ff6b4d?style=flat-square&labelColor=08080a)](https://moltamp.com/community)

**[Download Moltamp](https://moltamp.com/download)** &nbsp;&middot;&nbsp; **[Browse the Gallery](https://moltamp.com/community)** &nbsp;&middot;&nbsp; **[Skinning Guide](SKINNING.md)** &nbsp;&middot;&nbsp; **[Contributing](CONTRIBUTING.md)**

</div>

<br/>

## What's this repo?

This is the open-source theme library for **[Moltamp](https://moltamp.com)** — a skinnable cockpit UI for AI coding terminals like **Claude Code**, **Codex CLI**, **Gemini CLI**, and **Aider**. Think Winamp, but for the terminal you actually code in.

A Moltamp skin is **pure CSS**. No JavaScript, no build step, no dependencies. Drop a folder into Moltamp and the entire shell — terminal colors, side panels, vibes banner, telemetry ticker, animations — restyles itself. Skins react to live AI state through CSS data attributes, so your theme can pulse when Claude is thinking, dim when idle, or glow when a tool fires.

If you've ever wanted your AI terminal to feel like an LCARS console, a Blade Runner noir scene, a phosphor CRT, or a synthwave dashboard — this is the place.

> **Keywords:** Claude Code themes, Claude Code skins, AI terminal themes, terminal customization, vibe coding, Codex CLI themes, Gemini CLI themes, Aider themes, Moltamp.

<br/>

## Featured Skins

<table>
<tr>
<td align="center" width="33%"><b>Obsidian</b><br/><sub>Clean dark default</sub></td>
<td align="center" width="33%"><b>Phosphor</b><br/><sub>Green CRT terminal</sub></td>
<td align="center" width="33%"><b>Blade Runner</b><br/><sub>Amber noir with rain</sub></td>
</tr>
<tr>
<td align="center"><b>Deep Space</b><br/><sub>Cyan space station</sub></td>
<td align="center"><b>Kosmos</b><br/><sub>Soviet space program</sub></td>
<td align="center"><b>Biodiagnostic</b><br/><sub>Teal medical lab</sub></td>
</tr>
<tr>
<td align="center"><b>Ice Nine</b><br/><sub>Frozen blue crystalline</sub></td>
<td align="center"><b>LCARS</b><br/><sub>Star Trek computer</sub></td>
<td align="center"><b>Lunar</b><br/><sub>Moon phase observatory</sub></td>
</tr>
<tr>
<td align="center"><b>Neon Horizon</b><br/><sub>Synthwave magenta/cyan</sub></td>
<td align="center"><b>Deep Claw</b><br/><sub>Industrial amber brutalism</sub></td>
<td align="center"><i>Your skin here</i><br/><sub><a href="#contribute-a-skin">submit a PR</a></sub></td>
</tr>
</table>

Browse every community skin (with live previews) at **[moltamp.com/community](https://moltamp.com/community)**.

<br/>

## What's in this repo

```
moltamp-skins/
├── skins/              <- One folder per skin (drop-in installable)
│   ├── obsidian/
│   ├── phosphor/
│   ├── blade-runner/
│   └── ... (11 total)
├── screenshots/        <- Preview images for the README
├── SKINNING.md         <- Full authoring spec — variables, classes, reactive data
├── CONTRIBUTING.md     <- PR checklist + review criteria
└── README.md
```

<br/>

## Install a skin

**Inside Moltamp** (recommended):

> Settings &rarr; Skins &rarr; Import &rarr; pick the skin folder or `.zip`.

**From the command line:**

```bash
git clone https://github.com/shoot-here/moltamp-skins.git
cp -r moltamp-skins/skins/blade-runner ~/Moltamp/skins/
```

Restart Moltamp and pick your new skin from the skin browser.

<br/>

## Make a skin in 60 seconds

A skin is two files in a folder:

```
skins/my-skin/
  skin.json         <- manifest
  theme.css         <- all the styling
  assets/           <- optional: GIFs, images, fonts
```

**`skin.json`**

```json
{
  "id": "my-skin",
  "name": "My Skin",
  "version": "1.0.0",
  "author": "Your Name",
  "description": "What it looks like in one line.",
  "engine": "1.0"
}
```

**`theme.css`** (the bare minimum)

```css
:root {
  /* Terminal colors */
  --t-foreground: #e0e0e8;
  --t-background: #0a0a0f;

  /* Chrome colors */
  --c-chrome-bg: #0a0a0f;
  --c-chrome-accent: #d4a036;

  /* Custom palette — use the --skin-* prefix */
  --skin-overlay: rgba(7, 11, 7, 0.15);

  /* Toggleable effects appear in Settings automatically */
  --effect-scanlines: 1;
}

/* React to live AI state */
[data-shell-state="thinking"] .moltamp-terminal {
  filter: brightness(1.1);
}
```

That's a working skin. The full variable list, every targetable class, reactive data attributes, and a ready-to-paste AI prompt block all live in **[SKINNING.md](SKINNING.md)**.

<br/>

## Generate one with AI

Point Claude, ChatGPT, Codex, or any LLM at **[SKINNING.md](SKINNING.md)** — it includes a complete prompt block in the *"For AI-generated skins"* section. Moltamp's preflight system auto-fixes common rule violations at load time, so AI-generated skins work safely even when the model misses a detail.

<br/>

## Contribute a skin

1. **Fork** this repo
2. Create `skins/your-skin-id/` with `skin.json` + `theme.css`
3. Add a `preview.png` screenshot (1200x750 looks great)
4. Run through the checklist in **[CONTRIBUTING.md](CONTRIBUTING.md)**
5. **[Open a PR](../../pulls)** — we review weekly

Merged skins ship in the next Moltamp release and appear in the in-app skin browser plus **[moltamp.com/community](https://moltamp.com/community)**.

<br/>

## License

[MIT](LICENSE) — use, fork, remix, ship. Attribution appreciated but not required.

<br/>

<div align="center">

<a href="https://moltamp.com">
  <img src=".github/assets/logo.png" alt="Moltamp" width="32"/>
</a>

<br/>

<sub>Made for the community by <a href="https://moltamp.com">Moltamp</a></sub>

</div>

<!-- repo topics: moltamp, claude-code, claude-code-themes, claude-code-skins, ai-terminal, terminal-themes, terminal-customization, electron, vibe-coding, css-themes, codex-cli, gemini-cli, aider, developer-tools, theming -->
