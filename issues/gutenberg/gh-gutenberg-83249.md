# #83249: Social Icons: Add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Social`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`6bbc1fb`](https://github.com/WordPress/gutenberg/commit/6bbc1fb345b43adf1fba4853a75307c86d803aff)
- **Discussion:** [#83249](https://github.com/WordPress/gutenberg/pull/83249) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Social Icons block (`core/social-links`) now declares the `background.gradient` block support, enabling the Background panel's gradient controls in both Global Styles and the block inspector. This closes a gap where the block already supported color (including legacy `color.gradients`), border, layout, and spacing, but not the newer Background gradient API. The change is part of the broader design-tools consistency effort tracked in #43241.

## Impact

- **Site owners / editors:** Can now pick a background gradient for Social Icons in Global Styles (Blocks → Social Icons → Background) or per-instance in the block inspector, with responsive (mobile/tablet/desktop) variants. No code changes required.
- **Theme & plugin developers:** No action required. The support is declarative in `block.json`; no render-callback or PHP changes are needed because the block is static and the support is serialized by `useBlockProps.save()`. The legacy `color.gradients` support is retained alongside the new key, so existing themes that target the old gradient mechanism continue to work.
- **Headless / REST consumers:** No schema or route changes. The serialized markup gains the standard background-gradient CSS custom properties on the list wrapper element.

## Technical details

The diff adds a single key to the `supports` object in `packages/block-library/src/social-links/block.json`:

```json
"background": {
  "gradient": true
}
```

Because `core/social-links` is a static block (no dynamic render callback), the support is picked up automatically by `useBlockProps.save()` and emitted as CSS custom properties on the `<ul>` wrapper. No PHP file is touched.

The existing `color` support object (which includes `"gradients": true` for the legacy gradient mechanism) is left unchanged, so both the old and new gradient paths coexist.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — the Social Icons entry's supports list gains `background (gradient)`.
- `packages/block-library/src/social-links/README.md` — a new `background` section with `gradient: true` is inserted before the existing `color` section.

## Contribution

Opened by @aaronrobertshaw with @ramonjd as co-author. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The two comments on the PR are automated (performance metrics and co-author attribution); no design debate or alternative approaches are visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
