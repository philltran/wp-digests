# #80440: Replaces the `title` attributes used by revision inline diff annotations with `aria-describedby`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shail-mehta
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Package] Editor`, `[Feature] History`, `Backport to Gutenberg RC`, `Backported to WP Core`
- **Merged:** [`bd699da`](https://github.com/WordPress/gutenberg/commit/bd699da6cd5e556fd6aecb62cf50a5b3d4243b42)
- **Discussion:** [#80440](https://github.com/WordPress/gutenberg/pull/80440) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg editor replaces `title` attributes on revision inline diff markers with `aria-describedby` to improve screen reader compatibility. This ensures that added, removed, and format-changed text is reliably announced in low-verbosity or text-only reading modes where `title` is often ignored. The change also standardizes format-change markers to use the `<mark>` element and shifts from dynamic per-instance descriptions to a fixed set of shared visually hidden descriptions.

## Impact

- **Editor users & accessibility testers:** Inline diff annotations in the Post Revisions interface are now consistently conveyed to assistive technologies.
- **Plugin & theme developers:** No public API changes or deprecations. The modification is internal to the `@wordpress/editor` package.
- **Custom CSS authors:** Format-change markers now render as `<mark>` instead of `<span>`. If you target `.revision-diff-format-added`, `.revision-diff-format-removed`, or `.revision-diff-format-changed` with styles that assume `<span>` defaults, you may need to explicitly reset or override UA `<mark>` styling.
- **No action required** for standard editor usage or existing block/theme code.

## Technical details

The change touches three files in `packages/editor/src/components/post-revisions-preview/`:
- `diff-format-types.js`: Exports `DIFF_DESCRIPTION_IDS` mapping to static IDs (`revision-diff-removed-desc`, `revision-diff-added-desc`, `revision-diff-format-added-desc`, `revision-diff-format-removed-desc`, `revision-diff-format-changed-desc`). Changes the `tagName` for format markers from `span` to `mark` and updates the registered format attributes to use `'aria-describedby': 'aria-describedby'`.
- `block-diff.js`: Replaces `attributes: { title: __( 'Removed' ) }` (and similar) with `attributes: { 'aria-describedby': DIFF_DESCRIPTION_IDS.removed }` on `<del>`, `<ins>`, and format markers. Drops the dynamic `description` string for format changes to keep `aria-describedby` references static, as required by the spec.
- `revisions-canvas.js`: Adds a `DiffDescriptions` component that renders five `<span>` elements wrapped in `<VisuallyHidden>` inside the canvas iframe. Includes CSS to reset default `<mark>` styling so visual appearance remains unchanged.

Before/after example for a removed text marker:
```html
<!-- Before -->
<del title="Removed" class="revision-diff-removed">text</del>

<!-- After -->
<del aria-describedby="revision-diff-removed-desc" class="revision-diff-removed">text</del>
```

## Contribution

Opened by @shail-mehta as part of issue #77530, with review and co-authorship from @joedolson and @ellatrix. During review, @joedolson tested `aria-description` as a simpler single-attribute alternative but found it unsupported in JAWS and VoiceOver, leading to the `aria-describedby` approach. The team also debated whether to keep dynamic format-change counts (e.g., "2 formats changed") but agreed to drop them to avoid complex per-instance DOM generation, keeping the implementation lightweight and spec-compliant.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
