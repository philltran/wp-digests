# #82505: Theme: Make chroma capacity caching deterministic

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] Base styles`, `[Package] Theme`
- **Merged:** [`37fc4b7`](https://github.com/WordPress/gutenberg/commit/37fc4b734616a59e34e68d95efa7449561b2cc88)
- **Discussion:** [#82505](https://github.com/WordPress/gutenberg/pull/82505) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`taperChroma` in `@wordpress/theme` caches max-chroma capacity using a key built from rounded lightness, hue, and chroma cap, but trunk computed the cached value from the first unrounded input that hit each bucket. The same ramp could therefore differ depending on which theme populated the cache first. The fix computes each cached capacity at the rounded lightness and hue the key represents, keeps the hard chroma cap exact, and regenerates the derived color ramps and design tokens.

## Impact

- **Plugin & theme developers using `--wpds-*` tokens:** Generated token values shift slightly. Most are 1-step hex changes (e.g. `#826a00` → `#836b00`, `#008030` → `#007f2f`), but several `*-weak` surface tokens change visibly: `--wpds-color-background-surface-{success,info,warning,caution,error}-weak` and `--wpds-color-background-interactive-error-active` now resolve to `#f8f8f8`, a neutral gray, instead of a tinted near-white. Check any custom styling that depends on those values or on the fallback values in `@wordpress/base-styles`.
- **Consumers of `@wordpress/theme` ramp generation:** Ramp output is now independent of cache population order, so multi-theme setups get stable results.
- **Reviewers of the diff:** The gray `-weak` values are likely a side effect of the rounding, not an intended design change. A reviewer reported all Notice components rendering with the same background after this PR, and the author confirmed it was caused by the cache rounding too aggressively and said they would fix or revert. The merged diff shown here still contains the gray values, so a follow-up fix is probably needed.
- No API changes and no configuration or migration required.

## Technical details

The core change is in `packages/theme/src/color-ramps/lib/taper-chroma.ts`. The cache key already quantizes lightness (0.05 steps), hue (10° steps), and the chroma cap. Previously the max-chroma capacity stored under a key was computed from whichever raw `(L, h)` input arrived first. Now it is computed at the quantized `(L, h)` matching the key, so any input in the same bucket yields the same result. The hard chroma cap is not rounded for the calculation. A new test in `packages/theme/src/color-ramps/test/taper-chroma.test.ts` checks that an unrelated input in the same bucket cannot alter a later result.

The author benchmarked the `background ramp snapshots` test: median 249 ms on trunk vs 251 ms here (+0.8%). An exact-key prototype ran at 541 ms (+117%), which is why the bucketed cache was kept.

Because ramp output changes, these were regenerated:

- `packages/theme/src/color-ramps/lib/default-ramps.ts`
- `packages/theme/tokens/color.json`
- `packages/theme/prebuilt/css/design-tokens.css`
- `packages/theme/prebuilt/js/design-token-fallbacks.mjs`
- `packages/base-styles/internal/_wpds-token-fallbacks.scss`
- the ramp snapshot file

A `Bug Fixes` entry was added to `packages/theme/CHANGELOG.md`. Bundle-size output shows `design-tokens.css` shrinking by about 23 B, consistent with the rewritten values.

## Contribution

This is a follow-up to a review thread on #82294, addressing an order-dependence flaw in the earlier caching work. After merge, @t-hamano noticed all Notice components had the same background color; @ciampo attributed it to over-aggressive rounding and said he'd fix or revert. @jsnajdr questioned whether the 0.05 lightness and 10° hue steps were really coarse enough to cause that, pointing to the max-chroma surface having sharp "spikes" at the RGB cube corners where a regular 20x36 grid approximates poorly, and suggested including corner values exactly in an irregular grid. OpenAI Codex assisted with the implementation, test, benchmarks, and regenerated files.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
