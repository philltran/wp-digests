# Attackers Actively Exploiting Critical Vulnerability in Super Forms Plugin

- **Source:** Wordfence Blog
- **Type:** Blog post
- **Author:** unknown
- **Published:** 2026-09-03
- **Tags:** `Threat Research`, `Vulnerabilities`, `WordPress Security`, `WordPress Plugin Vulnerability News September 2026`, `WordPress Vulnerability News September 2026`
- **Link:** [https://www.wordfence.com/blog/2026/09/attackers-actively-exploiting-critical-vulnerability-in-super-forms-plugin/](https://www.wordfence.com/blog/2026/09/attackers-actively-exploiting-critical-vulnerability-in-super-forms-plugin/)
- **Usefulness:** 5/5

## Summary

Wordfence reports active, in-the-wild exploitation of a critical unauthenticated arbitrary file upload in the Super Forms – Drag & Drop Form Builder plugin (`super-forms`, ~13,000 active installs). The flaw lets unauthenticated attackers upload executable files such as PHP backdoors through the `submit_form` AJAX handler, leading to remote code execution. The vendor fixed it in 6.3.314, and Wordfence says its firewall has blocked over 250,000 exploit attempts.

## Impact

**Site owners**
- Any site running Super Forms `<= 6.3.313` is vulnerable and should update to `6.3.314` or later immediately.
- Because exploitation has been observed since July 14, 2026, sites that ran a vulnerable version during that window should be checked for compromise (unexpected PHP or other executable files in upload locations, unknown admin users, modified files).
- Firewall rules are not a substitute for patching. Wordfence Premium, Care and Response users got a rule on July 14, 2026. Free users got it 30 days later, on August 13, 2026.

**Agencies / hosting & platform teams**
- Inventory client sites for `super-forms` and confirm the installed version.
- Consider blocking PHP execution in upload directories at the web server level as a general mitigation for arbitrary-upload bugs.

**Plugin developers**
- No API changes. The bug class is a reminder to check capabilities and file types on `nopriv` AJAX handlers, and not to treat a nonce that unauthenticated users can freely obtain as an authorization barrier.

## Technical details

Per the advisory, the vulnerable code path is the `submit_form` function, registered as a `nopriv` AJAX handler.

- **Missing file type validation.** Uploaded content supplied through the `data` parameter (`datauristring` / `value`) is not validated, so executable files can be written.
- **No capability check.** The `submit_form` handler performs no capability check.
- **Weak nonce barrier.** The only gate is a session nonce that unauthenticated visitors can get from a separate `nopriv` endpoint. The advisory calls this requirement trivial to satisfy. The excerpt is truncated at this point, so the remaining detail is not available.

The patched version is 6.3.314. I did not see the diff or the exact fix in the provided excerpt, so how validation was added is not described here.

## Contribution

The record shows the vendor patched on July 8, 2026 and Wordfence disclosed the next day. Exploitation began on July 14, the same day the Premium firewall rule shipped, and the free-tier rule followed 30 days later. The excerpt contains no design debate.

---

*Reported by Wordfence. Summarized from their published advisory — see the linked original for the full analysis. Wordfence is a trademark of Defiant, Inc.; this digest is not affiliated with or endorsed by them.*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
