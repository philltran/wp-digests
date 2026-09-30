# #81120: Post Template: Fix mover arrows and labels for blocks inside the grid view

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Package] Block editor`
- **Merged:** [`6112126`](https://github.com/WordPress/gutenberg/commit/61121263ba0c376798d57a6aa3c102d7c03d0cf9)
- **Discussion:** [#81120](https://github.com/WordPress/gutenberg/pull/81120) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Blocks nested inside a Post Template (and Term Template) set to Grid view no longer inherit the grid layout in the editor UI. Previously, blocks such as Featured Image showed horizontal mover arrows and "Move left"/"Move right" labels even though they stack vertically inside each post item. The fix passes an explicit `{ type: 'default' }` layout to the template's inner blocks, so movers, insertion indicators, drag-and-drop targets, and grid child controls behave correctly.

## Impact

- **Site owners / editors:** Blocks inside a grid Query Loop now show up/down movers with "Move up"/"Move down" labels. Insertion indicators and drop targets use vertical orientation. Grid child controls no longer appear on blocks that are not grid items, and controls previously hidden (e.g. alignment) are exposed as in the list view.
- **Plugin & theme developers:** No front-end markup or saved-attribute changes. `useInnerBlocksProps` now honors a `layout` key in its options object; this is a new, undocumented-in-the-PR option, so treat it as an internal capability until documented. Custom blocks that render a container whose layout attribute does not apply to its children can use it to opt out.
- **Hosting / headless:** No impact; editor-only change.
- **Limitation:** Constrained layout on the Post Template still does not give template blocks wide/full alignment support; the `<ul>` layout applies to `<li>` items, not their children.
- No action required.

## Technical details

- `packages/block-editor/src/components/inner-blocks/index.js`: `useInnerBlocksProps` now destructures `layout: layoutOverride` from `options`, renames the `useBlockEditContext()` layout to `contextLayout`, and computes `const layout = layoutOverride ?? contextLayout`. The resolved layout feeds the manual grid placement check that disables the standard drop zone, so that check now uses the override when provided. The PR comment notes that this precedence already existed for the inner blocks settings.
- `packages/block-library/src/post-template/edit.js` and `term-template/edit.js`: each defines `const INNER_BLOCKS_LAYOUT = { type: 'default' }` and passes it as `layout` alongside `__unstableDisableLayoutClassNames: true` in `PostTemplateInnerBlocks` / `TermTemplateInnerBlocks`.

```js
// before
useInnerBlocksProps(
	{ className: clsx( 'wp-block-post', classList ) },
	{ __unstableDisableLayoutClassNames: true }
);

// after
useInnerBlocksProps(
	{ className: clsx( 'wp-block-post', classList ) },
	{
		__unstableDisableLayoutClassNames: true,
		layout: INNER_BLOCKS_LAYOUT, // { type: 'default' }
	}
);
```

- The Post Template's own `layout` attribute is unchanged and still applies to the `<ul>`.
- Adds an e2e test in `test/e2e/specs/editor/blocks/query.spec.js` that inserts a grid Post Template (`columnCount: 3`) with Title and Date blocks, selects Title, and asserts visible "Move up" and "Move down" buttons in the block toolbar.
- CHANGELOG entries added for `@wordpress/block-editor` and `@wordpress/block-library`.

## Contribution

During review, @andrewserong asked whether hard-coding the default layout was right given that Post Template supports custom `contentSize`/`wideSize` in constrained mode. @jorgefilipecosta argued that constrained layout constrains the `<li>` items rather than their grandchildren, so wide/full never reached template blocks anyway. Supporting it would require separating item arrangement (list/grid) from content width, which a single `layout` attribute cannot express and which would change existing front ends. He kept the minimal fix and left the larger redesign out of scope. The PR notes AI-assisted drafting reviewed by the author.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
