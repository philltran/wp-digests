# #82163: Border Box Control: Try moving the unlink button to the label row, and rename the panel to Borders

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Block editor`, `[Feature] Design Tools`
- **Merged:** [`394e4e4`](https://github.com/WordPress/gutenberg/commit/394e4e4c99ad0164151ce8a6ea1da5a87dc70c07)
- **Discussion:** [#82163](https://github.com/WordPress/gutenberg/pull/82163) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The block inspector and Global Styles "Border & Shadow" panel is now always titled "Borders", regardless of which border or shadow controls a block or theme enables. Because the title no longer varies, the "Border" and "Shadow" controls always render their own visible labels. `BorderBoxControl` now places its linked/unlinked toggle in the same row as that label, matching the border radius control's "Unlink sides" toggle. The `useBorderPanelLabel` hook is removed.

## Impact

- **Site owners / editors:** The panel reads "Borders" instead of "Border & Shadow" (or "Shadow" / "Border" depending on availability). The Border and Shadow labels are always visible, and the Border unlink toggle sits in the label row. This is a UI-only change.
- **Plugin & theme developers:**
  - `useBorderPanelLabel` is removed from `packages/block-editor/src/hooks/border.jsx`. It was not re-exported in the visible diff, and the source does not say whether it was public. Code importing it from internal paths will break.
  - `BorderBoxControl`'s visible label is now a `BaseControl.VisualLabel`, rendered as a `span` instead of a `label`. The CHANGELOG notes it was never associated with an input either way. CSS or tests selecting that `label` element may need updating.
  - With a visible `label`, the linked/unlinked toggle now renders in a header row beside the label. Without a visible label it stays beside the inputs as before.
  - E2E tests or docs that look up the panel by the old titles ("Border & Shadow", "Shadow") will need updating.
- **Block fixtures:** The PR description says some block fixtures were regenerated because attributes serialize in a different order (import-order change). It states existing blocks in the wild are not affected.
- **Hosting / headless:** No action required.

## Technical details

**Panel label**
- `useBorderPanelLabel` (which returned "Border & Shadow", "Shadow" or "Border" from `getBlockSettings` / `useHasBorderPanelControls`) is deleted from `hooks/border.jsx`, along with the now-unused `EMPTY_ARRAY`, `unlock` and `__` imports.
- `block-inspector/index.jsx` (`StyleInspectorSlots`, `StyleStateInspectorSlots`) and `inspector-controls-tabs/styles-tab.jsx` now pass `label={ __( 'Borders' ) }` to `InspectorControls.Slot group="border"`. `StyleInspectorSlots` no longer takes a `clientId` prop.
- `global-styles/border-panel.jsx` (`BorderPanel`) passes `label={ __( 'Borders' ) }` to `Wrapper`.

**Control labels**
- In `BorderPanel`, the conditional `BaseControl.VisualLabel as="legend"` wrapping the Border control (shown only when inheritance indicators or a shadow control were present) is removed. `BorderBoxControl` now receives `label={ __( 'Border' ) }`, and the `InheritanceToolsPanelItem` gets `hasInlineEndToggle`.
- The Shadow label is now an unconditional `<BaseControl.VisualLabel>`; the `hasBorderControl ? … : null` guard and the `as="legend"` are gone.

**`BorderBoxControl`** (`packages/components/src/border-box-control/border-box-control/component.tsx`)
- `BorderLabel` renders `BaseControl.VisualLabel` instead of the emotion `StyledLabel`, giving it the stable `.components-base-control__label` class. The hidden-from-vision path still uses `VisuallyHidden as="label"`.
- The component now destructures `hasVisibleLabel` and `headerClassName` from its hook. When `hasVisibleLabel` is true, it renders a two-column `Grid` (`templateColumns="1fr min-content"`, `alignment="center"`) containing the label and the linked button. The diff is truncated beyond this point, so the remaining hook/style changes (where these props are computed) and the no-label branch are not visible.
- The README for `label` documents the new layout.

**Tests:** Adds a `BorderPanel — panel and control labels` suite in `border-panel.jsdom.test.jsx`. It covers the "Borders" heading with and without border controls, the Border label with inheritance indicators off, the Shadow label without border controls, and DOM order of the "Unlink sides" button before the color picker. CHANGELOGs are updated for `block-editor` and `components`.

## Contribution

Andrew Serong opened this as an experiment based on an idea from @jasmussen in issue #73596, explicitly noting he was not wedded to it. The PR was developed with Claude Code, and the author flagged he had not fully reviewed the refactor before the weekend. @ramonjd linked #78830 as possibly related or needing a rollback; the author replied that removing `useBorderPanelLabel` eliminates that class of bug. @jasmussen judged the only risk to be users scanning for "Border & Shadow", considered very low.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
