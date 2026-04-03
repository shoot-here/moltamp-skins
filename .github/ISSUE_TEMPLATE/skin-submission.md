---
name: Skin Submission
about: Submit a new community skin
title: "[SKIN] "
labels: skin-submission
---

## Skin Info

- **Name:**
- **ID:** (alphanumeric + hyphens)
- **Author:**
- **Description:**

## Preview

<!-- Attach a screenshot (preview.png) showing the skin in action -->

## Checklist

- [ ] `skin.json` has all required fields (id, name, version, author, description, engine)
- [ ] All colors are CSS variables in `:root`
- [ ] Custom variables use `--skin-*` prefix
- [ ] No hardcoded colors outside `:root`
- [ ] Every `::before`/`::after` has `pointer-events: none`
- [ ] No `background` on `.moltamp-vibes`
- [ ] No external URLs or `@import`
- [ ] Assets under 5MB each, 20MB total
- [ ] Right-click context menus work on all panels
- [ ] Tested at different panel sizes

## Notes

<!-- Anything else — inspiration, special features, custom effects, etc. -->
