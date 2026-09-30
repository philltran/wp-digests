# #81840: Media Editor: refactor the panel layout

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Feature] Media`, `[Package] Base styles`
- **Merged:** [`b200d6d`](https://github.com/WordPress/gutenberg/commit/b200d6d22d5fca41c31a2e9fdeb9b2bfcffba1ec)
- **Discussion:** [#81840](https://github.com/WordPress/gutenberg/pull/81840) · 19 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg media editor modal now uses a layout it owns instead of `ComplementaryArea` and `InterfaceSkeleton` from `@wordpress/interface`. This fixes a bug where the Details fields could not be reached on phones, because `ComplementaryArea` force-closed the sidebar below the `small` breakpoint and gave no way to reopen it. The settings panel is also wider (280px to 320px, `--wpds-dimension-surface-width-sm`). Rotate, flip and zoom move out of the modal footer into the Crop panel, or into a row below the canvas where the panel is full-screen.

## Impact

**Plugin and theme developers using `@wordpress/media-editor`**
- The package is still in an unreleased cycle, and the CHANGELOG notes the removed APIs were added in that same cycle. Anyone consuming pre-release builds needs to adjust.
- `MediaEditor.ImageControls` is removed. The transform controls now live in the Crop panel, or below the canvas on narrow viewports.
- The `scope` prop on `MediaEditor` is removed. It named an `@wordpress/interface` scope that persisted the sidebar's open state, and that persistence went away with `ComplementaryArea`.
- `renderFrame` no longer receives `hasCanvas`. `HistoryActions` already renders nothing when a panel covers the canvas.
- `layout` is still passed to `renderFrame` and is now documented as `narrow` below the `small` breakpoint, so a frame can decide where the history cluster goes.
- `MediaEditorCropPanel` drops the `showTransformControls` prop and always renders the image controls.
- The `@wordpress/media-editor` package no longer depends on `@wordpress/interface` and now depends on `@wordpress/admin-ui`.

**Site owners and end users**
- Details fields are editable on phones through the header toggle.
- The sidebar is 40px wider.

**Styling and base-styles**
- The `.media-editor-modal__footer` z-index token is removed from `packages/base-styles/_z-index.scss`. Any code calling `z-index( ".media-editor-modal__footer" )` would no longer resolve.
- The footer's `is-wide` and `is-narrow` classes and the `.media-editor-modal__footer-row` class are gone.

## Technical details

**Layout replacement**
- `packages/media-editor/src/components/media-editor/index.tsx`: `MediaEditorSidebar` no longer wraps `ComplementaryArea`. It renders an `@wordpress/admin-ui` `NavigableRegion` (`ariaLabel` "Media settings") containing a visually hidden `<h2>` and a `Tabs.Root` / `Tabs.List` / `Tabs.Panel` set from `@wordpress/ui`.
- A `media-editor__tablist` wrapper draws the rule under the tab strip, because `Tabs.List` sizes itself with `fit-content`.
- The sidebar now tracks which panel is open through constants `DETAILS_PANEL` and `CROP_PANEL`, instead of tracking whether a sidebar is open. The tab type is renamed from `EditorTab` to `MediaEditorTab`, and `panel: JSX.Element` becomes `render: () => JSX.Element`.
- There is no in-panel close button. The header's pressed toggle dismisses the panel.

**Transform controls**
- `MediaEditorCropPanel` always renders `<MediaEditorImageControls withLabels />` above the aspect-ratio `SelectControl`.
- Below `small`, the controls sit in a row under the canvas.

**Footer**
- `ModalFooter` in `media-editor-modal/index.tsx` no longer takes a `layout` prop. It has a single tree: `HistoryActions` and `SaveActions`, with `padding: $grid-unit-30` and `margin-left: auto` on the save actions.

**Frame contract**
- `MediaEditorFrameProps.layout` remains `'wide' | 'narrow'` and is documented in the diff as the signal for placing the history cluster.
- The discussion shows this was restored after review: the first revision hard-coded history actions in the header, which regressed the editor route's narrow layout, where trunk puts them in the footer.

**Tests and build**
- Crop panel tests drop the `showTransformControls` cases and assert that rotate, flip and zoom always render above the aspect ratio selector.
- `package.json` and `package-lock.json` swap `@wordpress/interface` for `@wordpress/admin-ui`.
- The bot reports a -415 B net size change, with editor CSS shrinking and `build/scripts/editor/index.min.js` growing by 145 B.

The diff is truncated, so the docking breakpoint logic, the full-screen route changes and the remaining stylesheet changes are not visible here. The PR description is the source for the 600px docking behavior.

## Contribution

@andrewserong reviewed and tested the change. He flagged that an early revision hard-coded the history actions into the header, regressing the editor route's narrow layout where trunk puts them in the footer, which meant `layout` was still needed. @ramonjd fixed that. Sidebar width was also discussed: an interim 400px was considered too wide on laptop screens, 350px (the list view sidebar width) was tried, and the merged value is the 320px design token. The PR closes #81487 and addresses #81121, where the DataViews column felt cramped at 280px.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
