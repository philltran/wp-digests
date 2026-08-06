# #81193: Ensure device preview is always accurate when window is zoomed in

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Bug`, `General Interface`, `[Package] Editor`, `Backported to WP Core`
- **Merged:** [`04f245f`](https://github.com/WordPress/gutenberg/commit/04f245f71de1a32c2daa0f5d581ed457b6229e3f)
- **Discussion:** [#81193](https://github.com/WordPress/gutenberg/pull/81193) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a bug in the Gutenberg editor where browser zoom levels (e.g., 125% or 130%) cause the device preview iframe to render outside its intended media query breakpoint, breaking responsive styles. The fix insets the preview canvas width by up to one pixel to keep it safely within the breakpoint range, ensuring accurate device previews regardless of browser zoom.

## Impact

- **Block editor users & developers**: No action required. The editor’s device preview (tablet/mobile) will now consistently apply responsive styles even when the browser window is zoomed.
- **Theme & plugin developers**: No impact on frontend rendering, REST API, or block registration. This change only affects the editor canvas dimensions.
- **Hosting & platform teams**: No configuration or migration steps needed. The fix ships with the `@wordpress/editor` package and was manually backported to the `wp/7.1` branch.

## Technical details

The core logic change lives in `packages/editor/src/utils/device-type.js`. The `getCanvasWidthByDeviceType()` function previously returned the exact breakpoint value (e.g., `480` for mobile, `782` for tablet). It now calculates an inset width using a new `DEVICE_PREVIEW_WIDTH_OFFSET` constant and a safety margin: `offset = Math.min(1, (width - lowerBreakpoint) / 2)`. This prevents CSS media queries from being bypassed when browser zoom rounds the iframe viewport past the breakpoint threshold. The `getDeviceTypeByCanvasWidth()` selector was also refactored to read breakpoints directly from `getViewportBreakpoints()` rather than calling `getCanvasWidthByDeviceType()` recursively.

Before/after behavior for the mobile preset:
```js
// Before: exact breakpoint
return getViewportBreakpointValueInPixels( breakpoints.mobile ); // 480

// After: inset by up to 1px to stay inside the media query
return width - offset; // 479
```

Test expectations in `packages/editor/src/store/test/actions.js` and `packages/editor/src/utils/test/device-type.js` were updated to reflect the new inset values, and `getCanvasHeight` tests now account for the shifted `canvasWidth` inputs.

## Contribution

Opened by @tellthemachines to resolve #81072, with review and cross-browser testing support from @t-hamano and @jennydupuy. The implementation was assisted by an AI coding tool, which the author reviewed and tested before submission. After merging to `trunk`, a manual cherry-pick was required to resolve a conflict on the `wp/7.1` branch, which @t-hamano handled separately. Reviewers noted the fix improves accessibility by ensuring responsive styles render predictably at common zoom levels.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
