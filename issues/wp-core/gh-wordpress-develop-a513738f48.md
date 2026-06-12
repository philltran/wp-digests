# HTML API: preserve adjusted foreign attributes on serialization.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Dennis Snell
- **Committed:** 2026-06-11
- **Commit:** [`a513738f48`](https://github.com/WordPress/wordpress-develop/commit/a513738f4860c2bfb5122234d754233c0c6246f7)

## Summary

The HTML API incorrectly serialized adjusted foreign attributes (e.g., SVG’s `xlink:href`) by replacing the namespace colon with a space, producing malformed output containing two separate attributes instead of one correctly prefixed attribute. This commit fixes `WP_HTML_Processor::serialize_token()` to detect space-separated adjusted names and rejoin them with a colon during serialization, ensuring valid SVG and MathML markup is preserved.

## Impact

- **Plugin & theme developers** using `WP_HTML_Processor` or `WP_HTML_Tag_Processor` for parsing/serializing SVG or MathML will now see correctly formatted foreign attributes (e.g., `xlink:href` instead of `xlink href`).
- **No breaking changes**: This is a correctness fix that aligns serialization output with standard XML namespace syntax.
- **Zero code changes required** for existing implementations, though integrations that parse serialized HTML strings expecting space-separated namespaces may need updates.

## Technical details

The change targets `src/wp-includes/html-api/class-wp-html-processor.php` within the `serialize_token()` method. Previously, `$qualified_attribute_name` (which stores space-separated adjusted namespaces like `xlink href` per the adjustment table) was used directly in output logic. The diff introduces:
```php
$serialized_attribute_name = str_replace( ' ', ':', $qualified_attribute_name );
```
and updates the subsequent duplicate-check conditional and string concatenation to reference `$serialized_attribute_name` instead.
A clarifying docblock was also added to `WP_HTML_Tag_Processor::get_qualified_attribute_name()` in `class-wp-html-tag-processor.php`, explicitly documenting that SVG/MathML contexts return space-separated adjusted names while non-adjusted colons remain unchanged. New test cases in `tests/phpunit/tests/html-api/wpHtmlProcessor-serialize.php` verify correct serialization of adjusted and non-adjusted foreign attributes, including duplicate handling.

## Contribution

Opened and merged by Dennis Snell on June 11, 2026 (commit `a513738f`). Developed as pull request #12140 and tracked in Trac ticket #65372. Props jonsurrell for identifying the regression during fuzz testing. The patch is a targeted serialization correction; no alternative implementations were debated, focusing strictly on correcting the space-to-colon substitution introduced by the foreign attribute adjustment table.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
