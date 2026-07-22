# #80508: Icons: Store the sanitized SVG content when registering an icon

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `Backported to WP Core`, `[Feature] Icons`
- **Merged:** [`365e963`](https://github.com/WordPress/gutenberg/commit/365e96352042dcf0e1575075dfd40302ec1d193a)
- **Discussion:** [#80508](https://github.com/WordPress/gutenberg/pull/80508) · 4 comments · 0 reactions
- **Usefulness:** 5/5

## Summary

Fixes a security vulnerability in the icon registry where `wp_register_icon()` sanitized SVG markup but stored the original unsanitized string, causing malicious attributes and scripts to render in the DOM. The change ensures the registry stores the sanitized output instead, preventing cross-site scripting when icons are rendered via `wp_get_icon()`.

## Impact

- **Plugin & theme developers:** If you register icons using `wp_register_icon()` with a `content` property containing raw SVG, the stored value is now sanitized. This is a breaking change for any code that relied on the previous behavior of passing through custom attributes, inline scripts, or non-standard SVG elements.
- **Site owners & platform teams:** No direct action required. The change mitigates an XSS vector in core icon rendering.
- **Migration:** Audit custom icon registrations that inject dynamic or non-standard SVG attributes; they will now be stripped by the sanitization layer.

## Technical details

The diff modifies `lib/class-wp-icons-registry-gutenberg.php` and `lib/compat/wordpress-7.0/class-wp-icons-registry.php`. In both files, the `register()` method previously computed `$sanitized_icon_content` but never reassigned it, leaving `$icon_properties['content']` pointing to the raw input. The fix adds `$icon_properties['content'] = $sanitized_icon_content;` immediately after sanitization. A PHPUnit test (`test_register_icon_sanitizes_content`) verifies that dangerous attributes like `onload` and `<script>` tags are stripped, leaving only safe structural SVG elements.

## Contribution

Opened and merged by @t-hamano, with co-authorship credited to @mamaduka. The vulnerability was identified by @tyxla, who reviewed and approved the patch. The discussion was brief and focused on validating the sanitization behavior rather than debating alternatives. The change was subsequently cherry-picked to the `wp/7.1` branch for inclusion in the next WordPress release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
