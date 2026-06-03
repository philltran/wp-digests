# #78484: Navigation: Restore block_core_navigation_submenu_render_submenu_icon() as deprecated shim

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ecairol
- **Labels:** `Backwards Compatibility`, `[Type] Regression`, `[Package] Block library`, `First-time Contributor`, `Backport to WP Minor Release`
- **Merged:** [`f59cc26`](https://github.com/WordPress/gutenberg/commit/f59cc26140f79e130178f81f80694a8fd5bbd477)
- **Discussion:** [#78484](https://github.com/WordPress/gutenberg/pull/78484) · 7 comments · 1 reactions

## Summary

Restores `block_core_navigation_submenu_render_submenu_icon()` as a deprecated shim to prevent fatal PHP errors in WordPress 7.0+ caused by its accidental removal. The function now delegates to the new shared helper `block_core_shared_navigation_render_submenu_icon()`, ensuring themes and plugins using the `render_block_core/navigation-submenu` filter continue to operate without breaking.

## Impact

- **Theme & Plugin Developers:** Avoids fatal errors when calling `block_core_navigation_submenu_render_submenu_icon()`. Emits a `_deprecated_function()` notice directing callers to `block_core_shared_navigation_render_submenu_icon()`.
- **Gutenberg Plugin Environment:** The function is namespaced as `gutenberg_block_core_navigation_submenu_render_submenu_icon()` via the build prefix, preserving compatibility alongside Core 7.0+.
- **Action Required:** Migrate custom submenu icon rendering (typically hooked into `render_block_core/navigation-submenu`) to the new shared helper to suppress deprecation warnings in future releases.

## Technical details

- **File Modified:** `packages/block-library/src/navigation-submenu/index.php`
- **Implementation:** Re-adds `block_core_navigation_submenu_render_submenu_icon()` as a wrapper that calls `_deprecated_function( __FUNCTION__, '7.0.0', 'block_core_shared_navigation_render_submenu_icon()' )` and returns its output.
- **Test Coverage:** Introduces `phpunit/blocks/block-core-navigation-submenu-render-submenu-icon-test.php`. The test asserts the `gutenberg_`-prefixed shim matches the shared helper's output and expects a deprecation notice. A version guard (`! function_exists( 'block_core_shared_navigation_render_submenu_icon' )`) skips execution on older WordPress versions where the target helper is absent.
- **Build/Registry Impact:** The PR notes that `IS_WORDPRESS_CORE` build branches skip prefixing, meaning Core 7.0+ registers `block_core_navigation_submenu_render_submenu_icon()` directly, while the Gutenberg plugin registers `gutenberg_block_core_navigation_submenu_render_submenu_icon()`.

## Contribution

Opened by @ecairol to resolve a regression from #74853 that caused fatal PHP errors after syncing to Core 7.0. Co-authored by @ramonjd, @talldan, @tyxla, and @aaronjorbin. During review, CI failures on trunk were addressed by adding a `function_exists` guard to the PHPUnit test, ensuring it skips on older WordPress version matrices. Labeled for backport to the next minor release; merged as commit `f59cc26`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
