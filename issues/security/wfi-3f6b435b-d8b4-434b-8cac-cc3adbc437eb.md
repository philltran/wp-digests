# WordPress Core <= 7.0.3 - Authenticated (Author+) Remote Code Execution via Malicious File Upload

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-12
- **CVE:** [CVE-2026-65640](https://www.cve.org/CVERecord?id=CVE-2026-65640)
- **CVSS:** 8.8 (High)
- **CWE:** CWE-434: Unrestricted Upload of File with Dangerous Type
- **Affected:** WordPress (core) >= 4.7.0 and <= 4.7.34; >= 4.8.0 and <= 4.8.29; >= 4.9.0 and <= 4.9.30; >= 5.0.0 and <= 5.0.26; >= 5.1.0 and <= 5.1.23; >= 5.2.0 and <= 5.2.25; >= 5.3.0 and <= 5.3.22; >= 5.4.0 and <= 5.4.20; >= 5.5.0 and <= 5.5.19; >= 5.6.0 and <= 5.6.18; >= 5.7.0 and <= 5.7.16; >= 5.8.0 and <= 5.8.14; >= 5.9.0 and <= 5.9.15; >= 6.0.0 and <= 6.0.13; >= 6.1.0 and <= 6.1.11; >= 6.2.0 and <= 6.2.10; >= 6.3.0 and <= 6.3.9; >= 6.4.0 and <= 6.4.9; >= 6.5.0 and <= 6.5.9; >= 6.6.0 and <= 6.6.6; >= 6.7.0 and <= 6.7.6; >= 6.8.0 and <= 6.8.7; >= 6.9.0 and <= 6.9.6; >= 7.0.0 and <= 7.0.3
- **Patched in:** 4.7.35, 4.8.30, 4.9.31, 5.0.27, 5.1.24, 5.2.26, 5.3.23, 5.4.21, 5.5.20, 5.6.19, 5.7.17, 5.8.15, 5.9.16, 6.0.14, 6.1.12, 6.2.11, 6.3.10, 6.4.10, 6.5.10, 6.6.7, 6.7.7, 6.8.8, 6.9.7, 7.0.4
- **Researchers:** pwn.ai Team
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/3f6b435b-d8b4-434b-8cac-cc3adbc437eb](https://www.wordfence.com/threat-intel/vulnerabilities/id/3f6b435b-d8b4-434b-8cac-cc3adbc437eb)
- **Usefulness:** 5/5

## Summary

WordPress core's Imagick image editor accepted uploaded files whose real content did not match their WordPress-approved extension. `WP_Image_Editor_Imagick::load()` handed the file (or stream) to Imagick without sufficient validation, so ImageMagick could identify it as a PostScript-family or compressed delegate format from magic bytes, an embedded format specifier, or decompressed content — and dispatch it to an external delegate such as Ghostscript. Because this happens automatically during attachment metadata generation, any user with `upload_files` (Author and above) could upload a crafted "image" and reach remote code execution on sites running the Imagick editor with Ghostscript available. The patched releases tighten validation in that load path so the format Imagick acts on is constrained rather than inferred from attacker-controlled content.

## Impact

- **Site owners:** High-severity RCE reachable by any Author-, Editor-, Administrator- or Contributor-with-upload account, plus any compromised or self-registered account escalated to those roles. Multi-author sites, membership sites, and anything with loosely trusted contributors should treat this as an urgent update. Update to the patched release on your branch — every supported branch back to 4.7 received a fix.
- **Hosting & platform teams:** Exposure depends on the image editor in use. Sites where `WP_Image_Editor_Imagick` is selected **and** the ImageMagick build has a working Ghostscript delegate are the RCE case; stacks without Imagick (GD-only) or without the Ghostscript delegate are not exploitable for code execution via this path. Fleet-wide auto-update should be verified as having landed, since the backport list spans 24 branches.
- **Plugin & theme developers:** If your code accepts uploads from unauthenticated or low-privilege users and pushes them through `wp_handle_upload()` / `wp_generate_attachment_metadata()` / `wp_get_image_editor()`, you may convert this into an unauthenticated RCE on an unpatched site. The advisory explicitly notes unauthenticated exploitation is deployment-dependent and requires exactly such a public-upload path. Audit front-end upload forms, importers, and REST endpoints that generate attachment metadata.
- **No API changes.** There is no deprecation, no signature change, and no configuration migration — patching is the whole action.

## Technical details

The flaw is in the Imagick backend of the image-editor abstraction, `WP_Image_Editor_Imagick::load()` (`wp-includes/class-wp-image-editor-imagick.php`). WordPress gates uploads on extension/MIME (via `wp_check_filetype_and_ext()` and the `upload_mimes` allowlist), but once a file passed that gate it was handed to Imagick, which performs its own content-based format detection. ImageMagick will identify a file as a PostScript-family format (PS/EPS/PDF) or a compressed delegate format from magic bytes, an inline `format:` specifier, or the decompressed payload — none of which have to agree with the `.jpg`/`.png` extension WordPress validated. For PostScript-family input ImageMagick shells out to an external delegate (Ghostscript), which is where the crafted content gains code execution.

The trigger is entirely passive from the attacker's perspective: `wp_generate_attachment_metadata()` runs on upload and calls `wp_get_image_editor()`, which instantiates the Imagick editor and calls `load()` to read dimensions and generate the intermediate sizes. No second request, no admin action, and no rendering of the file on the front end is required.

The patch constrains what Imagick is allowed to interpret in that load path, so the decision is driven by the WordPress-accepted type rather than by attacker-supplied magic bytes or decompressed content.

For unpatched sites that cannot update immediately, the exposure is specific to the Imagick backend, so forcing the GD editor removes it (at the cost of Imagick's quality and format coverage):

```php
// Defense-in-depth for an unpatched site: never select the Imagick editor.
add_filter( 'wp_image_editors', static function () {
	return array( 'WP_Image_Editor_GD' );
} );
```

Hardening the ImageMagick `policy.xml` to deny the `PS`, `EPS`, `PDF`, and `XPS` coders is the equivalent mitigation at the platform layer. Neither substitutes for the core update, which is the actual fix.

## Contribution

The public record is an advisory only — it carries the researcher credit to the pwn.ai Team and the branch-by-branch fixed-version list, with no ticket discussion, patch link, or design debate disclosed. The one notable operational detail is the breadth of the response: the fix was backported across 24 release branches simultaneously, from 4.7 through 7.0, which is the pattern WordPress uses for security releases that reach the automatic background-update channel.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
