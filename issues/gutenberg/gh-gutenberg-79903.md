# #79903: Widget Dashboard: remove layout-settings editing

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] Widget Dashboard`
- **Merged:** [`e7861af`](https://github.com/WordPress/gutenberg/commit/e7861afc69d120603c2e56ef8ea1aadb9b222981)
- **Discussion:** [#79903](https://github.com/WordPress/gutenberg/pull/79903) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Widget Dashboard package removes all in-dashboard UI for editing layout settings, stripping out the 'Layout settings' toolbar button, the model-switcher drawer, and corresponding command palette commands. The `gridSettings` prop is now strictly read-only rendering configuration; host consumers are fully responsible for persisting and applying grid model preferences outside the dashboard.

## Impact

- **Host implementers / plugin & theme developers**: Remove usage of the `onGridSettingsChange` prop from `<WidgetDashboard>`. The package no longer manages layout-settings edits, meaning any custom UI for switching between `grid` and `masonry` models must be built externally.
- **End users**: No direct impact; standard widget reordering, resizing, and reset actions remain unchanged.
- **Action required**: Update dashboard host implementations to decouple grid-model switching from the internal staging/render flow and persist changes via your own data layer.

## Technical details

- Dropped `packages/widget-dashboard/src/components/layout-settings/` entirely (including `LayoutModelEditField`, thumbnail components, and associated CSS modules).
- Stripped `onGridSettingsChange` from `<WidgetDashboard>`'s public API contract.
- In `actions.tsx`, removed the `layout-icon` import, `canEditGridSettings` context hook, `setLayoutSettingsOpen` UI action, and the conditional rendering of the 'Layout settings' button.
- In `commands.tsx`, unregistered `core/dashboard/switch-to-masonry-layout` and `core/dashboard/switch-to-grid-layout` commands, removed `getGridModel`, `commitGridModelChange`, and related state hooks from the context provider.
- `gridSettings` now flows purely as read-only render configuration; `normalizeGridSettings` remains exported for host-side normalization, but layout staging is no longer exposed or editable within the dashboard.

## Contribution

Opened and merged by @retrofox (co-authored by @chihsuan) in PR #79903. The change supersedes #79891, which previously made the customize toolbar composable so hosts could optionally drop the layout button. The refactor was driven by a design decision to decouple widget arrangement from layout-engine configuration, keeping the dashboard focused on instance management while leaving grid-model persistence to the host application.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
