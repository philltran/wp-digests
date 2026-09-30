# #83360: Home Link: Add border support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Home Link`
- **Merged:** [`acbe785`](https://github.com/WordPress/gutenberg/commit/acbe785b3649bcf7ebe5762149a81619782a1036)
- **Discussion:** [#83360](https://github.com/WordPress/gutenberg/pull/83360) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Home Link block now supports border colour, radius, style, and width via the `__experimentalBorder` block-support key in its `block.json`. The four border controls are hidden behind the Border panel's overflow menu rather than appearing as standalone controls, matching the pattern established for Post Navigation Link (#83122). This closes a gap in the Home Link's design-tools coverage, which previously offered background, shadow, and typography but no border control.

## Impact

- **Site owners / editors:** Border properties for the Home Link block are now available under Global Styles → Blocks → Home Link → Border, and in the block inspector. No code or configuration changes required.
- **Plugin & theme developers:** No action required. The Home Link block is server-rendered through `get_block_wrapper_attributes()`, so border styles serialize onto the menu-item wrapper automatically. No PHP template or render-callback changes are needed.
- **Headless / REST consumers:** No schema or REST-route changes. The `__experimentalBorder` key is skipped by the docs generators, so no generated documentation output changes.

## Technical details

The sole file changed is `packages/block-library/src/home-link/block.json`. The diff inserts a new `__experimentalBorder` object into the block's `supports` array, alongside the existing `shadow` and `interactivity` entries:

```json
"__experimentalBorder": {
  "color": true,
  "radius": true,
  "style": true,
  "width": true,
  "__experimentalDefaultControls": {
    "color": false,
    "radius": false,
    "style": false,
    "width": false
  }
}
```

Setting every `__experimentalDefaultControls` value to `false` means the four border controls do not render as top-level inspector controls; they are accessible only through the Border panel's menu, the same behaviour used by Post Navigation Link (#83122). Because the Home Link block is server-rendered via `get_block_wrapper_attributes()`, the border CSS custom properties are emitted on the wrapper element without any PHP-side change. The block's label falls back to the string "Home", so the wrapper element is never empty and the border is always visible when styled.

## Contribution

Opened by @aaronrobertshaw and co-authored with @andrewserong. The PR body notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) and the change follows the established pattern from #83122, so no notable design debate or rejected alternatives are recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
