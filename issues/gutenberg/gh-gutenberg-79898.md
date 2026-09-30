# #79898: Prevent white screen crash during undo operations when Media Modal is open

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @sarthaknagoshe2002
- **Labels:** `[Type] Bug`, `[Package] Media Utils`
- **Merged:** [`87acb90`](https://github.com/WordPress/gutenberg/commit/87acb90b997ddf496eae5df19937cc92ebb981ee)
- **Discussion:** [#79898](https://github.com/WordPress/gutenberg/pull/79898) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Pressing Ctrl/Cmd+Z while the legacy Media Library modal (opened via `MediaUpload`) is showing no longer reaches the block editor's global undo shortcut. Previously the undo removed the underlying block, unmounted `MediaUpload` abruptly, and left an orphaned `.media-modal-backdrop` on `document.body` that blocked the UI (a "white screen"). The fix stops undo/redo keystrokes at the modal element and makes unmount close and tear down the Backbone frame properly.

## Impact

- **Site owners / editors:** Undo/redo shortcuts no longer delete the block behind the open Media Library modal, and the editor no longer gets stuck behind an orphaned backdrop.
- **Plugin & theme developers:** No API changes. Anything using `MediaUpload` from `@wordpress/media-utils` gets the behavior automatically. If you unmount a `MediaUpload` while its modal is open (e.g. via external state changes), the modal and backdrop are now cleaned up.
- **Custom shortcut handling:** Undo/redo key combos (`primary+z`, `primary+shift+z`, `primary+y`) pressed inside the modal no longer propagate to ancestors of the modal element. Custom listeners on `document` or `window` in bubble phase will not see them while the modal is focused.
- **Action required:** None.

## Technical details

Changes are in `packages/media-utils/src/components/media-upload/index.js` (legacy `MediaUpload` class component).

- **New `stopUndoRedoPropagation( event )` handler:** returns early unless `metaKey` or `ctrlKey` is set. It then checks `event.key?.toLowerCase()` for `z` or `y` (covering undo, redo, and the non-Apple `primary+y` alias without platform detection) and calls `event.stopPropagation()`. It does not call `preventDefault()`, so the event still reaches the focused field and native text-input undo keeps working inside the modal.
- **Listener registration:** in the frame's open handler, a capture-phase `keydown` listener is added to `this.frame.modal.el`. Since it is a capture listener on the modal element, propagation stops before the event bubbles to the document-level `useShortcut` listener.
- **Listener removal:** new `detachUndoRedoGuard()` removes the listener; it is called from `onClose()` and `componentWillUnmount()`.
- **Unmount teardown:** `componentWillUnmount()` previously only called `this.frame?.remove()`. It now calls `detachUndoRedoGuard()`, `this.frame?.close()`, `this.frame?.modal?.remove()`, then `this.frame?.remove()`, so Backbone removes the DOM overlay even on forced unmount. The PR description mentions only closing before removing; the explicit `modal.remove()` is additional in the diff.

```js
// before
componentWillUnmount() {
	this.frame?.remove();
}

// after
componentWillUnmount() {
	this.detachUndoRedoGuard();
	this.frame?.close();
	this.frame?.modal?.remove();
	this.frame?.remove();
}
```

Tests: two Playwright specs added to `test/e2e/specs/editor/various/undo.spec.js`. One presses `primary+z` with the modal open (and with focus on the close button) and asserts the image block survives. The other removes the image block via `core/block-editor` dispatch while the modal is open and asserts `.media-modal` and `.media-modal-backdrop` are gone. A CHANGELOG entry was added under Bug Fixes in `packages/media-utils`.

## Contribution

Closes issue #71755. Review feedback led to a follow-up commit and a rebase request from @ramonjd, who reported it testing well. @jsnajdr raised the broader design question of whether other global shortcuts (registered via `useShortcut` on `document`) should also be blocked inside the modal, since the problem is generic to any modal wanting to suppress global shortcuts; the record does not show that being resolved in this PR. The author also force-pushed early to remove accidentally committed files that triggered a CODEOWNERS review wave.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
