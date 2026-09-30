# #78010: Components: Add `defaultShown`, `onShownChange` to `ToolsPanelItem`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @im3dabasia
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Feature] Design Tools`
- **Merged:** [`843371b`](https://github.com/WordPress/gutenberg/commit/843371b8aedfb0fa0ac41854c5640fb2c6697e0c)
- **Discussion:** [#78010](https://github.com/WordPress/gutenberg/pull/78010) · 10 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`ToolsPanelItem` in `@wordpress/components` gains two props: `defaultShown`, which makes an optional item with no value visible on first render, and `onShownChange( isShown )`, which fires only when the user shows or hides the item via the panel's ⋮ menu. Previously there was no way to seed an optional item as visible or to detect a hide action on an item with no value, because `onDeselect` is gated on `hasValue()` and means "reset this control." This is groundwork for persisting which optional controls a user has shown (#41544); the editor-side persistence is not part of this PR.

## Impact

- **Plugin & theme developers using `ToolsPanel`:** Additive and backward compatible. `onSelect` and `onDeselect` behave as before, so no existing consumers need changes.
- **Developers who want to persist optional-control visibility:** You can now restore a saved choice with `defaultShown` and record changes with `onShownChange`.
- **Watch for double handling:** Showing an optional item with no value fires both `onShownChange( true )` and `onSelect`; hiding an item with a value fires both `onShownChange( false )` and `onDeselect`. Don't wire one handler to both.
- **Site owners, hosting, headless/REST:** No action required.

## Technical details

**New props (`tools-panel-item/hook.ts`, README, CHANGELOG):**

- `defaultShown?: boolean` (default `false`). Applies to optional items only. It seeds visibility when the item registers with the panel. An item with a value is always shown regardless of this prop.
- `onShownChange?: ( isShown: boolean ) => void`. Fires from the panel's `toggleItem`, so only a real menu click triggers it.

**When `onShownChange` does not fire:**
- On mount or registration, including with `defaultShown`.
- When an item is shown or hidden because it gained or lost a value.
- On `Reset all`.
- For items with `isShownByDefault`, which stay visible when toggled off.

**Hook implementation:**

- `defaultShown` is deliberately non-reactive. `useToolsPanelItem` holds it in `defaultShownRef`, updated by a `useLayoutEffect` declared before the registration effect. The ref keeps `defaultShown` out of the registration effect's dependencies, so changing the prop cannot deregister and re-register the item and discard the user's choice. A re-registration caused by something else, such as `panelId` changing, seeds from the latest value.
- `onShownChange` is wrapped with `useEvent` from `@wordpress/compose`. The item registers a stable callback while the panel invokes the latest one.
- `registerPanelItem` now receives `defaultShown` and `onShownChange` alongside `hasValue`, `isShownByDefault`, `label` and `panelId`.
- The menu-toggle effect gains a `wasRegistered = wasMenuItemChecked !== undefined` guard. Without it, an item registering already shown would look like the user had just selected it and would fire `onSelect`.
- A test covers that an item the user hid, stored as `false`, is not reseeded from `defaultShown` when another item registers.

```jsx
<ToolsPanelItem
	label="Scale"
	hasValue={ () => !! scale }
	defaultShown={ savedPrefs.scale }
	onShownChange={ ( isShown ) => savePref( 'scale', isShown ) }
	onDeselect={ resetScale }
>
	...
</ToolsPanelItem>
```

The diff was truncated, so the changes to the panel-side `toggleItem` and menu-state code are not visible here.

## Contribution

The first approach was to make `onDeselect` fire on visibility transitions regardless of `isValueSet`. That caused a double `onChange` when setting aspect ratio to "Original", which programmatically hides the scale item. @aaronrobertshaw floated `onShow`/`onHide` handlers, and possibly renaming `onDeselect` to `onReset`, while stressing backward compatibility and keeping visibility tracking at the item level. @ciampo suggested standard controlled/uncontrolled props (`shown`/`defaultShown`/`onShownChange`). Aaron argued against a controlled `shown` because an item with a value is always shown, so visibility isn't independent state. The merged result takes the uncontrolled half of Ciampo's naming, a single `onShownChange` plus `defaultShown`, and leaves `onSelect`/`onDeselect` untouched. @im3dabasia kept the editor persistence consumer out as a follow-up on #41544.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
