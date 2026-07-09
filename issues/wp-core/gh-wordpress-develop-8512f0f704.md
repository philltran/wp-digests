# HTML API: Apply HTML specification change to DOCTYPE parsing.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jon Surrell
- **Committed:** 2026-07-08
- **Commit:** [`8512f0f704`](https://github.com/WordPress/wordpress-develop/commit/8512f0f704af17ca9d256128eef3107b99bab74e)
- **Usefulness:** 3/5

## Summary

The HTML API's DOCTYPE parsing in `WP_HTML_Doctype_Info` was updated to align with the current WHATWG HTML specification, which removed the distinction between missing and empty SYSTEM identifiers when determining document rendering mode. Previously, an empty system identifier triggered limited-quirks mode, whereas now it (alongside a missing identifier) consistently yields quirks mode. This correction ensures WordPress's internal HTML serialization matches web standards and resolves edge-case DOCTYPE handling.

## Impact

- **Theme & Plugin Developers:** No public API changes or deprecations; standard `<!DOCTYPE html>` outputs remain unaffected. Only code explicitly parsing custom or legacy DOCTYPE strings with empty SYSTEM identifiers will see a rendering mode shift from limited-quirks to quirks.
- **Headless & REST Consumers:** HTML serialization endpoints may return marginally different DOCTYPE modes for malformed markup, but no REST schema or block.json changes are required.
- **Hosting & Platform Teams:** No migration steps or configuration changes required. Standard site output is unaffected.

## Technical details

The change modifies `src/wp-includes/html-api/class-wp-html-doctype-info.php` by removing the intermediate `$system_identifier_is_missing = null === $system_identifier;` check and immediately normalizing `$system_identifier` to `''` if `null`. All subsequent mode-determination branches now rely strictly on `'' === $system_identifier` or `'' !== $system_identifier` across four condition blocks matching W3C 4.01 frameset/transitional identifiers. Corresponding inline comments were updated to reflect the spec alignment referenced in whatwg/html#12023. Test expectations in `tests/phpunit/tests/html-api/wpHtmlDoctypeInfo.php` were adjusted: a DOCTYPE containing an empty system identifier now asserts `'quirks'` mode instead of `'limited-quirks'`.

## Contribution

Committed by Jon Surrell on 2026-07-08 as part of PR #12382, addressing Trac ticket #64504. DMSnell provided code review props. The update was a straightforward specification alignment patch with no alternate approaches or design debates noted in the commit or diff; it directly replaces the historical missing-vs-empty distinction with a unified check per the WHATWG HTML issue.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
