# #81443: Calendar: Support custom root roles

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] DataViews`, `[Package] UI`
- **Merged:** [`e7b9583`](https://github.com/WordPress/gutenberg/commit/e7b958379b76e948095f64d935e77064af91b0d4)
- **Discussion:** [#81443](https://github.com/WordPress/gutenberg/pull/81443) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`Calendar` and `RangeCalendar` in `@wordpress/ui` now accept a `role` prop for the root element, defaulting to `application` as before. The default accessible name now includes the currently displayed month (e.g. `Date calendar, May 2025`) and updates as the user navigates months. Consumers can override the role for a composition they have tested; docs warn that this can affect screen-reader keyboard navigation.

## Impact

- **Plugin & theme developers using `@wordpress/ui` Calendar/RangeCalendar:**
  - New optional `role` prop (`@default 'application'`). No action required if you keep the default.
  - The default accessible name changes from `Date calendar` / `Date range calendar` to `Date calendar, <Month Year>` / `Date range calendar, <Month Year>`. Tests that query `getByRole( 'application', { name: 'Date calendar' } )` with an exact string will fail and need a regex or updated string. This PR had to adjust such assertions in the DataViews date control tests.
  - A custom `aria-label` or `aria-labelledby` replaces the generated label entirely. Per the usage guidelines, when the role is `application`, include the current month in custom labels yourself.
  - Roles needing extra behavior (e.g. `dialog`) should be applied to a wrapper, not the calendar root.
- **Site owners / hosting / REST consumers:** Not affected.

## Technical details

The hard-coded `role: 'application'` was removed from `COMMON_PROPS` in `utils/constants.ts`. `Calendar` and `RangeCalendar` now destructure `role = 'application'` and `'aria-label'`, and pull the localized default `aria-label` out of `useLocalizationProps()` as `defaultAriaLabel`. `role` and `defaultAriaLabel` are passed to the custom `Root` component through `RootContext` (`utils/root-context.ts`, whose `RootContextValue` gains `role` and `defaultAriaLabel`), keeping the module-scope `Root` override stable so the calendar doesn't remount.

In `utils/components.tsx`, `Root` reads `months` and `labels` from `useDayPicker()`. If neither `aria-label` nor `aria-labelledby` is supplied, `defaultAriaLabel` exists, and `role === 'application'`, it builds the label via `sprintf( __( '%1$s, %2$s' ), defaultAriaLabel, labels.labelGrid( months[0].date ) )`, falling back to `defaultAriaLabel` if there is no month. It then renders with `props: { ...props, role, 'aria-label': ariaLabel }`. Consequently, with a non-`application` role and no explicit label, no generated label is applied.

`types.ts` adds `role?: ComponentProps<'div'>['role']` to `BaseProps` (with `role` still omitted from the inherited div props). Also updated: Storybook `SHARED_ARG_TYPES` (`role` as a text control), `components-manifest.yml`, a new "Root role" section in `usage-guidelines.mdx`, the `@wordpress/ui` CHANGELOG, and tests in `render-prop.test.tsx` covering month-aware label updates, custom role forwarding, and `aria-label`/`aria-labelledby` overrides.

## Contribution

This is a follow-up to a review discussion on PR #81337. The PR description states that OpenAI Codex was used for research, implementation, and verification. The only bot-reported CI issue in the discussion is an unrelated flaky Playwright interactivity e2e test that passed on retry.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
