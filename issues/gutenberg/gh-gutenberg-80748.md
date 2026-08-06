# #80748: Notes: Report save success consistently from note actions

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Backported to WP Core`, `[Feature] Notes`
- **Merged:** [`cb83cf9`](https://github.com/WordPress/gutenberg/commit/cb83cf9f1644bf9e09b630cca7d0343f10a3f36d)
- **Discussion:** [#80748](https://github.com/WordPress/gutenberg/pull/80748) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes inconsistent return values in the collaborative notes sidebar that caused form state desynchronization, potential content loss on failed saves, and incorrect focus shifts after failed deletions. The `useNoteActions` hook now consistently resolves truthy on success and `undefined` on failure, allowing callers to conditionally clear drafts and manage UI state without losing user input.

## Impact

- **Site editors / collaborative users:** Prevents lost edits when saving fails (the form stays open with the draft intact), ensures the edit form closes only after a successful save, and stops selection/focus from jumping when a note deletion fails.
- **Plugin & theme developers:** No direct API changes or breaking changes. This modifies internal block editor component behavior.
- **Platform & hosting teams:** No action required.

## Technical details

Modifies `packages/editor/src/components/collab-sidebar/hooks.js` (`useNoteActions`): `onEdit`, `onResolve`, and `onReply` now explicitly `return savedRecord` on success; `onDelete` returns `true` on success. Both paths return `undefined` implicitly on catch.

Updates `packages/editor/src/components/collab-sidebar/note-form.js` to check `if (result)` before clearing `inputComment`, removing the outer `try/catch` since actions handle errors internally.

Updates `packages/editor/src/components/collab-sidebar/note.js` to await `onEditNote` and only call `handleCancel()` if the result is truthy. Before/after usage pattern:
```jsx
// Before
onSubmit={ ( value ) => {
	onEditNote( { id: note.id, content: value } );
	setActionState( null );
	actionButtonRef.current?.focus();
} }

// After
onSubmit={ async ( value ) => {
	const saved = await onEditNote( { id: note.id, content: value } );
	if ( saved ) {
		handleCancel();
	}
	return saved;
} }
```

Updates `packages/editor/src/components/collab-sidebar/notes.js` to check the `onDelete` return value before moving focus/selection to adjacent threads. Also captures `clientId` synchronously in `onDelete` before the `await deleteEntityRecord` to prevent race conditions where block selection shifts mid-request.

## Contribution

Opened to address regressions exposed by recent collaborative notes changes. Reviewed by @adamsilverstein and co-authored with @mcliwanow. Merged and immediately cherry-picked to the `wp/7.1` branch for inclusion in the next WordPress release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
