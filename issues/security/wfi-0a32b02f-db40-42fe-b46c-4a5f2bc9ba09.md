# Elementor Pro <= 4.2.1 - Unauthenticated Arbitrary File Upload via Upload Field Array Validation Bypass

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-19
- **CVE:** [CVE-2026-32475](https://www.cve.org/CVERecord?id=CVE-2026-32475)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-434: Unrestricted Upload of File with Dangerous Type
- **Affected:** Elementor Website Builder Pro (plugin) <= 4.2.1
- **Patched in:** 4.2.2
- **Researchers:** Tin Pham (TF1T), Austin Ginder
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/0a32b02f-db40-42fe-b46c-4a5f2bc9ba09](https://www.wordfence.com/threat-intel/vulnerabilities/id/0a32b02f-db40-42fe-b46c-4a5f2bc9ba09)
- **Usefulness:** 5/5

## Summary

Elementor Pro's Form widget file-upload validation can be bypassed by unauthenticated users. In `Upload::validation()`, the loop over files in a multi-file field uses `return` instead of `continue` when the first array element reports `UPLOAD_ERR_NO_FILE`. That aborts the extension and file-type checks for every remaining file in the same field, so a disallowed file (for example a PHP script) can be uploaded, potentially leading to remote code execution. Fixed in Elementor Pro 4.2.2.

## Impact

**Site owners**
- Update Elementor Pro to 4.2.2 or later immediately. Sites on <= 4.2.1 are exposed to unauthenticated file upload and possible RCE.
- Exposure requires a published page containing an Elementor Pro Form widget with at least one **non-required** File Upload field. Sites with no such form are not reachable via this path.
- If you cannot update right away, temporarily unpublish pages with Form widgets that have File Upload fields, or remove those fields. This is a stopgap inferred from the preconditions, not a vendor-documented mitigation.
- Review upload directories for unexpected executable files (e.g. `.php`, `.phtml`), particularly on sites that ran a vulnerable version with public upload forms.

**Agencies / hosting & platform teams**
- Audit managed fleets for Elementor Pro <= 4.2.1 and prioritise sites with public forms that include file upload fields.
- Consider blocking PHP execution in the Elementor form uploads directory as defence in depth.

**Plugin & theme developers**
- No API changes. Custom code hooking into Elementor Pro form processing is not documented as affected, but verify behavior after upgrading.

## Technical details

The record attributes the flaw to the `process_field` code path, with the root cause in `Upload::validation()` (the Elementor Pro Forms upload field class).

The validation iterates over the entries of the uploaded file array for a field. When the first element has error code `UPLOAD_ERR_NO_FILE`, which is expected for an optional, empty file slot, the loop executes `return` instead of `continue`. The whole validation routine exits early, so extension and file-type checks are never applied to later elements of the same field. An attacker submits the field as an array with an empty first entry followed by a disallowed file, and the payload passes unchecked.

Conceptually:

```php
// vulnerable pattern
foreach ( $files as $file ) {
    if ( UPLOAD_ERR_NO_FILE === $file['error'] ) {
        return; // aborts validation of all remaining files
    }
    // extension / type checks ...
}

// correct pattern
foreach ( $files as $file ) {
    if ( UPLOAD_ERR_NO_FILE === $file['error'] ) {
        continue; // skip only the empty slot
    }
    // extension / type checks ...
}
```

The snippet is illustrative of the described `return`/`continue` behavior; the source record does not include the actual diff. The required-field case is not affected because a required empty field fails earlier. No hooks, REST routes, or schema changes are mentioned.

## Contribution

The record carries no discussion detail beyond the researcher credits and the patch release.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
