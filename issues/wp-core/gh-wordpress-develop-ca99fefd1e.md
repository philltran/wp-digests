# HTML API: Add HTML processing instruction support.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jon Surrell
- **Committed:** 2026-07-10
- **Commit:** [`ca99fefd1e`](https://github.com/WordPress/wordpress-develop/commit/ca99fefd1e2593af34f4615786756e6c693d8767)
- **Usefulness:** 4/5

## Summary

The HTML API now natively parses, navigates, and serializes HTML processing instructions (`<?...?>`). This aligns `WP_HTML_Tag_Processor` and `WP_HTML_Processor` with updated WHATWG HTML spec tokenization rules, treating valid processing instructions as distinct tokens rather than legacy bogus comments.

## Impact

- **Block & HTML API consumers:** Code traversing or modifying HTML strings with `WP_HTML_Tag_Processor` may now encounter a `#processing-instruction` token type alongside `#comment`, `#doctype`, and `#cdata-section`.
- **Plugin/Theme developers:** No immediate action required unless you are explicitly filtering, stripping, or rewriting `<?...?>` tokens (e.g., in legacy metadata handlers or custom block serialization).
- **Headless & serialization workflows:** Processors will now preserve and output valid processing instruction syntax (`<?target data?>`) instead of mangling them into comments, improving round-trip fidelity for non-standard HTML fragments.

## Technical details

- `WP_HTML_Tag_Processor` introduces the `STATE_PROCESSING_INSTRUCTION` parser state and returns `#processing-instruction` from `get_token_type()`.
- Parsing logic in `parse_next_tag()` validates processing instruction targets: they must start with an ASCII letter or `_`, continue with ASCII alphanumerics, `-`, or `_`, and cannot be the reserved `xml` or `xml-stylesheet` targets (which fall back to `COMMENT_AS_PI_NODE_LOOKALIKE` bogus comments).
- `WP_HTML_Processor::serialize_token()` now handles the `#processing-instruction` case, serializing tokens as `"<?" . $target . " " . $data . "?>"`.
- State transition handlers (`step_initial`, `step_in_head`, `step_in_body`, `step_in_table`, etc.) in `WP_HTML_Processor` now route `#processing-instruction` tokens to `insert_html_element()`, preserving them during DOM-like traversal.
- `get_tag()` was patched to exclude processing instructions from the legacy `IMAGE` → `IMG` tag name rewrite rule by explicitly checking `'#tag' === $this->get_token_type()`.

## Contribution

Merged on 2026-07-10 by Jon Surrell, with props from dmsnell and westonruter. The change closes #65582 and refines #65581. Implementation followed updated HTML parsing state machines, with review focus on target validation constraints and fallback behavior for reserved `xml` targets.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
