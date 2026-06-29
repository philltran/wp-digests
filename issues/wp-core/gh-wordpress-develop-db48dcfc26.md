# HTML API: Replace locale-dependent ctype check in HTML decoder.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jon Surrell
- **Committed:** 2026-06-29
- **Commit:** [`db48dcfc26`](https://github.com/WordPress/wordpress-develop/commit/db48dcfc2636fe84c913cadfc5ba1ed94dea21b7)
- **Usefulness:** 4/5

## Summary

Replaces a locale-dependent `ctype_alnum()` check in `WP_HTML_Decoder::read_character_reference()` with explicit ASCII byte-range comparisons. This fixes a parser edge-case where high-bit UTF-8 bytes were incorrectly treated as alphanumerics in certain locales, ensuring consistent decoding of legacy semicolonless character references across all PHP environments.

## Impact

- Plugin and theme developers using `WP_HTML_Decoder` will experience predictable attribute parsing regardless of server locale (e.g., `en_US.UTF-8` vs `C`).
- No immediate migration required; existing HTML API consumers gain reliable Unicode handling for attributes containing legacy character references.
- Headless/REST pipelines relying on accurate HTML decoding should expect consistent output where trailing characters previously triggered locale-specific branching.

## Technical details

In `src/wp-includes/html-api/class-wp-html-decoder.php`, the `read_character_reference()` method previously used `ctype_alnum( $text[ $after_name ] )` to determine if a named character reference (matched without a trailing semicolon) should be left as literal text when followed by another character. The unified diff replaces this with explicit byte checks against ASCII ranges:

```php
$follower_byte = ord( $text[ $after_name ] );
if (
    0x3D === $follower_byte || // EQUALS SIGN
    ( $follower_byte >= 0x30 && $follower_byte <= 0x39 ) || // ASCII digits 0-9
    ( $follower_byte >= 0x41 && $follower_byte <= 0x5A ) || // ASCII upper alpha A-Z
    ( $follower_byte >= 0x61 && $follower_byte <= 0x7A )     // ASCII lower alpha a-z
) {
    return null;
}
```

This bypasses PHP's locale-sensitive `ctype` extension, which can classify high-bit bytes as alphabetic in certain UTF-8 locales. The corresponding test suite (`wpHtmlDecoder.php`) was expanded to explicitly swap `LC_CTYPE` locales and verify that non-ASCII UTF-8 followers do not trigger ambiguous literal-flush behavior.

## Contribution

Developed by `jonsurrell` with props from `dmsnell`, this change was merged on 2026-06-29. It resolves ticket #65372, addressing long-standing inconsistencies in the HTML API's decoder when parsing legacy character references across different PHP environments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
