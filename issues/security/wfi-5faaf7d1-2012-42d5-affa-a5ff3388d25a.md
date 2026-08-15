# WordPress Core <= 7.0.2 - Authenticated (Contributor+) Stored Cross-Site Scripting via Emoji Settings Element

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-08
- **CVSS:** 6.4 (Medium)
- **CWE:** CWE-79: Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')
- **Affected:** WordPress (core) >= 4.7.0 and <= 4.7.33; >= 4.8.0 and <= 4.8.28; >= 4.9.0 and <= 4.9.29; >= 5.0.0 and <= 5.0.25; >= 5.1.0 and <= 5.1.22; >= 5.2.0 and <= 5.2.24; >= 5.3.0 and <= 5.3.21; >= 5.4.0 and <= 5.4.19; >= 5.5.0 and <= 5.5.18; >= 5.6.0 and <= 5.6.17; >= 5.7.0 and <= 5.7.15; >= 5.8.0 and <= 5.8.13; >= 5.9.0 and <= 5.9.13; >= 6.0.0 and <= 6.0.12; >= 6.1.0 and <= 6.1.10; >= 6.2.0 and <= 6.2.9; >= 6.3.0 and <= 6.3.8; >= 6.4.0 and <= 6.4.8; >= 6.5.0 and <= 6.5.8; >= 6.6.0 and <= 6.6.5; >= 6.7.0 and <= 6.7.5; >= 6.8.0 and <= 6.8.6; >= 6.9.0 and <= 6.9.5; >= 7.0.0 and <= 7.0.2
- **Patched in:** 4.7.34, 4.8.29, 4.9.30, 5.0.26, 5.1.23, 5.2.25, 5.3.22, 5.4.20, 5.5.19, 5.6.18, 5.7.16, 5.8.14, 5.9.14, 6.0.13, 6.1.11, 6.2.10, 6.3.9, 6.4.9, 6.5.9, 6.6.6, 6.7.6, 6.8.7, 6.9.6, 7.0.3
- **Researchers:** Asaf Mozes
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/5faaf7d1-2012-42d5-affa-a5ff3388d25a](https://www.wordfence.com/threat-intel/vulnerabilities/id/5faaf7d1-2012-42d5-affa-a5ff3388d25a)
- **Usefulness:** 5/5

## Summary

WordPress core allowed a stored cross-site scripting payload to be smuggled into post content via the emoji settings element, because submitted post content was not sufficiently restricted. Any authenticated user at Contributor level or above could save a post carrying the payload; the script then executes when someone previews that post. Because Contributors cannot publish, the realistic trigger is an Editor or Administrator previewing a pending submission for review — meaning the script runs in a higher-privileged session. The fix shipped as a coordinated security release across every branch WordPress still backports to, from 4.7 through 7.0.

## Impact

**Site owners / administrators**
- Update to the patched release on your branch. Sites on WordPress core auto-updates for minor versions will receive this without intervention; verify that background updates are actually enabled and completing rather than assuming.
- Highest exposure is on multi-author and open-submission sites where untrusted users hold `contributor` (or any role that can create drafts) and privileged users preview those drafts during editorial review.
- Interim hardening: audit who actually holds Contributor+ and remove stale accounts. Note that Contributors do not have `unfiltered_html`, so this was not a capability misconfiguration — the restriction gap was in core.

**Plugin & theme developers**
- No API change is disclosed in this record — no deprecated symbol, no signature change, nothing to migrate.
- If your plugin renders raw `post_content` in a preview-like or custom-preview context (a headless preview route, a live-preview panel, a custom `the_preview`-style filter chain), re-test after updating: custom render paths that bypass core's output handling may still be exposed even on a patched core.

**Hosting & platform teams**
- 24 branches received a patch simultaneously. Fleet inventories should be checked branch-by-branch — a site pinned to an old branch such as 5.6 or 6.2 is patched by `5.6.18` / `6.2.10`, not by "upgrade to latest."
- WAF/virtual-patching customers should confirm rule coverage for sites that cannot be updated immediately, but treat that as a stopgap, not a fix.

**Headless & REST consumers**
- Content authored through the REST API by Contributor-level tokens is the same input surface. If your front end renders preview content fetched from `/wp/v2/posts/<id>?context=edit` or an autosave/preview route, that render path is where the payload would land.

## Technical details

Wordfence classifies this as CWE-79 (stored XSS) with the root cause given as **insufficient restrictions on submitted post content**, with the emoji settings element as the injection vector and post preview as the execution sink. The record does not publish a diff, a Trac ticket number, or a changeset, so the specific function and sanitization callback that changed are not disclosed here.

For orientation only — not confirmed by the advisory — the "emoji settings element" in core is the inline script block printed by `print_emoji_detection_script()` in `wp-includes/formatting.php`, hooked onto `wp_head` and `admin_print_scripts`, which emits a `window._wpemojiSettings` object followed by the emoji detection loader. Do not treat that mapping as the patched code path without reading the actual core changeset for your branch.

What is stated, and is enough to reason about exposure:

- **Privilege required:** authenticated, Contributor or above. This is not an unauthenticated or a subscriber-level issue.
- **Persistence:** the payload is stored in post content, so it survives until the post is edited or removed. Patching core stops execution but does not retroactively scrub stored content — audit `post_content` on drafts and pending posts from untrusted authors if you suspect abuse.
- **Execution context:** preview of the affected post, which is typically performed by a user with more capability than the author who planted it.
- **CVSS 6.4 (Medium)** reflects that authenticated access plus a victim interaction (the preview) are both required.

No hook, filter, `block.json` field, REST schema, or database change is reported. There is no before/after usage pattern for developers to adopt — the correct response is the version bump, not a code change.

## Contribution

The Wordfence record carries no discussion, patch link, or design detail — only the researcher credit and the affected/patched version ranges, which is typical for a coordinated core security release where the Trac ticket stays private until after disclosure.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
