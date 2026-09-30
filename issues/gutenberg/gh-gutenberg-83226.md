# #83226: Heading: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Heading`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`579ee5d`](https://github.com/WordPress/gutenberg/commit/579ee5de977434186b7369dea8ab514068318f28)
- **Discussion:** [#83226](https://github.com/WordPress/gutenberg/pull/83226) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Heading block now supports the Background panel's image, size, and gradient controls in both the block inspector and Global Styles. Three new entries are added to the block's `supports` in `block.json`: `background.backgroundImage`, `background.backgroundSize`, and `background.gradient`. This brings the Heading block in line with other core blocks as part of the ongoing design-tools consistency effort (related to #43241). No PHP render-callback change is needed because Heading is a static block whose supports are serialized by `useBlockProps.save()`, and the existing `has-background` class already triggers the block's background padding.

## Impact

- **Theme & site builders:** The Heading block now exposes background image, size, and gradient controls in the block inspector and in Global Styles (Blocks → Heading → Background). The gradient layers over the image, so both render together.
- **Plugin & theme developers:** No code changes required. The legacy `color.gradients` support remains alongside the new `background.gradient`; both are active. If you previously worked around the missing background supports with custom CSS or a filter on `block_supports`, those workarounds can now be removed.
- **No action required** for existing sites. The change is additive; headings without a background render identically to before.

## Technical details

The diff modifies `packages/block-library/src/heading/block.json` to add a `background` supports object:

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

The `__experimentalDefaultControls` key means the image and gradient controls are visible by default in the block inspector without the user expanding the Background panel. `backgroundSize` is available but not in the default controls set.

Because Heading is a static block, these supports are serialized directly onto the `<h1>`–`<h6>` element by `useBlockProps.save()`. The PHP render callback (`render.php`) only adds a class via `WP_HTML_Tag_Processor` and does not need modification. A block-level background adds the `has-background` class, which activates the heading's existing background padding (the same mechanism already used for background color).

The legacy `color.gradients` support is retained alongside `background.gradient`; the new gradient layers over the background image per the behavior established in #75859.

Two documentation files are regenerated to list the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/heading/README.md`.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The author thanked the reviewer for a "speedy review"; the three comments in the thread contain no design debate or rejected alternatives. Merged at `579ee5d`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
