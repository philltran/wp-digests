# #80595: Base Styles: Remove accent color parameter from input-control

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Components`, `[Package] Block editor`, `[Package] Base styles`
- **Merged:** [`a98aa06`](https://github.com/WordPress/gutenberg/commit/a98aa0650c15a96ed40c5914364a862f0bf30c52)
- **Discussion:** [#80595](https://github.com/WordPress/gutenberg/pull/80595) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `input-control` and `input-style__focus` SCSS mixins in `@wordpress/base-styles` no longer accept an `$accent-color` parameter. Focus styling now defaults internally to `var(--wp-admin-theme-color)`, and all internal call sites across the block editor and components packages have been updated to remove redundant arguments. This change eliminates duplication and aligns focus ring behavior with the retirement of the experimental `--wp-components-color-accent` token.

## Impact

- **Plugin & theme developers using Gutenberg base styles:** Breaking change. The `$accent-color` parameter has been removed from `input-control` and `input-style__focus`. You must omit the argument in custom SCSS; focus styling will now always use `var(--wp-admin-theme-color)`.
- **Block editor & component consumers:** Invalid form states in `validated-form-controls` now override `--wp-admin-theme-color` instead of `--wp-components-color-accent` to render alert borders. No code changes required for standard block usage.
- **No action required** for end users or sites not compiling custom SCSS against the base-styles package.

## Technical details

In `packages/base-styles/_mixins.scss`, the signatures for `input-style__focus` and `input-control` were changed from accepting an optional `$accent-color` parameter to parameterless. The mixin body now hardcodes `border-color: var(--wp-admin-theme-color)` and `box-shadow: 0 0 0 0.5px var(--wp-admin-theme-color)`. All internal call sites were updated to remove the argument:

```scss
// Before
@include input-control( $components-color-accent );

// After
@include input-control;
```

In `packages/components/src/validated-form-controls/style.scss`, invalid state overrides now target `--wp-admin-theme-color` instead of `--wp-components-color-accent` to ensure the red alert color applies correctly to the newly parameterless mixins.

## Contribution

The change was implemented as a straightforward cleanup to remove a redundant parameter that was originally a workaround for experimental CSS tokens. Review focused on verifying that all component call sites were updated and that the default admin theme color correctly covers existing use cases, with no alternative approaches debated or rejected.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
