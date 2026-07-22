# #77279: Block Supports: Relocate text and bg color controls to Typography and Background panels

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Package] E2E Tests`, `[Feature] Design Tools`
- **Merged:** [`d46102b`](https://github.com/WordPress/gutenberg/commit/d46102bc80a063e2078979e417ecd3ec0b89ace4)
- **Discussion:** [#77279](https://github.com/WordPress/gutenberg/pull/77279) · 71 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The block editor has reorganized the block inspector and Global Styles sidebars to group color controls by their functional context instead of keeping them in a single catch-all "Color" panel. The root-level text color control (`color.text`) now lives in the Typography panel, while background color and gradient controls (`color.background`, `color.gradient`) moved to the Background panel. A new "Elements" panel was introduced in the block inspector to house element-specific color controls (links, headings, buttons, captions). This change improves UI consistency and aligns control placement with the style properties they modify, without altering the underlying storage paths or block support flags.

## Impact

- **Plugin & theme developers:** No breaking changes to block support flags or storage paths. Existing blocks continue to read/write `style.color.text`, `style.color.background`, and `style.color.gradient` unchanged.
- **Plugin developers using inspector slots:** The `group="color"` slot no longer contains text or background controls. If you previously slotted custom controls into `group="color"` expecting them to appear alongside root color pickers, they will now appear in the Elements panel or require a new group assignment.
- **Site owners & editors:** The block inspector and Global Styles UI will look different. Text color is now under Typography, background color/gradient under Background, and element colors under a new Elements panel. No migration required.
- **No action required** for standard block/theme development or site configuration.

## Technical details

The diff restructures `packages/block-editor/src/components/block-inspector/index.js` to reorder `InspectorControls.Slot` calls, introducing `group="typography"` and `group="elements"` while moving `group="background"` up. The legacy `group="color"` slot now only renders element-level controls. Storage paths remain untouched: `color.text` writes to `style.color.text`, and `color.background`/`color.gradient` write to `style.color.background` and `style.color.gradient` respectively.

The PR extracts shared logic into `ColorGradientDropdownItem`, `useColorGradientSettings`, and `getContrastWarning()` in `packages/block-editor/src/components/contrast-checker/index.js`. The `ContrastChecker` component gains a `messageOverride` prop for panel-specific warning copy. In `packages/block-editor/src/components/colors-gradients/control.js`, the contrast notice is now rendered inside the color popover via `noticeProps` to avoid remounting the `ColorPalette`. CSS is consolidated under a new `.block-editor-color-gradient-item` class in `style.scss`, replacing panel-specific selectors. Panel gating was updated so blocks with only `color.text` or `color.background` support skip the Color/Elements panels entirely, routing controls to Typography/Background instead.

Before/after slot ordering in `block-inspector/index.js`:
```diff
- <InspectorControls.Slot group="color" label={ __( 'Color' ) } />
- <InspectorControls.Slot group="background" label={ __( 'Background image' ) } />
- <InspectorControls.Slot group="typography" label={ __( 'Typography' ) } />
+ <InspectorControls.Slot group="typography" label={ __( 'Typography' ) } />
+ <InspectorControls.Slot group="color" label={ __( 'Color' ) } />
+ <InspectorControls.Slot group="background" label={ __( 'Background' ) } />
+ <InspectorControls.Slot group="elements" label={ __( 'Elements' ) } />
```

## Contribution

Opened by @aaronrobertshaw as an exploration of interim UI reorganization, the PR evolved through feedback from @jameskoster and @ramonjd. Early iterations debated whether to keep the legacy "Color" panel visible or hide it when empty; the final approach hides it when only text/background controls are present and introduces a dedicated "Elements" panel pushed lower in the sidebar. @jameskoster suggested moving the contrast warning into the color popover for better context, which was implemented before merge. The PR consolidated shared color/gradient UI logic and updated panel gating to prevent empty panels from rendering.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
