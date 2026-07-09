# HTML API: Apply deferred byte processing at read interfaces.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jon Surrell
- **Committed:** 2026-07-08
- **Commit:** [`60e382bfa7`](https://github.com/WordPress/wordpress-develop/commit/60e382bfa7cd51130e14b29ea934a07e48379bde)
- **Usefulness:** 4/5

## Summary

The HTML API in `WP_HTML_Tag_Processor` now consistently applies deferred input-stream preprocessing at all read interfaces. This aligns attribute names, tag names, and source document reads with browser behavior by normalizing newlines (CRLF/CR → LF) and replacing NULL bytes (U+0000 → U+FFFD) before decoding or returning values.

## Impact

• **Plugin & Theme Developers**: `get_attribute()`, `get_tag()`, and internal matching in `next_tag()`/`matches()` now consistently apply preprocessing to source document reads. Enqueued values (e.g., via `set_attribute()`) remain unprocessed per spec. If your code relies on exact raw byte extraction from the input string, behavior is now normalized.
• **No action required**: Most existing integrations using standard `WP_HTML_Tag_Processor` methods will continue working without modification; this corrects previously inconsistent read boundaries to match HTML spec expectations.

## Technical details

Modifies `src/wp-includes/html-api/class-wp-html-tag-processor.php`. Introduces a new private method `get_decoded_attribute_value( WP_HTML_Attribute_Token $attribute ): string` that applies newline normalization (`str_replace( "\r\n", "\n" )` and `str_replace( "\r", "\n" )`) and NULL byte replacement (`str_replace( "\x00", "\u{FFFD}" )`) before passing to `WP_HTML_Decoder::decode_attribute()`. 
• `parse_next_attribute()` now applies the NULL replacement before `strtolower()` when generating `$comparable_name`.
• `class_name_updates_to_attributes_updates()` replaces direct manual decoding with the new method.
• `get_attribute()` returns `$this->get_decoded_attribute_value( $attribute )` instead of raw `substr`.
• `get_tag()` now applies the NULL byte replacement to extracted tag names.
• `matches()` refactors name comparison to use `$this->get_tag()` for matching, ensuring queries operate on the replaced alphabet rather than raw source bytes.
Extensive test coverage was added in `tests/phpunit/tests/html-api/wpHtmlTagProcessor-input-preprocessing.php` validating CR/CRLF normalization, NULL handling across tag/attribute/enqueued paths, and strict matching against the preprocessed alphabet.

## Contribution

Committed by `jonsurrell` on 2026-07-08 (ticket #65372, linked to PR #12385). Props to `dmsnell`. The change consolidates scattered deferred byte processing into centralized read interfaces (`get_decoded_attribute_value`, updated `get_tag`, and refactored `matches`), ensuring browser-verified preprocessing behavior across the HTML API without altering how enqueued values are handled.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
