# #54292: Extract undo/redo as a separate package

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @youknowriad
- **Labels:** `Framework`, `[Type] New API`
- **Merged:** [`d9208b8`](https://github.com/WordPress/gutenberg/commit/d9208b8ee4037d8b7b4e2801b4ba7c2ff13bc094)
- **Discussion:** [#54292](https://github.com/WordPress/gutenberg/pull/54292) · 7 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

The `core-data` package now delegates undo/redo state management to a new `@wordpress/undo-manager` package. This extraction isolates the history logic, allowing third-party block editors to reuse the same undo/redo behavior without reimplementing it from scratch.

## Impact

- **Third-party block editor developers:** Can now instantiate `createUndoManager()` and integrate it into their editor setup to handle history without writing custom reducer logic.
- **Plugin & theme developers:** No action required. The change is internal to `core-data` and the new package is bundled, not published to npm.
- **Core developers:** `core-data`'s internal reducer and selectors were refactored to delegate to the new manager, simplifying the state tree.

## Technical details

The diff replaces the hand-rolled `undo` reducer in `packages/core-data/src/reducer.js` with a thin `undoManager` reducer that simply returns the manager instance. `core-data` actions (`editEntityRecord`, `undo`, `redo`, `__unstableCreateUndoLevel`) now call `select.getUndoManager().addRecord()`, `.undo()`, and `.redo()` instead of manipulating `state.undo.list` and `state.undo.offset`. Private selectors `getUndoEdits` and `getRedoEdits` are replaced by `getUndoManager`, while public selectors `hasUndo` and `hasRedo` now query `state.undoManager.getUndoRecord()` and `state.undoManager.getRedoRecord()`. The new `@wordpress/undo-manager` package is registered in `package.json` and `docs/manifest.json` but marked as bundled to avoid exposing it as a public WordPress API prematurely.

## Contribution

Opened and merged by @youknowriad. Reviewers @jorgefilipecosta and @ellatrix discussed whether to publish the manager as a public npm package or keep it internal. The team agreed to bundle it initially to serve third-party framework developers without committing to a stable public contract, leaving the door open for future public exposure or alternative implementations.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
