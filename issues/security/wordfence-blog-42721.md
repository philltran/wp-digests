# 600,000 WordPress Sites Affected by Arbitrary File Upload Vulnerability in Forminator Forms WordPress Plugin

- **Source:** Wordfence Blog
- **Type:** Blog post
- **Author:** unknown
- **Published:** 2026-08-17
- **Tags:** `Research`, `Vulnerabilities`, `WordPress Security`
- **Link:** [https://www.wordfence.com/blog/2026/08/600000-wordpress-sites-affected-by-arbitrary-file-upload-vulnerability-in-forminator-forms-wordpress-plugin/](https://www.wordfence.com/blog/2026/08/600000-wordpress-sites-affected-by-arbitrary-file-upload-vulnerability-in-forminator-forms-wordpress-plugin/)
- **Usefulness:** 5/5

## Summary

Forminator Forms fixed a critical unauthenticated arbitrary file upload vulnerability, tracked as CVE-2026-15748, that allowed attackers to upload arbitrary files, including executable PHP files, to a vulnerable site. The flaw could lead to remote code execution and full site compromise. It was exploitable only when a Forminator form contained both a File Upload field and a Select field. The patched release is 1.56.2.

## Impact

- **Site owners:** Update Forminator Forms to 1.56.2 as soon as possible. Sites running versions <= 1.56.1 are exposed, especially if any form includes both a File Upload field and a Select field.
- **Plugin and theme developers:** No Forminator API change is described, but any integration that relies on Forminator file-upload behavior should be reviewed after updating.
- **Hosting and platform teams:** Prioritize patching affected installations; the advisory notes Wordfence’s built-in Malicious File Upload protection blocks exploits targeting this vulnerability.
- **Headless and REST consumers:** No REST route, schema, or API change is described in the advisory.

## Technical details

The advisory describes an unauthenticated arbitrary file upload via forged upload field configuration in Forminator Forms. The affected range is <= 1.56.1, and the fixed version is 1.56.2. The exploit precondition is a form containing both a File Upload field and a Select field. No diff, function names, hooks, or file paths are provided in the source material, so the exact code-level fix cannot be identified from the advisory alone.

## Contribution

The advisory records a disclosure sequence: Wordfence received the report, provided details to the Forminator team, the developer acknowledged the report, and a patched release followed. No design debate, rejected alternative, or code-level review detail is included in the source.

---

*Reported by Wordfence. Summarized from their published advisory — see the linked original for the full analysis. Wordfence is a trademark of Defiant, Inc.; this digest is not affiliated with or endorsed by them.*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
