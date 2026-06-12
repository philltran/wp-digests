# #79060: Editor: Hide cmd palette shortcut in document bar when admin bar is shown

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @fushar
- **Labels:** `[Type] Enhancement`, `[Package] Editor`
- **Merged:** [`3427bb2`](https://github.com/WordPress/gutenberg/commit/3427bb26bbc837438fe8d9983d8f22bed3ea8921)
- **Discussion:** [#79060](https://github.com/WordPress/gutenberg/pull/79060) · 7 comments · 1 reactions

## Summary

The block editor's document bar now conditionally hides the ⌘K keyboard shortcut indicator when the experimental admin bar is active inside the editor. This prevents duplicate command palette buttons from appearing side-by-side when `window.__experimentalAdminBarInEditor` is enabled. The change isolates to a visual cleanup without altering the title's click behavior or other document bar interactions.

## Impact

['**Plugin/Theme Developers**: If you are customizing the document bar UI, note that `.editor-document-bar__shortcut` is now conditionally rendered based on `window.__experimentalAdminBarInEditor`. A new `.has-shortcut` class is applied to the parent container only when the shortcut is visible.', "**Editor Users**: No action required. The visibility toggle automatically adapts to the existing 'Toolbar in editor' experiment setting.", '**Breaking/Deprecation**: None. This is a defensive UI adjustment scoped to an experimental flag.']

## Technical details

The diff modifies `packages/editor/src/components/document-bar/index.js` and `packages/editor/src/components/document-bar/style.scss`.

In `index.js`, a `hasShortcut` boolean is computed as `! window.__experimentalAdminBarInEditor`. The `displayShortcut.primary('k')` span is now wrapped in `{ hasShortcut && ( ... ) }`, and the document bar root element receives the dynamic `'has-shortcut'` class via `clsx`.

In `style.scss`, the existing media query padding rule (`padding-left: $grid-unit-30` on medium breakpoints) is scoped to `.editor-document-bar.has-shortcut &`. This prevents unwanted left-padding/layout shift when the shortcut node is removed from the DOM.

**Before/After Pattern**:
```js
// Before: Always rendered
<span className="editor-document-bar__shortcut">
  { displayShortcut.primary( 'k' ) }
</span>

// After: Conditionally rendered based on experimental flag
{ hasShortcut && (
  <span className="editor-document-bar__shortcut">
    { displayShortcut.primary( 'k' ) }
  </span>
) }
```

## Contribution

PR #79060 was opened and merged by @fushar (co-authored by @arthur791004, @StevenDufresne, and @jasmussen). Initial review flagged that hiding the shortcut affected the clickable title container's behavior; the author deliberately scoped the change down to only hiding the shortcut icon for safety before merge. The commit follows up on the admin bar experiment implementation in #77964.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
