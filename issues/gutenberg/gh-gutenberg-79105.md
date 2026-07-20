# #79105: Add an `editableRoot` block support for native cross-block selection

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Enhancement`, `[Package] DOM`, `[Package] Components`, `[Package] Block library`, `[Package] Rich text`, `[Package] Block editor`, `Needs Dev Note`, `[Package] E2E Tests`
- **Merged:** [`3944b0e`](https://github.com/WordPress/gutenberg/commit/3944b0e352a25776f7e7081939f261d8c17e818d)
- **Discussion:** [#79105](https://github.com/WordPress/gutenberg/pull/79105) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Gutenberg introduces a new `supports.editableRoot` block support that promotes the editor canvas to a single `contentEditable` host when a block opts in. This enables the browser’s native selection engine to span across multiple blocks, solving persistent cross-block selection failures on iOS touch devices where synthetic pointer handling breaks. The `core/paragraph` block is the first to enable this support.

## Impact

- **Plugin & theme developers:** Extensions that attach event handlers via `editor.BlockEdit` or `editor.BlockListBlock` filters and rely on `event.target` or DOM focus residing on the block’s editable element may break. When `editableRoot` is active, keyboard, input, and clipboard events target the wrapper ancestor instead.
- **Block authors:** No immediate action required, but blocks that depend on `event.target` or explicit DOM focus for their own logic should avoid declaring `supports.editableRoot`.
- **Site owners & editors:** Improved cross-block text selection on touch devices (especially iOS) with no configuration needed.
- **No action required** for standard block usage or existing themes/plugins that don’t manipulate block-level event targets.

## Technical details

- Adds `editableRoot` to the block supports schema (`docs/reference-guides/block-api/block-supports.md`) and enables it on `core/paragraph` (`docs/reference-guides/core-blocks/README.md`).
- The `useEditableRoot` hook manages the wrapper’s `contentEditable` state, flipping it only on block-selection changes to avoid mid-keystroke focus swaps.
- Event handling shifts from `compose`’s `subscribeDelegatedListener` to `richText`’s `subscribeOwnedListener` and `ownsSelection` private APIs. This is visible across `packages/block-editor/src/components/rich-text/event-listeners/*.js` (e.g., `before-input-rules.js`, `delete.js`, `enter.js`, `input-events.js`, `paste-handler.js`).
- `use-focus-handler.js` now checks `isShiftClickInProgress()` and `isBlockMultiSelected()` to prevent focus events from collapsing native cross-block selections, and dispatches `selectBlock( clientId, null )` for editable targets to avoid caret placement interference.
- `use-focus-first-element.js` uses `getSelectionStart()` and checks `activeElement?.isContentEditable` to avoid overwriting a deliberate caret when the wrapper holds focus.
- **Before/After pattern (event ownership):**
  ```javascript
  // Before (delegated, target-based)
  const { subscribeDelegatedListener } = unlock( composePrivateApis );
  return subscribeDelegatedListener( element, 'beforeinput', onInput, true );

  // After (owned, selection-based)
  const { subscribeOwnedListener } = unlock( richTextPrivateApis );
  return subscribeOwnedListener( element, 'beforeinput', onInput, true );
  ```

## Contribution

Opened and merged by @ellatrix with co-authorship from @ciampo. The PR addresses long-standing iOS touch selection limitations by testing an alternative approach (toggling `contentEditable` off mid-gesture) that failed on WebKit, leading to the final wrapper-as-single-host model. @t-hamano requested a Dev Note for 7.1. The author noted the change is opt-in and acknowledged potential backward-compatibility friction for `editor.BlockEdit` extensions, with plans to evaluate and potentially add a forwarding layer for React `on*` handlers.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
