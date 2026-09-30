# #83180: Read More: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Read More`
- **Merged:** [`a905f9a`](https://github.com/WordPress/gutenberg/commit/a905f9ad5e26737298b96c894729c4a785211a73)
- **Discussion:** [#83180](https://github.com/WordPress/gutenberg/pull/83180) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Read More block (`core/read-more`) now supports the `shadow` block support, allowing box-shadow values to be applied via Global Styles or the block inspector. This closes a gap in the block's design tooling — it previously supported color, border, spacing, and typography but had no shadow control. Because the block is server-rendered through `get_block_wrapper_attributes()`, the new support serializes onto the wrapper automatically with no PHP-side change.

## Impact

- **Site builders / theme developers:** Read More blocks can now receive shadow presets (Natural, Deep, Sharp, Crisp, etc.) from Global Styles or per-block overrides, including responsive variants. No code changes required.
- **Plugin & theme developers:** No action required. The support is additive; existing Read More markup is unchanged when no shadow is set.
- **Headless / REST consumers:** No schema or route changes. The shadow value flows through the standard block attributes and wrapper attributes pipeline.

## Technical details

The functional change is a single line in `packages/block-library/src/read-more/block.json`, adding `"shadow": true` to the `supports` object:

```json
"supports": {
  // …existing color, spacing, typography…
  "shadow": true,
  "interactivity": {
    "clientNavigation": true
  }
}
```

Because `core/read-more` renders server-side via `get_block_wrapper_attributes()`, the shadow support is picked up by the standard block-supports serialization and emitted as inline `box-shadow` styles on the wrapper element. No changes to `render.php`, `index.js`, or any PHP template are needed.

Two documentation files are regenerated to reflect the new support:
- `packages/block-library/src/read-more/README.md` — adds `shadow: true` to the supports list.
- `docs/reference-guides/core-blocks/README.md` — adds `shadow` to the Read More entry's supports line.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @andrewserong. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. It references the broader design-tools consistency tracking issue #43241. The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
