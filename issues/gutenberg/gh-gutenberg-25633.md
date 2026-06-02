# #25633: Inner blocks: try hook approach

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Feature] Block API`, `[Feature] Nested / Inner Blocks`, `[Type] Experimental`, `[Type] New API`
- **Merged:** [`a8fb704`](https://github.com/WordPress/gutenberg/commit/a8fb704c09fae63b1a3496d9b0b9cd7dac5cfad2)
- **Discussion:** [#25633](https://github.com/WordPress/gutenberg/pull/25633) · 18 comments · 3 reactions

## Summary

PR #25633 introduces `__experimentalUseInnerBlocksProps`, a React hook that replaces the `__experimentalTagName` and `__experimentalPassedProps` props on `<InnerBlocks />`. Instead of delegating wrapper element control to `<InnerBlocks />` via those props, block authors now call the hook with their own props object and spread the result onto any element they render directly. This gives block authors full control over the wrapper element, including rendering additional children before or after the inner blocks content.

## Impact

**Plugin & theme / block developers:**
- **Breaking for experimental prop users:** `__experimentalTagName` and `__experimentalPassedProps` have been removed from `<InnerBlocks />`'s accepted props (and from the underlying `BlockList` component). Any block using those props must migrate to `__experimentalUseInnerBlocksProps`.
- Migration is mechanical: replace `<InnerBlocks __experimentalTagName="div" __experimentalPassedProps={ blockProps } { ...options } />` with the hook pattern and spread on your own element (see technical details below).
- The default `<InnerBlocks />` component continues to render a `div.block-editor-inner-blocks` wrapper, so blocks that do **not** use the experimental tag/passed-props API require no changes.
- The `has-overlay` CSS class logic and `block-editor-block-list__layout` class are now applied by the hook itself — do not manually add them.

**No action required** for site owners or headless/REST consumers; this is an editor-only JavaScript change.

## Technical details

**New export** — `packages/block-editor/src/components/index.js` adds:
```js
export {
  default as InnerBlocks,
  useInnerBlocksProps as __experimentalUseInnerBlocksProps,
} from './inner-blocks';
```

**Hook signature** (`packages/block-editor/src/components/inner-blocks/index.js`):
```js
export function useInnerBlocksProps( props = {}, options = {} )
```
- `props` — any props (including `ref`) to merge onto the wrapper element.
- `options` — the same options previously passed directly to `<InnerBlocks />` (`allowedBlocks`, `template`, `templateLock`, `orientation`, `renderAppender`, `value`/`onChange` for controlled mode, etc.).
- Returns a merged props object whose `className` includes `block-editor-block-list__layout` and conditionally `has-overlay`; `children` is set to the rendered `ControlledInnerBlocks` or `UncontrolledInnerBlocks` component.

**Before (removed pattern):**
```js
<InnerBlocks
  orientation="horizontal"
  __experimentalTagName="div"
  __experimentalPassedProps={ blockProps }
  allowedBlocks={ ALLOWED_BLOCKS }
/>
```

**After (new pattern):**
```js
const innerBlocksProps = useInnerBlocksProps( blockProps, {
  allowedBlocks: ALLOWED_BLOCKS,
  orientation: 'horizontal',
} );
// ...
<div { ...innerBlocksProps } />
```

**Internal refactor of `BlockList`** (`packages/block-editor/src/components/block-list/index.js`):
- `__experimentalTagName` and `__experimentalPassedProps` are removed from `BlockList`'s props.
- `BlockList` now always uses `div` (or `RootContainer` for the root) and delegates block rendering to a new internal `Items` component.
- A new named export `BlockListItems` wraps `Items` in an `AsyncModeProvider` and is consumed by `UncontrolledInnerBlocks` via `wrapperRef` instead of a forwarded ref.
- The `has-overlay` overlay detection logic moved out of `UncontrolledInnerBlocks` into `useInnerBlocksProps`.

**Core blocks migrated:** `core/buttons`, `core/column`, `core/columns`, `core/cover`, `core/group`, `core/media-text` all updated to the hook pattern in their respective `edit.js` files.

## Contribution

Opened and authored by @ellatrix. @ZebulanStanphill raised the naming issue in review — the original hook was named `useInnerBlockWrapperProps` and went through `useInnerBlocksWrapperProps` before settling on `useInnerBlocksProps` (matching the precedent set by `useBlockProps`). @youknowriad flagged a design concern about whether the hook could validate that its props are applied to the block's outermost element; @ellatrix noted a DOM-level check (verifying the parent node is a block list) could be added later. The PR was merged at commit `a8fb704`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
