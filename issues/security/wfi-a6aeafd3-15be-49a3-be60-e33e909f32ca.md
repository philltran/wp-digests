# The Pressengine <= 1.0 - Authentication Bypass

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-09-15
- **CVE:** [CVE-2026-86709](https://www.cve.org/CVERecord?id=CVE-2026-86709)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-287: Improper Authentication
- **Affected:** The Pressengine (plugin) <= 1.0
- **Researchers:** Naoki Kawahigashi
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/a6aeafd3-15be-49a3-be60-e33e909f32ca](https://www.wordfence.com/threat-intel/vulnerabilities/id/a6aeafd3-15be-49a3-be60-e33e909f32ca)
- **Usefulness:** 5/5

## Summary

The Pressengine plugin for WordPress, in all versions up to and including 1.0, contains an authentication bypass that lets unauthenticated attackers log in as any user, including administrators. No patched version is available. The record does not describe the vulnerable code path.

## Impact

**Site owners**
- Any site running The Pressengine <= 1.0 is exposed to full account takeover by unauthenticated remote attackers, including administrator accounts. That typically means complete site compromise.
- There is no known patch. Wordfence's guidance is to review the details, apply mitigations that fit your risk tolerance, and consider uninstalling the plugin and finding a replacement.
- If the plugin has been active on a site, treat it as potentially compromised: review administrator accounts, recent logins, and installed plugins, themes and users for unexpected changes, and rotate credentials and salts as appropriate.

**Plugin & theme developers / agencies**
- Audit client sites and managed fleets for `the-pressengine` (or the plugin's actual slug) and deactivate and remove it where found.

**Hosting & platform teams**
- Consider scanning for the plugin across hosted sites and blocking or disabling it until a fix ships.

**Headless & REST consumers**
- The record gives no detail on the entry point (REST, AJAX, or login flow), so do not assume any interface is safe.

## Technical details

The upstream record gives only the vulnerability class: CWE-287 (Improper Authentication), leading to authentication bypass that allows logging in as an arbitrary user. It does not name the vulnerable function, file, endpoint, hook, or parameter, and no diff or patch exists. Nothing further can be stated about the mechanism without inventing details.

Affected: all versions <= 1.0. Fixed version: none known.

## Contribution

The record carries no discussion detail beyond the researcher credit, and no patch or vendor response is noted.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
