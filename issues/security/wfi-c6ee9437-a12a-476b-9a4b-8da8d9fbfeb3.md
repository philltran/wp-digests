# Fluent Forms Pro 6.2.7 & Ninja Tables Pro 5.2.13 - Remote Code Execution via Backdoor

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-03
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-506: Embedded Malicious Code
- **Affected:** Ninja Tables Pro (plugin) >= 5.2.11 and <= 5.2.11, Fluent Forms Pro Add On Pack (plugin) >= 6.2.7 and <= 6.2.7
- **Patched in:** 5.2.13, 6.2.8
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/c6ee9437-a12a-476b-9a4b-8da8d9fbfeb3](https://www.wordfence.com/threat-intel/vulnerabilities/id/c6ee9437-a12a-476b-9a4b-8da8d9fbfeb3)
- **Usefulness:** 5/5

## Summary

Versions 6.2.7 of Fluent Forms Pro Add On Pack and 5.2.11 of Ninja Tables Pro shipped with a malicious backdoor embedded by threat actors who compromised the vendor’s build environment. The injected code enabled unauthenticated remote code execution on affected WordPress sites. Patched releases (6.2.8 and 5.2.13) remove the payload and restore secure plugin behavior.

## Impact

- **Site owners & plugin administrators**: Immediate action required. Update Fluent Forms Pro Add On Pack to ≥6.2.8 and Ninja Tables Pro to ≥5.2.13 to remove the backdoor.
- **Hosting & platform teams**: Scan existing installations for the malicious payload and enforce automated plugin updates to prevent exploitation.
- **No breaking changes or API deprecations**; this is a straightforward security patch with no migration steps.

## Technical details

The vulnerability is classified as CWE-506 (Embedded Malicious Code) with a CVSS score of 9.8. Threat actors compromised the vendor’s distribution pipeline and injected a backdoor into the affected plugin builds, allowing unauthenticated attackers to execute arbitrary code on the server. The advisory does not disclose specific file paths, function names, or hook signatures for the injected payload. Patched versions strip the malicious code and revert to the clean vendor builds.

## Contribution

The record contains no discussion detail; the advisory was published directly by Wordfence Intelligence following vendor compromise detection, with patched versions released by the plugin authors.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
