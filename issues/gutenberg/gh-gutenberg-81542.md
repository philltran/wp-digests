# #81542: Re-enable editableRoot for multi-selection on mobile web

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Enhancement`, `[Feature] Rich Text`, `[Package] Block library`, `[Package] Rich text`, `[Package] Block editor`
- **Merged:** [`cd5c8d0`](https://github.com/WordPress/gutenberg/commit/cd5c8d0d3b1bbdc3ea4abd571969527842ff6ec3)
- **Discussion:** [#81542](https://github.com/WordPress/gutenberg/pull/81542) · 5 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

This PR re-enables the private `editableRoot` opt-in for the core `paragraph` block, which was pulled from the WordPress 7.0 RC in #81184. With the opt-in on, once a selected paragraph has siblings the canvas wrapper (the iframe `body`) becomes the `contenteditable` editing host. The iOS double-tap problem that forced the removal was fixed separately in #82581. The PR also fixes four latent trunk bugs in focus, selection and wrapper-cleanup handling that surfaced once the host was engaged.

## Impact

- **Site owners / editors:** No configuration needed. Typing, merging and cross-block selection are intended to behave as before. The editing host changes how selection is handled, so mobile web and cross-block selection are the areas to watch.
- **Plugin & theme developers:**
  - No public API changes. The opt-in is a private `Symbol`-keyed block setting (`editableRootKey` from `@wordpress/blocks` private APIs), not a public `supports` key.
  - Code that adds handlers through `wrapperProps` on `editor.BlockListBlock` (e.g. `onKeyDown`, `onInput`, `onBeforeInput`) should keep working for hosted paragraphs. The new compat e2e tests cover delivery, `preventDefault`, `stopPropagation` and a working `nativeEvent`.
  - Code or tests that assume a selected paragraph's own element is the focused/`contenteditable` node may now see the canvas `body` as the host. The PR itself had to change a `toBeFocused()` assertion in `list-view.spec.js` to `editor.ownsSelection()`.
  - Tooling that inspects ARIA state on the canvas may see the host expose `role="textbox"`, `aria-multiline` and `aria-label="Editor canvas"`.
- **Performance:** The PR notes that typing metrics are expected to rise. With the canvas body as host, Chrome rebuilds its text input state after each insertion, which the author reports adds about 40% per keystroke on the large post fixture (1436 blocks). The author considers this still performant, and it depends on #80605, which is already merged.
- **Headless/REST:** Not affected.

## Technical details

From the diff:

- **Opt-in:** `packages/block-library/src/paragraph/index.js` imports `privateApis` from `@wordpress/blocks`, unlocks `editableRootKey`, and adds `[ editableRootKey ]: true` to the block `settings`. Only `paragraph` opts in; an e2e test asserts a `heading` is not hosted.
- **`block-list/block.jsx`:** In the first-child move cleanup, `getBlock( _clientId )` is now read once into `wrapper`. `removeBlock( _clientId, false )` only runs if `wrapper` still exists, is empty, and passes `isUnmodifiedBlock( wrapper, 'content' )`. This avoids acting on a wrapper the store already removed.
- **`writing-flow/use-selection-observer.js`:** When disengaging (`setContentEditableWrapper( node, false )`), focus is returned to the nearest `[contenteditable]` only if `ownerDocument.activeElement === node && ownerDocument.hasFocus()`. This stops the observer pulling focus back into the canvas after Escape moved it to the canvas stop in the parent document.
- **`rich-text/index.jsx` (`RichTextWrapper`):** The focus `useLayoutEffect` guard changes from `! element` to `element?.contentEditable !== 'true'`. A field made non-editable during a pointer press (see rich text's `preventFocusCapture`) is no longer focused, which previously made the block focus handler drop the text selection.
- **`rich-text/.../input-and-selection.js`:** The `window.queueMicrotask( handleSelectionChange )` after focus inside an editing host is removed. The removed comment said the caret placed after focus had to be synchronized in the same task. The PR description says this fixes a stale range being reported.
- **Tests:** New `editable-root.spec.js` and `editable-root-compat.spec.js`. Updates to `autocomplete-and-mentions`, `list-view`, `multi-block-selection` and `splitting-merging` specs. The autocomplete test checks that `aria-autocomplete`, `aria-haspopup`, `aria-controls`, `aria-owns` and `aria-activedescendant` are mirrored onto the host and cleared (omitted, not `null`) on Escape. The multi-block spec now expects the host to keep `contenteditable`, `role="textbox"`, `aria-multiline` and `aria-label` across a cross-block selection.

The diff was truncated, so the remaining spec changes are not described here.

## Contribution

Authored by @ellatrix with Claude (per the AI-use disclosure) and merged to trunk. The opt-in was first removed in #81184 because the behavior wasn't ready for the 7.0 RC. It was restored once the iOS double-tap fix (#82581) and the typing-performance work (#80605) had landed. The performance job reports an expected typing regression, which the author explicitly accepted. CodeRabbit raised no actionable comments and listed `ciampo` as a suggested reviewer; the discussion also shows a few flaky e2e failures that passed on retry.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
