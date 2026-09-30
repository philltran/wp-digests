# #83374: Accordion Panel: Add link, heading, and button colour support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`179775b`](https://github.com/WordPress/gutenberg/commit/179775b854bcaf9cb8bf872c946dd23500760de9)
- **Discussion:** [#83374](https://github.com/WordPress/gutenberg/pull/83374) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `core/accordion-panel` block now declares `color.link`, `color.heading`, and `color.button` in its `block.json`, enabling element-level colour styling from Global Styles and the block inspector. Previously, the panel's revealed content (headings, links, buttons) could not be individually coloured through the design tools. The change is purely additive: the block is static, so `useBlockProps.save()` serialises the new styles automatically and no PHP rendering change was required.

## Impact

- **Site owners / block-theme users:** Can now set link, heading, and button colours on Accordion Panel content via Styles → Blocks → Accordion Panel → Elements (Global Styles) or the block inspector's Styles → Elements panel. Responsive (mobile) overrides work as with other element colours.
- **Plugin & theme developers:** No action required. The change is additive to a core block's `block.json`; no new hooks, filters, or API surface are introduced.
- **Hosting & platform:** No migration, configuration, or code changes needed.
- **Headless & REST consumers:** No schema or route changes; the new supports are consumed by the editor and the existing `lib/block-supports/elements.php` renderer.

## Technical details

The diff modifies three files:

1. **`packages/block-library/src/accordion-panel/block.json`** — three keys added under `supports.color`:

```json
"color": {
    "background": true,
    "gradients": true,
    "link": true,
    "heading": true,
    "button": true
}
```

2. **`packages/block-library/src/accordion-panel/README.md`** — generated docs updated to list the three new sub-supports under `color`.

3. **`docs/reference-guides/core-blocks/README.md`** — the Accordion Panel entry's supports line updated from `color (background, gradients, text)` to `color (background, button, gradients, heading, link, text)`.

No changes to `save.jsx`, `index.js`, or any PHP file. The block is static: `save.jsx` supplies the wrapper via `useBlockProps.save()`, so the element-colour CSS custom properties are serialised into the saved markup automatically. On the front end, `lib/block-supports/elements.php` already renders element colours for any block that has not explicitly opted out of serialization; this PR adds the editor-side controls (Global Styles panel and block inspector) that were previously absent because the supports were not declared.

## Contribution

Opened by @aaronrobertshaw as part of the design-tools consistency effort tracked in #43241, with @im3dabasia listed as co-author. The PR attracted only 2 comments and 0 reactions, with no visible design debate or alternative approaches discussed. The record carries no further discussion detail beyond the automated performance and co-author bot comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
