# #77779: Revisions: Specify block level diff status via aria-label

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @himanshupathak95
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Package] Editor`, `[Package] Block library`, `[Feature] History`, `[Package] Block editor`, `Backported to WP Core`
- **Merged:** [`2fd6eb7`](https://github.com/WordPress/gutenberg/commit/2fd6eb7e51b255b32950181e5b6985e24943985d)
- **Discussion:** [#77779](https://github.com/WordPress/gutenberg/pull/77779) · 13 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Screen reader users now hear whether a block was added, removed, or modified when navigating the post revisions canvas. The change injects a status-specific `aria-label` into the block wrapper and the Paragraph block’s `RichText` component, replacing the previous behavior where only the block type was announced. This closes an accessibility gap in the history feature without altering any public APIs.

## Impact

- **Screen reader users:** Gains programmatic indication of revision diff status when navigating the revisions canvas.
- **Plugin & theme developers:** No action required. The implementation uses private editor APIs and does not expose new public hooks, filters, or `useBlockProps` arguments.
- **Block authors:** No changes needed. The Paragraph block was updated internally to consume the private context, and the pattern is isolated to the revisions preview filter.

## Technical details

The diff modifies three core files to route a diff status label through a private React context:
- `packages/block-editor/src/components/block-list/use-block-props/index.js` now destructures `ariaLabel` from `PrivateBlockContext` and applies it as `'aria-label': ariaLabel ?? blockLabel`, taking precedence over the default block label.
- `packages/block-library/src/paragraph/edit.js` unlocks `blockEditorPrivateApis`, reads `ariaLabel` from `PrivateBlockContext`, and passes it to the `RichText` component’s `aria-label` prop, ensuring the inner editable region also announces the status.
- `packages/editor/src/components/post-revisions-preview/revisions-canvas.js` introduces `BlockDiffLabelProvider`, which wraps blocks with a `__revisionDiffStatus` in `PrivateBlockContext.Provider`. The `withRevisionDiffClasses` filter conditionally applies this wrapper only when a diff status exists, preventing nested blocks from inheriting ancestor labels. A `getDiffStatusLabel` helper formats strings like `Added block: Paragraph`.

Before/after in `useBlockProps`:
```diff
- 'aria-label': blockLabel,
+ 'aria-label': ariaLabel ?? blockLabel,
```
- New E2E coverage in `test/e2e/specs/editor/various/revision-diff-aria-labels.spec.js` asserts correct announcements for added/removed/modified blocks, verifies variation-aware titles (e.g., `Added block: Row`), and confirms labels revert when diff highlighting is toggled off.

## Contribution

Opened by @himanshupathak95, the PR initially attempted to inject labels via `wrapperProps`, but review revealed `useBlockProps` explicitly overrides `aria-label` after spreading, breaking the approach. The team pivoted to a private React context (`PrivateBlockContext` via `blockEditorPrivateApis`) to avoid expanding the public `useBlockProps` API surface. @cbravobernal pushed the final implementation commits to ensure the feature remained strictly within private APIs, updating the Paragraph block to consume the context directly. The PR was merged with a backport label and includes comprehensive E2E tests.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
