# #83384: Post Template: Add heading and button colour support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Template`
- **Merged:** [`f1de34d`](https://github.com/WordPress/gutenberg/commit/f1de34dc43f4f53f5e71dbd668109cacfc384c77)
- **Discussion:** [#83384](https://github.com/WordPress/gutenberg/pull/83384) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Template block (`core/post-template`) now declares `color.heading` and `color.button` in its `block.json` supports, enabling heading and button colour controls in the block inspector and Global Styles. Previously the block supported text, background, gradient, and link colours but not these two element colours, creating an inconsistency with other blocks. The underlying CSS serialization already worked for these elements; this change exposes the UI controls that were missing.

## Impact

- **Block theme developers / site editors:** Heading and Button colours can now be set on the Post Template block via Global Styles (Blocks → Post Template → Elements) and the block inspector (Styles → Elements), including separate mobile values. No code changes required.
- **Plugin & theme developers:** No API change. The block's render callback already calls `get_block_wrapper_attributes()`, so any existing customisation of element colours on this block continues to work. No migration or configuration needed.
- **No action required** for existing sites; the change is purely additive (new controls appear where they were previously absent).

## Technical details

The diff modifies three files:

1. **`packages/block-library/src/post-template/block.json`** — two keys added to the existing `color` supports object:

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

2. **`packages/block-library/src/post-template/README.md`** — regenerated to list the new `heading` and `button` sub-supports under `color`.

3. **`docs/reference-guides/core-blocks/README.md`** — the auto-generated core-blocks reference updated; the Post Template supports line now reads `color (background, button, gradients, heading, link, text)`.

No PHP changes. The PR notes that `lib/block-supports/elements.php` only skips blocks that explicitly opt out of serialization, so the CSS for heading and button colours was already being emitted on the front end. The `block.json` declaration is what registers the controls in the inspector and Global Styles UI.

## Contribution

Opened by @aaronrobertshaw as part of the design-tools consistency work tracked in #43241. The PR was implemented, built, and screenshotted by a Claude Code agent from a predefined task, per the author's disclosure. @ramonjd is credited as co-author. The discussion is minimal (2 comments, 0 reactions) with no notable design debate or rejected alternatives in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
