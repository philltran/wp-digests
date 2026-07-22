# #80509: Fix crashes when manipulating locked blocks

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jsnajdr
- **Labels:** `[Type] Bug`, `[Package] Block library`
- **Merged:** [`1a92e4d`](https://github.com/WordPress/gutenberg/commit/1a92e4da2366bfb5097576a07e1185b2cd7bb8be)
- **Discussion:** [#80509](https://github.com/WordPress/gutenberg/pull/80509) · 7 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a JavaScript crash in the block editor when users attempt write operations (like pressing Backspace or Enter) on blocks locked via `lock` or `templateLock` attributes. Previously, the editor passed `undefined` for write callbacks to locked blocks, but several core block edit components unconditionally wrapped these callbacks, causing runtime errors when the underlying function was missing. The patch adds conditional guards across multiple block edit files to safely pass `undefined` when write operations are disabled, preserving the expected editor behavior.

## Impact

- **Block & theme developers:** No breaking changes or migration required. If your custom blocks wrap `onReplace`, `onRemove`, `insertBlocksAfter`, or `__unstableOnSplitAtEnd`/`__unstableOnSplitAtDoubleLineEnd` callbacks, ensure they check for `undefined` before execution to prevent crashes in locked contexts.
- **Site owners & editors:** Resolves editor crashes when interacting with locked blocks (e.g., pressing Backspace on a locked heading or Enter inside a locked template group).
- **No action required** for existing sites; this is a transparent bug fix that ships with the next Gutenberg/WordPress release.

## Technical details

The crash occurs because `BlockListBlock` passes `undefined` for write-like props when a block is locked, while block edit components unconditionally wrap them:

```js
// Before (heading/edit.js, post-title/edit.js, etc.)
onRemove={ () => onReplace( [] ) }
__unstableOnSplitAtEnd={ () => insertBlocksAfter( createBlock( getDefaultBlockName() ) ) }
```

The diff replaces these with conditional ternaries that preserve `undefined` when the underlying callback is unavailable:

```js
// After
onRemove={ onReplace ? () => onReplace( [] ) : undefined }
__unstableOnSplitAtEnd={ insertBlocksAfter ? () => insertBlocksAfter( createBlock( getDefaultBlockName() ) ) : undefined }
```

Changes span `packages/block-library/src/` across 15 edit components (`code`, `embed`, `heading`, `image`, `more`, `navigation-link`, `post-terms`, `post-title`, `preformatted`, `pullquote`, `read-more`, `site-tagline`, `site-title`, `verse`, `caption`). The `RichText` component explicitly relies on `undefined` callbacks to skip split/merge logic in `packages/block-editor/src/components/rich-text/event-listeners/enter.js`, which is why noop functions were rejected during review. Two new Playwright e2e tests in `test/e2e/specs/editor/blocks/heading.spec.js` and `post-title.spec.js` verify the fix.

## Contribution

Opened and merged by @jsnajdr, with co-authors @Mamaduka and @tyxla. The issue was surfaced via WordPress.com JS error monitoring. During review, @jsnajdr and @Mamaduka discussed whether to pass noop functions instead of `undefined`, but rejected the approach because `RichText`'s event listeners branch on the presence of `onSplitAtEnd` and `onSplitAtDoubleLineEnd`, and truthy callbacks would alter expected behavior. The PR was merged after adding targeted e2e tests and auditing core blocks for the same pattern.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
