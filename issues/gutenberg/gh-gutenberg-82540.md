# #82540: Stroke icons: 3rd batch, 142 icon updates

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`, `[Package] Icons`, `[Feature] Icons`
- **Merged:** [`6867b78`](https://github.com/WordPress/gutenberg/commit/6867b78dd1ef013b402bb38419787ce1a3bcdd14)
- **Discussion:** [#82540](https://github.com/WordPress/gutenberg/pull/82540) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Third batch of the ongoing stroke-icon conversion in `@wordpress/icons`: 141 fill-based icons are redrawn as stroke-based SVGs and the already stroke-based `image` icon is refined. The new SVGs use `style="fill: none" stroke="currentColor" stroke-width="1.5"` with `vector-effect="non-scaling-stroke"` on paths. Consumers that recolored these icons via CSS `fill` must now use `color`. The PR also fixes the malformed `formatIndent`, `formatIndentRTL`, `formatOutdent` and `formatOutdentRTL` geometry and adds validation so fill-only drawings can't be flagged as stroke-based.

## Impact

**Plugin & theme developers**
- **Breaking (per the `@wordpress/icons` changelog):** the 141 converted icons are stroke-based, so `fill: <color>` in CSS no longer recolors them. Use CSS `color` instead (the stroke resolves via `currentColor`). Custom CSS targeting `svg { fill: ... }` on these icons will stop working and may render them unfilled.
- The table icons are 1px smaller for consistency with other square icons; `capturePhoto` and `image` are also slightly smaller. Most other icons keep their original footprint. Visual diffs or snapshot tests that include these icons will change.
- Anything that snapshots icon markup (as the updated `BlockIcon`, `MenuItem` and navigation-link variation snapshots did) will need regenerating.
- Stroke icons can now respond to variable stroke width in the same way as the earlier batches.

**Gutenberg internals / site owners**
- Core consumers were updated: `.block-editor-patterns__pattern-icon` (synced color), block-visibility `-icon--checked`, and the document-bar template icon now use `color` rather than `fill`.
- A follow-up comment from @ramonjd suggests this PR may be behind a regression in icon stroke rendering for block controls; a separate fix PR (#82940) was opened but not confirmed correct.
- No action required for site owners.

## Technical details

**Icon sources (`packages/icons/src/library/*.svg`)**: Icons change from `fill="currentColor"` on the root to the stroke convention introduced in #78808:

```svg
<!-- before -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor">
  <path fill-rule="evenodd" d="..." />
</svg>

<!-- after -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" style="fill: none" stroke="currentColor" stroke-width="1.5">
  <path d="..." vector-effect="non-scaling-stroke" />
</svg>
```

Some icons (e.g. `audio`) keep a small solid element via `fill="currentColor" stroke="none"` alongside the stroked path. Per the PR discussion, `copySmall` and `sites` were converted to real non-scaling strokes, and `cart` and `mapMarker` were kept within the public SVG sanitizer's supported elements.

**Consumer CSS**: `fill:` becomes `color:` in `block-patterns-list/style.scss` (`--wp-block-synced-color`), `block-visibility/style.scss` (`$gray-300`), and `editor/.../document-bar/style.scss` (`$gray-600`).

**Validation (`packages/icons/lib/validate-collection.cjs`)**: Adds and exports `isStrokeBasedSvg(svgContent)`, which matches the root `<svg>` tag and tests for `style="fill: none"` (tolerating whitespace, optional semicolon and either quote style). Fill-none on child elements is ignored. `validateCollection()` now, for stroke-based SVGs, requires at least one graphical element (`circle|ellipse|line|path|polygon|polyline|rect`) that does not set `stroke="none"`, and requires every such stroked element to set `vector-effect="non-scaling-stroke"`. New test: `packages/icons/lib/test/validate-collection.js`.

**Snapshots**: Updated for `BlockIcon`, `MenuItem`, and navigation-link `enhanceNavigationLinkVariations`; the icon now renders `stroke="currentColor"`, `strokeWidth="1.5"`, `style={{fill: "none"}}` and `vectorEffect="non-scaling-stroke"` on paths.

**Changelogs**: `icons` (Breaking Changes, Enhancements, Bug Fixes), `block-editor`, and `editor` entries. No PHP, REST, or hook changes. Note: the diff shown is truncated, so the full list of 142 icon files was not visible.

## Contribution

@jasmussen authored the batch, using Claude Opus for batch normalization and Codex for rebase and review fixes. @ciampo pushed compatibility fixes (`fill` to `color` in consumers, real strokes for `copySmall`/`sites`, sanitizer-safe `cart`/`mapMarker`, the validation and tests, a consolidated changelog, and removal of an unrelated Calendar change). @mirka flagged duplicate paths and zero-area segments in the `cog` icon, which was re-exported and fixed. @jasmussen explained the 1px shrink on `image` as aligning all square-based icons to shared base shapes (tied to #65786). After merge, @ramonjd suspected this PR caused a stroke regression in block controls and linked a separate agent-generated fix (#82940); two more batches (remaining icons, plus updates and new icons) were planned.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
