# #81212: Block Editor: Return false from isBlockSelected when there is no client ID

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Block editor`
- **Merged:** [`2cd1c4a`](https://github.com/WordPress/gutenberg/commit/2cd1c4a58be8394803f178288510affb7688ed53)
- **Discussion:** [#81212](https://github.com/WordPress/gutenberg/pull/81212) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `isBlockSelected` selector in the block editor store now explicitly returns `false` when called without a `clientId`. Previously, passing `undefined` triggered an `undefined === undefined` comparison that incorrectly evaluated to `true`, causing the editor to believe a block was selected when it wasn't. This fix prevents premature rendering of inspector panels and pattern editing controls when the selection state is empty.

## Impact

- **Block & plugin developers**: If your code calls `isBlockSelected( state, clientId )` with an optional or potentially `undefined` `clientId`, the return value will now correctly be `false` instead of `true` when the editor has no active selection.
- **Block editor users**: Resolves a UI glitch where the content and list inspectors would briefly render for blocks outside a section when nothing was selected.
- **No action required** for standard block development, but review any custom selection logic or store selectors that pass optional IDs to `isBlockSelected`.

## Technical details

The change modifies `isBlockSelected` in `packages/block-editor/src/store/selectors.js`. The original implementation compared `selectionStart.clientId === clientId`, which returned `true` when both sides were `undefined`. The diff adds a truthiness guard to prevent the false match:

```js
// Before
return selectionStart.clientId === clientId;

// After
return !! clientId && selectionStart.clientId === clientId;
```

A corresponding unit test was added to `packages/block-editor/src/store/test/selectors.js` to verify that `isBlockSelected( state, undefined )` returns `false` when the selection state is empty. All other call sites pass a concrete `clientId`, so their behavior remains unchanged.

## Contribution

Opened and merged by @Mamaduka with review from @talldan. The PR was a straightforward bug fix addressing a specific edge case in selection state evaluation. No alternative approaches were debated; the author simply added a guard clause to prevent the `undefined === undefined` match, and the change was approved quickly after a minor related fix was noted in the comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
