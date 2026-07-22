# Realtyna Organic IDX plugin + WPL Real Estate <= 5.2.0 - Unauthenticated Remote Code Execution

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-07-09
- **CVE:** [CVE-2026-57811](https://www.cve.org/CVERecord?id=CVE-2026-57811)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-94: Improper Control of Generation of Code ('Code Injection')
- **Affected:** Realtyna Organic IDX plugin + WPL Real Estate (plugin) <= 5.2.0
- **Patched in:** 5.3.0
- **Researchers:** ParkHyunWoo
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/7da4e3a2-29b3-47f3-b3fb-3a6ad0b8b8ac](https://www.wordfence.com/threat-intel/vulnerabilities/id/7da4e3a2-29b3-47f3-b3fb-3a6ad0b8b8ac)
- **Usefulness:** 5/5

## Summary

The Realtyna Organic IDX plugin + WPL Real Estate plugin contained an unauthenticated remote code execution vulnerability in versions up to 5.2.0. The flaw allowed attackers to execute arbitrary server-side code without credentials. The vulnerability was resolved in version 5.3.0, which site administrators should apply immediately.

## Impact

- **Site owners & administrators**: Any WordPress installation running Realtyna Organic IDX or WPL Real Estate ≤ 5.2.0 is critically exposed to unauthenticated RCE. Update to ≥ 5.3.0 immediately.
- **Hosting & platform teams**: Monitor for exploitation attempts targeting this plugin and enforce version constraints in managed environments.
- **Plugin & theme developers**: No direct API changes or migration steps required; this is a vendor patch for a third-party plugin.
- **No action required** for sites that do not use this plugin or are already on ≥ 5.3.0.

## Technical details

The advisory does not disclose specific vulnerable functions, file paths, or payload mechanics. The fix in 5.3.0 addresses improper input handling that previously allowed unauthenticated code execution (CWE-94). Developers requiring implementation specifics should consult the vendor’s patch notes or the Wordfence Intelligence advisory linked in the metadata.

## Contribution

The record carries no discussion detail beyond the researcher attribution and the patch version. The vulnerability was disclosed and patched without public design debate or alternative approach documentation.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
