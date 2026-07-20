# #75985: Misc fixes for WordPress-Develop 7.0 merges

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @peterwilsoncc
- **Labels:** `Backwards Compatibility`, `[Type] Code Quality`, `[Package] Block library`, `Backported to WP Core`, `No Core Sync Required`
- **Merged:** [`82f67e2`](https://github.com/WordPress/gutenberg/commit/82f67e2ac32f81d1645a2a41dd991a5cde120ccf)
- **Discussion:** [#75985](https://github.com/WordPress/gutenberg/pull/75985) · 11 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request restores two deprecated navigation block functions to maintain backward compatibility with third-party integrations, corrects `@since` version tags in the navigation and page-list block PHP files, and fixes the single-page admin template to pass the current `$hook_suffix` to the `admin_footer` action instead of an empty string. These changes prevent breakage for legacy code and ensure accurate release metadata for WordPress 7.0.

## Impact

- **Plugin & theme developers**: No immediate action required. The restored functions remain marked `@deprecated 7.0.0` and will trigger `_deprecated_function()` notices. Code relying on them will continue to work but should migrate to the recommended replacements.
- **Custom admin page / headless developers**: The `admin_footer` action in the generated single-page admin template now receives the correct `$hook_suffix` context, which may affect hooks that inspect or filter this parameter.
- **General**: No configuration changes, migrations, or manual updates are required.

## Technical details

- `packages/block-library/src/navigation-link/index.php`: Reintroduces `block_core_navigation_link_build_css_font_sizes( $context )` and `block_core_navigation_link_render_submenu_icon()`. Both functions call `_deprecated_function()` and provide fallback logic or delegate to `block_core_shared_navigation_render_submenu_icon()`.
- `packages/block-library/src/navigation/index.php` & `packages/block-library/src/page-list/index.php`: Updates `@since` docblock tags from `6.9.0` to `7.0.0` to reflect the actual release version.
- `packages/wp-build/templates/page.php.template`: Changes the `admin_footer` action call to pass the correct context:
  ```php
  // Before
  do_action( 'admin_footer', '' );
  // After
  do_action( 'admin_footer', $hook_suffix );
  ```

## Contribution

Opened by @peterwilsoncc as a catch-all for merge-related fixes during the 7.0 cycle. @t-hamano guided the scope toward backward compatibility and `@since` tag corrections, removing a newly added function to keep the PR focused. The PR was co-authored by @westonruter, @youknowriad, @desrosj, and @ramonjd, and merged with plans to backport to 7.0.1 and 7.1 for consistency across the Gutenberg plugin and WordPress Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
