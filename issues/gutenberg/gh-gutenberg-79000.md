# #79000: Avoid dirtying related navigation entities during passive render

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamziel
- **Labels:** `[Type] Bug`, `[Package] Block library`, `Backported to WP Core`
- **Merged:** [`59a5ac2`](https://github.com/WordPress/gutenberg/commit/59a5ac250459e878717e8329d0f7d146cd97297c)
- **Discussion:** [#79000](https://github.com/WordPress/gutenberg/pull/79000) · 10 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a bug where passive rendering of related Navigation blocks incorrectly marks unedited template parts and menus as dirty. The `core/navigation` block's attribute normalization effect now only runs when the block or one of its inner blocks is actively selected, preventing stale attribute conversions from triggering unwanted entity saves during editor load or preview.

## Impact

- **Block & theme developers**: No immediate code changes required; behavior correction for how `core/navigation` handles attribute normalization during passive rendering.
- **Plugin & theme developers**: If relying on the WordPress data layer's entity dirty state to track unsaved changes, passive rendering will no longer incorrectly flag related template parts or menus as modified.
- **Site owners**: No action required. Standard editor experience remains unchanged.

## Technical details

In `packages/block-library/src/navigation/edit/index.js`, the existing `useEffect` hook that resets `submenuVisibility` to `'hover'` when orientation is horizontal was updated to gate its execution on block selection state.

**Before:**
```js
useEffect(() => {
  if (orientation === 'horizontal' && submenuVisibility === 'always') {
    setAttributes({ submenuVisibility: 'hover', showSubmenuIcon: true });
  }
}, [orientation, submenuVisibility, setAttributes]);
```

**After:**
```js
useEffect(() => {
  if (
    (isSelected || isInnerBlockSelected) &&
    orientation === 'horizontal' &&
    submenuVisibility === 'always'
  ) {
    setAttributes({ submenuVisibility: 'hover', showSubmenuIcon: true });
  }
}, [isSelected, isInnerBlockSelected, orientation, submenuVisibility, setAttributes]);
```

A new E2E regression test in `test/e2e/specs/editor/blocks/navigation-passive-rendering.spec.js` validates that enabling "Show template" leaves related dirty entity records empty and only dirties the post entity when edited.

## Contribution

Opened by @adamziel to resolve a state-dirtying regression introduced during passive template rendering. @draganescu approved the approach, noting it aligns with expected edit-time behavior. During review, @talldan proposed replacing the `useEffect` with block-level `migrate` logic or deriving computed values at render time to avoid `setAttributes` in effects entirely. @mcsf highlighted that block context passing makes a fully derived approach complex. The author opted to narrow the effect's trigger scope as a targeted fix, deferring deeper data-model reorganization for a follow-up. Merged and backported to WordPress Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
