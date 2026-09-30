# #83284: Inserter: Stop re-rendering the block list on block selection change

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Blocks`, `[Package] Block editor`
- **Merged:** [`d4c50c8`](https://github.com/WordPress/gutenberg/commit/d4c50c850569b090f43d2932a89a30b1999bec10)
- **Discussion:** [#83284](https://github.com/WordPress/gutenberg/pull/83284) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The block inserter's list no longer re-renders when the block selection changes while the inserter is open. Previously, the insertion destination (which tracks the selection) was threaded through as a parameter to item-building and callback construction, giving every `InserterListItem` a new prop identity per click and defeating its `memo`. The fix decouples item construction from the root, reads the destination from the store at call time inside callbacks, and moves the closest-allowed-container resolution into `onInsertBlocks`. A side effect: blocks inserted from the media tab now land in the closest accepting container or show a "can't be inserted" notice, instead of being silently dropped by `insertBlocks`.

## Impact

- **Site owners / editors:** No visible change in normal use. The inserter list simply stops flashing when you click between blocks with it open. At 4× CPU throttle the per-click cost drops by roughly 100 ms.
- **Plugin & theme developers:** No API or hook changes. `getInserterItems` still returns the same shape; the `isAllowedInCurrentRoot` flag on each item is unchanged. The only behavioral shift is that media-tab insertions now resolve the closest allowed container (or show the `inserter-notice` snackbar) rather than passing `undefined` to `insertBlocks` and silently dropping the block.
- **No action required** for existing code. The change is internal to `@wordpress/block-editor`'s inserter components and store selectors.

## Technical details

Three self-contained commits, each targeting one source of identity churn:

**1. `useInsertionPoint` — destination read at call time** (`use-insertion-point.js`)

A new module-level `getDestination(selectors, { rootClientId, insertionIndex, clientId, isAppender })` helper replaces the inline `useSelect` that previously returned `{ destinationRootClientId, destinationIndex }` and re-ran on every selection change. `onInsertBlocks` and `onToggleInsertionPoint` now call `getDestination(unlock(registry.select(blockEditorStore)), …)` inside the callback body, so the `useCallback` dependency arrays no longer list `destinationRootClientId` / `destinationIndex`.

When `_rootClientId === undefined` (the media-tab path), `onInsertBlocks` now calls `getClosestAllowedInsertionPoint(names, destination.destinationRootClientId)` and, if the result is `null`, fires `createErrorNotice` with the `inserter-notice` snackbar id before returning. This logic was previously in `onSelectItem` in `use-block-types-state.js`.

**2. `getInserterItems` — build once, filter per root** (`selectors.js`)

Two new `createSelector`-memoized selectors build items independent of the root:

- `getBlockTypeInserterItems(state)` — maps every registered block type that passes `hasBlockType(blockType, 'inserter', true)` through `buildBlockTypeInserterItem`, expands variations, and orders core blocks first. Inputs: `getBlockTypes()`, `getBlockVariationsRaw()`, `state.blocks.order`, `state.preferences.insertUsage`.
- `getReusableBlockInserterItems(state, reusableBlocks)` — maps reusable blocks through a new `buildReusableBlockInserterItem` helper. Inputs: `reusableBlocks`, `state.preferences.insertUsage`.

`getInserterItems` now calls these two selectors, then filters per root. A module-level `const itemsWithRootFlag = new WeakMap()` caches the `isAllowedInCurrentRoot`-flagged copy of each item per (item, flag) pair so the filter pass doesn't allocate new objects. Because the underlying item objects are stable across roots, `useSelect` in `use-block-types-state.js` can return the previous array reference when the filtered result is unchanged.

**3. `onSelectItem` slimmed down** (`use-block-types-state.js`)

The `getClosestAllowedInsertionPoint` call and `createErrorNotice` logic are removed from `onSelectItem`; it now simply calls `onInsert(insertedBlock, undefined, shouldFocusBlock)` without a fourth `destinationClientId` argument. The `useCallback` deps shrink to `[ onInsert ]`. The `options` object is replaced by module-level `FILTERED_OPTIONS` / `UNFILTERED_OPTIONS` constants (same pattern applied in `use-patterns-state.js`) so the `useSelect` deps are stable.

In `block-types-tab.jsx`, the partition of `items` into `itemsForCurrentRoot` / `itemsRemaining` is wrapped in `useMemo(…, [ items ])`.

Bundle delta: +85 B across `block-editor` and `blocks` scripts.

## Contribution

Opened by @Mamaduka closing #67805. @youknowriad raised a concern during review that items whose content depends on the selected block (e.g. child-block prioritization) might regress when the list stops re-rendering on selection change; the PR's approach of filtering a stable item set per root was the resolution. @ramonjd tested manually at 20× CPU throttling and reported no perceptible difference, while @Mamaduka noted the improvement was visible at 6×. The PR was AI-assisted (Claude). Co-authored by @ramonjd, @youknowriad, @ellatrix, and @jeryj.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
