# Leaner, steadier PHPUnit runs for upcoming releases

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Lance Willett
- **Published:** 2026-07-30
- **Tags:** `General`
- **Link:** [https://make.wordpress.org/core/2026/07/30/leaner-steadier-phpunit-runs-for-upcoming-releases/](https://make.wordpress.org/core/2026/07/30/leaner-steadier-phpunit-runs-for-upcoming-releases/)
- **Usefulness:** 2/5

## Summary

WordPress core has optimized its GitHub Actions PHPUnit CI pipeline to reduce job volume and improve reliability ahead of the 7.1 release. The changes trim the test matrix to boundary PHP versions, drop redundant database combinations, and cache the Gutenberg build across jobs instead of rebuilding it per job. This cuts per-run job counts by roughly half and slashes rerun rates from ~68% to ~36%, freeing CI capacity for release windows and daily development.

## Impact

- **Plugin & theme developers / external CI runners:** No action required. The changes affect only WordPress core’s internal GitHub Actions workflows and do not alter public APIs, test suites, or local development commands.
- **Core contributors & release managers:** Faster PR feedback loops and reduced GitHub Actions queue times during high-traffic periods. Local PHPUnit execution remains unchanged.
- **Hosting & platform teams:** No direct impact; this is strictly a core CI optimization.

## Technical details

The changes modify the core GitHub Actions PHPUnit workflow configuration to trim the PHP version matrix to boundary versions and drop redundant database combinations (#12719, #12720). A shared artifact step now fetches and caches the Gutenberg build once per workflow run instead of rebuilding it per job (#12701), and bounded retries were added to Docker image pulls to handle transient registry timeouts (#12703). These workflow-level adjustments reduce per-run job counts by ~52% and job-minutes by ~54%, while the rerun-to-green rate drops from ~68% to ~36%. No PHPUnit test code, core functions, or public APIs are altered.

## Contribution

The CI improvements were merged across multiple pull requests ahead of the 7.1 cycle, with contributions from adrianmoldovanwp, barry, garyj, johnbillion, jonsurrell, jorbin, lucasbustamante, and mukesh27. The team deliberately prioritized matrix trimming and build caching over test-level tuning, deferring individual test execution optimization to a future phase. The changes landed sequentially to stabilize CI throughput before the release candidate window.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
