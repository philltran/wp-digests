# #83750: PaletteEdit: Stop changing the slug when renaming a preset

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Package] Components`, `Global Styles`
- **Merged:** [`6c72fd5`](https://github.com/WordPress/gutenberg/commit/6c72fd500addacf597ac8bc122758a284446f25f)
- **Discussion:** [#83750](https://github.com/WordPress/gutenberg/pull/83750) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The `PaletteEdit` component in `@wordpress/components` no longer regenerates a preset's slug from its name on every keystroke during a rename. Previously, renaming a custom colour, gradient, or duotone in the Site Editor's Styles panel would call `kebabCase()` on the new name and overwrite the slug, silently detaching every block and CSS rule that referenced the old slug (e.g. `has-{slug}-color`, `--wp--preset--color--{slug}`). The fix keeps the original slug intact so blocks retain their styling after a rename.

## Impact

- **Site owners / editors:** Renaming a custom colour, gradient, or duotone in Styles no longer breaks blocks that use that preset. No action required; the fix is transparent.
- **Plugin & theme developers using `PaletteEdit` or `__experimentalPaletteEdit`:** The public `slugPrefix` prop on `PaletteEdit` is unchanged. The internal `OptionProps` and `PaletteEditListViewProps` types no longer include `slugPrefix`, but those types are not part of the public API surface. No code changes needed.
- **Saved presets:** Unaffected — they keep their existing slugs.
- **No breaking changes or deprecations.**

## Technical details

In `packages/components/src/palette-edit/index.tsx`, the `Option` component's name-input `onChange` handler previously computed a new slug on every keystroke:

```tsx
// Before
onChange( {
  ...element,
  name: nextName,
  slug: slugPrefix + kebabCase( nextName ?? '' ),
} )

// After
onChange( {
  ...element,
  name: nextName,
} )
```

The `kebabCase` import from `@wordpress/kebab-case` is removed. The `slugPrefix` prop is stripped from the internal `Option` and `PaletteEditListView` components (and their type definitions in `types.ts`), since nothing reads it anymore. The public `PaletteEdit` component still accepts `slugPrefix` for initial slug assignment when new items are added (e.g. `custom-color-N`), but it is no longer threaded down to the rename path.

The browser test in `packages/components/src/palette-edit/test/index.browser.test.tsx` is updated: the rename assertion now expects the original slug to be preserved rather than a kebab-cased version of the new name.

## Contribution

Opened by @ramonjd implementing an idea proposed by @tiennguyenvan in issue #50232. @annezazu reviewed, tested the fix in the Site Editor (confirming the new name appears on hover), and requested @davewhitley and @nyiriland for additional testing. The PR carried 3 comments and 1 reaction before merging as `6c72fd5`. No notable design debate or rejected alternatives appear in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
