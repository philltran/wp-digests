# #79069: Editor: Disable saving while a non-post entity is being saved

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Feature] Saving`, `[Package] Editor`
- **Merged:** [`b7b3236`](https://github.com/WordPress/gutenberg/commit/b7b323690a05b245361c677e7bf7d604a331c973)
- **Discussion:** [#79069](https://github.com/WordPress/gutenberg/pull/79069) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request fixes a race condition in the block editor where publishing or saving a post while creating a new taxonomy term would result in the post saving with pre-creation (stale) term IDs. The fix hoists the `isSavingNonPostEntityChanges` selector into the disabled logic for the `PostPublishButton`, `PostSavedState` button, and the global save keyboard shortcut. Saving is now declaratively blocked until the non-post entity request resolves, eliminating the need for imperative `lockPostSaving` calls in consumer components.

## Impact

- **Block editor users & developers**: No action required. The editor now automatically disables save/publish buttons and keyboard shortcuts while a new term (or other non-post entity) is being created.
- **Plugin/theme developers**: If you previously worked around this race condition by manually calling `lockPostSaving` or `lockPostAutosaving` in term selectors, you can remove those imperative locks. The editor now handles it declaratively via `isSavingNonPostEntityChanges`.
- **No breaking changes** to public APIs, component props, or existing data flows.

## Technical details

The diff modifies three editor components to check `isSavingNonPostEntityChanges` from the `editorStore` before allowing a save:
- `packages/editor/src/components/post-publish-button/index.js`: Updated `isButtonDisabled` and `isToggleDisabled` to include `isSavingNonPostEntityChanges` in their boolean logic.
- `packages/editor/src/components/post-saved-state/index.js`: Added `isSavingNonPostEntityChanges` to the `useSelect` return object and appended it to the `isDisabled` condition.
- `packages/editor/src/components/global-keyboard-shortcuts/index.js`: Added the selector to `useSelect` and updated the shortcut handler to early-return if the selector is true.

**Before/after (keyboard shortcut handler):**
```js
// Before
if ( isPostSavingLocked() ) {
    return;
}

// After
if ( isPostSavingLocked() || isSavingNonPostEntityChanges() ) {
    return;
}
```

The change replaces scattered imperative locking with a single declarative state check. It also simplifies `hasPublishAction` to use `!!` instead of `?? false`.

## Contribution

During review, the author proposed extending the fix to the global keyboard shortcut handler to ensure consistent behavior across all save triggers. The approach was adopted as the preferred alternative to a prior PR, favoring a declarative state check over imperative locking in term selector components. The change was quickly iterated on and merged after a brief review cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
