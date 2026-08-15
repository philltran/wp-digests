# WordPress 7.0.4 Release

- **Source:** WordPress News
- **Type:** Blog post
- **Author:** John Blackbourn
- **Published:** 2026-08-12
- **Tags:** `Releases`, `Security`, `minor-releases`, `releases`
- **Link:** [https://wordpress.org/news/2026/08/wordpress-7-0-4-release/](https://wordpress.org/news/2026/08/wordpress-7-0-4-release/)
- **Usefulness:** 5/5

## Summary

This WordPress security release fixes an authenticated Author+ remote code execution vulnerability triggered by a malicious file upload on sites that use Imagick and Ghostscript. The announcement recommends updating immediately because the issue allows remote code execution. The fix is also being backported to the 4.7 branch and the 7.1 RC3 release.

## Impact

- **Site owners:** Update to the latest security release immediately; automatic background updates may apply it.
- **Plugin and theme developers:** No API, hook, or configuration change is described; no code change is required unless a plugin or theme independently handles the affected file-upload path.
- **Hosting and platform teams:** Apply WordPress updates promptly; the vulnerability is tied to environments using Imagick and Ghostscript, and only the most recent WordPress version is actively supported.
- **Headless and REST consumers:** No REST route, schema, or API change is described.

## Technical details

The announcement does not provide a diff, file paths, function names, hooks, or REST schema changes. It describes the shipped behavior change as a security fix for an authenticated Author+ remote code execution issue via malicious file upload on sites that use Imagick and Ghostscript. The release notes also state that the fix is being backported to the 4.7 branch and the 7.1 RC3 release.

## Contribution

The record is a release announcement rather than a ticket or pull request, so it carries no discussion detail beyond contributor credits.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
