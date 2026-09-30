# #83181: Site Tagline: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Site Tagline`
- **Merged:** [`a7dce24`](https://github.com/WordPress/gutenberg/commit/a7dce2477eea44570c477fabfbd7afae7c8be744)
- **Discussion:** [#83181](https://github.com/WordPress/gutenberg/pull/83181) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Site Tagline block (`core/site-tagline`) now supports the `shadow` block support, adding box-shadow control via Global Styles and per-block overrides. This closes a gap where the block already supported color, border, spacing, and typography but had no shadow control, as part of the broader design-tools consistency effort tracked in #43241. The change is a single `"shadow": true` entry in the block's `block.json` supports object; because the block is server-rendered through `get_block_wrapper_attributes()`, no PHP render-callback change is required.

## Impact

- **Theme developers / site editors:** The Site Tagline block now accepts shadow presets (Natural, Deep, Sharp, Crisp, etc.) in Global Styles → Blocks → Site Tagline → Shadow, and per-instance shadow overrides in the block inspector. No code changes are needed; the support serializes onto the wrapper automatically.
- **Plugin & theme developers:** No breaking changes, no new hooks, no new REST schema fields. If you build custom tooling that enumerates a block's `supports`, the Site Tagline block will now report `shadow: true`.
- **No action required** for existing sites or themes. The block renders identically when no shadow is applied.

## Technical details

The functional change is one line in `packages/block-library/src/site-tagline/block.json`, adding `"shadow": true` to the existing `supports` object:

```json
"supports": {
  "align": ["full", "wide"],
  "anchor": true,
  "color": { "background": true, "gradients": true, "text": true },
  "contentRole": true,
  "interactivity": { "clientNavigation": true },
  "spacing": { "margin": true, "padding": true },
  "typography": { "fitText": true, "fontSize": true, "lineHeight": true, "textAlign": true },
  "shadow": true
}
```

Because `core/site-tagline` is server-rendered via `get_block_wrapper_attributes()`, the shadow support is serialized onto the wrapper element's inline styles automatically by the block-supports machinery — no change to a render callback or PHP template is needed.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — `shadow` added to the Site Tagline supports list.
- `packages/block-library/src/site-tagline/README.md` — a new `shadow: true` bullet added under the supports section.

## Contribution

Opened by @aaronrobertshaw with co-authors @ramonjd and @shail-mehta. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The two comments on the PR are both automated (performance metrics and flaky-test reports from `github-actions[bot]`); no design debate or alternative approaches are visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
