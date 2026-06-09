# #77992: Post Revisions: Upgrade `diff` from v4 to v8

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @manzoorwanijk
- **Labels:** `[Type] Task`, `[Package] Editor`, `[Package] Block editor`
- **Merged:** [`c25cea9`](https://github.com/WordPress/gutenberg/commit/c25cea9d60a61604df1ca126039d7364faad1c0a)
- **Discussion:** [#77992](https://github.com/WordPress/gutenberg/pull/77992) · 14 comments · 0 reactions

## Summary

Upgrades the `diff` library from `^4.0.2` to `^8.0.3` in `packages/editor` and `packages/block-editor`, resolving two regressions in the Post Revisions UI caused by v6+ behavior changes. The upgrade aligns internal package dependencies while preserving stable block-level matching and precise per-word inline diffs for content authors.

## Impact

["- **Plugin & theme developers**: No breaking API surface changes; `diff` remains an internal editor dependency. Developers relying on deep imports like `'diff/lib/diff/array'` may encounter stricter resolution behavior with v8's `exports` map if they maintain custom bundling setups, but core usage is updated to top-level imports.", '- **Editors & content authors**: Post Revisions preview behavior is unchanged; modified blocks pair cleanly without confusing dual inline diffs, and rich-text diffs retain per-word granularity inside links or attributes.', "- **No action required** for standard integration. Packages aligning to `packages/sync`'s v8 requirement benefit from unified dependency tree resolution."]

## Technical details

['**Dependency bump**: Bumped `"diff"` to `"^8.0.3"` in `packages/editor/package.json` and `packages/block-editor/package.json`. Added internal upgrade entries to both `CHANGELOG.md` files.', '**LCS tie-breaker mitigation** (`packages/editor/src/components/post-revisions-preview/block-diff.js`): v6+ introduced a "deletions before insertions" LCS tie-breaker that could select whitespace-only freeform pseudo-blocks as the anchor. Added `isWhitespaceRawBlock()` and updated `diffRawBlocks()` to filter these out (`blockName === null` with empty `innerHTML`) prior to `diffArrays()`, preventing mis-pairing in `pairSimilarBlocks`.', "**Placement heuristic update**: Extended `pairSimilarBlocks`'s crossing check to treat unchanged blocks (`status === undefined`) as current-revision anchors alongside unpaired additions. The updated `crossesCurrentContent` logic prevents modified blocks from incorrectly jumping before stable content during placement.", '**Rich-text diff precision**: Replaced `diffWords` with `diffWordsWithSpace` in `applyRichTextDiff()` and `applyDiffToBlock()` (and in `revision-fields-diff/index.js`). v8 stopped treating whitespace as a token; switching to `diffWordsWithSpace` preserves the per-word `<del>/<ins>` granularity.', "**Import modernization**: Migrated deep imports (`'diff/lib/diff/array'`, `'diff/lib/diff/character'`) to top-level `'diff'` in `preserve-client-ids.js` and `block-compare/index.js`. Removed a stale tree-shaking comment as v8's `sideEffects: false` sub-manifests make deep paths unnecessary.", '**Test adjustments**: Updated the swap test to assert the user-facing invariant (one unchanged, one modified pair) rather than a specific LCS pivot choice. Added canary tests for the whitespace pseudo-block filter and the `crossesCurrentContent` placement logic.']

## Contribution

Opened by @manzoorwanijk, merged at `c25cea9d` with review from @ciampo and @aduth. Initial reviews considered vendoring the v4 LCS algorithm or subclassing `diff`'s `Diff` class to override the tie-breaker. The author pivoted to filtering whitespace pseudo-blocks before matching and refining the placement heuristic in `pairSimilarBlocks`, which was accepted as a lighter, more robust approach that avoids maintaining an upstream algorithm port. @ellatrix reviewed for editor context and noted no backport to 7.0 would occur.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
