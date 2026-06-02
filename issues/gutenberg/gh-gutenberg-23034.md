# #23034: Light block: introduce useBlockWrapperProps

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Feature] Blocks`, `[Package] Block library`, `[Package] Block editor`, `[Type] Experimental`
- **Merged:** [`aa0abb9`](https://github.com/WordPress/gutenberg/commit/aa0abb915e843e2d636d4242aa249e504284abbc)
- **Discussion:** [#23034](https://github.com/WordPress/gutenberg/pull/23034) · 23 comments · 2 reactions

## Summary

`useBlockWrapperProps` is introduced in `@wordpress/block-editor` as a React hook that marks any DOM element as a block wrapper in the editor, replacing the component-based `__experimentalBlock` pattern. Rather than wrapping block output in a `<Block.div>` or similar component, a block's `edit` function calls the hook, spreads the returned props onto its own root element, and preserves its native element structure without an extra wrapper. The hook automatically merges `className` and `style` from both the block's own props and the internal `wrapperProps` context, and it accepts an optional `ref` so blocks that already hold a ref to their root element can pass it through. It is exported experimentally as `__experimentalUseBlockWrapperProps`.

## Impact

**Plugin & theme / block developers**
- A new `__experimentalUseBlockWrapperProps` hook is available from `@wordpress/block-editor`. Blocks can adopt it now for a cleaner pattern.
- `__experimentalBlock` (e.g. `Block.div`, `Block.p`) **is not deprecated in this PR** — existing blocks using it continue to work without changes.
- Because the hook moves `onKeyDown`, `onMouseLeave`, `onMouseOver`, and `onMouseOut` to native `addEventListener` calls inside `useEffect`, block authors can now safely include those same event handlers in their own props; React will no longer silently clobber one with the other.
- The hook is still marked experimental; the export name and signature may change before stabilisation.

**No action required** for anyone not actively authoring block `edit` components.

## Technical details

**Core refactor — `packages/block-editor/src/components/block-list/block-wrapper.js`**

The logic previously inside the `BlockComponent` `forwardRef` component is extracted into the exported `useBlockWrapperProps` function:

```js
export function useBlockWrapperProps( props = {}, { __unstableIsHtml } = {} ) {
    const fallbackRef = useRef();
    const ref = props.ref || fallbackRef;
    // reads BlockListBlockContext, useSelect, useDispatch …
    return {
        ...wrapperProps,
        ...props,
        ref,
        id: `block-${ clientId }${ htmlSuffix }`,
        tabIndex: 0,
        role: 'group',
        'aria-label': blockLabel,
        'data-block': clientId,
        'data-type': name,
        'data-title': blockTitle,
        className: classnames( className, props.className, wrapperProps.className, { 'is-hovered': isHovered } ),
        style: { ...wrapperProps.style, ...props.style },
    };
}
```

`BlockComponent` itself is now just a thin wrapper that calls the hook and renders `<TagName { ...blockWrapperProps }>{ children }</TagName>`.

**Event listeners moved out of React props** — `onKeyDown`, `onMouseLeave` (selection), `onMouseOver`/`onMouseOut` (navigation hover) are all attached via `ref.current.addEventListener` inside separate `useEffect` calls (keyed on `isSelected` and `isNavigationMode` respectively) and cleaned up on teardown. This removes them from the returned props object, so they no longer collide with props passed by the block author.

**`data-align` omission moved upstream** — previously stripped inside `block-wrapper.js`, the `omit( wrapperProps, [ 'data-align' ] )` call is moved to `block.js` before `wrapperProps` is written into `BlockListBlockContext`, keeping the context clean.

**Exports** — `packages/block-editor/src/components/index.js` (and `.native.js`) now export:
```js
export {
    Block as __experimentalBlock,
    useBlockWrapperProps as __experimentalUseBlockWrapperProps,
} from './block-list/block-wrapper';
```

**React Native stub** — `block-wrapper.native.js` adds a no-op implementation that simply returns its `props` argument, preserving the existing native rendering path.

**`RichText` fix (incidental)** — `packages/block-editor/src/components/rich-text/index.js` stops spreading `...props` into the inner `<RichText>` component; instead it explicitly whitelists all known props (including `unstableOnFocus`, several mobile-only props, and `__unstableMultilineRootTag`). The remaining `{ ...props }` spread is moved down to the outer `<TagName>` element, so block-wrapper-level attributes land on the correct DOM node.

**Before / after block usage:**
```jsx
// Before
import { __experimentalBlock as Block } from '@wordpress/block-editor';
return <Block.div><audio src={ src } /></Block.div>;

// After
import { __experimentalUseBlockWrapperProps as useBlockWrapperProps } from '@wordpress/block-editor';
const blockWrapperProps = useBlockWrapperProps();
return <div { ...blockWrapperProps }><audio src={ src } /></div>;
```

When the block holds its own ref, pass it through the hook: `useBlockWrapperProps( { ref } )`.

## Contribution

Opened and merged by @ellatrix (commit `aa0abb9`). @ZebulanStanphill raised the naming question — `useBlockProps` vs `useBlockWrapperProps` — and @ellatrix expressed mild preference for the shorter form but deferred; the PR shipped under the longer name. @youknowriad proposed applying `style` and `className` purely via DOM APIs rather than as React props to sidestep the merging problem entirely; @ellatrix rejected this because the `blocks.getSaveElement.extraProps` (editor props) filter allows third-party plugins to inject arbitrary props that must reach the React element. @youknowriad also flagged the residual risk that third-party-injected event handlers won't be automatically merged by the hook, flagging it as a known limitation rather than a blocker for merging.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
