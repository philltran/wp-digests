# PSA: Critical Unauthenticated Path Traversal Vulnerability Patched in WordPress Core

- **Source:** Wordfence Blog
- **Type:** Blog post
- **Author:** unknown
- **Published:** 2026-09-22
- **Tags:** `Research`, `Threat Research`, `Vulnerabilities`, `WordPress Security`, `WordPress Plugin Vulnerability News September 2026`, `WordPress Vulnerability News September 2026`
- **Link:** [https://www.wordfence.com/blog/2026/09/psa-critical-unauthenticated-path-traversal-vulnerability-patched-in-wordpress-core/](https://www.wordfence.com/blog/2026/09/psa-critical-unauthenticated-path-traversal-vulnerability-patched-in-wordpress-core/)
- **Usefulness:** 5/5

## Summary

WordPress 7.1.2 and security backports for every branch back to 4.7 fix a critical unauthenticated path traversal in Core that lets a remote attacker make WordPress include a readable PHP file from outside the active theme directory. Under the right theme layout and server conditions, this local file inclusion can lead to remote code execution and full site compromise. Wordfence describes it as reachable over the internet with no account or user interaction.

## Impact

**Site owners**
- Update immediately to the fixed release for your branch (7.1.2, or the backport for 7.0 down to 4.7) and confirm the update actually completed.
- Exposure is conditional, so do not infer risk from a theme name alone. Verify your installed theme and filesystem.

**Plugin & theme developers**
- The exploitable condition depends on the active parent or child theme containing a suitable top-level `page-*` directory. The official advisory cites legacy Twenty Twelve and Twenty Fourteen, plus Neve, Hestia, and Sydney, as examples of themes with the relevant layout.
- Themes shipping a top-level `page-*` directory should be audited.

**Hosting & platform teams**
- Prioritise rolling out the patched core across managed fleets, including old branches (down to 4.7) that received backports.
- Wordfence Premium, Care, and Response customers got a firewall rule on 2026-09-22. Free users get it on 2026-10-22. A firewall rule reduces exposure but does not replace updating Core.

**Headless & REST consumers**
- The excerpt does not say which request path is vulnerable, so do not assume headless setups are exempt.

This is not a no-action change: updating is required.

## Technical details

The provided excerpt is truncated, so the exact code change is not visible. What the source supports:

- **Class:** path traversal leading to local PHP file inclusion, unauthenticated.
- **Behavior:** an attacker can cause WordPress to include a readable PHP file from outside the active theme directory.
- **Preconditions for RCE:** the active parent or child theme has a suitable top-level `page-*` directory, and the server has a readable PHP file that does something useful when included.
- **Fix:** shipped in 7.1.2, with backports to 7.0 through 4.7.

The excerpt does not identify the affected function, file, or the diff of the patch, so no code-level description is given here. Consult the full Wordfence post or the official WordPress security release notes for that detail.

## Contribution

The record carries no discussion detail beyond a coordinated multi-branch release: a fix in 7.1.2 plus backports to every branch back to 4.7, all released on the disclosure date, with Wordfence's firewall rule for paid tiers shipping the same day and a 30-day delay for free users.

---

*Reported by Wordfence. Summarized from their published advisory — see the linked original for the full analysis. Wordfence is a trademark of Defiant, Inc.; this digest is not affiliated with or endorsed by them.*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
