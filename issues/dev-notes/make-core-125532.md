# WordPress 7.1 Release Candidate 4

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Benjamin Zekavica
- **Published:** 2026-08-17
- **Tags:** `Core`, `General`, `7.1`, `development`, `release`
- **Link:** [https://make.wordpress.org/core/2026/08/17/wordpress-7-1-release-candidate-4/](https://make.wordpress.org/core/2026/08/17/wordpress-7-1-release-candidate-4/)
- **Usefulness:** 3/5

## Summary

WordPress 7.1 RC4 is available for testing and contains more than 26 updates and fixes since RC3, including 8 in the Editor and 18 in Core. It is a pre-release milestone ahead of the scheduled August 19, 2026 final release and marks the hard string freeze point for 7.1. The release is intended for evaluation on test environments, not production or mission-critical sites.

## Impact

- **Site owners:** Do not install RC4 on production or mission-critical sites; test it on a staging or test server.
- **Plugin and theme developers:** Test compatibility with 7.1 RC4 and update the `Tested up to` version in plugin readmes to `7.1` when ready.
- **Hosting and platform teams:** Use RC4 to validate upgrade paths, rollout behavior, and hosting compatibility before the final release.
- **Translators:** RC4 is the hard string freeze point, so new strings should not be expected after this milestone.
- **No code migration is specified** in the announcement; the required action is testing and compatibility verification.

## Technical details

The announcement does not enumerate specific functions, hooks, files, REST routes, block APIs, options, constants, or database changes. It reports that RC4 includes more than 26 updates and fixes since RC3, with 8 in the Editor and 18 in Core, and points to Gutenberg commits and closed Core Trac tickets since August 12 for the detailed change list. It also provides installation paths for testing, including the WordPress Beta Tester plugin, direct download, WordPress Playground, and the WP-CLI command `wp core update --version=7.1-RC4`.

## Contribution

The record carries no design debate or alternative approach; it credits @amykamala, @krupajnanda, @wildworks, and @annezazu for proofreading and review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
