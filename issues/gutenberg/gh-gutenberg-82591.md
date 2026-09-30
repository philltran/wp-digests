# #82591: Theme: Remove chroma capacity cache

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] Base styles`, `[Package] Theme`
- **Merged:** [`42e0f55`](https://github.com/WordPress/gutenberg/commit/42e0f55e5b07f07faad88c49f4edfb4a31639343)
- **Discussion:** [#82591](https://github.com/WordPress/gutenberg/pull/82591) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/theme` color-ramp generator no longer uses a rounded chroma-capacity cache. The cache rounded a requested lightness of `0.98` to `1`, where chroma capacity is zero for every hue, so all four weak intent backgrounds (info, success, warning, error) rendered as neutral gray. Chroma capacity is now computed exactly at the requested lightness and hue, restoring the tinted weak backgrounds. Generated design tokens and fallbacks were rebuilt from the exact calculation.

## Impact

- **Plugin/theme developers using `@wordpress/theme` design tokens or Gutenberg components that consume them (e.g. Notice):** the `*-weak` surface tokens change from gray `#f8f8f8` to tinted values. Expect visible color changes, so update any visual-regression snapshots.
- **Other tokens shift slightly:** many normal-emphasis surface, stroke, and foreground tokens move by a channel value or two (e.g. `--wpds-color-foreground-content-success-weak` `#007f2f` → `#008030`, `--wpds-color-stroke-surface-info` `#aac6e5` → `#adc6e2`), because the ramps are recomputed exactly rather than from rounded cache values.
- **Custom `ThemeProvider` consumers generating many unique ramps:** bulk generation is slower. In the PR's 459-unique-ramp stress test, the median went from about 294 ms to about 1022 ms. The default workload (one background plus six accent ramps) goes from about 14.9 ms to about 20.8 ms.
- **Site owners:** no action required.

## Technical details

The change removes the chroma-capacity cache in the `packages/theme` color-ramp code, so max chroma is calculated per call for the requested lightness and hue instead of being looked up from a rounded key. Regression tests were added for both lightness boundaries, gradual tapering near white, and the restored default weak background colors. The `color-ramps` source file is not visible in the truncated diff, so this description is based on the PR text plus the visible generated output.

The visible diff is regenerated output and a changelog line:

- `packages/base-styles/internal/_wpds-token-fallbacks.scss`
- `packages/theme/prebuilt/css/design-tokens.css`
- `packages/theme/prebuilt/js/design-token-fallbacks.mjs`
- `packages/theme/CHANGELOG.md`, which adds a Bug Fixes entry

Default `surface2`-based weak backgrounds are now:

```css
--wpds-color-background-surface-info-weak:    #f3f9ff; /* was #f8f8f8 */
--wpds-color-background-surface-success-weak: #ebffed; /* was #f8f8f8 */
--wpds-color-background-surface-warning-weak: #fff7e1; /* was #f8f8f8 */
--wpds-color-background-surface-error-weak:   #fff6f5; /* was #f8f8f8 */
```

`--wpds-color-background-interactive-error-active` also changes from `#f8f8f8` to `#fff6f5`. No new hooks, APIs, or REST/DB changes.

## Contribution

This is a follow-up to #82505, which made ramp generation independent of cache population order. The author, @ciampo, evaluated three alternatives: a denser interpolated sampling grid (still approximate), exact memoization (unbounded, reaching 15,718 entries in the stress test), and removing the cache. He chose removal, favoring correctness over synthetic-benchmark speed, and noted #82294 would have eroded some of the cache's gains anyway. In discussion, @jsnajdr pointed to #82643, an independent analytic max-chroma calculation intended as a faster exact replacement. He was doubtful it would suit upstream color.js, since gamut-mapping algorithms are specified rather than open to creative variation. The PR description discloses that OpenAI Codex assisted with the work.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
