# #82023: PaletteEdit: Expose palette swatches as command buttons

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] Components`, `[Package] Block library`, `has dev note`
- **Merged:** [`b2732aa`](https://github.com/WordPress/gutenberg/commit/b2732aa830da11dcd7638ec6d479ac56fcf9ec52)
- **Discussion:** [#82023](https://github.com/WordPress/gutenberg/pull/82023) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

`PaletteEdit` swatches (color, gradient, duotone) are now exposed as plain command buttons instead of listbox options or toggle buttons, because activating them opens an editor rather than selecting a value. To support this, `CircularOptionPicker`, `ColorPalette`, `GradientPicker`, and `DuotonePicker` gain a `presentation` prop (`listbox` | `toggle-buttons` | `command-buttons`). The existing `asButtons` prop is deprecated in favor of `presentation="toggle-buttons"`.

## Impact

**Plugin & theme developers using `@wordpress/components`**
- `asButtons` on `ColorPalette`, `GradientPicker`, `DuotonePicker`, and `CircularOptionPicker` is deprecated (since 7.2). It still works as an alias for `presentation="toggle-buttons"` and logs a deprecation warning. Replace it with `presentation="toggle-buttons"`.
- If both props are passed, `presentation` wins.
- Use `presentation="command-buttons"` when swatches trigger actions (e.g. open an editor). Each swatch gets its own tab stop and exposes neither `aria-pressed` nor `aria-selected`, and no selected check icon is rendered.
- The `loop` prop only applies to the `listbox` presentation.
- In `command-buttons` mode, activating a predefined swatch always calls `onChange`, even if `value`/`selectedSlug` already matches it.
- Tests that assert on the deprecation console output or on `PaletteEdit` swatch roles may need updating.

**Site owners / editors**
- No visual change. Screen reader and keyboard users get button semantics in the Global Styles palette editor: Tab moves between swatches, and Enter/Space opens the matching editor.

**Hosting / headless / REST**
- Not affected.

## Technical details

Diff highlights (the diff was truncated, so `utils.tsx` and the `ColorPalette`/`GradientPicker`/`DuotonePicker`/`PaletteEdit` changes are not visible; only the `CircularOptionPicker` layer and the Cover block are confirmed):

- `circular-option-picker/types.ts`: `CircularOptionPickerProps` is now a single type, replacing the previous discriminated union on `asButtons`. It adds `presentation?: 'listbox' | 'toggle-buttons' | 'command-buttons'` (default `'listbox'`), keeps `asButtons?: boolean` marked `@deprecated`, and moves `loop` into the shared props. `ButtonsCircularOptionPickerProps` requires a non-listbox `presentation`. `CircularOptionPickerContextProps` gains `presentation`.
- `circular-option-picker.tsx`: calls `warnIfCircularOptionPickerAsButtonsIsSet( 'CircularOptionPicker', asButtons )` and `resolveCircularOptionPickerPresentation( presentation, asButtons )`, both new helpers in `utils.tsx` and re-exported from `index.tsx`. It renders `ListboxCircularOptionPicker` for `listbox`, otherwise `ButtonsCircularOptionPicker`, passing `presentation` through context. The listbox context now includes `presentation: 'listbox'`.
- `circular-option-picker-option.tsx`: `Option` now branches on context `presentation` rather than on `setActiveId !== undefined`. The default is `'toggle-buttons'` when rendered without a parent picker, preserving the old standalone behavior. `isPressed` is `!!isSelected` for `toggle-buttons` and `undefined` for `command-buttons`. The selected check `Icon` is not rendered for `command-buttons`.
- The deprecation message reads: ``asButtons` prop in wp.components.CircularOptionPicker is deprecated since version 7.2. Please use `presentation` instead.``
- `block-library/src/cover/edit/index.js`: the overlay `ColorPalette` swaps `asButtons` for `presentation="toggle-buttons"`.

```jsx
// Before
<ColorPalette asButtons ... />
// After
<ColorPalette presentation="toggle-buttons" ... />
```

The story `AsButtons` was renamed `AsToggleButtons`, and the README and CHANGELOGs were updated. Tests cover each presentation, the standalone `Option` fallback, and `presentation` taking precedence over `asButtons`.

## Contribution

This PR supersedes #81836 by @dilipom13, who is credited as a virtual co-author. @ciampo carried over the feedback rounds from that PR and merged without a formal approval after positive feedback from @ramonjd. The description notes that OpenAI Codex was used to implement the change and write tests and docs. The PR carries the `has dev note` label, with a WP 7.2 dev note included in the description.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
