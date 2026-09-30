# #83245: Read More: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Read More`
- **Merged:** [`c415e6c`](https://github.com/WordPress/gutenberg/commit/c415e6cf1f6f4a9327c08710d8c1dfb663db541d)
- **Discussion:** [#83245](https://github.com/WordPress/gutenberg/pull/83245) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Read More block (`core/read-more`) now supports the Background panel's image, size, and gradient controls in Global Styles and the block inspector. Three new keys — `background.backgroundImage`, `background.backgroundSize`, and `background.gradient` — are added to the block's `block.json` supports, bringing it in line with other core blocks as part of the ongoing design-tools consistency effort (related to #43241). Because the block is server-rendered via `get_block_wrapper_attributes()`, the new supports serialize onto the link element automatically with no PHP change.

## Impact

- **Theme & site-editor developers:** The Read More block now accepts background image, size, and gradient values in Global Styles (Blocks → Read More → Background) and per-block overrides. No code changes required; the existing `get_block_wrapper_attributes()` pipeline handles serialization.
- **Plugin developers:** No new hooks, filters, or REST schema changes. The legacy `color.gradients` support remains alongside the new `background.gradient` key, so existing gradient styling via the color panel is unaffected.
- **Site owners / content editors:** Can now set a background image (with Contain/Cover/Other sizing) and a gradient on the Read More block without custom CSS.
- **No action required** for existing sites; the change is purely additive.

## Technical details

The diff modifies three files:

1. **`packages/block-library/src/read-more/block.json`** — adds a `background` object under `supports`:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true
}
```

This sits alongside the existing `color` supports (`gradients`, `text`) and the existing `anchor`, `html`, `shadow`, `spacing`, and `typography` entries.

2. **`packages/block-library/src/read-more/README.md`** — regenerated to document the three new sub-keys under the `background` support.

3. **`docs/reference-guides/core-blocks/README.md`** — the core-blocks reference table for `core/read-more` is updated to list `background (backgroundImage, backgroundSize, gradient)` in the supports column.

No PHP, JS, or CSS source files are touched. The block renders server-side through `get_block_wrapper_attributes()`, which reads the registered supports and emits the corresponding inline styles (or CSS custom properties) on the `<a>` element. The gradient layers over the background image (per the behavior established in #75859), so both are visible simultaneously. The legacy `color.gradients` support is retained, not replaced.

## Contribution

Opened by @aaronrobertshaw and co-authored with @andrewserong, who flagged that a rebase on `core-blocks/README.md` was needed before merge. The author noted they were rebasing and repushing as conflicts appeared. The PR body states the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. No design debate or alternative approaches are recorded in the four comments; the discussion was limited to the rebase coordination.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
