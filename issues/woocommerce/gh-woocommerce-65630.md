# #65630: Fix PSR-4 upgrade fallback fataling on a foreign loader or torn file

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @vladolaru
- **Labels:** `plugin: woocommerce`
- **Merged:** [`a603110`](https://github.com/woocommerce/woocommerce/commit/a603110fb58e264370db52cc20e9b93410372e44)
- **Discussion:** [#65630](https://github.com/woocommerce/woocommerce/pull/65630) · 8 comments · 0 reactions

## Summary

WooCommerce's PSR-4 fallback autoloader now degrades gracefully instead of fataling when encountering a malformed Composer `ClassLoader` shape or a torn/partially-written class file during in-place upgrades. This resolves an edge-case regression that would kill requests and break defensive `class_exists()` probes, ensuring the bootstrap continues safely while leaving unresolved classes for later retry.

## Impact

['**Plugin & theme developers / Platform teams:** No breaking changes or configuration updates required. The fix only affects error paths during bootstrap or mid-upgrade scenarios where class files may be incomplete or loader contracts are violated.', '**Upgrade workflows:** Defensive probes like `class_exists()` will now return `false` instead of triggering an uncatchable autoload fatal, preventing deployment failures during file-replacement upgrades.', '**Debugging visibility:** Parse/link errors caught during the fallback are surfaced via `error_log()` when `WP_DEBUG` is enabled, replacing silent misses with visible diagnostic output.']

## Technical details

In `plugins/woocommerce/src/Autoloader.php`, the registration flow in `register_woocommerce_psr4_fallback()` was wrapped in a `try/catch (\Throwable $e)` block. Both `self::build_woocommerce_psr4_fallback()` and `$availability_probe->getPrefixesPsr4()` now return `null` on exception rather than propagating an error, aligning with the method's documented contract.

The SPL autoload handler closure was refactored to replace `require_once $file;` with a tracked `include` workflow:
- **Canonicalization:** `realpath($file)` is used to normalize paths for bookkeeping.
- **State tracking:** Two static arrays, `$loaded` and `$attempted`, record per-request execution state. `$loaded` tracks files that resolved cleanly; `$attempted` tracks every path tried by this handler.
- **Retry semantics:** A plain `include` is used because `require_once` marks paths in the engine's inclusion table before compilation, which would poison retry attempts for torn files. Failed includes are deliberately excluded from the `get_included_files()` skip check so they remain retryable once the upgrade completes.
- **Error surfacing:** The `catch (\Throwable $e)` block suppresses fatals, leaves the class unresolved, and logs a diagnostic message via `error_log()` when `WP_DEBUG` is true.

## Contribution

Opened and merged by @vladolaru in PR #65630 (commit `a603110`). The patch addresses regression #65629 introduced in #65338, where the PSR-4 fallback lacked guards around foreign loader shapes and partial file states. The implementation replaced the previous `require_once` path with a stateful `include` loop and expanded `AutoloaderTest.php` with six unit tests pinning degradation, retryability, and redeclaration guards for torn, link-failed, and already-included files.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
