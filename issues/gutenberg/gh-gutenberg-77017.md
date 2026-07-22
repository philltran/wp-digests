# #77017: Icon Block: Add flip and rotate transformation controls

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @dhruvikpatel18
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Icon`
- **Merged:** [`b08653a`](https://github.com/WordPress/gutenberg/commit/b08653ae83a20f69f4657f1da11b2e3d27683f5a)
- **Discussion:** [#77017](https://github.com/WordPress/gutenberg/pull/77017) · 6 comments · 2 reactions
- **Usefulness:** 4/5

## Summary

The `core/icon` block now supports horizontal flipping, vertical flipping, and 90-degree clockwise rotation via new toolbar controls. This enhancement introduces three block attributes and applies corresponding CSS transforms in both the editor and frontend rendering, giving content authors direct transformation capabilities without requiring custom CSS or external libraries.

## Impact

- **Block & theme developers**: New block attributes (`flipHorizontal`, `flipVertical`, `rotation`) are available for programmatic manipulation or conditional styling. No breaking changes or deprecations.
- **Plugin developers**: If you programmatically render, sanitize, or query `core/icon` blocks, you may need to account for the new attributes in your logic.
- **Site owners & editors**: Can now flip and rotate icons directly in the block toolbar.
- **No action required**: Existing blocks render unchanged; the new attributes default to `false`/`0` and only apply when toggled.

## Technical details

The diff adds `flipHorizontal` (boolean, default `false`), `flipVertical` (boolean, default `false`), and `rotation` (number, default `0`) to `packages/block-library/src/icon/block.json`. In `packages/block-library/src/icon/edit.js`, a new `ToolbarGroup` is appended to `BlockControls group="other"` when an icon is selected. It renders three `ToolbarButton` controls that toggle the boolean attributes and cycle `rotation` by `( ( rotation || 0 ) + 90 ) % 360`. The editor applies `is-flip-horizontal` and `is-flip-vertical` classes alongside an inline `rotate` style via `rotationStyle`. On the frontend, `packages/block-library/src/icon/index.php`'s `render_block_core_icon` function reads these attributes, appends the corresponding CSS classes to the SVG processor, and injects `rotate: Xdeg;` into the inline style attribute. `packages/block-library/src/icon/style.scss` defines the `transform` rules for `.is-flip-horizontal` (`scaleX(-1)`), `.is-flip-vertical` (`scaleY(-1)`), and their combined state.

## Contribution

Opened by @dhruvikpatel18 as part of issue #75715, the PR was reviewed by @t-hamano, @fcoveram, and @jasmussen. Reviewers suggested adding a toolbar separator after the replace control and auto-inserting a default icon on block creation, but both were explicitly deferred to follow-up PRs to keep the scope focused. @dhruvikpatel18 resolved trunk merge conflicts and addressed feedback before the PR was merged.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
