# #78242: Notes: Support inline rich text (bold, italic, link, code)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] DataViews`, `[Feature] Notes`
- **Merged:** [`36d0797`](https://github.com/WordPress/gutenberg/commit/36d0797c0e0ed7a0a41e78a2656b07b02d92e9a9)
- **Discussion:** [#78242](https://github.com/WordPress/gutenberg/pull/78242) · 22 comments · 2 reactions
- **Usefulness:** 4/5

## Summary

The Notes sidebar in the block editor now supports inline rich text formatting (bold, italic, links, and code) instead of plain text. This replaces the previous `<textarea>` input with a standalone `RichTextControl` component, enabling collaborators to emphasize labels, reference identifiers, and add links directly within notes without a visible toolbar. The change decouples the input from the block-editor selection store to prevent focus and canvas selection conflicts.

## Impact

- **Plugin & theme developers**: No direct impact; this is an internal editor component change.
- **Headless & REST consumers**: Note content now includes basic HTML markup (`<strong>`, `<em>`, `<a>`, `<code>`) instead of plain text. Existing consumers parsing note content should account for standard HTML.
- **Editor users**: Notes input now behaves like a contenteditable field. Keyboard shortcuts (`⌘B`, `⌘I`, `⌘K`, backticks for code) apply formatting. Focus management has changed to prevent canvas selection shifts.
- **No action required** for most developers, but REST API consumers or custom note renderers should verify HTML parsing/rendering expectations.

## Technical details

- **`@wordpress/dataviews`**: Exposes the `RichTextControl` assembly via a new `packages/dataviews/src/private-apis.ts` file using `lock`/`unlock` patterns, allowing other packages to consume it outside of `DataForm`.
- **`packages/editor/src/components/collab-sidebar/note-form.js`**: Replaces `TextareaAutosize` with `RichTextControl` (unlocked from `@wordpress/dataviews`). Configured with `allowedFormats = ['core/bold','core/italic','core/link','core/code']` and `hideLabelFromVision`. Submit logic now checks for `undefined` results on failure to preserve drafts, and strips HTML via `__unstableStripHTML` for length validation.
- **`packages/editor/src/components/collab-sidebar/add-note.js` & `note-thread.js`**: Updated blur guards to account for format popovers (e.g., `⌘K` link UI) portaling outside the form. Focus targets changed from `textarea` to `[role="textbox"]`.
- **`packages/editor/src/components/collab-sidebar/style.scss`**: Styles scoped to `[role="textbox"]` for line-height, max-height, and inline format styling (`code`, `a`).
- **Server/REST**: No changes. The default comment `kses` allowlist already permits `strong`, `em`, `a`, and `code`. Output rendering via `RawHTML` remains unchanged.

**Before/After**:
```jsx
// Before
<TextareaAutosize id={inputId} value={inputComment} onChange={...} />

// After
const { RichTextControl } = unlock( dataviewsPrivateApis );
<RichTextControl
  id={inputId}
  label={labels?.input ?? __('Note')}
  hideLabelFromVision
  value={inputComment}
  onChange={setInputComment}
  allowedFormats={ALLOWED_NOTE_FORMATS}
/>
```

## Contribution

Opened by @adamsilverstein and merged after review from @jasmussen, @Mamaduka, and others. The PR initially prototyped against `RichText` from `@wordpress/block-editor`, but @Mamaduka steered the author toward a decoupled approach to avoid block-editor selection coupling. This led to building on #78471, which introduced a standalone `RichTextControl` assembly in `@wordpress/dataviews`. The final diff exposes this assembly via private APIs and integrates it into the collab sidebar, resolving focus leaks and canvas selection shifts. Co-authored by @talldan, @peterwilsoncc, @scruffian, @Mamaduka, @jasmussen, and @jeffpaul.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
