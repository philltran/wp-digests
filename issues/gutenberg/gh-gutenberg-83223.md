# #83223: File: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] File`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`8c826d0`](https://github.com/WordPress/gutenberg/commit/8c826d04c5213db1a32181124e267896be0c14f4)
- **Discussion:** [#83223](https://github.com/WordPress/gutenberg/pull/83223) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The File block (`core/file`) now supports background image, background size, and gradient via the Background panel in Global Styles and the block inspector. This closes a gap where the block already supported color, border, and spacing but not the Background panel's image/size/gradient controls, as part of the broader design-tools consistency effort tracked in #43241. The change is purely declarative in `block.json`; no PHP or render-callback modification is required.

## Impact

- **Site owners / editors:** Can now assign a background image (with `cover`, `contain`, etc. sizing) and a gradient to File blocks through Global Styles → Blocks → File → Background, or per-instance in the block inspector. Gradient layers over the image (per #75859), so both render together.
- **Theme & plugin developers:** No code changes required. The supports are declarative; if you override or extend the File block's `block.json`, be aware the `background` key is now present. The legacy `color.gradients` support remains alongside the new `background.gradient`.
- **No action required** for existing sites. Previously saved File blocks render identically (no background) until a user explicitly sets one.

## Technical details

The diff adds a `background` object to the `supports` key in `packages/block-library/src/file/block.json`:

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

Because `core/file` is a static block, these supports are serialized onto the wrapper element by `useBlockProps.save()` at save time. The block's PHP render callback only adds Interactivity API attributes and adjusts the embedded preview via `WP_HTML_Tag_Processor`; it never touches the wrapper's `class` or `style`, so no PHP change was needed.

The legacy `color.gradients` support is retained alongside the new `background.gradient` key. The gradient renders as a layer over the background image (behavior established in #75859).

Two documentation files are regenerated to list the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/file/README.md`.

## Contribution

Opened by @aaronrobertshaw with @andrewserong as co-author. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives; the change follows the established pattern of adding Background-panel supports to a core block.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
