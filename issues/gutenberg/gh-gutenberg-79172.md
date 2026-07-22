# #79172: KSES: Allow SVG-specific presentation attributes in safe_style_css

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Feature] Icons`
- **Merged:** [`f905f73`](https://github.com/WordPress/gutenberg/commit/f905f734f42bf1fd04fcc17b28cf4cd0bf637a68)
- **Discussion:** [#79172](https://github.com/WordPress/gutenberg/pull/79172) · 8 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This change extends the `safe_style_css` filter to allowlist SVG presentation attributes, preventing them from being stripped by `wp_kses()` during content sanitization. Previously, `safecss_filter_attr()` only recognized standard CSS properties, causing inline styles on SVG elements (such as `fill`, `stroke`, `transform`, and `clip-path`) to be removed. By appending a comprehensive list of SVG-specific presentation attributes to the filter, developers can now safely use these properties in inline styles without manual sanitization workarounds.

## Impact

- **Plugin & theme developers:** No action required. Inline SVG styles using presentation attributes will now survive `wp_kses()` sanitization out of the box.
- **Headless & REST consumers:** No impact; this change only affects server-side KSES sanitization.
- **Hosting & platform teams:** No configuration changes needed. The compatibility shim is loaded automatically via the Gutenberg plugin.
- **Note:** If you previously extended `safe_style_css` or maintained custom SVG allowlists to bypass this stripping, audit those implementations to avoid duplicate attribute entries.

## Technical details

The diff introduces `lib/compat/wordpress-7.1/kses.php`, which registers a callback on the `safe_style_css` filter:
```php
function gutenberg_add_svg_to_safe_style_css( array $attr ): array {
    $svg_properties = array( /* 60+ attributes including fill, stroke, clip-path, mask, transform, pointer-events, etc. */ );
    return array_unique( array_merge( $attr, $svg_properties ) );
}
add_filter( 'safe_style_css', 'gutenberg_add_svg_to_safe_style_css' );
```
This file is loaded in `lib/load.php` under the WordPress 7.1 compatibility block. The filter executes inside `safecss_filter_attr()` during `wp_kses()` runs, effectively expanding the CSS property allowlist before validation. The implementation uses `array_unique()` to prevent duplicate entries if a theme or plugin has already added overlapping properties.

## Contribution

Opened by @t-hamano to resolve SVG style stripping ahead of upcoming icon registry features. During review, @afercia suggested extracting the logic into a dedicated `wp_kses_svg()` helper for better developer experience, but the author noted a similar approach was already being built for the icon registry and deferred extraction. @westonruter identified several missing presentation attributes (e.g., `transform`, `clip-path`, `mask`); the author addressed these gaps in a follow-up commit before merging. The change was co-authored by @mukeshpanchal27, @westonruter, and @afercia.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
