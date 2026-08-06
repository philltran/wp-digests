# #81198: DataViews: Pass only eligible items to a bulk action callback

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Package] DataViews`, `Backport to WP 7.1 Beta/RC`
- **Merged:** [`f7f5670`](https://github.com/WordPress/gutenberg/commit/f7f5670d563e8fce3308fb1546304a0ed007fc1e)
- **Discussion:** [#81198](https://github.com/WordPress/gutenberg/pull/81198) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

DataViews now filters the array passed to a bulk action's `callback` so it only contains items that pass the action's `isEligible` check. Previously, the callback received every selected item, which caused bulk actions to incorrectly process ineligible items (e.g., attempting to restore a published page alongside a trashed one). This fix prevents unintended state changes and ensures callbacks only operate on valid targets.

## Impact

- **Plugin & theme developers:** If you register bulk actions with an `isEligible` function, your `callback` will now receive a filtered subset of items instead of the full `selectedItems` array. Update any logic that assumes the callback receives all selected rows.
- **Site owners & editors:** Bulk actions like "Restore" or "Trash" will now behave correctly when mixed selections are active, preventing accidental status changes to ineligible items.
- **No action required** for users not interacting with DataViews bulk actions or customizing them via `isEligible`.

## Technical details

The change modifies `packages/dataviews/src/components/dataviews-bulk-actions/index.tsx`. In the `ActionButton` component's `onClick` handler, the callback invocation was updated from:
```tsx
await action.callback( selectedItems, { registry } );
```
to:
```tsx
await action.callback( selectedEligibleItems, { registry } );
```
`selectedEligibleItems` is computed by filtering `selectedItems` against the action's `isEligible` predicate. This aligns the callback's input with the visibility logic in `actionsToShow`, which already uses `.some()` to determine if an action should appear. A unit test was added to `packages/dataviews/src/dataviews/test/dataviews.tsx` verifying that a bulk action with `isEligible: ( item ) => item.id === 1` only receives that single item when multiple rows are selected.

## Contribution

Opened by @ramonjd and merged after review by @ntsekouras and @t-hamano. The fix was identified during unrelated research and quickly approved as a low-risk bug fix. It was backported to the `wp/7.1` branch following a manual cherry-pick to resolve a merge conflict, ensuring inclusion in the 7.1 RC cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
