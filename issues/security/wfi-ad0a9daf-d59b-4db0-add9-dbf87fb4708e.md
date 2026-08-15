# POUCO Import Users <= 1.0.0 - Unauthenticated Privilege Escalation

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-06
- **CVE:** [CVE-2026-16256](https://www.cve.org/CVERecord?id=CVE-2026-16256)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-266: Incorrect Privilege Assignment
- **Affected:** POUCO Import Users (plugin) <= 1.0.0
- **Researchers:** Khaled Alenazi (Nxploited)
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/ad0a9daf-d59b-4db0-add9-dbf87fb4708e](https://www.wordfence.com/threat-intel/vulnerabilities/id/ad0a9daf-d59b-4db0-add9-dbf87fb4708e)
- **Usefulness:** 5/5

## Summary

The POUCO Import Users plugin has an unauthenticated privilege escalation flaw that allows attackers to create new administrator accounts. No patched version is identified in the record. The practical response is to remove or replace the plugin and check for unauthorized administrative users.

## Impact

- **Site owners / administrators:** If the plugin is installed, uninstall it or replace it with a maintained alternative; no known patch is available.
- **Hosting & platform teams:** Audit managed sites for the plugin, remove it where present, and review user accounts for unexpected administrator additions.
- **Plugin & theme developers:** No code-level fix is documented in the record; avoid depending on this plugin for user import functionality.
- **Headless & REST consumers:** No specific REST route or API change is disclosed, but unauthenticated account creation may affect any site using the plugin.

## Technical details

The disclosed behavior is that unauthenticated attackers can register new administrative user accounts. The record does not identify the vulnerable endpoint, function, hook, file, or code path. No patch, configuration mitigation, or code change is provided.

## Contribution

The record carries no discussion detail beyond the advisory and the absence of a known patch.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
