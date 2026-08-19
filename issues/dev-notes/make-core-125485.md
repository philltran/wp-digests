# WordPress 7.1 Release Day Process

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Krupa Nanda
- **Published:** 2026-08-17
- **Tags:** `Updates`, `7.1`, `development`, `dry run`, `releases`
- **Link:** [https://make.wordpress.org/core/2026/08/17/wordpress-7-1-release-day-process/](https://make.wordpress.org/core/2026/08/17/wordpress-7-1-release-day-process/)
- **Usefulness:** 3/5

## Summary

WordPress 7.1 is shipping: RC4 went out on August 17, 2026 at 15:00 UTC and was followed immediately by the Dry Run, with the general release published on August 19, 2026 starting at 23:10 UTC from the WordCamp US stage in Phoenix. An **extended** mandatory code freeze on the `7.1` branch runs from August 17 at 14:00 UTC straight through to GA — longer than the usual 24 hours, at the Core Committers' request, so the packaged release can be verified in full before it ships. Any critical bug reported during the freeze is triaged by the release squad with committers and maintainers: a confirmed blocker means a new RC and a full restart of the Dry Run and freeze, while anything else is deferred to 7.1.1. The post also lays out the specific install, upgrade, and multisite scenarios the project wants community testers to cover against the RC packages.

## Impact

**Plugin & theme developers**
- The `7.1` branch is closed to commits from August 17, 14:00 UTC until GA — nothing you land now reaches 7.1.0, and committing does not reopen after the Dry Run completes. Anything not judged a blocker is retargeted at 7.1.1.
- Test your plugins and themes against the RC packages now: they are built with the same method as the final packages, so an RC pass is a meaningful signal for the 7.1.0 zip.
- If you find a critical regression during the freeze, escalate it to the release squad rather than assuming it will be picked up — blocker status is decided jointly with committers and maintainers, and a blocker restarts the whole release process.

**Hosting & platform teams**
- Plan for 7.1.0 to be published August 19, 2026 from 23:10 UTC. Auto-update fleets, mirrors, and support staffing should be scheduled around that window, allowing for the fact that a major release takes materially longer to publish than a beta or RC.
- The project is specifically asking for package testing on less common server configurations — **IIS in particular** — and on older PHP and MySQL versions. If you operate that kind of estate, testing the RC zip there is the highest-value contribution.

**Site owners**
- No action required beyond normal major-release hygiene: verify a staging upgrade against an RC before GA.

## Technical details

This is a release-process post, not a code change; the operative details are the freeze scope and the testing matrix.

**Freeze scope.** No source code for 7.1.0 — i.e. anything in the `7.1` branch in Subversion — may change between August 17, 2026 14:00 UTC and the general release on August 19. The freeze deliberately starts one hour ahead of the 15:00 UTC RC4 build so the tree is quiet before packaging, and the Dry Run follows the RC immediately.

**Blocker path.** If a critical bug surfaces during the freeze, the release squad convenes with committers and maintainers. A blocker triggers a new RC, which restarts the process from the top — the Dry Run repeats and the code freeze begins again. A non-blocker is milestoned to 7.1.1.

**Requested test coverage.** The post enumerates the scenarios most likely to catch packaging-level breakage:

- Fresh install via the manual five-minute install, WP-CLI, and one-click installers.
- Upgrades from a range of prior versions.
- Deleting `wp-config.php` and running a clean install.
- Single site and multisite, both subdirectory and subdomain networks, including the network upgrade path.
- Verifying that files listed in `$_old_files` are actually removed during upgrade — the classic source of stale-file bugs after a major.
- Desktop and mobile flows: publishing a post using a variety of blocks, commenting, installing and upgrading a plugin or theme, and switching the site language.

A deeper feature-level checklist lives in the separate "Help Test WordPress 7.1" post.

## Contribution

The notable wrinkle here is that the code freeze was extended past the customary 24 hours at the Core Committers' explicit request, to give everyone time to verify the release in full before it ships — and to accommodate a release party split across two venues, with the process steps run in the `#core` Slack channel and the actual publish happening live on stage at WordCamp US. The post credits @amykamala, @benjamin_zekavica, @westonruter, @joedolson, and @wildworks for review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
