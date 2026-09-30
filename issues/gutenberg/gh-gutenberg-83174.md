# #83174: Preformatted: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Preformatted`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`3952f65`](https://github.com/WordPress/gutenberg/commit/3952f6577fe1e7120e8e193b3e63af09287e03dc)
- **Discussion:** [#83174](https://github.com/WordPress/gutenberg/pull/83174) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Preformatted block (`core/preformatted`) now supports the `shadow` block support, allowing box-shadow styling via Global Styles and the block inspector. This closes a gap where the block already supported background, color, border, spacing, and typography but had no shadow control, as part of the broader design-tools consistency effort tracked in #43241. Because Preformatted is a static block, the change is a single `block.json` entry; no render-callback or PHP changes are required.

## Impact

- **Site owners / editors:** A new Shadow control appears under Blocks → Preformatted in Global Styles and in the block inspector. No migration or configuration needed; existing Preformatted blocks render unchanged until a shadow is explicitly applied.
- **Plugin & theme developers:** No code changes required. The shadow CSS is emitted by the existing block-supports infrastructure. If you build custom styles targeting `pre` elements from the Preformatted block, be aware that a `box-shadow` declaration may now appear on the wrapper.
- **Headless / REST consumers:** No schema or attribute changes. The shadow is a style, not a block attribute, so serialized block markup is unaffected unless a shadow is set.
- **No breaking changes, deprecations, or removed APIs.**

## Technical details

The functional change is a single line in `packages/block-library/src/preformatted/block.json`, adding `"shadow": true` to the `supports` object:

```json
"supports": {
  "color": { "background": true, "gradients": true, "text": true },
  "shadow": true,
  "spacing": { "padding": true, "margin": true },
  "typography": { "fontSize": true, "lineHeight": true }
}
```

Because Preformatted is a static block (no dynamic render callback), the shadow CSS is serialized onto the wrapper `<pre>` element by `useBlockProps.save()` at save time and by `useBlockProps()` in the editor. No changes to `render.php`, `index.js`, or any PHP file are present in the diff.

Two documentation files are regenerated to list the new support:
- `docs/reference-guides/core-blocks/README.md` — the Preformatted entry's supports line now includes `shadow`.
- `packages/block-library/src/preformatted/README.md` — a new bullet for `shadow: true` is added to the supports list.

## Contribution

Opened by @aaronrobertshaw and reviewed by @andrewserong, who is also listed as a co-author. The PR description notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion is minimal — the author's only substantive comment thanks the reviewer. The change is one of a series of similar per-block shadow-support additions under the #43241 design-tools consistency umbrella.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
