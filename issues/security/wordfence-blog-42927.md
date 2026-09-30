# 5 Million WordPress Sites Affected by SQL Injection Vulnerability in All-in-One WP Migration and Backup WordPress Plugin

- **Source:** Wordfence Blog
- **Type:** Blog post
- **Author:** unknown
- **Published:** 2026-09-01
- **Tags:** `Research`, `Vulnerabilities`, `WordPress Security`, `WordPress Plugin Vulnerability News August 2026`, `WordPress Vulnerability News August 2026`
- **Link:** [https://www.wordfence.com/blog/2026/09/5-million-wordpress-sites-affected-by-sql-injection-vulnerability-in-all-in-one-wp-migration-and-backup-wordpress-plugin/](https://www.wordfence.com/blog/2026/09/5-million-wordpress-sites-affected-by-sql-injection-vulnerability-in-all-in-one-wp-migration-and-backup-wordpress-plugin/)
- **Usefulness:** 5/5

## Summary

All-in-One WP Migration and Backup (5M+ installs) had an unauthenticated second-order SQL injection: attacker-supplied SQL is stored and later executed when a site administrator performs an archive restore. The injected SQL can leak the plugin's secret key, which can then be chained to remote code execution and full site takeover. The vendor fixed it in a patched release.

## Impact

**Site owners**
- Update All-in-One WP Migration and Backup to the patched release immediately. All versions up to and including 7.109 are affected; the fix is in 7.110.
- Exploitation requires an administrator to perform an archive restore after the payload is planted, so the risk window extends until the plugin is updated.
- Consider reviewing whether the plugin's secret key may have been exposed on sites that ran a vulnerable version and performed restores, and rotating it if so. The source excerpt does not describe a rotation procedure, so check the plugin's documentation.

**Agencies / hosting & platform teams**
- Audit managed fleets for the plugin and versions <= 7.109, and prioritize rollout given the install base.
- Wordfence Premium, Care and Response customers received a firewall rule on August 16, 2026; free-tier users receive it on September 15, 2026. The firewall rule is a stopgap, not a substitute for updating.

**Plugin & theme developers**
- No API changes are described. The broader takeaway is that data stored from unauthenticated input and later interpolated into SQL during a privileged operation remains exploitable, so use `$wpdb->prepare()` on stored values as well.

## Technical details

The source describes this as an **unauthenticated second-order SQL injection** reached via the archive restore flow. The attack has two stages:

1. An unauthenticated attacker causes SQL to be stored by the plugin.
2. When an administrator later runs an archive restore, the stored content is executed as SQL.

The injected SQL can be used to read the plugin's secret key. The advisory states this key can ultimately be leveraged to achieve remote code execution. The excerpt was truncated, so the specific functions, files, endpoints, and the exact mechanism for the key-to-RCE step are not available here and are not asserted. The fix shipped in 7.110, but the diff or patch details are not included in the provided material.

## Contribution

The report came in through the Wordfence Bug Bounty Program on August 14, 2026. Wordfence sent full disclosure to ServMask on August 15, and the vendor acknowledged on August 17 and shipped the patch on August 20, about six days after the initial submission. Wordfence shipped its firewall rule to paid users before the patch release, with free users following on a 30-day delay.

---

*Reported by Wordfence. Summarized from their published advisory — see the linked original for the full analysis. Wordfence is a trademark of Defiant, Inc.; this digest is not affiliated with or endorsed by them.*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
