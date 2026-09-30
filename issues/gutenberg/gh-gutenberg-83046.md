# #83046: Home Link: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Home Link`
- **Merged:** [`bb115b8`](https://github.com/WordPress/gutenberg/commit/bb115b86749ffd6dcea420a92b7fed6f467cdd1d)
- **Discussion:** [#83046](https://github.com/WordPress/gutenberg/pull/83046) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds `shadow` support to the Home Link block (`core/home-link`), bringing it in line with other core blocks as part of the design tools consistency effort tracked in #43241. The block previously supported typography (`fontSize`, `lineHeight`) but had no shadow control. Because the block is server-rendered through `get_block_wrapper_attributes()`, the change is a single-line addition to `block.json` with no PHP modification required.

## Impact

- **Block theme developers:** Home Link now accepts a shadow preset via Global Styles (Blocks → Home Link → Shadow) and a per-instance override in the block inspector. No code changes needed; the support is opt-in through the existing design tools UI.
- **Plugin & theme developers:** No breaking changes, no new hooks, no new REST schema fields. If you programmatically inspect `core/home-link`'s `block.json` supports, `shadow` will now appear.
- **No action required** for existing sites. The support is not exposed by default; shadows only render when a user explicitly selects a preset.

## Technical details

The functional change is a single key added to the `supports` object in `packages/block-library/src/home-link/block.json`:

```json
"supports": {
  "typography": {
    "lineHeight": true,
    "fontSize": true
  },
  "shadow": true,
  "interactivity": {
    "clientNavigation": true
  }
}
```

No PHP change is needed because the Home Link block renders server-side via `get_block_wrapper_attributes()`, which serializes the `shadow` support onto the `<li>` wrapper element automatically. The two README files (`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/home-link/README.md`) are regenerated to list `shadow` in the supports table.

## Contribution

Opened by @aaronrobertshaw and implemented by a Claude Code agent from a predefined task. @im3dabasia initially left an LGTM that was not backed by a review; after @aaronrobertshaw asked for a more complete review (what was tested, edge cases, screenshots), @im3dabasia provided a proper walkthrough. A notable edge case raised: Home Link has no padding support, so deeper shadow presets (Deep, Solid) sit tight against the text. @shail-mehta pointed to #43241 as the tracking issue for spacing; @aaronrobertshaw confirmed it is fine to land shadow early since both supports are simple and not exposed by default, and that a broader rollout of design tool adoptions is planned.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
