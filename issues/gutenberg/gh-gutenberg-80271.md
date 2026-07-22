# #80271: Editor: Restore fixed device preview height for mobile and tablet

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @faisalahammad
- **Labels:** `[Type] Bug`, `General Interface`, `[Package] Editor`, `First-time Contributor`, `Backported to WP Core`
- **Merged:** [`3a1fb6f`](https://github.com/WordPress/gutenberg/commit/3a1fb6f3620d7b96180ad1761682e2e69c5a5332)
- **Discussion:** [#80271](https://github.com/WordPress/gutenberg/pull/80271) · 11 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Gutenberg editor now restores device-shaped preview frames for mobile and tablet viewports, which previously collapsed to the full editor height after a width-only resizing model was introduced. The change calculates a fixed preview height based on the canvas width and device-specific aspect ratios (8:5 for mobile, 4:3 for tablet), applying it only when the canvas matches the preset width selected in the Preview dropdown. This ensures the editor canvas visually resembles a phone or tablet while preserving the flexible resizing behavior introduced in WordPress 7.1.

## Impact

- **Plugin & theme developers / block authors:** No action required. The change is internal to the editor UI and does not affect block rendering, REST API, or theme.json viewport settings.
- **Site owners / editors:** The Post Editor’s Preview dropdown will now display mobile and tablet frames with correct portrait aspect ratios, matching WordPress 7.0 behavior. Long content scrolls internally within the frame instead of stretching the editor vertically.
- **Hosting & platform teams:** No configuration or migration needed. The change is fully contained within the Gutenberg package and ships with WordPress 6.7+.

## Technical details

The diff introduces a new private store selector `getCanvasHeight` in `packages/editor/src/store/private-selectors.js`, alongside the existing `getCanvasWidth`. It derives the preview height using `DEVICE_ASPECT_RATIO_BY_DEVICE_TYPE` (`Mobile: 8 / 5`, `Tablet: 4 / 3`) and only returns a pixel value when `canvasWidth` exactly matches the preset width for that device type. Dragging the resize handle away from the preset returns `undefined`, allowing the frame to fill the editor.

`packages/editor/src/components/resizable-editor/index.js` is updated to read `canvasHeight` via `useSelect` and pass it to the `size` prop:
```js
// Before
height: enableResizing && height ? height : '100%',
// After
height:
    enableResizing && canvasHeight
        ? canvasHeight + 'px'
        : height || '100%',
```
A height transition is added to `packages/editor/src/components/resizable-editor/style.scss` to animate the frame size change, gated by `:not(.is-resizing)` and `prefers-reduced-motion`. The Site Editor and Styles canvas remain unaffected as they bypass `ResizableEditor` or use `enableResizing={ false }`.

## Contribution

Addressing a regression from #75121, @faisalahammad proposed restoring device-shaped preview frames. Reviewers @talldan and @ramonjd suggested replacing fixed pixel heights with dynamic aspect ratios to better support custom viewports. @faisalahammad implemented the ratio-based approach (8:5 mobile, 4:3 tablet) and aligned both to portrait orientation per @talldan’s feedback. The team also debated whether the shaped frame should persist during manual resizing; @ramonjd and @andrewserong preferred restricting it to the preset width only, a decision @faisalahammad adopted before merge. The PR received review from @ramonjd, @andrewserong, and @t-hamano before being merged and backported.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
