# Dev Chat summary: May 27, 2026

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Jb Audras
- **Published:** 2026-05-27
- **Tags:** `Devchat`, `General`, `Summaries`, `7.0`, `7.0.1`, `7.1`, `core`, `dev chat`
- **Link:** [https://make.wordpress.org/core/2026/05/27/summary-dev-chat-may-27-2026/](https://make.wordpress.org/core/2026/05/27/summary-dev-chat-may-27-2026/)

## Summary

The team triaged block editor publishing screen layout issues tracked in #65286 and settled on a dual-deployment strategy for the fix. To cover sites using custom post types without Classic Editor and those relying on auto-updates, the resolution commits to shipping the UI cleanup to both the Classic Editor plugin and the Hotfix plugin.

## Impact

- **Theme & Plugin Developers**: Components that customize or extend the block editor publishing screen should audit their UI against the upcoming plugin patches to prevent layout conflicts or unintended overrides.
- **Site Owners / Platform Teams**: No immediate core configuration or code changes are required. Monitor release notes for both the Classic Editor and Hotfix plugins, as each will receive the fix independently.
- **Action Required**: None at this stage. Verify CPT editor experiences once the plugin patches roll out, particularly if editors disable the block editor via hooks or registration args.

## Technical details

This entry documents a triage outcome rather than a merged core code change. Ticket #65286 addresses messy rendering on the block editor publishing screen, specifically affecting custom post types (CPTs) where Classic Editor is not active. The implemented resolution coordinates separate patching pipelines:
- `Classic Editor` plugin: Targets the existing user base and sites with auto-updates enabled.
- `Hotfix` plugin: Captures edge cases where the block editor is disabled via hooks, registration filters, or CPT configurations.
Core code paths remain untouched; the fix leverages plugin-level modifications to the block editor UI and publishing screens. Coordination between maintainers will ensure overlapping patches do not conflict during rollout.

## Contribution

@jorbin flagged the publishing screen regression during a 7.0.1 development cycle discussion. @desrosj and @audrasjb evaluated Classic Editor vs. Hotfix plugin deployment, noting install base differences and hook-based block editor disables. @davidbaumwald confirmed the issue impacts CPTs without Classic Editor. The final decision to deploy to both plugins was adopted by @jorbin and @desrosj, who agreed to coordinate the combined patch strategy.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
