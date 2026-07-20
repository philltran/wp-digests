# #80391: Editor: Disable canvas resizing while zoomed out

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Zoom Out`, `[Feature] Style States`
- **Merged:** [`b4cd0cc`](https://github.com/WordPress/gutenberg/commit/b4cd0ccfb54cecf2d3e4bd28f69057353dfe159e)
- **Discussion:** [#80391](https://github.com/WordPress/gutenberg/pull/80391) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg editor now disables canvas resizing when Zoom Out mode is active. Previously, enabling Zoom Out while a Mobile or Tablet device preview was selected left the canvas locked to the device width but still resizable, producing an unnatural horizontal scrollbar. This regression from the unified device preview and resizable canvas implementation is fixed by returning `undefined` from the `getCanvasWidth` selector during zoom out, which correctly triggers the existing resize-disabling logic in the visual editor.

## Impact

- **Plugin & theme developers:** No action required. `getCanvasWidth` is a private API, and the behavioral change only affects internal editor state.
- **Site owners & editors:** Resolves a visual bug where the editor canvas would remain resizable and show a scrollbar when switching to Zoom Out mode during device preview.
- **Platform & hosting teams:** No configuration or migration needed. The fix ships with the Gutenberg plugin and has been backported to WordPress 7.1.

## Technical details

The change modifies `packages/editor/src/store/private-selectors.js`. The `getCanvasWidth` selector is converted from a simple state accessor to a `createRegistrySelector` that checks the `isZoomOut()` state from `blockEditorStore`. When zoomed out, it returns `undefined` instead of `state.canvasWidth`. This aligns with `getDeviceType`, which already reports `Desktop` during zoom out, and ensures the `enableResizing` logic in the visual editor correctly evaluates to `false`.

Before:
```js
export function getCanvasWidth( state ) {
	return state.canvasWidth;
}
```

After:
```js
export const getCanvasWidth = createRegistrySelector(
	( select ) => ( state ) => {
		if ( unlock( select( blockEditorStore ) ).isZoomOut() ) {
			return undefined;
		}
		return state.canvasWidth;
	}
);
```

## Contribution

Opened and merged by @t-hamano, with co-authorship from @ramonjd. The PR was identified as a regression from #75121 and was quickly backported to the Gutenberg RC and WordPress 7.1 branches. During review, the author noted that the abrupt visual transition when switching to zoom-out mode could be smoother, but decided to merge the core fix first and address animation smoothing separately.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
