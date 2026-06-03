# #78015: Paste: keep `<img>` inside `<a>` when pasting plain-text HTML

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Feature] Paste`, `[Package] DOM`, `[Package] Blocks`
- **Merged:** [`1b8f5f8`](https://github.com/WordPress/gutenberg/commit/1b8f5f875219cfa07461448423c9996f8a7db124)
- **Discussion:** [#78015](https://github.com/WordPress/gutenberg/pull/78015) · 4 comments · 0 reactions

## Summary

Pasting linked images — HTML of the form `<a href="…"><img src="…"></a>` — was silently dropping the `<img>` element during Gutenberg's paste sanitization. The culprit was the phrasing-content schema inside `removeInvalidHTML` (`@wordpress/dom`), which did not list `img` as a permitted child of `a`. Because `<a>` follows the HTML spec's "transparent" content model, this was overly strict: the fix adds `img` to the allowed children for `a` in that schema. The regression closed a six-year-old issue first reported with Flickr embed codes.

## Impact

**Content editors**
- Pasting embed codes that wrap an image in a link (e.g., Flickr's HTML embed snippet) now preserves the `<img>` instead of silently discarding it. No manual repair needed.

**Plugin & theme developers**
- No action required. No public API was changed or deprecated.
- Custom code that calls `removeInvalidHTML` from `@wordpress/dom` will inherit the corrected schema automatically.

**No breaking changes.**

## Technical details

The schema change lives in `@wordpress/dom` (confirmed by the `+10 B` delta to `build/scripts/dom/index.min.js`). The phrasing-content schema used by `removeInvalidHTML` maps each HTML element to its set of allowed children; previously `a` had no `img` entry, so the sanitizer stripped any `<img>` nested inside `<a>` on both the plain-text and rich-HTML paste paths.

The PR adds `img` as a valid child of `a` in that schema. No other elements or rules were modified.

Two new unit tests were added in `packages/blocks/src/api/raw-handling/test/paste-handler.js`:

```js
// Plain-text paste path (HTML: '')
const result = pasteHandler({
  HTML: '',
  plainText: '<a href="https://example.com/"><img src="https://example.com/img.png" alt="x"/></a>',
  mode: 'INLINE',
  tagName: 'p',
});
// Before fix: <img> was stripped → only the bare <a> survived
// After fix:
// '<a href="https://example.com/"><img src="https://example.com/img.png" alt="x"></a>'
```

A parallel test covers the rich-HTML paste path (`HTML` populated). Both assert the full `<a><img></a>` structure is retained after sanitization.

## Contribution

The underlying bug (#17942) was filed by @mobilene (Jim Grey) in 2019. @Mamaduka opened and landed the fix; the merged commit is `1b8f5f8`. Co-authors listed in the merge commit include @ellatrix, @youknowriad, @rachelmcr, @slywy, and unlinked contributors @Galois44, @mikeselander, and @gwwar. Discussion on the PR was minimal — @mobilene left a single comment expressing appreciation that the six-year-old issue was finally resolved. No alternative approaches were discussed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
