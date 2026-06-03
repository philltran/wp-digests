# #78490: Global styles revisions: ensure stylebook shows revision previews

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Global Styles`
- **Merged:** [`2c1bffb`](https://github.com/WordPress/gutenberg/commit/2c1bffb00c1507dda0c7bf6a7251ade4aee9b747)
- **Discussion:** [#78490](https://github.com/WordPress/gutenberg/pull/78490) · 4 comments · 0 reactions

## Summary

Fixes a UI state mismatch in the Site Editor where the Style Book canvas fails to update when switching between global styles revisions. Previously, selecting a different revision while the Style Book was open would continue rendering the base theme's current styles instead of the revision-specific changes. The fix synchronizes the preview component with the active revision ID, ensuring the canvas accurately reflects each selected style set during editing.

## Impact

- **Site Editors / Theme Developers**: No action required. This is a targeted UI fix that resolves an editor state synchronization bug without altering public APIs, data stores, or configuration schemas.
- **Plugin/Theme Developers**: Zero impact. Global styles revisions remain managed through standard Editor data patterns; no migration, config changes, or code updates are needed.
- **Workflow Impact**: Existing revision apply/dismiss flows operate as before. The Style Book now correctly inherits and displays the selected revision's merged styles during interactive preview, eliminating the stale-preview bug.

## Technical details

The change corrects a path-aware rendering gap in the Style Book component when operating inside the `/revisions` route. When the Editor URL contains a revisions path, the canvas previously defaulted to reading the active theme's base global styles configuration. The fix introduces a route detection step that identifies the `/revisions` path within the Style Book canvas scope, extracts the target revision ID from the URL string, and resolves the corresponding revision object from the revisions data list. This revision object is then merged with the base global styles before being passed as the preview context into the `StyleBook` component. The logic mirrors the existing sidebar revision toggle behavior, ensuring the preview state correctly composes the selected revision on top of the base theme for rendering.

## Contribution

PR #78490 was authored by @ramonjd with co-authorship from @andrewserong. The discussion focused on validation, with reviewers confirming the fix resolves the preview stale state without interfering with normal revision switching or apply workflows. No alternative implementation strategies were debated in the provided thread; the PR moved directly to merge after successful testing against multiple revision swaps and confirmed that disabling the Style Book returns normal behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
