# #80205: Fix Color Picker Cursor Shaking Issue

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shail-mehta
- **Labels:** `[Type] Bug`, `[Package] Components`, `[Feature] Colors`, `Backport to WP 7.1 Beta/RC`
- **Merged:** [`46048e6`](https://github.com/WordPress/gutenberg/commit/46048e6380e389508a29f06194ae99e57dbcfb7e)
- **Discussion:** [#80205](https://github.com/WordPress/gutenberg/pull/80205) · 12 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a cursor jitter and shaking issue in the Gutenberg `ColorPicker` component when dragging the saturation pointer or adjusting HSL controls, particularly when adding gradient color stops. The change prevents unstable HSLA↔HSVA round-trips and delayed controlled prop echoes from overwriting internal state mid-drag, ensuring smooth pointer movement and accurate saturation preservation at achromatic boundaries.

## Impact

- **Block & plugin developers:** No breaking changes or public API modifications. The `ColorPicker` component's external props (`hsla`, `onChange`, `enableAlpha`) remain unchanged.
- **Theme developers:** No action required.
- **Headless & REST consumers:** No impact.
- **General:** Improves UX for gradient editing and color selection. No migration, configuration, or code changes are required.

## Technical details

The diff refactors `packages/components/src/color-picker/picker.tsx` to replace `HslColorPicker` and `HslaColorPicker` with `HsvColorPicker` and `HsvaColorPicker` from `react-colorful`, maintaining a local `hsva` state to decouple visual rendering from parent HSLA prop sync. It introduces `toHsvaFromHsla()` to preserve native HSVA saturation coordinates when lightness hits 0 or 100, and suppresses parent prop sync via a new `isPickerInteractingRef` flag during pointer/touch drags. In `component.tsx`, `UnconnectedColorPicker` now passes `onInteractionStart` and `onInteractionEnd` callbacks to the picker, and replaces string-based hex comparisons with `safeColordColor.isEqual()` and `colord().isEqual()` to prevent false-positive sync triggers. The component also adds pointer capture via `onPointerDown`, `onPointerUp`, and `onPointerCancel` to maintain drag continuity over iframes.

## Contribution

Opened by @shail-mehta and merged after review by @ciampo, @mirka, and others. The implementation evolved through iterative feedback focusing on the HSLA↔HSVA conversion logic and interaction state management. Initial approaches using direct HSLA sync were replaced with a local HSVA state model and interaction flags to resolve mid-drag race conditions. The fix was approved for backport to WP 7.1 Beta/RC.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
