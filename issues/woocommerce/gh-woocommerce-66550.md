# #66550: Restore deprecated product gallery image count helper

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @kmanijak
- **Labels:** `plugin: woocommerce`
- **Merged:** [`62cb0fb`](https://github.com/woocommerce/woocommerce/commit/62cb0fbe84a00f5b5158300e7d28451caf59dca5)
- **Discussion:** [#66550](https://github.com/woocommerce/woocommerce/pull/66550) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

WooCommerce restored the deprecated `ProductGalleryUtils::get_product_gallery_image_count()` compatibility method after it was removed during the product gallery videos refactor. The shim prevents fatal errors for external code still calling the old method and emits a deprecation notice pointing to the replacement `get_product_gallery_media_count()`.

## Impact

- **Plugin & theme developers**: Code calling `Automattic\WooCommerce\Blocks\Utils\ProductGalleryUtils::get_product_gallery_image_count()` will no longer fatal. A `wc_deprecated_function` notice will be triggered.
- **Migration required**: Update calls to `get_product_gallery_media_count()` to resolve the deprecation notice and align with the updated API.
- **Site owners / hosting / headless consumers**: No action required; this is a backend compatibility fix with no UI or REST schema changes.

## Technical details

The change adds a static method to `plugins/woocommerce/src/Blocks/Utils/ProductGalleryUtils.php`. The method signature is `get_product_gallery_image_count( $product )`. It immediately calls `wc_deprecated_function( __METHOD__, '11.1.0', 'get_product_gallery_media_count' )` and returns the result of `self::get_product_gallery_media_count( $product )`. This restores the public API surface without altering the underlying media-counting logic, as verified in the merged diff.

## Contribution

Opened by @kmanijak to address regression #66514, the PR was merged as commit `62cb0fb`. It was a straightforward compatibility shim added to resolve a fatal error introduced in PR #65396, with no major design debates or alternative approaches noted in the discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
