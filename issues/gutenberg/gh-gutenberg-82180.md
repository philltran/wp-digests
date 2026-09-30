# #82180: Components: Cut the ToolsPanel render cascade

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Components`
- **Merged:** [`584641f`](https://github.com/WordPress/gutenberg/commit/584641f121fa4ae9f8a757dbf0433fade8e72740)
- **Discussion:** [#82180](https://github.com/WordPress/gutenberg/pull/82180) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`ToolsPanel` and `ToolsPanelItem` in `@wordpress/components` no longer trigger a multi-pass render cascade when the selected block changes. Item state dispatches are folded into two layout effects, items stop deregistering/re-registering when the panel's `panelId` changes, and the panel's menu is now derived during render with `useMemo` instead of being rebuilt in the reducer on every dispatch. Two behavior fixes ride along: a default control whose value survives `Reset all` stays offered as resettable, and an item's `resetAllFilter` now sees current props rather than a stale capture.

## Impact

**Plugin & theme developers (using `ToolsPanel` / `ToolsPanelItem`, including via `InspectorControls`)**
- No API changes and no action required. Props on `ToolsPanel` and `ToolsPanelItem` are unchanged.
- `resetAllFilter` now always runs against the item's latest props (via `useEvent`). Filters that relied on the old frozen-per-`panelId` capture would see different behavior, though that was the buggy path.
- After `Reset all`, a default (`isShownByDefault`) control whose value is not cleared by `resetAll`, `onDeselect` or `resetAllFilter` stays resettable instead of reading as unmodified with no way to reset it.
- Consumers that call `useToolsPanelContext()` or provide a custom `ToolsPanelContext` should note the internal context shape changed (`deregisterPanelItem` now receives the registration as a second argument; items carry `resetAllFilter` on registration). `registerResetAllFilter` remains on the context because `block-editor`'s `inspector-controls/fill.js` registers filters directly.

**Site owners / editor users**
- Reduced work on block selection. Per the PR's measurements in the post editor: reducer dispatches 70 (two batches) → 18 (one); `ToolsPanel` renders 16 → 12; `ToolsPanelItem` renders 68 → 42. The author notes in the discussion this doesn't move block-selection metrics meaningfully.

**Hosting / headless**
- Not affected.

## Technical details

**`tools-panel-item/hook.ts`**
- Removes `usePrevious( currentPanelId )` and the `previousPanelId !== null` gating. Registration is now a single `useLayoutEffect` that registers whenever `hasMatchingPanel` (`currentPanelId === panelId || currentPanelId === null`) is true. Multi-selection (`panelId === null`) therefore no longer causes a deregister/register pair.
- The item object passed to `registerPanelItem` now includes `resetAllFilter`. The separate passive effect calling `registerResetAllFilter` / `deregisterResetAllFilter` is removed.
- Cleanup calls `deregisterPanelItem( label, item )`, passing the registration so the panel can ignore a late cleanup if a replacement already claimed the label (relevant when panels switch while items arrive through a Slot).
- `resetAllFilterCallback` changes from `useCallback( resetAllFilter, [ panelId ] )` to `useEvent( resetAllFilter )`. `hasValueCallback` stays `useCallback( hasValue, [ panelId ] )` because the panel calls it during render while deriving the menu, which `useEvent` forbids.
- The customization flag effect moves from `useEffect` to `useLayoutEffect`, and `flagItemCustomization( isValueSet, label )` drops its `group` argument.
- The `onSelect`/`onDeselect` passive effect now tracks the previous checked state in a `useRef` updated inside the effect (replacing `usePrevious`/`wasRegistered` render-time values).

**`tools-panel/hook.ts`**
- Reducer state changes from `{ panelItems, menuItemOrder, menuItems }` to `{ panelItems, menuItemOrder, menuItemValues }`. `menuItemValues` holds only what can't be read back off items: an optional item the user has shown and a default item flagged as customized. Everything else falls back to the item's `hasValue()`.
- `menuItems` is now a `useMemo` over `panelItems`, `menuItemOrder` and `menuItemValues`, replacing the reducer-side `generateMenuItems`. This preserves the #65564 guarantee (no half-built menu painted) since it derives in render rather than in an effect.
- `UNREGISTER_PANEL` accepts an optional `item`; `UPDATE_VALUE` drops `group`. `RESET_ALL` records `false` for optional items and drops default items so each falls back to the value it still holds.
- Types: `ToolsPanelItem` (registration shape) is renamed `RegisteredToolsPanelItem` in the hook's imports, and `ToolsPanelMenuItemsConfig` is no longer used there.

```js
// before
deregisterPanelItem( label );
// after
deregisterPanelItem( label, item );
```

**Tests** (`index.jsdom.test.tsx`) are updated for the new deregister signature and the no-churn behavior on `panelId: null`, and add cases for replacement items not inheriting outgoing state, `resetAllFilter` seeing latest props, and a default control staying resettable after `Reset all`. The diff was truncated, so remaining changes (e.g. `types.ts`, panel context wiring) are not reviewed here.

## Contribution

Opened by @Mamaduka while investigating block-selection performance, revisiting an earlier comment on #65564. He was candid that it doesn't shave anything meaningful off those metrics but considered fewer render passes and simpler logic worth it. Props-bot credits @ciampo alongside the author. The PR notes it was assisted by Claude. CI reported one flaky e2e test (guidelines spec) unrelated to the change.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
