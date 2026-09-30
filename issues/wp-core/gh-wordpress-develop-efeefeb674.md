# HTML API: Allow raw text which cannot close its own element.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jon Surrell
- **Committed:** 2026-09-07
- **Commit:** [`efeefeb674`](https://github.com/WordPress/wordpress-develop/commit/efeefeb67498aa936fe33ac416711df32b53c719)
- **Usefulness:** 3/5

## Summary

`WP_HTML_Tag_Processor::set_modifiable_text()` no longer rejects raw-text content that merely starts with the same characters as a closing (or, for SCRIPT, opening) tag name. Previously any occurrence of `</xmp`, `</iframe`, `<script`, and so on was refused, so harmless text like `</iframely>` or `</scriptx>` triggered `_doing_it_wrong()` and a `false` return. The check is now a regex that requires the tag name to be followed by a character that actually terminates the tag name in the HTML tokenizer (whitespace, `/`, or `>`).

## Impact

- **Plugin & theme developers using the HTML API:** `set_modifiable_text()` now succeeds for text such as `</iframely>`, `</xmp-tag>`, `<scriptish>` or `</scriptx>` in `IFRAME`, `NOEMBED`, `NOFRAMES`, `XMP` and non-JS `SCRIPT` elements. Code that previously worked around the false-positive rejection can be simplified.
- **Stricter in one place:** text containing `</xmp/>`, `</xmp\f>` or `</script\r>` is now rejected. The old check caught these via substring match anyway, so there is no regression, but tests now lock the behavior in.
- **Still rejected:** text with a real terminator after the tag name, e.g. `</script>`, `<script sneaky>`, `</script sneaky>`. Callers remain responsible for content-type-appropriate escaping.
- **Edge case:** text that *ends* in `</xmp`, `<script` or `</script` (no following character) is now allowed. The added tests show the result is `<xmp>Trailing </xmp</xmp>`, and the re-parse test asserts the text is preserved.
- No action required for most sites.

## Technical details

The change is in `src/wp-includes/html-api/class-wp-html-tag-processor.php`, in `set_modifiable_text()`.

**Non-JS SCRIPT branch** (before/after):

```php
// Before
if (
	false !== stripos( $plaintext_content, '<script' ) ||
	false !== stripos( $plaintext_content, '</script' )
) { ... }

// After
if ( 1 === preg_match( '~</?script[ \t\f\r\n/>]~i', $plaintext_content ) ) { ... }
```

**IFRAME / NOEMBED / NOFRAMES / XMP branch:**

```php
// Before
if ( false !== stripos( $plaintext_content, "</{$tag_name}" ) ) { ... }

// After
if ( 1 === preg_match( '~</' . preg_quote( $tag_name, '~' ) . '[ \t\f\r\n/>]~i', $plaintext_content ) ) { ... }
```

The character class mirrors the spec states where a tag name ends (`script-data-end-tag-name-state`, `script-data-double-escape-start-state`, `rawtext-end-tag-name-state`). The comment notes that a `<script` start tag after `<!--` enters the double-escaped states, where a later `</script>` no longer closes the element, which is why both start and end tags are checked for SCRIPT.

The error messages and `_doing_it_wrong()` calls are unchanged. Tests in `wpHtmlTagProcessorModifiableText.php` add a new `test_allows_raw_text_which_cannot_close_its_element` with provider `data_raw_text_resembling_a_closing_tag`, which also re-parses the output to verify round-tripping. The rejection provider gains cases for `</xmp/>`, `</xmp\f>` and `</script\r>`.

## Contribution

Developed in PR #12914 and committed to trunk (r63513) by Jon Surrell, fixing Trac #65824, with props to khokansardar and shailu25. The record carries no discussion detail beyond that.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
