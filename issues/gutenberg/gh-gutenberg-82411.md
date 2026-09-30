# #82411: List: indent and outdent multi-selected items with Tab

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] List`
- **Merged:** [`6d17486`](https://github.com/WordPress/gutenberg/commit/6d174863b068f386bac45ff82c83d002bb5bb214)
- **Discussion:** [#82411](https://github.com/WordPress/gutenberg/pull/82411) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

In the Gutenberg editor, Tab and Shift+Tab now indent and outdent a multi-selected set of `core/list-item` blocks, matching what already worked for a single item. Previously the indent/outdent toolbar buttons acted on the whole selection but the keys did nothing during a multi-selection. The items stay selected afterward, so the keys can be pressed repeatedly to move them further. If the selection cannot be indented or outdented, the key is left alone and the canvas Tab handling moves focus out as before.

## Impact

- **Site owners / editors:** Keyboard-only users can now indent or outdent several list items at once without reaching for the toolbar.
- **Plugin & theme developers:** No public API, hook, or block.json change. The internal list-item hooks `useIndentListItem`, `useOutdentListItem`, and the `moveBlocksToNestedList` helper were removed from `packages/block-library/src/list-item/` in favor of shared utilities. These are not exported public APIs, but code that copied or patched those private files will need to adjust.
- **Hosting / headless:** Not affected; editor-side UI only.
- **Action required:** None. Ships with the Gutenberg plugin; bundle size for `block-library/index.min.js` grows by about 122 B.

## Technical details

**New hook `useMultiSelectTab( clientId )`** (`list-item/hooks/use-multi-select-tab.js`), wired into `ListItemEdit` via `useMergeRefs` alongside `useEnter` and `useSpace`.

- A `useSelect` computes `isActive`: true only when `hasMultiSelection()` is true, the first multi-selected client ID equals this item's `clientId`, and every selected block is `core/list-item`. So only the first item's instance attaches a listener, and only while such a selection exists.
- When active, it adds a **capture-phase `keydown` listener on `element.ownerDocument`**, since during multi-selection focus sits on the writing-flow container rather than inside an item. Capture ordering lets it run before writing-flow handlers that gate on `event.defaultPrevented`.
- The handler ignores non-Tab keys, already-prevented events, and Alt/Meta/Ctrl combos. It also bails unless `event.target.contains( element )`, so Tabs coming from toolbars or panels while a multi-selection lingers are not hijacked.
- Shift+Tab calls `outdentListItems( registry )`, Tab calls `indentListItems( registry )`; `preventDefault()` is called only if the function returns truthy, so unactionable presses fall through to the canvas Tab stop.

**Refactor into `list-item/utils.js`:** the diff shows `indentListItems( registry, clientId? )`, `outdentListItems( registry, clientIds? )`, `getIndentTarget( select, clientId )`, and `getOutdentTarget( select, clientId )` being imported; the file itself is truncated from the diff, so its internals are not described here. Per the PR, selection is restored after the move (caret for a single item, `multiSelect` across the moved blocks for several), and the deleted `use-indent-list-item.js` shows the prior batched move-then-reselect logic.

**Call-site changes:** `IndentUI` now derives `canIndent`/`canOutdent` from `getIndentTarget`/`getOutdentTarget` and calls the utilities with `useRegistry()`. `useEnter`, `useMerge`, and `useSpace` read the store through `registry.select/dispatch` at event time instead of `useSelect`/`useDispatch` hooks. `use-merge.js` replaces its local `getParentListItemId` with `getOutdentTarget`. `useEnter`'s effect deps change from `[]` to `[ clientId, registry ]`.

Covered by new e2e tests in `test/e2e/specs/editor/blocks/list.spec.js`.

## Contribution

Authored by @ellatrix and built on the earlier #82314, which freed Tab inside the canvas to mean indentation where sensible. The PR discloses AI assistance in writing. CodeRabbit's review generated no actionable comments (it only flagged low docstring coverage), and the record shows no notable design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
