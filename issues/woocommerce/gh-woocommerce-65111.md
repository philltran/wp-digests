# #65111: Add mobile app QR login

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @JorgeMucientes
- **Labels:** `needs: documentation`, `plugin: woocommerce`, `feature highlight`
- **Merged:** [`61c7d27`](https://github.com/woocommerce/woocommerce/commit/61c7d27c143d8c39c0ad1bc5c870be6236bad5b8)
- **Discussion:** [#65111](https://github.com/woocommerce/woocommerce/pull/65111) · 7 comments · 0 reactions

## Summary

WooCommerce introduces a QR-based authentication flow for its mobile app, enabling administrators and shop managers to authorize site-scoped Application Passwords directly from the browser without requiring WordPress.com account linkage or manual credential entry. This reduces onboarding friction for remote store management and secures mobile access through a bounded token exchange. The feature ships as an integrated stack handling token generation, number-matching approval, rate limiting, and fallback states.

## Impact

- **Mobile app & admin users:** Seamless QR login replaces manual password entry; no configuration or migration required.
- **Plugin/Theme Developers:** No direct breaking changes to existing APIs. If your extensions authenticate via WooCommerce REST or Application Passwords, they remain compatible, though new rate limits apply to mobile QR token requests.
- **Hosting & Platform Teams:** New persisted rate limits will trigger on QR login endpoints; ensure infrastructure can handle slight increases in auth traffic. No database migrations or cache flushes required.
- **Action Required:** None. Existing Application Password flows continue to function as before.

## Technical details

The change ships a complete mobile authentication stack within WooCommerce Core:
- **REST Token Lifecycle:** Introduces REST endpoints for QR token generation and exchange, scoped to site-level Application Passwords.
- **Browser Approval Surface:** Renders a standalone login page at `admin.php?page=wc-admin&path=/mobile-app-login` featuring number-matching verification between the browser session and the mobile device.
- **State Management:** Handles success, consumed, revoked, and unavailable states. When Application Passwords are disabled or unsupported for a user, the flow automatically falls back to the existing magic-link path.
- **Security & Limits:** Applies persisted rate limiting across token generation and exchange endpoints to prevent abuse during scanning/approval cycles.
- **Notifications:** Triggers a sign-in notification email upon successful mobile authentication.

## Contribution

Merged by @JorgeMucientes as part of an extended mobile QR code login stack (referencing PRs #64302 through #64591). The PR consolidates the final layer of token exchange logic, UI surfaces, and email notifications into the main branch. The `needs: documentation` label indicates writer documentation is pending post-merge. No rejected alternatives or major design debates were recorded in the provided commit metadata.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
