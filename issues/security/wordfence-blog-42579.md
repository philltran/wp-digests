# WP2Shell WordPress Exploit Technical Analysis and Real Attack Data

- **Source:** Wordfence Blog
- **Type:** Blog post
- **Author:** unknown
- **Published:** 2026-07-29
- **Tags:** `Threat Research`, `Vulnerabilities`, `WordPress Security`, `WordPress Plugin Vulnerability News July 2026`, `WordPress Security News July 2026`
- **Link:** [https://www.wordfence.com/blog/2026/07/wp2shell-wordpress-exploit-technical-analysis-and-real-attack-data/](https://www.wordfence.com/blog/2026/07/wp2shell-wordpress-exploit-technical-analysis-and-real-attack-data/)
- **Usefulness:** 5/5

## Summary

WordPress core patched a critical vulnerability chain that allowed unauthenticated attackers to create an administrator account and subsequently execute arbitrary code. The chain combined an SQL injection in the `author__not_in` parameter (CVE-2026-60137) with a REST API batch request route confusion flaw (CVE-2026-63030). The Security Team issued forced automatic updates to close the gap across affected 6.8, 6.9, and 7.0 releases.

## Impact

- **Site owners & administrators:** Verify that WordPress is updated to the patched version. Sites with automatic background updates enabled should already be patched.
- **Plugin & theme developers:** No direct API changes or deprecations. Audit any custom code that processes unauthenticated author query parameters or interacts with the REST batch endpoint for injection or route confusion risks.
- **Hosting & platform teams:** Confirm that forced automatic update mechanisms are functioning correctly and that no legacy or custom WP installations remain on vulnerable version ranges.
- **No action required** for developers building on stable public APIs, as the fix is internal to core query and REST routing logic.

## Technical details

The patch resolves an unauthenticated SQL injection triggered by unsanitized input in the `author__not_in` query parameter, and a REST API batch request route confusion that allowed authentication bypass. Core query sanitization and the REST batch dispatcher were hardened to prevent parameter injection and route misrouting. No public hooks, filters, block.json fields, or REST schema endpoints were altered. Developers interacting with author metadata or the REST batch endpoint should verify that custom implementations properly escape inputs and validate request contexts.

## Contribution

The advisory record carries no detailed commit history or contributor handles, but confirms the Security Team released forced automatic updates on July 17, 2026, following active exploitation.

---

*Reported by Wordfence. Summarized from their published advisory — see the linked original for the full analysis. Wordfence is a trademark of Defiant, Inc.; this digest is not affiliated with or endorsed by them.*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
