# Results: Real Time Collaboration performance testing analysis

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** annezazu
- **Published:** 2026-05-08
- **Tags:** `General`, `Summaries`, `feature-real-time-collaboration`
- **Link:** [https://make.wordpress.org/core/2026/05/08/results-real-time-collaboration-performance-testing-analysis/](https://make.wordpress.org/core/2026/05/08/results-real-time-collaboration-performance-testing-analysis/)

## Summary

Real-time collaboration (RTC) storage architecture has been officially redirected from the `post-meta` baseline to a `custom-table-with-transients` model for all future development cycles following its removal from WordPress 7.0. Performance analysis across eight distinct hosting environments confirms this hybrid pattern delivers ~52% faster dispatch latency than the previous implementation and halves database query volume, establishing it as the mandatory default strategy for any future re-implementation.

## Impact

- **Plugin & theme developers / core contributors**: No immediate code changes are required, as RTC was stripped from WP 7.0. Any future development of synchronized editing features must adopt a dedicated custom table for state storage paired with transient-backed client awareness.
- **Hosting & platform teams**: Environments without persistent object cache will still benefit from query reduction but must monitor transient cleanup overhead. `post-meta-transients` is explicitly flagged as unacceptable due to pathological latency scaling on non-cached infrastructure.
- **Action required**: None for current installations; track core or third-party RTC re-iterations where this storage pattern will be applied.

## Technical details

The analysis evaluated four RTC storage architectures under sustained 30-second polling loads: `post-meta`, `custom-table`, `post-meta-transients`, and `custom-table-with-transients`. The chosen pattern shifts collaboration state from `wp_postmeta` queries to a dedicated custom table schema, while retaining transient logic strictly for client presence/awareness tracking. Key technical signals observed:
- **Query counts**: Independent of caching layer, the custom table cuts dispatch queries roughly in half compared to `post-meta` strategies.
- **Cache behavior**: With persistent object cache enabled, transient-based awareness drops per-dispatch DB hits to a single query; without it, `custom-table-with-transients` remains tightly coupled with `custom-table` performance (~0.05–0.17 ms variance), while `post-meta-transients` exhibits unbounded latency spikes.
- **Schema directive**: The analysis locks `custom-table-with-transients` as the default for next iterations; it explicitly rejects `post-meta-transients` even as a fallback due to degraded performance on shared/no-cache hosts.

## Contribution

Authored by @anneszazu aggregating anonymized test data submitted between April 29 and May 4 from eight hosting providers (including Kinsta, BlueHost, WordPress.com, Ionos, and XServer). Drafting involved @griffbrad, with technical review and testing by @dd32, @desrosj, @jmdodd, @peterwilsoncc, @jorbin, and @4thhubbard. The publication concludes the testing window was sufficient to lock `custom-table-with-transients` as the default architecture; `post-meta-transients` was explicitly rejected during analysis due to performance degradation on shared/no-cache hosts.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
