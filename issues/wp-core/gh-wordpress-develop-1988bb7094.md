# General: Bump the pinned hash for Gutenberg to `e73c3c4`.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Aki Hamano
- **Committed:** 2026-07-22
- **Commit:** [`1988bb7094`](https://github.com/WordPress/wordpress-develop/commit/1988bb7094554e1d9da43809695a09051f6c8a37)
- **Usefulness:** 4/5

## Summary

This commit updates the pinned Gutenberg repository hash in WordPress core, pulling in a batch of editor and block improvements. The most notable outward-facing change is the deprecation of two navigation block helper functions (`block_core_navigation_link_build_css_font_sizes` and `block_core_navigation_link_render_submenu_icon`) in favor of updated internal rendering logic. It also refactors how the Navigation block applies layout and state classes to inner containers, introduces viewport-aware custom property generation for responsive navigation layouts, and fixes several client-side media upload and editor UI behaviors.

## Impact

- **Plugin & theme developers:** Two navigation block helper functions are deprecated in 7.0.0. If your code calls `block_core_navigation_link_build_css_font_sizes()` or `block_core_navigation_link_render_submenu_icon()`, you must migrate to the new internal APIs or remove the calls.
- **Block developers:** The Navigation block's rendering filter `render_block` now uses `block_core_navigation_add_support_classes_to_container()` instead of the previous state-class-only function. Custom code hooking into `render_block` for navigation should account for the new layout custom property generation and scoping classes.
- **Site owners & editors:** Fixes for animated GIF uploads, HEIC upload snackbar double-counting in Safari, and responsive navigation layout styles will improve editor stability and media handling. No manual migration is required.
- **Hosting & platform:** No configuration changes needed. Standard WordPress upgrade applies.

## Technical details

The diff updates `package.json` to point to Gutenberg commit `4997026b75c922d8a6f77a03d72ed7cad04c7073`, which cascades into updated version hashes across `src/wp-includes/assets/script-loader-packages.php` and `src/wp-includes/assets/script-modules-packages.php`. In `src/wp-includes/blocks/navigation-link.php`, `block_core_navigation_link_build_css_font_sizes()` and `block_core_navigation_link_render_submenu_icon()` are marked `@deprecated 7.0.0` and now call `_deprecated_function()`. `src/wp-includes/blocks/navigation.php` introduces `block_core_navigation_get_layout_custom_property_declarations()` (7.1.0) to compute CSS custom properties (`--navigation-layout-justification-setting`, `--navigation-layout-direction`, etc.) based on layout configurations. The existing `block_core_navigation_add_state_class_to_container()` function is renamed to `block_core_navigation_add_support_classes_to_container()` and refactored to iterate through `WP_Theme_JSON_Gutenberg::get_viewport_media_queries()` (or `WP_Theme_JSON`), generating scoped CSS rules via `wp_style_engine_get_stylesheet_from_css_rules()` and applying a unique layout class to inner containers. The `render_block` filter is updated to point to the renamed function. `src/wp-includes/blocks/post-featured-image.php` now checks `if ( 'auto' !== $attributes['aspectRatio'] )` before injecting `aspect-ratio` styles, preventing unwanted intrinsic ratio overrides. Admin page builds (`font-library/page.php`, `options-connectors/page.php`) change `do_action( 'admin_footer', '' )` to `do_action( 'admin_footer', $hook_suffix )` to pass the correct hook suffix. The compat overlay slot in `connectors-home/content.js` now calls `ensureSlotIsAccessible()` to explicitly set `aria-hidden="false"`, resolving screen reader visibility issues.

## Contribution

The commit record carries no public design debate or alternative approaches; it follows the standard Gutenberg hash bump workflow, aggregating upstream PRs that were developed and reviewed independently before being synced to core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
