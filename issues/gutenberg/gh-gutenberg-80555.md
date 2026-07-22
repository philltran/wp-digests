# #80555: Global Styles: Limit the inherited value treatment to the Gutenberg plugin

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `Gutenberg Plugin`, `[Package] Block editor`, `Global Styles`, `[Feature] Design Tools`, `Backported to WP Core`
- **Merged:** [`8d37a3e`](https://github.com/WordPress/gutenberg/commit/8d37a3ed61d697ea0503cbb33d41f8dee1003adf)
- **Discussion:** [#80555](https://github.com/WordPress/gutenberg/pull/80555) · 8 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The block editor’s Global Styles inheritance UI now only surfaces inherited values and local override indicators when the Gutenberg plugin is active. The feature originally shipped in WordPress Core 7.1 Beta 3, but was gated behind a new `ENABLE_GLOBAL_STYLES_INHERITANCE` constant to prevent it from reaching Core without further review. The underlying logic remains in the repository and can be re-enabled for Beta 4 by removing the flag.

## Impact

- **Plugin & theme developers**: No action required. Block inspector panels (Typography, Color, Border, Dimensions, Background, Filters) will no longer display dotted underlines, placeholder inheritance values, or reset affordances in Core builds.
- **Gutenberg plugin users**: Unaffected; the inheritance UI continues to work as intended.
- **Platform & hosting teams**: No configuration changes needed. The feature is simply disabled in Core builds via a runtime flag.

## Technical details

The diff introduces `ENABLE_GLOBAL_STYLES_INHERITANCE` in `packages/block-editor/src/components/global-styles/inheritance/index.js`, which evaluates `globalThis.IS_GUTENBERG_PLUGIN` at runtime. This constant replaces the hardcoded `true` default for the `showInheritanceLabelIndicators` prop across six panel components (`background-panel.js`, `border-panel.js`, `color-panel.js`, `dimensions-panel.js`, `filters-panel.js`, `typography-panel.js`). In `inherited-value-context.js`, the `useResolvedStyle` hook now short-circuits to return `{ value: undefined, sources: undefined }` when the flag is false, preventing the cascade merge from populating inherited values. The typography panel also adjusts its `shouldSyncLinkColor` logic to compare against `inheritedValue` only when the flag is enabled, falling back to direct `value` comparison in Core to avoid unintended link color tracking.

## Contribution

Opened and merged by @t-hamano with co-authors @ramonjd and @andrewserong. The change was proposed after @youknowriad questioned why the team didn’t simply revert the original backport to `wp/7.1`. @t-hamano explained that the original PR had already been backported, making a targeted gate simpler than a full revert while preserving the full implementation for potential Beta 4 inclusion. @youknowriad approved the approach, and the PR was cherry-picked to both `wp/7.1` and `release/23.6`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
