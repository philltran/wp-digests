# #83254: Term Description: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Term Description`
- **Merged:** [`6cae414`](https://github.com/WordPress/gutenberg/commit/6cae414c7e5e0ededd7f3e8aec9737d294cd17d0)
- **Discussion:** [#83254](https://github.com/WordPress/gutenberg/pull/83254) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Term Description block (`core/term-description`) now accepts background image, background size, and gradient styles through the Background panel in Global Styles and the block inspector. Previously it supported color, border, shadow, spacing, and typography but lacked the Background panel's image/size/gradient controls. This closes a gap in the design-tools consistency work tracked in #43241, bringing the block in line with other core blocks that already expose these supports.

## Impact

- **Block theme developers:** The Term Description block now responds to `backgroundImage`, `backgroundSize`, and `gradient` styles set in Global Styles (Blocks → Term Description → Background) and to block-level overrides in the inspector. No code changes required; the styles serialize onto the wrapper automatically.
- **Plugin developers:** No API or hook changes. If you render `core/term-description` in a custom context, the new supports will appear in the block's `block.json` and in `get_block_wrapper_attributes()` output.
- **No action required** for existing sites. The block renders identically when no background styles are set; it returns an empty string when the term description is empty, so no empty box is painted.

## Technical details

The diff adds a `background` object to `packages/block-library/src/term-description/block.json`:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true,
  "__experimentalDefaultControls": {
    "backgroundImage": true
  }
}
```

No PHP changes are needed because `core/term-description` is server-rendered through `get_block_wrapper_attributes()`, which serializes all registered supports onto the wrapper element. The legacy `color.gradients` support remains alongside the new `background.gradient` for backward compatibility. Per #75859, the gradient renders as a layer over the background image, so both are visible simultaneously.

The core-blocks reference (`docs/reference-guides/core-blocks/README.md`) and the block's own `README.md` are regenerated to list the new supports. The `__experimentalDefaultControls` key enables the background image control by default in the block inspector.

## Contribution

Opened by @aaronrobertshaw with @andrewserong as co-author. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion is minimal (2 comments, 0 reactions) with no visible design debate; it follows the established pattern of adding Background-panel supports to a core block. It is part of the broader #43241 design-tools consistency effort.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
