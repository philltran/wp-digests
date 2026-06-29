# #74257: Add icon state classes to Accordion block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @deepakpra
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `First-time Contributor`, `[Block] Accordion`
- **Merged:** [`df88f07`](https://github.com/WordPress/gutenberg/commit/df88f07f4ff48272f529a7c9bceb4eba183ce61e)
- **Discussion:** [#74257](https://github.com/WordPress/gutenberg/pull/74257) · 7 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The Accordion Heading block now outputs CSS state classes (`.has-icon`, `.has-icon-left`, `.has-icon-right`) on the `.wp-block-accordion-heading` wrapper based on the `showIcon` and `iconPosition` attributes. This change exposes attribute-driven styling hooks in both editor and frontend markup, aligning the block with established Gutenberg patterns without altering existing schema or block structure.

## Impact

- **Theme & block developers**: Gain reliable CSS hooks to conditionally style the heading toggle, icon, or adjacent `.wp-block-accordion-panel` using sibling selectors.
- **Existing sites**: No breaking changes. A deprecated save version (`v3`) is added so that older serialized markup without these classes remains valid and renders correctly.
- **Action required**: None for frontend functionality; developers updating custom styles may want to leverage the new `.has-icon-*` hooks for cleaner, attribute-driven CSS.

## Technical details

Changes are concentrated in `packages/block-library/src/accordion-heading/edit.js` and `save.js`. The render logic now computes icon-related classes via `clsx()` and passes them into `useBlockProps()`:
```js
const blockProps = useBlockProps.save( {
	className: clsx( {
		'has-icon': showIcon,
		'has-icon-left': showIcon && iconPosition === 'left',
		'has-icon-right': showIcon && iconPosition === 'right',
	} ),
} );
```
The change also introduces `deprecated.js` version `v3`, which replicates the pre-state-class output to maintain backward compatibility for sites that already serialized accordion headings. Block rendering fixtures and README examples in `accordion-heading/`, `accordion-item/`, and `accordion/` were updated to reflect the new markup structure.

## Contribution

Opened by `@deepakpra` to address issue #74254, the PR was refined through review feedback from `@t-hamano` and `@mrleemon`. Initial discussion considered applying state classes to the parent Accordion block, but the team shifted the implementation to `.wp-block-accordion-heading` for more precise styling hooks. Merged as `df88f07`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
