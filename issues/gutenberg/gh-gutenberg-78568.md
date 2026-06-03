# #78568: Editor / Block Editor: Lazy-fetch user pattern categories

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Performance`, `[Package] Editor`, `[Package] Block editor`
- **Merged:** [`082c85a`](https://github.com/WordPress/gutenberg/commit/082c85a87c3b444a087115bded2eccbe0967c5dd)
- **Discussion:** [#78568](https://github.com/WordPress/gutenberg/pull/78568) · 2 comments · 0 reactions

## Summary

The block editor now lazy-fetches user pattern categories instead of requesting them on every editor boot. Previously, `useBlockEditorSettings` eagerly called `getUserPatternCategories()` inside its initial `useSelect`, triggering a REST request immediately upon initialization. By exposing a selector function via a private Symbol key and deferring invocation until the Patterns tab mounts, unnecessary network requests during editor startup are eliminated.

## Impact

- **Plugin & theme developers**: No breaking changes to public APIs or editor settings shapes; the internal store shape remains experimental (`__experimental*`).
- **Block editor consumers / tooling**: No migration required. Boot-time REST traffic for `/wp/v2/wp_pattern_category` is deferred until the inserter's Patterns tab is actively opened.
- **Performance & platform teams**: Reduces initial payload and time-to-interactive for editor mounts, particularly on installations with large pattern catalogs or slow network connections.
- **Action required**: None. The change is transparent to end-users and standard editor integrations.

## Technical details

- **File changes**: `packages/block-editor/src/store/private-keys.js` exports a new symbol: `userPatternCategoriesSelectKey = Symbol( 'userPatternCategoriesSelect' )`.
- **State shift in `useBlockEditorSettings`**: The eager data setting `__experimentalUserPatternCategories` is replaced by a function closure bound to the new symbol. This closure simply invokes the core store resolver when called.
- **Consumer update (Patterns tab)**: The inserter's state module now reads the symbol key from settings and conditionally invokes it inside its own `useSelect` hook, ensuring the REST resolver fires only on demand.
- **Before/After pattern**:
  ```javascript
  // Before (eager boot-time fetch):
  const userPatternCategories = settings.__experimentalUserPatternCategories;

  // After (lazy resolver triggered on Patterns tab mount):
  const selectFn = settings[ userPatternCategoriesSelectKey ];
  return {
    ...
    userPatternCategories: selectFn ? selectFn( select ) : [],
  };
  ```
- **Data flow**: `GET /wp-json/wp/v2/wp_pattern_category` is no longer attached to editor initialization. The resolver now fires synchronously only when the Patterns tab component registers its `useSelect` dependency.

## Contribution

Opened and merged by @ellatrix under commit `082c85a`. The PR was authored as a targeted performance optimization following an existing lazy-fetch pattern used for reusable blocks (`reusableBlocksSelectKey`). No significant design debate was recorded; the change focuses strictly on deferring network I/O until consumer demand occurs, with a negligible bundle size delta (+101 B in `block-editor/index.min.js`).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
