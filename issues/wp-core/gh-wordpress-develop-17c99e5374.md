# HTML API: Preserve raw text contents in serialization.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jon Surrell
- **Committed:** 2026-07-13
- **Commit:** [`17c99e5374`](https://github.com/WordPress/wordpress-develop/commit/17c99e53743755ad0d2bc4336f90e4bda50bd025)
- **Usefulness:** 4/5

## Summary

The WordPress HTML API now correctly preserves the literal contents of `iframe`, `noembed`, `noframes`, and `xmp` elements during serialization instead of stripping them out. This change also enables programmatic modification of these elements' text content via `WP_HTML_Tag_Processor::set_modifiable_text()`. Previously, raw text elements were serialized as empty because the serializer explicitly zeroed out their text nodes, breaking round-trip parsing and regeneration workflows.

## Impact

['Plugin and theme developers using `WP_HTML_Processor` or `WP_HTML_Tag_Processor` to manipulate or regenerate markup will now see raw text element contents preserved and fully modifiable.', 'Behavioral fix, not a deprecated API removal: Code that inadvertently relied on these elements serializing to empty strings will now output their actual contents. No migration or configuration changes are required for standard usage.', 'Headless and REST consumers: HTML returned via the Block Editor API or custom serializers that pass through the HTML API will now correctly serialize raw text element bodies.']

## Technical details

In `src/wp-includes/html-api/class-wp-html-processor.php`, the `$text = ''; break;` block for `IFRAME`, `NOEMBED`, and `NOFRAMES` inside `WP_HTML_Processor::serialize_token()` was removed, allowing the serializer to append raw text as-is. In `src/wp-includes/html-api/class-wp-html-tag-processor.php`, `WP_HTML_Tag_Processor::set_modifiable_text()` now includes case handlers for `IFRAME`, `NOEMBED`, `NOFRAMES`, and `XMP`. The method applies replacements via `$this->lexical_updates['modifiable text'] = new WP_HTML_Text_Replacement(...)`. A validation guard prevents injection of the element's own closing tag (e.g., `</iframe>`), triggering a `_doing_it_wrong()` notice since WP 7.1.0 and returning `false`. The error message for unsupported tags was also simplified to avoid hardcoding the allowlist.

Before (pre-merge):
```php
$processor->set_modifiable_text('<p>raw content</p>'); // Returns false for <iframe>
// serialize_token() outputs: <iframe></iframe>
```

After (merged):
```php
$processor->set_modifiable_text('<p>raw content</p>'); // Returns true
// serialize_token() outputs: <iframe><p>raw content</p></iframe>
```

## Contribution

Developed by Jon Surrell with review input from DMSnell, this commit landed on 2026-07-13 as a follow-up to r62542. It resolves ticket #65372 by correcting the serialization logic in `WP_HTML_Processor` and extending `set_modifiable_text()` to cover additional raw-text elements. The implementation includes guardrails to prevent malformed HTML when replacing raw text with closing tags matching the target element.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
