# #79808: Widgets: auto-save inline attribute edits

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`
- **Merged:** [`86b29a9`](https://github.com/WordPress/gutenberg/commit/86b29a9a2498e75a98fd9ab1d1332dd8e0d8fd2e)
- **Discussion:** [#79808](https://github.com/WordPress/gutenberg/pull/79808) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Introduces a shared, debounced auto-save mechanism (5-second delay) for inline attribute edits in the widget dashboard, ensuring staged layout and attribute changes persist when the user pauses typing or navigates away. It also fixes the Events location picker to only stage deliberate selections (`item-press`, `clear-press`, geolocation), preventing partial query text from being saved as a location.

## Impact

- **Widget & Dashboard Plugin Developers**: Inline attribute edits in the widget dashboard now auto-save after 5 seconds of inactivity instead of requiring manual commits or risking data loss on navigation/drawer opens.
- **Events Widget Users/Developers**: The `LocationPicker` no longer persists intermediate search text; only explicit selections are staged and saved.
- **Theme/Plugin Developers**: No immediate action required unless explicitly consuming the widget dashboard internal context. Public component APIs remain unchanged.

## Technical details

In `packages/widget-dashboard/src/context/dashboard-context.tsx`, a single shared debounce timer (`AUTO_SAVE_DELAY_MS = 5000`) replaces per-instance state/effect logic for staging commits. The context now provides `scheduleAutoSave()` (queues the staged layout for persistence) and `flushAutoSave()` (immediately publishes pending edits). `flushAutoSave` is automatically triggered when opening the settings drawer, entering customize mode, or unmounting the dashboard to prevent staged edits from commingling with drawer changes or being dropped on navigation.

In `widgets/events/components/location-picker/location-picker.tsx`, draft debouncing logic was removed. The `Autocomplete.Root.onValueChange` handler now checks `eventDetails.reason === 'item-press' || eventDetails.reason === 'clear-press'` before calling `onChange`, ensuring only deliberate selections are staged. Geolocation results also immediately call `onChange` when resolved.

## Contribution

Opened and merged by @retrofox (stacked on #79735). The PR shifted inline attribute staging from individual component hooks to a centralized dashboard context timer, simplifying the commit flow and adding explicit flush triggers for UI state transitions. Test coverage was added in `staging.test.tsx` to verify publish-on-debounce, immediate-flush, and unmount-persistence behaviors.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
