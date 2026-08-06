# #80994: Copy: preserve the block when its entire text is selected

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Rich text`, `[Package] Block editor`, `Backported to WP Core`
- **Merged:** [`8e2fc46`](https://github.com/WordPress/gutenberg/commit/8e2fc461a08620c865da39c3c1198404cc1a0d8e)
- **Discussion:** [#80994](https://github.com/WordPress/gutenberg/pull/80994) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a bug where copying the entire text of a block (such as a heading or paragraph) stripped its block type, pasting as plain text. When a block's full content is selected and copied, the block itself is now placed on the clipboard. Pasting into an empty block recreates the original block, while pasting into existing text inserts the content inline. This aligns the block editor with standard behavior in Word and Google Docs.

## Impact

- **Block editor users**: Copying all text from a heading or paragraph now preserves block type and formatting on paste, matching desktop editor expectations.
- **Plugin & theme developers**: No public API changes, but custom `RichText` components or clipboard integrations that bypass the block editor's default flow should respect `event.defaultPrevented` to avoid overwriting the clipboard claim.
- **Hosting & platform teams**: No action required.

## Technical details

The change modifies `packages/block-editor/src/components/writing-flow/use-clipboard-handler.js` to detect when a single block's entire text is selected via a new `isBlockEntirelySelected()` helper. When detected, the copy event is routed through `setClipboardBlocks` instead of allowing native copy. The paste handler now dispatches `replaceBlocks()` when pasting over a fully selected block. In `packages/rich-text/src/hook/event-listeners/copy-handler.js`, the handler now checks `event.defaultPrevented` to yield to the block editor's clipboard claim. A prior approach that carried the tag name on the rich text clipboard was reverted due to regressions in nested rich texts like image captions; this PR avoids that by verifying the editable element matches the block root rather than a nested field.

## Contribution

Opened and merged by @ellatrix with co-authors including @richtabor, @ndiego, and @danielbachhuber. The PR resolves a regression from a previous attempt (#48254) that was reverted after breaking copy/paste in nested rich texts like image captions. The final approach distinguishes block-level edits from nested fields by checking if the editable element is the block root. The change was backported to the wp/7.1 branch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
