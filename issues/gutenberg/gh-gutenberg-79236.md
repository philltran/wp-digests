# #79236: File Block: Replace on-mount downloadButtonText effect with a default variation

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] File`
- **Merged:** [`b786c86`](https://github.com/WordPress/gutenberg/commit/b786c86922ad3beb499ad0df8f2cfeeb8c237fa0)
- **Discussion:** [#79236](https://github.com/WordPress/gutenberg/pull/79236) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The File block now initializes its default `downloadButtonText` attribute using a declarative `isDefault` block variation instead of an imperative `useEffect` on mount. This change eliminates a StrictMode-related bug where reloading a saved post incorrectly marked the block as dirty, and aligns the block's initialization pattern with modern WordPress block standards.

## Impact

- **Block & plugin developers:** No breaking changes. The internal initialization pattern shifted from `useEffect` to a `variations` export, which may affect custom File block forks or overrides that relied on the previous mount-side-effect behavior.
- **Theme & hosting teams:** No action required. The block's external behavior, attributes, and save output remain identical.
- **End users:** No visible changes; the download button still defaults to the localized "Download" text.

## Technical details

The diff removes the `useEffect` hook and the `__unstableMarkNextChangeAsNotPersistent` destructuring from `packages/block-library/src/file/edit.js`. Instead, `packages/block-library/src/file/variations.js` defines a new `default` variation with `isDefault: true` and sets `attributes.downloadButtonText` via `_x( 'Download', 'button label' )`. This variation is exported in `packages/block-library/src/file/index.js`. Additionally, `packages/block-library/src/file/transforms.js` was updated to explicitly pass `downloadButtonText` in all `createBlock` calls, ensuring transforms bypass the default variation and correctly set the localized string.

## Contribution

Opened and merged by @Mamaduka, with architectural guidance credited to @t-hamano. The PR moved quickly through review with minimal discussion, focusing on replacing an imperative effect with a declarative variation to resolve a StrictMode side-effect bug.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
