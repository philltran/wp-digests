# #79198: Theme: Add stroke-surface tokens for the caution tone

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Theme`
- **Merged:** [`4b3e0aa`](https://github.com/WordPress/gutenberg/commit/4b3e0aac2b75791ae133e8a1ed407577166a233d)
- **Discussion:** [#79198](https://github.com/WordPress/gutenberg/pull/79198) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The WordPress Design System theme package now exposes two new CSS custom properties: `--wpds-color-stroke-surface-caution` and `--wpds-color-stroke-surface-caution-strong`. These tokens fill a gap in the status tone palette, giving the `caution` tone the same stroke-surface coverage as `success`, `info`, `warning`, and `error`. This ensures consistent surface boundary styling across all status tones in the design system.

## Impact

- **Theme & plugin developers**: Can now reference the new tokens for caution-toned surface borders without hardcoding colors.
- **Block developers**: No direct block-level impact; tokens are consumed via the theme package.
- **No action required**: The change is strictly additive and backward-compatible.

## Technical details

The diff modifies `packages/theme/tokens/color.json` to add `caution` and `caution-strong` entries under the `stroke-surface` group, mapping to `wpds-color.primitive.caution.stroke1` (`#cfc28d`) and `wpds-color.primitive.caution.stroke3` (`#826a00`). Running `npm run build` regenerates the prebuilt outputs: `src/prebuilt/css/design-tokens.css`, `src/prebuilt/js/design-token-fallbacks.mjs`, `src/prebuilt/js/design-tokens.mjs`, `src/prebuilt/ts/color-tokens.ts`, and `src/prebuilt/ts/token-types.ts`. The TypeScript type `SurfaceStrokeColor` is extended to include `'caution'` and `'caution-strong'`, and the `color-tokens.ts` mapping adds `caution-stroke1` and `caution-stroke3` aliases.

## Contribution

Opened by @ciampo and merged with review from @aduth. During review, @aduth questioned why the `caution` tone wasn't exposed as an `intent` option for the `Notice` component. @ciampo noted that `Badge` and `Notice` already use divergent intent value sets, and @jasmussen clarified that `Badge` intentionally supports a broader urgency/power spectrum, while `Notice` sticks to a simpler warning/success/error/info trio. The team agreed to track potential alignment separately in #79239 rather than scope it into this token addition.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
