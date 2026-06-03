# WordPress 7.0 Release Candidate 5

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Amy Kamala
- **Published:** 2026-05-19
- **Tags:** `General`, `7.0`, `development`, `release`
- **Link:** [https://make.wordpress.org/core/2026/05/19/wordpress-7-0-release-candidate-5/](https://make.wordpress.org/core/2026/05/19/wordpress-7-0-release-candidate-5/)

## Summary

WordPress 7.0 Release Candidate 5 (RC5) is available for testing ahead of the scheduled final release on May 20, 2026. This extra release candidate is issued to validate stability and catch remaining issues before the stable launch. Testing is strongly recommended on non-production or isolated test environments only.

## Impact

- **Developers & Testers**: Evaluate RC5 via WordPress Beta Tester (select Bleeding edge/Beta/RC streams), direct download, WP-CLI (`wp core update --version=7.0–RC5`), or WordPress Playground.
- **Production Environments**: No action required beyond standard upgrade verification; the final release remains scheduled for May 20. Refer to linked Trac tickets and Gutenberg commit logs for specific code changes and deprecations.
- **Tooling**: RC5 can be deployed via Playgrounds directly in the browser without local setup.

## Technical details

The provided upstream content is a release announcement and does not contain a unified diff or explicit API modifications. It links to 'Gutenberg commits since May 14' and 'Closed Trac tickets since May 14' as the primary sources for technical highlights, indicating bundled block editor updates are included in this RC cycle. No specific hooks, classes, database schema changes, or REST route modifications are detailed in this post.

Testing infrastructure references:
- Beta Tester plugin streams: `Bleeding edge`, `Beta`/`RC`.
- CLI command: `wp core update --version=7.0–RC5`
- No new constants, options, or block.json fields are documented in this entry.

## Contribution

Announced on May 19, 2026, by Amy Kamala via the Make WordPress Core blog. The release is designated as an 'extra RC' to maximize stability before the May 20 launch. Review and proofreading were credited to @sergeybiryukov and @huzaifaalmesbah. The post includes testing guidance and links to aggregated Trac/Gutenberg history for technical verification.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
