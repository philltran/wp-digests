# #83203: Buttons: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Buttons`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`de87faf`](https://github.com/WordPress/gutenberg/commit/de87faf0a704df2125b8346ccd0d4c008cabe438)
- **Discussion:** [#83203](https://github.com/WordPress/gutenberg/pull/83203) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Buttons block (`core/buttons`) now supports the Background panel's image, size, and gradient controls, bringing it in line with other core blocks as part of the design-tools consistency effort. The background paints the Buttons container behind its child buttons, not the buttons themselves. The legacy `color.gradients` support is retained alongside the new `background.gradient`, matching the pattern already used by Group, Quote, Pullquote, and Verse.

## Impact

- **Theme developers / Global Styles authors:** The Buttons block now exposes `backgroundImage`, `backgroundSize`, and `gradient` in the Background panel under Blocks → Buttons. No code changes required; the controls appear automatically in the Site Editor and block inspector.
- **Plugin & theme developers:** No breaking changes. The new supports are additive in `block.json`. If you build custom blocks that wrap or extend Buttons, the wrapper will now carry background CSS custom properties when a user sets them.
- **Site owners / editors:** New design controls are available in the block inspector and Global Styles. No migration or configuration needed.
- **No action required** for existing sites; the change is purely additive.

## Technical details

The diff adds a `background` object to `packages/block-library/src/buttons/block.json`:

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

Because Buttons is a static block, these supports are serialized onto the wrapper element by `useBlockProps.save()`; no PHP render callback change is needed. The existing `color.gradients` support remains in place alongside `background.gradient`, so both the legacy gradient path and the new Background-panel gradient path coexist (same pattern as Group, Quote, Pullquote, Verse). Per the PR description, the gradient layers over the background image (referencing #75859), so both render simultaneously.

The `__experimentalDefaultControls` key enables the image and gradient controls by default in the block inspector (as opposed to hiding them behind an "Advanced" toggle). The core-blocks reference (`docs/reference-guides/core-blocks/README.md`) and the block's own `README.md` are regenerated to list the new supports.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @ramonjd. The PR notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives; it was merged straightforwardly as part of the broader design-tools consistency work tracked in #43241.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
