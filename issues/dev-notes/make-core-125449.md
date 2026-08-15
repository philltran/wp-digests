# WordPress 7.1 Release Candidate 3

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Krupa Nanda
- **Published:** 2026-08-12
- **Tags:** `Core`, `General`, `7.1`, `development`, `release`
- **Link:** [https://make.wordpress.org/core/2026/08/12/wordpress-7-1-release-candidate-3/](https://make.wordpress.org/core/2026/08/12/wordpress-7-1-release-candidate-3/)
- **Usefulness:** 3/5

## Summary

The third release candidate for WordPress 7.1 is available for testing and contains more than 90 updates and fixes since the first release candidate, including 37 Editor changes and 57 Core changes. It is the latest pre-release milestone before the scheduled final release on August 19, 2026, and it marks the hard string freeze for the 7.1 release cycle. Developers should evaluate it on non-production sites and report issues to Trac or the Alpha/Beta support forums.

## Impact

- **Site owners:** No production action is required; the post explicitly advises against installing or testing this release candidate on production or mission-critical sites.
- **Plugin and theme developers:** Test your products against this release candidate and update the `Tested up to` version in plugin readme files to 7.1 when compatibility is confirmed.
- **Hosting and platform teams:** Run the release candidate through your hosting environments and distributed hosting tests to catch infrastructure or upgrade-path issues before the final release.
- **Translators:** This milestone is the hard string freeze point for 7.1, so translation work should be finalized accordingly.

## Technical details

The post does not itemize individual API, hook, REST route, block, or database changes. It reports that the release candidate includes more than 90 updates and fixes since the first release candidate, with 37 changes in the Editor and 57 in Core. For specifics, it points to the Gutenberg commits for 7.1 since August 05 and the closed WordPress Core Trac tickets since August 05.

Testing can be done through the WordPress Beta Tester plugin using the “Bleeding edge” channel and “Beta/RC” stream, by installing the direct download, through WordPress Playground, or with WP-CLI:

```bash
wp core update --version=7.1-RC3
```

## Contribution

The record is a release announcement rather than a code review or pull request. It credits @benjamin_zekavica, @amykamala, @wildworks, and @annezazu for proofreading and review, and notes that the announcement was published in light of the WordPress 7.0.4 security release, with the second release candidate having shipped alongside the 7.0.3 security release on August 6, 2026.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
