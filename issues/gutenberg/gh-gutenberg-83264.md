# #83264: Tag Cloud: Add background and link colour support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Tag Cloud`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`3db7bee`](https://github.com/WordPress/gutenberg/commit/3db7bee439aa6bcd26621103abd1d3ef74a01114)
- **Discussion:** [#83264](https://github.com/WordPress/gutenberg/pull/83264) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tag Cloud block (`core/tag-cloud`) gains background and link colour controls, declared as `color` supports in its `block.json`. This is a `block.json`-only change: no PHP or JavaScript rendering code is added. It supersedes the earlier attempt in #63592, which was stalled by doubled CSS output; that problem is now resolved by the `HtmlRenderer` merge introduced in #74228. The change is part of the broader design-tools consistency effort (related #43241) to bring the Tag Cloud block in line with other core blocks that already expose colour controls.

## Impact

- **Block-theme site owners / editors:** Tag Cloud now appears under Blocks → Tag Cloud → Colors in Global Styles with Background and Link colour pickers. Block-level overrides in the inspector also work, including per-breakpoint (Mobile) values.
- **Plugin & theme developers:** No code changes required. The block's `block.json` now declares the supports; any theme or plugin that reads `block.json` supports will see the new `color` entry. The Outline style's per-tag border follows the link colour automatically.
- **No breaking changes, deprecations, or migrations.** Existing Tag Cloud blocks render identically until a colour is explicitly set.
- **No action required** for existing sites or plugins.

## Technical details

The sole functional change is in `packages/block-library/src/tag-cloud/block.json`, which adds a `color` supports object:

```json
"color": {
  "background": true,
  "link": true,
  "text": false,
  "__experimentalDefaultControls": {
    "background": true,
    "link": true
  }
}
```

- `text` is explicitly `false` because the tag items are `<a>` elements; the link colour is what recolours them, and a text-colour control would appear inert.
- `__experimentalDefaultControls` ensures both Background and Link pickers are visible in the block inspector by default.
- Gradients are not added.
- **Front end:** The block is server-rendered through `get_block_wrapper_attributes()`, so the `has-background` and `has-link-color` classes (and their corresponding CSS custom properties) are emitted without any PHP change.
- **Editor:** Since #74228, the editor merges block props onto the server-rendered markup via `HtmlRenderer`, producing a single `wp-block-tag-cloud` wrapper with `has-background` rather than the nested/doubled structure that blocked #63592.
- The two README files (`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/tag-cloud/README.md`) are regenerated to list the new supports.

## Contribution

Opened by @aaronrobertshaw with @talldan as co-author. The PR supersedes #63592 and is related to the broader design-tools consistency tracker #43241. The record carries only two comments (both from the GitHub Actions bot) and no substantive design debate; the PR was implemented by a Claude Code agent from a predefined task and merged without visible review discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
