# #75353: List Block: Preserve ordered type on indent

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mustafabharmal
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] List`, `Backported to WP Core`
- **Merged:** [`3182e27`](https://github.com/WordPress/gutenberg/commit/3182e27321feedc0f9f063e98e1c4d540979698f)
- **Discussion:** [#75353](https://github.com/WordPress/gutenberg/pull/75353) · 5 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Indenting a list item in the List block now correctly preserves the parent list's type (ordered vs. unordered) instead of defaulting to an unordered list. This fixes a regression introduced in Gutenberg 14.0 where nested lists always became bulleted regardless of the parent's formatting. The change ensures consistent list hierarchy when using the indent button or Tab key.

## Impact

- **Block & theme developers:** No code changes required. The `core/list` block's `ordered` attribute is now correctly propagated to nested lists during editor indentation.
- **Site owners & content editors:** Indenting ordered lists will now produce nested ordered lists, matching expected behavior and preventing accidental formatting breaks.
- **No action required** for existing content; this only affects new indentation actions in the block editor.

## Technical details

The fix modifies the `useIndentListItem` hook in `packages/block-library/src/list-item/hooks/use-indent-list-item.js`. Previously, when a list item was indented and lacked inner blocks, the hook cloned the sibling and created a new `core/list` block without attributes:
```js
newListItem.innerBlocks = [ createBlock( 'core/list' ) ];
```
The updated diff retrieves the parent list's client ID via `getBlockRootClientId` and fetches its attributes using `getBlockAttributes` from `blockEditorStore`. It then passes the parent's `ordered` value to the new nested block:
```js
const parentListId = getBlockRootClientId( clientId );
const parentListAttributes = getBlockAttributes( parentListId );
newListItem.innerBlocks = [
    createBlock( 'core/list', {
        ordered: parentListAttributes.ordered,
    } ),
];
```
An e2e test was added to `test/e2e/specs/editor/blocks/list.spec.js` to verify that indenting an ordered list item outputs `<!-- wp:list {"ordered":true} -->` in the serialized content.

## Contribution

Opened by @Mustafabharmal and co-authored by @ellatrix, @felipevelzani, @threadi, and @skorasaurus, the PR closed two related issues tracking the regression. Reviewer @t-hamano initially noted that automatic cherry-picking might fail due to the forked submission, but the system successfully backported the commit to `wp/7.1`. The approach was straightforward—simply reading the parent block's `ordered` attribute during indentation—and no alternative implementations were debated.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
