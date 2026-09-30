# #83437: Inserter: Manage roving tab index to skip re-rendering every item

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Feature] Inserter`, `[Type] Performance`, `[Package] Block editor`
- **Merged:** [`df1a193`](https://github.com/WordPress/gutenberg/commit/df1a19300de1dcded6c60cb29fdb2364cde676be)
- **Discussion:** [#83437](https://github.com/WordPress/gutenberg/pull/83437) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Opening the Block Inserter no longer triggers two full re-renders of every list item after mount. The fix passes `tabbable` and `type="button"` to Ariakit's `CompositeItem` in `InserterListboxItem`, and derives `tabIndex` directly from the `data-active-item` attribute instead of relying on Ariakit's post-mount state updates. With 500 extra registered blocks, main-thread blocking from opening the inserter drops from roughly 88 ms to 55 ms.

## Impact

- **Site owners / editors:** No visible change. Keyboard navigation (Tab, Shift+Tab, Arrow keys) in the inserter behaves identically; a new e2e test verifies the roving tab index follows focus correctly.
- **Plugin & theme developers:** No action required. No public API, hook, or `block.json` field changed. The change is internal to `@wordpress/block-editor`'s inserter rendering.
- **Hosting & platform:** Sites with a large number of registered blocks (hundreds or more) will see reduced main-thread blocking when the inserter opens. No configuration or migration needed.
- **Headless & REST consumers:** No effect.

## Technical details

The change is confined to `packages/block-editor/src/components/inserter-listbox/item.jsx`.

Two props are added to the `CompositeItem` call:

- `tabbable` — tells Ariakit to skip its internal `tabIndex` management. Without it, `CompositeItem` marks every item tabbable on mount, then flips all non-active items to `tabIndex="-1"` once `renderedItems` is populated, causing one re-render per item.
- `type="button"` — tells Ariakit's `useCommand` hook to skip its `isNativeButton` state update in a post-mount effect. Without an explicit `type`, the hook sets state after mount, triggering a second re-render per item.

The `tabIndex` derivation in the `render` callback changes from:

```jsx
// before
tabIndex: isFirst ? 0 : htmlProps.tabIndex,
```

to:

```jsx
// after
tabIndex: isFirst || htmlProps[ 'data-active-item' ] ? 0 : -1,
```

This means only the previously active item and the newly active item re-render when focus moves, rather than every item in the list.

A new e2e test in `test/e2e/specs/editor/various/inserting-blocks.spec.js` (tagged `-webkit`) verifies that the first option has `tabindex="0"`, the second has `tabindex="-1"`, Tab leaves the list, Shift+Tab returns focus, and ArrowRight moves both focus and the `tabindex="0"` stop to the next item.

Bundle size impact: +22 B in `build/scripts/block-editor/index.min.js`.

## Contribution

Opened by @Mamaduka with AI assistance (Claude). Co-authored with @andrewserong and @mirka. The author requested merge with no objections; the PR carried 3 comments and no reactions, with no visible design debate or rejected alternatives in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
