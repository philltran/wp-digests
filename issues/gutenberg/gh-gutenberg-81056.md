# #81056: Background: Fix the legacy gradient UI where a gradient cannot be selected

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `[Feature] Design Tools`, `Backported to WP Core`
- **Merged:** [`511df82`](https://github.com/WordPress/gutenberg/commit/511df8215203a2c7b059864d4edf0ad764c972e6)
- **Discussion:** [#81056](https://github.com/WordPress/gutenberg/pull/81056) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a bug in the Block Editor where selecting a gradient preset in the legacy Background panel fails to apply or save the gradient when a theme disables the new gradient system via `settings.background.gradient` in `theme.json`. The issue stemmed from a mismatch between the UI component, which wrote to the legacy `gradient` attribute, and the block hook, which only checked for the new `background.gradient` block support. The fix aligns the hook’s validation logic with the panel’s fallback behavior, ensuring gradients persist correctly regardless of the `theme.json` setting.

## Impact

- **Theme & plugin developers:** No breaking changes. If your theme sets `"background": { "gradient": false }` in `theme.json` to force the legacy gradient UI, gradients will now correctly apply and save to blocks that declare `background.gradient` support.
- **Site owners:** No action required. Existing themes using the legacy gradient fallback will now work as expected in the editor.
- **Block developers:** Blocks declaring `background.gradient` support will now reliably save legacy gradient slugs when the global setting is disabled.

## Technical details

The root cause was in `packages/block-editor/src/hooks/background.js`. The `BackgroundImagePanel` component renders the legacy gradient UI when `settings.background.gradient` is `false` and writes the selected preset slug to the `gradient` attribute. However, the hook’s `backgroundGradientSupported` check only evaluated `hasBackgroundSupport( name, 'gradient' )`, causing it to expect the value at `style.background.gradient` and overwrite the legacy attribute with `undefined`.

The diff updates the condition to:
```js
const backgroundGradientSupported =
	hasBackgroundSupport( name, 'gradient' ) &&
	!! settings?.background?.gradient;
```

This ensures the hook respects the resolved `theme.json` setting. When the setting is disabled, the hook skips its new-gradient logic, allowing the legacy `gradient` attribute to persist. Unit tests in `packages/block-editor/src/hooks/test/background.js` verify both routing paths: `style.background.gradient` when enabled, and the legacy `gradient` attribute when disabled.

## Contribution

The record carries no design debate or alternative approaches; the fix was authored by @t-hamano following a logic mismatch identified during testing, and @ramonjd coordinated the backport to the `wp/7.1` branch after resolving a cherry-pick conflict.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
