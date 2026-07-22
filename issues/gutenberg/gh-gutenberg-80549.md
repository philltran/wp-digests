# #80549: Rich text: read the contentEditable attribute in ownsSelection to avoid forced layout

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Performance`, `[Package] Rich text`
- **Merged:** [`2d7bb27`](https://github.com/WordPress/gutenberg/commit/2d7bb2789f1a4e8f5fd9d2bddb3e27ea77969fec)
- **Discussion:** [#80549](https://github.com/WordPress/gutenberg/pull/80549) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This pull request optimizes the `ownsSelection` function in the Rich Text package by reading the `contentEditable` DOM attribute instead of the `isContentEditable` property. The change eliminates a forced style and layout recalculation that was triggered on every keystroke, addressing a typing performance regression introduced by the `editableRoot` implementation. By avoiding the layout thrashing, block editors with hundreds of blocks will see significantly smoother typing performance.

## Impact

- **Site owners & editors**: Improved typing responsiveness in the block editor, especially in posts with many blocks.
- **Plugin & theme developers**: No action required. The `ownsSelection` function is internal to the Rich Text package, and the behavioral contract remains identical.
- **Hosting & platform teams**: No action required.
- **Headless & REST consumers**: No impact.

## Technical details

The diff modifies `packages/rich-text/src/owns-selection.js`, replacing two reads of `isContentEditable` with direct attribute checks against the string `'true'`.

Before:
```js
! activeElement.isContentEditable ||
! element.isContentEditable ||
```

After:
```js
activeElement.contentEditable !== 'true' ||
element.contentEditable !== 'true' ||
```

The `isContentEditable` property triggers a synchronous style/layout recalculation to determine the effective state. Reading the reflected `contentEditable` attribute is a cheap DOM read that does not force layout. This substitution is safe because the checks only execute when the active element differs from the editable itself, and both the editing host (via `useEditableRoot`) and the editable element explicitly set `contenteditable="true"`, meaning inheritance is never relied upon in this code path.

## Contribution

Opened by @ellatrix as a performance fix following a regression from `editableRoot` (#79105). The PR was co-authored by @Mamaduka. During review, an alternative approach (#80548) that reordered the same checks to keep the read off the hot path was considered, but the author chose to remove the cost at the source by switching to the attribute read instead. The change was merged with minimal discussion, focusing on the measurable performance gain and behavioral equivalence.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
