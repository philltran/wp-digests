# #81265: Global styles: render element styles set only inside a breakpoint

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `Global Styles`, `Backported to WP Core`
- **Merged:** [`007f062`](https://github.com/WordPress/gutenberg/commit/007f062fe5139e8cea18681a40e734b50d898454)
- **Discussion:** [#81265](https://github.com/WordPress/gutenberg/pull/81265) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a Global Styles bug where element styles (e.g. `link`) defined only inside a breakpoint key such as `@mobile` or `@tablet` on a block produced no CSS. `WP_Theme_JSON_Gutenberg::get_block_nodes()` now collects element names from both `styles.blocks.<block>.elements` and each breakpoint's `elements` before looping, so breakpoint-only element styles render. The docs are also updated with a breakpoint-only example, and the sample CSS is corrected to the selector WordPress actually emits.

## Impact

- **Theme developers / site owners:** A `theme.json` that sets `styles.blocks.<block>.@mobile.elements.<el>` (or `@tablet`) without also styling that element outside the breakpoint will now output CSS where previously it silently output nothing. Output for themes that already style the element both inside and outside a breakpoint is unchanged.
- **Visual changes:** Themes that already contained breakpoint-only element styles (previously ignored) will start rendering them, which could change the appearance of existing sites.
- **Plugin developers:** No API changes; anything consuming `get_block_nodes()` output (including the `include_node_paths_only` mode) sees element nodes only for elements that actually exist outside a breakpoint.
- **Editor parity:** A follow-up PR (#81307) was opened for equivalent editor-side changes, so editor rendering may lag this PHP fix.
- **Not fixed:** Per the PR, `styles.blocks.<block>.elements.<el>["@mobile"]`, `styles.elements.<el>["@mobile"]`, and `styles.blocks.<block>.variations.<v>.elements` / `.blocks` still accept styles that never render.

## Technical details

The change is in `lib/class-wp-theme-json-gutenberg.php`, in `get_block_nodes()`. Previously, the loop iterated only over `$theme_json['styles']['blocks'][$name]['elements']`, and breakpoint element nodes were emitted from inside that loop. An element that existed only under a breakpoint was never iterated, so no node was created.

Now the code builds `$element_names` from:

- `array_keys( $block_node['elements'] ?? array() )`
- `array_keys( $block_node[ $breakpoint ]['elements'] ?? array() )` for each key of `$responsive_media_queries`

The merged list is passed through `array_unique()` and looped over. Within the loop:

- With `$include_node_paths_only`, a path node is added only if `$block_node['elements'][$element]` is set, then `continue`.
- Otherwise, elements with no entry in `$selectors[$name]['elements']` are skipped.
- The default (non-breakpoint) node is pushed only when `isset( $block_node['elements'][ $element ] )`. The existing responsive-element-node emission that follows is unchanged in the diff shown.

Example that now renders:

```json
"core/group": { "@mobile": { "elements": { "link": { "color": { "text": "red" } } } } }
```

```css
@media (width <= 480px){:root :where(.wp-block-group a:where(:not(.wp-element-button))){color: red;}}
```

Three new PHPUnit tests in `phpunit/class-wp-theme-json-test.php` cover: a breakpoint-only element, a breakpoint-only element pseudo (`:hover`), and an element styled in separate breakpoints (`@mobile` and `@tablet`). `docs/how-to-guides/themes/global-settings-and-styles.md` gains a breakpoint-only example, and the existing CSS sample now uses `.wp-block-group a:where(:not(.wp-element-button))` instead of `.wp-block-group a`. A `backport-changelog/7.1/12918.md` entry was added.

## Contribution

@ramonjd opened this after review on #81253 exposed the gap. @talldan had already prepared a core backport (wordpress-develop #12918) covering this PR and #81291, and @ramonjd, who had an auto-generated backport PR (#12926) of his own, agreed to go with @talldan's and close his. @tellthemachines confirmed the 7.1 release branch was not yet created at that point. After merge, the bot cherry-picked it to `wp/7.1`, and @tellthemachines noted equivalent editor changes were needed and opened #81307.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
