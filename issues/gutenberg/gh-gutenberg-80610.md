# #80610: Notes: Sync the sidebar selection to the inline marker under the caret

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Backported to WP Core`, `[Feature] Notes`
- **Merged:** [`1f27df2`](https://github.com/WordPress/gutenberg/commit/1f27df2962c8f459582eeb41435251713f30c810)
- **Discussion:** [#80610](https://github.com/WordPress/gutenberg/pull/80610) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Placing the caret inside an inline `core/note` marker now automatically selects that note in the collaboration sidebar. Previously, moving the caret between inline notes left the sidebar selection stale, requiring manual interaction to sync the view. This change aligns the sidebar state with the active editing position, improving workflow continuity for collaborative editing.

## Impact

- **Editor users & collaborative teams:** Sidebar selection now follows the caret when navigating between inline notes, reducing manual clicks.
- **Plugin & theme developers:** No breaking changes or API modifications. The `core/note` format behavior is internal to the block editor UI.
- **Hosting & platform teams:** No configuration or migration required.
- **Headless & REST consumers:** Unaffected; this is strictly a block editor UI behavior.
- **Action required:** None.

## Technical details

The change replaces the previous `edit: () => null` placeholder in `packages/editor/src/components/collab-sidebar/format.js` with a `NoteFormat` component. The component uses `useEffect` keyed on `[ isActive, noteId, getActiveComplementaryArea, getSelectedNote, selectNote ]` to imperatively sync the sidebar. It reads the active complementary area via `interfaceStore` and checks `SIDEBARS` to ensure the sidebar is open. If the marker is active and the note isn't already selected, it dispatches `selectNote( Number( noteId ) )` from `editorStore`. A deliberate design choice omits a cleanup function on unmount/leave to prevent dropping the note selection when the caret moves out of the marker but remains within the parent block. E2E tests were refactored to use a new `blockNoteUtils.selectBlockText()` helper for more reliable text range selection.

**Before:**
```js
edit: () => null,
```

**After:**
```js
edit: NoteFormat,
```

The `NoteFormat` component imports `useEffect`, `useDispatch`, `useSelect`, `interfaceStore`, `editorStore`, `SIDEBARS`, and `unlock`, then attaches the sync logic to the inline marker's lifecycle.

## Contribution

The change was authored and merged by the core editor team, with review from @adamsilverstein and approval from @jasmussen. During review, a cleanup function was proposed to deselect the note when the caret leaves the marker. The author evaluated the approach but rejected it, noting it would cause a behavior regression by dropping the note selection even when the caret remained inside the parent block. The PR was merged as a quality-of-life improvement with added e2e coverage.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
