# #13697: RichText: Format boundaries without DOM

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Feature] Rich Text`, `[Package] Rich text`
- **Merged:** [`b5a1e0f`](https://github.com/WordPress/gutenberg/commit/b5a1e0f8b700e19243f691d9837dc1947496bd54)
- **Discussion:** [#13697](https://github.com/WordPress/gutenberg/pull/13697) · 33 comments · 8 reactions

## Summary

This PR removes TinyMCE as the backing engine for format boundaries in the `@wordpress/rich-text` package, replacing it with a pure JavaScript implementation that tracks boundaries in the data layer rather than the DOM. Previously, format-edge "sticky" cursor behaviour was only applied to formats backed by specific registered element tags; with this change every format receives consistent boundary logic. The shift resolves 10 documented editor bugs covering cursor placement, focus loss, and format toggling edge cases. TinyMCE is now limited exclusively to the Classic block.

## Impact

**Plugin & theme developers**
- **Breaking:** The CSS class on every `contenteditable` RichText node changes from `.editor-rich-text__tinymce` to `.editor-rich-text__editable`. Any custom editor stylesheet or block stylesheet targeting `.editor-rich-text__tinymce` will silently stop applying.
- **Breaking:** The CSS attribute used to mark the cursor-adjacent boundary element changes from `[data-mce-selected]` to `[data-rich-text-format-boundary]`. Update selectors such as `a[data-mce-selected]` in editor SCSS accordingly (see `cover/editor.scss` and `gallery/editor.scss` for the canonical pattern).
- The padding `<br>` injected into empty editable regions previously carried `data-mce-bogus="1"`; it now carries `data-rich-text-padding="true"`. Any code that queries or filters on `data-mce-bogus` in block edit output will need updating.

**Platform / build tooling**
- `wp-tinymce` is removed as a declared dependency of both `wp-dom` and `wp-editor` in `lib/packages-dependencies.php`. Environments that infer the script dependency graph from that file should account for the removal.
- `tinymce@4.7.2` is dropped from the monorepo's `package-lock.json` and the `@wordpress/editor` package manifest, reducing the JavaScript bundle.

**Site owners / content editors**
- No action required. The change is internal to the block editor UI.

## Technical details

**CSS class rename — `.editor-rich-text__tinymce` → `.editor-rich-text__editable`**

Every `contenteditable` node rendered by the `RichText` component previously received the class `editor-rich-text__tinymce`. The diff replaces this across all core block SCSS files and block snapshot tests:

```scss
/* before */
.editor-rich-text__tinymce { … }

/* after */
[contenteditable] { … }           /* generic targeting in button/editor.scss */
.editor-rich-text__editable { … } /* class-based targeting elsewhere */
```

**Format boundary attribute — `data-mce-selected` → `data-rich-text-format-boundary`**

The boundary element (the inline element at the edge of a formatted range that receives focus) was previously identified via TinyMCE's `data-mce-selected` DOM attribute. It is now identified by `data-rich-text-format-boundary`:

```scss
/* before (cover/editor.scss) */
.editor-rich-text__tinymce:focus a[data-mce-selected] { box-shadow: none; }

/* after */
.editor-rich-text__editable:focus a[data-rich-text-format-boundary] { box-shadow: none; }
```

**Padding `<br>` attribute — `data-mce-bogus="1"` → `data-rich-text-padding="true"`**

TinyMCE injected a `<br data-mce-bogus="1">` into empty blocks to hold cursor position. The new implementation uses `<br data-rich-text-padding="true">` for the same purpose, as visible in every updated block snapshot.

**Dependency removal (`lib/packages-dependencies.php`)**

`wp-tinymce` is removed from the explicit dependency arrays of both `wp-dom` and `wp-editor`. The `tinymce` npm package is removed entirely from the editor's `package.json` / lock file.

**List block SCSS selector change**

The old selectors scoped list indentation through `.editor-rich-text__tinymce`; they are replaced with direct element selectors scoped to `.editor-styles-wrapper`:

```scss
/* before */
.block-library-list .editor-rich-text__tinymce,
.block-library-list .editor-rich-text__tinymce ul,
.block-library-list .editor-rich-text__tinymce ol { padding-left: 1.3em; }

/* after */
.editor-styles-wrapper .block-library-list ul,
.editor-styles-wrapper .block-library-list ol { padding-left: 1.3em; }
```

**Pullquote placeholder rules removed**

Several `[data-is-empty="true"]::before` placeholder-styling rules that targeted `.editor-rich-text__tinymce` in `pullquote/editor.scss` are deleted outright; placeholder rendering is now handled elsewhere in the component.

## Contribution

Authored entirely by @ellatrix (Ella van Durpe) and merged at commit `b5a1e0f`. During review, @youknowriad identified a focus-loss regression when clicking a toolbar format button (addressed before merge) and raised a question about inline image resizing, which @ellatrix resolved in a separate concurrent PR. @youknowriad also flagged a list-block margin difference from `master`; @jasmussen attributed it to an independent list-block style change in #12941 rather than this PR. @hypest was cc'd to notify the Gutenberg Mobile team. The PR carried 33 comments and closed 10 pre-existing issues.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
