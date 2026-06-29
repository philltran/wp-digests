# #75121: Integrate Resizable Editor with Device Preview and add Responsive editing

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `General Interface`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`, `[Package] Edit Post`
- **Merged:** [`9812cbe`](https://github.com/WordPress/gutenberg/commit/9812cbed4f23c32469e125e7e587ab86ae8aa6a2)
- **Discussion:** [#75121](https://github.com/WordPress/gutenberg/pull/75121) · 73 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This PR consolidates the block editor's device preview and resizable canvas into a single source of truth based on pixel width, enabling true responsive editing across post, template, pattern, and navigation editors. It deprecates the legacy `useResizeCanvas` hook (now reduced to a no-op) and shifts viewport/style state management from per-block to globally shared via new `core/block-editor` store actions and selectors. The device preview dropdown now acts as the primary control for setting canvas width and toggling responsive editing context, driving which breakpoint block inspector style changes apply to.

## Impact

- **Plugin & theme developers**: `useResizeCanvas()` from `@wordpress/block-editor` is deprecated (since v7.1) and returns an empty object. Developers relying on it for canvas sizing in custom panels or isolated editors must switch to the unified resizable editor or manage width manually.
- **Block editor package developers**: Internal private APIs changed: `onViewportStateChangeKey` was removed; viewport state is now global (`getStyleStateViewport`/`setStyleStateViewport`) rather than per-block. The `getSelectedBlockStyleState` selector now injects the global viewport into its return object, altering how consumers derive viewport-dependent styles.
- **Site owners / editor users**: No action required. The device preview dropdown automatically drives responsive editing mode, and style edits apply to the selected breakpoint without requiring manual per-block state toggles.

## Technical details

- **File**: `packages/block-editor/src/components/use-resize-canvas/index.js`
  - `useResizeCanvas()` is explicitly deprecated with hint `'Device preview is now handled by the editor canvas. This hook no longer does anything.'`. The implementation was reduced to a no-op and the accompanying README was deleted.
- **File**: `packages/block-editor/src/store/private-selectors.js` & `private-actions.js`
  - Added `getStyleStateViewport(state)` / `setStyleStateViewport(viewport)` to track the active breakpoint globally via `state.styleStateViewport`.
  - Rewrote `getSelectedBlockStyleState(state, clientId)` as a `createSelector` that merges `state.styleStateViewport` with per-block state, ensuring consumers receive a consistent viewport context without manual merging.
- **File**: `packages/block-editor/src/hooks/states.js`
  - Removed `getViewportStateOptions()` and the `viewportStates` prop from `BlockStatesControl`; viewport selection is now exclusively handled by the device preview UI rather than per-block controls.
- **File**: `packages/block-editor/src/components/block-inspector/index.js` & `BlockInspectorSingleBlock`
  - Replaced the `onViewportStateChange` callback with an `isResponsiveEditing` boolean derived from store state. Inspector rendering logic now uses `isEditingStyleState` (active block style selection OR responsive editing mode) to conditionally show style badges and toggles.
- **Diff-grounded behavior**: The unified diff confirms the architectural shift from discrete `deviceType` strings to a pixel-based `canvasWidth` flow, with the device dropdown driving that width and globally injecting viewport context into style state rendering.

## Contribution

Opened by @t-hamano to resolve #71210 and address follow-ups in #73735. Mid-review, the scope shifted from simply integrating Device Preview with Resizable Editor to restructuring how viewport state is tracked globally. Discussions with @Mamaduka and @tellthemachines clarified why a new `canvasWidth` source of truth was necessary over patching the legacy system: preserving existing `getDeviceType`/`setDeviceType` public APIs while fixing an edge case where `getDeviceType` incorrectly returned `'Desktop'` in resizable editors, breaking 'Hide on Tablet' visibility logic. The PR was merged with co-authorship from @Mamaduka, @youknowriad, @ramonjd, and others after aligning on the device dropdown's role as the primary responsive editing trigger.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
