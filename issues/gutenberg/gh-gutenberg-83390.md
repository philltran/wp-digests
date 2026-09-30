# #83390: Tabs: Add link, heading, and button colour support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Tabs`
- **Merged:** [`ecd83bb`](https://github.com/WordPress/gutenberg/commit/ecd83bb1746382b7c99972137d16265a1d3d7f1a)
- **Discussion:** [#83390](https://github.com/WordPress/gutenberg/pull/83390) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tabs block (`core/tabs`) now supports `color.link`, `color.heading`, and `color.button` element colours, bringing it in line with Group and Columns. Previously only `color.text` and `color.background` were available. The change is purely additive to the block's `block.json` supports; no PHP render-callback modification was needed because the existing `WP_HTML_Tag_Processor` logic already preserves saved `class` and `style` attributes.

## Impact

- **Site builders / theme authors (block themes):** Link, heading, and button colours for content inside Tabs panels are now settable in the block inspector (Styles → Elements) and in Global Styles (Blocks → Tabs → Elements), including per-breakpoint (mobile) values. No code changes required.
- **Plugin & theme developers:** No new hooks, filters, or REST schema changes. If you build custom Tabs-related tooling that reads `block.json` supports, the `color` object now includes three additional boolean keys.
- **No action required** for existing sites. The element-colour serialization in `lib/block-supports/elements.php` already renders these colours on the front end for any block that does not explicitly opt out; this PR only exposes the editor controls.
- **Caveat:** The button colour does **not** style the tab buttons themselves, which are bare `<button>` elements rendered by the `core/tab-list` block. It applies to `<button>` elements *inside* the tab panels.

## Technical details

The diff modifies three files:

1. **`packages/block-library/src/tabs/block.json`** — three keys added to the existing `supports.color` object:

```json
"color": {
  "text": true,
  "background": true,
  "heading": true,
  "button": true,
  "link": true,
  "__experimentalDefaultControls": {
    "text": true,
    "background": true
  }
}
```

2. **`packages/block-library/src/tabs/README.md`** — regenerated documentation listing the new `heading`, `button`, and `link` sub-keys under `color`.

3. **`docs/reference-guides/core-blocks/README.md`** — the auto-generated core-blocks reference updated to show `color (background, button, heading, link, text)` in the Tabs supports line.

No changes to `render.php`, `index.js`, or any PHP file. The PR description notes that `lib/block-supports/elements.php` already serialises element colours for any block that has not opted out, so the front-end CSS output (`.wp-block-tabs .has-link-color a`, etc.) was already functional; the `block.json` change is what registers the controls in the inspector and Global Styles UI.

## Contribution

Opened by @aaronrobertshaw and implemented via a Claude Code agent from a predefined task. @shail-mehta resolved merge conflicts and @im3dabasia reviewed. @aaronrobertshaw flagged in the final comment that the Global Styles settings-panel UI does not yet display inherited element colours for Tabs (tracked in issue #80438) and deferred a fix until the remaining element-style block supports land in the same release wave. The PR was merged at `ecd83bb` with no further iteration.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
