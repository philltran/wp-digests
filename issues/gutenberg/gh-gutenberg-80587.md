# #80587: Base styles: Allow overriding focus ring color in outset-ring__focus

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Base styles`
- **Merged:** [`60795eb`](https://github.com/WordPress/gutenberg/commit/60795ebd7ed1ce3c24f667ebb848529156ff0899)
- **Discussion:** [#80587](https://github.com/WordPress/gutenberg/pull/80587) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `outset-ring__focus` mixin in the `@wordpress/base-styles` package now exposes the focus ring color via a `--focus-color` CSS custom property, defaulting to `--wpds-color-stroke-focus`. This allows developers to override the focus ring color contextually without redefining the entire mixin or relying on `outline-color`. It simplifies state-based styling for form controls and aligns with the design system's token architecture.

## Impact

- **Theme & plugin developers**: Can now override focus ring colors for specific states (e.g., `:invalid`, `:hover`) by setting `--focus-color` on the element or a parent, without duplicating the mixin.
- **Base styles consumers**: No breaking changes; existing code using `@include mixins.outset-ring__focus()` continues to work identically.
- **No action required** for existing implementations.

## Technical details

The change modifies `packages/base-styles/_mixins.scss`. The `@mixin outset-ring__focus()` previously set the outline color directly using `wpds.var("--wpds-color-stroke-focus")`. It now declares `--focus-color: #{ wpds.var("--wpds-color-stroke-focus") };` and applies `var(--focus-color)` to the `outline` property. This shifts the color from a static Sass interpolation to a CSS custom property, enabling runtime/contextual overrides.

```scss
/* Before */
@mixin outset-ring__focus() {
  outline: wpds.var("--wpds-border-width-focus") solid wpds.var("--wpds-color-stroke-focus");
  outline-offset: wpds.var("--wpds-border-width-focus");
}

/* After */
@mixin outset-ring__focus() {
  --focus-color: #{ wpds.var("--wpds-color-stroke-focus") };
  outline: wpds.var("--wpds-border-width-focus") solid var(--focus-color);
  outline-offset: wpds.var("--wpds-border-width-focus");
}
```

## Contribution

Extracted from #80417, the PR was authored by @mirka and reviewed with co-authorship from @aduth. During review, @aduth proposed using optional Sass mixin arguments or overriding `--wpds-color-stroke-focus` directly. @mirka clarified that CSS custom properties better support state-based overrides and noted a project lint rule blocks direct token overrides. The team aligned on the custom property approach, with @mirka noting the pattern will be extended to `@wordpress/ui` focus utilities in a follow-up.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
