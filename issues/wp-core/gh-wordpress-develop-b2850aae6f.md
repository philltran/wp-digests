# HTML API: Fixes for issues discovered while fuzzing.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Dennis Snell
- **Committed:** 2026-06-01
- **Commit:** [`b2850aae6f`](https://github.com/WordPress/wordpress-develop/commit/b2850aae6f0dcbe35434089ef152f8fab21522a1)

## Summary

Fuzz-testing against the HTML API surfaced several bugs across four files: out-of-bounds string reads that emitted native PHP warnings, non-idempotent normalization (where `WP_HTML_Processor::normalize()` produced different output on a second pass), incorrect namespace handling in the open-elements stack, and a premature form-pointer reset that allowed a `<form>` opener to survive the first normalization pass but vanish on the second. This patch fixes all discovered cases and adds regression tests for each. The patch was proposed by Codex (OpenAI) and revised by dmsnell.

## Impact

**Plugin & theme developers**
- No API changes, deprecations, or removed symbols. All fixes are behavioral corrections to existing internals.
- If your code calls `WP_HTML_Processor::normalize()` on untrusted or malformed HTML (including HTML containing null bytes, SVG/MathML elements, or edge-case attribute syntax), upgrade to pick up idempotency and safety fixes.
- If your code uses `WP_Token_Map::contains()` or `WP_Token_Map::read_token()` with input that may contain `\x00`, those methods now guard against null bytes instead of silently producing wrong results or native PHP warnings.

**Hosting & platform**
- Sites running with `WP_DEBUG` enabled were susceptible to native PHP `E_WARNING`/`E_NOTICE` emissions from `WP_HTML_Tag_Processor` on certain incomplete HTML inputs (e.g., `<!---`, `<title></titl`). These are now silenced by proper bounds checks.

**No action required** for the vast majority of sites — the fixes are internal to the HTML API parsing and normalization pipeline.

## Technical details

**`src/wp-includes/class-wp-token-map.php`**

- `WP_Token_Map::contains()`: Added an early `return false` when `str_contains( $word, "\x00" )` is true. Null bytes were not previously rejected and could corrupt key lookups.
- `WP_Token_Map::read_token()`: Before extracting the group key with `substr()`, a `strcspn()` call now checks whether any null byte falls within the `$this->key_length` slice starting at `$offset`. If so, returns `null` immediately, preventing the group-string scanner from going off-track.

**`src/wp-includes/html-api/class-wp-html-open-elements.php`**

- `WP_HTML_Open_Elements::after_element_pop()`: The switch that updates precalculated stack state was keying on `$item->node_name` alone, so SVG or MathML elements with names that collide with HTML element names (e.g., `TITLE`) incorrectly triggered HTML-specific state transitions. The fix constructs `$namespaced_name` — either `$item->node_name` for `html` namespace or `"{$item->namespace} {$item->node_name}"` for foreign namespaces — and switches on that instead.

**`src/wp-includes/html-api/class-wp-html-processor.php`**

`next_visitable_token()`: Restructured the early-return condition so that when `$this->step()` returns `false` and `$this->last_error` is set, the method returns `false` rather than falling through to queue processing. Previously, a step error would silently allow execution to continue.

`serialize_token()`: Several fixes in the attribute serialization loop:
- `$qualified_name` for non-HTML tags (returned by `get_qualified_tag_name()`) now has null bytes replaced with `\u{FFFD}` — previously only the HTML-path `$tag_name` received this replacement.
- A `$seen_attribute_names` map deduplicates attributes: if a normalized `$qualified_attribute_name` (after null-byte replacement and `wp_scrub_utf8()`) has already been emitted, the attribute is skipped with `continue`.
- A `$previous_attribute_was_true` boolean guards against the ambiguous case where a boolean (`true`-valued) attribute is immediately followed by a qualified attribute name beginning with `=`. In that case, a synthetic `=""` is inserted to prevent the two tokens from merging in the output.

`step_in_body()` (`-FORM` case): The form pointer (`$this->state->form_element`) is now cleared **after** the validity guard block, not before it. The old sequence cleared the pointer before checking whether the end tag should be ignored (e.g., when a `</form>` appears inside an SVG `<title>` integration point). That caused the pointer to be lost even on ignored tokens, producing non-idempotent normalization — a `<form>` opener visible on the first pass would vanish on the second.

**`src/wp-includes/html-api/class-wp-html-tag-processor.php`**

`skip_rcdata()`: The end-tag bounds check was `( $at + $tag_length ) >= $doc_length`; corrected to `( $at + 2 + $tag_length ) >= $doc_length`, adding the two characters for `</`. The off-by-two allowed a potential out-of-bounds string read when an RCDATA end tag appeared near the very end of the document.

`parse_next_tag()` (abruptly-closed empty comments path): Before indexing `$html[ $closer_at + $span_of_dashes ]`, the code now checks `$doc_length <= $span_of_dashes + $closer_at`. If true, parser state is set to `STATE_INCOMPLETE_INPUT` and the method returns `false`, eliminating the out-of-bounds read that produced a native PHP warning on inputs like `<!---`.

**New tests** cover all discovered cases across `wpHtmlDecoder.php`, `wpHtmlProcessor-serialize.php` (30+ idempotency cases, native-error cases, and stay-supported cases), and `wpHtmlTagProcessor-token-scanning.php` (incomplete-token cases). Tests use `set_error_handler()` to capture and assert absence of native PHP errors, and the `wp_trigger_error_trigger_error` filter to suppress intentional `WP_DEBUG` notices where applicable.

## Contribution

Fuzz testing was conducted against the HTML API to surface edge cases; bugs were reported under Trac ticket #65372. The initial patch was proposed by Codex and subsequently revised by Dennis Snell (`dmsnell`). Development occurred in PR #11982 on `wordpress-develop`. The commit landed on trunk 2026-06-01 as SVN r62439.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
