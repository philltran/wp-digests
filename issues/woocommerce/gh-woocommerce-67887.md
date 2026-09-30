# #67887: Fix term count release version metadata

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @gigitux
- **Labels:** `plugin: woocommerce`
- **Merged:** [`b65d8bc`](https://github.com/woocommerce/woocommerce/commit/b65d8bc8f2af20eae6c07a246038168a042f23f1)
- **Discussion:** [#67887](https://github.com/woocommerce/woocommerce/pull/67887) · 3 comments · 0 reactions
- **Usefulness:** 1/5

## Summary

WooCommerce bumps the release-version metadata for the new `TermCount` service and the deprecation of `WC_Post_Data::recount_terms_for_product_visibility_change()` from 11.1.0 to 11.2.0. The underlying term-count change (PR #67669) was originally planned for 11.0 and then slipped to 11.2, so the `@since`, `@deprecated`, and `wc_deprecated_function()` version strings are corrected to match. No runtime behavior changes.

## Impact

- **Plugin & theme developers:** If you call or hook `WC_Post_Data::recount_terms_for_product_visibility_change()`, the deprecation notice will now report version `11.2.0` instead of `11.1.0`. The replacement is unchanged: term-count consistency is handled by the `TermCount` service.
- **Site owners / hosting:** No action required. There is no functional or data change.
- **Tooling:** Anything that parses deprecation notices or `@since`/`@deprecated` tags for version-based reporting will see 11.2.0.

## Technical details

The diff touches three files and changes only version strings and a changelog entry:

- `plugins/woocommerce/includes/class-wc-post-data.php`: in `recount_terms_for_product_visibility_change()`, the docblock `@deprecated 11.1.0` becomes `11.2.0`, and the call changes accordingly.
- `plugins/woocommerce/src/Internal/TermCount.php`: four `@since 11.1.0` tags (class docblock, `init()`, `handle_set_object_terms()`, and the deleted-relationships handler) become `11.2.0`.
- `plugins/woocommerce/changelog/fix-outofstock-update-since`: new changelog file (`Significance: patch`, `Type: dev`) with the message "Update term count release version metadata."

```php
// before
wc_deprecated_function( __FUNCTION__, '11.1.0' );
// after
wc_deprecated_function( __FUNCTION__, '11.2.0' );
```

The deprecated method's remaining logic (early return unless the taxonomy is `product_visibility`) is untouched.

## Contribution

Opened and merged by @gigitux after the team decided to move the term-count work from the WooCommerce 11.0 plan to the 11.2 release. CodeRabbit rated the merge risk minimal, and the record shows no design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
