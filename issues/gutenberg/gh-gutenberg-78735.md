# #78735: Dashboard: Replace grid row height controls with size presets.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`
- **Merged:** [`5f74b58`](https://github.com/WordPress/gutenberg/commit/5f74b58e6c02fce9d8ac19a85b2d79305b5a8a6f)
- **Discussion:** [#78735](https://github.com/WordPress/gutenberg/pull/78735) · 5 comments · 0 reactions

## Summary

The WordPress dashboard's Standard grid layout settings no longer expose an "Auto-fit row height to content" toggle or a freeform "Row height (px)" number input. Both are replaced by a **Small / Medium / Large** toggle group mapping to fixed track heights of 200 px, 300 px, and 400 px respectively, with Medium (300 px) as the default. The change enforces a clean semantic boundary: uniform track heights belong in Standard grid, while content-driven sizing belongs in Masonry. Any persisted `rowHeight: 'auto'` values or arbitrary pixel heights are automatically normalized to the nearest preset on load, so no manual migration is required for existing dashboards. This PR is a direct companion to #78732, which applied the same preset-over-knobs approach to column controls.

## Impact

**Dashboard users**
- The free-form row height number input and the auto-fit toggle are gone from the Layout settings drawer.
- Three presets (Small = 200 px, Medium = 300 px, Large = 400 px) replace them; Medium is selected by default on new dashboards.
- Row height is hidden entirely when the layout is set to Masonry.

**Plugin/theme developers extending the dashboard grid**
- `WidgetGridLayoutSettings` now expects `rowHeight` typed as `number` only; the `'auto'` union member is removed from that component's interface.
- Any code that writes `rowHeight: 'auto'` or an arbitrary numeric height directly to dashboard grid preferences will be silently coerced to the nearest preset value (`snapRowHeight()`) on next load or commit — no explicit migration step is needed, but stored non-preset values will not survive a round-trip unchanged.
- `useDashboardGridSettings` and `WidgetDashboardProvider` have been updated; any custom code forking or wrapping these should account for the removed `'auto'` path.

**`@wordpress/grid` package consumers**
- No change. The grid package itself still accepts `rowHeight: 'auto'`; the constraint is enforced only by the dashboard layer, which always passes a numeric height down to the grid renderer.

**No action required** for standard site operators or integrators who do not write custom code against dashboard grid internals.

## Technical details

**New utilities — `utils/row-height-presets/`**
- `ROW_HEIGHT_PRESETS` — constant defining the three preset entries (key → px value).
- `rowHeightToPreset( px )` — maps a stored pixel height to its preset key (`'small'` | `'medium'` | `'large'`).
- `presetToRowHeight( key )` — inverse mapping from preset key to px.
- `snapRowHeight( value )` — snaps an arbitrary number (or `'auto'`) to the nearest valid preset pixel value.

**Normalization layer — `normalizeGridSettings()`**
A new function that accepts raw grid settings and returns a normalized copy where `rowHeight` is guaranteed to be one of the three preset pixel values. `WidgetDashboardProvider` calls this at three lifecycle points: staging (load from preferences), resolve (conflict resolution), and commit (persist to preferences). `useDashboardGridSettings` is wired to the same normalization path.

**DataForm field change for `rowHeight`**

Before:
```js
// NumberControl — free-form numeric input
{ id: 'rowHeight', type: 'integer', ... }
// Plus a separate boolean field for auto-fit
{ id: 'autoFitRowHeight', type: 'boolean', ... }
```

After:
```js
// ToggleGroup — preset keys ↔ stored px via getValue/setValue
{
  id: 'rowHeight',
  type: 'text',
  Edit: 'toggleGroup',
  getValue: ( { item } ) => rowHeightToPreset( item.rowHeight ),
  setValue: ( { item, value } ) =>
    ( { ...item, rowHeight: presetToRowHeight( value ) } ),
  elements: [
    { value: 'small',  label: __( 'Small' )  },
    { value: 'medium', label: __( 'Medium' ) },
    { value: 'large',  label: __( 'Large' )  },
  ],
}
```

**`WidgetGridLayoutSettings`** — `rowHeight` prop narrowed to `number`; the `auto` branch and the auto-fit checkbox are removed from the component's render path.

**Default values** — `DEFAULT_ROW_HEIGHT` (300, the Medium preset) is set as the fallback in both the dashboard provider and the preferences hook, replacing whatever previous default existed.

**`@wordpress/grid` package is unchanged**; it continues to accept `rowHeight: 'auto'` for any non-dashboard consumers.

## Contribution

Opened and authored by @jameskoster, with screenshots dated 2026-05-27. Merged at commit `5f74b58`. The PR is explicitly framed as a parallel to #78732 (column controls), applying the same "opinionated preset" philosophy to row geometry and completing the Layout settings simplification effort. Five comments are recorded on the PR but their content is not available in the provided material; no alternative approaches or rejected designs are documented in the description.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
