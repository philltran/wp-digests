# #81054: Editor: Migrate settings sidebar to Tabs from @wordpress/ui

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] Interface`
- **Merged:** [`97f694c`](https://github.com/WordPress/gutenberg/commit/97f694c472d621eb5df7ff13f4589bbe1210f818)
- **Discussion:** [#81054](https://github.com/WordPress/gutenberg/pull/81054) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The editor settings sidebar now uses `Tabs` from `@wordpress/ui` instead of the private `Tabs` in `@wordpress/components`. This migration resolves a React tree boundary issue where tab state and ARIA linkage were split across a non-bubbling `Slot`/`Fill`, and aligns the sidebar with modern `@wordpress/ui` conventions.

## Impact

- **Plugin & theme developers:** No breaking changes. The `ComplementaryArea` component in `@wordpress/interface` now accepts an optional `render` prop for custom container elements, which could be useful for advanced sidebar integrations.
- **Internal/Editor team:** Removes reliance on `@wordpress/components` private APIs and `Tabs.Context` forwarding, simplifying the sidebar's React tree and improving keyboard/ARIA behavior.
- No action required for standard plugin/theme developers.

## Technical details

The `render` prop was added to `ComplementaryArea` and `ComplementaryAreaFill` in `packages/interface/src/components/complementary-area/index.js`. A new `renderContainer()` helper uses `cloneElement` to merge `className` and `style` props, replacing the default `<div>` scroll container. `packages/editor/src/components/sidebar/index.js` removes `Tabs.Context.Provider` workarounds and passes `render={ <Tabs.Root value={ tabName } onValueChange={ onTabSelect } /> }` to keep the tab list and panels in a single React subtree. `packages/editor/src/components/sidebar/header.js` updates to `@wordpress/ui`'s `Tabs.List` and `Tabs.Tab`, switching from `tabId` to `value` and adding `activateOnFocus={ false }`. ESLint suppressions for `@wordpress/use-recommended-components` are removed from the sidebar files.

## Contribution

Opened as an alternative to #81006, the PR was authored by @Mamaduka with co-authorship from @mirka. The discussion was minimal, focusing on the technical necessity of avoiding `Tabs.Context` forwarding due to Base UI's architecture. The approach was merged directly after passing e2e tests for sidebar navigation, block hierarchy, and plugin APIs.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
