# #82314: Tab trap escape hatch: leave/enter the canvas with Escape/Enter

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`
- **Merged:** [`a584324`](https://github.com/WordPress/gutenberg/commit/a584324f40fc1565882cea5d6838192df2b63d35)
- **Discussion:** [#82314](https://github.com/WordPress/gutenberg/pull/82314) · 3 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

The block editor canvas is now a single, named tab stop. Pressing Escape inside the canvas moves focus to a hidden `role="button"` element labelled "Editor canvas" (with a "Press Enter to edit the document" hint) while leaving the block selection intact. From that stop, Tab skips past the canvas to the sidebar, Shift+Tab reaches the block toolbar, and Enter, Space, F2 or Escape return focus to the remembered caret. This gives keyboard users an escape hatch when Tab is consumed by the content (indent, CodeMirror-style embedded editors, etc.).

## Impact

- **Editor users / keyboard and screen reader users:** Escape now does something in the editor. The first Escape steps out onto the canvas stop. With NVDA/JAWS in typing mode, an extra Escape is needed first to leave that mode. Tab/Shift+Tab arriving from outside the canvas still land directly back in the content, because focus arriving on the stop engages the canvas. Escape still undoes an eligible automatic change (e.g. `* ` list transform) first, without stepping out.
- **Plugin and theme developers:** No API removals or new public hooks. Third-party blocks that embed Tab-trapping editors now have a standard way out. E2E tests that locate controls by accessible name may need updates. The PR's own tests changed `Edit` to `Edit link` for the link button, and added a new `Editor canvas` button to the accessibility tree.
- **Custom editor integrations** (site editor, widgets screen, customizer) use the same `useWritingFlow` and get the behavior automatically. Code that relied on Escape bubbling from the canvas or on the old focus-capture divs should be re-tested.
- **Hosting/headless/REST:** Not affected.

## Technical details

**`use-tab-nav.jsx`**
- The `before` focus-capture div is now an interactive stop: `tabIndex="0"`, `role="button"`, `aria-label="Editor canvas"`, `aria-describedby` pointing at a hint child (`Press Enter to edit the document`, id via `useInstanceId`), class `block-editor-writing-flow__canvas-stop`. It handles `onFocus`, `onKeyDown` and `onClick` (screen readers activate buttons via click).
- The old `onFocusCapture` logic is extracted into `enterCanvas()`. It first calls `ownerDocument.defaultView.focus()` (a Firefox workaround for cross-document focus), then restores focus: multi-selection, the selected block/last focus, section root handling in zoom-out, or the first tabbable, falling back to focusing the container if nothing is tabbable.
- `ENTRY_KEYS = ['Enter', ' ', 'F2', 'Escape']` on the stop call `enterCanvas()`. Plain Tab on the stop calls `focus.tabbable.findNext( focusCaptureAfterRef.current )`.
- Escape handler in the canvas keydown listener: if unmodified, it returns early when `getEditedContentOnlySection()` is truthy (so the parent-document handler ends section editing first). Otherwise it sets `noCaptureRef.current = true` and focuses the stop, parking there without re-engaging the canvas. `onStopFocus` and the after-canvas `onFocusCapture` both skip `enterCanvas()` when that flag is set.
- The keydown handler is wrapped in `withIgnoreIMEEvents` so Escape/Tab during IME composition are not intercepted.
- `PREVENT_SCROLL_ON_FOCUS` gains `outline: 'none'`.

**New `use-undo-automatic-change.js`** replaces the deleted `rich-text/event-listeners/undo-automatic-change.js` (also removed from `allEventListeners`). It is a `useRefEffect` hook registered first in `useWritingFlow`'s `useMergeRefs`, so its Backspace/Escape keydown listener runs before tab-nav's on the same node and claims Escape when `didAutomaticChange()` and `__experimentalUndo` are available.

**`writing-flow/style.scss`** (new, `@use`d from `block-editor/src/style.scss`): focus ring drawn around the canvas (an `::after` overlay for the iframed `.block-editor-iframe__scale-container`, because Safari paints an iframe outline under its content; an outline on the sibling `.block-editor-writing-flow` otherwise) and a tooltip-like sticky hint shown only while the stop is focused. Bundle impact reported: ~+321 B JS, ~+174 B min CSS.

**Tests:** a new "Canvas as a single tab stop" group in `keyboard-navigable-blocks.spec.js`, plus changes to `list`, `toolbar-roving-tabindex`, `buttons`, site-editor navigation and customizing-widgets specs (the list spec asserts that the `Editor canvas` button is not focused after Escape undoes an automatic change). The diff is truncated, so the remaining spec changes are not fully visible.

## Contribution

Opened by @ellatrix as an alternative to #82310, motivated by the loss of Escape-to-navigation-mode (#72193) and the overloading of Tab (indent vs. moving focus), which she described as blocking several list and code-block Tab-indent iterations (including #82184). The design cites GitHub's editor and CodeMirror's Escape-then-Tab escape hatch as prior art, and the description addresses the objection that Escape already exits typing mode in JAWS/NVDA (two presses instead of one). The PR was tagged as written with AI assistance. CodeRabbit's automated review produced no actionable comments, and the thread contains no human review discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
