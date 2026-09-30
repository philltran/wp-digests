# MasterStudy LMS WordPress Plugin – for Online Courses and Education <= 3.7.42 - Unauthenticated Arbitrary File Deletion

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-24
- **CVE:** [CVE-2026-78284](https://www.cve.org/CVERecord?id=CVE-2026-78284)
- **CVSS:** 9.1 (Critical)
- **CWE:** CWE-22: Improper Limitation of a Pathname to a Restricted Directory ('Path Traversal')
- **Affected:** MasterStudy LMS WordPress Plugin – for Online Courses and Education (plugin) <= 3.7.42
- **Patched in:** 3.7.43
- **Researchers:** 20kilograma
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/fa8f0688-8c54-43f8-acea-2aec2fcfbdce](https://www.wordfence.com/threat-intel/vulnerabilities/id/fa8f0688-8c54-43f8-acea-2aec2fcfbdce)
- **Usefulness:** 5/5

## Summary

MasterStudy LMS versions up to and including 3.7.42 fail to properly validate file paths in a code path reachable without authentication, allowing an attacker to delete arbitrary files on the server. Deleting a file such as `wp-config.php` can force WordPress back into the installer flow and lead to remote code execution. The issue is fixed in 3.7.43.

## Impact

**Site owners**
- Any site running MasterStudy LMS <= 3.7.42 is exposed to unauthenticated attackers; no account or login is needed to exploit it.
- **Action required:** update to 3.7.43 or later immediately.
- If you cannot update right away, deactivate the plugin until you can.
- Review sites that ran a vulnerable version for missing or unexpected changes to core files (notably `wp-config.php`), unexpected reinstall prompts, or new admin users.

**Plugin & theme developers / agencies**
- Audit client sites and any bundles, themes, or distributions that include MasterStudy LMS as a dependency.
- Add the plugin to your patch-priority list given the CVSS 9.1 rating.

**Hosting & platform teams**
- Consider WAF rules or virtual patching for path traversal attempts against the plugin's endpoints while updates roll out.
- No API, hook, or schema changes are announced, so there is no code migration for downstream consumers.

## Technical details

The advisory classifies this as CWE-22 (path traversal): the plugin uses an attacker-influenced file path in a file deletion operation without sufficient validation, and the affected code path can be reached by unauthenticated users. The record does not name the specific function, AJAX action, REST route, or parameter involved, and no diff or patch is included, so the exact vulnerable code and the nature of the 3.7.43 fix cannot be described from the source material.

The advisory notes that deleting a file such as `wp-config.php` can easily lead to remote code execution. Beyond that, the impact scope is arbitrary file deletion on the server, limited by the web server user's filesystem permissions.

## Contribution

The record carries no discussion detail beyond the researcher credit and the patched version.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
