# HTML API: Ensure carriage returns are serialized in HTML.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jon Surrell
- **Committed:** 2026-07-13
- **Commit:** [`0c5aa8e95a`](https://github.com/WordPress/wordpress-develop/commit/0c5aa8e95a233efa01aa648904c32dada836d025)
- **Usefulness:** 3/5

## Summary

The `WP_HTML_Processor` class now explicitly serializes decoded carriage returns (`\r` / U+000D) as `&#xD;` character references during HTML normalization and output. Previously, the API’s serialization path inconsistently handled these characters, which could cause round-tripped HTML to differ from its source. This ensures that serializing or normalizing an HTML document preserves carriage returns exactly as specified by the HTML parsing standard.

## Impact

- **Plugin & theme developers / custom HTML API users:** No immediate code changes required. If you use `WP_HTML_Processor::normalize()` or extend the HTML API to parse/serialize content, carriage returns will now consistently appear as `&#xD;` in the serialized output rather than being dropped or left as raw control characters.
- **Headless & REST consumers:** No direct impact; the change operates solely within the server-side HTML serialization layer.
- **No action required** for standard theme/plugin development or block registration.

## Technical details

- **File modified:** `src/wp-includes/html-api/class-wp-html-processor.php`
- **Core change:** Replaces inline `htmlspecialchars()` calls within `WP_HTML_Processor::serialize_token()` with a new private static method: `self::escape_text_for_serialization( $text )`.
- **Before:** Text escaping and null-byte replacement were scattered across `serialize_token()`, e.g., `$html .= htmlspecialchars( $this->get_modifiable_text(), ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5, 'UTF-8' );` followed by ad-hoc `str_replace("\x00", "\u{FFFD}", ...)` calls.
- **After:** All escaping funnels through the new method, which applies `htmlspecialchars()` and then normalizes control characters: `$text = str_replace( "\r", "&#xD;", $text );` and `$text = str_replace( "\x00", "\u{FFFD}", $text );`.
- **Scope of application:** The centralized method now handles text nodes, RCDATA elements (`<title>`, `<textarea>`), attribute values, and foreign content serialization, while explicitly excluding RAWTEXT contexts (as noted in the new docblock). Test coverage added in `tests/phpunit/tests/html-api/wpHtmlProcessor-serialize.php` validates behavior across decimal/hex references, raw CR/CRLF inputs, and nested structures.

## Contribution

Submitted in PR #12466 to resolve #65372 and merged on 2026-07-13 by Jon Surrell. The work centered on consolidating HTML text escaping in the `WP_HTML_Processor` serialization path and adding targeted PHPUnit tests to guarantee carriage return preservation. The diff reflects a focused refactoring without noted design alternatives in the upstream discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
