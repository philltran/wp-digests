# Merge Proposal: Expanding WordPress Core Abilities

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Jorge Costa
- **Published:** 2026-07-02
- **Tags:** `General`
- **Link:** [https://make.wordpress.org/core/2026/07/02/merge-proposal-expanding-wordpress-core-abilities/](https://make.wordpress.org/core/2026/07/02/merge-proposal-expanding-wordpress-core-abilities/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 expands the Abilities API (initially shipped in 6.9) by introducing three read-only abilities: `core/read-settings`, `core/read-content`, and `core/read-users`. This establishes a canonical baseline for AI clients, agentic workflows, and future WebMCP integrations to safely query site configuration, post content, and user data without reinventing permission checks or schemas.

## Impact

**Plugin & theme developers:** Must account for the new `show_in_abilities` opt-in mechanism when exposing custom settings or post types to AI agents. Existing core entities are opted into a conservative default set.

**AI client/plugin authors:** Gains standardized tools for querying core entities with built-in permission callbacks and read-only guarantees, reducing fragmentation across mcp-adapters and similar integrations.

**Site owners & REST consumers:** No breaking changes to existing REST or REST-like interfaces; this is strictly additive.

## Technical details

The consolidated 7.1 scope defines three read-only ability handlers built on a shared foundation. `core/read-settings` returns a flat `name => value` object, supports group or specific name filtering, and enforces the `manage_options` capability. `core/read-content` supports single-item lookups and collections for post types exposed to abilities, returning a lean default response while deferring advanced querying (`meta_query`, `tax_query`). `core/read-users` allows lookup by ID, email, login, or nicename, plus collection filtering by roles or published-post status, ensuring inaccessible fields are omitted per user. All three rely on a new `show_in_abilities` opt-in flag (analogous to `show_in_rest`) and execute explicit permission callbacks before returning data.

## Contribution

Authored by Jorge Costa and published as a July 2026 merge proposal, this scope consolidates experimental pull requests from `wordpress-develop` (e.g., #12141, #12195, #10775) and the AI plugin repository. The design process moved from early exploratory queries to a focused read-only baseline, deliberately excluding advanced querying and prompt-injection protection to prioritize a stable, agent-ready surface for 7.1. Future management capabilities (`core/manage-*`) and broader entity support (comments, media, etc.) are explicitly scoped for 7.2+.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
