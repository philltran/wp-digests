# #81307: Render viewport state element styles in the editor

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Bug`, `Backported to WP Core`, `[Feature] Style States`
- **Merged:** [`e898723`](https://github.com/WordPress/gutenberg/commit/e89872315e3d5947006e76772f6ebf3d4dcc5ba8)
- **Discussion:** [#81307](https://github.com/WordPress/gutenberg/pull/81307) · 5 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The editor-side global styles renderer (`@wordpress/global-styles-engine`) now emits block-level element styles that exist only inside a responsive viewport state, such as `styles.blocks['core/group']['@mobile'].elements.heading`. Previously, when an element had no default-state styles, the viewport-only branch was never expanded, so those rules were missing from the editor stylesheet. This is a follow-up to #81265, which applied the same rule to the front-end output.

## Impact

- **Site owners / editors:** Viewport-specific element styles set in Global Styles for a block (e.g. a Heading inside a Group on mobile only) now show in the editor's responsive preview and match the front end.
- **Plugin & theme developers:** No API changes. If your `theme.json` or user styles define `elements` only under a viewport key for a block, the editor now renders them. Expect visual differences in the editor where those styles were previously missing.
- **Hosting / headless / REST:** No impact.
- No action required.

## Technical details

In `packages/global-styles-engine/src/core/render.tsx`, a new helper `getElementStylesByName( styleNode, responsiveMediaQueries )` shallow-copies `styleNode.elements`. For each responsive key in `responsiveMediaQueries`, it then folds `styleNode[breakpointKey].elements` into the matching element entry as `{ ...existing, [breakpointKey]: styles }`. The existing `getResponsiveStyleNodes` can then expand those entries into media-query rules.

`getNodesWithStyles` now iterates over this helper's output instead of raw `.elements` in three places:

- block-level elements (`typedNode`)
- block style variation elements (`typedVariation`)
- inner-block elements within variations (`variationBlockStyles`)

The block-level loop was also reindented. Also removed are the `WordPress dependencies` / `Internal dependencies` comment headers in the touched files.

New tests in `render.test.ts` cover these cases:

- A viewport-only `heading` element, which yields `@media (width <= 480px){:root :where(.wp-block-group h1,...){color: red;}}`.
- A viewport-only `link` `:hover` pseudo state.
- `@mobile` and `@tablet` link styles defined separately, which yield two distinct media queries.

The test mock for `__EXPERIMENTAL_ELEMENTS` gained a `heading` entry. The CHANGELOG notes a bug fix for block element styles defined only inside responsive viewport states.

## Contribution

Opened by @tellthemachines as a follow-up to #81265, with @ramonjd credited by the props bot. The author disclosed that it was written with Codex and reviewed by them. Automated backporting to `wp/7.1` hit a cherry-pick conflict, and @t-hamano backported it manually in #81311.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
