# #77301: Cover/Accordion: Exit on Enter

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ocean90
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Cover`, `[Block] Accordion`
- **Merged:** [`62bd7a6`](https://github.com/WordPress/gutenberg/commit/62bd7a60070a00031159f11c042800a7805eb637)
- **Discussion:** [#77301](https://github.com/WordPress/gutenberg/pull/77301) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Cover and Accordion Panel blocks now support exiting the block via the Enter key instead of inserting empty paragraphs. This change resolves a keyboard navigation regression where pressing Enter trapped focus inside the block and created collapsed empty paragraphs on the frontend. It applies the existing `__experimentalOnEnter` support key to these blocks' metadata configurations.

## Impact

- **Block & theme developers:** No code changes required. The behavior is controlled entirely via `block.json` metadata.
- **Site owners & editors:** Improved keyboard accessibility. Pressing Enter inside these blocks now moves the cursor to the next block instead of creating empty paragraphs.
- **No action required** for existing sites, plugins, or themes.

## Technical details

The diff adds `"__experimentalOnEnter": true` to the `supports` object in two block definition files:
- `packages/block-library/src/cover/block.json`
- `packages/block-library/src/accordion-panel/block.json`

This leverages an existing internal editor mechanism that intercepts the Enter key event within the block's content area. Previously, pressing Enter triggered the default paragraph insertion behavior, which collapsed into empty paragraphs on the frontend and trapped keyboard focus. With the flag enabled, the block editor's navigation logic handles the Enter key to move focus to the adjacent block. No new hooks, filters, REST routes, or database changes are introduced.

## Contribution

Opened and merged by @ocean90, closing #77294. @jasmussen reviewed and approved, noting the change resolves a keyboard trap introduced in #76568 and aligns with existing patterns used by other blocks. The approach was straightforward, relying on an already-implemented experimental support key rather than introducing custom event handlers or modifying block edit components.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
