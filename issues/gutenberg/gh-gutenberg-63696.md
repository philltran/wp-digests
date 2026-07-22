# #63696: Improve the various Alignment controls props handling

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @dhananjaykuber
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Package] Block library`, `[Package] Block editor`
- **Merged:** [`33cb585`](https://github.com/WordPress/gutenberg/commit/33cb58564a025e054a34c15929397b195ca40426)
- **Discussion:** [#63696](https://github.com/WordPress/gutenberg/pull/63696) · 16 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

This pull request standardizes the `label` and `description` props across the block editor’s alignment controls (`AlignmentUI`, `BlockAlignmentUI`, and `BlockVerticalAlignmentUI`) to improve accessibility and reduce user confusion. Previously, these components shipped with hardcoded, sometimes vague strings like "Align" or "Change vertical alignment". The update replaces these defaults with clearer, context-specific labels and makes the `description` prop optional, ensuring screen readers receive more precise instructions without forcing unnecessary verbosity.

## Impact

- **Block & plugin developers:** No breaking changes to public component APIs, but the default `label` strings for `BlockAlignmentUI` and `BlockVerticalAlignmentUI` have changed. If your e2e tests or custom UI overrides rely on exact button names (e.g., `getByRole('button', { name: 'Align' })`), you will need to update selectors or pass explicit `label` props.
- **Site owners & editors:** Improved clarity in the block toolbar alignment controls, particularly for screen reader users. No configuration changes required.
- **Platform & hosting teams:** No action required.

## Technical details

The diff modifies three core alignment components in `packages/block-editor/src/components/`:
- `alignment-control/ui.js`: Removes the hardcoded default `description = __( 'Change text alignment' )` and conditionally passes it to `toggleProps` only when provided.
- `block-alignment-control/ui.js`: Introduces `label` and `description` props. Changes the default `label` from `__( 'Align' )` to `__( 'Align block' )` and passes it directly to `ToolbarDropdownMenu`. The `description` is now only spread into `toggleProps` if truthy.
- `block-vertical-alignment-control/ui.js`: Adds a `label` prop, changing the default from `_x( 'Change vertical alignment', 'Block vertical alignment setting label' )` to `__( 'Align content vertically' )`.
- `packages/block-library/src/media-text/edit.js` and `packages/block-library/src/table/edit.js` are updated to pass explicit, context-aware `label` props to these components.

Before/after pattern for `BlockAlignmentUI` prop handling:
```jsx
// Before
function BlockAlignmentUI( { controls, isToolbar, isCollapsed = true } ) {
  // ...
  label: __( 'Align' ),
  // ...
  toggleProps: { description: __( 'Change alignment' ) },
}

// After
function BlockAlignmentUI( { controls, isToolbar, isCollapsed = true, label = __( 'Align block' ), description } ) {
  // ...
  label,
  // ...
  toggleProps: description ? { description } : {},
}
```
The change also updates corresponding Jest snapshots and Playwright e2e tests to reflect the new accessible names.

## Contribution

Opened by @dhananjaykuber to address inconsistent alignment control labels that confused screen reader users. The PR was initially slated for 6.7 but was punted to 6.8/6.9 to allow time for review and rebase. After @t-hamano requested a rebase on trunk, @dhananjaykuber updated the branch, refined the labels to remove redundant descriptions, and incorporated feedback from @colorful-tones, @kevin940726, and others. The final merge standardizes the props without introducing new public APIs, focusing purely on accessibility improvements and consistent prop handling.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
