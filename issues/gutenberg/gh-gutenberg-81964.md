# #81964: List View: Don’t focus the last Table cell on keyboard activation

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shail-mehta
- **Labels:** `[Type] Bug`, `[Block] Table`, `[Package] Block editor`, `Backported to WP Core`
- **Merged:** [`02a7c0e`](https://github.com/WordPress/gutenberg/commit/02a7c0e91c1bcc46b15925c9661377e0dafab310)
- **Discussion:** [#81964](https://github.com/WordPress/gutenberg/pull/81964) · 12 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Pressing Enter or Space on a block in the List View now moves editor focus to the start of the block's first field (`initialPosition = 0`) instead of the end of its last field (`-1`). This partially reverts the behavior from #76797 and fixes the Table block, where `-1` focused the last cell and could scroll the canvas to the bottom of a long table. Mouse clicks on List View items are unchanged and keep focus in the list.

## Impact

**Editor users (keyboard/AT)**
- Activating a Table from List View via keyboard now lands the caret in the first cell rather than the last.
- Activating a Paragraph from List View via keyboard puts the caret at the start of the block again, so pressing Enter splits before the text and inserts a new empty block *before* it (the #76797 behavior inserted it after).

**Plugin & theme developers**
- No API, hook, or block.json changes. Custom blocks that rely on the `initialPosition` handling in `selectBlock` will see `0` on keyboard activation rather than `-1`.
- E2E tests that assert the caret is at the end of a block after List View keyboard activation will need updating.

**Other**
- Labeled `Backported to WP Core`, so this is expected to reach WordPress core's copy of the block editor.
- No action required for most sites.

## Technical details

The change is a single-value edit in `packages/block-editor/src/components/list-view/block.jsx`, in the `selectEditorBlock` callback of `ListViewBlock`:

```diff
 const isKeyboardActivation = event?.detail === 0;
-selectBlock( event, clientId, isKeyboardActivation ? -1 : null );
+selectBlock( event, clientId, isKeyboardActivation ? 0 : null );
```

- Keyboard activation is still detected via `event.detail === 0`.
- `0` is the default initial position (start of the first field). `null` for mouse clicks still avoids transferring focus to the canvas.
- In `test/e2e/specs/editor/various/list-view.spec.js`, the existing caret test is renamed to "…at the start of the block…" and its expected block order is inverted (empty paragraph first, then First paragraph, then Second paragraph).
- A new e2e test, `should focus the first cell of a Table when activating from List View`, types `X` after activation and expects it in the first cell (`XR1C1`).
- A `packages/block-editor/CHANGELOG.md` entry was added under `ListView`.

## Contribution

The PR initially tried to fix the Table case in place, but @t-hamano and @Mamaduka argued for a full revert of #76797: focus targets vary by block (e.g. an image with a caption focuses the caption, without one focuses the block; Pullquote with and without a citation lands in different fields), so an end-of-block caret can't be predicted reliably. @Mamaduka also noted the original change looked more like a quality-of-life tweak than a bug fix. @shail-mehta first pushed a full revert, then @Mamaduka suggested a narrower partial revert that keeps the mouse-click behavior (`null`) and the e2e coverage, which is what merged.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
