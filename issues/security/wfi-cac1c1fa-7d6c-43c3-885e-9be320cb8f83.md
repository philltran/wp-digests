# WordPress Core <= 7.0.2 - Authenticated (Subscriber+) Email Change Confirmation Bypass

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-08
- **CVSS:** 4.3 (Medium)
- **CWE:** CWE-345: Insufficient Verification of Data Authenticity
- **Affected:** WordPress (core) >= 4.7.0 and <= 4.7.33; >= 4.8.0 and <= 4.8.28; >= 4.9.0 and <= 4.9.29; >= 5.0.0 and <= 5.0.25; >= 5.1.0 and <= 5.1.22; >= 5.2.0 and <= 5.2.24; >= 5.3.0 and <= 5.3.21; >= 5.4.0 and <= 5.4.19; >= 5.5.0 and <= 5.5.18; >= 5.6.0 and <= 5.6.17; >= 5.7.0 and <= 5.7.15; >= 5.8.0 and <= 5.8.13; >= 5.9.0 and <= 5.9.13; >= 6.0.0 and <= 6.0.12; >= 6.1.0 and <= 6.1.10; >= 6.2.0 and <= 6.2.9; >= 6.3.0 and <= 6.3.8; >= 6.4.0 and <= 6.4.8; >= 6.5.0 and <= 6.5.8; >= 6.6.0 and <= 6.6.5; >= 6.7.0 and <= 6.7.5; >= 6.8.0 and <= 6.8.6; >= 6.9.0 and <= 6.9.5; >= 7.0.0 and <= 7.0.2
- **Patched in:** 4.7.34, 4.8.29, 4.9.30, 5.0.26, 5.1.23, 5.2.25, 5.3.22, 5.4.20, 5.5.19, 5.6.18, 5.7.16, 5.8.14, 5.9.14, 6.0.13, 6.1.11, 6.2.10, 6.3.9, 6.4.9, 6.5.9, 6.6.6, 6.7.6, 6.8.7, 6.9.6, 7.0.3
- **Researchers:** 0ways
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/cac1c1fa-7d6c-43c3-885e-9be320cb8f83](https://www.wordfence.com/threat-intel/vulnerabilities/id/cac1c1fa-7d6c-43c3-885e-9be320cb8f83)
- **Usefulness:** 4/5

## Summary

WordPress core validated a changed email address inconsistently on user-profile updates, so an authenticated user at Subscriber level or above could set a new address on their own account without completing the ownership-confirmation step core normally requires. Instead of parking the address in a pending state and waiting for the confirmation link to be followed, the affected path wrote the new address to the account directly. The fix makes validation of the submitted email consistent across the profile-update paths, so the confirmation handshake is enforced. It is a self-service bypass — the attacker changes their own account's email, not another user's — which is why it lands at CVSS 4.3 rather than in account-takeover territory.

## Impact

**Site owners**
- Any site with open registration or a large low-privilege user base (membership, LMS, WooCommerce, community) is exposed: a Subscriber can point their account at an address they do not control.
- Sites on automatic background updates for minor releases will already have received the patched version for their branch; verify rather than assume, especially where auto-updates are disabled via `WP_AUTO_UPDATE_CORE`, the `auto_update_core_minor` filter, or a `wp-config.php` constant.
- No configuration change or data migration is required beyond the version bump.

**Plugin & theme developers**
- If your code relies on `user_email` having been proven-owned — email-domain-gated roles or tiers, SSO/identity mapping keyed on email, allowlists, or "verified user" badges — treat that assumption as unreliable on unpatched installs and re-check your own verification rather than inheriting core's.
- Worth re-auditing your own profile-editing surfaces: core's confirmation step lives in the admin profile flow, not inside `wp_update_user()`. A front-end profile editor or custom endpoint that calls `wp_update_user()` with a `user_email` never engaged the confirmation handshake in the first place and still does not after this patch. If you ship one, you own the verification.

**Hosting & platform teams**
- Patched builds exist for every supported branch back to 4.7, so a pinned-version fleet can be brought current without a major-version jump. Reconcile pinned core versions against the patched list per branch.
- Legacy sites pinned below 4.7 receive no fix and should be treated as permanently affected.

**Headless & REST consumers**
- Anything that reads `user_email` off the users endpoint and treats it as a trusted identity signal should be reviewed; an email present on an account is not evidence the account holder controls it.

**No action required** beyond applying the patched release for your branch — there is no API change, deprecation, or removed symbol to accommodate.

## Technical details

The advisory does not publish the patched file, function, or diff, and no proof-of-concept accompanies it — what follows is the core mechanism the described bypass sits on, not a reconstruction of the patch.

WordPress does not commit a changed email address on the spot. When the address on the profile form differs from the stored `user_email`, `send_confirmation_on_profile_email()` (hooked to `personal_options_update` / `edit_user_profile_update`) stores the pending address and a hash in the `_new_email` user meta, mails a confirmation link to the *new* address, and leaves `user_email` untouched. `edit_user()` unsets the submitted email so the subsequent `wp_update_user()` call cannot write it. The account only flips when the `newuseremail` link is followed and the hash in `_new_email` matches, at which point the meta is deleted. `new_user_email_admin_notice()` surfaces the pending state in the admin. The vulnerability is that validation of the submitted address was not applied consistently across the update paths, leaving one where the address reached the account without that `_new_email` round trip.

Behaviourally, before the patch a Subscriber-level user could end up with `user_email` set to an arbitrary address and no pending `_new_email` meta and no confirmation click; after the patch the address is held pending until confirmed.

Worth knowing when auditing your own code: the confirmation step is a property of the admin profile handler, not of the user API underneath it.

```php
// Bypasses core's confirmation by design — the pending-email flow is
// never engaged, so this writes the address immediately.
wp_update_user( array(
    'ID'         => get_current_user_id(),
    'user_email' => $_POST['email'],
) );
```

If you maintain a front-end profile editor, implement the ownership round trip yourself rather than expecting `wp_update_user()` to supply one. Detection on existing installs: look for accounts whose `user_email` changed with no corresponding `_new_email` meta history, and treat email-derived authorization as unverified until re-confirmed.

## Contribution

The record carries no ticket, commit, or discussion detail — only the researcher credit and the per-branch backport list. No public technical write-up or proof-of-concept has been published alongside it.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
