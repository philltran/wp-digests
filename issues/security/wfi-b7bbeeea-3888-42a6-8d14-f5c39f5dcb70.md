# Customer Reviews for WooCommerce <= 5.120.0 - Missing Authorization to Unauthenticated Arbitrary Attachment Deletion via 'items[][media]' Parameter

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-09-24
- **CVE:** [CVE-2026-89055](https://www.cve.org/CVERecord?id=CVE-2026-89055)
- **CVSS:** 9.1 (Critical)
- **CWE:** CWE-862: Missing Authorization
- **Affected:** Customer Reviews for WooCommerce (plugin) <= 5.120.0
- **Patched in:** 5.121.0
- **Researchers:** HumbertoSP
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/b7bbeeea-3888-42a6-8d14-f5c39f5dcb70](https://www.wordfence.com/threat-intel/vulnerabilities/id/b7bbeeea-3888-42a6-8d14-f5c39f5dcb70)
- **Usefulness:** 5/5

## Summary

Customer Reviews for WooCommerce up to 5.120.0 lacks an authorization check in its review-submission handling, so an unauthenticated visitor can inject arbitrary Media Library attachment IDs through the `items[][media]` parameter of a review. When that review is later trashed and purged, the referenced attachments are permanently deleted. Version 5.121.0 fixes the flaw.

## Impact

- **Site owners:** Update to 5.121.0 or later immediately. The CVSS score is 9.1 (Critical). Successful exploitation permanently deletes Media Library items, including product images, logos, and documents, so unpatched sites risk unrecoverable data loss unless backups exist.
- **Exploit preconditions:** The attacker needs a public review-form link containing a 13-hex `formId`, which is normally distributed to customers by e-mail. That link exposes the nonce needed to reach the handler. No WordPress account or session is required.
- **Timing of the damage:** Deletion happens when the tampered review is later trashed and purged, not at submission. Sites with reviews containing unexpected media IDs should audit them before trashing or purging.
- **Agencies and platform teams:** Check managed sites for the plugin at <= 5.120.0. Ensure backups of `wp-content/uploads` and the attachment records in the database are current.
- **Other audiences:** The advisory does not describe any new API, deprecation, or migration step.

## Technical details

The advisory record contains no code or diff, so the details below are limited to what it states.

- **Vulnerability class:** CWE-862, Missing Authorization. The plugin does not verify that the requester is authorized to perform the action.
- **Vector:** The `items[][media]` request parameter accepts attachment IDs that are attached to a review without any check that the submitter owns or uploaded them.
- **Trigger:** When the review is trashed and purged, the associated attachments are permanently deleted from the Media Library.
- **Access path:** The nonce required to reach the handler is exposed via the public review-form link (13-hex `formId`), so no authenticated session is needed.
- **Fix:** Patched in 5.121.0. The advisory does not describe the specific code change.

## Contribution

The record carries no discussion detail beyond the researcher credit and the disclosure metadata.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
