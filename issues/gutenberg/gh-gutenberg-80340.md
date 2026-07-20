# #80340: Custom CSS: Fix cascade order against block style variations

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Backport to Gutenberg RC`, `[Feature] Block Style Variations`, `Backported to WP Core`
- **Merged:** [`f74d9fb`](https://github.com/WordPress/gutenberg/commit/f74d9fb2ae61443bd2ecefbe19431fb03c2fcd60)
- **Discussion:** [#80340](https://github.com/WordPress/gutenberg/pull/80340) · 12 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a bug in the block editor where custom CSS added via a block’s Advanced panel was incorrectly overridden by block style variation styles. The editor now renders custom CSS after variation styles, matching the front-end cascade order and ensuring custom CSS wins at equal specificity.

## Impact

- **Block & theme developers:** Custom CSS entered in the block editor now reliably overrides block style variation styles in the editor canvas, aligning editor rendering with the front end.
- **Site owners & editors:** No direct action required; visual consistency between the editor and published content is improved.
- **No breaking changes or deprecations.**

## Technical details

The `useBlockProps` hook in `packages/block-editor/src/hooks/custom-css.js` now calls `usePrivateStyleOverride` (imported from `./utils`) instead of `useStyleOverride`, passing `clientId` and `__unstableType: 'custom-css'`:

```js
// Before
useStyleOverride( { css: transformedCSS } );

// After
usePrivateStyleOverride( {
    css: transformedCSS,
    clientId,
    __unstableType: 'custom-css',
} );
```

In `packages/block-editor/src/components/editor-styles/index.js`, the `EditorStyles` component partitions the `overrides` map to render non-custom-css overrides first, followed by custom-css overrides last. This guarantees custom CSS prints after variation styles, resolving the cascade tie at 0-1-0 specificity. The `clientId` parameter ensures custom CSS overrides maintain block order relative to each other, preventing instability when variations are switched. Front-end behavior was already correct via stylesheet enqueue order, so no PHP changes were made. The change includes a new e2e test in `test/e2e/specs/editor/various/block-custom-css.spec.js` and a unit test for `getStyleOverrides` sorting.

## Contribution

Opened and merged by @ramonjd with review and testing contributions from @andrewserong and @noruzzamans. During review, @noruzzamans identified that switching style variations temporarily broke the cascade in the editor. @ramonjd adjusted the override sorting logic to partition custom CSS last, and dropped an earlier stylesheet dependency approach after discovering it risked resurrecting dequeued styles via `wp_dequeue_style` compatibility issues. The change was backported to WordPress Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
