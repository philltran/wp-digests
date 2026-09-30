# WordPress 7.1.2 Release

- **Source:** WordPress News
- **Type:** Blog post
- **Author:** John Blackbourn
- **Published:** 2026-09-22
- **Tags:** `Releases`, `Security`
- **Link:** [https://wordpress.org/news/2026/09/wordpress-7-1-2-release/](https://wordpress.org/news/2026/09/wordpress-7-1-2-release/)
- **Usefulness:** 5/5

## Summary

WordPress 7.1.2 is a security release fixing a critical vulnerability in page template resolution. Under certain conditions, an unauthenticated attacker could cause template resolution to include a chosen readable local PHP file located outside the active theme directories. When server-environment and active-theme pre-conditions are both met, this can result in remote code execution (RCE). The fix was backported to every branch eligible for security fixes, currently through 4.7.

## Impact

**Site owners / hosting & platform teams**
- Update to 7.1.2 immediately. The release is flagged critical and the announcement recommends immediate updating.
- Sites with automatic background updates enabled will update on their own; confirm the update landed rather than assuming it did.
- Sites on older branches should receive the backported fix through their branch's security release. Only the most recent WordPress version is actively supported, so moving to the current release is the durable answer.

**Plugin & theme developers**
- The announcement says exploitation depends on the active theme meeting certain pre-conditions, but it does not say what they are. Do not assume a custom theme is unaffected; update the core install regardless.
- No API changes, deprecations, or migration steps are mentioned in the announcement.

**Headless & REST consumers**
- Nothing in the announcement ties the issue to REST. Follow the same guidance: update core.

## Technical details

The announcement is deliberately sparse. What it states:

- **Vulnerability class:** page template resolution can be made to include a chosen, readable local PHP file outside the active theme directories (local file inclusion, escalating to RCE).
- **Authentication:** none required.
- **Conditions:** both the server environment and the active theme must meet certain unstated pre-conditions for RCE.

The post names no functions, files, hooks, or diffs, and no patch is included in the source material, so the exact code path changed is not described here. Consult the linked advisory (CVE-2026-87902 / GHSA-7hp8-65ch-5whp) for further detail. There are no stated new hooks, filters, schema, or database changes.

## Contribution

The record carries no discussion detail beyond the release credits. The fix was coordinated as a security release and backported to all branches eligible for security fixes, currently through 4.7.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
