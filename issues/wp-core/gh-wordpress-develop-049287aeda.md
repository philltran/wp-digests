# HTML API: Escape syntax characters in RCDATA.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Dennis Snell
- **Committed:** 2026-08-31
- **Commit:** [`049287aeda`](https://github.com/WordPress/wordpress-develop/commit/049287aeda8855d6c29af747839377d7669a9fe6)
- **Usefulness:** 3/5

## Summary

`WP_HTML_Tag_Processor::set_modifiable_text()` now escapes `<`, `&`, and `>` in the text it writes into `TITLE` and `TEXTAREA` (RCDATA) elements. Previously only a literal closing tag matching the element name (e.g. `</title`) was neutralized. Escaping is not required by the HTML spec for RCDATA, but it keeps weaker downstream parsers such as `DOMDocument` from misreading plaintext like `<img>` as child markup.

## Impact

**Plugin & theme developers**
- Output of `set_modifiable_text()` on `TITLE`/`TEXTAREA` changes byte-for-byte: `the <img> is text` is now serialized as `the &lt;img&gt; is text`. The parsed result under a spec-compliant HTML parser is unchanged.
- Code or tests that compare `get_updated_html()` as a raw string against unescaped RCDATA content will need updating. Prefer semantic HTML comparison (the core test switched to `assertEqualHTML`).
- Code that passes already-escaped text (e.g. `&amp;`) will see it escaped again (`&amp;amp;`), consistent with the 6.9.0 behavior of escaping all character references rather than avoiding double-escaping.

**Site owners / hosting / headless consumers**
- No action required. Output is more robust for any pipeline that re-parses the HTML with a non-HTML5 parser (libxml/`DOMDocument`, regex-based tools).

## Technical details

The change is in the `TEXTAREA`/`TITLE` case of `WP_HTML_Tag_Processor::set_modifiable_text()` in `src/wp-includes/html-api/class-wp-html-tag-processor.php`.

Before, a `preg_replace_callback()` only rewrote `</TAGNAME` (case-insensitive) to `&lt;/TAGNAME`. Now a `strtr()` escapes the full syntax set:

```php
$plaintext_content = strtr(
	$plaintext_content,
	array(
		'<' => '&lt;',
		'&' => '&amp;',
		'>' => '&gt;',
	)
);
```

A `@since 7.2.0` line is added to the method docblock, and one nearby `/*` comment is changed to `/**`. No new hooks, filters, or public signatures.

Tests in `wpHtmlTagProcessorModifiableText.php`: a new `test_escapes_rcdata_content()` (data provider `data_rcdata_element_names()` for `TEXTAREA` and `TITLE`) sets `the <img> is text`, asserts the output is HTML-equivalent to the input, asserts `<img>` no longer appears in the output, and, if `DOMDocument` is available, checks that the first child node's value equals the original text. `test_updates_basic_modifiable_text_on_supported_nodes` moves from `assertSame` to `assertEqualHTML`.

## Contribution

Developed in PR #13327 and discussed in Trac #65984, committed by dmsnell with props to jonsurrell and westonruter. The record contains no further discussion detail about alternatives considered.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
