# #83193: Terms List: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Categories`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`96ddd8f`](https://github.com/WordPress/gutenberg/commit/96ddd8f503d25e2b387e06bb03e9a418bb4969b4)
- **Discussion:** [#83193](https://github.com/WordPress/gutenberg/pull/83193) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds `shadow` to the design-tools supports of the Terms List block (`core/categories`), bringing it in line with other core blocks that already expose shadow controls. The change is a single `"shadow": true` entry in the block's `block.json`; no PHP or template changes are required because the block is server-rendered through `get_block_wrapper_attributes()`, which serializes the shadow CSS onto the rendered element. This is part of the broader design-tools consistency effort tracked in #43241.

## Impact

- **Site owners / block-theme users:** Can now apply a shadow to a Terms List block via Global Styles (Blocks → Terms List → Shadow) or the block inspector, including responsive (mobile/tablet/desktop) variants. No action required.
- **Theme & plugin developers:** No code changes needed. If you build custom styles or overrides for `core/categories`, be aware the block now emits a `box-shadow` declaration when a shadow is set. No new hooks, filters, or attributes are introduced.
- **Headless / REST consumers:** No schema or route changes. The block's serialized HTML will include an inline `box-shadow` style when a shadow is configured, same as any other supported block.

## Technical details

The diff adds one key to the `supports` object in `packages/block-library/src/categories/block.json`:

```json
"supports": {
  /* …existing entries… */
  "shadow": true
}
```

Because `core/categories` is a server-rendered block, the shadow CSS is produced by the core block-supports machinery and attached via `get_block_wrapper_attributes()`. In list mode the attribute lands on the `<ul>`; in dropdown mode (`displayAsDropdown: true`) it lands on the wrapper `<div>` that contains the `<label>` and `<select>`. No PHP template file is modified.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — the `core/categories` line now lists `shadow` in its supports.
- `packages/block-library/src/categories/README.md` — a new bullet for `shadow: true` is added under the supports section.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @shail-mehta. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. Discussion was minimal (2 comments, 0 reactions), consistent with a straightforward, well-scoped addition to an ongoing consistency effort (#43241). No design debate or alternative approaches are recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
