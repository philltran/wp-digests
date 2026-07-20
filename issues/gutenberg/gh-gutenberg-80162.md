# #80162: Responsive editing: Apply crop dimensions to image block placeholder

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Image`
- **Merged:** [`737ebab`](https://github.com/WordPress/gutenberg/commit/737ebab1f81a0d5cc33aca0d192bd96f49589dd6)
- **Discussion:** [#80162](https://github.com/WordPress/gutenberg/pull/80162) · 1 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

When responsive editing is enabled and a tablet or mobile viewport is selected, the image block’s placeholder previously ignored the configured aspect ratio. This PR fixes the bug by extending the `dimensions` selector in the image block’s `block.json` to target the placeholder element, ensuring the placeholder respects the selected crop dimensions during responsive editing. This keeps pattern and layout spacing consistent across viewports before an image is uploaded.

## Impact

- **Block & theme developers:** No code changes required. The fix is internal to the core image block.
- **Site owners / editors:** No action required. The placeholder now correctly reflects aspect ratios in responsive editing mode.
- **Hosting & platform teams:** No action required.
- **Breaking changes / deprecations:** None. No public APIs, hooks, or block attributes were altered.

## Technical details

The change modifies `packages/block-library/src/image/block.json` (and its accompanying `README.md`). It updates the `selectors.dimensions` property to include the placeholder component alongside the image element:

```json
// Before
"dimensions": ".wp-block-image img",

// After
"dimensions": ".wp-block-image img, .wp-block-image .components-placeholder",
```

This selector is consumed by the block editor’s style engine to apply CSS custom properties for aspect ratio and crop dimensions. By appending `.components-placeholder`, the editor now applies the same dimension styles to the placeholder UI as it does to the actual `<img>` element, resolving the responsive editing viewport mismatch.

## Contribution

Opened and merged by @talldan with co-authorship from @andrewserong. The PR was a straightforward bug fix targeting the image block’s `block.json` selectors. Review was minimal, with the author noting that cover and feature image blocks already handled this correctly, making the fix a narrow, targeted adjustment that was merged without further debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
