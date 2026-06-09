# #77599: Theme: Increase stroke1 contrast target to 2.9

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `Needs Design Feedback`, `[Package] Theme`
- **Merged:** [`7c2761c`](https://github.com/WordPress/gutenberg/commit/7c2761c3d7234105018ee85aa93bb22318f2d8e6)
- **Discussion:** [#77599](https://github.com/WordPress/gutenberg/pull/77599) · 8 comments · 0 reactions

## Summary

The Gutenberg theme package increased the `stroke1` contrast target from 2.6 to 2.9, resulting in slightly lighter UI dividers. This update primarily impacts the DataViews table layout and aligns the current stroke color with the legacy `$gray-100` variable.

## Impact

- **Theme & Frontend Developers**: The `--wpds-color-stroke-surface-neutral-weak` CSS custom property now renders at a lighter tone. No code modifications or configuration updates are required; simply update to the new release.
- **DataViews Consumers**: Table layouts and list-based components will display marginally lighter row dividers compared to the previous version.
- **No action required**: This is a visual asset regeneration with no breaking APIs, deprecated hooks, or migration steps.

## Technical details

The change modifies the `stroke1` contrast target configuration within the theme package, bumping it from 2.6 to 2.9. This directly updates the color computation that outputs `--wpds-color-stroke-surface-neutral-weak`, bringing it into alignment with the legacy `$gray-100` SASS variable. The update triggers a regeneration of prebuilt assets across `color.json`, default color ramps, UI snapshots, and compiled CSS/JS bundles in the `build/` directory. No PHP functions, block.json schemas, or REST endpoints were altered.

## Contribution

Opened by @jameskoster as a targeted follow-up to #75204 to decouple DataViews visual tweaks from core theme concerns. The author noted that reviewing the changes revealed the strokes are now "marginally lighter" and consistent with the pre-#75204 state. After brief design feedback, the change was merged at `7c2761c`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
