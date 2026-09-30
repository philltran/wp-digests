# Paytium: Mollie payment forms & donations <= 5.0.3 - Unauthenticated Privilege Escalation via 'pt_form_field[pt-user-role]' Parameter

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-09-23
- **CVE:** [CVE-2026-18467](https://www.cve.org/CVERecord?id=CVE-2026-18467)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-269: Improper Privilege Management
- **Affected:** Paytium: Mollie payment forms & donations (plugin) <= 5.0.3
- **Patched in:** 5.0.4
- **Researchers:** Aydan Arabadzha
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/caa1a42a-6bb2-42c8-aae7-fb1c895573a8](https://www.wordfence.com/threat-intel/vulnerabilities/id/caa1a42a-6bb2-42c8-aae7-fb1c895573a8)
- **Usefulness:** 5/5

## Summary

Paytium <= 5.0.3 allows unauthenticated attackers to create an administrator account. The 5.0.3 fix added a `wp_hash()`/`hash_equals()` signature check on the `pt-paytium-user-data` field, but a second filter, `pt_cf_checkout_meta()` on the `pt_meta_values` hook, still copies every `$_POST['pt_form_field'][*]` key into the payment meta without verification. A forged `pt-user-role` value therefore overwrites the signed builder's output and is later passed to `wp_insert_user()` as the role. Version 5.0.4 is the patched release.

## Impact

**Site owners**
- Any site running Paytium <= 5.0.3 with a public `[paytium]` shortcode form is exposed to full site takeover (CVSS 9.8). Update to 5.0.4 immediately.
- Sites that updated to 5.0.3 believing the earlier issue was fixed are still vulnerable.
- Audit the user list for unexpected administrator accounts created via Paytium payment flows, and review lost-password activity on those accounts. Remove any unrecognised administrators.

**Plugin & theme developers / agencies**
- Check client sites for the plugin and its version, including staging and multisite installs.
- If you extend Paytium through the `pt_meta_values` hook, be aware that filter ordering affected which values won.

**Hosting & platform teams**
- Consider a virtual patch or WAF rule for requests carrying `pt_form_field[pt-user-role]` until sites are updated.

No REST or headless-specific action is indicated beyond updating.

## Technical details

The record describes a fix that was incomplete. Version 5.0.3 added a signature gate (`wp_hash()` compared with `hash_equals()`) around the `pt-paytium-user-data` field. However:

1. `pt_cf_checkout_meta()`, registered on `pt_meta_values` **after** the signed builder, copies every `$_POST['pt_form_field'][*]` key verbatim into the payment meta array with no signature verification.
2. A submitted `pt_form_field[pt-user-role]` value overwrites the role produced by the signed path.
3. The value is persisted as `_pt-user-role` post meta.
4. After the payment flow completes, `paytium_user_data_processing()` reads `_pt-user-role` and passes it directly as the `role` argument to `wp_insert_user()`.

The attacker supplies their own email address, submits a payment via a public `[paytium]` form, completes the payment flow, and then uses the standard lost-password flow to take over the new administrator account. This is classed as CWE-269 (improper privilege management).

The advisory does not describe the exact 5.0.4 code change, so it is not stated here. A sound fix would restrict which meta keys can be set from user input, or validate the role server-side.

## Contribution

The record carries no discussion detail beyond noting that this vulnerability persists after the 5.0.3 patch for the same field.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
