# #78973: Navigation Link: fix duplicate block html attributes in editor

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Package] Block library`, `[Block] Navigation Link`
- **Merged:** [`9f7acdb`](https://github.com/WordPress/gutenberg/commit/9f7acdba579019a610dd328e9031e33c015af431)
- **Discussion:** [#78973](https://github.com/WordPress/gutenberg/pull/78973) · 3 comments · 0 reactions

## Summary

The Navigation Link block's editor no longer applies `blockProps` to its inner wrapper, eliminating duplicate DOM attributes that previously caused focus navigation bugs and redundant drag-and-drop event listeners. This ensures keyboard navigation correctly advances focus to the next block without inadvertently targeting a shadowed container element.

## Impact

- **Plugin & Theme Developers**: No action required. The change is isolated to the block editor's render logic and does not affect frontend output or REST/block schema.
- **Site Editors / Content Creators**: Improved block navigation behavior in the site canvas; pressing `Right Arrow` at the end of link text now immediately advances focus to the next block instead of shifting to a duplicate inner container.
- **E2E Test Authors**: Two assertions in `navigation.spec.js` were updated to expect a single `ArrowRight` press rather than two, reflecting the corrected focus traversal.

## Technical details

In `packages/block-library/src/navigation-link/edit.js`, the spread operator for `blockProps` was removed from the configuration object passed to `useInnerBlocksProps()`. The inner container is strictly used as a dropzone for drag-and-drop operations and does not require structural props, ARIA labels, or custom properties. This removes duplicated attributes (including `id`, `data-block`, `data-type`, and inline styles) from the nested `.block-editor-block-list__layout` div. Corresponding e2e tests in `test/e2e/specs/editor/blocks/navigation.spec.js` were updated to align keystroke expectations with the corrected navigation flow.

## Contribution

Opened and merged by @talldan to resolve a long-standing editor duplication issue affecting focus management and DOM integrity. Review and co-authorization were provided by @scruffian (per merge commit metadata). The diff reflects a minimal configuration cleanup without altering frontend output or block schema behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
