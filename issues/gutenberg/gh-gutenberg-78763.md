# #78763: Hide Cover overlay controls for viewport states

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Package] Block library`
- **Merged:** [`70d3b6e`](https://github.com/WordPress/gutenberg/commit/70d3b6efd3e7f40a62d5f8c4751c9df299c5d2a9)
- **Discussion:** [#78763](https://github.com/WordPress/gutenberg/pull/78763) · 3 comments · 0 reactions

## Summary

The Cover block's overlay color/gradient controls in the inspector are now hidden whenever a viewport ("style") state is active in the editor. Until the overlay controls are made to work correctly with viewport states, showing them while a state is selected was misleading, so this change gates their rendering on `! hasSelectedStyleState`. This is a stop-gap UI fix that is part of the broader viewport states effort (#77817), not a removal of the controls themselves — they reappear once no state is selected.

## Impact

- **Plugin & theme developers / block users:** When editing a Cover block with a viewport state enabled, the overlay **Color** controls (overlay color + opacity, rendered via the `color` `InspectorControls` group) will no longer appear in the sidebar. They still appear normally when no state is selected. No stored block attributes change and existing Cover blocks render identically on the front end.
- **No action required.** This is an editor-only visibility change with no API, schema, or markup impact. No hooks, filters, or `block.json` fields were altered.
- Affects Gutenberg trunk and was cherry-picked into the `release/23.3` branch.

## Technical details

The change is entirely in `packages/block-library/src/cover/edit/inspector-controls.js`. A new derived flag combines the existing color-availability check with the viewport-state check:

```js
const showOverlayControls =
    colorGradientSettings.hasColorsOrGradients && ! hasSelectedStyleState;
```

The render condition for the overlay `ColorGradientSettingsDropdown` (inside `<InspectorControls group="color">`) was then swapped from the raw color check to the new flag:

```jsx
// Before
{ colorGradientSettings.hasColorsOrGradients && (
    <InspectorControls group="color">
        <ColorGradientSettingsDropdown __experimentalIsRenderedInSidebar … />

// After
{ showOverlayControls && (
    <InspectorControls group="color">
        <ColorGradientSettingsDropdown __experimentalIsRenderedInSidebar … />
```

`hasSelectedStyleState` is the existing signal indicating a viewport/style state is active in the editor. The diff also updates `packages/block-library/src/cover/test/edit.js` to cover the new behavior. Net bundle impact is −10 B on `block-library/index.min.js`.

## Contribution

Opened and authored by **@tellthemachines** (Isabel Brison) as part of the viewport states initiative tracked in #77817, with **@talldan** credited as co-author. It merged in commit `70d3b6e`, and **@cbravobernal** cherry-picked it to `release/23.3` for the next release. The discussion was brief and procedural (size-change report and props bot); the change is explicitly framed as a temporary measure until Cover overlay controls are made compatible with viewport states.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
