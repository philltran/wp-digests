# #80396: Rich Text: Restore the selection when focus returns to the editable

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Rich text`, `Backport to Gutenberg RC`, `Backported to WP Core`
- **Merged:** [`5e5c9ea`](https://github.com/WordPress/gutenberg/commit/5e5c9ea27334d4e803db5da78970eb6e51fc659d)
- **Discussion:** [#80396](https://github.com/WordPress/gutenberg/pull/80396) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Rich Text package now correctly restores the user’s text selection when focus returns to an editable field after interacting with the link popover (e.g., pressing `Cmd`+`K` then `Escape`). Previously, a vestigial `domOnly: true` flag in the focus handler prevented `applyRecord` from syncing the selection state back to the DOM, causing the caret to collapse to the start of the text instead of preserving the selection. This fix applies to same-document rich text contexts like the non-iframed block editor, widgets screen, and `RichTextControl`.

## Impact

- **Plugin & theme developers**: If you use the `RichText` component or `RichTextControl` in same-document contexts (non-iframed editor, widgets screen, block notes), focus interactions with the link popover will now correctly preserve text selection.
- **No action required**: This is a bug fix with no API changes, deprecations, or migration steps.
- **Hosting & platform teams**: Ships automatically with the Gutenberg plugin and WordPress core; no configuration changes needed.

## Technical details

The change modifies `packages/rich-text/src/hook/event-listeners/input-and-selection.js`. In the focus handler, the call to `applyRecord( record.current, { domOnly: true } )` is replaced with `applyRecord( record.current )`. The `domOnly: true` flag was originally added in #58745 to work around #58322, but became unnecessary after #59635 introduced `shouldAutoFocus`, which stopped the Link UI from stealing focus. The diff also adds an e2e assertion in `test/e2e/specs/editor/various/block-notes.spec.js` that verifies `window.getSelection().toString()` matches the expected text after closing the link popover with `Escape`.

## Contribution

Opened by @Mamaduka and merged as `5e5c9ea`. @ellatrix and @noruzzamans verified the fix on `trunk` and the feature branch without regressions. The PR was backported to the `wp/7.1` branch for inclusion in the next WordPress release. The discussion was minimal, focusing on testing confirmation and the historical context of the `domOnly` flag.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
