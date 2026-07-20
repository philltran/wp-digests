# #80339: Responsive styles: Use viewport dropdown to control states for in-editor global styles sidebar

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Style States`
- **Merged:** [`c233133`](https://github.com/WordPress/gutenberg/commit/c233133c44c628a65ae47480696e1d234cb754bc)
- **Discussion:** [#80339](https://github.com/WordPress/gutenberg/pull/80339) · 7 comments · 3 reactions
- **Usefulness:** 3/5

## Summary

The in-editor global styles sidebar now delegates responsive viewport state control to the main editor viewport dropdown, removing a redundant dropdown from the sidebar itself. This change also fixes a UI state bug where enabling responsive editing would unexpectedly switch the active sidebar to the block inspector, even when the global styles sidebar was already open. The update unifies responsive editing behavior across the editor and prevents conflicting UI states.

## Impact

- **Theme & plugin developers:** No public API changes or breaking changes. Internal editor components now accept `showResponsiveStateControls` and `selectedViewport` props, but these are not exposed for external consumption.
- **Site builders & editors:** The global styles sidebar no longer shows its own viewport dropdown; responsive states are controlled exclusively via the top-bar viewport dropdown. Toggling responsive editing while the global styles sidebar is open will no longer force a switch to the block inspector.
- **No action required** for existing codebases or custom block/theme development.

## Technical details

The diff modifies `packages/editor/src/components/global-styles-sidebar/index.js` to fetch the current viewport state via `getStyleStateViewport()` from `@wordpress/block-editor` and pass it as `selectedViewport` to `GlobalStylesUIWrapper`, while explicitly setting `showResponsiveStateControls={ false }`. This prop chain flows through `GlobalStylesUI`, `ContextScreens`, `BlockStylesNavigationScreens`, and `ScreenBlock` (`packages/global-styles-ui/src/...`). In `ScreenBlock`, the component now accepts `controlledSelectedViewport` and falls back to local state only when not controlled: `const selectedViewport = controlledSelectedViewport ?? localSelectedViewport;`. When `showResponsiveStateControls` is `false`, `onChangeViewport` is omitted and `viewportStates` is passed as an empty array to `StateControl`, effectively hiding the sidebar’s responsive controls. Additionally, `packages/editor/src/components/preview-dropdown/index.js` now checks `activeComplementaryArea` before calling `enableComplementaryArea( 'core', sidebars.block )`, preventing the block inspector from auto-opening when responsive editing is toggled and a complementary area is already active.

## Contribution

Opened by @talldan and merged as `c233133`, with co-authors @andrewserong, @ramonjd, @tellthemachines, and @himanshupathak95. During review, @ramonjd identified that toggling responsive editing would unexpectedly switch focus to the block inspector. @talldan addressed this by adding an `activeComplementaryArea` guard in `preview-dropdown/index.js`. @tellthemachines noted a related UI cleanup (hiding style variations when a viewport is active) which was deferred to a separate PR (#80341). The change was backported to the Gutenberg RC and WordPress Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
