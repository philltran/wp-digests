# #79698: Icons: Add repeat all icon

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @scruffian
- **Labels:** `[Type] Enhancement`, `[Package] Icons`
- **Merged:** [`9f8a103`](https://github.com/WordPress/gutenberg/commit/9f8a103178013d9b5036f5935aa4f70bce8f29f7)
- **Discussion:** [#79698](https://github.com/WordPress/gutenberg/pull/79698) · 1 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds a dedicated `repeat-all` icon to the `@wordpress/icons` package and registers it in the icon manifest. This provides a standardized SVG asset for developers implementing playlist repeat controls, eliminating the need to repurpose existing icons or maintain custom assets.

## Impact

- Plugin & block developers: Can immediately consume `icons('repeat-all')` or `<Icon icon="repeat-all" />` in UI components.
- No breaking changes, deprecations, or migration steps.
- Developers must update to a WordPress/Gutenberg release that includes this merge to access the asset.

## Technical details

The diff introduces `packages/icons/src/library/repeat-all.svg` (a 24x24 viewBox path) and appends a new entry to `packages/icons/src/manifest.json`. The manifest registration maps `slug: "repeat-all"` to the SVG file, which triggers the standard build pipeline to auto-generate module exports. In the modern `@wordpress/icons` collection, adding a manifest entry ensures the asset is statically analyzed, bundled, and exposed via the `icons()` registry without requiring manual React component wrappers.

## Contribution

Opened by @scruffian to address an unmet requirement for playlist controls. Merged with co-authorship from @t-hamano after passing standard collection validation (`validate-collection.cjs`) and package build checks. The change followed existing icon contribution workflows with no design reviews or alternative approaches proposed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
