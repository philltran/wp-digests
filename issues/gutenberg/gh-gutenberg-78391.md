# #78391: Grid: hide resize handles and actions while on tile is resizing

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @simison
- **Labels:** `[Type] Experimental`, `[Feature] Dashboard`
- **Merged:** [`9a76e60`](https://github.com/WordPress/gutenberg/commit/9a76e606d99b940c51bb74568d097c5202d9cd75)
- **Discussion:** [#78391](https://github.com/WordPress/gutenberg/pull/78391) · 6 comments · 0 reactions

## Summary

This PR updates the dashboard grid layout to suppress resize handles and action chrome on non-active tiles while a tile is actively being resized. By hiding adjacent interactive elements during drag operations, the change reduces visual clutter and prevents overlapping UI layers from interfering with the active resize gesture. It ensures a cleaner, more predictable editing experience when adjusting widget or grid tile dimensions.

## Impact

- **Plugin & theme developers / Block editor users:** No direct code changes required; this is an internal UI refinement to the experimental grid layout used by dashboard widgets and blocks.
- **Platform/Editor consumers:** Improved interaction stability during drag-resize operations in the block editor and dashboard widget layouts.
- **Migration/Config:** None. This is a purely visual/state-management tweak with no breaking changes, API modifications, or deprecated symbols. No action required.

## Technical details

The grid layout component was updated to conditionally hide resize handles and non-content chrome (specifically referencing `actionableArea`) on tiles that are not currently being resized. Rather than managing visibility through separate boolean props for each UI element, the implementation leverages a shared state mechanism via CSS class toggles—targeting selectors like `is-other-widget-dragging` or `is-other-widget-resizing` to control sibling tile visibility. This keeps the active tile's handles visible while suppressing others to prevent overlapping layer conflicts during drag operations.

## Contribution

Opened by @simison as a follow-up to #78389, this PR was merged following internal review. @retrofox collaborated on the implementation strategy, specifically debating whether to manage visibility via separate props or leverage existing CSS state selectors. The `is-other-widget-dragging` / `is-other-widget-resizing` class approach was aligned upon to keep logic DRY and consistent with existing widget drag-and-drop patterns. Co-authored by @simison and @retrofox, then merged into the Gutenberg trunk.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
