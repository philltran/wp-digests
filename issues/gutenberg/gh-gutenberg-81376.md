# #81376: Editor: Speed up the hierarchical term selector for large taxonomies

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Editor`
- **Merged:** [`1e6382d`](https://github.com/WordPress/gutenberg/commit/1e6382df014f90cd6d590d0a4ab7ec6e487a66b7)
- **Discussion:** [#81376](https://github.com/WordPress/gutenberg/pull/81376) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The hierarchical term selector in the post editor (the Categories panel and any other hierarchical taxonomy) is optimized for sites with thousands of terms. The PR reports that with 5,000 categories, ticking one checkbox blocked the main thread for about 157ms. The fix builds the term tree in a single pass, replaces a comparator sort with a partition, and memoizes each checkbox so toggling one term no longer re-renders the whole list.

## Impact

- **Site owners / editors:** Selecting and filtering terms should feel faster on sites with very large hierarchical taxonomies. No visible UI or behavior change is intended.
- **Plugin & theme developers:** No new public API or hook. `buildTermsTree` (in `packages/editor/src/utils/terms.js`) now mutates the `children` property of the objects passed in instead of returning fresh clones. If you call it directly with objects you reuse elsewhere, be aware of this (see technical details).
- **Hosting / headless / REST consumers:** No impact.
- No action required for most projects.

## Technical details

All changes are React-side, in `hierarchical-term-selector.js` and `utils/terms.js`. A content-visibility CSS approach was dropped from this PR.

**`buildTermsTree`**
- Before: grouped terms by parent, then recursively cloned each term with `{ ...term, children }` (`fillWithChildren`).
- After: one pass builds a `termsById` map (created with `Object.create( null )`) and resets `term.children = []`. A second pass pushes each term into its parent's `children`, or into the root array when `` `${ term.parent }` === '0' ``. Terms whose parent is missing are dropped, as they were unreachable before too.
- The `flatTermsWithParentAndChildren` objects are mutated and reused rather than cloned, so any `children` on the input is discarded. The diff does not show how that array is created upstream of this hunk, so whether callers' original objects are affected depends on code outside the diff.

**`sortBySelected`**
- Before: `Array.sort` with a comparator that called `treeHasSelection` on both terms, re-walking subtrees on each comparison, using `terms.indexOf`.
- After: selection is a `Set`, `treeHasSelection` uses `children?.some(...)`, and top-level terms are partitioned into `selected` and `unselected`, then concatenated. Relative order within each group is preserved.

**`getFilterMatcher`**
- `normalizeTextString( filterValue )` is computed once instead of per visited term.

**Rendering**
- `renderTerms` is replaced by a recursive `TermRow` component and a `memo`-wrapped `TermCheckbox` that takes primitive props (`id`, `name`, `checked`, `onToggle`).
- `selectedTerms` is a memoized `Set` from `terms`, and `onToggleTerm` is a stable callback via `useEvent` from `@wordpress/compose`. It parses the id and adds or removes it from `terms`, then calls `onUpdateTerms`, which dispatches `editPost( { [ taxonomy.rest_base ]: termIds } )`.
- The old `onChange` handler is removed, and `onUpdateTerms` is moved above the early return so it can be used by the hook-based handler.

The bundle size check reported `build/scripts/editor/index.min.js` growing by 114 B.

## Contribution

The PR was initially broader, including CSS `content-visibility` and search debouncing. After review from @tyxla, who suggested splitting it, @Mamaduka extracted debouncing into a separate PR, then later dropped the CSS work too to be handled separately, leaving only the React-side optimizations. The author notes the change was assisted by Claude.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
