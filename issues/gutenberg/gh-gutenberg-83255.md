# #83255: Term Name: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`6ca54e1`](https://github.com/WordPress/gutenberg/commit/6ca54e1c7bcd32eceb6081927f1c550097a258b2)
- **Discussion:** [#83255](https://github.com/WordPress/gutenberg/pull/83255) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Term Name block (`core/term-name`) gains `background.backgroundImage`, `background.backgroundSize`, and `background.gradient` supports in its `block.json`, bringing it in line with other core blocks as part of the design-tools consistency work (related to #43241). Because the block is server-rendered through `get_block_wrapper_attributes()`, the new supports serialize onto the wrapper element automatically with no PHP or JavaScript change. The legacy `color.gradients` support is retained alongside the new `background.gradient`, and the gradient layers over the background image.

## Impact

- **Block-theme developers:** The Term Name block now responds to background image, size, and gradient controls in Global Styles (Blocks → Term Name → Background) and in the block inspector. No code changes are required; the supports are picked up automatically.
- **Plugin developers:** No API change. If you register a custom block that mirrors Term Name's supports, you may want to add the same `background` keys for consistency, but nothing is forced.
- **Site owners / existing content:** No action required. Existing Term Name blocks render identically until a background is explicitly applied.
- **No breaking changes.** The legacy `color.gradients` support remains in `block.json` alongside `background.gradient`.

## Technical details

The diff is confined to three files. The functional change is in `packages/block-library/src/term-name/block.json`, which adds a `background` object to the existing `supports` key:

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

The `__experimentalDefaultControls` sub-object enables the background-image and gradient controls in the block inspector by default (the size control is not default-enabled).

No PHP or JS files are touched. The block's render path calls `get_block_wrapper_attributes()`, which reads the `supports` declaration and emits the corresponding inline styles (e.g. `background-image`, `background-size`, `background` for the gradient) on the wrapper `<span>`. The gradient is layered over the image per the behavior established in #75859.

The remaining two files are regenerated documentation: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/term-name/README.md` both list the new `background` supports alongside the existing `color`, `shadow`, `spacing`, and `typography` entries.

## Contribution

Opened by @aaronrobertshaw with @andrewserong as co-author. The PR record carries only two comments (both from the GitHub Actions bot reporting performance metrics and a flaky test) and no design discussion. The PR body notes the implementation was produced by a Claude Code agent from a predefined task, consistent with the batch of design-tools consistency PRs tracked under #43241.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
