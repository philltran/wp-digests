# #83178: Quote: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Quote`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`148b160`](https://github.com/WordPress/gutenberg/commit/148b1605f6b0af7f7356183f15bb9540ad144ae5)
- **Discussion:** [#83178](https://github.com/WordPress/gutenberg/pull/83178) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Quote block now supports the `shadow` design tool, bringing it in line with other core blocks that already expose shadow controls. The change is a single `"shadow": true` entry in the block's `block.json` supports object; because Quote is a static block, the shadow CSS is serialized onto the wrapper `blockquote` by `useBlockProps.save()` with no PHP render-callback change. This is part of the broader design-tools consistency effort tracked in #43241.

## Impact

- **Site owners / content editors (block themes):** A new Shadow control appears under Blocks → Quote in Global Styles and in the block inspector. No action required; existing posts render unchanged until a shadow is explicitly applied.
- **Theme developers (block themes):** The Quote block now participates in the standard shadow cascade (Global Style → block instance → responsive state). No theme code changes are needed; the support is handled by the block editor's built-in serialization.
- **Plugin & theme developers (classic themes / custom render callbacks):** No effect. The change is confined to the block's `block.json` and the static-block save path.
- **Headless / REST consumers:** No schema or attribute change. The `shadow` support does not add a new block attribute; it is a CSS-level design tool applied at render time.

## Technical details

The functional change is one line in `packages/block-library/src/quote/block.json`, inside the existing `supports` object:

```json
"supports": {
  "align": ["full", "left", "right", "wide"],
  "allowedBlocks": [],
  "anchor": true,
  "background": { "backgroundImage": true, "backgroundSize": true, "gradient": true },
  "color": { "background": true, "gradients": true, "heading": true, "link": true, "text": true },
  "dimensions": { "minHeight": true },
  "interactivity": { "clientNavigation": true },
  "shadow": true,
  "layout": { "allowEditing": false },
  "spacing": { "blockGap": true, "margin": true, "padding": true },
  "typography": { "fontSize": true, "lineHeight": true }
}
```

Because the Quote block is static (no `render` callback in PHP), `useBlockProps.save()` in the block's `save.js` emits the shadow CSS custom properties directly onto the wrapper `<blockquote>` element. No PHP file is touched.

The remaining two files in the diff are regenerated documentation:
- `docs/reference-guides/core-blocks/README.md` — `shadow` added to the Quote block's supports list.
- `packages/block-library/src/quote/README.md` — a new bullet for the `shadow` support with a link to the Block Supports reference.

## Contribution

Opened by @aaronrobertshaw with @andrewserong as co-author. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion contains only two automated comments from `github-actions[bot]` (performance metrics and flaky-test report); no design debate or alternative approaches are visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
