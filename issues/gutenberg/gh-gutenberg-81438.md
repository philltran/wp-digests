# #81438: UI: Align Calendar styling with WPDS tokens

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`c3a157c`](https://github.com/WordPress/gutenberg/commit/c3a157cb6af72165a69e2b536a70e228e704da8c)
- **Discussion:** [#81438](https://github.com/WordPress/gutenberg/pull/81438) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The `Calendar` and `RangeCalendar` components in `@wordpress/ui` now style day states with the neutral interactive WPDS color tokens instead of mixing in brand tokens and `color-mix()` calculations. Unselected days follow the neutral minimal `Button` state model, range middles use the neutral weak active background, and range previews use the neutral interactive stroke token. Two public CSS custom properties are removed as a result.

## Impact

**Plugin & theme developers using `@wordpress/ui` Calendar / RangeCalendar**
- **Breaking:** `--wp-ui-calendar-range-middle-background-color` and `--wp-ui-calendar-preview-border-color` are removed. Any override of these properties will silently stop working.
- Replacements: range middles now read `--wpds-color-background-interactive-neutral-weak-active`, and previews read `--wpds-color-stroke-interactive-neutral`. Customize via those tokens (or via the DS theming layer) instead.
- Hover, focus and range colors shift visually from brand-tinted to neutral. Visual regression snapshots that include these components will need updating.

**Site owners / end users**
- Calendar day hover, focus and range fills look neutral rather than brand-colored. The selected range fill has more contrast. No action required.

**Everyone else:** No action required.

## Technical details

All changes are in `packages/ui/src/calendar/style.module.css`, plus a CHANGELOG entry under Breaking Changes and Bug Fixes.

- **Removed** the internal-but-public variables `--wp-ui-calendar-range-middle-background-color` and `--wp-ui-calendar-preview-border-color` from `.root`; both were `color-mix()` of `--wpds-color-background-interactive-brand-strong`.
- **`.day`:** now sets a base `color: var(--wpds-color-foreground-interactive-neutral)`. The hover/`:focus-visible` color changes from `--wpds-color-foreground-interactive-brand` to `--wpds-color-foreground-interactive-neutral-active`.
- **`.day-button::before`:** gets `background-color: var(--wpds-color-background-interactive-neutral-weak)`. Disabled days use `...-neutral-weak-disabled`. `:hover:not(:disabled)` and `:focus-visible` use `...-neutral-weak-active`.
- **`.outside`:** text color changes from `--wpds-color-foreground-content-neutral-weak` to `--wpds-color-foreground-interactive-neutral-weak`.
- **`.range-middle .day-button::before`:** uses `--wpds-color-background-interactive-neutral-weak-active`.
- **`.preview svg`:** color is now `--wpds-color-stroke-interactive-neutral` (still `inherit` in forced-colors).
- **New rule:** `.preview.selected:not(.range-middle) svg { display: none; }` so the dashed preview outline no longer renders over selected range endpoints.

```css
/* Before */
.range-middle .day-button::before {
  background-color: var(--wp-ui-calendar-range-middle-background-color);
}
/* After */
.range-middle .day-button::before {
  background-color: var(--wpds-color-background-interactive-neutral-weak-active);
}
```

The PR notes a known limitation: the hover/focus/active neutral tokens currently show little visual difference from each other. This is to be addressed separately, and the calendars will inherit the fix through token consumption.

## Contribution

This is a follow-up to #81337, prompted by review discussions there about not mixing neutral and brand tokens, and it parallels a similar discussion in #81088. The PR description says the implementation was done with OpenAI Codex. Review was brief: @jasmussen approved, and the change merged with @mirka credited among the interacting accounts.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
