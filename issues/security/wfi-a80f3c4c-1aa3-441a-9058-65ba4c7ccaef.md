# MStore API – Create Native Android & iOS Apps On The Cloud <= 4.20.0 - Unauthenticated Privilege Escalation

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-11
- **CVE:** [CVE-2026-27543](https://www.cve.org/CVERecord?id=CVE-2026-27543)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-266: Incorrect Privilege Assignment
- **Affected:** MStore API – Create Native Android & iOS Apps On The Cloud (plugin) <= 4.20.0
- **Patched in:** 4.21.0
- **Researchers:** Taylsec
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/a80f3c4c-1aa3-441a-9058-65ba4c7ccaef](https://www.wordfence.com/threat-intel/vulnerabilities/id/a80f3c4c-1aa3-441a-9058-65ba4c7ccaef)
- **Usefulness:** 5/5

## Summary

The affected plugin had an unauthenticated privilege escalation flaw that allowed attackers to gain administrator access. The issue is fixed in the patched release. Sites running the affected plugin should update immediately. The advisory does not disclose the vulnerable endpoint or code change.

## Impact

- Site owners: update the plugin to the patched release; this is a security fix.
- Plugin and theme developers: no code changes are described in the advisory.
- Hosting and platform teams: ensure all sites running the affected plugin are updated.

## Technical details

The record does not disclose the vulnerable endpoint, function, or code change. It describes an unauthenticated privilege escalation that allows an attacker to gain administrator access.

## Contribution

The record carries no discussion detail.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
