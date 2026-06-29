# HTML API: Prevent HTML newline normalization on foreign elements.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jon Surrell
- **Committed:** 2026-06-29
- **Commit:** [`23a6bbf248`](https://github.com/WordPress/wordpress-develop/commit/23a6bbf248a826ae213a012e91441c4e29d5f34c)
- **Usefulness:** 3/5

## Summary

The HTML API now prevents HTML-specific leading newline normalization from being applied to foreign elements, ensuring that `<textarea>` inside SVG or MathML contexts retains its original whitespace. This aligns the processor's serialization behavior with HTML parsing specifications by respecting document namespace boundaries during token handling.

## Impact

- **Plugin & theme developers**: No action required unless you are manually constructing, serializing, or normalizing mixed-namespace HTML strings via `WP_HTML_Processor`. The change corrects whitespace preservation for foreign `<textarea>` elements.
- **Core/Platform teams**: Ensures `WP_HTML_Processor::normalize()` adheres to spec-compliant namespace handling. No configuration migrations or fallback requirements needed.

## Technical details

In `src/wp-includes/html-api/class-wp-html-processor.php`, the `serialize_token()` method previously unconditionally prepended a leading `\n` when encountering `TEXTAREA`, `PRE`, or `LISTING` elements to counteract parser newline stripping. The commit adds an `$in_html` guard: `if ( $in_html && ( 'TEXTAREA' === $tag_name || 'PRE' === $tag_name || 'LISTING' === $tag_name ) )`. This restricts the HTML-spec behavior to the native HTML namespace only. Test assertions in `tests/phpunit/tests/html-api/wpHtmlProcessor-serialize.php` were tightened to verify byte-exact serialization (`assertSame`) alongside semantic equality, and new data provider cases for `<math>` and `<svg>` descendants were added to confirm foreign elements bypass the newline injection.

## Contribution

Opened as PR #12322 and merged by Jon Surrell on 2026-06-29. The change follows up on prior normalization work (#61747, ticket #65372) to correctly handle namespace boundaries in the HTML API. The provided metadata indicates a straightforward spec-compliance patch without recorded design debates; the final diff shipped with tightened test assertions and added foreign-element coverage.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
