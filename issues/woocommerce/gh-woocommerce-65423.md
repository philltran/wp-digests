# #65423: Fix SKU duplication corrupting hyphenated SKUs

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @faisalahammad
- **Labels:** `needs: documentation`, `plugin: woocommerce`, `type: community contribution`
- **Merged:** [`ae4f5b7`](https://github.com/woocommerce/woocommerce/commit/ae4f5b784de74c2da779e1dcfd4055af861dbf51)
- **Discussion:** [#65423](https://github.com/woocommerce/woocommerce/pull/65423) · 7 comments · 0 reactions

## Summary

Fixes a regression in WooCommerce product duplication where hyphenated SKUs were incorrectly modified instead of having an incremental suffix appended. Duplicating a product with SKU `SKU-123` now correctly produces `SKU-123-1`, rather than erroneously stripping the trailing number and incrementing it to `SKU-124`. This bug was introduced in WooCommerce 10.5.0 via PR #62628.

## Impact

- **WooCommerce admins & plugin developers:** The admin product duplicate action now preserves the full original SKU and appends `-1`, `-2`, etc. If custom scripts or third-party integrations relied on the previous `SKU-124` output, they will need to adjust expectations or update their duplication logic.
- **Hook consumers:** The `wc_product_pre_has_unique_sku` and `wc_product_has_unique_sku` filters are now invoked with the correctly formed candidate SKU. The fallback path explicitly delegates to `wc_product_generate_unique_sku()` to maintain filter consistency across all branches.
- No action required for standard workflows; existing unit tests cover the corrected behavior.

## Technical details

- **File/Method:** `plugins/woocommerce/includes/admin/class-wc-admin-duplicate-product.php` → `WC_Admin_Duplicate_Product::generate_unique_sku( $product )`
- **Core Logic Shift:** Replaced the regex-based suffix stripping (`$root_sku = preg_replace( '/-[0-9]+$/', '', $product->get_sku() )`) with `$original_sku = $product->get_sku()`. The method now queries existing SKUs matching `LIKE $wpdb->esc_like( $original_sku ) . '-%'` and uses `stripos()` for case-insensitive prefix matching, aligning PHP logic with MySQL's default `LIKE` collation.
- **Suffix Calculation & Filters:** Extracts the maximum pure-numeric suffix from matches, proposes `$original_sku . '-' . ( $max_suffix + 1 )`, and routes it through `wc_product_pre_has_unique_sku` and `wc_product_has_unique_sku`. If rejected, it falls back to `wc_product_generate_unique_sku( $product_id, $original_sku, $max_suffix + 1 )`.
- **Test Coverage:** Added `test_duplicate_product_with_hyphenated_sku_preserves_original()` and `test_duplicate_product_handles_case_insensitive_sku_conflict()` to `tests/php/includes/admin/class-wc-admin-duplicate-product-test.php`.

## Contribution

Authored by @faisalahammad and merged as `ae4f5b7`. The review cycle addressed alignment between PHP prefix matching and MySQL's case-insensitive `LIKE` behavior (switching `strpos` to `stripos`), refined the fallback logic to start from `$max_suffix + 1` to prevent collisions with trashed products, and ensured filter hooks are consistently applied. Comprehensive unit tests were added to cover hyphenated SKUs, sequential duplicates, and case-insensitive SKU conflicts.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
