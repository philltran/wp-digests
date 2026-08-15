# ProSolution WP Client <= 2.0.10 - Unauthenticated Arbitrary File Upload via Content-Disposition Header Filename Override

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-15
- **CVE:** [CVE-2026-16098](https://www.cve.org/CVERecord?id=CVE-2026-16098)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-434: Unrestricted Upload of File with Dangerous Type
- **Affected:** ProSolution WP Client (plugin) <= 2.0.10
- **Patched in:** 2.0.11
- **Researchers:** Nir Yehoshua
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/3c4a7aef-09ec-4d17-87d6-f507d64e0afa](https://www.wordfence.com/threat-intel/vulnerabilities/id/3c4a7aef-09ec-4d17-87d6-f507d64e0afa)
- **Usefulness:** 5/5

## Summary

The ProSolution WP Client plugin has an unauthenticated arbitrary file upload vulnerability in `proSol_handleFileUpload` that can lead to remote code execution. The handler trusts an attacker-controlled `Content-Disposition` header filename, which overrides the allow-listed multipart filename before the file is saved, and a post-save extension check does not delete the already-written file. The nonce required to reach the upload handler is publicly exposed via `wp_localize_script` on front-end pages rendering the job portal shortcode, allowing unauthenticated visitors to bypass that gating check. Sites should update to the patched release immediately.

## Impact

- **Site owners**: If the plugin is installed, update to the patched release immediately; unauthenticated attackers can upload files that may be executable and achieve remote code execution.
- **Incident response**: Check for unexpected files in upload directories, new users, web shells, and other signs of compromise.
- **Plugin and theme developers**: If implementing upload handlers, validate the final filename from trusted input, not attacker-controlled `Content-Disposition` values, and enforce extension/MIME checks before writing files; do not rely on post-save deletion.
- **Hosting and platform teams**: Treat this as a critical exposure and consider restricting script execution in upload directories while patching.
- **No action required** only if the plugin is not installed or is already on the patched release.

## Technical details

- Vulnerable handler: `proSol_handleFileUpload`.
- The attacker-controlled `Content-Disposition` header filename overrides the allow-listed multipart filename before the file is saved, enabling dangerous file types to be written.
- A post-save extension check is ineffective because it runs after the file has already been written and fails to delete the already-written file.
- The upload handler's nonce is exposed via `wp_localize_script` on any front-end page rendering the job portal shortcode, so unauthenticated visitors can obtain a valid nonce and reach the handler.

## Contribution

The record carries no discussion detail beyond the advisory.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
