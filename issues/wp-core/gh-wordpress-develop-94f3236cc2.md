# HTML API: Ensure slash in attribute value is not a self-closing flag.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jon Surrell
- **Committed:** 2026-06-30
- **Commit:** [`94f3236cc2`](https://github.com/WordPress/wordpress-develop/commit/94f3236cc2eb1a7c03505843dd53a566d8ab7a45)
- **Usefulness:** 4/5

## Summary

The HTML API in `WP_HTML_Tag_Processor` has been corrected to properly handle trailing slashes inside unquoted attribute values. Previously, markup like `<div id=test/>` incorrectly triggered the self-closing flag logic, treating the `/` as a tag closer rather than part of the `id` attribute value. This fix ensures slash-only or trailing-slash attributes are parsed correctly without corrupting DOM structure or affecting void element expectations.

## Impact

- **Plugin & Theme Developers / Headless Consumers**: If you rely on `WP_HTML_Tag_Processor::has_self_closing_flag()` for unquoted attributes containing trailing slashes, the return value will now accurately reflect only true self-closing syntax (`<tag />`).
- **No action required** for most users. The change is a behavioral correction that aligns parsing with HTML standards and prevents unexpected tree corruption in specific foreign content scenarios.

## Technical details

- **File**: `src/wp-includes/html-api/class-wp-html-tag-processor.php`
- **Mechanism**: Refactors `parse_next_attribute()` to track `/>` as an explicit boolean flag instead of relying on a fragile end-of-token string offset check.
- **Code Change**:
  ```php
  // Before: Fragile offset calculation against raw HTML
  return '/' === $this->html[ $this->token_starts_at + $this->token_length - 2 ];

  // After: Explicit flag tracked during attribute parsing
  if ( '>' === $this->html[ $this->bytes_already_parsed ] ) {
      if ( $skipped_length > 0 && '/' === $this->html[ $this->bytes_already_parsed - 1 ] ) {
          $this->has_self_closing_flag = true;
      }
      return false;
  }
  ```
- Updates `has_self_closing_flag()` to simply return the `$has_self_closing_flag` property. Test suites in `wpHtmlProcessor.php` and `wpHtmlTagProcessor.php` now verify that `<div attr=/>` correctly returns `false`, preserving accurate self-closing detection for void elements.

## Contribution

Developed in GitHub PR #12319 to resolve Trac ticket #65372, this patch was authored by Jon Surrell with review from dmsnell. The implementation shifted from a reactive end-of-token string check to a proactive state flag during the attribute parsing phase, ensuring `/>` is only flagged when it immediately precedes the closing `>`. Merged on 2026-06-30.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
