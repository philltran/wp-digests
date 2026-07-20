# #80094: Icons: Add "sites" icon.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Enhancement`, `Needs Design Feedback`, `[Package] Icons`
- **Merged:** [`c359466`](https://github.com/WordPress/gutenberg/commit/c35946615fea36ad93077eb058fcc88c4d8baf96)
- **Discussion:** [#80094](https://github.com/WordPress/gutenberg/pull/80094) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/icons` package now ships a new `sites` icon to fill a gap in the library. This addition provides a standardized visual asset for representing multisite or networked site interfaces in the block editor and admin UI.

## Impact

- **Site owners**: No action required.
- **Plugin & theme developers**: No immediate action required. The icon is now available in the standard registry for use in `block.json` or React components.
- **Hosting & platform teams**: No impact.
- **Headless & REST consumers**: No impact.

## Technical details

The diff adds `packages/icons/src/library/sites.svg`, a 24x24 SVG containing three overlapping window paths. It registers the asset in `packages/icons/src/manifest.json` with the slug `sites`, label `Sites`, and `filePath` `library/sites.svg`. The `@wordpress/icons` package automatically exposes this to the icon registry, making it available via the standard `Icon` component without additional configuration or build steps.

## Contribution

Opened by @jasmussen to address a missing icon in the library. The design was iterated on with feedback from @t-hamano and @fushar, who are listed as co-authors. The PR was merged after passing checks, with the `packages/icons/CHANGELOG.md` updated to document the addition.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
