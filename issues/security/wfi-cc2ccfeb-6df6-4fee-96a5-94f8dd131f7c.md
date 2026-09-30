# The Events Calendar <= 6.17.3 - Unauthenticated Code Injection to Remote Code Execution via Widget 'classes' Map Callable Invocation

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-09-11
- **CVE:** [CVE-2026-78159](https://www.cve.org/CVERecord?id=CVE-2026-78159)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-94: Improper Control of Generation of Code ('Code Injection')
- **Affected:** The Events Calendar (plugin) <= 6.17.3
- **Patched in:** 6.17.3.1
- **Researchers:** Chloe Chamberland, Wordfence Argus
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/cc2ccfeb-6df6-4fee-96a5-94f8dd131f7c](https://www.wordfence.com/threat-intel/vulnerabilities/id/cc2ccfeb-6df6-4fee-96a5-94f8dd131f7c)
- **Usefulness:** 5/5

## Summary

The Events Calendar through 6.17.3 contains an unauthenticated remote code execution flaw. A plain-array payload in a widget `classes` map bypasses the `is_safe_widget_instance()` object check and reaches a callable-invocation sink in `Element_Classes::parse_array()`. Version 6.17.3.1 patches it.

## Impact

**Site owners**
- Update The Events Calendar to 6.17.3.1 or later immediately. The CVSS is 9.8 and the attacker needs no authentication.
- Exploitation requires comments enabled on `tribe_events` posts and at least one submitted comment containing a crafted `wp:legacy-widget` block. The payload fires when `do_blocks()` renders the single-event HTML, including the comment area.
- If you cannot update right away, disabling comments on `tribe_events` posts removes a precondition described in the advisory. The advisory does not offer this as an official mitigation, so treat it as a stopgap only.
- Sites that have run affected versions with open event comments should review existing comments on event posts for `wp:legacy-widget` block markup and check for signs of compromise.

**Plugin & theme developers / agencies**
- Audit client sites and fleets for The Events Calendar <= 6.17.3.
- The pattern is relevant beyond this plugin. User-supplied comment content that reaches `do_blocks()` can instantiate blocks such as `core/legacy-widget`, so any callable sink reachable from block attributes is exposed.

**Hosting & platform**
- Consider WAF rules or scanning for `wp:legacy-widget` in comment content on sites running this plugin.

**Headless & REST consumers**
- The advisory describes the trigger as server-side rendering of the single-event HTML. It does not discuss REST exposure.

## Technical details

Per the advisory, the root cause is insufficient validation of the widget `classes` map.

- `is_safe_widget_instance()` checks whether a widget instance is an object. A plain-array payload is not caught by that check.
- The array then reaches `Element_Classes::parse_array()`, which invokes it as a callable. This is the sink.
- The attack chain begins when `do_blocks()` processes the single-event HTML, including the comment area. A comment containing a crafted `wp:legacy-widget` block supplies the payload.
- Weakness class: CWE-94, code injection.

The record does not include the patch diff, so the exact fix in 6.17.3.1 is not described here.

## Contribution

The record carries no discussion detail beyond the credit and the patched version.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
