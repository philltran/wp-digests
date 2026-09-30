# #82816: Enable editableRoot for the list item block for multi-selection on mobile

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Package] Rich text`, `[Package] Block editor`
- **Merged:** [`d83b5cd`](https://github.com/WordPress/gutenberg/commit/d83b5cd99252ffe93df8374e704e1273b4969106)
- **Discussion:** [#82816](https://github.com/WordPress/gutenberg/pull/82816) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The list item block now opts into the private `editableRoot` editing-host mechanism (the same setting the paragraph block received in #81542), enabling native touch selection to extend across multiple list items on mobile. The PR also reverts the approach from #81042 that stripped `contenteditable` and `tabindex` from hosted fields, instead keeping the field editable and focusable so focus does not fall through to the `ul` or a parent container. An iOS-specific guard prevents the mousedown handler from cancelling double-tap word selection.

## Impact

- **Mobile web editors (all users):** Multi-selection across sibling list items now works via native touch gestures. Previously the selection was constrained to a single item.
- **Block editor DOM structure:** Under an engaged editing host, the rich-text field now retains `contenteditable="true"` and `tabindex="0"` (previously the attribute was removed and `tabindex` was set to `null`). E2E tests or plugins that assert the field is non-focusable under the host will need updating.
- **Plugin & theme developers:** No public API change. The `editableRoot` key remains a private block setting. No action required unless you inspect the editor's internal DOM for list items.
- **Desktop editors:** Click, drag, and shift+click multi-selection behavior is unchanged.

## Technical details

**Opt-in.** `packages/block-library/src/list-item/index.js` sets the private `editableRoot` key on the block definition, mirroring the paragraph block.

**Editing-host eligibility simplified.** In `packages/block-editor/src/components/rich-text/index.jsx`, the `isEditingHost` selector no longer checks `isBlockMultiSelected(clientId)` for non-selected blocks. It now returns `false` early unless `isBlockSelected` is true, then calls `canHostEditableRoot(getSelectedBlockClientId())`. The `clientId` dependency is removed from the `useSelect` deps array.

**Field stays editable and focusable under the host.** The `contentEditable` prop is now unconditionally `! shouldDisableEditing` (the previous `isEditingHost ? undefined : ! shouldDisableEditing` ternary is gone). `tabIndex` under the host is `props.tabIndex ?? 0` instead of `null`. The `focusUnderHostRef` (which overrode `element.focus()` to place the caret and focus the host) is removed entirely.

**Caret placement simplified.** In `use-focus-first-element.js`, the special branch that called `selection.collapse()` + `target.focus()` for non-focusable targets under the host is removed. The `hasCaret` check now requires `activeElement?.isContentEditable && activeElement.contains(target)` in addition to the anchor-node containment check.

**iOS mousedown guard.** In `use-click-selection.js`, a module-level constant detects iOS WebKit via `window.CSS?.supports?.( '-webkit-touch-callout', 'none' )`. A `pointerdown` listener records `event.pointerType`. In `mousedown`, if all of the following hold, `event.preventDefault()` is called: the platform is iOS, the pointer type is `touch`, the clicked block is the already-selected block, the wrapper is `contenteditable="true"`, and the wrapper is the `activeElement`. This prevents the focus flicker that cancels double-tap word selection.

**Drag-selection field detection.** In `use-drag-selection.js`, the check for whether the pointer is over a field changes from `target.contentEditable === 'true' || (target.isContentEditable && target.dataset.block === getSelectedBlockClientId())` to `target.getAttribute('contenteditable') !== 'true'`, because all descendants of the wrapper return `true` for the `contentEditable` property.

**Selection sync on keydown.** In `use-selection-observer.js`, `ensureMultiBlockSelectionSync` is now also called on `keydown` (in addition to copy, cut, and paste), so a Tab or Shift+Tab immediately after a shift+arrow cross-item selection reads a fresh store value.

**Focus handover tightened.** The selection observer's focus-handover condition now requires `getBlockClientId(activeElement) === collapsedClientId` instead of `activeElement.contains(selection.anchorNode)`, and the fallback branch for `activeElement === ownerDocument.body` is removed.

## Contribution

Opened and authored by @ellatrix, who noted the code was written with Claude Code and reviewed/tested by them. The PR references two prior pieces of work: #81542 (which introduced `editableRoot` for the paragraph block) and #81042 (which introduced the attribute-stripping approach this PR reverts). CodeRabbit flagged a low merge risk around keyboard navigation landing on the wrong element in edge cases and suggested restoring a fallback before merge. No significant design debate is visible in the 7 comments; the discussion is primarily automated review output.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
