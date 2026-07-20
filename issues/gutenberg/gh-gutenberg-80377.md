# #80377: Autocompleters: Don't pre-encode mention search terms

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Backport to Gutenberg RC`, `Backported to WP Core`
- **Merged:** [`80f9a92`](https://github.com/WordPress/gutenberg/commit/80f9a9281c0ae382b5888c44da78355094daaa39)
- **Discussion:** [#80377](https://github.com/WordPress/gutenberg/pull/80377) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The block editor's user mention autocompleter (`@`) was double-encoding search terms, causing non-Latin characters (Georgian, Japanese, Cyrillic, emoji) to be stripped by the REST API's `sanitize_text_field()`. This resulted in an empty search query that incorrectly returned every user on the site instead of filtering by the typed name. The fix removes the manual `encodeURIComponent` call, allowing the internal `addQueryArgs` utility to handle encoding correctly.

## Impact

- **Plugin & theme developers / Block authors:** No action required. This is an internal editor bug fix with no outward-facing API changes.
- **Site owners / Editors:** Will see correct mention suggestions when typing non-Latin usernames in the block editor.
- **Hosting & platform teams:** No configuration or migration steps needed.
- **Breaking changes:** None. This restores expected autocompleter behavior.

## Technical details

The bug lived in `packages/editor/src/components/autocompleters/user.js` and `packages/editor/src/components/collab-sidebar/note-mention-completer.tsx`. Both completers passed `search: encodeURIComponent( filterValue )` to the `getUsers` selector. Since `getUsers` internally uses `addQueryArgs` to build the REST request, the term was double-encoded. The REST endpoint's `search` parameter runs through `sanitize_text_field()`, which strips percent-encoded sequences, leaving an empty string that matches all users. The diff removes `encodeURIComponent` from both files, passing `filterValue` directly. A new e2e test in `test/e2e/specs/editor/various/autocomplete-and-mentions.spec.js` verifies non-Latin filtering.

Before/after pattern:
```js
// Before
search: encodeURIComponent( filterValue ),
// After
search: filterValue,
```

## Contribution

Opened by @Mamaduka and merged as `80f9a92`. The fix aligns the mention completers with the existing `links` completer pattern by removing redundant client-side encoding. @noruzzamans and @t-hamano contributed to testing and co-authorship. The change was backported to the `wp/7.1` branch for inclusion in the next release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
