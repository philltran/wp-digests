# #79947: Visual revisions: Make the autosave notice work with the visual revisions UI

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @priethor
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Feature] History`
- **Merged:** [`bfdd9e1`](https://github.com/WordPress/gutenberg/commit/bfdd9e1c6eb33c77273faf564524ba383a9f78c3)
- **Discussion:** [#79947](https://github.com/WordPress/gutenberg/pull/79947) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The editor's autosave warning notice now opens the visual revisions UI instead of redirecting to the classic `revision.php` screen. When a post has a newer autosave, clicking "View the autosave" extracts the revision ID from `settings.autosave.editLink` and calls `setCurrentRevisionId()` to highlight the changes in-place. If visual revisions are disabled for the post, it gracefully falls back to the classic screen. Restoring the autosave now automatically dismisses the notice after a successful save.

## Impact

- **Plugin & theme developers:** No code changes required. The behavior change is internal to the block editor and does not expose new public APIs or hooks.
- **Site owners & editors:** Improves workflow by keeping autosave previews within the visual editor, reducing context switching.
- **Hosting & platform teams:** No configuration or migration needed. The fallback to `revision.php` ensures compatibility with posts that have classic meta boxes or disabled visual revisions.
- **No action required** for existing codebases or custom editor configurations.

## Technical details

The change modifies `packages/editor/src/components/provider/index.js` and `packages/editor/src/store/private-actions.js`. In the provider, `getQueryArg` extracts the autosave ID from `settings.autosave.editLink`. The notice action's `url` property is replaced with an `onClick` handler that reads `disableVisualRevisions` at runtime and either calls `setCurrentRevisionId( autosaveId )` or falls back to the classic screen:

**Before:**
```js
actions: [
  {
    label: __( 'View the autosave' ),
    url: settings.autosave.editLink,
  },
]
```

**After:**
```js
actions: [
  {
    label: __( 'View the autosave' ),
    ...( autosaveId
      ? { onClick: () => {
          const { disableVisualRevisions } = registry.select( editorStore ).getEditorSettings();
          if ( disableVisualRevisions ) {
            window.location.href = settings.autosave.editLink;
            return;
          }
          setCurrentRevisionId( autosaveId );
        } }
      : { url: settings.autosave.editLink } ),
  },
]
```

In `private-actions.js`, the `restoreRevision` private action now checks `select.didPostSaveRequestFail()` after `dispatch.savePost()` and explicitly removes the `'autosave-exists'` notice via `noticesStore` on success. An e2e test was added to `test/e2e/specs/editor/various/autosave.spec.js` to verify the visual revisions flow and notice dismissal.

## Contribution

Opened and merged by @priethor, with co-authors @Mamaduka and @annezazu. The PR was part of the broader visual revisions initiative (#79120). Reviewers confirmed that checking `disableVisualRevisions` at click time rather than render time was necessary because the flag is only set after mount for posts with classic meta boxes. @Mamaduka noted a planned notice relocation in #79801, but it did not block this enhancement. The change shipped with a +131 B size increase to `build/scripts/editor/index.min.js`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
