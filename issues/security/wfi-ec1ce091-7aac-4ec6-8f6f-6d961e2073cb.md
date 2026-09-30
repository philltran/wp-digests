# wpShopGermany IT-RECHT KANZLEI <= 2.3 - Unauthenticated Remote Code Execution

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-09-15
- **CVE:** [CVE-2026-88795](https://www.cve.org/CVERecord?id=CVE-2026-88795)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-94: Improper Control of Generation of Code ('Code Injection')
- **Affected:** wpShopGermany IT-RECHT KANZLEI (plugin) <= 2.3
- **Patched in:** 2.4
- **Researchers:** Naoki Kawahigashi
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/ec1ce091-7aac-4ec6-8f6f-6d961e2073cb](https://www.wordfence.com/threat-intel/vulnerabilities/id/ec1ce091-7aac-4ec6-8f6f-6d961e2073cb)
- **Usefulness:** 5/5

## Summary

The wpShopGermany IT-RECHT KANZLEI plugin, versions up to and including 2.3, passes user-supplied input to a code-execution sink without sufficient validation. An unauthenticated attacker can run arbitrary code on the server. Version 2.4 fixes the issue.

## Impact

- **Site owners:** Any site running wpShopGermany IT-RECHT KANZLEI <= 2.3 is exposed to unauthenticated remote code execution (CVSS 9.8). Update to 2.4 or later immediately. If the plugin cannot be updated right away, deactivate it.
- **Agencies / hosting & platform teams:** Audit managed sites for this plugin and its version, and prioritize patching. Because exploitation needs no authentication, sites that ran a vulnerable version should be checked for signs of compromise (unexpected files, new admin users, modified plugin/theme code, unfamiliar scheduled tasks).
- **Plugin & theme developers:** No API changes. The advisory gives no other details.
- **Headless & REST consumers:** The advisory does not say which entry point is exploitable, so no specific route can be named.

## Technical details

The advisory is brief. It classifies the flaw as CWE-94 (Improper Control of Generation of Code) and says user-supplied input is insufficiently validated "before it is executed." The record does not name the vulnerable file, function, hook, or endpoint, nor the delivery mechanism (AJAX action, REST route, or direct request). It also does not describe what the 2.4 patch changes. No diff or patch details were provided, so none are described here.

Remediation is to update the plugin to 2.4 or a newer patched version.

## Contribution

The record carries no discussion or disclosure-timeline detail beyond the credits and identifiers already shown.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
