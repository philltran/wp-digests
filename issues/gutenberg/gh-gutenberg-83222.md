# #83222: Details: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Details`
- **Merged:** [`006bbc1`](https://github.com/WordPress/gutenberg/commit/006bbc1d8ba4da4ecaa3d6e064ee6377b419d514)
- **Discussion:** [#83222](https://github.com/WordPress/gutenberg/pull/83222) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Details block (`core/details`) now supports background image, background size, and gradient via the Background panel in both the block inspector and Global Styles. This closes a gap where the block already supported color, border, layout, shadow, spacing, and typography but lacked the Background panel's image/size/gradient controls. The change is part of the broader design-tools consistency effort tracked in #43241.

## Impact

- **Theme & site builders:** The Details block now exposes `backgroundImage`, `backgroundSize`, and `gradient` in the Background panel (block inspector) and in Global Styles → Blocks → Details → Background. No code changes needed; the controls appear automatically.
- **Plugin & theme developers:** If you register custom styles or query the Details block's supports, the `background` key is now present in `block.json`. The legacy `color.gradients` support remains alongside the new `background.gradient`, so existing gradient styling via the Color panel is unaffected.
- **No action required** for existing sites. The change is purely additive; blocks without a background set render identically to before.

## Technical details

The diff adds a `background` object to `packages/block-library/src/details/block.json`:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true,
  "__experimentalDefaultControls": {
    "backgroundImage": true,
    "gradient": true
  }
}
```

Because Details is a static block, these supports are serialized onto the `<details>` wrapper element by `useBlockProps.save()` at save time; no PHP render-callback change is needed (the existing render filter only touches images). The legacy `color.gradients: true` entry is retained alongside `background.gradient`, matching the pattern used by the Group block.

Per #75859, the gradient renders as a layer over the background image, so both are visible simultaneously. The `__experimentalDefaultControls` key enables the image and gradient controls by default in the inspector without requiring the user to toggle them on.

The core-blocks reference (`docs/reference-guides/core-blocks/README.md`) and the block's own `README.md` are regenerated to list the new supports.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. In the one substantive discussion point, @aaronrobertshaw acknowledged that block-level background styles currently overwrite rather than merge with Global Styles values, but agreed that fixing that merge behavior is a separate concern and that adopting the supports here does not lock in the overwrite behavior. No alternative approaches were proposed or rejected.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
