# #79798: Block position: Allow options dropdown to flip

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Bug`, `[Feature] Blocks`, `[Package] Block editor`
- **Merged:** [`e7c9f7e`](https://github.com/WordPress/gutenberg/commit/e7c9f7e60ffeaea93dc03b77e436721220d71f5a)
- **Discussion:** [#79798](https://github.com/WordPress/gutenberg/pull/79798) · 10 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

Fixes a clipped dropdown bug in the block editor's Position control by migrating `CustomSelectControl` to the portal-based `SelectControl` from `@wordpress/ui`. The previous component forced the popover downward, causing it to shrink and clip at the bottom of low-viewport screens; the new implementation automatically flips and resizes to remain fully visible within the inspector sidebar.

## Impact

- **Block & Plugin Developers**: No public APIs or block.json schemas changed. However, any code that hooks into or overrides the Position control's DOM/CSS must update targets, as the underlying component and its class names have shifted.
- **Core/Platform Teams**: Establishes a portal-based popup pattern for inspector controls, reducing clipping edge cases across other `@wordpress/components` popovers.
- No migration or configuration changes required for site owners or standard theme developers.

## Technical details

The change modifies `packages/block-editor/src/hooks/position.js` to replace the `CustomSelectControl` with `SelectControl` from `@wordpress/ui`. Key behavioral and structural shifts include:

- **Option shape**: Internal option objects now use a `label` property instead of `name`.
- **Callback signature**: `onChange` is replaced by `onValueChange`, which receives the selected item object.
- **Hint rendering**: Help text moves from a parent container's `help` prop to be rendered inside individual `SelectControl.Item` components, wired via dynamic `id`/`aria-describedby` attributes.

**Before (CoreSelectControl pattern)**:
```js
<BaseControl help={ stickyHelpText }>
  <CustomSelectControl
    label="Position"
    options={ options }
    value={ selectedOption }
    onChange={ ({ selectedItem }) => onChangeType( selectedItem.value ) }
  />
</BaseControl>
```

**After (Portal-based SelectControl)**:
```js
<SelectControl
  description={ stickyHelpText }
  items={ options }
  value={ selectedOption }
  onValueChange={ ( { selectedItem } ) => onChangeType( selectedItem.value ) }
>
  { options.map( ( option ) => (
    <SelectControl.Item key={ option.key } value={ option } label={ option.label } aria-describedby={ hintId }>
      <div>{ option.label }</div>
      {/* hint rendered inside item */}
    </SelectControl.Item>
  )) }
</SelectControl>
```

CSS additions in `packages/block-editor/src/hooks/block-hooks.scss` introduce `.block-editor-hooks__position-control-item` (to constrain popover width against the anchor) and `.block-editor-hooks__position-control-item-hint` for styling the embedded hint text. The portal-based popup correctly avoids clipping by the sticky sidebar header and viewport boundaries.

## Contribution

Opened by @talldan to resolve clipped dropdowns in low-viewport editors. Initial exploration considered an `allowPopoverFlip` opt-in prop on `CustomSelectControl`, but reviewers flagged that flipping could still overlap the sticky sidebar header (referencing #63180). The PR was refactored to adopt `@wordpress/ui`'s `SelectControl`, which uses a portal-based popup that reliably positions above or below regardless of layout constraints. The change merged with co-author credits from @andrewserong, @mirka, @ciampo, and @ramonjd.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
