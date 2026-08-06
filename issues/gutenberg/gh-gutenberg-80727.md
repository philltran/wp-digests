# #80727: Dynamic Gallery: Rename toolbar button to Detach and add a modal explaining what will happen

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Gallery`, `Backported to WP Core`
- **Merged:** [`794b677`](https://github.com/WordPress/gutenberg/commit/794b6778f11aca2cbc4b3a4431f427cfcc73b17e)
- **Discussion:** [#80727](https://github.com/WordPress/gutenberg/pull/80727) · 14 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The dynamic Gallery block’s toolbar and inspector controls now rename the “Convert to images” action to “Detach” and gate it behind a confirmation modal. Previously, clicking the button immediately materialized the gallery into static, editable inner blocks. The new modal explains that detaching preserves the current images but breaks the live link to the post’s media library, reducing user confusion about the one-way conversion.

## Impact

- **Block editor users:** The dynamic-to-static conversion now requires an explicit confirmation step, preventing accidental loss of the dynamic source link.
- **Plugin & theme developers:** No breaking changes or API modifications. The underlying `convertToStatic` behavior remains identical, but the UI trigger is now asynchronous and modal-driven.
- **No action required** for existing codebases; this is a pure editor UX improvement.

## Technical details

The change lives in `packages/block-library/src/gallery/dynamic-gallery.js`. It introduces a `DetachGalleryDialog` component wrapping `ConfirmDialog` and a new `isConfirmingDetach` state in both `GallerySourcePanel` and `GalleryDynamicView`. The toolbar `ToolbarButton` and inspector `Button` now set `isConfirmingDetach` to `true` on click instead of directly invoking `convertToStatic`. The modal’s `onConfirm` callback triggers `convertToStatic()` and clears the state. The component uses `__next40pxDefaultSize` and `variant="secondary"` for the inspector button, and the dialog is rendered conditionally alongside the existing `ToolsPanel` and `BlockControls`. The CHANGELOG notes the behavior shift under Enhancements.

## Contribution

Opened to address UX confusion around the dynamic Gallery mode, the PR quickly moved through review with a focus on copy and dialog sizing. Contributors debated whether the confirmation should be a `small` or `medium` dialog and refined the explanatory text to balance clarity with brevity. The final implementation settled on a `medium` dialog with the agreed-upon wording, and the PR was merged after passing CI.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
