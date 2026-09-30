# #83182: Site Title: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Site Title`
- **Merged:** [`57f1251`](https://github.com/WordPress/gutenberg/commit/57f125120092d69fd6302b772a249427ef399d14)
- **Discussion:** [#83182](https://github.com/WordPress/gutenberg/pull/83182) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Site Title block (`core/site-title`) now supports the `shadow` design tool, bringing it in line with other core blocks that already expose color, border, spacing, and typography controls. The change is a single addition of `"shadow": true` to the block's `supports` in `block.json`; because the block is server-rendered via `get_block_wrapper_attributes()`, the support serializes onto the wrapper automatically with no PHP change. This is part of the broader design-tools consistency effort tracked in #43241.

## Impact

- **Theme developers (block themes):** No code change required. The new `shadow` support appears automatically in Global Styles (Blocks → Site Title → Shadow) and in the block inspector. Existing themes that do not set a shadow will render identically to before.
- **Plugin developers:** No action required. No new hooks, filters, or REST schema fields are introduced.
- **Site owners / editors:** A new Shadow control is available for the Site Title block in both Global Styles and the per-block inspector, including responsive (mobile/tablet/desktop) variants.
- **No breaking changes, deprecations, or migrations.**

## Technical details

The functional change is a single key added to the `supports` object in `packages/block-library/src/site-title/block.json`:

```json
// before
"supports": {
  "align": ["full", "wide"],
  "anchor": true,
  "color": { "background": true, "gradients": true, "link": true, "text": true },
  "spacing": { "margin": true, "padding": true },
  "typography": { "fitText": true, "fontSize": true, "lineHeight": true, "textAlign": true },
  "interactivity": { "clientNavigation": true },
  "border": { "color": true, "width": true, "style": true }
}

// after — adds:
"shadow": true
```

Because `core/site-title` is rendered server-side through `get_block_wrapper_attributes()`, the shadow support is serialized onto the block wrapper's `style` attribute (e.g. `--wp--preset--shadow--natural`) without any change to the block's `render.php` or PHP render callback. The two documentation files (`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/site-title/README.md`) are regenerated to list `shadow` in the supports table.

## Contribution

Opened and merged by @aaronrobertshaw with co-authorship from @andrewserong. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The record carries no substantive design debate or alternative-approach discussion beyond the two bot comments (performance metrics and contributor attribution).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
