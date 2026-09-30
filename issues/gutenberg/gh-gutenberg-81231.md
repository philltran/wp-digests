# #81231: Block editor: render a real default block in place of the default appender

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Package] Block editor`
- **Merged:** [`c98049d`](https://github.com/WordPress/gutenberg/commit/c98049dbf4ff661450b23e721f42eb7642a42330)
- **Discussion:** [#81231](https://github.com/WordPress/gutenberg/pull/81231) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

When a block list is empty, the block editor now renders a real "ghost" instance of the container's default block (a paragraph on the empty canvas, a button inside an empty Buttons block) instead of the `<p role="button">` default appender lookalike. The ghost is a `createBlock()` object rendered before it exists in the store, and it is inserted unchanged, with the same client ID, DOM element and caret, when the user clicks or focuses it. Containers can now influence the empty state through the block list settings' `defaultBlock`, including its `attributes`.

## Impact

- **Plugin & theme developers (block authors):** Blocks that declare a `defaultBlock` in their block list settings (e.g. via `useInnerBlocksProps` / `InnerBlocks`) now see it rendered in the empty state, with its attributes. Blocks with a custom appender (`renderAppender`) are unaffected.
- **Editor customizations / tests:** The empty state is no longer `role=button`. It is a `role=document` element labelled "Add default block" until entry. Custom CSS or e2e tests targeting the default appender in the empty canvas need updating. The core e2e specs were changed from `role=button[name="Add default block"i]` to `role=document[name="Add default block"i]`.
- **Block authors using `PrivateBlockContext.ariaLabel`:** The paragraph block no longer reads `ariaLabel` from `PrivateBlockContext` and passes `aria-label` to `RichText`. It now passes it through `useBlockProps`. A block that wants the ghost label to be overridden should route its label via `useBlockProps`.
- **Behavioral caveat:** Per the PR, selecting an empty restricted container (e.g. Buttons) moves focus into the ghost, which materializes it, so selecting the container creates its first child.
- **Site owners / end users:** The empty post body shows a real empty paragraph with the usual placeholder. The post has no content until the user enters the ghost. No migration is needed.

## Technical details

**`block-list/index.js` (`Items`)**
- While the list is empty, `hasAppender` is true and there is no custom appender, the default block is resolved from `getBlockListSettings( rootClientId )?.defaultBlock`, falling back to `{ name: getDefaultBlockName() }`. It must pass `canInsertBlockType`.
- The selector's appender conditions were factored into `appenderAllowed` (section-block check, editing mode not `disabled`, template lock absent or `contentOnly`, `hasAppender`, not zoom-out). The ghost renders whenever `appenderAllowed` holds, whether or not the block is selected. `shouldRenderAppender` still requires `hasCustomAppender`, `hasSelectedRoot` or `showRootAppender`.
- A memoized `createBlock( ghostBlockName, ghostBlockAttributes )` produces a ghost that is stable per empty period. Its client ID is appended to the rendered items, and `BlockListBlock` receives a `ghostBlock` prop. `AsyncModeProvider` is never async for the ghost. `BlockListAppender` is skipped when `showGhost` is true.

**`block-list/block.js` (`BlockListBlockProvider`)**
- `getBlockWithoutAttributes( clientId )` and `getBlockAttributes( clientId )` fall back to the ghost object (a memoized `{ clientId, name, isValid: true }` plus `ghostBlock.attributes`) when the store has no entry.
- `ghostBlock` and `rootClientId` are added to the private context.
- Unrelated optimization: `hasSelectedInnerBlock( sectionBlockClientId, checkDeep )` is now skipped when there is no section block.

**`use-block-props/index.js`**
- New `useGhostMaterialize( rootClientId, ghostBlock )` ref effect adds capture-phase native `pointerdown` and `focusin` listeners. On the first event it removes both listeners and calls `insertBlocks( [ ghostBlock ], undefined, rootClientId, true, 0 )`, inserting the same object so the client ID, key and element persist.
- `aria-label` now resolves as: `ariaLabel` → `__( 'Add default block' )` for the ghost → `props[ 'aria-label' ]` → `blockLabel`.

**Block library**
- `paragraph/edit.js`: the empty/non-empty `aria-label` moved from `RichText` into `useBlockProps( { 'aria-label': ... } )`, and the `PrivateBlockContext` import was dropped.
- `post-content/edit.js`: removed `renderAppender: InnerBlocks.DefaultBlockAppender` for empty content, since the ghost now provides the writing prompt.

No REST, DB, or new public hook changes. The PR notes a theoretical one-tick window on keyboard entry where a focus-time selection dispatch could precede the insert, but says selection sync rides the async `selectionchange` event, so no inconsistent state has been observed.

## Contribution

Authored by @ellatrix and merged after a small discussion thread (only bot comments: props list, bundle size +290 B, and two flaky e2e reports with `socket hang up` errors on unrelated specs). The PR names a follow-up that would replace the Cover block's scaffolded title paragraph with a declared `defaultBlock`, building on the discussion in #80028.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
