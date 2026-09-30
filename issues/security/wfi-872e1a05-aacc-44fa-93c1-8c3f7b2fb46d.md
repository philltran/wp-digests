# Ultra Addons for Contact Form 7 <= 3.5.50 - Unauthenticated Arbitrary File Upload via Signature Form Field

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-09-26
- **CVE:** [CVE-2026-82901](https://www.cve.org/CVERecord?id=CVE-2026-82901)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-434: Unrestricted Upload of File with Dangerous Type
- **Affected:** Ultra Addons for Contact Form 7 (plugin) <= 3.5.50
- **Patched in:** 3.5.51
- **Researchers:** Supakiad S. (m3ez)
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/872e1a05-aacc-44fa-93c1-8c3f7b2fb46d](https://www.wordfence.com/threat-intel/vulnerabilities/id/872e1a05-aacc-44fa-93c1-8c3f7b2fb46d)
- **Usefulness:** 5/5

## Summary

Ultra Addons for Contact Form 7 versions up to and including 3.5.50 fail to properly validate file types in `uacf7_wpcf7_mail_components()`, allowing unauthenticated attackers to upload arbitrary files through the signature form field. Because the uploaded file type is not restricted, this can lead to remote code execution. The flaw is only reachable when the plugin's PDF Generator module is enabled (it is disabled by default). Version 3.5.51 patches it.

## Impact

**Site owners**
- Update Ultra Addons for Contact Form 7 to 3.5.51 or later.
- Sites are only exposed if the **PDF Generator** module is enabled; it is off by default. Check the plugin's module settings to see whether it is on.
- If you ran an affected version with the PDF Generator module enabled, review the uploads directory and the web root for unexpected files (especially executable types such as `.php`) and check access logs for anonymous POSTs to Contact Form 7 form endpoints.

**Plugin & theme developers / agencies**
- Audit managed sites for this plugin at version <= 3.5.50, prioritizing those with the PDF Generator module enabled.
- No API changes are documented; no code migration is required beyond updating.

**Hosting & platform teams**
- The vulnerability is unauthenticated and rated CVSS 9.8, so it is a candidate for virtual patching or WAF rules and for fleet-wide version scans.
- Denying PHP execution in the uploads directory reduces the impact of arbitrary file upload in general.

**Headless & REST consumers**
- No REST-specific details are provided in the advisory.

## Technical details

The advisory attributes the flaw to insufficient file type validation in the `uacf7_wpcf7_mail_components` function, which is the plugin's handler for Contact Form 7 mail components (the name suggests it hooks the `wpcf7_mail_components` filter, though the advisory does not say so). The vulnerable path involves the signature form field and is reached only when the PDF Generator module is enabled. The weakness class is CWE-434 (Unrestricted Upload of File with Dangerous Type). Because the attacker needs no authentication and can place arbitrary files on the server, remote code execution becomes possible if the file lands in an executable location.

The advisory does not include the diff or describe the specific code change in 3.5.51, so the exact validation added (extension allowlist, MIME check, or otherwise) is not documented in the source material.

## Contribution

The record carries no discussion detail beyond the researcher credit and the fixed version.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
