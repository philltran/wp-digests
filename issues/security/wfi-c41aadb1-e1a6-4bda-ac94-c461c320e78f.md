# WordPress Core <= 7.1 - HTML API set_modifiable_text() Comment Boundary Break

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-09-17
- **CVSS:** 3.7 (Low)
- **CWE:** CWE-116: Improper Encoding or Escaping of Output
- **Affected:** WordPress (core) >= 6.7 and <= 6.7.7; >= 6.8 and <= 6.8.8; >= 6.9 and <= 6.9.7; >= 7.0 and <= 7.0.4; >= 7.1 and <= 7.1
- **Patched in:** 6.7.8, 6.8.9, 6.9.8, 7.0.5, 7.1.1
- **Researchers:** Jeremy Felt
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/c41aadb1-e1a6-4bda-ac94-c461c320e78f](https://www.wordfence.com/threat-intel/vulnerabilities/id/c41aadb1-e1a6-4bda-ac94-c461c320e78f)
- **Usefulness:** 4/5

## Summary

`WP_HTML_Tag_Processor::set_modifiable_text()` could produce HTML where text written into a comment node ended the comment early. The guard that checks for comment closers used the pattern `/--!?>/`, which catches `-->` and `--!>` but misses the abrupt-closing `-?>` form. Any text after that point would then be parsed as live markup. The fix hardens the guard in core's HTML API. The advisory describes it as developer-facing hardening: WordPress core has no code path of its own that makes this exploitable.

## Impact

**Site owners**
- Update to a patched core release. The advisory rates it Low (CVSS 3.7) and says core itself cannot be directly exploited through it.

**Plugin & theme developers**
- You are exposed only if your code passes attacker-controlled text through the HTML API's `set_modifiable_text()` into a **comment node**. On unpatched cores, that text can close the comment and inject markup or script into the output.
- If you support older cores that may not get patched, do not use `set_modifiable_text()` on comment nodes as a sanitization boundary. Validate or strip closer sequences yourself before calling it.
- If your code only uses `set_modifiable_text()` on text or other non-comment nodes, the advisory does not describe you as affected.

**Hosting & platform**
- Deliver the patch through normal minor-release or auto-update channels. No configuration change is needed.

## Technical details

- **Affected API:** `WP_HTML_Tag_Processor::set_modifiable_text()`, introduced in WordPress 6.7.0.
- **Flaw:** when the target node is an HTML comment, the method rejects or guards text that would terminate the comment. It detected closers with the regex `/--!?>/`, which covers `-->` and `--!>` only.
- **Missed case:** the abrupt-closing form `-?>`. In the HTML parsing spec, a comment whose content begins with `>` or `->` closes immediately, so text like this bypassed the guard and ended the comment early.
- **Classification:** CWE-116 (Improper Encoding or Escaping of Output).

The advisory does not include the patch diff, so the exact replacement check is not described here. There are no new hooks, filters, or API signature changes.

## Contribution

The advisory contains only credits and identifiers; there is no public discussion or design detail.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
