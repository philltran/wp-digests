# #83198: Archives: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Archives`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`b20fc51`](https://github.com/WordPress/gutenberg/commit/b20fc516c46bfb4789ad1ede7b4d9fa021da3aa0)
- **Discussion:** [#83198](https://github.com/WordPress/gutenberg/pull/83198) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Archives block (`core/archives`) now supports background image, background size, and gradient via the `background` supports key in its `block.json`. This brings the block in line with other core blocks (Quote, Pullquote, Verse) as part of the ongoing design-tools consistency effort tracked in #43241. Because Archives is server-rendered through `get_block_wrapper_attributes()` in both list and dropdown modes, the new supports serialize onto the wrapper automatically with no PHP template change.

## Impact

- **Site owners / editors:** The Background panel in Global Styles (Blocks → Archives) and the block inspector now expose image, size, and gradient controls for the Archives block. Gradient layers over the image (per #75859), so both render together.
- **Theme & plugin developers:** No code changes required. The supports are declarative in `block.json` and flow through `get_block_wrapper_attributes()`. If you override the Archives block template, the new CSS custom properties (`--wp--style--background-image`, `--wp--style--background-size`, `--wp--style--gradient`) will be present on the wrapper.
- **No breaking changes.** The legacy `color.gradients` support remains alongside the new `background.gradient`, matching the pattern already used by Quote, Pullquote, and Verse.
- **No action required** for existing sites; the change is purely additive.

## Technical details

The diff modifies three files:

1. **`packages/block-library/src/archives/block.json`** — adds a `background` supports object:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true,
  "__experimentalDefaultControls": {
    "backgroundImage": true,
    "gradient": true
  }
}
```

The `__experimentalDefaultControls` key means the image and gradient controls are visible by default in the block inspector without the user expanding the Background panel.

2. **`packages/block-library/src/archives/README.md`** — regenerated to document the new `background` supports sub-keys.

3. **`docs/reference-guides/core-blocks/README.md`** — the core-blocks reference table is updated to list `background (backgroundImage, backgroundSize, gradient)` in the Archives supports column.

No PHP, JS, or CSS source files are touched. The block's existing server-rendering path (`get_block_wrapper_attributes()`) picks up the new supports and emits the corresponding inline styles / CSS custom properties on the wrapper element. An empty archive (no posts in any month) prints the string "No archives to show." rather than rendering an empty wrapper with background styles.

## Contribution

Opened and authored by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives. It is part of the broader #43241 design-tools consistency track.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
