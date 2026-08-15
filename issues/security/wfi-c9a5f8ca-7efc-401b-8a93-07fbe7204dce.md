# Formidable Digital Signatures <= 3.0.6 - Unauthenticated Arbitrary File Deletion via Signature Field

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-11
- **CVE:** [CVE-2026-16230](https://www.cve.org/CVERecord?id=CVE-2026-16230)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-23: Relative Path Traversal
- **Affected:** Formidable Digital Signatures (plugin) <= 3.0.6
- **Patched in:** 3.1
- **Researchers:** Rafie Muhammad
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/c9a5f8ca-7efc-401b-8a93-07fbe7204dce](https://www.wordfence.com/threat-intel/vulnerabilities/id/c9a5f8ca-7efc-401b-8a93-07fbe7204dce)
- **Usefulness:** 5/5

## Summary

Formidable Digital Signatures <= 3.0.6 has an unauthenticated arbitrary file deletion vulnerability in the plugin's `delete_file` function. Insufficient file path validation allows an attacker to delete files on the server by sending a crafted filename in `item_meta[field_id][content]` together with the `delete_saved_image` flag during a normal entry-creation POST on a form that accepts anonymous submissions. The issue is fixed in version 3.1. Sites using the plugin should upgrade immediately.

## Impact

- **Site owners / hosting & platform**: Upgrade Formidable Digital Signatures to 3.1 or newer; this is an unauthenticated critical issue and no configuration workaround is described.
- **Plugin & theme developers**: If you extend this plugin's signature field or entry-creation flow, review any code that passes user-controlled filenames to file deletion or path handling; do not treat `item_meta[field_id][content]` as a safe path.
- **Headless & REST consumers**: No action required unless you submit entries to affected forms; the vulnerability is in the plugin's entry-creation POST flow, not a documented REST API change.

## Technical details

The vulnerable behavior is in the `delete_file` function. During the standard entry-creation POST flow, the plugin accepted the `delete_saved_image` flag and used the value from `item_meta[field_id][content]` as a filename or path without sufficient validation. An unauthenticated attacker could supply an attacker-controlled filename to cause deletion of files on the server. The fix in 3.1 addresses the insufficient file path validation; the advisory does not provide a diff or exact replacement API.

## Contribution

The record carries no discussion detail.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
