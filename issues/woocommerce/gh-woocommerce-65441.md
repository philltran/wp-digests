# #65441: [dev] Update .ai/skills/woocommerce-performance/options-cache-priming.md

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @kalessil
- **Labels:** `plugin: woocommerce`
- **Merged:** [`7105b03`](https://github.com/woocommerce/woocommerce/commit/7105b03a41ec4c7c84b51d524b7d250164f95a37)
- **Discussion:** [#65441](https://github.com/woocommerce/woocommerce/pull/65441) · 9 comments · 0 reactions

## Summary

This PR updates an internal AI-agent skill doc (`.ai/skills/woocommerce-performance/options-cache-priming.md`) and removes a now-redundant inline comment in the variable product data store. It documents "Pattern 4": passing transient-prefixed option names (`_transient_*`, `_transient_timeout_*`, `_site_transient_*`, `_site_transient_timeout_*`) to `wp_prime_option_caches()` is unsafe on sites running a persistent object cache. Because those transients live in the object cache's `transient` group rather than `wp_options`, priming records each name in core's `notoptions` negative cache permanently, causing unbounded growth that can raise per-request cost on sharded backends. The fix is to guard such priming with `! wp_using_ext_object_cache()`.

## Impact

This is a documentation/guidance change plus a comment tweak — no runtime behavior change ships in this PR (the actual guard was added in the related #65440).

- **Plugin & theme developers (esp. WooCommerce extension authors):** If your code calls `wp_prime_option_caches()` with transient-named keys, wrap it in `! wp_using_ext_object_cache()`. Unguarded priming silently inflates the `notoptions` cache on persistent-cache sites.
- **Hosting & platform teams:** Explains a real-world degradation mode — `notoptions` growth on backends (e.g. sharded Redis) where the key resolves to a single per-request read. No config change required.
- **Site owners:** No action required.

No APIs are deprecated or removed.

## Technical details

The doc adds a new section establishing the anti-pattern and the corrected guard. Core stores transients in the object cache `transient` group when a persistent cache is active, so `wp_prime_option_caches()` (which reads `wp_options`) finds no rows and records each name — `_transient_<name>` and `_transient_timeout_<name>` — in `notoptions`. Those entries never clear because the rows are never created, so `add_option`/`update_option` invalidation never fires.

Correct pattern from the doc:

```php
if ( ! wp_using_ext_object_cache() ) {
    wp_prime_option_caches( $transient_option_names );
}
```

For mixed key lists, the doc instructs splitting the call so regular options are still primed unconditionally:

```php
wp_prime_option_caches( $regular_option_names );
if ( ! wp_using_ext_object_cache() ) {
    wp_prime_option_caches( $transient_option_names );
}
```

The doc adds an audit rule: any `wp_prime_option_caches()` call whose key list contains names beginning with `_transient_`, `_transient_timeout_`, `_site_transient_`, or `_site_transient_timeout_` must be guarded with `! wp_using_ext_object_cache()`.

The only code change is in `plugins/woocommerce/includes/data-stores/class-wc-product-variable-data-store-cpt.php` (`read_product_data()`): a long explanatory comment above the already-guarded `wp_prime_option_caches()` block is collapsed to `// Prime caches to reduce future queries.`. The `! wp_using_ext_object_cache()` guard itself was unchanged here.

## Contribution

Authored and merged by **@kalessil** (merge commit `7105b03`), as a follow-up to feedback on #65440 (which implemented the actual `wp_using_ext_object_cache()` guard and tests). @kalessil explicitly invited **@coderabbitai** to review the skill content; CodeRabbit flagged five precision issues, of which four were incorporated in commit `208637d`: rewording "per variable product" to "per transient name," adding all four transient prefixes to the audit rule, adding explicit mixed-key-list guidance, and adding positive framing for why the guard is safe on persistent-cache sites. The one suggestion declined was a "Common locations to check" subsection, omitted because no known current offenders exist in the codebase.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
