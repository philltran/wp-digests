# CryptoPayment Gateway 1.2.1 - 1.2.2 - Unauthenticated Arbitrary File Deletion

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-09-10
- **CVE:** [CVE-2026-81648](https://www.cve.org/CVERecord?id=CVE-2026-81648)
- **CVSS:** 9.1 (Critical)
- **CWE:** CWE-22: Improper Limitation of a Pathname to a Restricted Directory ('Path Traversal')
- **Affected:** CryptoPayment Gateway (plugin) >= 1.2.1 and <= 1.2.2
- **Researchers:** Pedro Pinho
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/43fbc5df-dd8c-4a1d-92e9-5f3881cfb2e0](https://www.wordfence.com/threat-intel/vulnerabilities/id/43fbc5df-dd8c-4a1d-92e9-5f3881cfb2e0)
- **Usefulness:** 5/5

## Summary

The CryptoPayment Gateway plugin, in versions 1.2.1 through 1.2.2, fails to properly validate file paths before deleting files. An unauthenticated attacker can delete arbitrary files on the server. Deleting a file such as `wp-config.php` can lead to remote code execution, for example by forcing the site back into the installer. No patched version is listed.

## Impact

**Site owners**
- Any site running CryptoPayment Gateway 1.2.1 or 1.2.2 is exposed to unauthenticated attackers. No login or privileges are needed.
- Wordfence lists **no known patch**. Its guidance is to review the details and apply mitigations based on your risk tolerance, and it suggests that uninstalling the plugin and finding a replacement may be the best option.
- Deactivating the plugin is the minimum step, and removing it is safer. Confirm the plugin's files are no longer present on disk.

**Plugin & theme developers / agencies**
- Audit client sites and managed fleets for this plugin at the affected versions.
- If the plugin cannot be removed immediately, consider blocking access to its public endpoints at the WAF or web server. The source record does not identify those endpoints, so you would need to determine them from the plugin code.
- Check for signs of prior exploitation: missing core or config files (especially `wp-config.php`) and unexpected reinstall prompts.

**Hosting & platform teams**
- File-deletion-to-RCE chains are practical against unpatched sites. Consider adding WAF rules or making sensitive files read-only where your platform allows it.

**Headless & REST consumers**
- The record does not say whether the vulnerable path is exposed through REST, admin-ajax, or another entry point, so no REST-specific guidance can be given.

## Technical details

The source record is sparse. It states only that the vulnerability is **CWE-22 (path traversal)**, caused by "insufficient file path validation," and that it allows **arbitrary file deletion** by unauthenticated attackers in versions 1.2.1–1.2.2.

The record does not name the vulnerable function, hook, endpoint, or parameter, and no diff or patch is available. The likely pattern is a user-controlled path passed to a deletion call such as `unlink()` without normalization or containment checks, but that is an inference and not stated in the source.

The CVSS score is 9.1 (Critical). The record gives the impact as remote code execution when the right file is deleted, with `wp-config.php` as the example.

There is no fix to describe.

## Contribution

The record carries no discussion detail, and no patch has been published for the affected versions.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
