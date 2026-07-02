# #78393: Revisions: Use CSS outline as secondary non-color indicator for diff blocks

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @himanshupathak95
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `Needs Design Feedback`, `[Package] Editor`, `[Feature] History`, `Backported to WP Core`
- **Merged:** [`8f3c483`](https://github.com/WordPress/gutenberg/commit/8f3c483fafdaeb9d37e5555ecbef72b1f15a0065)
- **Discussion:** [#78393](https://github.com/WordPress/gutenberg/pull/78393) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Block Editor's Revisions preview now applies CSS `outline` as a secondary, non-color indicator to distinguish added, removed, and modified blocks. This change improves High Contrast Mode visibility and prevents layout interference or conflicts with user-applied image styles by using semantic outline shapes (solid/dashed/dotted) instead of relying solely on color.

## Impact

- **Editor Users & Site Owners**: Enhanced visual differentiation for revision diffs in High Contrast Mode; no configuration or migration required.
- **Plugin & Theme Developers**: Safe to target `.is-revision-*` classes for canvas styling, noting the introduction of explicit colors (#00a32a, #d63638, #9a7000) and `outline-offset: 2px` across diff states.
- **Platform & Accessibility**: Standardizes non-color indicators to maintain readability without affecting document flow or overlapping rendered content.

## Technical details

The unified diff modifies `packages/editor/src/components/post-revisions-preview/revisions-canvas.js` to inject explicit `outline` rules into the existing revision diff styles:
- `.is-revision-added`: Gains `outline: 3px solid #00a32a; outline-offset: 2px;`
- `.is-revision-removed`: Gains `outline: 3px dashed #d63638; outline-offset: 2px;`
- `.is-revision-modified`: Overwrites previous styling to `outline: 3px dotted #9a7000 !important; outline-offset: 2px;`
These outlines are layered over existing `box-shadow` and SVG filter attributes, ensuring they do not trigger layout reflow or conflict with user-defined image styles.

## Contribution

Open by @himanshupathak95 as part of #77530, this PR implements CSS-based shape indicators for the revision diff canvas to support accessibility standards and prevent layout shifts. Co-authored during review with @t-hamano, @joedolson, and @jasmussen. Merged via commit `8f3c483` and subsequently backported to the WordPress 7.0 minor release by @cbravobernal. Review focused on validating the semantic mapping of solid/dashed/dotted styles to added/removed/modified states.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
