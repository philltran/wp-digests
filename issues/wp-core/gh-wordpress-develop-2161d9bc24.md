# HTML API: Preserve decoder match length on named-reference miss.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jon Surrell
- **Committed:** 2026-07-02
- **Commit:** [`2161d9bc24`](https://github.com/WordPress/wordpress-develop/commit/2161d9bc24ad9d85a2b5aeb30e32903721b8a670)
- **Usefulness:** 3/5

## Summary

The `WP_HTML_Decoder` previously allowed a pass-by-reference match byte length parameter to be mutated even when no HTML named character reference matched. This commit ensures that failed character reference lookups leave the referenced variable unmodified, preventing inaccurate token offset tracking during internal HTML parsing.

## Impact

- **Plugin & theme developers**: No action required. The `WP_HTML_Decoder` class is part of core's internal HTML API and does not expose new public interfaces.
- **HTML API consumers**: Ensures reliable token positioning and avoids silent offset drift when parsing content containing unrecognized or malformed named character references (e.g., `&bogus;`).
- **Migration/Configuration**: None required.

## Technical details

In `src/wp-includes/html-api/class-wp-html-decoder.php`, the failure guard in `read_character_reference()` was changed from `false === $replacement` to `null === $replacement`. This aligns the early return with `$html5_named_character_references->read_token()`'s actual miss response, ensuring the `$name_length` pass-by-reference variable remains at its initial value (`0`) when no match occurs. A new PHPUnit test verifies this behavior across invalid names and short-name candidates in both `data` and `attribute` contexts.

## Contribution

Author Jon Surrell merged this fix on 2026-07-02 as part of PR #12379, following up on ticket #58281. The patch closes #65372 and corrects the decoder's handling of unmatched named references to keep pass-by-reference length tracking consistent with successful matches. Props include dmsnell and mukesh27.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
