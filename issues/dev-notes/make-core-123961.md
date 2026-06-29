# WordPress 7.0.1 Release Schedule

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Aaron Jorbin
- **Published:** 2026-06-18
- **Tags:** `General`, `7.0`, `7.0.1`, `7.0.x`, `minor releases`
- **Link:** [https://make.wordpress.org/core/2026/06/18/wordpress-7-0-1-release-schedule/](https://make.wordpress.org/core/2026/06/18/wordpress-7-0-1-release-schedule/)
- **Usefulness:** 2/5

## Summary

WordPress Core published the maintenance release schedule for 7.0.1, outlining a sequential bug scrub cadence from June 18 through July 9, 2026, culminating in a general release on July 9. The post establishes 7.0.1 as a strict bug-fix maintenance release scoped to regressions introduced during the 7.0 cycle or intentionally deferred tickets, while also coordinating locale string translation updates.

## Impact

- Site owners & maintainers: Monitor the linked Trac report and GitHub repository for critical bugs; a community-maintained hotfix plugin is already available for one reported issue if immediate remediation is needed before upgrading.
- Plugin & theme developers: No action required. This release is explicitly scoped to bug fixes rather than new features, APIs, or behavioral changes.
- Release coordinators & platform teams: Align internal testing windows with the published scrub schedule and monitor the #core Slack room for RC availability announcements; coordinate any patch contributions via the 7.0.x editor tasks board.

## Technical details

The upstream item contains no code diffs, function modifications, or API alterations. It documents a procedural timeline (June 18–July 9, 2026) for the 7.0.1 maintenance cycle with co-leadership assigned to @cbravobernal, @estelaris, @masteradhoc, and @jorbin. The only technical references are a deferred hotfix plugin (specific functionality not detailed in the post) and the inclusion of locale string translations alongside core bug fixes. No hooks, REST endpoints, block.json schemas, or configuration constants are changed.

## Contribution

The schedule was authored by @jorbin with editorial assistance from @cbravobernal, @masteradhoc, and @estelaris, and coordination support from @jeffpaul, @annezazu, and @4thubbard. The release co-lead team is formally established for the 7.0.1 cycle. Contribution efforts are directed to Trac report #4 (7.0.x editor tasks) for proposed fixes and GitHub for code reviews, with bug scrub cadence finalized through Slack coordination in #core and #7-0-release-leads.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
