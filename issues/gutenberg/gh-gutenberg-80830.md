# #80830: DataViews: Add `time` field type and control

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Enhancement`, `[Feature] DataViews`, `[Package] DataViews`
- **Merged:** [`ca1f27e`](https://github.com/WordPress/gutenberg/commit/ca1f27ea6330826c0595dfe0b9c6ea7705d67113)
- **Discussion:** [#80830](https://github.com/WordPress/gutenberg/pull/80830) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The DataViews package now supports a `time` field type and matching `time` control, filling a gap where only `date` and `datetime` were previously available. Values are stored as wall-clock `HH:mm` or `HH:mm:ss` strings without date or timezone context, rendering identically regardless of the visitor’s locale. This enables developers to capture and filter pure time-of-day data in DataViews-driven interfaces.

## Impact

- **Plugin & theme developers:** Can now define `type: 'time'` in DataViews field configurations. The `format.time` property accepts PHP date format strings (e.g., `'g:i a'`) and automatically toggles the seconds field in the native `<input type="time">` control.
- **DataViews consumers:** Filter operators (`on`, `notOn`, `before`, `beforeInc`, `after`, `afterInc`, `between`) now apply to temporal values, allowing time-of-day comparisons alongside dates.
- **No action required** for existing DataViews implementations; this is a backward-compatible addition.

## Technical details

The change introduces `packages/dataviews/src/components/dataform-controls/time.tsx`, which registers the control via `packages/dataviews/src/components/dataform-controls/index.tsx`. The control normalizes incoming values to zero-padded `HH:mm[:ss]` using a `toInputValue()` helper and dynamically calculates the `step` attribute based on whether the `format.time` string or stored values contain seconds. Filter logic in `packages/dataviews/src/components/dataviews-filters/filter.tsx` was updated to treat `null` or `undefined` bounds in `between` ranges as unfilled, preventing dangling filter chips. The `README.md` documents the new `format.time` configuration and operator support. Values are anchored to a fixed date internally to safely use `dateI18n` for formatting, ensuring DST transitions do not shift the wall-clock value.

## Contribution

Opened and merged by @ntsekouras with co-authors @jorgefilipecosta and @elazzabi. Reviewer @dmallory42 validated the implementation and suggested that `getStep()` should return `1` when seconds are present in the value, a refinement that was incorporated before merge. The PR also bundled a minor filter chip bug fix for incomplete `between` ranges.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
