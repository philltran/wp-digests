# #83188: Tag Cloud: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Tag Cloud`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`1d3f591`](https://github.com/WordPress/gutenberg/commit/1d3f591e4ca8f7c99e039b12808461a864aa4964)
- **Discussion:** [#83188](https://github.com/WordPress/gutenberg/pull/83188) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tag Cloud block (`core/tag-cloud`) now supports the `shadow` block support, allowing site editors to apply box-shadow presets via Global Styles or the block inspector. This closes a gap where the block already supported border, spacing, and typography but had no shadow control, as part of the broader design-tools consistency effort tracked in #43241. Because the block is server-rendered through `get_block_wrapper_attributes()`, the support serializes onto the wrapper `<p>` element with no PHP-side change required.

## Impact

- **Site owners / block-theme editors:** Can now set a shadow on Tag Cloud blocks in Global Styles (Blocks → Tag Cloud → Shadow) or per-instance in the block inspector, including responsive (mobile/tablet/desktop) variants. No code changes needed.
- **Plugin & theme developers:** No action required. The change is additive to `block.json`; no new hooks, filters, or PHP APIs are introduced. Themes that already handle `get_block_wrapper_attributes()` output will pick up the shadow automatically.
- **Headless / REST consumers:** The `shadow` key appears in the block's `supports` object in the REST block schema. No new routes or attributes are added.

## Technical details

The functional change is a single line in `packages/block-library/src/tag-cloud/block.json`, adding `"shadow": true` to the `supports` object:

```json
"supports": {
  "anchor": true,
  "html": false,
  "align": true,
  "shadow": true,
  "spacing": { "margin": true, "padding": true },
  ...
}
```

Because Tag Cloud is a dynamic block rendered server-side via `get_block_wrapper_attributes()`, the shadow CSS custom properties (`--wp--custom--shadow--*`) are emitted on the wrapper `<p>` element by the existing block-supports machinery. No changes to `render.php`, `index.js`, or any PHP template are needed. The shadow applies to the entire cloud container, not to individual tag links, mirroring how the existing `border` support behaves.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — `shadow` added to the Tag Cloud supports list.
- `packages/block-library/src/tag-cloud/README.md` — a new bullet for `shadow: true` added under the supports section.

## Contribution

Opened by @aaronrobertshaw and co-authored with @ramonjd. The PR body notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no design debate or rejected alternatives visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
