# #81059: [WP 7.1] Background: Fix the legacy gradient UI where a gradient cannot be selected

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `[Feature] Design Tools`
- **Merged:** [`e6fb0cd`](https://github.com/WordPress/gutenberg/commit/e6fb0cd23df8f05bea414a89aead632214b94544)
- **Discussion:** [#81059](https://github.com/WordPress/gutenberg/pull/81059) · 2 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug in the Block Editor’s legacy gradient UI where selecting a gradient preset silently failed when a theme opted out of the new gradient background system. The change ensures the fallback path correctly applies the selected gradient to the block’s legacy attribute instead of doing nothing. This restores expected editor behavior for themes that disable the modern gradient controls via theme.json.

## Impact

- **Theme developers**: If your theme sets `settings.background.gradient` to `false` or omits it, the legacy gradient picker in the Block Editor will now correctly apply presets.
- **Plugin & block developers**: No public API changes or deprecations. The `BackgroundImagePanel` component now respects the `settings.background.gradient` flag alongside block-level support.
- **Site owners & editors**: No action required. Fixes a broken UI state that previously prevented gradient selection in the editor when the new gradient system was disabled.

## Technical details

In `packages/block-editor/src/hooks/background.js`, the gradient routing logic inside `BackgroundImagePanel` was updated to check both block support and the theme’s `theme.json` setting. Previously, the component only verified block-level support:

```js
const backgroundGradientSupported = hasBackgroundSupport(
	name,
	'gradient'
);
```

The diff adds a check for the explicit theme setting:

```js
const backgroundGradientSupported =
	hasBackgroundSupport( name, 'gradient' ) &&
	!! settings?.background?.gradient;
```

When `settings.background.gradient` is falsy, selecting a preset now writes the gradient slug to the legacy `color.gradient` attribute and leaves `style.background.gradient` undefined. Unit tests in `packages/block-editor/src/hooks/test/background.js` verify both paths: the new system writes to `style.background.gradient`, while the legacy fallback writes to the `gradient` attribute.

## Contribution

Opened as a follow-up patch to a larger background system PR, the change was authored by @ramonjd with co-authors @t-hamano and @andrewserong. Review focused on verifying the legacy fallback routing and ensuring the new `theme.json` flag was properly evaluated alongside block support. The patch was merged quickly after passing automated tests, with no alternative approaches debated.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
