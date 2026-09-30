# #83380: Media & Text: Add button colour support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Media & Text`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`9338f3e`](https://github.com/WordPress/gutenberg/commit/9338f3ebe542b7d3ffa44619644324b1d6541dd0)
- **Discussion:** [#83380](https://github.com/WordPress/gutenberg/pull/83380) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds `color.button` to the Media & Text block's `block.json` supports, enabling button colour controls in the block inspector and Global Styles. The block already supported `link` and `heading` colours but not `button`, leaving a gap for a common element in the text column. This closes that gap as part of the broader design-tools consistency effort tracked in #43241.

## Impact

- **Site owners (block themes):** Can now set a button colour for Media & Text blocks via Global Styles → Blocks → Media & Text → Elements → Button, or per-block in the inspector. Mobile-state button colours are also available.
- **Plugin & theme developers:** No code changes required. The existing `save.js` wrapper and the render callback's `WP_HTML_Tag_Processor` already emit the necessary classes; `lib/block-supports/elements.php` already serialises element colours for blocks that do not opt out. This PR only adds the editor-side controls.
- **No breaking changes, deprecations, or migration steps.**

## Technical details

The functional change is a single line in `packages/block-library/src/media-text/block.json`, adding `"button": true` to the existing `color` supports object alongside `background`, `gradients`, `heading`, `link`, and `text`:

```json
"color": {
  "background": true,
  "gradients": true,
  "heading": true,
  "link": true,
  "button": true,
  "__experimentalDefaultControls": { "background": true, "text": true }
}
```

No PHP changes were made. The PR description notes that the block's `save.js` already wraps content in the element-colour wrapper, and the render callback's `WP_HTML_Tag_Processor` only adds or removes classes, so the front-end output was already correct. The change surfaces the existing capability in the editor (block inspector Styles panel and Global Styles editor).

Two documentation files were regenerated to list the new support:
- `docs/reference-guides/core-blocks/README.md` — the Media & Text entry now reads `color (background, button, gradients, heading, link, text)`.
- `packages/block-library/src/media-text/README.md` — adds a `button: true` bullet under the `color` supports section.

## Contribution

Opened by @aaronrobertshaw with @ramonjd as co-author. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The two comments on the PR are both automated (performance metrics and co-author attribution); no design debate or alternative approaches are recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
