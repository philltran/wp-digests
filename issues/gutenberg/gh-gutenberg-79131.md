# #79131: Style Book: Fix crash when previewing variations for blocks without examples

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @SainathPoojary
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Global Styles`, `[Feature] Style Book`
- **Merged:** [`a9d38c6`](https://github.com/WordPress/gutenberg/commit/a9d38c6ba2760f0a3c9b9b1571ee4f3393834ada)
- **Discussion:** [#79131](https://github.com/WordPress/gutenberg/pull/79131) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Style Book in the Site Editor now safely handles blocks that lack standalone examples when previewing style variations. Previously, selecting a variation for a block like `core/column` triggered a `TypeError` crash because the code attempted to map over an undefined examples array. This fix adds a guard condition to prevent the crash and allows the UI to render a fallback state instead.

## Impact

- **Site editors & theme developers:** No action required. The crash is resolved, and the Style Book UI will gracefully show an empty or fallback preview for blocks without dedicated examples.
- **Plugin developers:** No impact. This change is isolated to the Site Editor's Style Book component.
- **Hosting/platform teams:** No configuration or migration needed.

## Technical details

The fix modifies `packages/editor/src/components/style-book/index.js` inside the `StyleBookPreview` component. The original condition `if ( blockVariation )` allowed execution to proceed even when `filteredExamples.examples` was `undefined`, causing `applyBlockVariationsToExamples` to fail on `.map()`. The guard was updated to check for a valid examples array before proceeding:

```diff
- if ( blockVariation ) {
+ if ( blockVariation && filteredExamples?.examples?.length ) {
```

This ensures the variation application logic only runs when a valid examples array exists, aligning with a reviewer suggestion to move the validation upstream to the `displayedExamples` resolution path.

## Contribution

Opened to resolve a crash reported in #79130, the PR initially added a guard directly inside `applyBlockVariationsToExamples`. During review, @t-hamano suggested moving the check upstream to `displayedExamples` to keep the variation logic cleaner and avoid unnecessary function calls. The author implemented the upstream check, and the PR was merged after a quick approval cycle. The discussion was brief, focusing solely on the optimal placement of the null check.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
