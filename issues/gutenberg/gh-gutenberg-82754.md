# #82754: Stroke icons: final batch, 96 icons

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Enhancement`, `Needs Design Feedback`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`, `[Package] Icons`, `[Feature] Icons`
- **Merged:** [`7e60af5`](https://github.com/WordPress/gutenberg/commit/7e60af52f5e7ec204fcdcd1e1590046d3b3b2f88)
- **Discussion:** [#82754](https://github.com/WordPress/gutenberg/pull/82754) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The final batch of the `@wordpress/icons` stroke conversion redraws 95 remaining fill-based icons as stroke-based and refines the already stroke-based `commentAuthorAvatar`. Icons now use `stroke="currentColor"`, `stroke-width="1.5"`, `style="fill: none"` and `vector-effect="non-scaling-stroke"` paths instead of `fill="currentColor"`. Several drawings change visually: letter-based icons (bold, italic, underline, letter case, h1-h6), HTML/404/word-count (now unboxed), pagination, people icons (hollow heads), lock/unlock, and triangles replaced by uniform chevrons. Six icons intentionally stay fill-based: `insertAfter`, `insertBefore`, `lockSmall`, `offline`, `pinSmall` and `wordpress`.

## Impact

**Plugin & theme developers using `@wordpress/icons`**
- **Breaking (per the package CHANGELOG):** the changelog entry now reads "A further 236 icons are now stroke-based. Use CSS `color` rather than `fill` to recolor them." Any CSS or inline style that recolors these icons via `fill` will no longer work; switch to `color` (or `stroke`).
- Icons that used to be solid glyphs are now outlines. Custom CSS that assumed a filled shape (e.g. `fill` on hover/active states) needs review.
- Icon geometry and optical size changed for some icons, so layouts, pixel-precise UI and visual regression / snapshot tests that embed icon SVG markup or `path` data will need updating.
- Six icons remain fill-based, so recoloring differs between icons in the library: `insertAfter`, `insertBefore`, `lockSmall`, `offline`, `pinSmall`, `wordpress`. The PR author suggests these may be candidates for deprecation, but that is explicitly a separate discussion and not part of this change.

**Site owners / editor users**
- Visible redesign of editor icons: typography icons, lock/unlock, pagination, people icons, HTML/404/word-count. No action required.

**Hosting & headless**
- No action required. The bundle size change is small (about -4.27 kB total in the PR's bundle report).

## Technical details

The diff converts SVG sources under `packages/icons/src/library/*.svg` from the fill convention to the stroke convention introduced in #78808.

Before (`accordion.svg`, filled shapes):
```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor">
  <path fill-rule="evenodd" clip-rule="evenodd" d="M19.5 9.25L9.5 9.25..."/>
  <path d="M4.5 6L8.5 8.5L4.5 11L4.5 6Z"/>
</svg>
```

After:
```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" style="fill: none" stroke="currentColor" stroke-width="1.5">
  <path d="M4.755 10.5L7 8L4.755 5.5M4.755 18.5L7 16L4.755 13.5M10 8H20M10 16H20" vector-effect="non-scaling-stroke" />
</svg>
```

Observed in the (truncated) diff:
- Solid-triangle disclosure markers in `accordion`, `accordion-item` and `accordion-heading` become open chevrons.
- Small solid dots (e.g. in `breadcrumbs`, and the tag icon's dot) are kept as `fill="currentColor" stroke="none"` sub-paths, so a stroke icon can still contain filled elements.
- `brush`, `bug`, `at-symbol`, `border`, `add-submenu` are fully redrawn; `border` is now a set of dashed arc segments.
- `packages/block-editor/src/components/block-lock/style.scss`: `.block-editor-block-lock-modal__lock-icon` changes from `fill: $gray-900` to `color: $gray-900`, since the `lock` icon is now stroke-based.
- Jest snapshots updated for `navigation-link/test/__snapshots__/hooks.jsdom.test.js.snap` and `post-saved-state/test/__snapshots__/index.jsdom.test.jsx.snap`, reflecting the new `<SVG stroke="currentColor" strokeWidth="1.5" style={{fill: "none"}}>` output and `vectorEffect` on paths.
- CHANGELOG updates: `packages/icons/CHANGELOG.md` (breaking-change count raised from 141 to 236, adds `lock` and `post` to the malformed-drawing fixes and notes a restored second bar in `pause`) and `packages/block-editor/CHANGELOG.md` (Block Lock added to the preserved-colors fix).

The diff is truncated, so the full list of 96 icon files is not visible here; the details above are limited to what the diff shows.

## Contribution

Opened by @jasmussen as the closing piece of the stroke-icon conversion (issue #81274, following #78808 and #82540). @ciampo helped with fixes (border, swatch, pause), and @t-hamano is credited in the props list. @jasmussen initially asked that it not be merged until a visual review could land, because of the deliberate visual changes (especially the letter-based icons), then merged it after checking in with a few reviewers and letting it sit. The decision on whether to deprecate the six remaining fill-based icons was deferred to a separate discussion, and follow-up PRs with new icons and optical rebalancing are planned based on feedback in the make/core icons post.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
