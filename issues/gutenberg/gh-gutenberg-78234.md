# #78234: Widget dashboard: skip tile hover elevation while resizing

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`, `[Package] Grid`
- **Merged:** [`c1f09a1`](https://github.com/WordPress/gutenberg/commit/c1f09a1b7ab8a87d443194438e3fa5febb3440c9)
- **Discussion:** [#78234](https://github.com/WordPress/gutenberg/pull/78234) · 3 comments · 0 reactions

## Summary

The Widget Dashboard now attaches a `data-wp-dashboard-grid-resizing` attribute to the `DashboardGrid` root element while any tile is actively being resized. This prevents hover elevation shadows from flickering or overriding during drag gestures, improving visual stability in the dashboard editor without altering functional behavior.

## Impact

- **Site owners**: No action required; this is an internal UI refinement.
- **Plugin & theme developers**: No breaking changes or API removals. If you manually style dashboard tiles via CSS overrides, note that the grid now exposes a resize state attribute to suppress hover elevation programmatically.
- **Hosting & platform teams**: No configuration or migration steps needed; behavior is isolated to the Gutenberg/Full Site Editing editor surface.
- **Headless & REST consumers**: Unaffected; changes are strictly client-side UI layer adjustments.

## Technical details

The diff modifies three files to gate hover elevation during resize gestures:
- `packages/grid/src/dashboard-grid/index.tsx`: Mirrors an existing internal `isResizing` state onto the root element as a stable `data-wp-dashboard-grid-resizing` attribute.
- `widgets.module.css` (referenced in PR body): Introduces a higher-specificity override under `:global([data-wp-dashboard-grid-resizing])` that locks tile elevation to `--wpds-elevation-xs`, explicitly suppressing `.tileEditMode:hover` and `:focus-visible` states for the gesture duration.
- `.stylelintrc.js`: Adds `selector-pseudo-class-no-unknown` configuration to ignore the `global` pseudo-class in CSS Modules, allowing unscoped markup targeting (like the grid root's data attribute) without triggering lint errors.
No PHP hooks, block.json fields, REST routes, or database changes are involved.

## Contribution

Merged as commit `c1f09a1` by @jameskoster (co-authored with @retrofox). The PR resolves a visual flicker caused by persistent hover states during widget resizing. Instead of toggling JavaScript classes or interrupting pointer events, the solution exposes an existing `isResizing` flag as a declarative `data-*` attribute, allowing CSS-level state suppression. A parallel stylelint rule update was included to support the new `:global()` pattern for targeting unscoped root markup.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
