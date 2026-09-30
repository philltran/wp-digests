# #81855: Preserve layout and style attributes on Column transform

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Columns`
- **Merged:** [`c368c68`](https://github.com/WordPress/gutenberg/commit/c368c6810d112ff9cd345fa9b1cd08cc39303e33)
- **Discussion:** [#81855](https://github.com/WordPress/gutenberg/pull/81855) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Transforming a Columns block into a Row or Grid (Group variations) previously discarded the per-Column styles and layout, because each Column was replaced by a bare Group with a hard-coded `constrained` layout. The Columns transforms now copy every attribute of each Column that is also defined on `core/group` (styles, layout, preset colors/fonts, `className`, `anchor`, `lock`, `templateLock`) onto the replacement Group wrapper. Columns with no such attributes and a single inner block still unwrap as before.

## Impact

- **Site owners / editors:** Converting Columns to Row or Grid now keeps each column's colors, typography, spacing, borders and layout type instead of silently resetting them. Anchors, custom classes and locks on individual columns also carry over.
- **Plugin & theme developers:** No API changes. Custom attributes added to `core/column` are copied only if `core/group` also defines them (for example, via a filter on block registration). Attributes specific to Column are not copied: `width` (still converted to Row child sizing) and `verticalAlignment`.
- **Behavior change to note:** Wrapper Groups created for columns now default to `layout: { type: 'default' }` (flow) instead of `{ type: 'constrained' }` when the Column had no layout. Rendered output of such wrappers may differ slightly from before, and any tests asserting `constrained` will need updating.
- No migration or configuration required.

## Technical details

All changes are in `packages/block-library/src/columns/transforms.js`.

- New helper `getGroupAttributes( attributes )` uses `getBlockType( 'core/group' )?.attributes` and keeps only entries of the Column's attributes whose names exist on Group (via `Object.hasOwn`). This replaced an earlier narrower approach after review feedback, per the discussion.
- New constant `DEFAULT_COLUMN_LAYOUT = { type: 'default' }`, used when the Column has no `layout`.
- `getGridInnerBlocks`: previously wrapped a column in a Group only when it had more than one inner block. Now it also wraps when the Column has any Group-compatible attribute, a non-empty `style`, or a non-empty `layout`. The Group gets `...groupAttributes`, `layout` (Column's or default), and `style` if non-empty.
- `getRowInnerBlocks`: the single-inner-block unwrap shortcut now applies only when the Column has no Group attributes, style or layout. Otherwise a Group is created with `...groupAttributes`, the Column's layout (or default), and `style` merged as `{ ...columnStyle, layout: { ...columnStyle.layout, ...childLayout } }`. The Column `width` still becomes `selfStretch: 'fixed'` / `flexSize`, and an existing `style.layout.columnSpan` is preserved.

```js
// before
createBlock( 'core/group', { layout: { type: 'constrained' } }, columnInnerBlocks );

// after
createBlock( 'core/group', {
	...groupAttributes,
	layout: hasColumnLayout ? columnLayout : { type: 'default' },
	...( hasColumnStyle && { style: columnStyle } ),
}, columnInnerBlocks );
```

Tests in `columns/test/transforms.js` were updated (`constrained` → `default`) and new cases cover styles, layouts, root-level preset attributes, and anchor/lock/templateLock with `width` and `verticalAlignment` excluded. The bundle grows about 200 B.

## Contribution

A follow-up to #81802 and #78713, which added the Row and Grid transforms but missed per-Column attributes. In review, @ramonjd found that anchors and locks were dropped, while noting neither was a big deal. @andrewserong echoed the anchor point. @tellthemachines responded by broadening the fix to copy all Group-valid Column attributes, which also simplified the logic. The author disclosed use of AI tooling (Codex).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
