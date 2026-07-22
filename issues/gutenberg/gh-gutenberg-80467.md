# #80467: Block Editor: Guard against non-string spacing preset values

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `CSS Styling`, `Backported to WP Core`
- **Merged:** [`6755a4d`](https://github.com/WordPress/gutenberg/commit/6755a4d4d3de44dc23be3ab457e2baf2475c9f5f)
- **Discussion:** [#80467](https://github.com/WordPress/gutenberg/pull/80467) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Block Editor now safely handles malformed spacing preset values that are not strings, preventing a `TypeError` that previously crashed the editor into a white screen. When a block's `blockGap` or similar spacing property contains an array or object instead of a string, `getSpacingPresetCssVar` previously called `.match()` on it, throwing an error. The fix adds a type guard so non-string values return `undefined` early, keeping the editor stable while the corresponding PHP-side fatal error is addressed separately.

## Impact

- **Plugin & theme developers / Block authors:** No immediate code changes required, but blocks that programmatically generate or manipulate spacing data should ensure `blockGap` and related spacing values are strings to avoid triggering the guard.
- **Site owners / Editors:** Editors will no longer encounter a white-screen crash in the Block Editor when loading posts with malformed spacing data (e.g., arrays in `blockGap`).
- **Hosting & platform teams:** No configuration changes needed. The fix is backported to WordPress Core.
- **No action required** for standard usage.

## Technical details

The change modifies `getSpacingPresetCssVar` in `packages/block-editor/src/components/spacing-sizes-control/utils.js`. Previously, the function only checked for falsy values before calling `.match()`. The diff updates the guard to explicitly verify the type:

```js
// Before
if ( ! value ) {
    return;
}

// After
if ( ! value || typeof value !== 'string' ) {
    return;
}
```

This prevents `TypeError: value.match is not a function` when `value` is an array (e.g., `{ top: [ '1rem' ] }`), an object, or a number. Unit tests in `packages/block-editor/src/components/spacing-sizes-control/test/utils.js` were added to cover array, object, and number inputs, verifying they return `undefined`. The fix runs client-side in the Block Editor and complements a server-side PHP fix for the same malformed data pattern.

## Contribution

Opened by @ramonjd and co-authored by @tyxla, the PR was merged with minimal review discussion. @t-hamano manually re-applied the backport label to ensure the fix was cherry-picked to both `wp/7.1` and `release/23.6`. No alternative approaches were proposed or debated.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
