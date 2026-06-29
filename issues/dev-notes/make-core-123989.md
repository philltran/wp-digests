# Merge Proposal: Guidelines built on Knowledge

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Greg Ziółkowski
- **Published:** 2026-06-22
- **Tags:** `Core`, `Proposals`, `7.1`, `guidelines`, `Knowledge`, `merge-proposals`
- **Link:** [https://make.wordpress.org/core/2026/06/22/merge-proposal-guidelines-built-on-knowledge/](https://make.wordpress.org/core/2026/06/22/merge-proposal-guidelines-built-on-knowledge/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 introduces `wp_knowledge`, a new internal custom post type paired with the `wp_knowledge_type` taxonomy, as a canonical storage primitive for site standards and authoring notes. The merged implementation provides native revisions, private-by-default records, and a capability-gated REST surface to replace fragmented plugin-specific knowledge stores. Guidelines ships as the first consumer feature, offering a Settings-driven UI for capturing voice, tone, and per-block rules, but core deliberately omits AI processing, decay mechanisms, or retrieval algorithms.

## Impact

- **Plugin & theme developers**: Can extend built-in record types via `add_filter( 'wp_knowledge_types', 'callback' )`. Existing knowledge/notes plugins remain fully functional; no forced migration is required unless unified revisions and interoperability are desired.
- **Hosting & platform teams**: The post type is internal (`public => false`) with no front-end exposure. Private records default on creation, and collection reads require authentication, keeping the data surface narrow but predictable for audit/logging pipelines.
- **Headless & REST consumers**: `GET /wp/v2/knowledge` returns an authenticated collection; individual reads are gated by `read_post` capability checks. Scope metadata is exposed read-only at `/wp/v2/knowledge/guideline-scopes`.
- **No action required** for standard publishing workflows or sites that do not maintain editorial standards programmatically.

## Technical details

- Registers the `wp_knowledge` CPT with native revisions enabled and autosave explicitly disabled. Records default to private status and are strictly non-publicly queryable.
- Introduces the `wp_knowledge_type` taxonomy. Taxonomy terms for built-in slugs (`guideline`, `memory`, `note`) are created lazily on first save rather than at registration time.
- Access model uses a `*_knowledge_item` capability namespace (e.g., `create_knowledge_item`, `edit_others_knowledge_item`). Matrix restricts Subscribers/Contributors to their own private records; Contributors can manage their own set; Authors/Editors may read others'; Admins manage all. Revisions are retained across the lifecycle.
- Exposes two primary REST endpoints: `GET /wp/v2/knowledge` (authenticated collection) and `GET /wp/v2/knowledge/guideline-scopes` (read-only scope registry).
- Provides `add_filter( 'wp_knowledge_types', 'callback' )` accepting an array and returning an extended array with metadata like `'title'`. Plugins can use this to register custom slugs (e.g., `glossary`) for cross-tool consistency.
- The shipped "Guidelines" UI consumes the scope registry to render Settings → Guidelines sections per reserved slug. No AI models, decay schedules, or consolidation logic ship in core.

## Contribution

Proposed by Greg Ziółkowski and tracked in Trac #65476 / `wordpress-develop#12201`. The patch consolidates earlier Gutenberg experiment work (landed in Gutenberg 22.7) into a stabilized core implementation. Review feedback from @aagam94, @artpi, @jason_the_adams, and @jorgefilipecosta shaped the namespace freeze, capability boundaries, and lazy taxonomy term creation. The merge targets WP 7.1, with all names and access models scheduled for stabilization at the July 15 beta cutoff.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
