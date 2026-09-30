# #81042: Rich text: fix iOS double tap selection, preventing focus capture under editable host

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Rich text`, `[Package] Block editor`
- **Merged:** [`456a7d9`](https://github.com/WordPress/gutenberg/commit/456a7d93ebe41f134ec1c066ddc5f242841b2059)
- **Discussion:** [#81042](https://github.com/WordPress/gutenberg/pull/81042) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes iOS double-tap word selection in paragraphs when the `editableRoot` editing host is active. The rich text element previously retained its own `contenteditable="true"` attribute and a `tabIndex`, making it a nested focusable editable. On iOS, each tap focused that nested element, the host code immediately moved focus back to the wrapper, and the resulting focus flip cancelled the native word-selection gesture before it could form. The fix removes both attributes from the rich text element under the host and adds a `focus()` override that places the caret in the element and focuses the host instead.

## Impact

- **Site owners / mobile editors:** Double-tap-to-select-word now works reliably on iOS when the editing host is active. No configuration or code changes required.
- **Plugin & theme developers:** No public API change. However, any code that detects rich text elements by checking for the `contenteditable` attribute will no longer match the selected block's field under the editing host (the attribute is removed, not set to `"inherit"`). Use the `data-wp-rich-text-attributke-key` attribute or the block selector instead.
- **No action required** for the vast majority of developers; the change is internal to the block editor's focus and selection management.

## Technical details

The core change is in `packages/block-editor/src/components/rich-text/index.jsx`. When `isEditingHost` is true, the rich text element now renders with `contentEditable={undefined}` (attribute absent) and `tabIndex={null}` (attribute removed) instead of `contentEditable={true}` and `tabIndex={0}`:

```jsx
// Before
contentEditable={ ! shouldDisableEditing }
tabIndex = props.tabIndex ?? 0;

// After
contentEditable={ isEditingHost ? undefined : ! shouldDisableEditing }
tabIndex = null; // when isEditingHost
```

A new `focusUnderHostRef` (a `useRefEffect`) overrides `element.focus()` so that calling `.focus()` on the inert field collapses the selection into the element and then focuses the nearest `[contenteditable="true"]` ancestor (the host), with a Gecko-specific workaround to restore the selection range after the host takes focus.

`isEditingHost` was broadened: it now returns `true` not only for the selected block that can host an editable root, but also for any block in a multi-selection (`isBlockMultiSelected( clientId )`), since those fields are inside the host's range and must not be independent editing areas.

Downstream adjustments across the writing-flow and block-list hooks:

- **`use-focus-first-element.js`** — `hasCaret` no longer requires `activeElement?.isContentEditable`; a new branch handles the inert-field case by collapsing the selection and calling `target.focus()` directly.
- **`use-focus-handler.js`** — early-returns when a click on the selected block's inert field would focus the wrapper (Firefox focuses the nearest focusable ancestor), preserving the existing text selection.
- **`use-click-selection.js`** — when clicking a block that can host an editable root, immediately removes the `contenteditable` attribute and calls `selectBlock` synchronously to avoid a re-render race that would drop the caret the browser just placed.
- **`use-drag-selection.js`** — the "is this a field" check changed from `target.getAttribute('contenteditable') !== 'true'` to `target.contentEditable === 'true' || (target.isContentEditable && target.dataset.block === getSelectedBlockClientId())`.
- **`use-arrow-nav.js`** (`getClosestTabbable`) — handles the case where the target is not in the focusable-nodes list (no `tabIndex` under the host) by using `compareDocumentPosition` to find focusables on the navigation side.
- **`typewriter/index.jsx`** — removed `getActiveEditableElement()` and `isLastEditableNode()`; `isSelectionEligibleForScroll()` now checks only whether the anchor node's element is `isContentEditable`.

## Contribution

Opened by @ellatrix and merged as `456a7d9`. @Mamaduka reviewed and noted the work; @ellatrix responded to a performance-metric observation about `mousedown` event timing. The PR was written with the assistance of Claude Code. No significant design debate or rejected alternatives are visible in the five comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
