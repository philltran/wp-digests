# #82403: Global Styles: Retain responsive styles in block style variation partials

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @guzel
- **Labels:** `[Type] Bug`, `First-time Contributor`, `Global Styles`, `Backport to WP Minor Release`, `[Feature] Block Style Variations`, `[Feature] Style States`
- **Merged:** [`d0ceb5c`](https://github.com/WordPress/gutenberg/commit/d0ceb5c86f534dfa7f69ed7c852865c1d7a9da6a)
- **Discussion:** [#82403](https://github.com/WordPress/gutenberg/pull/82403) · 12 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Responsive `@tablet`/`@mobile` styles, and pseudo-selector styles such as `:hover`, are no longer silently stripped from block style variations declared as standalone JSON partials in a theme's `styles/` directory. `WP_Theme_JSON_Gutenberg::sanitize()` now adds these states to the top-level `styles` schema when the input has a `blockTypes` property. Before the fix, the base rule was generated but no media query CSS was emitted, and no error surfaced. Variations declared inline in `theme.json` were never affected.

## Impact

- **Theme developers:** Variation partials that declare `@tablet`, `@mobile`, or pseudo-selectors at the root of `styles` now produce the expected CSS. If you previously duplicated responsive rules in custom CSS or moved variations inline to work around the bug, you can revert that.
- **Pseudo-selectors:** These are scoped to the partial's `blockTypes`. A `core/button` partial gets button's selectors, and a partial for a block with none gets none.
- **Regular `theme.json`:** Unchanged. Root-level `@tablet` and `:hover` are still removed.
- **Custom states:** Custom states such as `-current` on navigation link are not supported for variations, inline or partial, and remain unsupported.
- **Gutenberg plugin users:** The plugin's `WP_Theme_JSON_Gutenberg` overrides core's class, so this port is needed for them. The core fix is tracked as Trac #65992 and is milestoned 7.1.1.
- **Action:** None required beyond upgrading. Retest any partials that previously lost their responsive styles.

## Technical details

**Root cause:** Partials declare styles at the root of `styles`, and `WP_Theme_JSON_Resolver_Gutenberg::get_style_variations()` sanitizes them through `WP_Theme_JSON_Gutenberg`, so they are validated against the top-level styles schema. The breakpoint states had only been added to the block, element, and variation branches, so `remove_keys_not_in_schema()` dropped root-level `@tablet` and `@mobile` as unknown keys. The variation branch could not cover this case. It validates a different tree location, and its guard requires the variation to be in `$valid_variations`. Partials are not registered until `gutenberg_register_block_style_variations_from_theme_json_partials()` runs, after `get_style_variations()` returns.

**Fix in `lib/class-wp-theme-json-gutenberg.php`:**
- If `$input['blockTypes']` is a non-empty array, the pseudo-selectors from `VALID_BLOCK_PSEUDO_SELECTORS` for each listed block type are merged and de-duplicated.
- For each breakpoint state, `$schema['styles'][ $breakpoint_state ]` is set to `$styles_non_top_level`, with `elements` added and each pseudo-selector nested inside.
- Each pseudo-selector is also added directly at `$schema['styles']`.
- The existing variation branch no longer sets `$variation_schema[ $breakpoint_state ]['blocks']`. As a result, `blocks` nested inside a breakpoint state is dropped for both partials and inline variations, since that shape generates no CSS. The supported shape nests the breakpoint inside `blocks`, and two new tests lock this in.
- A `@since 7.1.1` note was added to `sanitize()`.

**Schema and docs:**
- `schemas/json/theme.json` gains a `stylesVariationPartialRootPropertiesComplete` definition (root style properties plus responsive and pseudo-selector states). It was cherry-picked from a schema change proposed in #82652.
- The `styles` description and `theme-json-living.md` now document the partial behavior.

**Tests:** New fixtures `block-style-variation-responsive.json` and `block-style-variation-pseudo.json` cover sanitization and generated CSS (media queries `@media (480px < width <= 782px)` and `@media (width <= 480px)`). Other tests assert regular root styles still reject the states. The `data_get_style_variations()` expectation in the resolver test was updated for the new fixtures. A `backport-changelog/7.1/13325.md` entry links the core PR.

```json
{
  "blockTypes": [ "core/heading" ],
  "styles": {
    "typography": { "fontSize": "40px" },
    "@tablet": { "typography": { "fontSize": "28px" } },
    "@mobile": { "typography": { "fontSize": "16px" } }
  }
}
```

## Contribution

This was a first-time contribution by @guzel that ports the core fix (wordpress-develop #13325). The PR notes AI assistance (Claude Code), with the tests written first and confirmed failing. @talldan's review widened the scope from responsive states to pseudo-selectors (`:hover`, `:focus`) and raised the schema question. @guzel then limited the new states to inputs with `blockTypes` and added a test that regular `theme.json` roots still strip them. Custom states like `-current` were deliberately left out, because `get_block_nodes()` emits custom-state nodes only for blocks, not variations, so that would need generator work. Because the PR came from a fork, @talldan could not stack a schema PR (#82652) on it. @guzel cherry-picked its schema commit instead.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
