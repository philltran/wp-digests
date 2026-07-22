# #79196: Plugin: Bump minimum required WordPress version to 6.9

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `Gutenberg Plugin`, `[Package] Block library`, `No Core Sync Required`
- **Merged:** [`597c50a`](https://github.com/WordPress/gutenberg/commit/597c50a1eb39d792e197f1c440cb175e6ecd3652)
- **Discussion:** [#79196](https://github.com/WordPress/gutenberg/pull/79196) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg plugin now requires WordPress 6.9 as its minimum version and drops backward-compatibility fallbacks for 6.8. This change removes the `lib/compat/wordpress-6.9/block-bindings.php` shim layer, updates the plugin header and `GUTENBERG_MINIMUM_WP_VERSION` constant, and refreshes the GitHub Actions performance benchmark reference commit. It aligns the plugin with its standard support policy of maintaining the last two major WordPress releases.

## Impact

- **Site owners & administrators**: Must upgrade to WordPress 6.9 or newer to install or update the Gutenberg plugin.
- **Plugin & theme developers**: No direct API changes affect external code. The removed `lib/compat/wordpress-6.9/block-bindings.php` contained internal compatibility shims and helper functions that were never part of the public plugin API.
- **Hosting & platform teams**: No configuration changes required beyond ensuring PHP 7.4+ and WP 6.9+ environments.
- **No action required** for sites already running WordPress 6.9 or later.

## Technical details

- `gutenberg.php`: Updated `Requires at least` header and `GUTENBERG_MINIMUM_WP_VERSION` constant from `6.8` to `6.9`.
- `lib/compat/wordpress-6.9/block-bindings.php`: Deleted. This file previously provided fallback implementations for block bindings that were stabilized in core. It hooked into `block_bindings_supported_attributes`, `block_editor_settings_all`, and `render_block`, and defined internal functions including `gutenberg_block_bindings_render_block`, `gutenberg_get_block_bindings_supported_attributes`, `gutenberg_process_block_bindings`, and `gutenberg_replace_html`.
- `.github/workflows/performance.yml`: Updated the performance reference commit hash from `dae102af1458310b05de3c1281b1654951a729ab` to `28d414f1327652e2b49e784ddc12098768991c62` to maintain accurate CI benchmark baselines.
- Before/after for the constant definition:
  ```php
  // Before
  defined( 'GUTENBERG_MINIMUM_WP_VERSION' ) or define( 'GUTENBERG_MINIMUM_WP_VERSION', '6.8' );
  // After
  defined( 'GUTENBERG_MINIMUM_WP_VERSION' ) or define( 'GUTENBERG_MINIMUM_WP_VERSION', '6.9' );
  ```
- The diff confirms the removal of the compat layer and the constant/header updates, with no new public hooks or filters introduced.

## Contribution

Opened and merged by @t-hamano with review contributions from @adamsilverstein and @mamaduka. The change follows the standard release cadence for the Gutenberg plugin, which maintains support for the two most recent WordPress majors. The PR temporarily enabled previous-WP unit tests in CI to validate the transition, with instructions to revert that flag before merge. No alternative approaches or significant design debates were recorded in the discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
