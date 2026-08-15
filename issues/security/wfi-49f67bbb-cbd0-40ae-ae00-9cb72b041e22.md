# RapiSafe <= 1.0.4 - Unauthenticated Arbitrary File Deletion via 'rsmfcf7_session' and 'file_name' Parameters

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-14
- **CVE:** [CVE-2026-14484](https://www.cve.org/CVERecord?id=CVE-2026-14484)
- **CVSS:** 9.1 (Critical)
- **CWE:** CWE-22: Improper Limitation of a Pathname to a Restricted Directory ('Path Traversal')
- **Affected:** RapiSafe – Secure Multi File Upload for Contact Form 7 (plugin) <= 1.0.4
- **Researchers:** Spy0x7
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/49f67bbb-cbd0-40ae-ae00-9cb72b041e22](https://www.wordfence.com/threat-intel/vulnerabilities/id/49f67bbb-cbd0-40ae-ae00-9cb72b041e22)
- **Usefulness:** 5/5

## Summary

RapiSafe's upload-removal AJAX handler allows unauthenticated attackers to delete arbitrary server files because it trusts client-supplied session and file-name values without adequate path validation. The handler's nonce is exposed in public JavaScript, so an attacker can obtain it from any Contact Form 7 page that renders a RapiSafe upload field. No patch is available, so the practical response is to remove or disable the plugin.

## Impact

- **Site owners:** Uninstall or disable the affected RapiSafe plugin; the advisory states no known patch is available and recommends replacing the software.
- **Hosting and platform teams:** Treat this as a critical exposure and remove the plugin or block the vulnerable removal handler; monitor for deletion of sensitive files such as `wp-config.php`.
- **Plugin developers:** Do not rely on a publicly exposed nonce as the only access control for file-removal actions; validate paths to a restricted directory and enforce appropriate capability checks.
- **Headless and REST consumers:** No direct REST API change is described, but any integration that depends on RapiSafe upload fields should assume the plugin is unsafe until removed or mitigated.

## Technical details

The flaw is in the `handleAjaxRemoveUpload` function, which processes the `rsmfcf7_session` and `file_name` parameters with insufficient file path validation. An unauthenticated attacker can supply values that resolve to arbitrary server files and delete them. The required nonce is exposed in public-facing JavaScript as `RSMFCF7Vars.nonce` on every Contact Form 7 page that renders a RapiSafe upload field, making it obtainable without authentication. Deleting a file such as `wp-config.php` can lead to remote code execution. The advisory does not include a diff, patched code, or a fixed version.

## Contribution

The record carries no discussion detail beyond the advisory.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
