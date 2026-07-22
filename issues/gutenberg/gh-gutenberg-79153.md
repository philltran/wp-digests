# #79153: Theme: forward ThemeProvider cornerRadius preset to :root for root providers

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Theme`
- **Merged:** [`e0d24f5`](https://github.com/WordPress/gutenberg/commit/e0d24f531f60a043767007890b9fe3f77a3cbdff)
- **Discussion:** [#79153](https://github.com/WordPress/gutenberg/pull/79153) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `ThemeProvider` component in the WordPress Design System now forwards its `cornerRadius` preset to the `:root` (document `<html>`) element when instantiated with `isRoot={true}`. This aligns `cornerRadius` behavior with the existing `color` and `cursor` props, ensuring that border-radius design tokens cascade to PHP-rendered admin UI and other non-React contexts outside the React app root.

## Impact

- **Plugin & theme developers using the Design System:** If you render a root `ThemeProvider` with `isRoot={true}` and a `cornerRadius` preset, the `--wpds-border-radius-*` custom properties will now apply to `<html>` instead of just the provider's scoping `<div>`.
- **Consumers of `design-tokens.css`:** The stylesheet is now strictly required for `cornerRadius` presets to take effect on `:root`, as the values are resolved via static CSS selectors rather than inline runtime styles.
- **No breaking changes or deprecations.** Existing non-root or nested `ThemeProvider` instances continue to scope their `cornerRadius` presets locally without modification.

## Technical details

The diff modifies `packages/theme/terrazzo.config.ts` to append a `:root:has([data-wpds-root-provider="true"][data-wpds-corner-radius="..."])` selector to each of the four corner-radius modes (`none`, `subtle`, `moderate`, `pronounced`). Upon build, `packages/theme/src/prebuilt/css/design-tokens.css` is regenerated to include these selectors, allowing the `--wpds-border-radius-*` tokens to cascade to the document root when the root provider's attributes match.

Before:
```css
[data-wpds-corner-radius='none'] {
  --wpds-border-radius-xs: 0;
  /* ... */
}
```

After:
```css
[data-wpds-corner-radius='none'],
:root:has( [data-wpds-root-provider='true'][data-wpds-corner-radius='none'] ) {
  --wpds-border-radius-xs: 0;
  /* ... */
}
```

The `ThemeProvider` component itself does not change its runtime logic; it relies on the prebuilt CSS to handle the forwarding. This matches the existing pattern for `cursor` and aligns with how `color` handles root-level token distribution.

## Contribution

Opened by @ciampo as a follow-up to #78816, the PR was merged after a brief review where @aduth questioned the implementation divergence from `color`/`cursor` and the necessity of `design-tokens.css`. @ciampo clarified that the static CSS approach was intentional to avoid runtime computation for preset values, and that root forwarding was needed to maintain consistency across mixed React/PHP environments. The change was approved and merged without further iteration.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
