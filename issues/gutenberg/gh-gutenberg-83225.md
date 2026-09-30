# #83225: Gallery: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Gallery`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`d9b5c19`](https://github.com/WordPress/gutenberg/commit/d9b5c19cd5ebd9588aa536652ba924d49593751f)
- **Discussion:** [#83225](https://github.com/WordPress/gutenberg/pull/83225) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Gallery block now supports the Background panel's image, size, and gradient controls via the `background` block support, bringing it in line with other core blocks (Group, Cover, etc.) as part of the design-tools consistency effort. The change is purely declarative in `block.json`; no PHP render-callback modification is needed because Gallery already serializes its wrapper through `useBlockProps.save()` and applies classes via `WP_HTML_Tag_Processor`. The legacy `color.gradients` support is retained alongside the new `background.gradient`, matching the pattern used by the Group block.

## Impact

- **Theme & site builders (Global Styles):** The Gallery block now exposes Background → Image, Size, and Gradient controls in the Site Editor's Styles panel. You can set a background image (with `cover`/`contain`/etc. sizing) and a gradient that layers over the image, with responsive (mobile/tablet) overrides.
- **Block-level overrides:** The same three controls appear in the block inspector for individual Gallery instances, overriding Global Style values.
- **Plugin & theme developers:** No code changes required. The supports are serialized automatically. If you build custom Gallery wrappers or filter `WP_HTML_Tag_Processor` output, be aware the wrapper element may now carry `background-image`, `background-size`, and `background` (gradient) CSS properties.
- **No breaking changes.** The legacy `color.gradients` support remains active; existing sites are unaffected until a user explicitly sets a background value.

## Technical details

The diff adds a single `background` object to the `supports` key in `packages/block-library/src/gallery/block.json`:

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

`__experimentalDefaultControls` enables the image and gradient pickers by default in the block inspector (the size control is available but not shown by default). The background paints the gallery's wrapper container behind the images; the gradient layers over the image (per the pattern established in #75859).

No changes to `render.php`, `index.js`, or any PHP file are present in the diff. The block's existing save path (`useBlockProps.save()`) and its `WP_HTML_Tag_Processor`-based class application in the render callback are sufficient for the supports to serialize into the saved markup.

Two documentation files are regenerated to list the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/gallery/README.md`.

## Contribution

Opened by @aaronrobertshaw and co-authored with @andrewserong. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion is minimal (3 comments, 0 reactions): @aaronrobertshaw thanked a reviewer for adding screenshots and mentioned hitting a GitHub upload throttle. No design debate or rejected alternatives are visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
