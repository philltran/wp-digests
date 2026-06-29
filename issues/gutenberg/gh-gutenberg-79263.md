# #79263: Knowledge: Dissolve the Guidelines singleton into per-scope rows

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aagam-shah
- **Labels:** `[Type] Breaking Change`, `No Core Sync Required`, `[Feature] Guidelines`
- **Merged:** [`9de6a7f`](https://github.com/WordPress/gutenberg/commit/9de6a7f5fe789d0edcc7095fe12b23752e07939b)
- **Discussion:** [#79263](https://github.com/WordPress/gutenberg/pull/79263) · 8 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

The Gutenberg Knowledge layer for `Settings → Guidelines` is refactored from a singleton post holding all categories as `_guideline_*` meta into one published `wp_knowledge` row per scope (prefixed `guideline-`). The custom `/wp/v2/content-guidelines` REST controller and hand-rolled client store are removed. The admin interface remains visually identical, but all reads/writes now route through the standard `/wp/v2/knowledge` endpoint via `useEntityRecords`.

## Impact

['- **Plugin & theme developers / headless consumers**: Direct calls to the legacy `/wp/v2/content-guidelines` singleton route and expectations around `_guideline_*` post meta will fail. Any custom integrations must migrate to the standard `/wp/v2/knowledge` collection using `guideline-` prefixed slugs.', '- **Gutenberg plugin / experimental maintainers**: The guidelines data model now relies on a new `wp_guideline_scopes` filter for registry drives and a read-only `/wp/v2/knowledge/guideline-scopes` controller. Reclaim logic is embedded in the save flow rather than relying on REST pre-insert guards.', '- **Editors & site owners**: No visible change. The Settings → Guidelines UI behaves identically, though underlying revision history is hidden for now and import/export JSON shapes remain compatible.']

## Technical details

The diff removes `lib/experimental/knowledge/class-gutenberg-content-guidelines-rest-controller.php` (800+ lines) and its associated revisions controller. The new architecture stores each scope in a single `guideline`-typed `wp_knowledge` row, enforced by a `guideline-` slug prefix (e.g., `guideline-copy`, `guideline-block-core_paragraph`) and the canonical block name in `post_title`. A `wp_guideline_scopes` filter serves as the registry source of truth, exposed via a read-only `/wp/v2/knowledge/guideline-scopes` controller. The client shifts to `@wordpress/core-data` using `useEntityRecords` plus a runtime `guidelineScope` entity. Slug uniqueness is handled by allowing exact slugs only for the published row; duplicates get suffixed (`-2`) or are automatically reclaimed on save (restore from trash, publish, overwrite content/author). No data migration runs; existing singleton meta is dropped per experimental status.

## Contribution

Opened by @aagam-shah as a follow-up to #79149. Reviewer @gziolo pushed for simplifying slug conflict handling: replace 409 errors with automatic row reclamation (restore from trash, publish, overwrite) and enforce exact-slug uniqueness only for published rows via `wp_unique_post_slug`. The final diff landed after rebasing to trunk, retaining the scoped-row architecture while stripping legacy singleton controllers and client store files. Co-authored by @aagam-shah and @gziolo.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
