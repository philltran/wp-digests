# #81136: List View: Reduce per-row store subscriptions

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Feature] List View`, `[Package] Block editor`
- **Merged:** [`52c7652`](https://github.com/WordPress/gutenberg/commit/52c7652c34250e677ad0af0644ef986dd30eaad6)
- **Discussion:** [#81136](https://github.com/WordPress/gutenberg/pull/81136) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The List View component in the Block Editor now reduces per-row store subscriptions by inlining data selection directly into `ListViewBlock`, `ListViewBlockSelectButton`, and `ListViewBranch`. Previously, these components relied on `useBlockDisplayInformation` and `useBlockLock` hooks, which triggered separate `useSelect` calls and computed more data than needed. The change consolidates these reads into each component's existing `useSelect`, cutting subscriptions from seven to four per row and two to one per branch, improving editor responsiveness when many blocks are present.

## Impact

- **Block editor developers:** No breaking changes or migration required. The refactoring is internal to `@wordpress/block-editor`.
- **Site owners & editors:** Improved typing and scrolling performance in the List View, particularly on posts with many blocks or synced patterns/template parts.
- **Hosting & platform teams:** No configuration changes needed.
- **Action required:** None.

## Technical details

- Replaced `useBlockDisplayInformation( clientId )` and `useBlockLock( clientId )` with targeted `useSelect` calls in `packages/block-editor/src/components/list-view/block.js`, `block-select-button.js`, and `branch.js`.
- Each component now reads only the specific fields it needs (`icon`, `anchor`, `isSticky`, `isLocked`, `positionLabel`, `isSynced`) from the `blockEditorStore` and `blocksStore`.
- Added a new private selector `isSyncedBlock` in `packages/block-editor/src/store/private-selectors.js` to determine if a block should render with synced styling, replacing the previous pattern-matching logic in the hook.
- Exported `getPositionTypeLabel` from `packages/block-editor/src/components/use-block-display-information/index.js` so `ListViewBlock` can compute position labels directly from block attributes.
- Replaced the `useBlockRename` hook with a direct `hasBlockSupport( blockName, 'renaming', true )` check in `ListViewBlock`, removing unnecessary indirection.
- Store subscription counts per rendered row dropped from 7 to 4, and per branch from 2 to 1.

**Before:**
```js
const blockInformation = useBlockDisplayInformation( clientId );
const { isLocked } = useBlockLock( clientId );
const { canRename } = useBlockRename( blockName );
```

**After:**
```js
const { isLocked, isSynced, positionLabel } = useSelect(
  ( select ) => {
    const { isLockedBlock, isSyncedBlock } = unlock( select( blockEditorStore ) );
    return { isLocked: isLockedBlock( clientId ), isSynced: isSyncedBlock( clientId ), positionLabel: getPositionTypeLabel( getBlockAttributes( clientId ) ) };
  },
  [ clientId ]
);
const canRename = !! blockName && hasBlockSupport( blockName, 'renaming', true );
```

## Contribution

The change addresses a known performance bottleneck where the `block-editor` store subscriptions multiplied with each List View row, impacting typing metrics. The author noted a similar refactoring was applied to the canvas component a few releases prior to improve typing performance. The approach focused on eliminating redundant hook calls and inlining lightweight checks, with straightforward review and no significant design debate or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
