# Tutor LMS <= 4.0.5 - Unauthenticated Arbitrary Zero-Argument Function Invocation

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-26
- **CVE:** [CVE-2026-19092](https://www.cve.org/CVERecord?id=CVE-2026-19092)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-94: Improper Control of Generation of Code ('Code Injection')
- **Affected:** Tutor LMS – eLearning and online course solution (plugin) <= 4.0.5
- **Patched in:** 4.0.6
- **Researchers:** Jakub Herman
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/37848512-fbdf-48cd-9d59-5bfb1bdbe73d](https://www.wordfence.com/threat-intel/vulnerabilities/id/37848512-fbdf-48cd-9d59-5bfb1bdbe73d)
- **Usefulness:** 5/5

## Summary

Tutor LMS through 4.0.5 has an unauthenticated remote code execution flaw. A template-rendering path calls `extract()` on a request-influenced `$data` array, which lets an attacker overwrite the internal `$method_map` lookup table. A dynamic `is_callable()` dispatch then invokes whatever zero-argument PHP function the attacker names and returns its output. Version 4.0.6 fixes it.

## Impact

**Site owners**
- Update Tutor LMS to 4.0.6 or later immediately. The flaw is exploitable without authentication, so any site running <= 4.0.5 with the plugin active is exposed.
- Because attackers get the function's output back, zero-argument calls such as `phpinfo()` can disclose server and configuration details. Treat exposed sites as potentially compromised and review logs for unusual requests to Tutor LMS endpoints.

**Plugin & theme developers / agencies**
- Audit custom code or add-ons that pass request-derived data into Tutor LMS template-loading helpers or that copy the plugin's `extract()` pattern.
- No API changes are documented in the advisory, so no code migration is stated.

**Hosting & platform teams**
- Consider WAF rules or virtual patching until sites are updated, and check fleet-wide plugin versions.

## Technical details

The advisory describes a two-part weakness (CWE-94):

1. `extract()` is applied to a `$data` template variable that contains request-controlled values. By default `extract()` overwrites existing local variables, so an attacker can shadow the local `$method_map` callable lookup table.
2. A dynamic `is_callable()` dispatch then uses the shadowed `$method_map` to choose what to call. Because the callable is attacker-supplied and no arguments are passed, any zero-argument PHP function can be invoked and its output returned.

The advisory does not name the specific file, function, or request parameter involved, and no diff is provided. The fixed release is 4.0.6. The usual remediation for this class of bug is to avoid `extract()` on untrusted input, or to use `EXTR_SKIP`, and to restrict dispatch to a fixed allowlist, but the record does not confirm which approach 4.0.6 takes.

## Contribution

The record carries no discussion detail beyond the researcher credit and the patched version.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
