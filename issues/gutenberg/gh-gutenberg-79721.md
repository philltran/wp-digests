# #79721: UnitControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Components`, `[Package] Block library`, `[Package] Block editor`
- **Merged:** [`00345dc`](https://github.com/WordPress/gutenberg/commit/00345dc93d937c2808d08b576256d74433ce2931)
- **Discussion:** [#79721](https://github.com/WordPress/gutenberg/pull/79721) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Hard deprecates the `__next40pxDefaultSize` boolean prop on `UnitControl`. The larger 40px default sizing is now unconditionally applied, making the previously opt-in flag redundant and slated for complete removal. This cleanup standardizes the component's internal API while preserving existing UI spacing in block editor panels.

## Impact

- **Plugin & theme developers:** Custom blocks or patterns explicitly passing `__next40pxDefaultSize` to `<UnitControl>` must strip the prop from their call sites. Runtime behavior remains identical since the larger size is now default.
- **Platform/monorepo teams:** Requires alignment with WordPress 7.1+ component standards and cleanup of internal `@wordpress/components` consumers.
- *Action:* Remove `__next40pxDefaultSize` (and the legacy `__shouldNotWarnDeprecated36pxSize`) from JSX call sites. No configuration or migration scripts are required.

## Technical details

The core change lives in `packages/components/src/unit-control/index.tsx`, which removes the `useDeprecated36pxDefaultSizeProp` hook and the `maybeWarnDeprecated36pxSize` utility, then destructures and discards `__next40pxDefaultSize`. The component now unconditionally applies the larger default height/spacing without conditional branching or deprecation warnings. Monorepo usages across `packages/block-editor/src/components/*` (e.g., `dimensions-tool/width-height-tool.js`, `height-control/index.js`, `spacer/controls.js`, `font-size-picker/index.tsx`) and `packages/block-library/*` (e.g., `column/edit.js`, `cover/edit/inspector-controls.js`, `search/edit.js`) are stripped of the prop. Examples in `packages/components/src/unit-control/README.md` are updated to omit it entirely.

## Contribution

Merged by @mirka with co-authorship from @ciampo. The PR executes a scheduled hard deprecation after the opt-in period for the larger default size closed. Review and CI discussion focused on verifying flaky test outcomes and finalizing prop cleanup across component stories, internal packages, and documentation.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
