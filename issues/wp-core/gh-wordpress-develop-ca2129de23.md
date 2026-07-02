# General: Bump the pinned hash for Gutenberg to `v23.3.0`.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jonathan Desrosiers
- **Committed:** 2026-06-30
- **Commit:** [`ca2129de23`](https://github.com/WordPress/wordpress-develop/commit/ca2129de238d5b018b0350171d051e13a88ac829)
- **Usefulness:** 2/5

## Summary

This commit synchronizes WordPress trunk with the Gutenberg `v23.3.0` release, updating the pinned commit SHA and refreshing internal dependency hashes. The bump introduces new optional block schema attributes (`opensInNewTab` and `description` on the Home Link block; `isDecorative` on the Image block), updates lightbox accessibility strings, and adds a backward-compatibility shim for an older navigation submenu icon function.

## Impact

['- **Plugin & theme developers**: New block attributes are registered but currently disabled by default. Developers querying or rendering `home-link` or `image` blocks programmatically should be aware of the added schema definitions (`opensInNewTab`, `description`, `isDecorative`) to avoid stale attribute expectations during serialization.', '- **Platform & core maintainers**: Internal JS package version hashes in `script-loader-packages.php` and `script-modules-packages.php` have been updated. Forking or patching Gutenberg within the WordPress source tree requires a full rebuild of the npm packages to match the new SHAs.']

## Technical details

The diff primarily acts as a dependency sync, but patches specific core files to align with v23.3.0 features. `package.json` pins the Gutenberg SHA to `14db4ab9395a9e96430eed678e4288a59eecbd15`. Hundreds of version hashes are updated across `src/wp-includes/assets/script-loader-packages.php` and `src/wp-includes/assets/script-modules-packages.php`. Block JSON schemas are patched in `home-link/block.json` (adding `opensInNewTab` boolean and `description` string) and `image/block.json` (adding `isDecorative` boolean), with corresponding updates to `blocks-json.php`. The `render_block_core_home_link()` function in `home-link.php` is updated to conditionally output `target="_blank"` and render a `<span class="wp-block-navigation-item__description">` when the new attributes are present. `image.php` changes the lightbox trigger aria-label from `null` to `__( 'Enlarge' )`. A deprecated wrapper is added to `navigation-submenu.php` redirecting `block_core_navigation_submenu_render_submenu_icon()` to `block_core_shared_navigation_render_submenu_icon()`. Build constants in `build/constants.php` are bumped to `23.3.0`.

## Contribution

Merged on 2026-06-30 by Jonathan Desrosiers as a standard dependency sync commit. The change aggregates over 100 upstream Gutenberg PRs spanning UI slot migrations, RTC fixes, dashboard widget rollouts, and React 19 preparation into a single npm snapshot update. No major design debates are reflected in the merge message; it functions as a routine maintenance bump to keep WordPress core aligned with the latest release branch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
