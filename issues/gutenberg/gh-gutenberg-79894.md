# #79894: Classic block: Remove migration notice and restore inserter availability

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tyxla
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Classic`
- **Merged:** [`03a0ed8`](https://github.com/WordPress/gutenberg/commit/03a0ed89add2ec58d0093ecf5ae88c0e4823dcd6)
- **Discussion:** [#79894](https://github.com/WordPress/gutenberg/pull/79894) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This pull request reverts experimental deprecation work for the Classic block by removing its migration notice and restoring full inserter availability. The `window.__needsClassicBlock` runtime gate is dropped, editor UX returns to a pre-experiment state featuring a “Convert to blocks” toolbar button, and the legacy `wp_classic_block_supports_inserter` filter is permanently removed.

## Impact

- **Theme & Plugin Developers**: The `wp_classic_block_supports_inserter` filter is gone. Because it never shipped in a WordPress release, existing plugins are unaffected, but any experimental opt-in code should be cleaned up.
- **Editors & Site Owners**: The Classic block is again visible in the inserter by default. The deprecation warning and migration actions are removed; editing reverts to the previous “Convert to blocks” toolbar flow.
- No configuration or migration steps required for standard installations.

## Technical details

- Deletes `lib/compat/wordpress-7.1/classic-block.php` (which previously hooked into `enqueue_block_editor_assets` to inject `window.__needsClassicBlock = true;`) and removes its `require` in `lib/load.php`. Corresponding PHPUnit tests in `phpunit/script-loader-test.php` are dropped.
- Simplifies `packages/block-library/src/freeform/index.js` so `init()` calls `initBlock( { name, metadata, settings } )` directly, allowing `supports.inserter` to fall back to its default `true` value.
- Replaces the deleted `migration-notice.js` with a restored `convert-to-blocks-button.js` component in `packages/block-library/src/freeform/edit.js`. Removes `onReplace` and `removeBlock` logic, keeping only the toolbar button and original placeholder instructions.
- E2E assertions in `test/e2e/specs/editor/blocks/classic.spec.js` revert to using `editor.clickBlockToolbarButton('Convert to blocks')` instead of canvas-based queries.

## Contribution

Opened by @tyxla to roll back experimental deprecation gating that was causing editor friction (tracked in #78067). Merged with co-authorship from @youknowriad. The review discussion was minimal, with @tyxla noting the failing e2e test was an unrelated pre-existing regression. The change cleanly undoes the prior PHP gate, JS migration warning, and inserter filtering logic.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
