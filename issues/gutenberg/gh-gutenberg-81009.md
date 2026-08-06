# #81009: Fix: Tabs block: Start with empty tab labels with placeholders

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @hbhalodia
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Backported to WP Core`, `[Block] Tabs`
- **Merged:** [`156a134`](https://github.com/WordPress/gutenberg/commit/156a13447c91cb3d6a12b507ee75df0592f2b88b)
- **Discussion:** [#81009](https://github.com/WordPress/gutenberg/pull/81009) · 8 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Tabs block now inserts new tab panels with empty labels instead of pre-filling them with the hardcoded word "Tab". Each empty label displays a "Tab title" placeholder in the editor, and the `core/tab-panel` block now accepts an optional `placeholder` attribute so patterns and templates can supply contextual prompts. This aligns the Tabs block with the rest of the block library, which defaults to empty fields with placeholders rather than pre-filled text.

## Impact

- **Plugin & theme developers**: No breaking changes or deprecations. Existing posts serialize identically. Patterns and templates can now pass a `placeholder` attribute to `core/tab-panel` to override the default editor prompt.
- **Site owners / editors**: New tabs start empty with a "Tab title" prompt. Publishing a post with untitled tabs renders empty tab buttons on the front end (consistent with how empty Headings behave).
- **No action required** for existing sites, plugins, or themes.

## Technical details

- `packages/block-library/src/tab-panels/edit.js`: `TAB_PANELS_TEMPLATE` changed from `[ [ 'core/tab-panel', { label: __( 'Tab' ) } ], [ 'core/tab-panel', { label: __( 'Tab' ) } ] ]` to `[ [ 'core/tab-panel' ], [ 'core/tab-panel' ] ]`, removing the `__( 'Tab' )` import entirely.
- `packages/block-library/src/tabs/use-tab-actions.js`: `insertTab` now calls `createBlock( 'core/tab-panel' )` without a `label` prop, so toolbar and keyboard additions also start empty.
- `packages/block-library/src/tab-panel/block.json`: Added an optional `placeholder` string attribute.
- `packages/block-library/src/tab-list/edit.js`: `RichText` components now read the panel's `placeholder` attribute, falling back to `__( 'Tab title' )`. The placeholder is editor-only; `tab-list/save.js` does not pass it to `RichText.Content`, so it never appears in saved markup or front-end output.
- Before/after insertion pattern:
  ```js
  // Before
  createBlock( 'core/tab-panel', { label: __( 'Tab' ) } )
  // After
  createBlock( 'core/tab-panel' )
  ```

## Contribution

Opened by @hbhalodia and merged by @t-hamano, with review contributions from @Mamaduka, @hanneslsm, and @tyxla. Reviewers initially debated whether to ship only the empty-label fix for the 7.1 backport, suggesting the customizable `placeholder` attribute be deferred to a follow-up ticket. The author retained the attribute to support pattern-level context, and the PR merged with both changes. A manual cherry-pick resolved a changelog conflict during the backport to `wp/7.1`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
