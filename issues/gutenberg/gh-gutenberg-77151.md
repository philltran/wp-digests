# #77151: Quote: Ensure paragraph placeholder appears after deleting nested blocks

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @dpmehta
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Quote`, `[Package] Block editor`, `First-time Contributor`, `Backported to WP Core`
- **Merged:** [`9376767`](https://github.com/WordPress/gutenberg/commit/9376767645abd0d9f6855bf7543e5c2f8b2701c7)
- **Discussion:** [#77151](https://github.com/WordPress/gutenberg/pull/77151) · 8 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Quote block now conditionally renders its default block appender when it contains no inner blocks, restoring the paragraph placeholder after nested blocks are deleted. Previously, `renderAppender` was hardcoded to `false`, which caused the editor to lose focus in the citation field when the last inner block was removed. This change ensures the block reverts to a selectable paragraph state, improving the editing flow for container blocks.

## Impact

- **Block & theme developers:** No breaking changes. The `renderAppender` prop behavior for the Quote block is now dynamic rather than static.
- **Site owners & editors:** Improved UX when deleting nested blocks inside a Quote; the editor now correctly shows a placeholder appender instead of trapping focus in the citation field.
- **Platform/Hosting:** No configuration or migration required. The change is fully backward compatible and requires no action.

## Technical details

The diff modifies two core files to restore the empty-state appender:

- `packages/block-library/src/quote/edit.js` replaces the static `renderAppender: false` with a conditional check. It introduces a `useSelect` hook querying `getBlockCount( clientId ) > 0` from `blockEditorStore` to determine `hasInnerBlocks`.

```javascript
// Before
renderAppender: false,

// After
const { hasInnerBlocks } = useSelect(
  ( select ) => {
    const { getBlockCount } = select( blockEditorStore );
    return { hasInnerBlocks: getBlockCount( clientId ) > 0 };
  },
  [ clientId ]
);
renderAppender: hasInnerBlocks ? false : undefined,
```

- `packages/block-editor/src/components/default-block-appender/content.scss` updates the selector from `&:only-child` to `&:first-child`. The updated comment clarifies that the appender always renders after inner blocks, so being the first child indicates an empty container (non-block siblings like the citation may still exist).
- An e2e test in `test/e2e/specs/editor/blocks/quote.spec.js` verifies that backspacing the last inner block reveals the appender and allows refilling the quote with a paragraph.

## Contribution

The PR addresses a focus-trap bug introduced when the appender was previously disabled. During review, @Mamaduka noted the original removal was intended to prevent layout shifts, while @jasmussen raised the broader question of whether container blocks should share a generic empty-state solution rather than bespoke logic. @ellatrix approved the targeted fix as a temporary measure, with plans to explore a more universal approach for the default appender in a future release. The change was subsequently cherry-picked to the `wp/7.1` branch for inclusion in the next WordPress release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
