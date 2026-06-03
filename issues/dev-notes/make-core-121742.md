# Extending Unicode support in email addresses.

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** agulbra
- **Published:** 2026-05-22
- **Tags:** `Proposals`, `charset`, `email`, `unicode`
- **Link:** [https://make.wordpress.org/core/2026/05/22/extending-unicode-support-in-email-addresses-usernames-and-slugs/](https://make.wordpress.org/core/2026/05/22/extending-unicode-support-in-email-addresses-usernames-and-slugs/)

## Summary

WordPress proposes relaxing email validation and storage to support full Unicode (UTF-8) addresses for user accounts, moving away from legacy US-ASCII restrictions. The change aligns with modern email standards and allows users to store localized names in email identifiers, provided the underlying database supports utf8mb4 encoding.

## Impact

- **Plugin & theme developers**: Existing code assuming single-byte US-ASCII emails will require auditing. Functions relying on byte lengths (e.g., `strlen()`) or hex-escaping logic (historically baked into `antispambot()`) will break on multi-byte strings without explicit updates.
- **Hosting & platform teams**: Databases lacking full UTF-8 support (utf8mb4) will fail to save or retrieve Unicode emails. The proposed implementation gates this feature via a filter, necessitating clear system-level messaging to administrators when database encoding is insufficient.
- **Site owners & end users**: No immediate action required upon merge, but administrators may need UI guidance if their hosting environment cannot support UTF-8 identifiers.

## Technical details

The proposal (PR #5237) modifies core email handling by extending `is_email()`, `sanitize_email()`, and `antispambot()` to accept valid UTF-8 sequences. It introduces a WP_Email_Address class for parsing and returning decoded local and domain parts, alongside new filter-targeted functions wp_is_unicode_email() and wp_sanitize_unicode_email() to maintain legacy behavior where needed. Validation is explicitly locked to the browser's <input type="email"> specification to ensure predictable user input handling. A dedicated filter restricts Unicode enforcement exclusively to environments confirmed to use utf8mb4. This targets user account storage and identifier parsing; PHPMailer 6.9’s existing capability to send to Unicode addresses serves as a baseline but is not modified by this PR.

## Contribution

Authored by @agulbra on the Make WordPress Core blog as a formal request for comment (RFC). The technical implementation originated from a contributor challenge sponsored by geoTLD.group and ICANN during WordCamp Vienna, with collaborative design and coding from @akirk, @benniledl, and @dmsnell. PR #5237 was drafted to address these specifications; the proposal remains under review, seeking community feedback on normalization edge cases and character confusables before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
