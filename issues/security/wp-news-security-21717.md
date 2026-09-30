# WordPress 7.1.1 Maintenance and Security Release

- **Source:** WordPress News
- **Type:** Blog post
- **Author:** Aaron Jorbin
- **Published:** 2026-09-17
- **Tags:** `Releases`, `Security`
- **Link:** [https://wordpress.org/news/2026/09/wordpress-7-1-1-maintenance-and-security-release/](https://wordpress.org/news/2026/09/wordpress-7-1-1-maintenance-and-security-release/)
- **Usefulness:** 5/5

## Summary

WordPress 7.1.1 is a short-cycle maintenance and security release containing 11 security fixes, 17 Core bug fixes, and 19 Block Editor bug fixes. The security fixes cover stored XSS in `wpautop()` and in themes with custom headers, an HTML API comment breakout in `set_modifiable_text()`, path traversal in the REST Templates Controller, and several authorization and capability-check gaps. The announcement recommends updating immediately. The next major release is 7.2, planned for December.

## Impact

**Site owners**
- Update immediately. Sites with automatic background updates enabled will update on their own.
- Security fixes are being backported to all branches eligible for security fixes (currently through 4.7). The announcement says backports are in progress and will ship as they become ready.

**Plugin & theme developers**
- The `wpautop()` XSS and the HTML API `set_modifiable_text()` comment breakout affect code that filters or generates content through those functions. Review any code that passes untrusted text to them.
- Themes that support custom headers were affected by a stored XSS. The announcement does not name the specific code path.
- Code that relies on `customize_changeset` posts being publishable via XML-RPC, on non-admins being able to reparent comments, or on other loose capability behavior may see different results after the fix. The post does not detail the behavior changes.

**Hosting & platform teams**
- Roll this out promptly on managed fleets. The fixes span REST, XML-RPC, the admin UI, and comments, so they are not confined to one surface.

**Action required:** update to 7.1.1. The announcement lists no deprecations or API removals.

## Technical details

The announcement is a release post and does not include diffs, so only the vulnerability classes listed below are supported by the source:

- **Stored XSS in `wpautop()`**: an unauthenticated visitor can inject script, subject to comment approval.
- **HTML API**: `set_modifiable_text()` allowed breaking out of a comment via abrupt-closing sequences.
- **Custom headers**: stored XSS in some themes that support custom headers.
- **Theme install/preview**: specially crafted URLs could automatically install and preview an inactive theme from WordPress.org.
- **Network-only plugins**: a Site Administrator could network-activate an installed Network-only plugin.
- **REST**: authenticated path traversal in the WP REST Templates Controller.
- **XML-RPC**: `customize_changeset` posts could be published in a way that bypasses the `edit_css` check.
- **Post overwrite**: Contributor+ arbitrary post overwrite.
- **`attachment_submitbox_metadata()`**: a missing `read_post` check leaked a private parent-post title.
- **Draft/pending slug disclosure**: missing authorization allowed Contributor+ users to see draft or pending post slugs.
- **Comments**: comments, including notes, could be reparented by any authenticated user.

The post does not describe the specific 17 Core and 19 Block Editor bug fixes. See the HelpHub page and the Trac milestone for that list.

## Contribution

The release was led by Adam Silverstein, Adrian Duffell, Andrei Draganescu, and Aaron Jorbin, with a long list of contributors and company representatives from Automattic, Bluehost, GoDaddy, Pantheon, and WP Engine. Backports to older branches were still in progress at publication. The post carries no design-debate detail.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
