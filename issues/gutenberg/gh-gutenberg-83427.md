# #83427: Post Content: Add text align support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Content`
- **Merged:** [`c93d327`](https://github.com/WordPress/gutenberg/commit/c93d3278358d0d46ad2122be47686cfb2e9e3182)
- **Discussion:** [#83427](https://github.com/WordPress/gutenberg/pull/83427) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Content block (`core/post-content`) now supports `typography.textAlign`, closing a gap where it was the only post-data block (alongside Post Title, Post Excerpt, and Comment Content) without text alignment. The change is a single `"textAlign": true` entry in the block's `block.json` supports; because the block is server-rendered and its callback already calls `get_block_wrapper_attributes()`, the `has-text-align-*` class is emitted on the wrapper with no PHP change. Alignment set on the block cascades to all inner content, while any inner block that sets its own alignment still wins.

## Impact

- **Site owners / template editors:** Text alignment for post body content can now be set in Global Styles (Blocks → Content → Typography → Text alignment) or via the block toolbar, including per-breakpoint (mobile/desktop) values. No action required on existing sites; the default remains left-aligned.
- **Theme developers:** The post content wrapper will now carry a `has-text-align-*` class when alignment is set. If a theme targets the post content wrapper with its own `text-align` rule, it will need to account for the new class or use `!important` / higher specificity to override it.
- **Plugin & block developers:** No API change. The block's `block.json` gains one supports key; no new hooks, attributes, or render-callback arguments are introduced.
- **No breaking changes, deprecations, or migrations.**

## Technical details

The diff touches three files:

1. **`packages/block-library/src/post-content/block.json`** — adds `"textAlign": true` inside the existing `"typography"` supports object (alongside `fontSize`, `lineHeight`, and the `__experimental*` font keys).
2. **`packages/block-library/src/post-content/README.md`** — regenerated to list `textAlign: true` under the typography supports section.
3. **`docs/reference-guides/core-blocks/README.md`** — regenerated; the Post Content supports line changes from `typography (fontSize, lineHeight)` to `typography (fontSize, lineHeight, textAlign)`.

No PHP, JS, or CSS files are modified. The block's server-side render callback already wraps output in `get_block_wrapper_attributes()`, which reads the `textAlign` support and emits the corresponding `has-text-align-left|center|right|justify` class on the wrapper element. Because the block returns an empty string when there is no content, no empty wrapper is produced. Inner blocks that set their own `textAlign` attribute continue to override the wrapper class, matching the behaviour of Post Title, Post Excerpt, and Comment Content.

Unlike the Verse block's textAlign adoption (#74724), which required deprecations to migrate an existing `textAlign` attribute, Post Content has no pre-existing alignment attribute, so no deprecation or migration path is needed.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @shail-mehta. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion record contains only two bot comments (performance metrics and co-author attribution) and no human review comments or design debate. It follows the same minimal `block.json`-only pattern as the earlier Comment Date textAlign adoption (#74599) and is linked to the broader tracking issue #43241.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
