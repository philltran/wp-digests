# #65523: Rename Products (Beta) block to Products (Deprecated)

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @Aljullu
- **Labels:** `needs: documentation`, `plugin: woocommerce`, `focus: e2e tests`, `block-type: product-query`
- **Merged:** [`f6bdc5d`](https://github.com/woocommerce/woocommerce/commit/f6bdc5d512fbbc032976bdceacdb888825590356)
- **Discussion:** [#65523](https://github.com/woocommerce/woocommerce/pull/65523) · 5 comments · 1 reactions

## Summary

WooCommerce renames the legacy `woocommerce/product-query` block from "Products (Beta)" to "Products (Deprecated)" to clarify its lifecycle status and steer editors toward the modern Product Collection block. The change also scopes a PHP render filter to core Query blocks only and adds a defensive type check to the editor's allowed-controls hook.

## Impact

- **Site Editors**: No functional breakage. Block names in the inserter, inspector, and migration notices update to "Products (Deprecated)", providing clearer guidance that the block will not progress to stable.
- **Theme/Plugin Developers**: The `ProductQuery` class no longer hooks into the broad `render_block` filter; it now uses `render_block_core/query`. Custom callbacks that previously intercepted this block's markup via generic render interception will see changed execution context and timing. Review filter priorities if you rely on `render_block` to modify WooCommerce query output.
- **Headless & REST Consumers**: Unaffected. Block registration, REST routes, and query parameters remain unchanged.
- **Action Required**: None immediately required, but developers relying on generic `render_block` interception for product queries should update their callbacks to `render_block_core/query` or specifically target the variation.

## Technical details

- **Label & Notice Sync**: `plugins/woocommerce/client/blocks/assets/js/blocks/product-query/variations/product-query.tsx` updates `title: __( 'Products (Beta)' )` to `'Products (Deprecated)'`. Inspector upgrade notices, internal comments, and e2e test assertions are synchronized to match.
- **PHP Render Filter Narrowing**: `plugins/woocommerce/src/Blocks/BlockTypes/ProductQuery.php` changes the primary registration hook from `render_block` to `render_block_core/query`, limiting execution to core Query blocks. This avoids unnecessary evaluation during other block renders.
  ```php
  // Before
  add_filter( 'render_block', array( $this, 'enqueue_styles' ), 10, 2 );
  
  // After
  add_filter( 'render_block_core/query', array( $this, 'enqueue_styles' ), 10, 2 );
  ```
- **Style Enqueue Logic Simplification**: The `enqueue_styles()` method drops the explicit `$block['blockName'] === 'core/query'` check, relying instead on `self::is_woocommerce_variation( $block )` to determine style injection. This reduces conditional branching without changing which blocks receive styles.
- **Defensive JS Guard**: `plugins/woocommerce/client/blocks/assets/js/blocks/product-query/utils.tsx` adds a type validation to `useAllowedControls`: `if ( ! Array.isArray( controls ) ) { return []; }`, preventing runtime crashes when the controls payload is unexpectedly null or non-array.

## Contribution

Opened and merged by @Aljullu. The rename was proposed to prevent false expectations about the block's stability timeline, with reviewers agreeing that explicit deprecation messaging better aligns with WooCommerce's shift toward Product Collection. Minor architectural improvements (hook scoping, defensive array validation) were bundled as low-risk cleanup before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
