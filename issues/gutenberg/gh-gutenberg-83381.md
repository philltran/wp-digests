# #83381: Details: Add heading and button colour support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Details`
- **Merged:** [`aa4d2fb`](https://github.com/WordPress/gutenberg/commit/aa4d2fb3528f7b0d26f47fd88e93120111bf29b6)
- **Discussion:** [#83381](https://github.com/WordPress/gutenberg/pull/83381) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Details block (`core/details`) now declares `color.heading` and `color.button` in its `block.json` supports, enabling element-level heading and button colour controls in the block inspector and Global Styles. Previously only link colour was available alongside background and text. This is part of the ongoing design-tools consistency effort to bring element colour supports to blocks that commonly contain headings and buttons.

## Impact

- **Site builders / theme developers:** Heading and button text and background colours inside a Details block can now be set per-block (inspector → Styles → Elements) and via Global Styles (Blocks → Details → Elements), including mobile responsive values. No code changes required.
- **Plugin & theme developers:** No API change. The `color.heading` and `color.button` supports keys already existed in the block-supports system; this PR simply opts the Details block in. If you build custom blocks that wrap headings or buttons, the same `block.json` pattern applies.
- **No action required** for existing sites. Element colours set on a Details block already rendered on the front end (the serialization in `lib/block-supports/elements.php` only skips blocks that explicitly opt out); this change adds the editor controls, not the rendering.

## Technical details

The diff is confined to three files:

1. **`packages/block-library/src/details/block.json`** — two keys added under the existing `color` supports object:

```json
"color": {
  "gradients": true,
  "link": true,
  "heading": true,
  "button": true,
  "__experimentalDefaultControls": {
    "background": true,
    "text": true
  }
}
```

2. **`packages/block-library/src/details/README.md`** — regenerated documentation listing the new `heading` and `button` sub-keys under `color`.

3. **`docs/reference-guides/core-blocks/README.md`** — the auto-generated core-blocks reference updated to show `color (background, button, gradients, heading, link, text)` in the Details block's supports line.

No PHP, JS, or CSS changes. The block is static (its `save.jsx` supplies the wrapper markup), so the existing element-colour serialization pipeline in `lib/block-supports/elements.php` handles front-end output without modification. The pattern mirrors the earlier Columns block work (#54104).

## Contribution

Opened and merged by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes it was implemented by a Claude Code agent from a predefined task. The approach directly mirrors the Columns block element-colour PR (#54104); no alternative approaches or design debate are visible in the two comments on the PR.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
