# #79071: Grid: Prepare `@wordpress/grid` for npm publishing as experimental 0.1.0

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `npm Packages`, `[Package] Grid`
- **Merged:** [`46ccbaf`](https://github.com/WordPress/gutenberg/commit/46ccbaf69cb9495d158d7d09ff317622f0032a61)
- **Discussion:** [#79071](https://github.com/WordPress/gutenberg/pull/79071) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/grid` package is now published to npm as an experimental `0.1.0` release, making its React layout engine available to external projects outside the Gutenberg monorepo. Previously excluded via the `private` flag, the package ships two components—`DashboardGrid` and `DashboardLanes`—that handle drag-to-reorder, resize, and masonry-style layouts. This enables dashboard-style surfaces in custom WordPress admin screens, block themes, or headless setups without relying on internal monorepo paths.

## Impact

- **Plugin & theme developers:** Can now install `@wordpress/grid` via npm to build interactive dashboard layouts. No longer requires pulling from the Gutenberg monorepo or using internal paths.
- **Headless & platform teams:** The package is pure JS/React with runtime-injected CSS Modules, making it straightforward to integrate into external React apps or custom admin UIs.
- **Configuration/Migration:** Consumers outside standard editor screens must install and import `@wordpress/theme/design-tokens.css` (or define the `--wpds-*` CSS custom properties) to get correct visual defaults for gaps, elevations, and motion.
- **No action required** for existing Gutenberg core development workflows.

## Technical details

The diff removes `"private": true` from `packages/grid/package.json` and sets `"version": "0.1.0-prerelease"`, ensuring the automated release workflow publishes `0.1.0` instead of `1.0.0`. `packages/grid/CHANGELOG.md` is collapsed into a single initial-release entry under `### New Features`, which the release tooling reads to calculate version bumps. `packages/grid/README.md` receives an experimental callout, `Installation`/`Setup` sections, and documentation for the previously missing `renderGridOverlay` render prop and `--wp-grid-overlay-tile-bg` CSS custom property. `docs/manifest.json` is updated to index the package slug. The package exports `DashboardGrid`, `DashboardLanes`, and associated types including `DashboardGridLayoutItem`, `DashboardGridProps`, `DragPreviewRenderProps`, and `GridOverlayRenderProps`.

## Contribution

The change was authored by @retrofox with co-authorship from @chihsuan, focusing on unblocking npm publication by stripping the private flag, adjusting the prerelease version string to align with semver expectations for experimental packages, and finalizing the documentation and changelog for public consumption. The discussion record contains only automated CI reports and size checks, with no documented design debates or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
