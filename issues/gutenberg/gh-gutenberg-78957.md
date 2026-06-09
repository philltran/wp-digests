# #78957: Patterns: fix focus loss when closing the Create pattern dialog from the block toolbar

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mustafabharmal
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Feature] Patterns`, `[Package] Patterns`
- **Merged:** [`7945bcc`](https://github.com/WordPress/gutenberg/commit/7945bccbdc161b503e228a978f0adda450435568)
- **Discussion:** [#78957](https://github.com/WordPress/gutenberg/pull/78957) · 3 comments · 1 reactions

## Summary

Fixes a focus management regression in the Block Editor where dismissing the "Create pattern" dialog via Escape caused focus to be lost. The change ensures the parent block settings dropdown remains open so keyboard focus correctly returns to the triggering menu item, aligning with the behavior of other Options menu dialogs like Lock and Rename.

## Impact

- **Editors & Keyboard/Screen Reader Users**: Resolves a UX regression where focus disappeared when canceling the Create Pattern workflow. No migration or code changes required.
- **Plugin/Theme Developers**: Unaffected; this is an internal editor behavior fix within the Patterns package.

## Technical details

- Modified `packages/patterns/src/components/pattern-convert-button.js` to remove `closeBlockSettingsMenu()` from the `CreatePatternModal`'s `onClose` handler.
- Removed code:
  ```diff
  onClose={ () => {
      setIsModalOpen( false );
-     closeBlockSettingsMenu();
  } }
  ```
- The dropdown closing logic is preserved in the success flow (`handleSuccess`) to ensure the menu closes after a pattern is actually created.
- Aligns behavior with other Options menu dialogs (Lock, Rename, Hide) which keep the parent dropdown open on dismiss.

## Contribution

Merged PR #78957 by @Mustafabharmal, co-authored with @t-hamano, @talldan, and @afercia. The pull request addressed a regression in focus restoration when dismissing the dialog. Review validated that removing the dropdown-closing call did not reintroduce a prior issue from PR #75405 related to unsynced pattern menu items.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
