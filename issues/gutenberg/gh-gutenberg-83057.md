# #83057: Verse: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Verse`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`ada86d0`](https://github.com/WordPress/gutenberg/commit/ada86d054fcc957e65b06bbae76aab5f1332b1b6)
- **Discussion:** [#83057](https://github.com/WordPress/gutenberg/pull/83057) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Verse block (labeled "Poetry" in the editor) now supports the `shadow` block support, closing a gap where it already had background, color, border, dimensions, spacing, and typography but no shadow control. This is part of the broader design-tools consistency effort tracked in #43241. Because Verse is a static block, the shadow is serialized onto the wrapper element at save time via `useBlockProps.save()`; no PHP rendering change is required.

## Impact

- **Site owners / block-theme users:** Can now apply a shadow to Poetry blocks through Global Styles (Blocks → Poetry → Shadow) or the block-level Shadow control in the inspector. No migration or configuration change needed.
- **Plugin & theme developers:** No code changes required. The existing `shadow` block-support infrastructure handles serialization and CSS generation. If you build custom Verse-based blocks or extend the block, the new `supports.shadow` key is now present in the registered block definition.
- **Hosting / platform:** No action required. No new DB columns, REST routes, or PHP rendering paths are introduced.
- **Headless / REST consumers:** The `shadow` attribute will appear in the block's serialized HTML (as an inline style on the wrapper) when set, but no new REST schema fields are added.

## Technical details

The functional change is a single line in `packages/block-library/src/verse/block.json`, adding `"shadow": true` to the `supports` object:

```json
"supports": {
  // …existing keys…
  "shadow": true,
  "spacing": { "margin": true, "padding": true, … }
}
```

Because Verse is a **static** block (its content is stored as the `content` attribute and rendered directly in the saved HTML), the block editor's `useBlockProps.save()` serializes the shadow CSS onto the wrapper `<p>` element at save time. There is no `render.php` or `render_callback` change.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — `shadow` added to the Verse supports list.
- `packages/block-library/src/verse/README.md` — a new bullet for `shadow: true` under the supports section.

No new hooks, filters, REST schema fields, or DB changes are introduced.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @jorgefilipecosta. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The record shows only 2 comments (both automated bot posts) and no design debate or alternative approaches discussed before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
