# #83214: Post Comments Form: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`113ce17`](https://github.com/WordPress/gutenberg/commit/113ce174aff6454a1f6e3d98a75a3a012f172986)
- **Discussion:** [#83214](https://github.com/WordPress/gutenberg/pull/83214) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Comments Form block (`core/post-comments-form`) now supports background image, background size, and gradient via the `background` block supports key. This brings the block in line with other core blocks as part of the ongoing design-tools consistency effort (related to #43241). Because the block is server-rendered through `get_block_wrapper_attributes()`, the new supports serialize onto the wrapper automatically with no PHP change required.

## Impact

- **Theme developers / site builders:** The Comments Form block now exposes Background panel controls (image, size, gradient) in Global Styles and the block inspector. No code changes needed; the controls appear automatically once the block is present in a template or post.
- **Plugin developers:** No API change. If you programmatically inspect `block.json` supports for `core/post-comments-form`, the `background` key is now present.
- **No action required** for existing sites. The change is additive; blocks without background settings render identically to before.

## Technical details

The diff adds a `background` object to the `supports` key in `packages/block-library/src/post-comments-form/block.json`:

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

The pre-existing `color.gradients: true` support is retained alongside the new `background.gradient`, so both the legacy gradient path and the new background-panel gradient path remain active. The `__experimentalDefaultControls` sub-key enables the image and gradient controls by default in the block inspector (background size is not shown by default).

No PHP or JS render changes are needed: `core/post-comments-form` is server-rendered, and `get_block_wrapper_attributes()` serializes all registered supports onto the wrapper element. The gradient layers over the background image (per the behavior established in #75859), so both are visible simultaneously.

Documentation regenerated in `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/post-comments-form/README.md` to list the new supports.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @andrewserong. The PR notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no notable design debate; it follows the established pattern of adding `background` supports to core blocks one at a time.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
