# #80978: RichText: don't apply input transform when there's no onReplace

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jsnajdr
- **Labels:** `[Type] Bug`, `[Package] Block editor`
- **Merged:** [`d1182f6`](https://github.com/WordPress/gutenberg/commit/d1182f6d8a2bf0eb85e8f4ee1a123f2b9065e8b4)
- **Discussion:** [#80978](https://github.com/WordPress/gutenberg/pull/80978) · 4 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Fixes a crash in the `RichText` component that occurs when an input transform (such as typing `---` to create a separator) is triggered but the field lacks an `onReplace` handler. The change guards the input transformation logic behind an `onReplace` check, preventing the editor from attempting to replace a block that cannot be swapped. This ensures blocks with locked removal or fields that explicitly omit `onReplace` remain stable when users type trigger sequences.

## Impact

- **Block & plugin developers**: No action required. The fix is internal to the `RichText` component and prevents crashes in edge cases where `onReplace` is intentionally omitted or undefined.
- **Site owners & editors**: Editors using locked blocks (e.g., `core/paragraph` with `lock.remove: true`) or blocks without inner blocks (e.g., `core/pullquote`) will no longer experience editor crashes when typing input-trigger sequences like `---`.
- **Hosting & platform teams**: No configuration or migration needed.

## Technical details

The diff modifies `packages/block-editor/src/components/rich-text/event-listeners/input-rules.js`. Previously, the `input-rules` event listener unconditionally fetched `getBlockTransforms( 'from' )` and called `onReplace( transformation.transform() )` when a match was found, causing a crash if `onReplace` was undefined. The fix wraps the entire transform lookup and execution in an `if ( onReplace )` guard.

**Before:**
```js
const transforms = getBlockTransforms( 'from' ).filter(
	( transform ) => transform.type === 'input'
);
const transformation = findTransform( transforms, ( item ) => {
	return item.regExp.test( value.text );
} );

if ( transformation ) {
	onReplace( transformation.transform() );
	registry
		.dispatch( blockEditorStore )
		.__unstableMarkAutomaticChange();
	return;
}
```

**After:**
```js
if ( onReplace ) {
	const transforms = getBlockTransforms( 'from' ).filter(
		( transform ) => transform.type === 'input'
	);
	const transformation = findTransform( transforms, ( item ) =>
		item.regExp.test( value.text )
	);

	if ( transformation ) {
		onReplace( transformation.transform() );
		registry
			.dispatch( blockEditorStore )
			.__unstableMarkAutomaticChange();
		return;
	}
}
```
A new e2e test in `test/e2e/specs/editor/blocks/separator.spec.js` verifies that typing `---` in a locked `core/paragraph` leaves the text as-is instead of crashing.

## Contribution

The discussion centered on whether a `noop` fallback would suffice instead of guarding on `onReplace`; the author clarified that passing `undefined` explicitly signals that replacement is unavailable, making a `noop` inappropriate since it would falsely indicate a successful block swap. The change was straightforward and merged without further iteration.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
