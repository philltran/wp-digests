# #81145: Interface: Increase footer breadcrumb height to prevent focus ring clipping

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `Needs Design Feedback`, `[Package] Interface`, `[Package] Base styles`, `Backported to WP Core`
- **Merged:** [`9043ec1`](https://github.com/WordPress/gutenberg/commit/9043ec14b2064c6db5b604e4fdddc71bbf854e30)
- **Discussion:** [#81145](https://github.com/WordPress/gutenberg/pull/81145) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg interface footer breadcrumb height was increased from 24px to 32px to prevent focus rings from being clipped. This change addresses a visual regression caused by switching focus indicators from `box-shadow` to `outline`, which renders outside the element bounds. The adjustment ensures keyboard navigation indicators remain fully visible in the editor footer.

## Impact

- **Plugin & theme developers**: No action required. The change is scoped entirely to the Gutenberg editor interface and does not affect frontend themes, REST APIs, or block registration.
- **Site owners & editors**: Improved keyboard navigation visibility in the block editor footer. No configuration changes needed.
- **Hosting & platform teams**: No action required. The update ships as a standard Gutenberg package patch.

## Technical details

The unified diff modifies two SCSS files in the `@wordpress/interface` and `@wordpress/base-styles` packages to replace hardcoded `24px` values with the WordPress Design System token `--wpds-dimension-size-md` (32px). 

In `packages/interface/src/components/interface-skeleton/style.scss`, the footer height and container padding are updated:

```scss
// Before
height: $button-size-small;
padding-bottom: $button-size-small + $border-width;

// After
height: var(--wpds-dimension-size-md);
padding-bottom: calc(var(--wpds-dimension-size-md) + #{$border-width});
```

A related fix in `packages/base-styles/_mixins.scss` updates the `snackbar-container()` mixin to use `wpds.var("--wpds-dimension-size-md")` instead of a hardcoded `bottom: 24px`. The change relies on the `--wpds-dimension-size-md` token to maintain consistent spacing across interface components and prevents `outline`-based focus rings from being clipped by the footer container.

## Contribution

The PR was opened by @t-hamano and merged after review by @jasmussen and @mirka. During review, @jasmussen raised design concerns about intentionally compact footer dimensions and noted user feedback regarding breadcrumb visibility, suggesting alternative approaches like adjusting button sizes or using negative `outline-offset`. The author acknowledged the trade-off but emphasized that focus ring clipping is a recurring issue with the new `outline`-based focus style and referenced prior fixes. The change was subsequently backported to the `wp/7.1` branch to ensure the fix ships with the upcoming release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
