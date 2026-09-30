# #63484: Writing flow: select next block on Enter key

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Enhancement`, `[Feature] Writing Flow`, `[Package] Block library`, `[Package] Block editor`
- **Merged:** [`0f61d4a`](https://github.com/WordPress/gutenberg/commit/0f61d4aa2e311028b13bb596d8f2e100b26c2b92)
- **Discussion:** [#63484](https://github.com/WordPress/gutenberg/pull/63484) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

In the block editor, pressing Enter with a block selected now moves selection to the next block whenever a default block can't be inserted or the block can't be split (e.g. a locked template). "Next" means the first child block, else the next sibling, else the next sibling of the nearest ancestor that has one. To make this work, the Enter handling in rich text and writing flow was reordered so the select-next fallback runs after the split check but before the rich text line-break fallback. This fixes #54984, where Enter did nothing useful in locked templates.

## Impact

**Editors / site owners**
- In locked templates or contexts where no default block can be inserted, Enter on a selected block (e.g. Post Title in the Site Editor) now advances selection to the next block instead of doing nothing.
- Focus is set on the newly selected block without moving the caret to its start or end.

**Plugin & block developers**
- Blocks that use `RichText` without the `splitting` support and without `disableLineBreaks` (e.g. Verse) keep their line-break-on-Enter behaviour; they are not turned into select-next.
- The deprecated `multiline` RichText path was rewritten internally (see below). Custom blocks using `multiline` should behave the same, but are worth a quick smoke test of Enter/split/merge.
- Code that relied on Enter on a selected block wrapper calling `insertAfterBlock` from the old wrapper key handler should re-check: that logic now lives in writing flow's `useInput`.
- A small regression from this change was reported afterward and fixed in #82424.

No migration or code change is required for typical sites.

## Technical details

Based on the (truncated) diff:

- **`writing-flow/use-input.js`**: The Enter handler no longer returns early when the block lacks `splitting` support. It now branches:
  1. If not fully selected, a block can be inserted (default block or the block's own type), and the block supports `splitting` or `event.__deprecatedOnSplit` is set: `__unstableSplitSelection()`.
  2. Otherwise, only if the active element is the block wrapper (`data-block === clientId`) or a contenteditable within the block (`getBlockClientId( activeElement ) === clientId`): it checks the parent's `getBlockListSettings( rootClientId ).defaultBlock` (falling back to `getDefaultBlockName()`) and, if insertable, calls `insertAfterBlock( clientId )`. Otherwise it descends into an empty container by inserting its default block, or selects the next block via `selectBlock`. The visible diff cuts off before the rest of this branch. This uses newly imported `getNextBlockClientId`, `getBlockOrder`, `getBlockEditingMode`, `getBlockListSettings`, `insertBlock` and `selectBlock`.
  - The `event.shiftKey || __unstableIsFullySelected()` early return became `event.shiftKey` only.
- **`use-selected-block-event-handlers.js`**: Removes the `insertAfterBlock` call on Enter from the wrapper's key handler. Enter now only resets zoom-out (and calls `preventDefault` in that case); Backspace/Delete still call `removeBlock`.
- **`use-focus-first-element.js`**: `initialPosition === true` now just calls `ref.current.focus()`, so `selectBlock( id, true )` sets focus without caret repositioning.
- **`block-list/block.js`**: Adds `supportsSplitting` (via `hasBlockSupport( blockName, 'splitting', false )`) to `PrivateBlockContext`.
- **`rich-text/index.js`**: `RichTextWrapper` reads `supportsSplitting` from `PrivateBlockContext` and passes it to the enter listener via props. `RichTextWrapper` is now exported as a `forwardRef` component.
- **`rich-text/event-listeners/enter.js`**: The line-break fallback moves out of the window-level delegated listener into an owned capture-phase `keydown` listener on the element. It handles Shift+Enter, `onSplitAtEnd`, sets `__deprecatedOnSplit` when `onReplace && onSplit`, and otherwise (when `! supportsSplitting && ! disableLineBreaks && ! event.defaultPrevented`) inserts `\n`, or splits via `onSplitAtDoubleLineEnd` on a triple Enter. The remaining window listener (`onDefaultKeyDown`) only calls `preventDefault()` as the last interception point.
- **`rich-text/multiline.js`**: The per-line `onKeyDown` React prop is replaced by a new `Line` component and `useEnterRef`, which attaches a native capture-phase `keydown` listener so it runs before the generic enter listener (which skips defaultPrevented events). Split and merge logic is otherwise carried over.

```js
// Enter on a selected block, previously (wrapper handler)
insertAfterBlock( clientId );
// now handled in writing flow: split, else insert default block,
// else descend / select next block
```

## Contribution

Authored by @ellatrix, who noted it grew into a larger refactor because the select-next handler had to run right after the split check while the rich text line-break fallback had to run before it. @alexstine raised an accessibility concern about selection/focus behaviour and asked for clearer test steps; @jasmussen supplied a Site Editor Pages-template walkthrough. @ellatrix clarified the intent was to avoid moving focus when calling `selectBlock()`, and added the `true` initial position to keep focus without caret movement. @Mamaduka later found a small regression and opened #82424 with a fix.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
