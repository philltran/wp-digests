# #83312: Block Editor: Consolidate search ranking into one shared module

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`, `[Package] Edit Site`
- **Merged:** [`2a933f2`](https://github.com/WordPress/gutenberg/commit/2a933f2202a44d16b24f45b599cad54afbf63822)
- **Discussion:** [#83312](https://github.com/WordPress/gutenberg/pull/83312) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Five duplicated copies of the block/pattern search-ranking algorithm (inserter, both site-editor pattern lists, Query/Template Part pattern modals, template swap modal) are replaced by a single shared module at `packages/block-editor/src/utils/search-ranking.ts`. The new algorithm ranks each field independently in explicit tiers (exact title match → title prefix → word-in-title starts-with → substring → all-terms-appear-somewhere) instead of concatenating all fields into one string and scoring the result. A core-block preference that previously added points to the rank is now a separate `tiebreak` comparator, so it can no longer pull a weak match above a strong one.

## Impact

- **Plugin & theme developers using `@wordpress/block-editor` private APIs:** `extractWords` and `getNormalizedSearchTerms` are removed from the private API surface in `private-apis.js`; `SEARCH_RANK` is added in their place. The `searchItems` function now accepts a `SearchOptions` object (`fields`, `filter`, `tiebreak`) instead of the old `config` object with per-field getter functions. Any code importing those private symbols will break.
- **Site owners / end users:** Inserter and pattern search results are reordered. Searching `loop` now ranks "Query Loop" above a block that merely mentions "loop" in its description (previously both scored 10 and order was arbitrary). No configuration or migration required.
- **No action required** for developers who do not import the affected private APIs or override inserter search behavior.

## Technical details

The new module `packages/block-editor/src/utils/search-ranking.ts` (TypeScript, ~446 lines) exports:

- **`SEARCH_RANK`** — a const enum object with tiers `NO_MATCH: 0`, `MATCHES: 1`, `CONTAINS: 2`, `WORD_STARTS_WITH: 3`, `STARTS_WITH: 4`, `EQUAL: 5`, modeled on `match-sorter`'s ranking enum.
- **`searchItems<T>(items, searchInput, options)`** — the shared ranking function. `options` is a `SearchOptions<T>` with `fields?: SearchField<T>[]`, `filter?: (item) => boolean`, and `tiebreak?: (a, b) => number`. Each `SearchField` has a `get` accessor and an optional `maxRank` cap so a keyword or description hit cannot outrank a title hit.
- **`normalizeString(input)`** — strips diacritics, leading slash, and lowercases (moved from the old file, same behavior).
- A `WeakMap`-based per-item cache for normalized field values, avoiding re-tokenization on every keystroke.

The old `packages/block-editor/src/components/inserter/search-items.js` is reduced to a thin `searchBlockItems` wrapper that builds the field array (title uncapped, `name` and `keywords` capped at `WORD_STARTS_WITH`, category/collection/description capped at `CONTAINS`) and passes a `getCorePriority` tiebreak. The old `getItemSearchRank`, `extractWords`, `getNormalizedSearchTerms`, and the inline `searchItems` implementation are deleted.

Before (old scoring, all fields concatenated):
```js
const terms = [name, title, description, ...keywords, category, collection].join(' ');
if (unmatchedTerms.length === 0) rank += 10;
if (rank !== 0 && name.startsWith('core/')) rank += isCoreBlockVariation ? 1 : 2;
```

After (per-field tiered ranking with tiebreak):
```ts
return searchItems(items, searchInput, {
  fields,
  tiebreak: (a, b) => getCorePriority(b) - getCorePriority(a),
});
```

`private-apis.js` now exports `SEARCH_RANK` and `normalizeString` from `./utils/search-ranking` instead of `extractWords`, `getNormalizedSearchTerms`, and `searchItems` from `./components/inserter/search-items`.

Bundle impact is negligible: +44 B total (block-editor +409 B, block-library −83 B, edit-site −198 B, editor −84 B).

## Contribution

Authored by @Mamaduka with co-author @ntsekouras (Nik), who provided testing. The PR notes AI assistance (Claude). The discussion is minimal — three comments, no design debate or rejected alternatives visible in the record. The motivation stated in the PR body is purely maintenance: the same algorithm had to be patched five times across the inserter, both site-editor pattern lists, the Query/Template Part pattern modals, and the template swap modal.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
