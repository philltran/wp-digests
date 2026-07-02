# #79735: Widgets: add attribute `relevance` and inline editing in the tile toolbar

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] Widget primitives`, `[Package] Widget Dashboard`
- **Merged:** [`8d5add3`](https://github.com/WordPress/gutenberg/commit/8d5add3f471a383b032fdc3809e1287932675493)
- **Discussion:** [#79735](https://github.com/WordPress/gutenberg/pull/79735) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds a `relevance` hint (`'high' | 'low'`) to the widget attribute metadata schema, enabling inline editing of high-priority attributes directly in the tile toolbar. The new `WidgetAttributeControls` component filters for `relevance: 'high'` and renders them as a compact, label-less `DataForm` row with immediate publishing. This shift moves commonly edited fields (demonstrated by the `events` widget's `location` attribute) out of the settings drawer and onto the tile header itself.

## Impact

- **Widget developers**: Can opt into inline editing by adding `relevance: 'high'` to their attribute definitions. All existing attributes default to `low`. No breaking changes, but represents a structural API shift for the widget primitives package.
- **Platform/Host**: The widget dashboard now conditionally renders the settings trigger (`WidgetSettingsTrigger`) only when un-promoted attributes remain. If all attributes are marked high, the gear icon is hidden entirely.
- **Action required**: None for current users. Adopting the feature requires updating attribute schemas and reviewing the new `editMode` / `headerToolbar` props on tile chrome components.

## Technical details

The diff introduces `relevance?: 'high' | 'low'` to `WidgetAttributeField< Item >` and `WidgetAttribute< Item >` in `@wordpress/widget-primitives`. It adds a new `WidgetAttributeControls` component that extracts high-relevance attributes and renders them via `DataForm` with `labelPosition: 'none'`, applying a pending-commit pattern for immediate saves.
- `WidgetChrome`, `WidgetFrame`, `WidgetHeader`, and `WidgetToolbar` are updated to accept `editMode` and `headerToolbar` props, moving the toolbar into the in-card header for normal mode while keeping it interactive outside the card during customize/full-bleed modes.
- `WidgetSettingsTrigger` now checks `hasNonPromotedAttributes` (filtering for `relevance !== 'high'`) and returns `null` when all fields are promoted, changing its icon to `moreVertical`.

## Contribution

Opened by `@retrofox` to support the broader widget settings redesign (#79246). `@simison` provided focused UI/UX feedback on empty states, placeholder text, and debounce timing, deferring mobile responsiveness and staged saves to follow-ups. Co-authored by `@nikschavan` and `@simison`, merged after a design-focused review cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
