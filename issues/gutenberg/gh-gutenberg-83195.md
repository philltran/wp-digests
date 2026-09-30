# #83195: Post Title: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Title`
- **Merged:** [`93bf3d7`](https://github.com/WordPress/gutenberg/commit/93bf3d78caa09c10c0f2ca3d53e765c6dfa4d69e)
- **Discussion:** [#83195](https://github.com/WordPress/gutenberg/pull/83195) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `core/post-title` block now declares `shadow` in its `block.json` supports, enabling box-shadow styling through Global Styles and per-block overrides. This closes a gap in the design-tools consistency effort (tracked in #43241) where the Title block already supported color, border, spacing, and typography but not shadow. Because the block is server-rendered via `get_block_wrapper_attributes()`, the support serializes onto the wrapper element with no PHP change required.

## Impact

- **Theme & site builders:** The Title block now appears under Blocks → Title → Shadow in the Global Styles panel. Presets (Natural, Deep, Sharp, Crisp, etc.) and custom values can be applied globally or per-block-instance, including responsive (mobile/tablet) variants.
- **Theme developers:** Global Styles shadow rules for `core/post-title` also apply to the Title block that a theme template renders natively (e.g., the title above the post in a single-post template). Block-level overrides apply only to the specific block instance.
- **Plugin developers:** No API change. If you register a custom block that wraps or extends `core/post-title`, no code change is needed. If you build tooling that enumerates a block's supports, `shadow` will now appear for `core/post-title`.
- **No action required** for existing sites; the change is purely additive and renders no shadow unless one is explicitly set.

## Technical details

The functional change is a single line in `packages/block-library/src/post-title/block.json`, adding `"shadow": true` to the `supports` object:

```json
"supports": {
  "color": { "background": true, "gradients": true, "link": true },
  "shadow": true,
  "spacing": { "margin": true, "padding": true },
  "typography": { "fontSize": true, "lineHeight": true, "textAlign": true }
}
```

No PHP, JS, or CSS files are modified. The block's server-side render path calls `get_block_wrapper_attributes()`, which reads the `shadow` support and emits the corresponding `box-shadow` CSS custom property on the wrapper `<h1>`/`<h2>` element. When no post or title is present the render returns an empty string, so no empty element is produced for the shadow to paint on.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — `shadow` added to the supports list for `core/post-title`.
- `packages/block-library/src/post-title/README.md` — a new bullet for `shadow: true` under the supports section.

## Contribution

Opened by @aaronrobertshaw and co-authored with @ramonjd. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no design debate or rejected alternatives visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
