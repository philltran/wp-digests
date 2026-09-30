# Avada <= 7.16 and Fusion Builder <= 3.16 - Unauthenticated Remote Code Execution via Arbitrary File Write

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-25
- **CVE:** [CVE-2026-18431](https://www.cve.org/CVERecord?id=CVE-2026-18431)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-862: Missing Authorization
- **Affected:** Avada (Fusion) Builder (plugin) <= 3.16, Avada | Website Builder For WordPress & WooCommerce (theme) <= 7.16
- **Patched in:** 3.16.1, 7.16.1
- **Researchers:** Alex Thomas, Wordfence Argus
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/5bef5bd3-8ec9-4a5b-bcdd-98952c7ef390](https://www.wordfence.com/threat-intel/vulnerabilities/id/5bef5bd3-8ec9-4a5b-bcdd-98952c7ef390)
- **Usefulness:** 5/5

## Summary

Avada theme versions up to 7.16, when paired with an active Fusion Builder plugin at 3.16 or below, allow unauthenticated attackers to write attacker-controlled files to the server. Because the written files can be PHP, this escalates to remote code execution and full site compromise. The flaw is a chain of missing authorization and input validation weaknesses spanning both components. Fixed in Avada 7.16.1 and Fusion Builder 3.16.1.

## Impact

**Site owners**
- Any site running Avada <= 7.16 together with Fusion Builder <= 3.16 is exposed to unauthenticated RCE (CVSS 9.8). Update both components to 7.16.1 and 3.16.1 immediately.
- Exploitation requires both the theme and plugin to be installed and active, and some administrator-authored content to be present. Do not treat this as a mitigating factor; many production Avada sites will meet these conditions.
- Because the primitive is arbitrary file write, sites that may have been exposed before patching should be checked for unexpected PHP files (e.g. in `wp-content/uploads` or theme/plugin directories), new admin users, and modified files.

**Agencies and hosting/platform teams**
- Audit fleets for Avada and Fusion Builder versions; both must be updated, since the vulnerability spans the two components.
- Consider WAF rules or blocking PHP execution in upload directories as defense in depth while patches roll out.

**Plugin and theme developers**
- No API, hook, or deprecation changes are described. Child themes and add-ons do not need code changes beyond receiving the patched parent theme and plugin.

## Technical details

The advisory does not disclose the vulnerable endpoint, function, or file paths, so the exact mechanism is not public in this record. What is stated:

- Classified as CWE-262... see below.

Correction of the above: the record classifies the issue as **CWE-862: Missing Authorization**, described as a chain of authorization and input validation weaknesses across the Avada theme and the Fusion Builder plugin.
- The chain lets an unauthenticated request cause attacker-controlled content to be written to disk, which can then be used to create and execute arbitrary PHP files.
- Exploitation depends on certain administrator-authored content being present, suggesting the write path consumes or is keyed to existing admin-created data, though the record does not say how.
- Fix shipped as theme 7.16.1 and plugin 3.16.1. No diff or changelog detail is provided, so the specific code changes are unknown.

## Contribution

The record carries no discussion detail beyond the researcher credit and patched versions.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
