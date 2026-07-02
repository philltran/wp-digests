# #79722: IconButton: Use length zero for inline padding

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] UI`
- **Merged:** [`64ddefc`](https://github.com/WordPress/gutenberg/commit/64ddefc06e20be9fd720db18bf4c7d733456969d)
- **Discussion:** [#79722](https://github.com/WordPress/gutenberg/pull/79722) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Updates the `--wp-ui-button-padding-inline` CSS custom property on `IconButton` from a unitless `0` to an explicit length `0px`. This follow-up to #79627 ensures the padding token remains a valid length when consumed by shared button CSS, specifically inside `Button`'s `--wp-ui-button-min-width` `calc()` expression.

## Impact

- **Plugin & theme developers**: No visual or behavioral change; no code modifications required.
- **Gutenberg/UI package maintainers**: The CSS token is now explicitly typed as a length, preventing potential `calc()` parsing issues if the custom property is consumed elsewhere in shared UI styles without its current override.
- **Site owners & frontend consumers**: No action required. The change maintains internal CSS validity without altering layout or interaction.

## Technical details

The diff modifies `packages/ui/src/icon-button/style.module.css` to enforce a unit type on the inline padding variable:

```css
/* Before */
--wp-ui-button-padding-inline: 0;

/* After */
/* stylelint-disable-next-line length-zero-no-unit -- This custom property is reused in Button's min-width calc(), where a unitless zero is invalid. */
--wp-ui-button-padding-inline: 0px;
```

The variable resides within the `.icon-button` CSS composition layer and scopes a shared token that `Button` references during length calculations. A targeted `stylelint-disable-next-line` comment was added to exempt this specific declaration from the `length-zero-no-unit` rule while keeping broader linting strict.

## Contribution

Opened by @ciampo as a targeted follow-up to #79627, focusing strictly on CSS token validity and lint configuration. Reviewed and approved by @aduth during a brief discussion that scoped further stylelint improvements to separate work streams. The change was merged on commit `64ddefc` with minimal scope, including an updated changelog entry in `packages/ui/CHANGELOG.md`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
