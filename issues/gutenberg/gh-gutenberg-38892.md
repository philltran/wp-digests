# #38892: Multi-selection: allow partial block selection

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Enhancement`, `[Feature] Writing Flow`, `[Feature] Block Multi Selection`
- **Merged:** [`721e3c4`](https://github.com/WordPress/gutenberg/commit/721e3c4d3c1854ca839b0dd0713e8756060aaacd)
- **Discussion:** [#38892](https://github.com/WordPress/gutenberg/pull/38892) · 38 comments · 14 reactions

## Summary

The block editor now supports partial cross-block selection: users can drag, shift+click, or shift+arrow to select text that spans block boundaries without selecting entire blocks. `Backspace`, `Delete` (forward), and direct typing over such a selection are all handled — content is merged or removed using each block type's existing `merge` function. When two blocks are not mergeable, the editor falls back to selecting the full blocks. This resolves long-standing issue #27481 and works across all browsers including Firefox.

## Impact

**Plugin & theme / block developers**
- The internal `use-multi-selection.js` file and its exported `setContentEditableWrapper()` helper and `useMultiSelection()` hook have been **deleted** from `packages/block-editor/src/components/block-list/use-block-props/`. Any non-upstream code importing these paths will break.
- The internal `use-scroll-into-view.js` hook (`useScrollIntoView`) has also been deleted and removed from `useBlockProps`.
- The `delete-multi-selection` keyboard shortcut's behaviour has changed: the old `BlockTools` handler that called `removeBlocks(clientIds)` on all selected blocks has been removed. Deletion now routes through the RichText/writing-flow layer. The shortcut description was updated from `"Remove multiple selected blocks."` to `"Delete selection."`
- The `merge` function declared in a block type's registration is now exercised during partial cross-block operations. Blocks that do not declare `merge` will fall back to whole-block selection rather than partial merge/delete. Review your custom block's `merge` implementation if users will edit mixed-block content.
- No changes to public `@wordpress/block-editor` JS API surface, `block.json` schema, REST endpoints, or PHP.

**Site owners / editors**
- Improved editing UX: natural mouse and keyboard selection across block boundaries behaves more like a standard rich-text editor.

**No action required** for hosting/platform teams or headless consumers.

## Technical details

**Deleted files**
- `packages/block-editor/src/components/block-list/use-block-props/use-multi-selection.js` — contained the per-block `mouseleave`-triggered approach that toggled `contentEditable` on the wrapping element to allow cross-block native selection, then called `multiSelect()` / `selectBlock()` on `selectionchange`. Fully removed.
- `packages/block-editor/src/components/block-list/use-block-props/use-scroll-into-view.js` — `useScrollIntoView()` hook using `dom-scroll-into-view`. Fully removed.

**`use-block-props/index.js`**
- Both `useScrollIntoView` and `useMultiSelection` are no longer composed into the ref merger returned by `useBlockProps`.

**`use-focus-first-element.js`**
- Import of `setContentEditableWrapper` removed.
- `isMultiSelecting()` check moved out of the selector (where it would have prevented the hook registering) and into the `useEffect` body, so the effect exits early when multi-selecting:
```js
// Before
if ( isMultiSelecting() || isNavigationMode() ) { return; }
// ...
setContentEditableWrapper( ref.current, false );
placeCaretAtHorizontalEdge( target, isReverse );

// After
if ( ! isBlockSelected( clientId ) || isMultiSelecting() ) { return; }
// ...
placeCaretAtHorizontalEdge( target, isReverse );
```

**`use-focus-handler.js`**
- Added an early-return guard so that when the block node's nearest `[contenteditable="true"]` ancestor indicates the whole editor is editable (the new cross-block selection mode), focus events do not trigger `selectBlock()`:
```js
if ( node.parentElement.closest( '[contenteditable="true"]' ) ) {
    return;
}
```

**`block-tools/index.js`**
- The `isMatch( 'core/block-editor/delete-multi-selection', event )` branch that called `removeBlocks( clientIds )` when `clientIds.length > 1` has been **removed**. Deletion now falls through to the RichText/writing-flow layer which uses each block type's `merge` function.

**`block-list/style.scss`**
- The selection outline (`::after` pseudo-element with `box-shadow`) is now applied to `.is-multi-selected:not([contenteditable])` — blocks in the interior of a cross-block selection that are not themselves the active editable element — rather than all `.is-multi-selected` blocks indiscriminately.
- Added `.is-highlighted ~ .is-multi-selected` selector for sibling-based outline propagation.
- Hard-coded `$border-width` replaced with `--wp-admin-border-width-focus` CSS custom property in `box-shadow` values.

**`block-list-appender/index.js`**
- `contentEditable={false}` added to the appender element so that arrow keys at the end of the last block collapse into the block rather than entering the appender when the editor root is `contenteditable`.

**`rich-text/index.js`** (diff truncated)
- `useFirefox` import added, indicating Firefox-specific handling was introduced to support cross-block native selection in Gecko.

## Contribution

Opened and driven by @ellatrix, fixing #27481 (tracked since at least #3629). During review @mcsf surfaced consistent editor crashes when undoing after a cross-block `Enter`; @ellatrix confirmed `Enter` was not yet implemented and had disabled native `Enter` handling as a guard, with two forward paths discussed (merge-then-split, or partial-delete-then-insert-paragraph). @mtias raised a design concern about the multi-select outline visibility and suggested it should only appear during block-level operations rather than during partial-text selection; he also floated an optional shift+pointer modifier to force whole-block selection, which was deferred. Keyboard selection (shift+arrow) was added by @ellatrix mid-review before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
