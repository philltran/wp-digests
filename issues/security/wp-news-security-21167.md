# WordPress 7.0.2 Release

- **Source:** WordPress News
- **Type:** Blog post
- **Author:** John Blackbourn
- **Published:** 2026-07-17
- **Tags:** `Releases`, `Security`, `minor-releases`, `releases`
- **Link:** [https://wordpress.org/news/2026/07/wordpress-7-0-2-release/](https://wordpress.org/news/2026/07/wordpress-7-0-2-release/)
- **Usefulness:** 5/5

## Summary

WordPress 7.0.2 is a security release that patches one critical and one high-severity vulnerability. The critical issue combines a REST API batch-route confusion bug with SQL injection to achieve Remote Code Execution, and the high-severity issue is a separately reported, facilitated SQL injection. Because of the severity, WordPress.org enabled forced auto-updates for sites running affected versions rather than waiting for site owners to update manually.

## Impact

- **Site owners:** Update immediately — affected sites are being force-updated via the auto-update system regardless of normal auto-update settings; if a site somehow doesn't receive it, update manually via Dashboard → Updates or by downloading 7.0.2 from WordPress.org.
- **Plugin/theme developers:** No public API was deprecated or changed; nothing to update in code. Worth reviewing any custom code that issues or handles REST API batch requests, since the RCE chain runs through batch-route handling.
- **Hosting & platform teams:** Patch or force-update any fleet still on 6.8.x, 6.9.x, or the 7.1 beta track. Backports are available: 6.9.5 and 6.8.6 fix the relevant issue(s) for those branches, and 7.1 beta2 fixes both for sites running the beta. Versions prior to 6.8 are confirmed not affected — no action needed for those.
- **Headless & REST API consumers:** The vulnerable path runs through the REST API's batch-request handling, so any headless setup that relies on batch endpoints should confirm it's running a patched core version; no client-side integration changes are required.

## Technical details

The release note does not include a diff, so specifics are limited to what WordPress.org disclosed: (1) a "facilitated SQL injection" issue, reported as a team by TF1T, dtro, and haongo, tracked as CVE-2026-60137 / GHSA-fpp7-x2x2-2mjf; and (2) a "REST API batch-route confusion" issue that combines with SQL injection to enable Remote Code Execution, reported by Adam Kues (Assetnote / Searchlight Cyber), tracked as CVE-2026-63030 / GHSA-ff9f-jf42-662q. The route-confusion description implies the core REST API's batch-processing dispatch (the `/batch/v1` endpoint and its internal sub-request routing) could be manipulated to invoke an unintended internal route/handler, which was then chained with a SQL injection primitive to execute arbitrary code. WordPress 6.9 is affected by both issues (fixed in 6.9.5); WordPress 6.8 is affected only by the first (fixed in 6.8.6); WordPress 7.1 beta was affected by both (fixed in 7.1 beta2); versions before 6.8 are not affected.

## Contribution

Published July 17, 2026 by John Blackbourn, who led the release together with Barry Abrahamson. The two vulnerabilities came from external security research: a team report from TF1T, dtro, and haongo, and a separate report from Adam Kues of Assetnote / Searchlight Cyber, both credited under WordPress's responsible-disclosure process. Beyond the named release leads and researchers, the note credits a broad list of contributors (including Aaron Jorbin, Alex Concha, David Baumwald, Dominik Schilling, Jonathan Desrosiers, Sergey Biryukov, and others) plus representatives from Altis, Automattic, Bluehost, Cloudflare, GoDaddy, Hostinger, and WP Engine — reflecting the coordinated cross-host backport effort (6.8.6, 6.9.5, 7.1 beta2) rather than a single PR discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
