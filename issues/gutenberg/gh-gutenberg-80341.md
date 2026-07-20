# #80341: Hide block style variations when state is enabled in global styles

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Bug`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Style States`
- **Merged:** [`4e91395`](https://github.com/WordPress/gutenberg/commit/4e91395ee4c806ba7efc4f67546e8f8920a3cc42)
- **Discussion:** [#80341](https://github.com/WordPress/gutenberg/pull/80341) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Global Styles UI now hides block style variations when a pseudo or viewport state is active. Previously, selecting a state like `:hover` or a tablet viewport would still display the full list of style variations, creating an ambiguous styling path since state selection does not persist when navigating between variations. This change clarifies the editor interface by only rendering the variations panel when the default state is selected.

## Impact

- **Block & theme developers:** No code changes required. The change affects the Global Styles UI experience when styling blocks that expose variations and states.
- **Site owners & editors:** When editing a block in the Global Styles panel with a non-default state active, the style variations dropdown will be hidden, preventing accidental styling of variations while in a specific state context.
- **No breaking changes, deprecations, or migration steps.**

## Technical details

The change modifies the `ScreenBlock` component in `packages/global-styles-ui/src/screen-block.tsx`. It updates the `hasVariationsPanel` boolean to include a check for `hasSelectedState`.

Before:
```tsx
const hasVariationsPanel = !! blockVariations?.length && ! variation;
```

After:
```tsx
const hasVariationsPanel =
	!! blockVariations?.length && ! variation && ! hasSelectedState;
```

The `hasSelectedState` flag reflects when a pseudo or viewport state is active in the global styles editor. By adding `! hasSelectedState` to the condition, the variations panel is conditionally hidden, ensuring the UI only presents variations when the default state is selected.

## Contribution

Opened and merged by @tellthemachines, with review and approval from @andrewserong. The fix originated during testing of PR #80339. During review, @tellthemachines noted that state selection does not persist across navigation, making it confusing to select a variation while a state is active. @andrewserong agreed that hiding variations in this context clarifies the styling path, leading to a quick merge. The change was backported to the `wp/7.1` branch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
