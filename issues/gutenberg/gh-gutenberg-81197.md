# #81197: [WP 7.1] Fix: Tabs block: Start with empty tab labels with placeholders

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @hbhalodia
- **Labels:** `[Type] Enhancement`, `[Package] Block library`
- **Merged:** [`ab8421a`](https://github.com/WordPress/gutenberg/commit/ab8421ab6f367a58fe7028c44e03a28725708add)
- **Discussion:** [#81197](https://github.com/WordPress/gutenberg/pull/81197) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tabs block now initializes new tab panels with an empty label and a visible "Tab title" placeholder instead of a default "Tab" text. This applies to both the initial block template and any tabs added via the editor toolbar. The change improves the authoring experience by prompting editors to provide meaningful labels rather than leaving generic placeholder text.

## Impact

- **Block authors & theme developers:** No code changes required. Existing saved content remains unaffected.
- **Site owners & editors:** New tabs created in the block editor will start empty with a visible placeholder, reducing the need to delete default "Tab" text before editing.
- **No action required** for existing sites, plugins, or themes; this is a purely editorial UX improvement.

## Technical details

The diff removes the hardcoded `__( 'Tab' )` default from two locations in the block library:
- `packages/block-library/src/tab-panels/edit.js`: The `TAB_PANELS_TEMPLATE` constant now instantiates `core/tab-panel` blocks without a `label` attribute.
- `packages/block-library/src/tabs/use-tab-actions.js`: The `createBlock( 'core/tab-panel', { label: __( 'Tab' ) } )` call in the tab insertion logic is updated to `createBlock( 'core/tab-panel' )`.
- The `@wordpress/i18n` import is removed from both files since it is no longer used.
- An e2e test in `test/e2e/specs/editor/blocks/tabs.spec.js` verifies that newly inserted tabs render a `[data-rich-text-placeholder]` attribute with the value `Tab title`, confirming the RichText component handles the placeholder when the `label` attribute is absent.

Before/after pattern:
```js
// Before
createBlock( 'core/tab-panel', { label: __( 'Tab' ) } )
// After
createBlock( 'core/tab-panel' )
```

## Contribution

This change is a manual backport of upstream PR #81009 to the `wp/7.1` branch, initiated because label-triggered cherry-picks fail for forked repositories. The implementation was authored by @hbhalodia with co-authorship from @t-hamano, and received a brief review from @hanneslsm. The record carries no extended design debate or alternative approaches.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
