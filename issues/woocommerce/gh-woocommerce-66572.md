# #66572: docs: widen AGENTS.md backward-compatibility guardrail beyond PHP signatures

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @rtio
- **Merged:** [`868d5d7`](https://github.com/woocommerce/woocommerce/commit/868d5d76c2848f5155d53ce190aafc120af0137d)
- **Discussion:** [#66572](https://github.com/woocommerce/woocommerce/pull/66572) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

WooCommerce expanded its `AGENTS.md` backward-compatibility guardrails to cover the full WordPress platform surface, not just PHP function and class signatures. The update explicitly treats hook and filter contracts, global state assumptions, multisite topology, and install layout as binding compatibility boundaries that require impact statements before modification. This ensures developers and automated coding agents systematically evaluate external-facing changes to prevent regressions in third-party integrations.

## Impact

- **Plugin & theme developers**: No immediate code changes required. The updated guidelines now explicitly warn against unsafe assumptions about global state (`$post`, `WC()`), multisite options (`get_option` vs `get_site_option`), and install layouts.
- **AI/LLM coding agents**: The expanded `AGENTS.md` checklist will be consumed by automated guardrails to flag high-risk changes to hooks, filters, and environment assumptions.
- **Hosting & platform teams**: No action required; this is a documentation and process update with no runtime impact.

## Technical details

The change modifies only `AGENTS.md` in the WooCommerce repository. It adds a new subsection titled "The compatibility surface is wider than PHP signatures" that defines four binding contract layers: hooks/filters (requiring `do_action_deprecated()` or `apply_filters_deprecated()` for retirement), global state (mandating `function_exists`, `class_exists`, `isset`, `did_action`, and explicit `WC()` initialization checks), multisite behavior (distinguishing `get_option` from `get_site_option` and per-site tables), and install layout (enforcing `plugins_url()`, `plugin_dir_path()`, `wp_upload_dir()`, and the `home_url()` vs `site_url()` distinction). It also appends a five-step pre-change checklist for identifying and documenting exposure of these surfaces.

## Contribution

Opened by @rtio and merged in commit `868d5d7`. The PR was a documentation-only update to the `AGENTS.md` file, expanding the existing backward-compatibility section to cover WordPress platform contracts beyond PHP signatures. Automated review bots passed without requiring code changes. The change closes internal ticket ARC-1839 and was successfully cherry-picked to `release/11.0`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
