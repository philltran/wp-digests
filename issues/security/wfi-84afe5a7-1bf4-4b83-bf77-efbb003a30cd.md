# Wishlist Member X <= 3.34.1 - Unauthenticated Account Takeover via 'mergewith' Parameter

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-13
- **CVE:** [CVE-2026-12949](https://www.cve.org/CVERecord?id=CVE-2026-12949)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-640: Weak Password Recovery Mechanism for Forgotten Password
- **Affected:** Wishlist Member (plugin) <= 3.34.1
- **Patched in:** 3.34.2
- **Researchers:** bashu
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/84afe5a7-1bf4-4b83-bf77-efbb003a30cd](https://www.wordfence.com/threat-intel/vulnerabilities/id/84afe5a7-1bf4-4b83-bf77-efbb003a30cd)
- **Usefulness:** 5/5

## Summary

Wishlist Member versions up to 3.34.1 allow an unauthenticated attacker to take over an existing WordPress account by passing an arbitrary user ID in the `mergewith` POST parameter during registration. The plugin's `wpm_register()` function checks the registration cookie only against the `reg` GET parameter and does not verify that `mergewith` points to a temporary or incomplete registrant tied to the current registration transaction. The attacker-controlled value is then used to update the target user's username, password, email, first name, and last name, while WordPress password and email change notifications are suppressed. If the supplied `wpm_id` does not match a membership level, the update payload omits the role key, so `wp_update_user()` preserves the target's existing role, including administrator.

## Impact

- **Site owners**: Update Wishlist Member to the patched release immediately. If the site was exposed, treat user accounts as potentially compromised and rotate credentials.
- **All WordPress users on affected sites**: Accounts are at risk, including administrators, because the flaw can overwrite credentials and preserve existing roles.
- **Hosting and platform teams**: Force plugin updates and review recent user changes, especially username, email, password, and role changes, on sites that ran the vulnerable version.
- **Plugin and theme developers**: No action is required for unrelated plugins or themes. Audit any custom integration that depends on Wishlist Member registration or account-merge behavior.
- **Headless and REST consumers**: No REST route or block API change is described; the exposure is through the plugin's registration flow.

## Technical details

The flaw is in the Wishlist Member registration handler, `wpm_register()`. It validates the registration cookie only against the `reg` GET parameter, then accepts `mergewith` and `wpm_id` from POST without confirming that `mergewith` identifies a temporary or incomplete registrant bound to the current registration transaction. An attacker can therefore supply any existing user ID as `mergewith`. The plugin then updates that user with attacker-controlled values for username, password, email address, first name, and last name; the username is also written via a direct `$wpdb` UPDATE. WordPress password and email change notification emails are explicitly suppressed. When `wpm_id` references a non-existent membership level, no role key is added to the update payload, so `wp_update_user()` leaves the target user's existing role unchanged, allowing an administrator account to remain administrator after takeover.

## Contribution

The record carries no discussion detail beyond the advisory.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
