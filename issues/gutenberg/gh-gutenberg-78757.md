# #78757: Image cropper: round zoom control values and display as percentages

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Feature] Media`
- **Merged:** [`910a273`](https://github.com/WordPress/gutenberg/commit/910a2738ee4586bdb7b7874583aacea3a6769885)
- **Discussion:** [#78757](https://github.com/WordPress/gutenberg/pull/78757) · 7 comments · 0 reactions

## Summary

The block editor's image cropper (image editing modal) now displays its zoom control as a percentage rather than a raw multiplier. Previously the control surfaced the zoom multiplier directly, which could render imprecise float values like `3.749999999999999` in the input. The control is now labelled `Zoom (%)` and shows clean percentage values such as `150`, `375`, or `1000`. The underlying cropper state and crop math are unchanged — only the displayed/entered unit is converted.

## Impact

- **Site owners / editors:** The image editor crop panel now shows zoom as an intuitive percentage with a `Zoom (%)` label instead of an `x` multiplier with long floating-point values. No action required.
- **Plugin & theme developers:** No API change. The cropper continues to store zoom as a multiplier internally and still calls `setZoom` with a multiplier; the percentage is purely a presentation-layer conversion on the `RangeControl`. No breaking changes, deprecations, or migration needed.

## Technical details

The change reuses the existing `RangeControl` in the cropper's zoom UI. On render, the stored zoom multiplier is multiplied to a percentage for display; on input, the entered percentage is divided back to a multiplier before being passed to `setZoom`. Rounding the percentage value avoids exposing floating-point artifacts in the input field.

Before/after of what the user sees for an equivalent zoom level:

```
Before:  label "Zoom"      value 3.749999999999999
After:   label "Zoom (%)"  value 375
```

The cropper's internal state and crop calculations are untouched — the conversion is confined to the control's displayed value and its `onChange` handler. The build impact was a +34 B increase to `build/scripts/editor/index.min.js`, consistent with a small display-conversion change. No new hooks, filters, `block.json` fields, REST schema, or DB changes are introduced.

## Contribution

Authored and merged by @ramonjd (merge commit `910a273`), labelled `[Type] Enhancement` / `[Feature] Media`. The percentage-display idea came from a suggestion by @annezazu, relayed in review by @andrewserong, who asked whether showing the value as a percentage would make zoom clearer than the `x` multiplier. @ramonjd agreed the unit is largely immaterial since the control exists mainly for manual editing, and adopted the percentage presentation. Props recorded for @ramonjd and @andrewserong.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
