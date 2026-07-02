# #79616: Navigation Link: Fix "[object Object]" in link preview for untitled entities

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @taipeicoder
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Navigation Link`
- **Merged:** [`57f50ed`](https://github.com/WordPress/gutenberg/commit/57f50ed6d4469785c26f8607bf482cfc525bedcd)
- **Discussion:** [#79616](https://github.com/WordPress/gutenberg/pull/79616) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a UI regression in the Navigation Link block where untitled pages or entities displayed `[object Object]` in the 'Link to' settings preview instead of a readable fallback. The `useLinkPreview` hook's title computation previously fell through to the raw title object when `rendered` was an empty string, causing React to coerce it into an unhelpful display string. This PR introduces a dedicated helper that explicitly checks for empty values and returns the standard `(no title)` placeholder.

## Impact

- **Block & Plugin Developers:** The `useLinkPreview` hook behavior is corrected for edge-case entity records; no public API changes, deprecations, or breaking alterations are introduced.
- **Site/Theme Admins:** No action required. The fix resolves a localized editor preview bug without affecting published frontend output or navigation markup.
- **Migration/Config:** None required.

## Technical details

The change is contained in `packages/block-library/src/navigation-link/shared/use-link-preview.js`. The previous inline title resolution:
```js
const title = entityRecord?.title?.rendered || entityRecord?.title || entityRecord?.name;
```
is replaced by a new internal helper `getEntityTitle( entityRecord )`. This helper explicitly handles three cases: returns string titles directly, unwraps `{ raw, rendered }` objects and falls back to `__( '(no title)' )` when `rendered` is empty, or defaults to `entityRecord.name`. The replacement ensures the truthy evaluation for the rich-fetch guard (`useRemoteUrlData( title ? null : url )`) remains unchanged while preventing the object from being coerced into `[object Object]` by the React renderer. Unit tests in `use-link-preview.test.js` verify the new fallback and confirm `mockUseRemoteUrlData` receives `null` when a valid title or placeholder is present.

## Contribution

Opened by `@taipeicoder` with co-authors `@ramonjd` and `@t-hamano`. The PR moved through standard block-library review cycles, with reviewers noting unrelated E2E failures but confirming the fix addressed a regression dating back several months. It was merged without requiring an RC backport, following typical Gutenberg merge practices.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
