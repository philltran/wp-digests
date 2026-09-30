# Wordfence Argus Finds Unauthenticated Arbitrary File Upload Vulnerability in Gravity Forms

- **Source:** Wordfence Blog
- **Type:** Blog post
- **Author:** unknown
- **Published:** 2026-09-01
- **Tags:** `AI`, `Research`, `Threat Research`, `Vulnerabilities`, `WordPress Security`, `WordPress Plugin Vulnerability News September 2026`, `WordPress Vulnerability News September 2026`
- **Link:** [https://www.wordfence.com/blog/2026/09/wordfence-argus-finds-unauthenticated-arbitrary-file-upload-vulnerability-in-gravity-forms/](https://www.wordfence.com/blog/2026/09/wordfence-argus-finds-unauthenticated-arbitrary-file-upload-vulnerability-in-gravity-forms/)
- **Usefulness:** 5/5

## Summary

Gravity Forms up to and including 3.0.2 has an unauthenticated arbitrary file upload flaw in the multi-file upload path. Insufficient validation of chunk state in `GFAsyncUpload::upload()` lets an attacker write files with attacker-chosen extensions into a publicly reachable temporary upload directory, which can lead to remote code execution. Version 3.0.3 contains the fix.

## Impact

**Site owners**
- Update Gravity Forms to 3.0.3 or later. All versions <= 3.0.2 are affected, and the plugin has an estimated 1M+ active installs.
- The flaw requires no authentication, so any site with the plugin active should be treated as exposed until patched.
- Consider reviewing the temporary upload directory for unexpected files, especially executable extensions such as `.php`. The source material does not describe specific indicators of compromise.

**Agencies and hosting/platform teams**
- Prioritize fleet-wide updates to 3.0.3. The advisory rates this high severity (CVSS 8.1).
- Wordfence Premium, Care and Response customers have had a firewall rule since August 13, 2026. Free users receive it on September 12, 2026. A WAF rule is not a substitute for patching.

**Plugin/theme developers**
- No API, hook or deprecation changes are described. The excerpt does not say whether add-ons that depend on the async upload flow are affected.

## Technical details

The advisory attributes the bug to insufficient validation of multi-file upload chunk state in `GFAsyncUpload::upload()`. The vulnerability title, "Unauthenticated Arbitrary File Upload via State/Chunk Hash Confusion", indicates the upload state or chunk hash is not validated correctly, so an attacker can control the resulting file's extension. Files are written to a public temporary upload directory, and the attacker-selected extension is what enables RCE.

The provided excerpt is truncated and includes no diff or patch. The exact validation added in 3.0.3, the specific temp path, and the precise request parameters are not available here, so consult the full Wordfence advisory or the 3.0.3 changelog for those details.

## Contribution

The record carries no discussion detail beyond a short disclosure timeline: reported to the vendor through the Wordfence Vulnerability Management Portal on August 11, 2026, acknowledged and patched in 3.0.3 on August 20, and paid firewall protection shipped on August 13.

---

*Reported by Wordfence. Summarized from their published advisory — see the linked original for the full analysis. Wordfence is a trademark of Defiant, Inc.; this digest is not affiliated with or endorsed by them.*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
