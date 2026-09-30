# #67669: Fix term counts after removing out-of-stock visibility

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @gigitux
- **Labels:** `plugin: woocommerce`
- **Merged:** [`c4ca65b`](https://github.com/woocommerce/woocommerce/commit/c4ca65bc5c0afe7222dce7bc2eb5f8b0d46a1b73)
- **Discussion:** [#67669](https://github.com/woocommerce/woocommerce/pull/67669) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

WooCommerce now refreshes category, tag, brand, and ancestor term counts when a count-affecting `product_visibility` relationship is deleted, for example when `outofstock` is removed via `wp_remove_object_terms()`. Previously, with "Hide out of stock items from the catalog" enabled, such removals left counts stale. The visible case was converting an out-of-stock simple product to External/Affiliate. The logic moves into a new internal, container-managed `TermCount` service, and the old `WC_Post_Data` hook callback is deprecated.

## Impact

- **Site owners:** With hidden out-of-stock items enabled, category/tag/brand counts now stay correct after an out-of-stock product is converted to external or its `outofstock` term is otherwise removed directly. No action required.
- **Plugin & theme developers:**
  - `WC_Post_Data::recount_terms_for_product_visibility_change()` is deprecated since 11.1.0 and is now a no-op that calls `wc_deprecated_function()`. It is no longer hooked to `set_object_terms` by core. If you unhooked it, called it directly, or re-added it yourself, remove that code. Re-adding it will only emit a deprecation notice and do nothing.
  - `TermCount` is marked `@internal`, so don't depend on it.
  - Code that calls `wp_remove_object_terms()` or `wp_set_object_terms()` on `product_visibility` will now trigger `_wc_recount_terms_by_product()` for count-affecting changes.
- **Hosting & platform:** No configuration or migration. Existing stale counts are not repaired retroactively by this change (a recount such as `wc_recount_all_terms()` or the tools action would be needed); the PR does not state otherwise.
- **Headless & REST consumers:** No API changes.

## Technical details

**New service:** `plugins/woocommerce/src/Internal/TermCount.php`, resolved in `WooCommerce::init_hooks()` via `$container->get( TermCount::class )`. Its `init()` registers two actions:

- `deleted_term_relationships` (priority 10, 3 args) → `handle_deleted_term_relationships( $object_id, $tt_ids, $taxonomy )`
- `set_object_terms` (priority 10, 6 args) → `handle_set_object_terms(...)`

**Behavior:**

- Both handlers bail unless the taxonomy is `product_visibility` and the object ID is a positive integer (`absint`). IDs are normalized with `absint` and zeros are filtered out.
- Count-affecting term taxonomy IDs are `exclude-from-catalog`, plus `outofstock` only when `get_option( 'woocommerce_hide_out_of_stock_items' )` is `'yes'`. They are resolved via `wc_get_product_visibility_term_ids()`.
- If a deleted relationship intersects those IDs, the handler calls `_wc_recount_terms_by_product( $object_id )`.
- `handle_set_object_terms` recounts only for *added* count-affecting terms. It skips when a count-affecting term was removed, because the deletion hook already ran earlier in `wp_set_object_terms()`. This avoids double recounts.

**Deprecation:** in `class-wc-post-data.php`, the `add_action( 'set_object_terms', ..., 'recount_terms_for_product_visibility_change', 10, 6 )` line is removed. The method gains `@deprecated 11.1.0` and a `wc_deprecated_function( __FUNCTION__, '11.1.0' )` call at the top. The diff context shows the remaining body is left in place after that call, so it still executes; the PR summary calls it a "no-op", which the diff doesn't fully show.

**Tests:** `tests/php/src/Internal/TermCountTest.php` counts recounts via the `woocommerce_product_recount_terms` filter. It asserts exactly one recount for direct removal, `wp_set_object_terms` add/remove, `wp_add_object_terms`, and replacement. It also verifies an external-conversion scenario where the category `product_count_product_cat` goes from `0` to `1`.

## Contribution

Opened by @gigitux, who states it supersedes the narrower, path-specific recount approach in #67624 in favor of a central listener on the relationship deletion, so the existing external-conversion path is fixed through its existing `wp_remove_object_terms()` call. The PR notes AI tooling (Codex) assisted with implementation and tests. It was initially targeted at 11.1.0 and auto-backported to `release/11.1` (PR #67885), but later discussion between @gigitux and @kalessil moved the milestone to 11.2; @kalessil asked whether `@since` tags would need updating as a result. Note that the diff still carries `11.1.0` in `@since` and `@deprecated`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
