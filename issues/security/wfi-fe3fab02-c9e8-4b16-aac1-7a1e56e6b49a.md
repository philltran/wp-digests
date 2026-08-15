# Webinfos <= 1.2 - Unauthenticated Arbitrary File Upload

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-06
- **CVE:** [CVE-2026-12872](https://www.cve.org/CVERecord?id=CVE-2026-12872)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-434: Unrestricted Upload of File with Dangerous Type
- **Affected:** Webinfos (plugin) <= 1.2
- **Researchers:** Khaled Alenazi (Nxploited)
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/fe3fab02-c9e8-4b16-aac1-7a1e56e6b49a](https://www.wordfence.com/threat-intel/vulnerabilities/id/fe3fab02-c9e8-4b16-aac1-7a1e56e6b49a)
- **Usefulness:** 5/5

## Summary

Webinfos for WordPress is vulnerable to unauthenticated arbitrary file upload in all versions up to and including 1.2. The issue is caused by missing file type validation, allowing an unauthenticated attacker to upload arbitrary files to the server. This can lead to remote code execution. No known patch is available, so affected sites should remove the plugin or apply compensating controls.

## Impact

- **Site owners / administrators**: If Webinfos <= 1.2 is installed, uninstall it or replace it with a maintained alternative. No vendor patch is listed.
- **Hosting / platform teams**: Review web roots and upload directories for unexpected executable files, and consider blocking direct execution of uploaded files or restricting upload paths at the server/WAF layer.
- **Plugin & theme developers**: Do not rely on Webinfos for file upload handling; any custom integration should enforce strict file type validation and authorization.
- **No action required** only if the plugin is not installed or is already removed.

## Technical details

The advisory describes a missing file type validation flaw in Webinfos <= 1.2 that permits unauthenticated arbitrary file upload. The source does not identify a specific function, hook, REST route, or file path. The practical effect is that an attacker can place arbitrary files on the server, which may enable remote code execution depending on server configuration and file handling.

## Contribution

The record carries no discussion detail beyond the advisory.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
