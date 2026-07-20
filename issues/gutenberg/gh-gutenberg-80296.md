# #80296: Notes: Add placeholders to the RichText fields

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Notes`
- **Merged:** [`bf57851`](https://github.com/WordPress/gutenberg/commit/bf57851401a37e0ba206342dfc150d9de8ef1b9c)
- **Discussion:** [#80296](https://github.com/WordPress/gutenberg/pull/80296) · 12 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg editor's Block Notes feature now renders explicit placeholder text in the RichText input fields for creating new notes and replying to existing ones. Previously, these fields relied solely on visually hidden labels, leaving the input area blank until focused. The change improves discoverability by displaying "Add a note or @ mention" for new notes and "Reply or @ mention" for replies, aligning the collaboration UI with standard input patterns.

## Impact

- **Editor users & content teams:** Improved UX for the Block Notes sidebar with clearer input guidance.
- **Plugin & theme developers:** No direct impact. The `RichText` component's `labels` prop now accepts a `placeholder` key, but this is an internal editor component change.
- **Platform & hosting teams:** No action required.
- **Backward compatibility:** No breaking changes or deprecations. Existing code using the Block Notes feature will simply render the new placeholders.

## Technical details

The diff modifies three files in `packages/editor/src/components/collab-sidebar/`. In `add-note.js`, the `labels` object passed to the `RichText` component is expanded to include a `placeholder` key. In `note-form.js`, the `RichText` instance now reads this value via `placeholder={ labels?.placeholder }`. In `note-thread.js`, the reply form receives a `placeholder` key in its `labels` prop. The change also removes an e2e test (`test/e2e/specs/editor/various/block-notes.spec.js`) that previously asserted no `aria-placeholder` attribute or `[data-rich-text-placeholder]` element existed, as placeholders are now intentionally rendered.

**Before:**
```js
labels={ { input: __( 'New note' ) } }
```

**After:**
```js
labels={ {
	input: __( 'New note' ),
	placeholder: __( 'Add a note or @ mention' ),
} }
```

## Contribution

Opened by @Mamaduka and merged with co-authors @t-hamano, @adamsilverstein, @jasmussen, @noruzzamans, @fcoveram, and @joedolson. The discussion centered on terminology consistency, with reviewers initially suggesting "Comment or @ mention" or a unified placeholder. @Mamaduka clarified that the feature deliberately uses "note" and "reply" terminology, and the team settled on distinct but parallel phrasing to maintain clarity between the two states.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
