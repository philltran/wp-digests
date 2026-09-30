# Attackers Actively Exploiting Critical Vulnerability in WooCommerce Wholesale Lead Capture Plugin

- **Source:** Wordfence Blog
- **Type:** Blog post
- **Author:** unknown
- **Published:** 2026-09-14
- **Tags:** `Threat Research`, `Vulnerabilities`, `WordPress Security`, `WordPress Plugin Vulnerability News September 2026`, `WordPress Vulnerability News September 2026`
- **Link:** [https://www.wordfence.com/blog/2026/09/attackers-actively-exploiting-critical-vulnerability-in-woocommerce-wholesale-lead-capture-plugin/](https://www.wordfence.com/blog/2026/09/attackers-actively-exploiting-critical-vulnerability-in-woocommerce-wholesale-lead-capture-plugin/)
- **Usefulness:** 5/5

## Summary

Wordfence reports active, large-scale exploitation of an unauthenticated arbitrary file upload flaw in the premium WooCommerce Wholesale Lead Capture plugin. The plugin's `wwlc_file_upload_handler` AJAX action, used by the wholesale registration form's file upload fields, performs no file type validation, so unauthenticated visitors can upload PHP files and achieve remote code execution. The issue is fixed in 2.0.3.2, and Wordfence's firewall has blocked over 100,000 exploit attempts.

## Impact

**Site owners**
- Any site running WooCommerce Wholesale Lead Capture `<= 2.0.3.1` is exposed to unauthenticated RCE. Update to `2.0.3.2` or later immediately.
- Because exploitation is active, treat unpatched sites as potentially compromised: look for unexpected PHP or other executable files in the plugin's upload destination and the wider uploads directory, unfamiliar admin users, and modified core/plugin files.
- Estimated footprint is about 6,000 active installs.

**Agencies & hosting/platform teams**
- Inventory client sites for the `woocommerce-wholesale-lead-capture` plugin slug and verify versions. Because it is a premium plugin, updates come through the vendor's licensing channel rather than WordPress.org, so sites with lapsed licenses may not see the update.
- Consider blocking PHP execution in upload directories at the web server level as defense in depth.

**Plugin & theme developers**
- No API changes. The general lesson is that any `wp_ajax_nopriv_*` handler accepting files needs server-side type validation.

## Technical details

The vulnerable entry point is the AJAX action `wwlc_file_upload_handler`, which processes uploads from the plugin's wholesale registration form file fields and is reachable without authentication. Per the advisory, the root cause is missing file type validation in all versions up to and including 2.0.3.1, so an attacker can place arbitrary files, including PHP backdoors, on the server.

The provided excerpt is truncated before the handler's code walkthrough and before any description of the 2.0.3.2 fix, so the specific validation added in the patched release is not covered here. Consult the full Wordfence post for the code analysis.

Detection and protection timeline (from the post): the flaw was publicly disclosed on February 20, 2026; the Wordfence firewall rule went to Premium, Care and Response users on February 27, 2026, and to free users on March 29, 2026.

## Contribution

The record carries no discussion detail beyond the disclosure and firewall-rule rollout timeline, which staggered protection between paid and free Wordfence tiers by 30 days.

---

*Reported by Wordfence. Summarized from their published advisory — see the linked original for the full analysis. Wordfence is a trademark of Defiant, Inc.; this digest is not affiliated with or endorsed by them.*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
