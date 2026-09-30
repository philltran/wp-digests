# #81908: Block Supports: Bail early in state styles when a block has no style attribute

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mukeshpanchal27
- **Labels:** `[Type] Performance`, `Backported to WP Core`, `[Feature] Style States`
- **Merged:** [`9559b67`](https://github.com/WordPress/gutenberg/commit/9559b67353a83d048daf9427057b961398805e0d)
- **Discussion:** [#81908](https://github.com/WordPress/gutenberg/pull/81908) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `render_block` callback for the Style States block support (`gutenberg_render_block_states_support()` in the Gutenberg plugin, `wp_render_block_states_support()` in core) now returns early when a block has no `style` attribute, or when it is empty or not an array. Previously the function ran a registry lookup and a viewport settings/media-query computation for every block on every request, then discarded the results. The change came out of a 7.0.4 → 7.1 benchmark that showed a server-side PHP regression on both Block and Classic themes.

## Impact

- **Site owners / all front-end requests:** Lower PHP render time on pages with many blocks. No visible output change; blocks without a `style` attribute already fell through to an unchanged `$block_content` return.
- **Plugin & theme developers:** No API, hook, or signature changes. No action required.
- **Hosting & platform:** Small per-request server-time reduction on block-heavy pages. The PR reports about 1.3 µs saved per style-less block (roughly 14x on this code path).
- **Edge case:** A block whose `style` is non-empty but has no state or responsive keys still takes the full path. Unregistered block names with a `style` attribute now run alias resolution before the registry check, which returns early.

## Technical details

The diff touches `lib/block-supports/states.php` and adds `backport-changelog/7.2/13051.md`.

What the diff does inside `gutenberg_render_block_states_support()`:

- Reads the raw style before any lookup: `$raw_style = $block['attrs']['style'] ?? array();`
- Returns `$block_content` if `empty( $raw_style ) || ! is_array( $raw_style )`.
- Calls `gutenberg_resolve_style_state_aliases( $raw_style, $block_name )` only after that guard passes.
- Keeps the `WP_Block_Type_Registry::get_instance()->get_registered( $block_name )` lookup after the guard, still returning early for unregistered blocks.
- Keeps `WP_Theme_JSON_Gutenberg::VALID_BLOCK_PSEUDO_SELECTORS` and the rest of the function unchanged, including `gutenberg_get_global_settings( array( 'viewport' ) )` and `get_viewport_media_queries()`.

The guard is safe because every write to `$css_rules` is keyed off `$style`, and the function already returns unchanged content when `$css_rules` is empty.

The PR description sketches the guard on `$style` directly. The merged diff instead guards on the raw attribute and resolves aliases afterward, so alias resolution is skipped for style-less blocks too.

The author's measurement (20k iterations on a `core/paragraph` with no style) is 1396.6 ns/block on trunk versus 100.4 ns/block with the change.

The PR also identifies, but does not fix, related issues:

- The viewport computation could be memoized per request.
- `wp_get_global_settings()` returns the entire settings tree for a missing path, because of the `_wp_array_get()` fallback.
- `_wp_register_default_icons()` runs on every front-end `init`.

## Contribution

Opened by @mukeshpanchal27 and paired with wordpress-develop PR 13051, with a backport changelog under `backport-changelog/7.2/`. @westonruter asked for a complete PR description and an AI-tooling disclosure covering this PR and the core one; the description was then updated, and the PR was merged after the author flagged it ready. The author deliberately kept the viewport memoization, the `wp_get_global_settings()` fallback fix, and the icon registration cost out of scope as separate follow-ups to keep review small. A CodeRabbit run noted that alias resolution still happens for styled blocks of unknown type.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
