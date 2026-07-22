# Paytium: Mollie payment forms & donations <= 5.0.2 - Unauthenticated Privilege Escalation

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-06-23
- **CVE:** [CVE-2026-56030](https://www.cve.org/CVERecord?id=CVE-2026-56030)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-266: Incorrect Privilege Assignment
- **Affected:** Paytium: Mollie payment forms & donations (plugin) <= 5.0.2
- **Patched in:** 5.0.3
- **Researchers:** Nabil Irawan
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/9ef3e6c7-b75a-4afc-b1c7-6e74b0c894a9](https://www.wordfence.com/threat-intel/vulnerabilities/id/9ef3e6c7-b75a-4afc-b1c7-6e74b0c894a9)
- **Usefulness:** 5/5

## Summary

The Paytium: Mollie payment forms & donations plugin contained an unauthenticated privilege escalation vulnerability in versions up to 5.0.2. The flaw allowed attackers to bypass authentication checks and elevate their user capabilities on affected WordPress installations. The vulnerability was resolved in version 5.0.3.

## Impact

- Site owners & administrators: Update to version 5.0.3 or later immediately to close the privilege escalation vector.
- Plugin & theme developers: No direct API changes are required, but audit any custom integrations that interact with Paytium's payment endpoints or AJAX handlers to ensure they enforce capability checks.
- Hosting & platform teams: Flag versions <= 5.0.2 for immediate remediation via automated patching or WAF rules.

## Technical details

The advisory does not include a patch diff, specific file paths, or the exact function responsible for the capability bypass. The vulnerability is classified as CWE-266 (Incorrect Privilege Assignment) with a CVSS 9.8 score, indicating a critical failure in capability verification that was addressed in the 5.0.3 release. Developers should review any custom code that relies on Paytium's internal capability checks or REST/AJAX endpoints until the patch is applied.

## Contribution

The advisory record carries no discussion detail, design debate, or patch timeline beyond the researcher attribution and version numbers.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
