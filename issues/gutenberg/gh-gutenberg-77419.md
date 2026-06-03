# #77419: Prevent font-size propagation in Navigation items causing `em` compounding

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @sarthaknagoshe2002
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Navigation Link`
- **Merged:** [`a7c4952`](https://github.com/WordPress/gutenberg/commit/a7c4952a0044cc61e816d070a55dd4ec5873844c)
- **Discussion:** [#77419](https://github.com/WordPress/gutenberg/pull/77419) · 6 comments · 0 reactions

## Summary

The Navigation block's child item blocks — `core/navigation-link`, `core/navigation-submenu`, `core/page-list`, and `core/home-link` — no longer re-apply font-size classes or inline styles to their `<li>` wrapper elements. Previously, the shared helper `block_core_shared_navigation_build_css_font_sizes()` propagated font size from the root `core/navigation` block context down to every child item, causing exponential scaling when relative units were used: a `1.5em` preset would compound to `2.25em`, `3.375em`, and beyond at deeper nesting levels. The fix removes the per-item application entirely and relies on standard CSS inheritance from the root navigation wrapper. The shared helper function and its test file have been deleted from the codebase.

## Impact

**Plugin & theme developers**
- **Breaking removal**: `block_core_shared_navigation_build_css_font_sizes()`, previously defined in `packages/block-library/src/navigation-link/shared/build-css-font-sizes.php`, has been **deleted**. Any code calling this function directly will fatal.
- **CSS selector breakage**: Themes targeting `.has-{slug}-font-size` on `<li>` elements inside Navigation blocks will no longer match — those classes are no longer added to child item wrappers. Only the root `core/navigation` wrapper element retains them.
- **Inline styles removed**: The inline `font-size` style attribute is no longer written to individual navigation item `<li>` elements. Font size now inherits from the root wrapper.
- To restore legacy per-item behavior, hook into the `render_block_core/navigation-link`, `render_block_core/navigation-submenu`, and `render_block_core/home-link` filters with a `WP_HTML_Tag_Processor`-based callback that re-reads `$block->context['fontSize']` / `$block->context['style']['typography']['fontSize']` and writes the class or inline style back onto the `<li>` element (example snippet provided in the PR description).

**Site owners**
- Navigation menus using `em`-based or other relative font-size presets will now render at a consistent size at all nesting depths. No action required for most sites.

## Technical details

**Deleted file**: `packages/block-library/src/navigation-link/shared/build-css-font-sizes.php` — this file defined `block_core_shared_navigation_build_css_font_sizes( $context )`, which read `$context['fontSize']` to emit a `has-{slug}-font-size` class, or `$context['style']['typography']['fontSize']` to emit an inline style via `wp_get_typography_font_size_value()`.

All four render files previously included the helper and called it behind an `IS_GUTENBERG_PLUGIN` guard:

```php
// Before — repeated in navigation-link, navigation-submenu,
//          home-link, and page-list index.php files
require_once __DIR__ . '/navigation-link/shared/build-css-font-sizes.php';

if ( defined( 'IS_GUTENBERG_PLUGIN' ) && IS_GUTENBERG_PLUGIN ) {
    $font_sizes = gutenberg_block_core_shared_navigation_build_css_font_sizes( $block->context );
} else {
    $font_sizes = block_core_shared_navigation_build_css_font_sizes( $block->context );
}
// $font_sizes['css_classes'] merged into $classes
// $font_sizes['inline_styles'] passed as 'style' => ... to get_block_wrapper_attributes()
```

After the change:

```php
// After — navigation-link/index.php
$classes = array(); // no font-size classes
// 'style' key removed from get_block_wrapper_attributes() call entirely
```

Specific per-file changes:
- **`navigation-link/index.php`**: `require_once` removed; `$classes` initialised as `array()` instead of `array_merge( $font_sizes['css_classes'] )`; `'style' => $style_attribute` key dropped from the `get_block_wrapper_attributes()` call.
- **`navigation-submenu/index.php`**: `require_once` removed; `$font_sizes` call and `$style_attribute` assignment removed; `array_merge( $classes, $font_sizes['css_classes'] )` replaced by the plain `$classes` array; `'style'` key dropped from `get_block_wrapper_attributes()`.
- **`home-link/index.php`** and **`page-list/index.php`**: `require_once` removed; `array_merge` on `$colors['css_classes']` and `$font_sizes['css_classes']` simplified to just `$colors['css_classes']`; `$style_attribute` reduced to `$colors['inline_styles']` only.

The context data (`fontSize`, `style.typography.fontSize`) propagated from `core/navigation` still exists on child block instances — it is now simply ignored at render time by the child blocks.

**Deleted test file**: `phpunit/blocks/block-core-shared-navigation-build-css-font-sizes-test.php` (`Block_Core_Shared_Navigation_Build_Css_Font_Sizes_Test`, 167 lines covering named sizes, custom sizes, empty context, and precedence rules) removed alongside the function.

## Contribution

PR #77419 was opened by @sarthaknagoshe2002, initially scoped to `core/navigation-link` and `core/navigation-submenu`. @talldan pointed out that `core/page-list` and `core/home-link` carried the same bug and requested they be included, along with a dev note and a filter snippet to restore legacy behavior. @scruffian noted the original propagation logic was introduced in PR #19127 and was never actually needed below the root wrapper. @tellthemachines's review feedback led to outright deletion of the function rather than the `_deprecated_function()` wrapper approach described in an earlier version of the PR description. The PR was merged at commit `a7c4952` with co-authored credits to @sarthaknagoshe2002, @tellthemachines, @talldan, @scruffian, @sagarjadhav, and @webmandesign.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
