# PSA: WordPress Core Patched Unauthenticated Remote Code Execution Vulnerability Chain

- **Source:** Wordfence Blog
- **Type:** Blog post
- **Author:** unknown
- **Published:** 2026-07-17
- **Tags:** `Vulnerabilities`, `WordPress Security`
- **Link:** [https://www.wordfence.com/blog/2026/07/psa-wordpress-core-patched-unauthenticated-remote-code-execution-vulnerability-chain/](https://www.wordfence.com/blog/2026/07/psa-wordpress-core-patched-unauthenticated-remote-code-execution-vulnerability-chain/)
- **Usefulness:** 5/5

## Summary

WordPress core received emergency security updates on July 17, 2026 patching two chainable vulnerabilities: an unauthenticated SQL injection (CVE-2026-60137, CVSS 7.5) via the `author__not_in` parameter affecting core 6.8–7.0.1, and a critical unauthenticated remote code execution flaw (CVE-2026-63030, CVSS 9.8) in the REST API batch request route affecting core 6.9–7.0.1. The RCE stems from a route/validation desynchronization in the batch endpoint that lets a validated sub-request be dispatched to an unintended callback, bypassing the `allow_batch` restriction and the target route's input sanitization — when chained with the SQL injection, this can escalate to full unauthenticated remote code execution. WordPress's Security Team pushed automatic background updates to affected sites rather than waiting for manual upgrades, underscoring the severity.

## Impact

- **Site owners:** Confirm your site has actually landed on a patched version — 6.8.6, 6.9.5, or 7.0.2 depending on your branch — rather than assuming the automatic background update succeeded, especially on hosts or configs where core auto-updates are disabled or delayed.
- **Plugin/theme developers:** No code changes required from third-party developers, but anyone with custom code touching REST API batch requests (`/wp-json/batch/v1`), the `allow_batch` mechanism, or query code using `author__not_in` should re-check behavior against the patched core.
- **Hosting & platform teams:** Treat this as an emergency patch cycle across the fleet — unauthenticated RCE at CVSS 9.8 with no authentication barrier means unpatched sites are immediately exploitable; verify managed-WordPress and bulk-hosting environments forced the update rather than relying on end-user action.
- **Headless & REST API consumers:** Any integration that talks to the batch endpoint should be reviewed for anomalous or unexpected batch requests in logs as part of incident triage.
- **Wordfence customers:** Premium, Care, and Response users received a firewall rule against the RCE on July 17, 2026; Wordfence Free users don't get the equivalent rule until August 16, 2026 — free-tier sites should not rely on the firewall as a stopgap and must prioritize the core update itself.
- No action is required for sites already confirmed on 6.8.6, 6.9.5, or 7.0.2.

## Technical details

Wordfence deliberately withheld exploit-level technical detail while sites are still updating, so specifics are limited to what the advisory discloses. CVE-2026-60137 is an unauthenticated SQL injection reachable via the `author__not_in` parameter, present in core 6.8 through 7.0.1. CVE-2026-63030 is a route/validation desynchronization in the REST API batch request handler (`/wp-json/batch/v1`) present in core 6.9 through 7.0.1: a sub-request that passes validation for one route can be dispatched to a different, unintended callback, which bypasses both the `allow_batch` restriction that's supposed to gate which routes are batchable and the target route's own input sanitization. Because the sanitization bypass carries through attacker-controlled parameters, chaining it with the `author__not_in` SQL injection escalates the impact to unauthenticated remote code execution (combined CVSS 9.8). Patched releases are 6.8.6, 6.9.5, and 7.0.2 — no new hooks, filters, or public APIs were disclosed as part of the fix.

## Contribution

The vulnerability chain is credited in Wordfence Intelligence's summary to researcher Adam Kues. The WordPress Security Team coordinated and shipped the fix on July 17, 2026 across the 6.8.x, 6.9.x, and 7.0.x branches, triggering automatic background updates for sites on vulnerable versions rather than waiting for manual action, consistent with how core handles critical security releases. Wordfence published this PSA the same day without full technical detail specifically to limit exploitation during the update rollout window, and staggered firewall-rule availability by tier (paid customers immediately, Free tier after a 30-day delay to August 16, 2026) per its standard responsible-disclosure practice.

---

*Reported by Wordfence. Summarized from their published advisory — see the linked original for the full analysis. Wordfence is a trademark of Defiant, Inc.; this digest is not affiliated with or endorsed by them.*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
