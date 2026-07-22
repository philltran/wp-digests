# #80423: Widget Dashboard: measure the header fit and document the chrome

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] Widget Dashboard`
- **Merged:** [`11fbad9`](https://github.com/WordPress/gutenberg/commit/11fbad9e1caa486df7091d08f28697b8dcf20ee1)
- **Discussion:** [#80423](https://github.com/WordPress/gutenberg/pull/80423) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Widget Dashboard tile header now dynamically measures available space instead of relying on fixed pixel reserves, preventing inline attribute controls from overflowing when help tooltips or action menus are present. The header title truncates with an ellipsis on narrow widths, and the internal fit logic has been simplified to compare natural width directly against the measured available space. This change also introduces Storybook documentation for the dashboard chrome and standardizes component naming across the package.

## Impact

- **Plugin & theme developers:** Internal component names and file paths in `@wordpress/widget-dashboard` have been renamed. If you imported `WidgetAttributeControls`, `widget-header-size.ts`, or the `WidgetAttributes` type, update your imports to `WidgetAttributes`, `widget-header-fit.ts`, and `WidgetAttributeValues` respectively.
- **Widget Dashboard users:** Tile headers now correctly collapse inline controls into a dropdown at the appropriate width, even with help notes or action menus present. Long titles truncate cleanly instead of wrapping.
- **No action required** for standard dashboard usage or widgets that do not extend or import the renamed internal paths.

## Technical details

The diff replaces the constant-based budgeting in `widget-header-size.ts` with a dynamic measurement system in `widget-header-fit.ts`. The header now publishes `availableSize` via `WidgetHeaderAvailableSizeContext`, and trailing sections register their footprint using the new `useReserveHeaderSpace` hook. This hook uses a `ResizeObserver` with `box: 'border-box'` to track width plus column gap, passing it to `registerReserved`/`unregisterReserved` in `WidgetHeaderReserveContext`.

The fit hook no longer accepts a `reservedSize` option, simplifying the collapse decision:

```diff
-const { reservedSize = 0, locked = false } = options;
+const { locked = false } = options;

-const computed = naturalSize > availableSize - reservedSize;
+const computed = naturalSize > availableSize;
```

`WidgetActions` wraps its menu in a `<span ref={ reserveRef }>` to reserve space, and `WidgetAttributes` uses `useReserveHeaderSpace( 'settings' )` for the settings trigger. CSS class `.widget-actions` now explicitly uses `display: inline-flex` to align with the flex-based reserve model.

## Contribution

Opened and merged by @retrofox, with co-authorship from @chihsuan. The PR consolidated a scattered fit calculation into a single reserve path, replacing hardcoded constants with live `ResizeObserver` measurements to eliminate layout drift. A deliberate rename pass was applied to align the vocabulary with `widget-actions` and free up the `WidgetAttributes` type name, executed via `git mv` to preserve history. The discussion remained focused on validating the measurement approach, with no alternative strategies pursued before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
