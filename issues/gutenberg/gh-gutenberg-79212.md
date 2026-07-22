# #79212: Icon block: Default to core/info via block.json instead of an insert-time effect

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Icon`
- **Merged:** [`a0aa5da`](https://github.com/WordPress/gutenberg/commit/a0aa5da578771011af2f59bc676230c7d080c6f2)
- **Discussion:** [#79212](https://github.com/WordPress/gutenberg/pull/79212) · 13 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Icon block now sets its default icon (`core/info`) declaratively via a block variation instead of applying it imperatively through an insert-time `useEffect`. This eliminates a render flash and race condition, removes the need to mark the change as non-persistent for undo history, and ensures existing or explicitly empty blocks are never silently altered when loading a post.

## Impact

- **Plugin & theme developers:** No breaking API changes. The visual default for a freshly inserted Icon block is now `core/info` instead of a placeholder. Existing blocks saved without an `icon` attribute will render the info icon in the editor and frontend.
- **Content editors / site owners:** New Icon blocks will immediately show the info icon without a flash. To restore the previous placeholder state, explicitly set `icon: ""` in the block attributes.
- **No action required** for standard implementations, but audit any custom block rendering or patterns that relied on the previous empty-icon placeholder behavior.

## Technical details

The imperative side-effect in `packages/block-library/src/icon/edit.js` was removed, along with `useDispatch`, `wasBlockJustInserted`, and the `clientId` prop from the `Edit` component. Instead, `packages/block-library/src/icon/variations.js` now exports a `default` variation with `isDefault: true` and `attributes: { icon: 'core/info' }`, which is registered in `packages/block-library/src/icon/index.js`. The `IconPlaceholder` component remains active for blocks where the `icon` attribute is explicitly set to an empty string.

Before/after pattern for the edit component:
```jsx
// Before
export function Edit( { attributes, setAttributes, clientId } ) {
  const wasJustInserted = useSelect( ( select ) => select( blockEditorStore ).wasBlockJustInserted( clientId ), [ clientId ] );
  useEffect( () => {
    if ( ! icon && wasJustInserted ) {
      __unstableMarkNextChangeAsNotPersistent();
      setAttributes( { icon: 'core/info' } );
    }
  }, [ icon, wasJustInserted, setAttributes, __unstableMarkNextChangeAsNotPersistent ] );
}

// After
export function Edit( { attributes, setAttributes } ) {
  // Side-effect removed; default applied via block variation
}
```

## Contribution

Opened by @Mamaduka as a follow-up to #79111, the PR replaced an imperative insert-time effect with a declarative block variation to fix a render race condition and simplify the codebase. During review, @jasmussen and @t-hamano discussed whether to maintain a placeholder material for empty blocks and whether a UI "Reset" action was needed; the team agreed to follow the Button block pattern (no reset button) and keep `IconPlaceholder` for explicitly empty attributes. The approach was merged after a brief review cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
