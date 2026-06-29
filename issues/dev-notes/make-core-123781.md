# Call for Testing: Unicode email addresses.

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Dennis Snell
- **Published:** 2026-06-10
- **Tags:** `General`, `call for testing`, `email`, `unicode`
- **Link:** [https://make.wordpress.org/core/2026/06/10/call-for-testing-unicode-email-addresses/](https://make.wordpress.org/core/2026/06/10/call-for-testing-unicode-email-addresses/)
- **Usefulness:** 4/5

## Summary

WordPress now accepts non-ASCII (Unicode) characters in email local parts, aligning server-side validation with the WHATWG HTML specification. This enhancement is gated behind core filters that can be removed to restore ASCII-only validation if third-party integrations lack UTF-8 mailbox routing support. A new `WP_Email_Address` class provides a structured parser that safely decodes Punycode domains and separates ASCII/Unicode representations for appropriate output contexts.

## Impact

- Plugin & theme developers: `is_email()` and `sanitize_email()` now return valid for UTF-8 local parts (e.g., `grå@example.com`) when the database uses `utf8mb4`. Codebases assuming strict ASCII byte strings will break unless updated or explicitly reverted.
- Integration authors: Third-party services connected via email fields must be verified for UTF-8 support; otherwise, account creation, notifications, or credential syncs may fail.
- Action required: Audit hardcoded validation assumptions, migrate email parsing to `WP_Email_Address::from_string()`, and test mailto link generation with Unicode domains/local parts. Use the provided filter overrides to force ASCII-only validation during third-party transitions.

## Technical details

- Core filters `wp_is_unicode_email` and `wp_sanitize_unicode_email` now attach to the `is_email` and `sanitize_email` hooks respectively, enabling UTF-8 validation when the DB charset is `utf8mb4`.
- The new `WP_Email_Address` class exposes structural parsing via `from_string()`, returning `null` on invalid input. Accessors include `get_ascii_address()` for HTML attributes (preserving Punycode/xn-- domains) and `get_unicode_address()` for text nodes, while automatically decoding domain representations.
- Third-party reversion is possible by removing the new filters and re-adding `wp_is_ascii_email`/`wp_sanitize_ascii_email` at priority `10` to enforce legacy ASCII validation.
- Validation logic now mirrors the WHATWG email spec, synchronizing PHP-side constraints with `<input type=email>` browser behavior.

## Contribution

Following a call for input on ticket #31992, the Unicode email enhancement was merged in changeset [62482]. The implementation focuses on aligning core validation with modern web standards while providing backward-compatible disable hooks. Review by @amykamala and @jorbin shaped the final approach ahead of targeting WordPress 7.1.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
