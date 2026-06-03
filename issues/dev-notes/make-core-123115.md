# Real-time collaboration will not ship in WordPress 7.0

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** annezazu
- **Published:** 2026-05-08
- **Tags:** `Updates`, `7.0`, `feature-real-time-collaboration`
- **Link:** [https://make.wordpress.org/core/2026/05/08/rtc-removed-from-7-0/](https://make.wordpress.org/core/2026/05/08/rtc-removed-from-7-0/)

## Summary

Real-time collaboration (RTC) has been removed from the WordPress 7.0 release cycle. Core leadership deferred the feature after fuzz testing exposed recurring bugs, race conditions, and concerns around server load and memory efficiency. The codebase will be stripped of RTC modules to prioritize a stable 7.0 launch, with broader testing planned for a future iteration. The official release schedule remains unchanged.

## Impact

- **Plugin & theme developers**: No migration steps required; the RTC infrastructure will not be present in the 7.0 distribution. Monitor Trac ticket #65205 for post-release cleanup and architectural updates.
- **Hosting & platform teams**: No server configuration changes are needed, as the capability remains unshipped. The release schedule is unchanged. Await documentation on experimental plugin distributions once future testing plans are published.
- **Editors & site owners**: Collaborative editing workflows will not be available upon upgrading to 7.0. Expect further communication regarding expanded testing programs for upcoming releases.

## Technical details

This entry documents a high-level feature deferral rather than a code merge; no specific diff or API symbols are included in the announcement. The removal is orchestrated through Trac ticket #65205 and the `#feature-realtime-collaboration` Slack channel. Developers should review the linked ticket for the final patch that excises RTC-related files, hooks, data stores, and frontend sync modules from the 7.0 branch. No new filters, REST schema changes, or block.json fields were introduced; this is a complete deferral of the experimental RTC architecture.

## Contribution

Core leadership announced the deferral on May 8, 2026, prioritizing release stability over feature completeness. Implementation of the removal is coordinated by @matt and tracked in ticket #65205 alongside the `#feature-realtime-collaboration` channel. A formal roadmap for continued testing and iterative development will be published once the immediate 7.0 cleanup phase concludes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
