# #66036: Fix deprecated AssetDataRegistry argument notice

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @kmanijak
- **Labels:** `plugin: woocommerce`, `developer advisory`
- **Merged:** [`07d836d`](https://github.com/woocommerce/woocommerce/commit/07d836d46076a87bf26178d2ee21ee33ba013048)
- **Discussion:** [#66036](https://github.com/woocommerce/woocommerce/pull/66036) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request corrects a logic bug in WooCommerce's `AssetDataRegistry::add()` and `hydrate_data_from_api_request()` methods where the `$check_key_exists` deprecation notice only fired for truthy values. The fix ensures `wc_deprecated_argument()` triggers whenever the third argument is explicitly passed, regardless of whether it is `true` or `false`. This aligns WooCommerce's internal usage with WordPress deprecation standards and gives developers accurate signal that the parameter must be removed.

## Impact

- **Plugin & theme developers:** If you call `AssetDataRegistry::add()` or `hydrate_data_from_api_request()` with a third argument, you will now consistently receive a `wc_deprecated_argument` notice. Update your code to drop the `$check_key_exists` parameter.
- **Hosting & platform teams:** No impact. This is an internal developer-facing correction to deprecation hygiene.
- **Action required:** Remove the third parameter from any custom invocations of these methods. Existing callers relying on the default `false` behavior will see no functional changes.

## Technical details

The diff modifies `plugins/woocommerce/src/Blocks/Assets/AssetDataRegistry.php` to replace truthy checks with explicit argument counting:

```php
// Before: only fired when $check_key_exists was truthy
if ( $check_key_exists ) {
    wc_deprecated_argument( 'Automattic\WooCommerce\Blocks\Assets\AssetDataRegistry::add()', '8.9', 'The $check_key_exists parameter is no longer used...' );
}

// After: fires whenever the parameter is explicitly provided (count >= 3)
if ( 3 <= func_num_args() ) {
    wc_deprecated_argument( __METHOD__, '8.9', 'The $check_key_exists parameter is no longer used...' );
}
```

Both `add()` and `hydrate_data_from_api_request()` retain the `$check_key_exists = false` signature for backward compatibility, tagged with a `phpcs:ignore` comment to suppress linting noise. Inside `hydrate_data_from_api_request()`, the diff removes `$check_key_exists` from the internal `$this->add( $key, function () { ... } )` call chain, preventing WooCommerce itself from triggering its own deprecation notice during hydration. Updated PHPUnit tests in `plugins/woocommerce/tests/php/src/Blocks/Assets/AssetDataRegistry.php` verify that omitting the argument suppresses the notice, while explicit `true`/`false` calls trigger it.

## Contribution

Open by @kmanijak to address internal issue #46349, this PR was labeled a developer advisory and merged into trunk without design debate. The author replaced the buggy truthy check with `func_num_args()` to guarantee consistent deprecation signaling while preserving the method signature for third-party backward compatibility. Automated CI passed linting and unit tests covering both omitted and explicit argument states.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
