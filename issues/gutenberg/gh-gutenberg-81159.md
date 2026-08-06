# #81159: List View: Split the context and drop dead props to cut re-renders on expand and collapse

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Block library`, `[Feature] List View`, `[Package] Block editor`
- **Merged:** [`f8c8421`](https://github.com/WordPress/gutenberg/commit/f8c8421e38c335e26914aeedf1462892436ab71b)
- **Discussion:** [#81159](https://github.com/WordPress/gutenberg/pull/81159) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Splits the monolithic `ListViewContext` into three isolated contexts (`ListViewContext`, `ListViewTreeStateContext`, `ListViewInsertedBlockContext`) and removes unused props to prevent unnecessary re-renders in the block editor's List View. By separating static configuration from frequently updating expansion and drag state, `React.memo()` can now effectively guard individual list rows when branches are expanded or collapsed.

## Impact

- **Block editor developers:** No public API changes. Internal component contracts shifted: `BlockSettingsMenu` now receives a `getExpansionState()` callback instead of an `expansionState` object. Dead props (`listPosition`, `isSelected`, `position`, `siblingBlockCount`, `level`) were removed from `ListViewBlock` and `ListViewBlockSelectButton`.
- **Editors:** Noticeably smoother List View interactions when expanding/collapsing nested branches or dragging blocks, due to reduced re-render cascades.
- **No action required** for theme/plugin developers or site owners.

## Technical details

The diff refactors `packages/block-editor/src/components/list-view/context.js` to export `ListViewTreeStateContext` and `ListViewInsertedBlockContext`. `packages/block-editor/src/components/list-view/index.js` nests these providers and updates the `expansion` reducer to support a `replace` action type, using `isShallowEqual` to bail out of state updates when unchanged. `ListViewBranch` now consumes `useListViewTreeState()` instead of the full context, isolating expansion/drag state updates. `ListViewBlock` drops the `expansionState` prop and passes `updateExpansion` directly to `BlockSettingsMenu`. The `updateExpansion` calls throughout the codebase now consistently pass `clientIds` as arrays:

```js
// Before
updateExpansion( { type: 'expand', clientIds: clientId } );

// After
updateExpansion( { type: 'expand', clientIds: [ clientId ] } );
```

`block-contents.js` extracts an `InsertedBlockContent` component to prevent inserted-block state from triggering sibling row re-renders, and `use-list-view-drop-zone.js` removes the now-unnecessary `expansionState` parameter from its throttled expansion logic.

## Contribution

Opened and merged by @Mamaduka, with co-authors @talldanwp and @jsnajdr. The refactor was driven by local render observations rather than automated benchmarks, and review feedback prompted alignment of the expansion reducer with changes in #81138. The team prioritized context isolation and dead prop removal over broader memoization fixes, deferring `ListViewBranch` optimization to a follow-up.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
