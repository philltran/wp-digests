# #83386: Preformatted: Add link colour support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Preformatted`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`9ba5f6d`](https://github.com/WordPress/gutenberg/commit/9ba5f6dff6f98da92251d47dee12eaf47685e607)
- **Discussion:** [#83386](https://github.com/WordPress/gutenberg/pull/83386) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Preformatted block now supports link colour via the `color.link` block support, adding a link colour control to the block inspector and Global Styles. Previously the block supported text, background, and gradient colours but had no way to style links within its content. This brings the Preformatted block in line with other text blocks as part of the ongoing design-tools consistency effort (related to #43241).

## Impact

- **Site owners / content editors:** A new "Link" element colour control appears under Styles → Elements in the Preformatted block inspector and under Global Styles → Blocks → Preformatted → Elements. No migration or configuration change is needed.
- **Theme developers:** No code change required. The block is static and `save.jsx` already supplies the wrapper via `useBlockProps.save()`, so the existing element-colour serialization in `lib/block-supports/elements.php` handles rendering. No PHP template or CSS changes are needed.
- **Plugin developers:** No API change. This is a `block.json` supports addition on a core block; no new hooks, filters, or REST schema fields are introduced.
- **No action required** for any audience. The change is purely additive.

## Technical details

The diff adds a single key to the `color` supports object in `packages/block-library/src/preformatted/block.json`:

```json
"color": {
  "gradients": true,
  "link": true,
  "__experimentalDefaultControls": {
    "background": true,
    "text": true
  }
}
```

This mirrors the pattern used for Tag Cloud in #83264. Because the Preformatted block is static (its `save.jsx` renders the wrapper through `useBlockProps.save()`), no PHP render-callback change is needed. The PR description notes that element colours set on a block already render on the front end even without the support declared, since `lib/block-supports/elements.php` only skips blocks that explicitly opt out of serialization. The `block.json` change therefore gates the *editor controls* (block inspector and Global Styles), not the front-end rendering.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — the Preformatted entry's supports list now reads `color (background, gradients, link, text)`.
- `packages/block-library/src/preformatted/README.md` — adds `link: true` under the `color` supports section.

## Contribution

Opened and merged by @aaronrobertshaw with co-authorship from @andrewserong. The PR notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no design debate or rejected alternatives recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
