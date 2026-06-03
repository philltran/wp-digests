# #78281: Grid: visualize columns without outline

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @simison
- **Labels:** `[Package] Block editor`, `[Type] Experimental`, `[Package] Grid`
- **Merged:** [`8c11289`](https://github.com/WordPress/gutenberg/commit/8c11289526ef807331bea68fc18409ef7283573d)
- **Discussion:** [#78281](https://github.com/WordPress/gutenberg/pull/78281) · 7 comments · 0 reactions

## Summary

The Block Editor's experimental Grid package now visualizes column boundaries using a solid background fill rather than dashed outlines. This resolves rendering inconsistencies where top/bottom lines appeared doubled and horizontal/vertical dashed gaps had mismatched sizes.

## Impact

- **Block Editor Developers**: Column visualization styling has shifted from `outline` to `background-color`. No runtime code changes are required.
- **Plugin & Theme Developers**: External stylesheets or overrides targeting `.is-grid-container` outlines may need review if they relied on the previous dashed-line rendering.
- **Site Owners / Platform Teams**: No action required.

## Technical details

- Replaces the CSS `outline` property used for grid column dividers with a solid background fill, increasing the alpha from 10% to 20%. 
- Addresses the limitation where `border-dash-gap` cannot be uniformly controlled across horizontal and vertical axes using standard outlines.
- The change is scoped to the experimental Grid component in the block editor, affecting only the visual feedback during layout manipulation.

## Contribution

- Merged by @simison as a follow-up to PR #78199, addressing UI refinements for the experimental Grid package.
- Co-authored and reviewed by @retrofox and @jameskoster, who weighed in on swapping dashed outlines for solid lines to improve visual consistency.
- Merged into `trunk` following standard experimental package review cycles.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
