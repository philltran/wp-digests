# TrueBooker <= 1.2.3 - Missing Authorization to Unauthenticated Arbitrary Password Reset via 'truebooker_wp_user_id'

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-06
- **CVE:** [CVE-2026-14365](https://www.cve.org/CVERecord?id=CVE-2026-14365)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-862: Missing Authorization
- **Affected:** TrueBooker – Appointment Booking and Scheduler System (plugin) <= 1.2.3
- **Patched in:** 1.2.4
- **Researchers:** Nhien Pham (nhienit)
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/afe10c10-cace-4ce4-a813-6fda04c8d3dd](https://www.wordfence.com/threat-intel/vulnerabilities/id/afe10c10-cace-4ce4-a813-6fda04c8d3dd)
- **Usefulness:** 5/5

## Summary

TrueBooker contained a missing authorization flaw in its password-reset flow. An unauthenticated attacker could supply a `truebooker_wp_user_id` value and reset the password of an arbitrary WordPress user, including administrators. This allowed account takeover without prior credentials. The issue is fixed in the patched plugin release.

## Impact

- **Site owners:** Update the plugin to the patched release. If a vulnerable version was exposed, review user accounts and audit logs for unauthorized password changes.
- **Plugin and theme developers:** Ensure any endpoint that accepts a user identifier for password reset enforces proper authentication and authorization before acting on that identifier.
- **No action required** if the plugin is already updated to the patched release.

## Technical details

The advisory identifies the root cause as missing authorization: the plugin did not properly verify that the requester was authorized to perform the password-reset action. The exposed parameter is `truebooker_wp_user_id`, which can be used to target an arbitrary user account. No diff or code-level patch details are provided in the record.

## Contribution

The public record contains no discussion detail beyond the advisory.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
