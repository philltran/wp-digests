# #80600: Theme: Omit color properties when neither provided nor inherited

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aduth
- **Labels:** `[Type] Bug`, `Backported to WP Core`, `[Package] Theme`
- **Merged:** [`c6b1373`](https://github.com/WordPress/gutenberg/commit/c6b1373629fe58e7c6211e16b7c3371574db0624)
- **Discussion:** [#80600](https://github.com/WordPress/gutenberg/pull/80600) · 8 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a regression in `@wordpress/theme` where `ThemeProvider` forcibly injected default color CSS custom properties even when no local or inherited seed values existed. This broke the CSS inheritance chain for the admin color scheme, causing editor notice action buttons to ignore the user's selected profile color scheme. The fix ensures color tokens are only emitted when explicitly provided or inherited from an ancestor provider, falling back to prebuilt design-tokens or external CSS otherwise.

## Impact

- **Plugin & theme developers**: If you use `ThemeProvider` from `@wordpress/theme` to scope colors in React trees or portaled content, color CSS custom properties will no longer be injected when the provider receives no `color` prop and has no ancestor provider. Rely on the prebuilt design-tokens stylesheet or explicit prop passing instead.
- **Block editor users**: Editor notice/action buttons now correctly respect the user's admin color scheme.
- **No action required** for most sites, but developers who relied on the previous fallback-to-default behavior for color tokens should verify their `ThemeProvider` usage in custom React components or portaled UI.

## Technical details

The change modifies `packages/theme/src/use-theme-provider-styles.ts`. The hook now calculates a `hasColor` boolean by checking if `color.primary`/`background` are defined locally or if `inheritedSettings.color` contains them. If `hasColor` is false, `primary` and `background` are set to `undefined`, and the `colorStyles` memo returns an empty object `{}`. This prevents `ThemeProvider` from emitting `--wp-admin-theme-color` and semantic tokens like `--wpds-color-background-interactive-brand-strong` in total-default scenarios. The behavior now aligns with how `cursor` and `cornerRadius` are handled. Tests in `use-theme-provider-styles.test.tsx` and `theme-provider.test.tsx` were updated to verify that color tokens are omitted by default but re-emitted when inherited, which is critical for portaled subtrees that need to re-apply ancestor color values.

## Contribution

The fix was opened by @aduth after @mirka flagged the regression in a related PR. @habiburdev verified the behavior against the testing instructions, and @t-hamano coordinated the backport to the `wp/7.1` branch after resolving a `CHANGELOG.md` merge conflict. The implementation was refined through manual review to simplify the logic and collapse test cases, with the author noting AI-assisted research but final manual adjustments to the diff and comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
