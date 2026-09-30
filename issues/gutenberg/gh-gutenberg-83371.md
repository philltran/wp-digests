# #83371: Accordion Item: Add link, heading, and button colour support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`f17dbdd`](https://github.com/WordPress/gutenberg/commit/f17dbdd3137fa6061e2744b56a9e908c54e2a4e3)
- **Discussion:** [#83371](https://github.com/WordPress/gutenberg/pull/83371) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Accordion Item block (`core/accordion-item`) now declares `link`, `heading`, and `button` in its `color` supports, enabling element-level colour controls in the block inspector and Global Styles. Previously the block only exposed `background` and `gradients` colour supports, so there was no UI to set colours for links, headings, or buttons rendered inside an accordion item. This brings the block in line with other container blocks (Columns, Group) as part of the ongoing design-tools consistency work tracked in #43241.

## Impact

- **Site owners / block-theme builders:** Can now set link, heading, and button colours for Accordion Items via Global Styles → Blocks → Accordion Item → Elements, or per-instance in the block inspector. Mobile/responsive element colours are also supported.
- **Plugin & theme developers:** No code changes required. The `wp-elements-*` CSS classes were already output by the elements support; this PR only adds the editor-side controls. No new hooks, filters, or REST schema changes.
- **No breaking changes.** Existing `style.elements` values saved on Accordion Item blocks already rendered on the front end; they simply now have corresponding UI controls.

## Technical details

The functional change is confined to `packages/block-library/src/accordion-item/block.json`. Three keys are added to the existing `color` supports object:

```json
"color": {
  "background": true,
  "gradients": true,
  "link": true,
  "heading": true,
  "button": true
}
```

Because the block's render callback only appends interactivity attributes, the `wp-elements-*` class emitted by the elements colour support is preserved, and no PHP render change is needed. The two README files (`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/accordion-item/README.md`) are updated to document the new supports entries. No new hooks, filters, `block.json` fields beyond the supports keys, or database changes are introduced.

## Contribution

Opened by @aaronrobertshaw with @talldan as co-author. The PR body notes it was "Implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion is minimal (2 comments, 0 reactions), with no visible design debate or rejected alternatives. It is part of the broader #43241 effort to bring element-colour supports to remaining core blocks.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
