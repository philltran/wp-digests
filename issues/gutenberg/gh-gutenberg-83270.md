# #83270: Components: Align RadioControl colors with UI Radio

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Base styles`
- **Merged:** [`b97d497`](https://github.com/WordPress/gutenberg/commit/b97d49745bfc4c2fa910374ae26e7f7e05dc31a0)
- **Discussion:** [#83270](https://github.com/WordPress/gutenberg/pull/83270) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `radio-control` SCSS mixin in `@wordpress/base-styles` now uses the same `--wpds-*` design tokens as the `@wordpress/ui` Radio primitive, so `@wordpress/components` `RadioControl` matches it visually. The default and hover border, selected fill and thumb, and disabled fill, stroke and thumb colors all change. This is part of the effort (#76135) to let `Radio` sit alongside `RadioControl` in Gutenberg.

## Impact

- **Plugin & theme developers using `RadioControl`:** Colors change with no API change. The unchecked border moves from `$gray-900` to the neutral interactive stroke token (`#8d8d8d` per the PR's testing notes). It darkens to `#6e6e6e` on hover. The selected thumb is now `#eff0f2` instead of white. The disabled fill is transparent instead of light gray. Visual regression snapshots that include `RadioControl` (including DataViews single-select filters) may need updating.
- **Consumers of the `radio-control` mixin from `@wordpress/base-styles/mixins`:** The mixin now emits `wpds.var(...)` tokens, so anything that includes it inherits the new look. It also now includes `:hover` and `[aria-disabled="true"]`/`:disabled` rules, which it did not have before.
- **Custom CSS overriding the old accent colors:** Code that relied on `--wp-admin-theme-color` driving the checked or focus color now gets `--wpds-color-background-interactive-brand-strong` and `--wpds-color-stroke-focus`. Check those overrides.
- **Site owners:** No action required. The change is purely cosmetic.
- **Bundle size:** The PR's bundle report shows about +913 B of CSS across the builds.

## Technical details

**`packages/base-styles/_mixins.scss` (`@mixin radio-control`):**

- Default border changes from `variables.$border-width solid colors.$gray-900` to `wpds.var("--wpds-border-width-xs") solid wpds.var("--wpds-color-stroke-interactive-neutral")`. A `background` of `--wpds-color-background-interactive-neutral-weak` is added.
- A new `&:hover` sets `border-color` to `--wpds-color-stroke-interactive-neutral-active`.
- The checked thumb (`&:checked::before`) uses `currentColor` for `background-color` and its 4px border, instead of `colors.$white`. The Windows High Contrast fallback border still exists.
- `&:checked` now sets `background: --wpds-color-background-interactive-brand-strong`, `border-color: transparent` (previously `border: none`), and `color: --wpds-color-foreground-interactive-brand-strong`. That `color` value is what `currentColor` resolves to for the thumb.
- The focus ring's outer color changes from `var(--wp-admin-theme-color)` to `wpds.var("--wpds-color-stroke-focus")`.
- A new `&[aria-disabled="true"], &:disabled` block sets these:
  - background: `--wpds-color-background-interactive-neutral-weak-disabled`
  - border-color: `--wpds-color-stroke-interactive-neutral-disabled`
  - color: `--wpds-color-foreground-interactive-neutral-disabled`
  - `cursor: default`
  - `opacity: 1`, to override wp-admin `forms.css`
  - a `@media (forced-colors: active)` branch using `GrayText`

**`packages/components/src/radio-control/style.scss`:** The now-redundant component-level overrides are removed. These are the `$components-color-accent` checked background and border, the `$components-color-gray-100` disabled background, the explicit disabled border, and the `$components-color-gray-400` disabled thumb border. The `../utils/theme-variables` import is also dropped.

Changelog entries are added to both `base-styles` and `components`.

## Contribution

Authored by @mirka, with @ciampo credited in the props list. The record shows only the bot-generated bundle-size and performance reports and props-bot comments, with no design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
