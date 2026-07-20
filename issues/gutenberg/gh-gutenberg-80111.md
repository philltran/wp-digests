# #80111: Move inspector controls styles slot back to previous position

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jeryj
- **Labels:** `[Type] Bug`, `[Package] Block editor`
- **Merged:** [`c9a5646`](https://github.com/WordPress/gutenberg/commit/c9a5646c87a560b0ce9137c3bc44337fd140e507)
- **Discussion:** [#80111](https://github.com/WordPress/gutenberg/pull/80111) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request fixes a block editor UI regression by restoring the original render position of the `styles` inspector slot. A previous change accidentally moved the `<InspectorControls.Slot group="styles" />` component, altering the visual order of the block inspector panels. This PR reverts that placement to ensure controls injected via the `styles` slot group render in their intended location.

## Impact

- **Plugin & theme developers:** If you inject controls into the block inspector using `<InspectorControls.Slot group="styles" />`, your controls will now render in the correct position again.
- **Block editor users:** The visual order of the Styles pane and related inspector panels is restored to the expected layout.
- **No code changes required.** Upgrade to the next WordPress/Gutenberg release to apply the fix.

## Technical details

The diff modifies two components in the block editor package:
- `packages/block-editor/src/components/block-inspector/index.js`
- `packages/block-editor/src/components/inspector-controls-tabs/styles-tab.js`

In both files, the `<InspectorControls.Slot group="styles" />` element is moved from immediately after the Background panel support to the end of the inspector content list (after `PositionControls` and before the Bindings slot in the main inspector). This restores the DOM order and visual stacking of inspector slots to match the pre-regression state.

## Contribution

Opened and merged by @jeryj, this PR addresses a regression introduced in #80060. The author noted the layout shift was an unintended side effect of an AI-assisted change and coordinated with @t-hamano to align the slot placement across both the main inspector and the styles tab before merging as `c9a5646`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
