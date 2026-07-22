# #80497: Global Styles panels: fix wrong preset committed and shown when two color presets share a hex

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Backported to WP Core`
- **Merged:** [`90e8e76`](https://github.com/WordPress/gutenberg/commit/90e8e76bb25229f28972a9516e8e53cc8bec0e3c)
- **Discussion:** [#80497](https://github.com/WordPress/gutenberg/pull/80497) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug in the Global Styles inspector where color presets sharing the same hex value caused incorrect preset commits and duplicate UI selection states. Previously, accepting an inherited color or setting a local color would re-encode the hex and match against the palette array, returning the first matching entry and marking both identical-hex swatches as selected. The fix introduces explicit `userSlug` and `inheritedSlug` tracking so the picker and commit logic resolve by preset identifier rather than hex value.

## Impact

- **Block & theme developers:** No action required. This is an internal block editor UI fix with no public API changes.
- **Site owners & editors:** Resolves confusing inspector behavior where clicking a color swatch would apply the wrong preset or show multiple swatches as selected when palettes contain duplicate hex values.
- **Platform & hosting teams:** No configuration or migration steps needed. The fix ships in the block editor package and is backported to WP Core.
- **Note:** Gradient presets with duplicate hex values are not addressed in this PR and will require a separate follow-up.

## Technical details

The change modifies `packages/block-editor/src/components/global-styles/color-gradient-dropdown-item.js` to accept a new `userSlug` prop alongside `inheritedSlug`. The `ColorGradientTab` component now computes `displayedSlug` by preferring `userSlug` when a local value exists, falling back to `inheritedSlug` otherwise. This slug is passed to `ColorPalette` via `colorSlug`, replacing the previous hex-based matching logic. The accept-inherited interceptor in `onChange` now calls `setValue( inheritedValue, inheritedSlug )` to preserve the correct preset reference.

Before:
```js
const displayed = userValue ?? inheritedValue;
// ...
colorSlug={ isGradient || userValue !== undefined ? undefined : inheritedSlug }
```

After:
```js
const displayed = userValue ?? inheritedValue;
const displayedSlug = userValue !== undefined ? userSlug : inheritedSlug;
// ...
colorSlug={ isGradient ? undefined : displayedSlug }
```

Panel components (`background-panel.js`, `color-panel.js`, `typography-panel.js`) were updated to extract and pass `userSlug` using `extractPresetSlug()` for their respective values. Tests in `test/color-panel.js` and `test/typography-panel.js` verify that duplicate-hex presets commit and render by slug identity.

## Contribution

The PR was opened as a follow-up to a discussion on #77894 and refined during review. @andrewserong suggested renaming `localSlug` to `userSlug` for consistency with `inheritedValue`/`inheritedSlug`, which was implemented. @talldanwp flagged it for the 7.1 release and coordinated the backport. The author noted that gradient presets share the same bug but require a separate PR due to `GradientPicker` API changes, which was deferred to a future iteration.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
