# #65332: Deprecate get_block_type

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @dinhtungdu
- **Labels:** `plugin: woocommerce`
- **Merged:** [`3de12e6`](https://github.com/woocommerce/woocommerce/commit/3de12e67bbe4ef821e15e47c42a0cb0d8bda3699)
- **Discussion:** [#65332](https://github.com/woocommerce/woocommerce/pull/65332) · 4 comments · 0 reactions

## Summary

`AbstractBlock::get_block_type()` is deprecated in WooCommerce 10.9.0 in favor of `AbstractBlock::get_full_block_name()`. The two methods were functionally identical — both returned `$this->namespace . '/' . $this->block_name` — so the one with the less descriptive name was retired. Internal call sites in `register_block_type()` and `ProductFilterTaxonomy`'s `data-wp-key` computation have been updated to call the canonical method directly.

## Impact

**Plugin & theme developers / WooCommerce extension authors**
- If your block type class extends `AbstractBlock` and calls `$this->get_block_type()`, replace every call with `$this->get_full_block_name()`. The return value is identical, so no behavioral change occurs.
- Calling the deprecated method will trigger `wc_deprecated_function()` notices (visible with `WP_DEBUG` enabled), but it continues to work — it now delegates to `get_full_block_name()` internally.
- No hook, filter, REST schema, DB, or `block.json` changes.

**Site owners / hosting & platform / headless & REST consumers**
- No action required. Block type strings (e.g. `woocommerce/product-filter-taxonomy`) are unchanged.

## Technical details

Two files changed in `plugins/woocommerce/src/Blocks/BlockTypes/`:

**`AbstractBlock.php`**
- `get_full_block_name()` docblock updated: previously described it as returning the interactivity namespace; now correctly states it returns the full block name including namespace.
- `register_block_type()` (line ~264) switched from `$this->get_block_type()` to `$this->get_full_block_name()`.
- `get_block_type()` body replaced: now calls `wc_deprecated_function( __METHOD__, '10.9.0', 'get_full_block_name' )` then returns `$this->get_full_block_name()` — the original inline expression is gone.

**`ProductFilterTaxonomy.php`**
- `data-wp-key` attribute in the render method switches from `wp_unique_prefixed_id( $this->get_block_type() )` to `wp_unique_prefixed_id( $this->get_full_block_name() )`.

Before/after for subclass usage:
```php
// Before
protected function my_method() {
    $type = $this->get_block_type(); // e.g. "woocommerce/my-block"
}

// After
protected function my_method() {
    $type = $this->get_full_block_name(); // same return value
}
```

## Contribution

Opened and merged by @dinhtungdu. The PR had a brief directional reversal: an earlier comment in the discussion argued for deprecating `get_full_block_name` instead, but @dinhtungdu concluded the opposite — keeping the more descriptive name as canonical — and the change proceeded accordingly. Merged as commit `3de12e6`; no changelog entry was filed since it is an internal developer-facing deprecation only.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
