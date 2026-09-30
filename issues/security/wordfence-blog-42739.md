# 400,000 WordPress Sites Affected by Account Takeover Vulnerability in TranslatePress WordPress Plugin

- **Source:** Wordfence Blog
- **Type:** Blog post
- **Author:** unknown
- **Published:** 2026-08-25
- **Tags:** `Research`, `Vulnerabilities`, `WordPress Security`, `WordPress Plugin Vulnerability News August 2026`, `WordPress Vulnerability News August 2026`
- **Link:** [https://www.wordfence.com/blog/2026/08/400000-wordpress-sites-affected-by-account-takeover-vulnerability-in-translatepress-wordpress-plugin/](https://www.wordfence.com/blog/2026/08/400000-wordpress-sites-affected-by-account-takeover-vulnerability-in-translatepress-wordpress-plugin/)
- **Usefulness:** 5/5

## Summary

TranslatePress – Multilingual (`translatepress-multilingual`) versions up to and including 3.3.1 leak an administrator's password reset key to unauthenticated visitors. An attacker can use the leaked key to reset the administrator's password and log in, which gives complete site takeover. The leak only occurs when the target administrator's profile language is set to a published secondary language. The vendor fixed it in 3.3.2.

## Impact

**Site owners**
- Update TranslatePress to 3.3.2 or later immediately. The issue is rated CVSS 9.8 (Critical) and requires no authentication.
- Sites are exposed only if an administrator's profile language is set to a published secondary language, but that condition is easy to meet on multilingual sites. Audit administrator profile languages if you cannot update right away.
- If you ran 3.3.1 or earlier with such an administrator, consider whether the account may have been compromised. Review recent password resets, new administrator users, and unexpected plugin or theme changes.

**Plugin & theme developers / agencies**
- Check managed client sites for the installed version and roll out 3.3.2 across the fleet.
- No API changes or migrations are described in the source.

**Hosting & platform teams**
- Wordfence Premium, Care, and Response users received a firewall rule on August 13, 2026. Free Wordfence users receive it on September 12, 2026. Patching is the actual fix, since the rule only covers known exploits.

## Technical details

The available excerpt is truncated and does not include the code-level root cause, so the exact leaking code path is not confirmed here. What the advisory states:

- Vulnerability class: unauthenticated account takeover via password reset link disclosure.
- The password reset key for an administrator is exposed to unauthenticated attackers. The attacker can then complete the standard reset flow (using the key) to set a new password and log in.
- The exposure is conditional on the target user's profile language being a published secondary language in TranslatePress, which suggests the leak is tied to TranslatePress's multilingual handling of that user's locale. This is an inference, not a stated cause.
- Affected: `<= 3.3.1`. Fixed: `3.3.2`. Firewall rule scope: 3.3.1, slug `translatepress-multilingual`.

Refer to the linked Wordfence post and the plugin's changelog or diff between 3.3.1 and 3.3.2 for the specific code change.

## Contribution

The report came in through the Wordfence Bug Bounty Program, and the vendor's turnaround was unusually fast: Cozmoslabs acknowledged the report and shipped the patched release on the same day, August 13, one day after receiving full disclosure details.

---

*Reported by Wordfence. Summarized from their published advisory — see the linked original for the full analysis. Wordfence is a trademark of Defiant, Inc.; this digest is not affiliated with or endorsed by them.*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
