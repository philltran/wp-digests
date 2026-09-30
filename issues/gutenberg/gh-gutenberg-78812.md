# #78812: Icons: Redraw 64 icons to be stroke-based.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`, `[Package] Edit Post`, `[Package] Edit Widgets`, `[Package] Icons`, `[Package] Edit Site`, `[Package] DataViews`, `[Feature] Icons`
- **Merged:** [`bb3752e`](https://github.com/WordPress/gutenberg/commit/bb3752ee7b8201ef96fc23d4fee4b2f566cbcfb0)
- **Discussion:** [#78812](https://github.com/WordPress/gutenberg/pull/78812) · 26 comments · 3 reactions
- **Usefulness:** 3/5

## Summary

Redraws 64 icons in `@wordpress/icons` as stroke-based SVGs (`stroke="currentColor"` plus `vector-effect="non-scaling-stroke"`) instead of filled paths, so stroke width stays constant across 16/24/32px sizes. The batch covers mostly arrow, alignment, justification, chevron, line and utility icons. Because some consumers colored these icons through CSS `fill`, the PR also updates CheckboxControl, ColorPalette, DataViews filters, the welcome guides and Block Manager to color through `color` instead. Most icons are visually identical to trunk; the exceptions are `next` (now an exact mirror of `previous`) and `upload`/`download` (fractional coordinates rounded).

## Impact

**Plugin & theme developers**
- If your CSS or components color `@wordpress/icons` SVGs with `fill` (e.g. `svg { fill: ... }`), stroke-based icons will not pick it up. Set `color` instead; the icons inherit through `currentColor`.
- Snapshot tests that embed these icons' SVG markup will change: `fill="currentColor"` is replaced by `stroke`, `stroke-width="1.5"`, `style="fill: none;"` and `vector-effect` on the paths.
- Third-party block icons (arbitrary SVGs passed to `BlockIcon`) are not converted. Block Manager keeps a `fill` fallback for ones that don't use `currentColor`. Forced-colors rules (`fill: CanvasText` in `block-icon`, `Button`, `Placeholder`, `list-view`) were left in place, and the PR flags them as worth a second look for hardcoded-fill third-party icons.
- Icon-registry contexts: `rss` loses its dot until #75550 lands. The dot is a `<rect>`, and the registry's `wp_kses` allowlist currently permits only `svg`, `path` and `polygon`.

**Site owners / editor users**
- Essentially no visible change; `next`, `upload` and `download` have tiny geometry tweaks. No keyboard or interaction changes.

No migration is required unless you style these icons via `fill`.

## Technical details

Each converted icon now uses `stroke="currentColor"`, `stroke-width="1.5"`, `style="fill: none;"` on the `<svg>`, and `vector-effect="non-scaling-stroke"` on its `<path>`s. For example, `align-left` goes from a filled outline `M13 5.5H4V4h9v1.5Z…` to the stroked `M4 5H13M4 12H20M4 19H13`.

Icons that mix strokes and solid areas (block alignment and vertical alignment controls) combine stroked `<path>`s with a `<rect fill="currentColor" stroke="none">` for the solid block. The `category` icon becomes four stroked rounded-square paths. `check` becomes `M7 12L10 15L17 8`.

Consumer changes visible in the diff and PR description:
- **Block Manager** (`block-manager/style.scss`): `.block-editor-block-icon` now sets both `fill: $gray-900` (kept as fallback for custom icons) and `color: $gray-900`.
- **Block variation picker** (`content.scss`): comment added documenting the retained `fill` fallback.
- **ColorPalette**: the selected checkmark receives the computed contrast color via `color` (snapshot shows `color="#000"` on the `<svg>` instead of `fill="#000"`).
- **CheckboxControl**, DataViews filters, and the Edit Post/Site/Widgets welcome guides: colored via `color` rather than `fill`.
- CHANGELOG entries added under `Unreleased` for `block-editor` and `components` (and others in the truncated portion of the diff). Affected jsdom snapshots regenerated.

The diff is truncated, so the full icon asset list beyond the description was not reviewed. Bundle size shrinks by about 1.5 kB overall per the size bot.

## Contribution

The PR was a sibling to #78808 and part of the broader icon redraw effort in #81274. It sat blocked until #78808 landed because of stroke-related issues found there, then was rebased and marked ready for review, with @jasmussen asking @WordPress/gutenberg-design for a look. During the work he found additional icons affected by code that set `fill` instead of `color`, and split those consumer fixes into their own commit, expecting it to draw the most feedback. @ciampo then pushed updates covering the remaining welcome guides, Block Manager's `fill` fallback for custom block icons, and a ColorPalette checkmark contrast regression. @jasmussen later rebased and adjusted the RSS icon. The author notes AI tools assisted with SVG geometry, CSS audit and rebase work.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
