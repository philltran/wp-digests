# #78779: Fix: Delete key on empty heading transforms following paragraph into heading

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @coderGtm
- **Labels:** `[Type] Bug`, `[Feature] Writing Flow`, `[Package] Block editor`
- **Merged:** [`07d89d5`](https://github.com/WordPress/gutenberg/commit/07d89d593e79ba4ce1b71635dfd327c829ca1526)
- **Discussion:** [#78779](https://github.com/WordPress/gutenberg/pull/78779) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Pressing Delete (forward delete) in an empty Heading block no longer converts the following paragraph into a heading. The `mergeBlocks` store action now treats an empty text block of a different type as having nothing to contribute, so it removes that block and leaves the next block's type and content intact, with focus moved to it.

## Impact

**Site owners / content editors**
- Forward-deleting an empty heading now removes it and leaves the following paragraph as a paragraph. Previously the paragraph silently became a heading.

**Plugin & theme developers**
- No API changes. Custom blocks that define a `merge` function and have `content`-role attributes will now be removed, rather than merged into, when they are empty and followed by a block of a different name.
- Custom blocks without a `merge` function (and container blocks with no content attributes, such as Columns) are unaffected.

**Action required:** none. If you maintain e2e tests that depend on merge behavior between an empty non-default text block and a differently-typed next block, re-check them.

## Technical details

The change is in `mergeBlocks` in `packages/block-editor/src/store/actions.js`. Before, only `isUnmodifiedDefaultBlock( blockA )` short-circuited to removing `blockA`. An empty heading is not the default block, and the following paragraph had content, so execution reached the merge path, which calls `switchToBlockType( blockB, blockA.name )` and turned the paragraph into a heading.

The condition is now:

```js
if (
	isUnmodifiedDefaultBlock( blockA ) ||
	( !! blockAType.merge &&
		blockA.name !== blockB.name &&
		isUnmodifiedBlock( blockA, 'content' ) )
) {
	// remove blockA, focus blockB
}
```

The extra clauses are guards:
- `blockAType.merge` keeps container blocks out, since a Columns block has no content attributes and would always count as empty.
- `blockA.name !== blockB.name` limits the new path to different-type merges. This was added after review feedback about possible list block regressions and e2e failures.

Backspace was never affected, because Backspace on an empty heading with no preceding block goes through `switchToDefaultOrRemove()`.

Two e2e tests were added to `test/e2e/specs/editor/various/splitting-merging.spec.js`: forward delete on an empty heading before a paragraph with content (the paragraph survives, with the caret at its start), and before an empty paragraph. The PR description mentions a unit test, but the provided diff contains only the e2e specs.

## Contribution

Opened by @coderGtm to close #78767, with GitHub Copilot used for exploration, implementation, and tests. @jasmussen confirmed the fix but deferred to writing-flow experts, naming @ellatrix. @youknowriad flagged e2e failures that suggested regressions in the list block. The guard was then narrowed with the `blockA.name !== blockB.name` check and the `merge` requirement, and the tests were updated.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
