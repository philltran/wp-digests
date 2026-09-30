# #76696: Fix: Stop stripping spaces inside inline HTML elements on paste

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @gregsullivan
- **Labels:** `[Type] Bug`, `[Feature] Paste`, `[Package] DOM`
- **Merged:** [`be2453e`](https://github.com/WordPress/gutenberg/commit/be2453efe366c1cce5807990d4bc831cd19460bd)
- **Discussion:** [#76696](https://github.com/WordPress/gutenberg/pull/76696) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Pasting content into the block editor no longer deletes whitespace-only inline elements outright. Previously `The following space:<em> </em>will be...` collapsed to `space:will`, merging words (a common artifact of Word-pasted content full of `<i> </i>`). `cleanNodeList` in `@wordpress/dom` now unwraps whitespace-only phrasing-content elements, keeping the space, and still removes truly empty ones and non-phrasing elements.

## Impact

- **Content editors / site owners:** Pasting from Word and similar sources no longer merges adjacent words when a whitespace-only `<em>`, `<i>`, `<span>`, etc. sits between them.
- **Plugin & theme developers:** No API change. Code that calls `pasteHandler`, `removeInvalidHTML`, or `cleanNodeList` (from `@wordpress/dom` / `@wordpress/blocks`) will see the space retained where it was previously dropped. Custom tests or snapshots that depended on the old stripping may need updating.
- **Known limitation:** Nested whitespace-only wrappers such as `<strong><em> </em></strong>` are only partially cleaned. The outer element is unwrapped but the inner `<em> </em>` is re-parented and not re-visited, so it survives. This is noted as a pre-existing limitation and is not fixed here.
- **Hosting / headless:** Not applicable.

## Technical details

The change is in `packages/dom/src/dom/clean-node-list.js`. In the branch handling `children && ! allowEmpty && isEmpty( node )`, the old code always called `remove( node )`. The new code checks `isPhrasingContent( node ) && node.hasChildNodes()` and calls `unwrap( node )` in that case, otherwise `remove( node )` as before.

```js
// Before
if ( children && ! allowEmpty && isEmpty( node ) ) {
	remove( node );
	return;
}

// After
if ( children && ! allowEmpty && isEmpty( node ) ) {
	if ( isPhrasingContent( node ) && node.hasChildNodes() ) {
		unwrap( node );
	} else {
		remove( node );
	}
	return;
}
```

The `hasChildNodes()` check distinguishes whitespace-only elements (which have a text child) from truly empty ones like `<em></em>`, which are still removed. Non-phrasing elements such as `<figure> </figure>` are also still removed.

Tests added:
- Unit tests in `packages/dom/src/test/dom.js` cover whitespace preservation, empty inline removal, and empty block-level removal.
- Integration tests in `test/integration/blocks-raw-handling.test.js` cover `The<em> </em>quick<span> </span>brown` yielding `The quick brown`, and `a<em></em>b` yielding `ab`.

A `@wordpress/dom` CHANGELOG entry was added under Unreleased.

## Contribution

The PR resolves long-standing issue #50898. In review, @Mamaduka approved the fix and flagged the nested-element limitation, offering a sample unit test. @mcsf deferred to @ellatrix and pointed to an unresolved question from the related PR #74754 about what happens if neither removal nor unwrapping is done. @gregsullivan discussed two alternatives: recursively cleaning child nodes before unwrapping, or leaving whitespace-only elements untouched. He argued that unwrapping best matches user intent for Word-style pastes. He also noted that elements like `<s> </s>` or `<u> </u>` visibly affect a space, so preserving those could be debated. A commenter later reported the problem still appearing in the editor while the front end looked correct.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
