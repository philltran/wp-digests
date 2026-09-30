# #64208: TextareaAutosize: Replace with native textareas and `field-sizing`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tyxla
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`
- **Merged:** [`1a05c62`](https://github.com/WordPress/gutenberg/commit/1a05c6289befa2534a884d780f9830ee0abf5688)
- **Discussion:** [#64208](https://github.com/WordPress/gutenberg/pull/64208) · 25 comments · 3 reactions
- **Usefulness:** 3/5

## Summary

Gutenberg drops the unmaintained `react-autosize-textarea` dependency and its local `patch-package` patch. The auto-growing textareas in `PlainText`, the block "Edit as HTML" field, and the post editor's Code editor view are now native `<textarea>` elements sized with CSS `field-sizing: content`. Browsers without `field-sizing` support fall back to native textarea sizing with scrolling.

## Impact

**Plugin & theme developers**
- If you use `PlainText` from `@wordpress/block-editor`, `rows` and `cols` no longer determine size in browsers that support `field-sizing: content`. Use CSS `min-height` / `max-height` to constrain height instead.
- `PlainText` keeps its `onChange( value )` signature and still renders a `textarea`. The forwarded `ref` now points directly to the native `textarea` element, not a `TextareaAutosize` component.
- The `async` prop was removed from the HTML block modal editors. Any custom code that relied on the autosize component's props (e.g. `onResize`, `async`) will no longer have them honoured.
- Code that targets `.block-editor-plain-text` gets a new shared stylesheet rule (`width: 100%; field-sizing: content`), so overrides may need adjusting.

**Site owners / editor users**
- Fields still grow and shrink with content in browsers that support `field-sizing` (Safari 26.2+, Firefox 152+, Chromium). In older browsers they no longer auto-grow and instead keep native size and scroll.

**Build / tooling**
- `react-autosize-textarea` and its transitive dependencies (`autosize`, `line-height`, `computed-style`) are removed from `@wordpress/block-editor` and `@wordpress/editor`. The bundle shrinks (about -2.7 kB for block-editor and -3.1 kB for editor scripts), offset by a small CSS increase.
- No action is required unless you depend on the removed package transitively or set `rows` on `PlainText`.

## Technical details

**Component swaps** (all `TextareaAutosize` / `Textarea` from `react-autosize-textarea` replaced by `<textarea>`):
- `packages/block-editor/src/components/plain-text/index.jsx`
- `packages/block-editor/src/components/block-list/block-html.jsx`
- `packages/editor/src/components/post-text-editor/index.jsx`

The `async` prop was also removed from the three editors in `packages/block-library/src/html/modal.jsx`.

**Styles**
- `block-list/content.scss` (`.block-editor-block-list__block-html-textarea`) and `post-text-editor/style.scss` (`textarea.editor-post-text-editor`) gain `field-sizing: content` and change `overflow: hidden` to `overflow: auto`, so long content stays scrollable where the property is unsupported.
- A new `plain-text/style.scss` provides `.block-editor-plain-text { width: 100%; field-sizing: content; }`. It is imported in both `src/style.scss` and `src/content.scss`, so the rule applies in the shared stylesheet and in the canvas stylesheet. The `width: 100%` was removed from `plain-text/content.scss` to avoid duplication.

**Dependencies / patches**
- `react-autosize-textarea` removed from both `package.json` files and `package-lock.json` (along with `autosize`, `computed-style`, `line-height`).
- `patches/react-autosize-textarea+7.1.0.patch` and its `patches/README.md` entry are deleted. `patch-package` stays because `@arraypress/waveform-player` is still patched. That patch had worked around removed React pointer-capture event types and a CJS default-export incompatibility with Node ESM.

**Docs**: the `PlainText` README and JSDoc now say `ref` is forwarded to the `textarea` element and document the `field-sizing` behaviour. Changelog entries were added to `block-editor` and `editor`.

```css
/* height constraints now belong in CSS, not `rows` */
.my-plain-text {
	min-height: 4em;
	max-height: 20em;
}
```

## Contribution

Opened by @tyxla in August 2024 (fixing #39619), the PR sat for a long time waiting on browser support: @ciampo noted WebKit had implemented `field-sizing` and Firefox was still under discussion, Safari 26.2 shipped it, and Firefox 152 finally did, which @manzoorwanijk flagged on release day. An earlier version refactored the fields to `TextareaControl`, but that was dropped after review because the wrapper elements changed `PlainText` behaviour and needed compensating CSS; the refactor is left for a separate PR. @jsnajdr noted the package's lack of ESM exports and bundler-incompatible CJS export as another reason to remove it. A Stylelint upgrade landed on trunk in the meantime, removing the need for inline `property-no-unknown` disables. The author disclosed that Codex assisted with verification and review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
