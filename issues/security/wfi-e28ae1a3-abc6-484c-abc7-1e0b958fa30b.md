# AI Copilot – Content Generator <= 1.5.6 - Unauthenticated Privilege Escalation via Custom Workflow Route

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-07
- **CVE:** [CVE-2026-14526](https://www.cve.org/CVERecord?id=CVE-2026-14526)
- **CVSS:** 9.8 (Critical)
- **CWE:** CWE-269: Improper Privilege Management
- **Affected:** AI Copilot – Content Generator (plugin) <= 1.5.6
- **Researchers:** d.v4n_s3c
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/e28ae1a3-abc6-484c-abc7-1e0b958fa30b](https://www.wordfence.com/threat-intel/vulnerabilities/id/e28ae1a3-abc6-484c-abc7-1e0b958fa30b)
- **Usefulness:** 5/5

## Summary

The AI Copilot – Content Generator plugin has an authorization bypass that allows unauthenticated visitors to create an administrator account. The issue occurs when a malicious workflow is saved and executed through the plugin's custom workflow route, using a `wp_create_user` action node with `role=administrator`. The plugin's nonce check is not a reliable authorization barrier because the `waic-nonce` value is exposed in public frontend JavaScript as `WAIC_DATA.waicNonce` when the `[aiwu-form]` shortcode or public chatbot is rendered. No patched version is listed, so affected sites should treat this as an active site-takeover risk.

## Impact

- **Site owners:** Immediate action is required. Uninstall or disable the plugin, remove any `[aiwu-form]` shortcode or public chatbot from frontend pages, and audit user accounts and workflow data for unauthorized administrator creation.
- **Plugin & theme developers:** Do not rely on the exposed `waic-nonce` value as an authorization control. If integrating with or extending this plugin, ensure workflow save and execution paths enforce server-side capability checks.
- **Hosting & platform teams:** Block or quarantine the plugin where possible, monitor for new administrator accounts, and inspect stored workflow entries for malicious `wp_create_user` nodes.
- **Headless & REST consumers:** No direct API change is described, but any frontend surface that renders the plugin's form or chatbot should be treated as an exposure vector.

## Technical details

The plugin does not properly verify that the requesting user is authorized to perform workflow-related actions. An unauthenticated attacker can submit a malicious workflow containing a `wp_create_user` action node with `role=administrator`, causing the plugin to create an administrator-level user account. The advisory identifies the exposure as tied to frontend rendering: when the `[aiwu-form]` shortcode or public chatbot is present, the `waic-nonce` value is emitted into publicly accessible JavaScript as `WAIC_DATA.waicNonce`, making the nonce check ineffective as an authorization barrier. The record does not provide a specific route path, payload, or patched version.

## Contribution

The advisory record carries no discussion detail about vendor response, remediation, or alternative fixes.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/ Copyright 1999-2026 The MITRE Corporation. CVE Usage: MITRE hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute Common Vulnerabilities and Exposures (CVE®). Any copy you make for such purposes is authorized provided that you reproduce MITRE's copyright designation and this license in any such copy. Licence terms: https://www.cve.org/Legal/TermsOfUse*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
