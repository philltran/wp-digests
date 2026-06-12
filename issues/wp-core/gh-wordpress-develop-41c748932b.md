# HTML API: Ensure that code points always encode to UTF-8

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Dennis Snell
- **Committed:** 2026-06-11
- **Commit:** [`41c748932b`](https://github.com/WordPress/wordpress-develop/commit/41c748932b65a01afcaa267e2d0609d7e355c7c1)

## Summary

The HTML API's character decoder now explicitly forces UTF-8 encoding when converting numeric character references to strings, preventing potential corruption or invalid byte sequences on sites configured with non-UTF-8 charsets. This ensures deterministic output regardless of the site's internal encoding settings, addressing an edge case uncovered during fuzz testing.

## Impact

- **Plugin & Theme Developers**: No action required for standard usage. If you directly call `WP_HTML_Decoder::code_point_to_utf8_bytes()` or rely on the HTML decoder's output, results will now consistently be UTF-8 strings.
- **Hosting & Platform Configurations**: Sites running with legacy non-UTF-8 database charsets may see deterministic behavior changes in HTML decoding pipelines, though broader charset handling remains a known limitation per upstream notes.
- **Headless & REST Consumers**: No action required; the fix operates at the internal decoder layer and does not alter REST schema or API contracts.

## Technical details

The change modifies `src/wp-includes/html-api/class-wp-html-decoder.php` within the `WP_HTML_Decoder::code_point_to_utf8_bytes()` method. Previously, it relied on PHP's `mb_chr()` without specifying an encoding, which inherits the current internal encoding and could produce non-UTF-8 byte sequences when a site was configured with a non-UTF-8 charset. The diff updates the call to explicitly pass `'UTF-8'` as the second argument:
```diff
-			$string = mb_chr( $code_point );
+			$string = mb_chr( $code_point, 'UTF-8' );
```
This guarantees that decoded numeric character references always yield valid UTF-8 strings, preventing silent corruption during `read_character_reference()` processing and aligning with the HTML API's encoding contract.

## Contribution

Merged on 2026-06-11 by Dennis Snell (dmsnell) following review from Jon Surrell (jonsurrell). The fix was developed as a direct follow-up to [62424] after fuzz testing revealed that `mb_chr()`'s default encoding behavior could produce invalid byte sequences under non-UTF-8 site configurations. The diff was kept minimal, updating only the single method invocation without introducing additional polyfills or altering the public API contract.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
