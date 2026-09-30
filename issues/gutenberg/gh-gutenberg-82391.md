# #82391: UI, DataViews: Give input and selection controls solid backgrounds

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] Base styles`, `[Package] DataViews`, `[Package] Theme`, `[Package] UI`
- **Merged:** [`cc790f0`](https://github.com/WordPress/gutenberg/commit/cc790f098655660affd6ab7ce48701896c9102bc)
- **Discussion:** [#82391](https://github.com/WordPress/gutenberg/pull/82391) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds three new semantic color tokens to `@wordpress/theme` (`--wpds-color-background-interactive-neutral`, `-active`, and `-disabled`) and uses them to give `@wordpress/ui` input fields and checkboxes, plus unselected DataViews multi-selection filter indicators, solid themed backgrounds. Previously these controls used the `neutral-weak` interactive tokens, which resolve to transparent (`#0000`) and so picked up whatever surface sat behind them. All three new tokens currently map to the same theme color, so the visible change is a stable solid fill across resting, hover and disabled states.

## Impact

**Plugin & theme developers using `@wordpress/ui` / DataViews**
- Inputs, checkboxes and unselected multi-selection filter indicators now render with an opaque background (`#fff` in the default theme, derived from the background seed under `ThemeProvider`). Custom CSS or visual tests that assumed a transparent fill behind these controls may see differences.
- Combobox chips keep their existing weak background family, and minimal `Select` triggers and neutral minimal/outline `Button`s remain transparent.

**Theme/token consumers**
- Three new public CSS custom properties are available: `--wpds-color-background-interactive-neutral`, `--wpds-color-background-interactive-neutral-active`, `--wpds-color-background-interactive-neutral-disabled`. `InteractiveBackgroundColor` in the token types gains a `'neutral'` member.
- Because the states are separate tokens, themes can later differentiate active/disabled fills without component changes.

**Compatibility**
- Component styles use a nested fallback to `--wpds-color-background-surface-neutral-strong`, so a newer bundle running under an older `ThemeProvider` (which doesn't emit the new tokens) still honors custom background theming.

No migration required.

## Technical details

**Token definition** (`packages/theme/tokens/color.json`): adds `background.interactive.neutral`, `neutral-active` and `neutral-disabled`, each with `$value` `{wpds-color.primitive.bg.surface3}`. In `src/prebuilt/ts/color-tokens.ts` the `bg-surface3` primitive now lists the three new tokens alongside `background-surface-neutral-strong`, which is why they track the background ramp. Generated outputs are updated: `prebuilt/css/design-tokens.css` (`#fff` defaults), `design-tokens.mjs`, `design-token-fallbacks.mjs`, and `base-styles/internal/_wpds-token-fallbacks.scss`. `docs/tokens.md` documents them.

**Contrast pairs** (`semantic-color-contrast-pairs.ts`): adds pairs for `background.interactive.neutral` and `neutral-active` against `foreground.interactive.neutral` and `neutral-weak`, plus `neutral-weak-active` against `foreground.interactive.neutral-active`.

**Component usage**
- `ui/.../checkbox/style.module.css`: resting uses `--wpds-color-background-interactive-neutral`; hover/`[data-highlighted]` adds `-active`; `[data-disabled]` uses `-disabled` (replacing `neutral-weak-disabled`).
- `ui/.../input-layout/style.module.css`: the resting background switches from `neutral-weak` to the new token (the diff is truncated here, so the active/disabled handling is not visible).
- `dataviews-filters/style.scss`: `.dataviews-filters__search-widget-listitem-multi-selection` and the checkbox mixin background use the new token.

```css
/* before */
background-color: var(--wpds-color-background-interactive-neutral-weak);
/* after */
background-color: var(--wpds-color-background-interactive-neutral, var(--wpds-color-background-surface-neutral-strong));
```

Each fallback line carries a `stylelint-disable-next-line plugin-wpds/no-token-fallback-values` comment. Tests: a new parity test in `build-plugin-parity.test.ts` checks that nested `var()` fallbacks are expanded identically by the PostCSS plugin and Lightning CSS (`var(--a, var(--b, #fff))`), and a `use-theme-provider-styles` test asserts the three new properties equal `--wpds-color-background-surface-neutral-strong` for a `#222222` background seed.

## Contribution

Review centered on why the token was needed and how many to add. @mirka questioned whether the transparent `#0000` background tokens were a bug and suggested a surface background token for inputs; @ciampo explained the transparency serves the minimal button's resting state. @mirka then argued these controls do have active and disabled states and that future affordance cues (absent cursor cues) would likely flow through `background-interactive` states, so the PR should not add a single token. @ciampo expanded it to resting, active and disabled tokens sharing one value, deferring any subtler or solid active fill to a follow-up. Per the description, the implementation was done with Codex and reviewed by the author.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
