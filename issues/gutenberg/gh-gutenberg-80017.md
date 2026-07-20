# #80017: Notes: Remove snackbar when resolving or reopening a note

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Package] Editor`
- **Merged:** [`58e0d3d`](https://github.com/WordPress/gutenberg/commit/58e0d3d9b3401a1a4051fc76e4d93530e971187a)
- **Discussion:** [#80017](https://github.com/WordPress/gutenberg/pull/80017) · 8 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg editor no longer displays a snackbar notice when a collaborator resolves or reopens a block note. Because the note's visual state already updates in place (collapsing in the sidebar or disappearing in floating mode), the toast was redundant. The change replaces the snackbar with a `speak()` call that announces the status change to screen readers, while snackbars remain active for content edits, additions, and deletions.

## Impact

- **Plugin & theme developers / Editor users:** No code changes or configuration updates required. The change only affects the client-side collaboration (Notes) UI feedback loop.
- **Accessibility users:** Screen reader confirmation for resolve/reopen actions is preserved via `speak()`, maintaining the feedback previously provided by the snackbar.
- **E2E test authors:** Automated tests that previously waited for the resolve/reopen snackbar to appear now gate on the note's actual UI state (e.g., button disabled/enabled states, inline marker removal, or thread visibility).
- **No breaking changes or deprecations.**

## Technical details

The change modifies `packages/editor/src/components/collab-sidebar/hooks.js` inside the `useNoteActions` hook's `onEdit` handler. Previously, the handler used a `messages` lookup object and dispatched `createNotice( 'snackbar', ... )` for all status changes (`approved`, `hold`, `updated`). The diff removes the `messages`/`messageType` logic and conditionally dispatches `createNotice` only for content edits. For `approved` (resolve) and `hold` (reopen) branches, it now imports and calls `speak()` from `@wordpress/a11y` to announce the status to assistive technology.

**Before:**
```js
const messageType = status ? status : 'updated';
const messages = {
    approved: __( 'Note marked as resolved.' ),
    hold: __( 'Note reopened.' ),
    updated: __( 'Note updated.' ),
};
// ... later ...
createNotice( 'snackbar', messages[ messageType ] ?? __( 'Note updated.' ), {
    type: 'snackbar',
    isDismissible: true,
} );
```

**After:**
```js
if ( status === 'approved' || status === 'hold' ) {
    // ... save logic ...
    speak( status === 'approved' ? __( 'Note marked as resolved.' ) : __( 'Note reopened.' ) );
} else {
    // ... save logic ...
    createNotice( 'snackbar', __( 'Note updated.' ), {
        type: 'snackbar',
        isDismissible: true,
    } );
}
```

The e2e test suite in `test/e2e/specs/editor/various/block-notes.spec.js` was updated to remove assertions for the snackbar's visibility and instead verify UI state changes like `resolveButton` disabled/enabled states and `thread` visibility.

## Contribution

Opened and merged by @adamsilverstein, with co-authors including @mamaduka, @t-hamano, @jeffpaul, @jasmussen, and @joedolson. The initial implementation removed the snackbar entirely, but during review @t-hamano noted the need to preserve screen reader feedback. The author adjusted the implementation to use `speak()` for the resolve/reopen actions, which was verified with Voiceover before final merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
