# #80163: Stabilize Tabs block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Tabs`
- **Merged:** [`6fd0fd7`](https://github.com/WordPress/gutenberg/commit/6fd0fd73c3a6aa868728a5933d92106b16231afc)
- **Discussion:** [#80163](https://github.com/WordPress/gutenberg/pull/80163) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Tabs block family (`core/tabs`, `core/tab-list`, `core/tab-panels`, `core/tab-panel`) is now stable and registered unconditionally. They no longer require the `gutenberg-block-experiments` flag to be enabled. The change also relaxes KSES to allow the `tabindex` attribute on all HTML elements, which is required for the Tab Panel block to maintain keyboard focus.

## Impact

- **Plugin & theme developers:** The blocks are now available by default. Code that conditionally registers or checks for experimental tab blocks can be simplified or removed.
- **Site owners & editors:** The Tabs block appears in the inserter immediately without navigating to Gutenberg > Experiments.
- **Security/KSES:** The `tabindex` attribute is now permitted on every HTML tag via the `wp_kses_allowed_html` filter. This is a minor KSES relaxation but necessary for the block's accessibility implementation.
- **No action required** for existing sites, but developers relying on the experimental flag or filtering block registration should update their code.

## Technical details

- Removed `"__experimental": true` from the `block.json` files for `core/tabs`, `core/tab-list`, `core/tab-panels`, and `core/tab-panel`.
- In `packages/block-library/src/index.js`, the blocks are now pushed unconditionally to the `getAllBlocks()` array, removing the `window?.__experimentalEnableBlockExperiments` guard.
- Added `gutenberg_add_tabindex_to_kses_allowed_html()` in `lib/compat/wordpress-7.1/kses.php`, which hooks into `wp_kses_allowed_html` to inject `'tabindex' => true` into every allowed tag array.
- E2E tests in `test/e2e/specs/editor/blocks/tabs.spec.js` no longer toggle `gutenberg-block-experiments`.
- Before/after registration pattern:
  ```js
  // Before
  if ( window?.__experimentalEnableBlockExperiments ) {
      blocks.push( tabList );
      blocks.push( tabs );
      blocks.push( tabPanel );
      blocks.push( tabPanels );
  }
  // After
  blocks.push( tabs, tabList, tabPanels, tabPanel );
  ```

## Contribution

Opened and merged by @t-hamano with co-authors @jasmussen, @Mamaduka, and @tyxla. The PR was prepared for the WordPress 7.1 release cycle. Reviewers confirmed stabilization could proceed independently of two dependent PRs since those contained only internal adjustments and bug fixes. The final diff reflects the unconditional registration, KSES filter addition, and documentation cleanup.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
