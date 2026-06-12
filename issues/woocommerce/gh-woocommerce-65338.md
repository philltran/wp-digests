# #65338: Add a Composer PSR-4 fallback autoloader to prevent in-place-upgrade class-not-found fatals

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @vladolaru
- **Labels:** `plugin: woocommerce`
- **Merged:** [`3d45970`](https://github.com/woocommerce/woocommerce/commit/3d459701722b0e0b14d76d98a59dba484904b3e1)
- **Discussion:** [#65338](https://github.com/woocommerce/woocommerce/pull/65338) · 8 comments · 0 reactions

## Summary

Resolves "class not found" fatals that occur during WordPress in-place plugin upgrades when a new first-party WooCommerce class is introduced mid-request. The fix registers a Composer PSR-4 autoloader scoped strictly to `Automattic\WooCommerce\` as a lowest-priority SPL fallback, ensuring newly swapped classes resolve correctly even after the primary Jetpack Autoloader's in-memory classmap snapshot becomes stale. This prevents fatal errors when plugins or themes attempt to lazy-load classes that only exist in the upgraded plugin files.

## Impact

- **Plugin & theme developers:** No action required. The appended fallback transparently resolves new `Automattic\WooCommerce\*` classes without interfering with existing autoloading, version coordination, or third-party vendor resolution.
- **Hosting & platform teams:** No configuration or migration steps needed. The bootstrap registration is idempotent and gracefully degrades to a null miss if composer assets are missing or torn.
- **Existing upgrade paths:** Sites upgrading from plugin versions that predate this change (e.g., 10.7 → 10.8) will still experience the initial fatal, as the fix is forward-looking. Subsequent in-place upgrades automatically benefit from the appended fallback.

## Technical details

- Modifies `plugins/woocommerce/src/Autoloader.php` to introduce three bootstrap methods:
  - `Autoloader::build_woocommerce_psr4_fallback()`: Locates `vendor/composer/autoload_psr4.php`, instantiates a `Composer\Autoload\ClassLoader`, and registers only the exact `Automattic\WooCommerce\` prefix (excluding `Vendor\`, `Blueprint\`, and non-runtime prefixes). Returns `null` if composer files are unreadable or malformed.
  - `Autoloader::register_woocommerce_psr4_fallback()`: Appends an SPL autoloader handler via `spl_autoload_register( $handler, true, false )`. The closure is idempotent (cached in `static $registered_handler`) and delegates each miss to a throwaway `ClassLoader` instance to bypass Composer's per-instance `missingClasses` negative cache.
  - `Autoloader::find_scoped_file()`: Resolves classes against the captured PSR-4 map inside a try/catch block, returning `null` for non-matches or degraded states without triggering fatal errors.
- Updates `plugins/woocommerce/woocommerce.php` to invoke `Autoloader::register_woocommerce_psr4_fallback()` during plugin initialization.
- Adds `plugins/woocommerce/tests/php/src/AutoloaderTest.php` validating prefix scoping, negative cache isolation (`test_build_woocommerce_psr4_fallback_is_not_shared`), and end-to-end resolution of mid-request file swaps.
- *Behavioral shift:* Previously, lazy-loading a class introduced mid-upgrade would fail after the Jetpack Autoloader's snapshot missed it. Now, the appended SPL fallback transparently `require`s the newly swapped file on disk while respecting existing autoloader priority and version-coordination boundaries.

## Contribution

Opened and merged by `@vladolaru` (commit `3d45970`). The PR pivoted away from an initial approach that re-hooked `AssetDataRegistry::enqueue_asset_data()` at `PHP_INT_MAX`, which was identified during review as ineffective for constructor-registered callbacks and prone to silent data loss. Following the pivot, the design focused on a root-cause fix: an appended PSR-4 fallback scoped strictly to `src/`. Review rounds addressed Composer's negative cache poisoning gap (switching to per-miss `ClassLoader` instantiation) and tightened namespace scoping to exclude bundled vendor packages. The change was stress-tested in multi-plugin environments before merging.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
