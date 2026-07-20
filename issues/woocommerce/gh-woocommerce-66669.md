# #66669: [Backport to release/11.0] docs: widen AGENTS.md backward-compatibility guardrail beyond PHP signatures

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @woocommercebot
- **Merged:** [`c6ffd19`](https://github.com/woocommerce/woocommerce/commit/c6ffd1990905a3b990be7b2808e022c6b5220b54)
- **Discussion:** [#66669](https://github.com/woocommerce/woocommerce/pull/66669) · 1 comments · 0 reactions
- **Usefulness:** 2/5

## Summary

WooCommerce updated its `AGENTS.md` backward-compatibility guidelines to explicitly cover non-PHP-surface contracts. The change expands the required backward-compatibility impact statement to include hook and filter signature/timing changes, assumptions about global state and execution scope, multisite behavior, and WordPress install layout. This ensures developers and integrated AI agents evaluate the full compatibility surface before modifying externally exposed behavior.

## Impact

- Plugin & theme developers: No immediate code changes required, but the updated guidelines clarify that hook removal/reordering, unsafe global state reads, and hard-coded paths/URLs are now explicitly flagged as high-risk compatibility issues.
- AI/LLM agent users: The `AGENTS.md` file now includes a 5-step pre-change checklist and expanded rules for backward compatibility, which will shape how integrated agents review or generate WooCommerce code.
- Site owners, hosting teams, and REST consumers: No action required.

## Technical details

The change modifies `AGENTS.md` in the WooCommerce repository. It adds a new subsection under the Backward Compatibility section titled "The compatibility surface is wider than PHP signatures" and a "Before changing any public or externally exposed surface (agent checklist)". The diff introduces four explicit rules:
- Hooks and filters are treated as public contracts; changes require `do_action_deprecated()` / `apply_filters_deprecated()` for retirement.
- Global state and execution scope must be guarded using `function_exists`, `class_exists`, `isset`, `did_action`, and explicit `WC()` initialization checks.
- Multisite behavior must account for `get_option` vs `get_site_option`, per-site tables, and upload paths.
- Install layout must use `plugins_url()`, `plugin_dir_path()`, `wp_upload_dir()`, and distinguish `home_url()` from `site_url()` instead of root-path concatenation.
The checklist mandates identifying the contract, assuming unseen consumers, preferring additive changes, stating impact in PR descriptions, and halting if impact cannot be established.

## Contribution

This is a backport of PR #66572 to the `release/11.0` branch, merged as commit `c6ffd19`. The change was authored by `@woocommercebot` (automated cherry-pick) and closed ARC-1839. The discussion was minimal, consisting only of a bot reminder to verify testing instructions, as the PR is strictly documentation. The guidelines were expanded to cover hooks, global state, multisite, and install layout based on prior compatibility incidents, such as the WooCommerce 10.9.0 `FeedInterface` revert.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
