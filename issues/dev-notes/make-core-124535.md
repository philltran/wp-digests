# Responsive block styles and configurable viewports in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Isabel Brison
- **Published:** 2026-08-05
- **Tags:** `General`, `7-1`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/08/05/responsive-block-styles-and-configurable-viewports-in-wordpress-7-1/](https://make.wordpress.org/core/2026/08/05/responsive-block-styles-and-configurable-viewports-in-wordpress-7-1/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 introduces responsive style states for blocks, allowing developers and site builders to define distinct `@mobile` and `@tablet` styles alongside the base desktop style. These responsive rules can be applied globally via `theme.json` or directly on individual block instances, and themes can now customize the breakpoint thresholds using the new `settings.viewport` configuration. This eliminates the need for custom media queries or JavaScript to handle viewport-specific styling in the block editor and on the front end.

## Impact

- **Theme & plugin developers:** Can now define responsive block styles declaratively in `theme.json` or via block attributes without writing custom CSS. Themes can override default breakpoints using `settings.viewport`.
- **Site builders & editors:** Gain direct control over how blocks render on tablet and mobile viewports through the editor’s States dropdown and View menu.
- **Hosting & platform teams:** No immediate action required; existing content and styles render unchanged.
- **Opt-out capability:** Sites can disable responsive style editing via the `responsiveEditingEnabled` editor setting, though pre-saved responsive styles continue to generate and apply media-query CSS on the front end.

## Technical details

Responsive styles are defined using `@mobile` and `@tablet` keys nested under block styles in `theme.json` or within a block’s `style` attribute. The editor generates media-query-scoped CSS and attaches a stable generated class to rendered blocks. Non-layout per-instance declarations are marked `!important` to override default inline styles, while layout values and `blockGap` are scoped via the existing layout support system.

Breakpoints are configurable at the top level of `theme.json`:
```json
"settings": {
  "viewport": {
    "mobile": "30rem",
    "tablet": "45rem"
  }
}
```
Valid values must be non-negative numeric lengths (`px`, `em`, or `rem`). If only one valid value is provided, a single maximum-width query is used; if tablet ≤ mobile, only the mobile breakpoint applies. The editor interface can be disabled via the `block_editor_settings_all` filter:
```php
add_filter( 'block_editor_settings_all', function( $settings ) {
    $settings['responsiveEditingEnabled'] = false;
    return $settings;
} );
```
Pseudo-states can be chained under viewport keys (e.g., `@mobile > :hover`).

## Contribution

The feature was developed and merged for the 7.1 cycle, extending the existing block supports and global styles architecture to handle viewport-based state nesting. The implementation keeps the editor’s device preview and states dropdown synchronized with the configured breakpoints, and ships with the default `responsiveEditingEnabled` toggle active to preserve backward compatibility for existing sites.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
