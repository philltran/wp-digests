# HTML API: Ensure correct serialization of XMP contents.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jon Surrell
- **Committed:** 2026-06-22
- **Commit:** [`bcf665d94c`](https://github.com/WordPress/wordpress-develop/commit/bcf665d94c08dc85ed262dc7bb25c1f3a664b95f)
- **Usefulness:** 3/5

## Summary

Fixes a serialization bug in the HTML API where `<xmp>` element contents were incorrectly escaped as HTML character references. This ensures `WP_HTML_Processor` treats `XMP` as a raw text element, preserving its inner text literally during normalization and serialization as per the HTML specification.

## Impact

- **Plugin & Theme Developers**: No immediate action required unless your code explicitly uses `WP_HTML_Processor` for custom raw-text extraction or sanitization pipelines.
- **Existing `<xmp>` consumers**: If you rely on serialized output from the HTML API to render legacy or imported content, text nodes will now remain unescaped, preventing double-encoded entities (e.g., `&amp;` instead of `&`).

## Technical details

In `src/wp-includes/html-api/class-wp-html-processor.php`, the `serialize_token()` method was updated to include `'XMP'` in the switch case alongside `'SCRIPT'` and `'STYLE'`. This bypasses the default HTML character-reference escaping logic for text nodes within these tags. The change aligns with the HTML standard's generic raw text element parsing algorithm, ensuring text is appended literally except for NULL bytes, which are replaced by U+FFFD replacement characters. Test coverage was added to `tests/phpunit/tests/html-api/wpHtmlProcessor-serialize.php` verifying literal preservation and NULL byte handling.

## Contribution

Committed by Jon Surrell on 2026-06-22, developed alongside PR #12193 and addressing Trac ticket #65372. The patch strictly follows the HTML spec's treatment of raw text elements, adding minimal control flow to `serialize_token()` and corresponding PHPUnit tests for serialization idempotency and NULL byte replacement.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
