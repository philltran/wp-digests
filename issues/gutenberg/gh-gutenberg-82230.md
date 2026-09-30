# #82230: Background image: support setting the image from a URL

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Feature] Design Tools`
- **Merged:** [`bfb6e57`](https://github.com/WordPress/gutenberg/commit/bfb6e57a74fcc92c072b0188737fb72f68ec9bf0)
- **Discussion:** [#82230](https://github.com/WordPress/gutenberg/pull/82230) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The background image control's media replace popover now includes a URL field, so users can set a background image from an externally hosted address instead of only picking from the Media Library or uploading. It reuses the `MediaReplaceFlow` built-in URL input (the same flow the Image block uses), which validates input and shows inline errors. The chosen value is stored as `{ url, source: 'url' }` with no attachment `id`.

## Impact

- **Site owners / editors:** Can paste an image URL into Styles → Background → Image, in both the block inspector and Global Styles. Invalid input shows an inline validation error. Submitting an empty field does not remove the current image.
- **Plugin & theme developers:** No API changes. Background image values may now carry `source: 'url'` with no `id`, so code that reads `style.background.backgroundImage` and assumes an attachment `id` should tolerate its absence. Externally hosted images bring the usual concerns of availability, hotlinking, and CSP `img-src`/`style-src` rules.
- **theme.json authors:** No schema change. The editor now handles a plain string `backgroundImage` when populating the URL field, but only absolute `http(s)` URLs are shown there.
- **Known gaps:** Theme-relative values (`file:./…`) are not shown in the URL field. The PR notes that relative URLs in theme.json are not displayed in Global Styles controls on trunk, tracked separately in #82242. The "Current media URL:" label is a fixed string inside `MediaReplaceFlow`, and the field styling is left for follow-up.
- **Headless / REST consumers:** No action required.

## Technical details

Changes are in `packages/block-editor/src/components/background-image-control/index.jsx` and `style.scss`, plus a CHANGELOG entry.

**Behavior**
- `BackgroundImageControls` now passes `onSelectURL` to `MediaReplaceFlow`, which enables its URL field.
- The new `onSelectURL( newURL )` handler returns early if `newURL` is empty or equal to the current `url`. This is what keeps clearing the field from removing the image. Otherwise it calls `onChange( setImmutably( style, [ 'background' ], { ...style?.background, backgroundImage: { url: newURL, source: 'url' } } ) )`.
- `mediaURL` passed to `MediaReplaceFlow` is now `currentURL` rather than `url`. It is computed from `rawURL`, which is `style.background.backgroundImage` when that is a string (theme.json allows a plain string) and the object's `url` otherwise.
- `currentURL` is `rawURL` only if it matches `/^https?:\/\//`, and is otherwise `undefined`. This excludes theme-relative `file:./…` values and the `'none'` sentinel stored when an inherited image is removed.

**Styles**
- `.block-editor-global-styles-background-panel__media-replace-popover .components-popover__content` changes from a fixed `width: 226px` to `width: auto`, so the popover is sized by the URL field (`LinkControl` min-width).
- The padding rule is narrowed from `.components-button` to `.components-menu-item__button.components-button`, so the URL field's buttons keep their own padding.

```js
// stored value
backgroundImage: { url: 'https://example.com/img.jpg', source: 'url' }
```

## Contribution

This is an alternative to #77286, reworked after review feedback there, and is part of the broader background image tracking issue #54336. Rather than a custom input, it reuses the existing `MediaReplaceFlow` URL flow. Review from @jasmussen was positive, with the input's visual polish and the fixed "Current media URL:" label deferred as separate work. A reviewer's question about theme-relative URLs led to the `http(s)`-only regex, and testing that surfaced a separate trunk bug with relative URLs in Global Styles, fixed in #82242.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
