# #80435: Backport ColorPicker cursor shaking fix to WordPress 7.1

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] Components`, `Backport to Gutenberg RC`
- **Merged:** [`f58512d`](https://github.com/WordPress/gutenberg/commit/f58512d348fc7abbdc81a6da11abe51e776ba0da)
- **Discussion:** [#80435](https://github.com/WordPress/gutenberg/pull/80435) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `ColorPicker` component in `@wordpress/components` no longer jitters or shakes its visual picker cursor during drag interactions or when syncing controlled HSLA values. The fix switches the underlying `react-colorful` surface from HSLA to HSVA, introduces local state to decouple the visual pointer from parent prop updates during gestures, and refines color equality checks to prevent lossy round-trips from overwriting internal state mid-drag.

## Impact

- **Block & plugin developers:** No breaking changes. The `ColorPicker` API remains identical, but controlled usage patterns that previously caused UI jitter (especially with gradients or HSL inputs) will now behave smoothly.
- **Theme developers:** No direct impact; `ColorPicker` is primarily a block editor component.
- **Action required:** None. Existing implementations continue to work without modification.

## Technical details

The diff modifies `packages/components/src/color-picker/picker.tsx` and `component.tsx`. `picker.tsx` replaces `HslColorPicker` and `HslaColorPicker` with `HsvColorPicker` and `HsvaColorPicker`, and manages a local `hsva` state. It introduces `toHsvaFromHsla()` to preserve the native HSVA saturation coordinate at black/white boundaries, preventing pointer snapping. An `isPointerInteractingRef` flag suppresses parent `hsla` prop sync during drag gestures. `component.tsx` adds an `isPickerInteractingRef` to block `useEffect`-driven `internalHSLA` updates while the user interacts, and replaces string-based hex comparisons with `safeColordColor.isEqual()` and `colord( nextHex ).isEqual( previousHex )` to avoid false positives from RGB/HSL round-trips. New props `onInteractionStart` and `onInteractionEnd` bridge the interaction lifecycle between the parent and the visual surface.

## Contribution

Opened and merged by @ciampo as a backport of #80205 to the `wp/7.1` branch. The automatic cherry-pick initially failed due to a changelog conflict with PR #80441, which @t-hamano resolved. The PR was reviewed and merged with automated unit, lint, and TypeScript checks passing.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
