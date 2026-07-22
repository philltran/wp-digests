# ProfileGrid – User Profiles, Groups and Communities <= 5.9.9.6 - Unauthenticated Privilege Escalation via Password Reset

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-07-08
- **CVE:** [CVE-2026-57697](https://www.cve.org/CVERecord?id=CVE-2026-57697)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-521: Weak Password Requirements
- **Affected:** ProfileGrid – User Profiles, Groups and Communities (plugin) <= 5.9.9.6
- **Patched in:** 5.9.9.7
- **Researchers:** Jakub Herman
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/59aebe74-cf53-49f4-8f20-ebb5503d7dbd](https://www.wordfence.com/threat-intel/vulnerabilities/id/59aebe74-cf53-49f4-8f20-ebb5503d7dbd)
- **Usefulness:** 5/5

## Summary

The ProfileGrid plugin contained a critical vulnerability allowing unauthenticated attackers to escalate privileges by resetting arbitrary user passwords. The flaw stemmed from missing identity validation during the password update process, which permitted account takeover for any user, including administrators. This issue has been resolved in version 5.9.9.7 by enforcing strict identity verification before processing password changes.

## Impact

- **Site owners & administrators**: Update ProfileGrid to 5.9.9.7 or later immediately to prevent unauthenticated account takeover.
- **Plugin & theme developers**: No direct code changes required, but verify that any custom integrations or overrides of ProfileGrid's password flows are updated to the patched version.
- **Hosting & platform teams**: Ensure automated update policies include ProfileGrid, as this vulnerability allows full administrative compromise without authentication.

## Technical details

The vulnerability existed in the plugin's password update routine, which accepted credential modifications without verifying that the requester actually owned the target account. The patch in 5.9.9.7 introduces identity verification prior to processing password changes, ensuring that only legitimate users can update their credentials. Specific file paths or function names are not disclosed in the advisory, so the fix is described at the behavioral level.

## Contribution

The vulnerability was reported by Jakub Herman and patched in 5.9.9.7; the advisory record carries no additional design debate or implementation discussion.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
