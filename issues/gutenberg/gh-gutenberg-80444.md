# #80444: Misc fixes for WordPress-Develop 7.0 merges

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `Backwards Compatibility`, `[Type] Code Quality`, `[Package] Block library`, `Backport to Gutenberg RC`
- **Merged:** [`b252138`](https://github.com/WordPress/gutenberg/commit/b252138119d2caa6f9c3c0069507ad979995129a)
- **Discussion:** [#80444](https://github.com/WordPress/gutenberg/pull/80444) · 1 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request backports changes from #75985 to the `wp/7.1` branch, resolving a changelog merge conflict. It deprecates two internal navigation-link functions and fixes the `admin_footer` action in the single-page admin template to correctly pass the current `$hook_suffix` instead of an empty string.

## Impact

- **Plugin & theme developers**: Direct calls to `block_core_navigation_link_build_css_font_sizes()` and `block_core_navigation_link_render_submenu_icon()` will trigger `_deprecated_function` notices in 7.0+. Migrate to the shared navigation rendering approach or remove these calls.
- **Custom admin page developers**: If you use the Gutenberg single-page admin template, the `admin_footer` action now receives the correct `$hook_suffix`, improving compatibility with plugins that expect a string argument.
- **Site owners & block authors**: No action required.

## Technical details

- In `packages/block-library/src/navigation-link/index.php`, `block_core_navigation_link_build_css_font_sizes()` and `block_core_navigation_link_render_submenu_icon()` are marked `@deprecated 7.0.0`. The latter explicitly recommends `block_core_shared_navigation_render_submenu_icon()` as a replacement.
- In `packages/wp-build/templates/page.php.template`, the `admin_footer` action call was updated:
  ```php
  // Before
  do_action( 'admin_footer', '' );
  // After
  do_action( 'admin_footer', $hook_suffix );
  ```
- Docblock `@since` tags in `packages/block-library/src/navigation/index.php` and `packages/block-library/src/page-list/index.php` were updated from `6.9.0` to `7.0.0`.

## Contribution

Opened and merged by @t-hamano to resolve a changelog conflict during the WordPress-Develop 7.0 merge process. The PR manually backports the changes from #75985 to the `wp/7.1` branch, with no additional design debates or alternative approaches noted in the discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
