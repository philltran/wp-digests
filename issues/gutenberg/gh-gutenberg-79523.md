# #79523: Wrap notices in ThemeProvider with 0 corner radius

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @CookieDarb
- **Labels:** `[Type] Bug`, `General Interface`, `[Package] Editor`, `[Package] Edit Post`, `[Package] Edit Widgets`, `Backported to WP Core`
- **Merged:** [`6b235aa`](https://github.com/WordPress/gutenberg/commit/6b235aa7f3f3ae285b1840123d1ac7004ae5204a)
- **Discussion:** [#79523](https://github.com/WordPress/gutenberg/pull/79523) · 19 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Pinned editor notices in the Post and Widgets editors now render with zero corner radius and flush stacking to eliminate visual mismatches against non-white or dark editor themes. The change wraps the `InlineNotices` component in a `ThemeProvider` with `cornerRadius="none"` and adjusts the associated SCSS to remove outer padding, internal gaps, and redundant borders.

## Impact

- **Plugin & theme developers:** No public API changes. However, custom scripts or styles targeting the meta-box resize container via `:scope > .notices-inline-notices-wrapper` will need to query `.editor-notices` instead.
- **Site owners & editors:** Visual improvement for notice stacking on dark or custom themes; no action required.
- **Hosting & platform teams:** No action required.

## Technical details

The diff wraps `InlineNotices` in `packages/editor/src/components/editor-interface/index.js` and `packages/edit-widgets/src/components/notices/index.js` with `<ThemeProvider cornerRadius="none">` imported from `@wordpress/theme`. Both packages now declare `@wordpress/theme` as a dependency in their `package.json` and `tsconfig.json`. SCSS in `packages/editor/src/components/editor-interface/style.scss` and `packages/edit-widgets/src/components/notices/style.scss` sets `gap: 0` on `.notices-inline-notices-wrapper` and `.components-notice-list`, and removes `border-top-width` and `border-inline-width` on `.components-notice` to collapse adjacent borders. In `packages/edit-post/src/components/layout/index.js`, the meta-box resize handler’s container query shifts from `':scope > .notices-inline-notices-wrapper'` to `'.editor-notices'` to accommodate the new wrapper structure.

## Contribution

Opened to address visual clipping on dark themes, the implementation evolved through design feedback that recommended using `ThemeProvider` over component-level CSS overrides to avoid conflicting with recent `@wordpress/components` updates. The author iterated on the approach after initial CSS-only attempts were flagged as potentially reverting prior alignment work. The fix was backported to the WordPress 7.1 Beta/RC cycle ahead of the code freeze and merged with multiple co-authors.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
