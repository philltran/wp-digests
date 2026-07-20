# #79982: Components/Button: Don't use box-shadow for secondary buttons

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`
- **Merged:** [`eb24e81`](https://github.com/WordPress/gutenberg/commit/eb24e81eb05de53abb7238a9e6b0b7882b4bd490)
- **Discussion:** [#79982](https://github.com/WordPress/gutenberg/pull/79982) · 5 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg `Button` component now renders its secondary variant using a standard CSS `border` instead of an inset `box-shadow`. This change aligns the component with modern CSS practices, eliminates legacy high-contrast mode workarounds, and ensures consistent box sizing across all button variants by reserving a transparent 1px border on the base element.

## Impact

- **Plugin & theme developers:** Custom CSS that targets `.components-button.is-secondary` with `box-shadow` to style secondary buttons will no longer apply. Switch to `border-color` or `border`.
- **Block editor users:** No direct action required; visual appearance remains nearly identical, but button sizing is now consistent across variants.
- **High contrast mode users:** The previous `@media (forced-colors: active)` outline fallbacks are removed, as native borders now render correctly in forced-colors environments.
- **No migration required** for standard block or theme usage, but any custom overrides relying on the old `box-shadow` inset pattern will need adjustment.

## Technical details

The diff modifies `packages/components/src/button/style.scss` to replace the `box-shadow: inset 0 0 0 $border-width ...` pattern with `border-color` for the `.is-secondary` variant. A base rule `border: $border-width solid transparent;` is added to `.components-button` to reserve space for the border and prevent layout shifts when variants apply a visible border. The `@media (forced-colors: active)` media queries that previously forced `outline: 1px solid ButtonBorder;` are removed. Icon-only buttons receive `margin-inline: -$border-width` on their `svg` and `.dashicon` children to maintain a square footprint. Related overrides in `packages/block-editor/src/components/block-styles/style.scss`, `packages/block-editor/src/components/button-block-appender/content.scss`, `packages/block-library/src/navigation/editor.scss`, and `packages/editor/src/components/post-featured-image/style.scss` are updated to use `border-color` instead of `box-shadow`.

## Contribution

Opened and merged by @t-hamano, with review contributions from @mirka. The change was driven by a prior discussion noting that the `box-shadow` workaround was no longer necessary and was causing issues in high-contrast mode. The author iterated on icon button sizing and removed multiple legacy style overrides across the block editor and components packages before merging.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
