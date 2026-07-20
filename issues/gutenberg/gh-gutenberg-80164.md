# #80164: Classic block: Remove migration notice and restore inserter availability

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Classic`
- **Merged:** [`fa48e3b`](https://github.com/WordPress/gutenberg/commit/fa48e3b7c25985e26ef8845a0bc7a135ad1304ce)
- **Discussion:** [#80164](https://github.com/WordPress/gutenberg/pull/80164) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Classic block is now permanently available in the inserter by default, reversing its previous hidden-by-default state. The migration warning notice is removed and replaced with a direct "Convert to blocks" toolbar button. This change simplifies the editing experience and removes the opt-in mechanism that previously gated inserter availability.

## Impact

- **Plugin & theme developers:** The `wp_classic_block_supports_inserter` filter and `wp_declare_classic_block_necessary()` function are removed. Any code relying on these to control Classic block inserter visibility will break.
- **Site owners & editors:** The Classic block will always appear in the inserter. The previous migration warning is gone, replaced by a toolbar button to convert legacy content to blocks.
- **No action required** for most users unless you explicitly used the removed PHP filter/function.

## Technical details

- Deletes `lib/compat/wordpress-7.1/classic-block.php`, removing `wp_declare_classic_block_necessary()` and the `wp_classic_block_supports_inserter` filter.
- Replaces `packages/block-library/src/freeform/migration-notice.js` with `packages/block-library/src/freeform/convert-to-blocks-button.js`, which uses `rawHandler()` and `serialize()` from `@wordpress/blocks` to convert content on click.
- Updates `packages/block-library/src/freeform/edit.js` to render the new button inside `<BlockControls>` instead of the `<MigrationNotice>` component, and removes the `onReplace` prop from the `FreeformEdit` signature.
- Modifies `packages/block-library/src/freeform/index.js` `init()` to call `initBlock( { name, metadata, settings } )` unconditionally, dropping the `inserter: !! window?.__needsClassicBlock` override.
- `packages/block-library/src/index.js` now always pushes the classic block to the registry.

**Before/After (PHP gating logic):**
```php
// Before (removed)
if ( ! (bool) apply_filters( 'wp_classic_block_supports_inserter', false, get_post() ) ) {
    return;
}
wp_add_inline_script( 'wp-block-library', 'window.__needsClassicBlock = true;', 'before' );

// After (always available)
// No PHP gating logic; inserter visibility is controlled purely by block metadata defaults.
```

## Contribution

Opened and merged by @t-hamano (co-authored with @tyxla and @mamaduka) as a manual cherry-pick of #79894 to the `release/23.5` branch after an automated cherry-pick failed. The change was prepared for the Gutenberg 23.5.2 release and merged following standard CI validation.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
