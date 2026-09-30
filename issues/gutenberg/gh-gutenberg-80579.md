# #80579: Editor: Merge adjacent revision inline diffs

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shail-mehta
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Package] Editor`, `[Feature] History`
- **Merged:** [`1b11803`](https://github.com/WordPress/gutenberg/commit/1b118032878bdec74818ad1e53822a2651f731ff)
- **Discussion:** [#80579](https://github.com/WordPress/gutenberg/pull/80579) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The editor's Visual History revision comparison previously emitted one `<del>`/`<ins>` pair per changed word when using `diffWordsWithSpace`, producing inconsistent and verbose markup for screen readers. This PR adds a `mergeTextDiffParts()` function that collapses adjacent removed/added tokens into a single `<del>` and a single `<ins>` element, so a run of consecutive word changes renders as one grouped pair instead of many. The change is part of the broader revision-diff consistency effort tracked in #77530.

## Impact

- **Screen reader / accessibility users:** Revision diffs in Visual History now produce fewer, grouped `<del>`/`<ins>` elements, reducing verbosity when multiple adjacent words change.
- **Plugin & theme developers:** No public API change. The revision-diff HTML structure changes (fewer sibling `<del>`/`<ins>` pairs; inline formatting like `<strong>` may now appear *inside* the `<del>`/`<ins>` rather than wrapping them). If you style or parse `.revision-diff-removed` / `.revision-diff-added` elements, verify your selectors still match.
- **Site owners / hosting:** No action required. No configuration, migration, or code changes needed.

## Technical details

A new function `mergeTextDiffParts( parts )` is added to `packages/editor/src/components/post-revisions-preview/block-diff.js`. It walks the array of `{ added?, removed?, value }` objects produced by `diffWordsWithSpace()` and collapses consecutive removed/added tokens (including intervening whitespace) into a single `{ removed: true, value }` and a single `{ added: true, value }` entry.

The call site in `applyRichTextDiff()` changes from:

```js
const textDiff = diffWordsWithSpace( previousText, currentText );
```

to:

```js
const textDiff = mergeTextDiffParts(
  diffWordsWithSpace( previousText, currentText )
);
```

Whitespace tokens between changed words are included in both the removed and added strings so the merged output preserves spacing. The function does not alter how non-changed (unchanged) parts are handled — they pass through as-is.

The test suite confirms the structural shift. For example, changing `"our site"` to `"the website"` inside a link previously produced:

```html
<del class="revision-diff-removed">our</del><ins class="revision-diff-added">the</ins> <del class="revision-diff-removed">site</del><ins class="revision-diff-added">website</ins>
```

and now produces:

```html
<del class="revision-diff-removed">our site</del><ins class="revision-diff-added">the website</ins>
```

A second test shows that inline formatting is now nested *inside* the merged elements: `<del>Hello <strong>world</strong></del><ins>Goodbye <strong>everyone</strong></ins>` instead of the prior pattern where `<strong>` wrapped individual `<del>`/`<ins>` pairs.

## Contribution

Opened by @shail-mehta as part of the #77530 revision-diff consistency effort, with @joedolson credited as co-author. The PR was merged with minimal discussion (5 comments, 0 reactions). The author disclosed use of AI tooling in drafting the contribution. No notable design debate or rejected alternatives appear in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
