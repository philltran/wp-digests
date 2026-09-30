# #74040: Format library: visualise non-breaking spaces

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Enhancement`, `[Package] Format library`, `[Package] Block editor`
- **Merged:** [`e66edb0`](https://github.com/WordPress/gutenberg/commit/e66edb016f97f5b281cf44247ff42d0f380736bb)
- **Discussion:** [#74040](https://github.com/WordPress/gutenberg/pull/74040) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The block editor now visually marks non-breaking space characters (U+00A0) in rich text. Each NBSP is wrapped in an editor-only `span.non-breaking-space` format that renders with a thin outline while its block is selected, and a popover labelled "Non-breaking space" appears when the selection is exactly one NBSP. This makes pasted or auto-inserted NBSPs (e.g. from MS Word) discoverable, since they otherwise cause unexplained early line wraps.

## Impact

**Site owners / editors**
- NBSPs inserted via paste, browser behavior, or the `Shift+Primary+Space` shortcut are now visible in selected blocks, making it easier to diagnose odd line breaks.

**Plugin & theme developers**
- No action required. The format is editor-only and, per the PR, is not serialized, so saved markup and front-end output should be unchanged.
- Editor-side CSS targeting `.non-breaking-space` or custom code inspecting the rich-text DOM/formats may now encounter the extra `span` wrappers inside editable content.
- The `core/non-breaking-space` format type's `tagName` changed from `nbsp` to `span` and `className` from `null` to `non-breaking-space`. Code that referenced the old `nbsp` tag for this format would be affected.

**Headless & REST consumers**
- No change to stored content.

## Technical details

Changes are in `packages/format-library/src/non-breaking-space/index.jsx`, `packages/block-editor/src/components/rich-text/content.scss`, and an ESLint suppression entry.

- The `core/non-breaking-space` format now uses `tagName: 'span'` and `className: 'non-breaking-space'` (previously `tagName: 'nbsp'`, `className: null`).
- It adds `__experimentalCreatePrepareEditableTree()`, returning a function `(formats, text)` that scans `text` for every `\u00a0` and calls `applyFormat( record, { type: name }, index, index + 1 )`, returning the resulting `formats`. This decorates the editable tree at render time rather than mutating stored value, which is how the format stays out of serialization.
- `edit()` computes `selectedValue` from `value.text.slice( value.start, value.end )` and, when it equals `'\u00a0'`, renders a new `PopoverAnchor` component. That component uses `useAnchor( { editableContentElement: contentRef.current, settings: nonBreakingSpace } )` and renders a `Popover` with the text "Non-breaking space". The existing `RichTextShortcut` (`primaryShift` + space) is retained.
- `content.scss` adds:

```scss
.is-selected .non-breaking-space {
	outline: 1px solid currentColor;
	outline-offset: -1px;
	opacity: 0.8;
}
```

- `tools/eslint/suppressions.json` adds two `react-hooks/refs` suppressions for the file (from reading `contentRef.current` during render).
- Bundle impact reported by CI: about +321 B total, mostly in `format-library/index.min.js`.

Note: `edit()` guards with `value.start && value.end`, which is falsy when the offset is `0`, so an NBSP at the very start of a block would not trigger the popover. This is inferred from the diff, not discussed in the PR.

## Contribution

Opened by @ellatrix as a follow-up to issue #72232, in which a user reported invisible pasted spaces from MS Word. Reviewers @dmsnell and @jhmonroe suggested framing this around "hidden/invisible characters" generally rather than only NBSP, since the same logic could apply to non-breaking hyphens and narrow no-break spaces. @jasmussen offered design help after the PR sat for a while, and @ellatrix merged it with "Let's just get it in and iterate," leaving the broader hidden-character question for follow-ups.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
