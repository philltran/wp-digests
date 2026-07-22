# #80531: Notes: remove "Add note" from the inline styles dropdown

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `Backported to WP Core`, `Collaborative Workflows`, `[Feature] Notes`
- **Merged:** [`e5da6f8`](https://github.com/WordPress/gutenberg/commit/e5da6f8acd0fe794d56705224dd96ee37e87d340)
- **Discussion:** [#80531](https://github.com/WordPress/gutenberg/pull/80531) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Removes the "Add note" entry from the rich-text formatting toolbar's "More" dropdown in the block editor. The item was an unintended side effect of registering a rich-text format with an `edit` component, which incorrectly placed a collaborative workflow action alongside inline styles. The change consolidates note creation and thread management to the block options menu and its keyboard shortcut, aligning the UI with the original design direction for the feature.

## Impact

- Block editor users: The "Add note" button no longer appears in the inline styles dropdown; note creation and thread management now only occur via the block options (ellipsis) menu or the `core/editor/new-note` keyboard shortcut.
- Plugin & theme developers: No action required. The `core/note` rich-text format remains registered and continues to serialize `<mark class="wp-note">` anchors into post content.
- No breaking API changes, deprecations, or migration steps.

## Technical details

In `packages/editor/src/components/collab-sidebar/format.js`, the `noteFormat` object previously included an `edit: NoteFormatEdit` property. The diff removes this property and deletes the `NoteFormatEdit` component entirely. Because `edit` is optional in the rich-text format API, `FormatEdit` returns `null` for this format, preventing it from rendering in `RichText.ToolbarControls` (which populates the "More" dropdown). The format's `title` is also changed from `'Add note'` to `'Note'`. The `onCreate` logic remains untouched, reading the selection from the block-editor store to create inline or block-level notes as appropriate. The only removed behavior is clicking the toolbar button while the caret sits inside an existing note marker to open that thread; clicking the highlight itself remains the affordance. E2E tests in `test/e2e/specs/editor/various/block-notes.spec.js` are updated to route through `editor.clickBlockOptionsMenuItem('Add note')` instead of navigating the rich-text dropdown.

## Contribution

Opened by @adamsilverstein and co-authored by @t-hamano and @Mamaduka, the PR addresses a UI placement oversight from the initial inline notes implementation (#78218). Reviewers noted the toolbar entry contradicted the original design direction to keep notes in the block options menu. The change was quickly reviewed, merged, and cherry-picked to the `wp/7.1` and `release/23.6` branches for inclusion in the next WordPress release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
