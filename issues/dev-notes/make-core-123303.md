# WordPress 7.0 Release Day Process

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Amy Kamala
- **Published:** 2026-05-20
- **Tags:** `General`, `7.0`, `development`, `dry run`, `releases`
- **Link:** [https://make.wordpress.org/core/2026/05/20/wordpress-7-0-release-day-process/](https://make.wordpress.org/core/2026/05/20/wordpress-7-0-release-day-process/)

## Summary

This post outlines the operational timeline for the WordPress 7.0 release, confirming the conclusion of the dry run and the commencement of a mandatory 24-hour code freeze on the `7.0` Subversion branch. The release will be finalized during an online release party on May 20, with the primary focus shifting to validating RC4 package integrity across diverse server environments before public distribution.

## Impact

- **Site Owners & Plugin/Theme Developers**: No functional changes or API updates are introduced by this process announcement. No code modifications or configuration shifts are required.
- **Server Administrators & QA Testers**: Should validate WP 7.0 RC4 packages against less common stacks (IIS) and older PHP/MySQL versions. Verify single/multisite upgrade paths, core install flows (including `wp-config.php` removal), and block/plugin rendering to catch regressions before the final rollout.
- **No action required** for standard site operations outside of pre-release validation efforts.

## Technical details

This entry documents a release process update rather than a code change; no files, hooks, classes, or APIs are modified in the `7.0` branch. The announcement specifies that RC4 packages (built via the final package methodology) are available for testing. Key validation scopes documented include:
- Manual and WP-CLI/one-click install success.
- Upgrade paths verifying the removal of legacy files tracked in `$_old_files`.
- Multisite (subdirectory/subdomain) network upgrade integrity.
- Block rendering, commenting, language switching, and plugin/theme extension compatibility during both new installs and upgrades.

## Contribution

Drafted by Amy Kamala on May 20, 2026, mapping the dry run conclusion to the final release party window. The post was editorially assisted by @jeffpaul. It synthesizes the release squad's timeline for the `7.0` branch, coordinating community QA efforts around the RC4 packages without introducing technical deviations or alternative merge strategies.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
