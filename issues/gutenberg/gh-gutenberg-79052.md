# #79052: Math format: seed LaTeX input from the current selection

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Enhancement`, `[Package] Format library`, `[Block] Math`
- **Merged:** [`a2ed6c2`](https://github.com/WordPress/gutenberg/commit/a2ed6c22f19242c6c1e3ba879875de4383a783fe)
- **Discussion:** [#79052](https://github.com/WordPress/gutenberg/pull/79052) · 11 comments · 0 reactions

## Summary

Previously, applying the Math inline format ignored the editor’s active selection and opened an empty popover input. This enhancement seeds the LaTeX input field directly from any selected text, eliminating unnecessary manual entry. Clicking the toolbar button a second time now toggles the active math object back to its raw LaTeX source, preserving the text selection for immediate editing or re-formatting.

## Impact

- **Editor users:** The Math format popover auto-populates with selected text instead of opening blank. Toggling an active math format now reverts to plain LaTeX text rather than silently clearing the object.
- **Plugin/Theme developers (Format Library):** No API breaking changes. Developers extending the math format or intercepting rich-text operations should account for the updated selection state and `onClick` toggle behavior.
- **Headless/REST consumers:** Unaffected; server-side rendering and `data-latex` attribute persistence remain unchanged.
- *No action required* unless you programmatically manipulate the Math format's rich-text attributes or parse active object states.

## Technical details

- Updated `packages/format-library/src/math/index.js`, modifying the `onClick` handler in the `Edit` component.
- Imported `insert` from `@wordpress/rich-text` to handle text replacement alongside the existing `insertObject`.
- When `isObjectActive` is true, the handler now calls `insert(value, latex)` to revert `<math data-latex="...">` elements back to their raw string. It explicitly calculates `newValue.start = newValue.end - latex.length`, triggers `onChange()` and `onFocus()`, then returns early.
- On initial application with a selection, it extracts text via `value.text.slice(value.start, value.end)`, seeds the `'data-latex'` attribute (previously hardcoded to `''`), and computes a preview `innerHTML` via `latexToMathML(selectedText, { displayMode: false })`.
- Added an E2E test in `test/e2e/specs/editor/various/format-library/math.spec.js` validating the selection seeding, toggle reversion, and persistent selection state.

## Contribution

Opened by @jasmussen to address #73035. Review feedback from @Mamaduka and @ellatrix focused on the UX of toggling an active math format: whether clicking the button again should clear it or revert to raw LaTeX. The implementation was adjusted to revert to the LaTeX source while keeping the text selected, finalizing the behavior reflected in the merged diff (`a2ed6c2`).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
