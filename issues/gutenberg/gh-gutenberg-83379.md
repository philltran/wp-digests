# #83379: Post Content: Add button colour support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Content`
- **Merged:** [`109c01e`](https://github.com/WordPress/gutenberg/commit/109c01ec28738b7074b889e9433caa0c764d933a)
- **Discussion:** [#83379](https://github.com/WordPress/gutenberg/pull/83379) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Content block (`core/post-content`) now declares `color.button` in its `block.json` supports, enabling button text and background colour controls in the block inspector and Global Styles. The block already supported `color.heading` and `color.link`; buttons rendered inside post content could not be styled from the design tools. No PHP or render-callback change was needed because the block already uses `get_block_wrapper_attributes()`, and `lib/block-supports/elements.php` serialises element colours for any block that has not explicitly opted out.

## Impact

- **Block theme developers / site editors:** Buttons appearing inside the Post Content block (e.g. in a Single Post template) can now be coloured via Global Styles → Blocks → Content → Elements → Button, and via the block inspector's Styles → Elements → Button panel. Mobile-state colours are supported.
- **Plugin & theme developers:** No code change required. If you were previously injecting button styles into post content via custom CSS or a filter, you can now rely on the native element-colour pipeline instead.
- **No breaking changes, deprecations, or migration steps.** The change is additive to the block's `supports` object.

## Technical details

The diff is confined to three files:

1. **`packages/block-library/src/post-content/block.json`** — adds `"button": true` inside the existing `"color"` supports object, alongside the already-present `"gradients"`, `"heading"`, and `"link"` keys.

2. **`packages/block-library/src/post-content/README.md`** — regenerated to list the new `button` sub-key under `color`.

3. **`docs/reference-guides/core-blocks/README.md`** — regenerated; the Post Content entry's supports line now reads `color (background, button, gradients, heading, link, text)` instead of the previous list without `button`.

No changes to `render.php`, `index.js`, or any PHP file. The block's render callback already calls `get_block_wrapper_attributes()`, which picks up element-colour CSS custom properties emitted by `lib/block-supports/elements.php`. That file only skips serialization for blocks that explicitly opt out, so the front-end CSS was already being generated; the missing piece was the `block.json` declaration that surfaces the controls in the editor UI.

## Contribution

Opened by @aaronrobertshaw as part of the design-tools consistency work tracked in #43241. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." Co-authored with @shail-mehta and @ramonjd. The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives; it merged as `109c01e`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
