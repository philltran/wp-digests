# #81291: Style states: Fix phantom pseudo element style output

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Bug`, `Backported to WP Core`, `props-bot`, `[Feature] Style States`
- **Merged:** [`102eb74`](https://github.com/WordPress/gutenberg/commit/102eb74ff857536bd46798d646e1b83e932a0abe)
- **Discussion:** [#81291](https://github.com/WordPress/gutenberg/pull/81291) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Fixes a Style States bug in `WP_Theme_JSON_Gutenberg` where defining an element pseudo-state (e.g. `link` `:hover`) only inside a responsive state such as `@mobile` caused an extra, undefined rule to be emitted at the default level. That rule copied the element's base styles onto the pseudo selector (e.g. `a:hover{color: blue;}`). The default pseudo node is now generated only when the pseudo is actually styled outside a breakpoint. The PR also removes an unreachable `$include_node_paths_only` branch.

## Impact

- **Theme authors / site owners:** Themes using responsive pseudo styles (e.g. `@mobile` > `elements.link.:hover`) without a matching default pseudo style no longer get a stray `:hover` rule carrying the base element's styles. Output now matches what the theme.json defined. This could change the rendered appearance on desktop for affected themes, since the phantom hover rule is gone.
- **Plugin developers:** No API changes. If you assert on generated stylesheet output for Style States, expect the phantom rule to disappear.
- **Core / Gutenberg maintainers:** The change was cherry-picked to the `wp/7.1` branch and is tracked via `backport-changelog/7.1/12918.md` alongside #81265.
- No action required for most sites.

## Technical details

The change is in `WP_Theme_JSON_Gutenberg::get_block_nodes()` (`lib/class-wp-theme-json-gutenberg.php`), in the loop over `VALID_ELEMENT_PSEUDO_SELECTORS`.

**Before:** a `$has_element_pseudo` flag was true if the pseudo existed at the default level *or* under any responsive breakpoint. When true, a default pseudo node (`path` = `styles.blocks.{name}.elements.{element}`, selector = element selector + pseudo) was always added. If only a breakpoint defined the pseudo, `get_styles_for_block()` found no pseudo key at that path and fell back to the element's base styles, emitting `...a:hover{color: blue;}`.

**After:** the default node is added only if `$theme_json['styles']['blocks'][$name]['elements'][$element][$pseudo_selector]` is set. This mirrors the check already used for block-level pseudo styles. The per-breakpoint loop now runs independently of the default check, so responsive pseudo nodes (with `media_query`) are still generated. The per-breakpoint logic is otherwise unchanged.

**Dead code removed:** the `if ( $include_node_paths_only ) { $nodes[] = array( 'path' => ... ); continue; }` block was unreachable because an earlier `continue` in the loop already handles that case.

Example from the PR output:

```css
/* Before */
:root :where(.wp-block-group a:where(:not(.wp-element-button))){color: pink;}
:root :where(.wp-block-group a:where(:not(.wp-element-button)):hover){color: pink;}
@media (width <= 480px){...:hover{color: green;}}

/* After */
:root :where(.wp-block-group a:where(:not(.wp-element-button))){color: pink;}
@media (width <= 480px){...:hover{color: green;}}
```

A new PHPUnit test, `test_get_stylesheet_does_not_duplicate_base_element_styles_into_pseudo_rule`, asserts that the output is the base link rule plus only the `@media (width <= 480px)` hover rule.

## Contribution

Authored by @talldan as a follow-up stacked on the branch from #81265, which it depends on. @ramonjd reviewed it (LGTM), rebased it locally, and the merged change was cherry-picked to `wp/7.1` by the bot. The description discloses AI tooling use (OpenCode / Kimi K3).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
