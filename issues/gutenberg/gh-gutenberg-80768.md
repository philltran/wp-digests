# #80768: Editor: leave undo to the browser in fields that handle their own undo

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`, `[Package] Edit Post`, `[Package] Edit Widgets`, `Backported to WP Core`
- **Merged:** [`b9d5bdd`](https://github.com/WordPress/gutenberg/commit/b9d5bdd7b087a140b6231ea2ea0b9109d3ed34d1)
- **Discussion:** [#80768](https://github.com/WordPress/gutenberg/pull/80768) · 11 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The block editor now delegates undo and redo keyboard shortcuts to the browser’s native undo stack when focus is inside fields that manage their own local state, rather than rewinding the global editor history. This resolves a long-standing bug where typing in the link dialog, "Edit as HTML" textarea, Custom HTML block modal, or classic meta box fields would incorrectly undo unrelated canvas changes. The change ensures that undo/redo behavior matches user expectations in these isolated editing contexts without altering how block attributes are tracked.

## Impact

- **Plugin & theme developers:** No action required. The registration mechanism is intentionally private and exposed only via `privateApis`, so third-party fields synced to block attributes continue using editor undo by default.
- **Site owners & editors:** Improved undo/redo reliability in dialogs and meta boxes; no configuration or migration steps needed.
- **Hosting & platform teams:** No action required. The change is fully backward compatible, does not modify public APIs, block schemas, or REST endpoints, and ships as a standard editor patch.

## Technical details

Introduces `packages/block-editor/src/utils/native-undo.js`, which exports a `useNativeUndo()` ref callback and a `usesNativeUndo(event)` utility. The utility checks a module-private `WeakSet` (`nativeUndoNodes`) to determine if the keyboard event originates from a registered element, correctly resolving iframe focus via `target.contentDocument?.activeElement`. Global shortcut handlers in `packages/editor/src/components/global-keyboard-shortcuts/index.js`, `packages/edit-post/src/components/keyboard-shortcuts/index.js`, `packages/edit-widgets/src/components/keyboard-shortcuts/index.js`, and `packages/customize-widgets/src/components/keyboard-shortcuts/index.js` now gate their undo/redo dispatches behind a `usesNativeUndo( event )` check:

```javascript
// Before
useShortcut( 'core/editor/undo', ( event ) => {
	undo();
	event.preventDefault();
} );

// After
useShortcut( 'core/editor/undo', ( event ) => {
	if ( usesNativeUndo( event ) ) {
		return;
	}
	undo();
	event.preventDefault();
} );
```

Components that hold local state—`BlockHTML`, `LinkControl`, `HTMLEditModal`, and `MetaBoxesArea`—attach the `useNativeUndo()` ref to their root containers. The functions are registered in `packages/block-editor/src/private-apis.js` and unlocked via `blockEditorPrivateApis` in consuming packages. No public hooks, filters, or REST routes are added or modified.

## Contribution

Opened by @ellatrix to resolve #80541 and #38731, the PR focused on isolating editor shortcuts from local-state fields. During review, @Mamaduka suggested using a `data-native-undo="true"` attribute, but the author rejected it to avoid exposing a de facto public API that would constrain future refactors. @scruffian raised questions about cross-browser E2E coverage and changelog entries, which were addressed by confirming the opt-in nature and adding targeted regression tests. @t-hamano contributed a follow-up commit (`df3b53ca`) to extend the `usesNativeUndo` guard to the `edit-widgets` and `customize-widgets` shortcut handlers, ensuring consistent behavior across all editor surfaces. The change was subsequently backported to WordPress Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
