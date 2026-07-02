# #79683: Widget Dashboard: Anchor settings drawer to the right and toggle it from the gear

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`, `[Package] Widget Dashboard`
- **Merged:** [`717ab62`](https://github.com/WordPress/gutenberg/commit/717ab628fc4ca189068c16b3a25a0fae5f62b514)
- **Discussion:** [#79683](https://github.com/WordPress/gutenberg/pull/79683) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Widget Dashboard settings drawer now always slides in from the right edge, replacing the previous logic that dynamically calculated left or right anchoring based on a widget’s viewport position. Additionally, clicking a widget’s gear icon now toggles its settings drawer open and closed instead of only opening it. This simplifies the dashboard’s internal state management and removes viewport-based layout calculations that added unnecessary complexity.

## Impact

* **Widget Dashboard & internal package developers**: Internal context shape in `@wordpress/widget-dashboard` changed. The `settingsDrawerSide` and `settingsDrawerInset` state keys (and their setters) have been permanently removed from `DashboardUIContextValue`. Direct consumption or extension of this context will throw type/runtime errors until updated.
* **Plugin & theme developers**: No outward-facing API or public contract changes. Unless you are actively forking, extending, or deeply customizing the `@wordpress/widget-dashboard` package internals, no migration is required.
* **Hosting, platform & headless consumers**: No impact. Drawer anchoring and gear behavior are client-side UX elements isolated to the dashboard UI layer.

## Technical details

The change consolidates drawer behavior and state in three files within `packages/widget-dashboard/src/components/widget-settings/`:

- **`ui-context.tsx`**: Removed the `DrawerSide` type alias and dropped `settingsDrawerSide`, `setSettingsDrawerSide`, `settingsDrawerInset`, and `setSettingsDrawerInset` from the `DashboardUIContextValue` interface. The provider no longer initializes or exposes these values.
- **`widget-settings-trigger.tsx`**: Replaced the previous `open` callback with a `toggle` function. It now checks if `settingsWidgetUuid === widget.uuid`; if true, it calls `cancel()` to discard staged edits and sets the UUID to `null`. Otherwise, it assigns the active UUID.
- **`widget-settings.tsx`**: Hardcoded `swipeDirection="right"` on `<Drawer.Root>`, removing the dynamic side selection. The `popupStyle` `useMemo` hook and its inline style prop were removed from `<Drawer.Popup>`.
- **Utility cleanup**: Deleted `utils/get-admin-menu-inset.ts` and its re-export from `utils/index.ts`. The dynamic inset calculation that previously offset left-anchored drawers past the fixed admin menu is entirely gone.

## Contribution

Opened by @retrofox with co-authorship from @simison, the PR targeted simplification of the dashboard's UI context. Review discussion confirmed that removing viewport-position calculations and dynamic anchoring favored predictable placement over computational complexity. The gear toggle behavior was added to match standard open/close UX patterns. Merged following lightweight review without structural or behavioral trade-offs.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
