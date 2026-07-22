# wp2shell Aftermath: The First Critical Unauthenticated WordPress Core RCE in Nearly a Decade

- **Source:** Wordfence Blog
- **Type:** Blog post
- **Author:** unknown
- **Published:** 2026-07-20
- **Tags:** `AI`, `PSA`, `Research`, `Threat Research`, `Vulnerabilities`, `WordPress Security`, `wordpress core security`, `wordpress security july 2026`, `wp2shell`
- **Link:** [https://www.wordfence.com/blog/2026/07/wp2shell-aftermath-the-first-critical-unauthenticated-wordpress-core-rce-in-nearly-a-decade/](https://www.wordfence.com/blog/2026/07/wp2shell-aftermath-the-first-critical-unauthenticated-wordpress-core-rce-in-nearly-a-decade/)
- **Usefulness:** 5/5

## Summary

WordPress Core shipped emergency patches on July 17, 2026 (versions 6.8.6, 6.9.5, and 7.0.2) for a two-vulnerability chain, dubbed wp2shell, that allows unauthenticated remote code execution against unmodified WordPress installs. The chain combines a SQL injection flaw (CVE-2026-60137) with a second vulnerability (CVE-2026-63030) that together let an attacker execute arbitrary code without any plugin, theme, or authenticated access being involved. WordPress.org rated the pair one critical and one high severity issue and — because of that severity — force-enabled automatic updates across affected versions rather than waiting for sites to update on their own schedule. The chain was discovered and responsibly disclosed by Adam Kues of Assetnote/Searchlight Cyber, and Wordfence is calling it the first critical unauthenticated Core RCE in nearly a decade.

## Impact

- **Site owners**: update to 6.8.6, 6.9.5, or 7.0.2 immediately if auto-update hasn't already applied; verify the update actually completed, then audit administrator accounts and recently modified plugin/theme files for signs of compromise, since exploitation attempts began within hours of the patch (and thus the vulnerability's public disclosure).
- **Hosting/platform teams**: WordPress.org force-enabled automatic background updates for affected versions given the severity — confirm this propagated on managed fleets, especially for sites where auto-updates are normally disabled, pinned to a major version, or managed through a custom deployment pipeline that bypasses core's update mechanism.
- **Wordfence customers**: Premium, Care, and Response tiers received firewall rules the same day as disclosure (July 17); Free-tier users don't get the equivalent rule until August 16, 2026 (the standard 30-day delay), leaving a roughly month-long exposure window for sites relying solely on the free firewall and running an unpatched version.
- **Plugin/theme developers**: not directly implicated — this is a Core vulnerability, not a plugin/theme API issue — but development and staging environments running affected Core versions are equally exploitable and should be patched.
- **Headless/REST consumers**: no specific REST-route involvement is described; the exposure is at the Core application level rather than a documented API surface, so no separate action beyond the general Core update is indicated by the source material.

## Technical details

The wp2shell chain is tracked under two CVEs: CVE-2026-60137, a SQL injection vulnerability, and CVE-2026-63030, which combines with it to reach remote code execution. WordPress 6.9.x and 7.0.x are exposed to the full RCE chain; WordPress 6.8.x is exposed only to the SQL injection component (CVE-2026-60137) and not the RCE escalation. Patches landed in 6.8.6, 6.9.5, and 7.0.2 on July 17, 2026, and — notably — WordPress.org enabled forced automatic updates for the affected release lines given the severity, overriding sites' normal auto-update configuration. The source material (a Wordfence advisory summary) does not specify the vulnerable files, functions, or endpoints involved, as that detail falls in the truncated portion of the post; readers needing exploit-level specifics should consult the full Wordfence writeup or the official WordPress security release notes.

## Contribution

The vulnerability chain was discovered and responsibly disclosed to the WordPress Security Team by Adam Kues of Assetnote/Searchlight Cyber. WordPress shipped coordinated patches on July 17, 2026. Wordfence, which provided same-day firewall coverage to its paid tiers, published its analysis and timeline (this post) on July 20, 2026, noting that exploitation attempts and endpoint probing began the same evening the patches went out, with public proof-of-concept code following in the days after.

---

*Reported by Wordfence. Summarized from their published advisory — see the linked original for the full analysis. Wordfence is a trademark of Defiant, Inc.; this digest is not affiliated with or endorsed by them.*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
