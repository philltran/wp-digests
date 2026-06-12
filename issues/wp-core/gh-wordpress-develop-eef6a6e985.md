# Charset: Replace polyfill wp_has_noncharacters() with direct PCRE version.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Dennis Snell
- **Committed:** 2026-06-11
- **Commit:** [`eef6a6e985`](https://github.com/WordPress/wordpress-develop/commit/eef6a6e9854febe3f5b4433992a2d0dd173bd962)

## Summary

The `wp_has_noncharacters()` utility now uses a single, byte-level PCRE regex instead of branching to a polyfill when Unicode PCRE support is unavailable. This resolves a bug discovered during HTML API fuzzing where malformed UTF-8 sequences previously caused the Unicode regex to fail entirely. By matching noncharacter code points directly via their raw UTF-8 byte representations, the function reliably detects them regardless of surrounding invalid bytes or environment constraints.

## Impact

- **Plugin & theme developers**: No public API surface is affected; `wp_has_noncharacters()` remains a private internal utility.
- **Core maintainers & platform teams**: The internal `_wp_has_noncharacters_fallback()` in `compat-utf8.php` is explicitly deprecated as of 7.1.0 and its implementation has been removed.
- **Action required**: None for external consumers. The function's behavior is now stricter and more robust against malformed input, which may alter internal validation outcomes if downstream code previously relied on the failing Unicode PCRE branch.

## Technical details

- **File**: `src/wp-includes/utf8.php`
- **Change**: The conditional `if ( _wp_can_use_pcre_u() ) : ... else : ... endif;` block is removed. `wp_has_noncharacters()` now executes a direct pattern match:
  ```php
  function wp_has_noncharacters( string $text ): bool {
      return 1 === preg_match(
          '~\xEF(?:\xB7[\x90-\xAF]|\xBF[\xBE\xBF])|(?:\xF0[\x9F\xAF\xBF]|[\xF1-\xF3][\x8F\x9F\xAF\xBF]|\xF4\x8F)\xBF[\xBE\xBF]~x',
          $text
      );
  }
  ```
- **Mechanism**: The regex matches the exact raw UTF-8 byte sequences for U+FDD0–U+FDEF, U+FFFE–U+FFFF, and U+nFFFE/U+nFFFF ranges. This bypasses PCRE's Unicode mode errors when invalid bytes are present in the string.
- **Removal**: In `src/wp-includes/compat-utf8.php`, `_wp_has_noncharacters_fallback()` is marked `@deprecated 7.1.0` and replaced with a simple delegation to the new implementation. Legacy fallback tests were removed from `tests/phpunit/tests/unicode/wpHasNoncharacters.php`, replaced by coverage for invalid UTF-8 edge cases.

## Contribution

Merged on 2026-06-11 by Dennis Snell (`dmsnell`), developed in PR #12148 and discussed in Trac ticket #65372. The change was discovered during fuzzing work targeting the HTML API, which exposed that the previous Unicode PCRE pattern failed when invalid UTF-8 bytes were present in the input string. Rather than maintaining a complex byte-scanning fallback loop, Snell replaced the pattern with a direct raw-byte match, allowing the removal of the polyfill entirely. Props include Jon Surrell (`jonsurrell`).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
